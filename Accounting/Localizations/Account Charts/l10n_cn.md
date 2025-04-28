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

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2008-2008 凯源吕鑫 lvxin@gmail.com   <basic chart data>
#                         维智众源 oldrev@gmail.com  <states data>
# Copyright (C) 2012-2012 南京盈通 ccdos@intoerp.com <small business chart>
# Copyright (C) 2008-now  开阖软件 jeff@osbzr.com    < PM and LTS >
# Copyright (C) 2018-now  jeffery9@gmail.com

{
    'name': 'China - Accounting',
    'version': '1.9',
    'category': 'Accounting/Localizations/Account Charts',
    'author': 'www.openerp-china.org',
    'maintainer': 'jeff@osbzr.com',
    'website': 'http://openerp-china.org',
    'description': r"""
Includes the following data for the Chinese localization
========================================================

Account Type/科目类型

State Data/省份数据

    科目类型\会计科目表模板\增值税\辅助核算类别\管理会计凭证簿\财务会计凭证簿

    添加中文省份数据

    增加小企业会计科目表

    修改小企业会计科目表

    修改小企业会计税率

We added the option to print a voucher which will also 
print the amount in words (special Chinese characters for numbers)
correctly when the cn2an library is installed. (e.g. with pip3 install cn2an)
    """,
    'depends': ['base', 'account', 'l10n_multilang'],
    'data': [
        'data/account_tax_group_data.xml',
        'data/l10n_cn_chart_data.xml',
        'data/account.account.template.csv',
        'data/l10n_cn_chart_post_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_chart_template_data.xml',
        'views/account_move_view.xml',
        'views/account_report.xml',
        'views/report_voucher.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'post_init_hook': 'load_translations',
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","account_type","chart_template_id/id","reconcile"
"l10n_cn_1012","Other Monetary Funds","1012","asset_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1101","Transactional Financial Assets","1101","asset_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1121","Bills Receivable","1121","asset_receivable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_1122","Accounts Receivable","1122","asset_receivable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_1123","Advance Payment","1123","asset_prepayments","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1124","Accounts Receivable (PoS)","1124","asset_receivable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_1131","Divident Receivable","1131","asset_receivable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_1132","Interest Receivable","1132","asset_receivable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_1221","Other Receivable","1221","asset_receivable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_1231","Bad Debt Provisions","1231","asset_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1401","Material Purchasing","1401","asset_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1402","Materials in transit","1402","asset_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1403","Raw Material","1403","asset_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1404","Material Cost Variance","1404","asset_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1405","Merchandise Inventory","1405","asset_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1406","Goods shipped in transit","1406","asset_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1407","Differences between purchasing and selling price","1407","asset_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1408","Consigned processing materials","1408","asset_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1471","Inventory falling price reserves","1471","asset_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1501","Held to maturity Investment","1501","asset_non_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1502","Provision for impairment of investments held to maturity","1502","asset_non_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1503","Available for sale financial assets","1503","asset_non_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1511","Long-term equity investment","1511","asset_non_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1512","Impairment provision for long-term equity investments","1512","asset_non_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1521","Investmental real estate","1521","asset_non_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1531","Long-term receivables","1531","asset_non_current","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_1601","Fixed assets","1601","asset_fixed","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1602","Accumulated depreciation","1602","expense_depreciation","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1603","Fixed assets depreciation reserves","1603","expense_depreciation","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1604","Construction in progress","1604","asset_non_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1605","Engineering materials","1605","asset_non_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1606","Liquidation of fixed assets","1606","expense_depreciation","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1701","Intangible Assets","1701","asset_non_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1702","Accumulated amortization","1702","expense_depreciation","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1703","Intangible Assets Depreciation Reserves","1703","expense_depreciation","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1711","Goodwill","1711","asset_non_current","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_1801","Long-term amortized expenses","1801","expense_depreciation","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_2001","Short-term borrowing","2001","liability_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2201","Bills Payable","2201","liability_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2202","Accounts Payable","2202","liability_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2203","Deposit Received","2203","liability_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2211","Payroll payable","2211","liability_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2221","Tax payable","2221","liability_current","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2231","Interest payable","2231","liability_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2241","Dividents payable","2241","liability_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2501","Other payable","2501","liability_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2502","Bonds Payable","2502","liability_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2701","Long Term payables","2701","liability_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2711","Account payable special funds","2711","liability_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2801","Projected liabilities","2801","liability_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_2901","Deferred Tax Liability","2901","liability_payable","l10n_cn.l10n_chart_china_small_business","True"
"l10n_cn_4001","Paid in capital","4001","equity","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_4002","Capital Surplus","4002","equity","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_4003","Other Comprehensive Income","4003","equity","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_4101","Surplus Reserve","4101","equity","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_4103","Profit for the year","4103","equity","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_4104","Profit distribution","4104","equity","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_5001","Production Costs","5001","expense_direct_cost","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_5101","Manufacturing Expenses","5101","expense_direct_cost","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_5201","Service Cost","5201","expense_direct_cost","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_5301","R & D expenditure","5301","expense_direct_cost","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6001","Main Business Income","6001","income","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6051","Other Business Income","6051","income_other","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6101","Gains and Losses of fair value change","6101","income_other","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6111","Income from investment","6111","income_other","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6301","Non-operating Income","6301","income_other","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6401","Main Business Cost","6401","expense","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6402","Other Operating Costs","6402","expense","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6403","Operating Taxes and Surcharges","6403","expense","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6601","Selling Expenses","6601","expense","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6602","Management Expenses","6602","expense","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6603","Financial Expenses","6603","expense","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6701","Assets impairment Loss","6701","expense","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6711","Non-operating expenses","6711","expense","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6801","Income Tax Expense","6801","expense","l10n_cn.l10n_chart_china_small_business","False"
"l10n_cn_6901","Prior year income adjustment","6901","expense","l10n_cn.l10n_chart_china_small_business","False"

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
        <field name="country_id" ref="base.cn"/>
    </record>
    <record id="l10n_cn_tax_group_vat_9" model="account.tax.group">
        <field name="name">VAT 9%</field>
        <field name="country_id" ref="base.cn"/>
    </record>
    <record id="l10n_cn_tax_group_vat_13" model="account.tax.group">
        <field name="name">VAT 13%</field>
        <field name="country_id" ref="base.cn"/>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_cn_2221'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
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
            <field name="spoken_languages" eval="'en_US;zh_CN'"/>
            <field name="country_id" ref="base.cn"/>
            <field name="use_storno_accounting" eval="True"/>
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

## File: migrations\1.9\pre-migrate.py

```python
def migrate(cr, version):
    # Set noupdate property of "account.tax.template" records to False
    cr.execute(
        """UPDATE ir_model_data
              SET noupdate=false
            WHERE module='l10n_cn'
              AND model='account.tax.template'
        """
    )

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

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="6.75" width="50.4" height="36.43" maskUnits="userSpaceOnUse">
      <rect x="6.06" y="7.09" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
    <rect x="6.01" y="10.57" width="48.45" height="31.57" rx="1" style="fill: #393939;opacity: 0.44;isolation: isolate"/>
    <g style="mask: url(#b)">
      <image width="640" height="427" transform="translate(4.8 6.75) scale(0.08 0.09)" xlink:href="data:image/png;base64,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AQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUBkBCABQGQEIAFAZAQgAUJlGq9UqbQAAGCH/HyLbjSpeB5wGAAAAAElFTkSuQmCC"/>
    </g>
  </g>
</svg>

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

