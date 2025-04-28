# Odoo Module: l10n_cn_standard

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2008-2008 凯源吕鑫 lvxin@gmail.com   <basic chart data>
#                         维智众源 oldrev@gmail.com  <states data>
# Copyright (C) 2012-2012 南京盈通 ccdos@intoerp.com <small business chart>
# Copyright (C) 2008-now  开阖软件 jeff@osbzr.com    < PM and LTS >
# Copyright (C) 2017-now  jeffery9@gmail.com

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2008-2008 凯源吕鑫 lvxin@gmail.com   <basic chart data>
#                         维智众源 oldrev@gmail.com  <states data>
# Copyright (C) 2012-2012 南京盈通 ccdos@intoerp.com <small business chart>
# Copyright (C) 2008-now  开阖软件 jeff@osbzr.com    < PM and LTS >
# Copyright (C) 2017-now  jeffery9@gmail.com

{
    'name': 'China - Standard CoA',
    'version': '2.0',
    'category': 'Localization',
    'author': ['lvxin@gmail.co', 'oldrev@gmail.co', 'ccdos@intoerp.com', 'jeff@osbzr.com', 'jeffery9@gmail.com'],

    'website': 'http://shine-it.net',
    'description': """
Including the following data in the Accounting Standards for Business Enterprises
包含企业会计准则以下数据

* Chart of Accounts
* 科目表模板

* Account templates
* 科目模板

* Tax templates
* 税金模板

    """,
    'depends': ['l10n_cn'],
    'data': [
        'data/l10n_cn_standard_chart_data.xml',
        'data/account.account.template.csv',
        'data/account_tax_template_data.xml',
        'data/account_chart_template_data.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,code,name,reconcile,user_type_id/id,chart_template_id/id
account_1011,1011,存放同业,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1012,1012,其他货币资金,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1021,1021,结算备付金,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1031,1031,存出保证金,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1101,1101,交易性金融资产,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1111,1111,买入返售金融资产,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1121,1121,应收票据,TRUE,account.data_account_type_receivable,l10n_chart_china_standard_business
account_1122,1122,应收账款,TRUE,account.data_account_type_receivable,l10n_chart_china_standard_business
account_1123,1123,预付账款,TRUE,account.data_account_type_prepayments,l10n_chart_china_standard_business
account_1124,1124,应收账款 (PoS),TRUE,account.data_account_type_receivable,l10n_chart_china_standard_business
account_1131,1131,应收股利,TRUE,account.data_account_type_receivable,l10n_chart_china_standard_business
account_1132,1132,应收利息,TRUE,account.data_account_type_receivable,l10n_chart_china_standard_business
account_1201,1201,应收代位追偿款,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1211,1211,应收分保账款,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1212,1212,应收分保合同准备金,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1221,1221,其他应收款,TRUE,account.data_account_type_receivable,l10n_chart_china_standard_business
account_1231,1231,坏账准备,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1301,1301,贴现资产,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1302,1302,拆出资金,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1303,1303,贷款,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1304,1304,贷款损失准备,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1311,1311,代理兑付证券,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1321,1321,代理业务资产,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1401,1401,材料采购,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1402,1402,在途物资,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1403,1403,原材料,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1404,1404,材料成本差异,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1405,1405,库存商品,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1406,1406,发出商品,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1407,1407,商品进销差价,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1408,1408,委托加工物资,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1411,1411,周转材料,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1421,1421,消耗性生物资产,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1431,1431,贵金属,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1441,1441,抵债资产,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1451,1451,损余物资,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1461,1461,融资租赁资产,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1471,1471,存货跌价准备,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1501,1501,持有至到期投资,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1502,1502,持有至到期投资减值准备,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1503,1503,可供出售金融资产,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1511,1511,长期股权投资,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1512,1512,长期股权投资减值准备,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1521,1521,投资性房地产,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1531,1531,长期应收款,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1532,1532,未实现融资收益,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1541,1541,存出资本保证金,FALSE,account.data_account_type_non_current_assets,l10n_chart_china_standard_business
account_1601,1601,固定资产,FALSE,account.data_account_type_fixed_assets,l10n_chart_china_standard_business
account_1602,1602,累计折旧,FALSE,account.data_account_type_fixed_assets,l10n_chart_china_standard_business
account_1603,1603,固定资产减值准备,FALSE,account.data_account_type_fixed_assets,l10n_chart_china_standard_business
account_1604,1604,在建工程,FALSE,account.data_account_type_fixed_assets,l10n_chart_china_standard_business
account_1605,1605,工程物资,FALSE,account.data_account_type_fixed_assets,l10n_chart_china_standard_business
account_1606,1606,固定资产清理,FALSE,account.data_account_type_fixed_assets,l10n_chart_china_standard_business
account_1611,1611,未担保余值,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1621,1621,生产性生物资产,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1622,1622,生产性生物资产累计折旧,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1623,1623,公益性生物资产,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1631,1631,油气资产,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1632,1632,累计折耗,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1701,1701,无形资产,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1702,1702,累计摊销,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1703,1703,无形资产减值准备,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1711,1711,商誉,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1801,1801,长期待摊费用,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1811,1811,递延所得税资产,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1821,1821,独立账户资产,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_1901,1901,待处理财产损溢,FALSE,account.data_account_type_current_assets,l10n_chart_china_standard_business
account_2001,2001,短期借款,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2002,2002,存入保证金,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2003,2003,拆入资金,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2004,2004,向中央银行借款,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2011,2011,吸收存款,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2012,2012,同业存放,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2021,2021,贴现负债,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2101,2101,交易性金融负债,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2111,2111,卖出回购金融资产款,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2201,2201,应付票据,TRUE,account.data_account_type_payable,l10n_chart_china_standard_business
account_2202,2202,应付账款,TRUE,account.data_account_type_payable,l10n_chart_china_standard_business
account_2203,2203,预收账款,TRUE,account.data_account_type_payable,l10n_chart_china_standard_business
account_2211,2211,应付职工薪酬,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221,2221,应交税费,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_1_1,2221.01.01,进项税额,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_1_2,2221.01.02,已交税金,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_1_3,2221.01.03,转出未交增值税,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_1_4,2221.01.04,减免税款,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_1_5,2221.01.05,销项税额,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_1_6,2221.01.06,出口退税,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_1_7,2221.01.07,进项税额转出,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_1_8,2221.01.08,出口抵减内销产品应纳税额,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_1_9,2221.01.09,转出多交增值税,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_1_10,2221.01.10,未交增值税,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_2,2221.02,应交营业税,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_3,2221.03,应交消费税,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_4,2221.04,应交资源税,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_5,2221.05,应交所得税,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_6,2221.06,应交土地增值税,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_7,2221.07,应交城市维护建设税,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_8,2221.08,应交房产税,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_9,2221.09,应交土地使用税,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_10,2221.10,应交车船使用税,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2221_11,2221.11,应交个人所得税,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2231,2231,应付利息,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2232,2232,应付股利,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2241,2241,其他应付款,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2251,2251,应付保单红利,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2261,2261,应付分保账款,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2311,2311,代理买卖证券款,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2312,2312,代理承销证券款,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2313,2313,代理兑付证券款,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2314,2314,代理业务负债,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2401,2401,递延收益,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2501,2501,长期借款,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2502,2502,应付债券,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2601,2601,未到期责任准备金,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2602,2602,保险责任准备金,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2611,2611,保户储金,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2621,2621,独立账户负债,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2701,2701,长期应付款,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2702,2702,未确认融资费用,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2711,2711,专项应付款,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2801,2801,预计负债,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_2901,2901,递延所得税负债,FALSE,account.data_account_type_current_liabilities,l10n_chart_china_standard_business
account_3001,3001,清算资金往来,FALSE,account.data_account_off_sheet,l10n_chart_china_standard_business
account_3002,3002,货币兑换,FALSE,account.data_account_off_sheet,l10n_chart_china_standard_business
account_3101,3101,衍生工具,FALSE,account.data_account_off_sheet,l10n_chart_china_standard_business
account_3201,3201,套期工具,FALSE,account.data_account_off_sheet,l10n_chart_china_standard_business
account_3202,3202,被套期项目,FALSE,account.data_account_off_sheet,l10n_chart_china_standard_business
account_4001,4001,实收资本,FALSE,account.data_account_type_equity,l10n_chart_china_standard_business
account_4002,4002,资本公积,FALSE,account.data_account_type_equity,l10n_chart_china_standard_business
account_4101,4101,盈余公积,FALSE,account.data_account_type_equity,l10n_chart_china_standard_business
account_4102,4102,一般风险准备,FALSE,account.data_account_type_equity,l10n_chart_china_standard_business
account_4103,4103,本年利润,FALSE,account.data_account_type_equity,l10n_chart_china_standard_business
account_4104,4104,利润分配,FALSE,account.data_account_type_equity,l10n_chart_china_standard_business
account_4201,4201,库存股,FALSE,account.data_account_type_equity,l10n_chart_china_standard_business
account_5001,5001,生产成本,FALSE,account.data_account_off_sheet,l10n_chart_china_standard_business
account_5101,5101,制造费用,FALSE,account.data_account_off_sheet,l10n_chart_china_standard_business
account_5201,5201,劳务成本,FALSE,account.data_account_off_sheet,l10n_chart_china_standard_business
account_5301,5301,研发支出,FALSE,account.data_account_off_sheet,l10n_chart_china_standard_business
account_5401,5401,工程施工,FALSE,account.data_account_off_sheet,l10n_chart_china_standard_business
account_5402,5402,工程结算,FALSE,account.data_account_off_sheet,l10n_chart_china_standard_business
account_5403,5403,机械作业,FALSE,account.data_account_off_sheet,l10n_chart_china_standard_business
account_6001,6001,主营业务收入,FALSE,account.data_account_type_revenue,l10n_chart_china_standard_business
account_6011,6011,利息收入,FALSE,account.data_account_type_revenue,l10n_chart_china_standard_business
account_6021,6021,手续费及佣金收入,FALSE,account.data_account_type_revenue,l10n_chart_china_standard_business
account_6031,6031,保费收入,FALSE,account.data_account_type_revenue,l10n_chart_china_standard_business
account_6041,6041,租赁收入,FALSE,account.data_account_type_revenue,l10n_chart_china_standard_business
account_6051,6051,其他业务收入,FALSE,account.data_account_type_revenue,l10n_chart_china_standard_business
account_6061,6061,汇兑损益,FALSE,account.data_account_type_revenue,l10n_chart_china_standard_business
account_6101,6101,公允价值变动损益,FALSE,account.data_account_type_revenue,l10n_chart_china_standard_business
account_6111,6111,投资收益,FALSE,account.data_account_type_revenue,l10n_chart_china_standard_business
account_6201,6201,摊回保险责任准备金,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6202,6202,摊回赔付支出,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6203,6203,摊回分保费用,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6301,6301,营业外收入,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6401,6401,主营业务成本,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6402,6402,其他业务成本,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6403,6403,营业税金及附加,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6411,6411,利息支出,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6421,6421,手续费及佣金支出,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6501,6501,提取未到期责任准备金,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6502,6502,提取保险责任准备金,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6511,6511,赔付支出,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6521,6521,保单红利支出,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6531,6531,退保金,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6541,6541,分出保费,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6542,6542,分保费用,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6601,6601,销售费用,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6602,6602,管理费用,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6603,6603,财务费用,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6604,6604,勘探费用,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6701,6701,资产减值损失,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6711,6711,营业外支出,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6801,6801,所得税费用,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
account_6901,6901,以前年度损益调整,FALSE,account.data_account_type_expenses,l10n_chart_china_standard_business
```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <data>
        <record id="l10n_chart_china_standard_business" model="account.chart.template">
            <field name="property_account_receivable_id" ref="account_1122" />
            <field name="property_account_payable_id" ref="account_2202" />
            <field name="property_account_expense_categ_id" ref="account_6401" />
            <field name="property_account_income_categ_id" ref="account_6001" />
            <field name="income_currency_exchange_account_id" ref="account_6061" />
            <field name="expense_currency_exchange_account_id" ref="account_6061" />
            <field name="default_pos_receivable_account_id" ref="account_1124" />
        </record>
    </data>

    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_cn_standard.l10n_chart_china_standard_business')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <!-- sales tax included -->
    <record id="l10n_cn_standard_sales_included_17" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_standard_business"/>
        <field name="name">税收17％（含） - 中国会计科目表-企业会计准则</field>
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
                'account_id': ref('account_2221_1_5'),
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
                'account_id': ref('account_2221_1_5'),
            }),
        ]"/>
    </record>
    <record id="l10n_cn_standard_sales_included_11" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_standard_business"/>
        <field name="name">税收11％（含） - 中国会计科目表-企业会计准则</field>
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
                'account_id': ref('account_2221_1_5'),
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
                'account_id': ref('account_2221_1_5'),
            }),
        ]"/>

    </record>
    <record id="l10n_cn_standard_sales_included_6" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_standard_business"/>
        <field name="name">税收6％（含） - 中国会计科目表-企业会计准则</field>
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
                'account_id': ref('account_2221_1_5'),
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
                'account_id': ref('account_2221_1_5'),
            }),
        ]"/>
    </record>
    <record id="l10n_cn_standard_sales_included_3" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_standard_business"/>
        <field name="name">税收3％（含） - 中国会计科目表-企业会计准则</field>
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
                'account_id': ref('account_2221_1_5'),
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
                'account_id': ref('account_2221_1_5'),
            }),
        ]"/>
    </record>

    <!-- sales tax excluded -->
    <record id="l10n_cn_standard_sales_excluded_17" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_standard_business"/>
        <field name="name">税收17％ - 中国会计科目表-企业会计准则</field>
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
                'account_id': ref('account_2221_1_5'),
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
                'account_id': ref('account_2221_1_5'),
            }),
        ]"/>
    </record>
    <record id="l10n_cn_standard_sales_excluded_11" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_standard_business"/>
        <field name="name">税收11％ - 中国会计科目表-企业会计准则</field>
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
                'account_id': ref('account_2221_1_5'),
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
                'account_id': ref('account_2221_1_5'),
            }),
        ]"/>
    </record>
    <record id="l10n_cn_standard_sales_excluded_6" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_standard_business"/>
        <field name="name">税收6％ - 中国会计科目表-企业会计准则</field>
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
                'account_id': ref('account_2221_1_5'),
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
                'account_id': ref('account_2221_1_5'),
            }),
        ]"/>
    </record>
    <record id="l10n_cn_standard_sales_excluded_small_3" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_standard_business"/>
        <field name="name">税收3％ - 中国会计科目表-企业会计准则</field>
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
                'account_id': ref('account_2221_1_5'),
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
                'account_id': ref('account_2221_1_5'),
            }),
        ]"/>
    </record>


    <!-- purchase tax excluded -->
    <record id="l10n_cn_standard_purchase_excluded_17" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_standard_business"/>
        <field name="name">税收17％ - 中国会计科目表-企业会计准则</field>
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
                'account_id': ref('account_2221_1_1'),
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
                'account_id': ref('account_2221_1_1'),
            }),
        ]"/>
    </record>
    <record id="l10n_cn_standard_purchase_excluded_11" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_standard_business"/>
        <field name="name">税收11％ - 中国会计科目表-企业会计准则</field>
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
                'account_id': ref('account_2221_1_1'),
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
                'account_id': ref('account_2221_1_1'),
            }),
        ]"/>
    </record>
    <record id="l10n_cn_standard_purchase_excluded_6" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_standard_business"/>
        <field name="name">税收6％ - 中国会计科目表-企业会计准则</field>
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
                'account_id': ref('account_2221_1_1'),
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
                'account_id': ref('account_2221_1_1'),
            }),
        ]"/>
    </record>
    <record id="l10n_cn_standard_purchase_excluded_3" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_chart_china_standard_business"/>
        <field name="name">税收3％ - 中国会计科目表-企业会计准则</field>
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
                'account_id': ref('account_2221_1_1'),
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
                'account_id': ref('account_2221_1_1'),
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\l10n_cn_standard_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <data>
        <!-- Chart template -->
        <record id="l10n_chart_china_standard_business" model="account.chart.template">
            <field name="name">中国会计科目表 （财会[2006]3号《企业会计准则》</field>
            <field name="code_digits" eval="6" />
            <field name="currency_id" ref="base.CNY" />
            <field name="cash_account_code_prefix">1001</field>
            <field name="bank_account_code_prefix">1002</field>
            <field name="transfer_account_code_prefix">1003</field>
        </record>
    </data>
</odoo>

```

## File: i18n_extra\l10n_cn_standard.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
#	* l10n_cn_standard
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 12.0alpha1+e\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2017-11-30 13:11+0000\n"
"PO-Revision-Date: 2017-11-30 13:11+0000\n"
"Last-Translator: <>\n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_4102
#: model:account.account.template,name:l10n_cn_standard.account_4102
msgid "一般风险准备"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2711
#: model:account.account.template,name:l10n_cn_standard.account_2711
msgid "专项应付款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.chart.template,name:l10n_cn_standard.l10n_chart_china_standard_business
msgid "中国会计科目表 （财会[2006]3号《企业会计准则》"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6401
#: model:account.account.template,name:l10n_cn_standard.account_6401
msgid "主营业务成本"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6001
#: model:account.account.template,name:l10n_cn_standard.account_6001
msgid "主营业务收入"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1111
#: model:account.account.template,name:l10n_cn_standard.account_1111
msgid "买入返售金融资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2101
#: model:account.account.template,name:l10n_cn_standard.account_2101
msgid "交易性金融负债"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1101
#: model:account.account.template,name:l10n_cn_standard.account_1101
msgid "交易性金融资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2314
#: model:account.account.template,name:l10n_cn_standard.account_2314
msgid "代理业务负债"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1321
#: model:account.account.template,name:l10n_cn_standard.account_1321
msgid "代理业务资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2311
#: model:account.account.template,name:l10n_cn_standard.account_2311
msgid "代理买卖证券款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1311
#: model:account.account.template,name:l10n_cn_standard.account_1311
msgid "代理兑付证券"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2313
#: model:account.account.template,name:l10n_cn_standard.account_2313
msgid "代理兑付证券款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2312
#: model:account.account.template,name:l10n_cn_standard.account_2312
msgid "代理承销证券款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6901
#: model:account.account.template,name:l10n_cn_standard.account_6901
msgid "以前年度损益调整"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6521
#: model:account.account.template,name:l10n_cn_standard.account_6521
msgid "保单红利支出"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2611
#: model:account.account.template,name:l10n_cn_standard.account_2611
msgid "保户储金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6031
#: model:account.account.template,name:l10n_cn_standard.account_6031
msgid "保费收入"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2602
#: model:account.account.template,name:l10n_cn_standard.account_2602
msgid "保险责任准备金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6101
#: model:account.account.template,name:l10n_cn_standard.account_6101
msgid "公允价值变动损益"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1623
#: model:account.account.template,name:l10n_cn_standard.account_1623
msgid "公益性生物资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6402
#: model:account.account.template,name:l10n_cn_standard.account_6402
msgid "其他业务成本"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6051
#: model:account.account.template,name:l10n_cn_standard.account_6051
msgid "其他业务收入"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2241
#: model:account.account.template,name:l10n_cn_standard.account_2241
msgid "其他应付款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1221
#: model:account.account.template,name:l10n_cn_standard.account_1221
msgid "其他应收款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1012
#: model:account.account.template,name:l10n_cn_standard.account_1012
msgid "其他货币资金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_1_4
#: model:account.account.template,name:l10n_cn_standard.account_2221_1_4
msgid "减免税款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_1_8
#: model:account.account.template,name:l10n_cn_standard.account_2221_1_8
msgid "出口抵减内销产品应纳税额"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_1_6
#: model:account.account.template,name:l10n_cn_standard.account_2221_1_6
msgid "出口退税"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6542
#: model:account.account.template,name:l10n_cn_standard.account_6542
msgid "分保费用"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6541
#: model:account.account.template,name:l10n_cn_standard.account_6541
msgid "分出保费"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6411
#: model:account.account.template,name:l10n_cn_standard.account_6411
msgid "利息支出"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6011
#: model:account.account.template,name:l10n_cn_standard.account_6011
msgid "利息收入"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_4104
#: model:account.account.template,name:l10n_cn_standard.account_4104
msgid "利润分配"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_5101
#: model:account.account.template,name:l10n_cn_standard.account_5101
msgid "制造费用"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_5201
#: model:account.account.template,name:l10n_cn_standard.account_5201
msgid "劳务成本"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6604
#: model:account.account.template,name:l10n_cn_standard.account_6604
msgid "勘探费用"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2111
#: model:account.account.template,name:l10n_cn_standard.account_2111
msgid "卖出回购金融资产款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1403
#: model:account.account.template,name:l10n_cn_standard.account_1403
msgid "原材料"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1406
#: model:account.account.template,name:l10n_cn_standard.account_1406
msgid "发出商品"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1503
#: model:account.account.template,name:l10n_cn_standard.account_1503
msgid "可供出售金融资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2012
#: model:account.account.template,name:l10n_cn_standard.account_2012
msgid "同业存放"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2004
#: model:account.account.template,name:l10n_cn_standard.account_2004
msgid "向中央银行借款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2011
#: model:account.account.template,name:l10n_cn_standard.account_2011
msgid "吸收存款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1411
#: model:account.account.template,name:l10n_cn_standard.account_1411
msgid "周转材料"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1407
#: model:account.account.template,name:l10n_cn_standard.account_1407
msgid "商品进销差价"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1711
#: model:account.account.template,name:l10n_cn_standard.account_1711
msgid "商誉"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1601
#: model:account.account.template,name:l10n_cn_standard.account_1601
msgid "固定资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1603
#: model:account.account.template,name:l10n_cn_standard.account_1603
msgid "固定资产减值准备"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1606
#: model:account.account.template,name:l10n_cn_standard.account_1606
msgid "固定资产清理"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1604
#: model:account.account.template,name:l10n_cn_standard.account_1604
msgid "在建工程"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1402
#: model:account.account.template,name:l10n_cn_standard.account_1402
msgid "在途物资"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1231
#: model:account.account.template,name:l10n_cn_standard.account_1231
msgid "坏账准备"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,description:l10n_cn_standard.1_l10n_cn_standard_purchase_excluded_11
#: model:account.tax.template,description:l10n_cn_standard.l10n_cn_standard_purchase_excluded_11
msgid "税收11％"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,name:l10n_cn_standard.1_l10n_cn_standard_purchase_excluded_11
#: model:account.tax.template,name:l10n_cn_standard.l10n_cn_standard_purchase_excluded_11
msgid "税收11％ - 中国会计科目表-企业会计准则"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,description:l10n_cn_standard.1_l10n_cn_standard_sales_excluded_11
#: model:account.tax,description:l10n_cn_standard.1_l10n_cn_standard_sales_included_11
#: model:account.tax.template,description:l10n_cn_standard.l10n_cn_standard_sales_excluded_11
#: model:account.tax.template,description:l10n_cn_standard.l10n_cn_standard_sales_included_11
msgid "税收11％"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,name:l10n_cn_standard.1_l10n_cn_standard_sales_excluded_11
#: model:account.tax.template,name:l10n_cn_standard.l10n_cn_standard_sales_excluded_11
msgid "税收11％ - 中国会计科目表-企业会计准则"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,name:l10n_cn_standard.1_l10n_cn_standard_sales_included_11
#: model:account.tax.template,name:l10n_cn_standard.l10n_cn_standard_sales_included_11
msgid "税收11％（含） - 中国会计科目表-企业会计准则"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,description:l10n_cn_standard.1_l10n_cn_standard_purchase_excluded_17
#: model:account.tax.template,description:l10n_cn_standard.l10n_cn_standard_purchase_excluded_17
msgid "税收17％"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,name:l10n_cn_standard.1_l10n_cn_standard_purchase_excluded_17
#: model:account.tax.template,name:l10n_cn_standard.l10n_cn_standard_purchase_excluded_17
msgid "税收17％ - 中国会计科目表-企业会计准则"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,description:l10n_cn_standard.1_l10n_cn_standard_sales_excluded_17
#: model:account.tax,description:l10n_cn_standard.1_l10n_cn_standard_sales_included_17
#: model:account.tax.template,description:l10n_cn_standard.l10n_cn_standard_sales_excluded_17
#: model:account.tax.template,description:l10n_cn_standard.l10n_cn_standard_sales_included_17
msgid "税收17％"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,name:l10n_cn_standard.1_l10n_cn_standard_sales_excluded_17
#: model:account.tax.template,name:l10n_cn_standard.l10n_cn_standard_sales_excluded_17
msgid "税收17％ - 中国会计科目表-企业会计准则"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,name:l10n_cn_standard.1_l10n_cn_standard_sales_included_17
#: model:account.tax.template,name:l10n_cn_standard.l10n_cn_standard_sales_included_17
msgid "税收17％（含） - 中国会计科目表-企业会计准则"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,description:l10n_cn_standard.1_l10n_cn_standard_purchase_excluded_3
#: model:account.tax.template,description:l10n_cn_standard.l10n_cn_standard_purchase_excluded_3
msgid "税收3％"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,name:l10n_cn_standard.1_l10n_cn_standard_purchase_excluded_3
#: model:account.tax.template,name:l10n_cn_standard.l10n_cn_standard_purchase_excluded_3
msgid "税收3％ - 中国会计科目表-企业会计准则"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,description:l10n_cn_standard.1_l10n_cn_standard_sales_excluded_small_3
#: model:account.tax,description:l10n_cn_standard.1_l10n_cn_standard_sales_included_3
#: model:account.tax.template,description:l10n_cn_standard.l10n_cn_standard_sales_excluded_small_3
#: model:account.tax.template,description:l10n_cn_standard.l10n_cn_standard_sales_included_3
msgid "税收3％"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,name:l10n_cn_standard.1_l10n_cn_standard_sales_excluded_small_3
#: model:account.tax.template,name:l10n_cn_standard.l10n_cn_standard_sales_excluded_small_3
msgid "税收3％ - 中国会计科目表-企业会计准则"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,name:l10n_cn_standard.1_l10n_cn_standard_sales_included_3
#: model:account.tax.template,name:l10n_cn_standard.l10n_cn_standard_sales_included_3
msgid "税收3％（含） - 中国会计科目表-企业会计准则"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,description:l10n_cn_standard.1_l10n_cn_standard_purchase_excluded_6
#: model:account.tax.template,description:l10n_cn_standard.l10n_cn_standard_purchase_excluded_6
msgid "税收6％"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,name:l10n_cn_standard.1_l10n_cn_standard_purchase_excluded_6
#: model:account.tax.template,name:l10n_cn_standard.l10n_cn_standard_purchase_excluded_6
msgid "税收6％ - 中国会计科目表-企业会计准则"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,description:l10n_cn_standard.1_l10n_cn_standard_sales_excluded_6
#: model:account.tax,description:l10n_cn_standard.1_l10n_cn_standard_sales_included_6
#: model:account.tax.template,description:l10n_cn_standard.l10n_cn_standard_sales_excluded_6
#: model:account.tax.template,description:l10n_cn_standard.l10n_cn_standard_sales_included_6
msgid "税收6％"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,name:l10n_cn_standard.1_l10n_cn_standard_sales_excluded_6
#: model:account.tax.template,name:l10n_cn_standard.l10n_cn_standard_sales_excluded_6
msgid "税收6％ - 中国会计科目表-企业会计准则"
msgstr ""

#. module: l10n_cn_standard
#: model:account.tax,name:l10n_cn_standard.1_l10n_cn_standard_sales_included_6
#: model:account.tax.template,name:l10n_cn_standard.l10n_cn_standard_sales_included_6
msgid "税收6％（含） - 中国会计科目表-企业会计准则"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_3201
#: model:account.account.template,name:l10n_cn_standard.account_3201
msgid "套期工具"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1408
#: model:account.account.template,name:l10n_cn_standard.account_1408
msgid "委托加工物资"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2002
#: model:account.account.template,name:l10n_cn_standard.account_2002
msgid "存入保证金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1031
#: model:account.account.template,name:l10n_cn_standard.account_1031
msgid "存出保证金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1541
#: model:account.account.template,name:l10n_cn_standard.account_1541
msgid "存出资本保证金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1011
#: model:account.account.template,name:l10n_cn_standard.account_1011
msgid "存放同业"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1471
#: model:account.account.template,name:l10n_cn_standard.account_1471
msgid "存货跌价准备"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_4001
#: model:account.account.template,name:l10n_cn_standard.account_4001
msgid "实收资本"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_5401
#: model:account.account.template,name:l10n_cn_standard.account_5401
msgid "工程施工"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1605
#: model:account.account.template,name:l10n_cn_standard.account_1605
msgid "工程物资"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_5402
#: model:account.account.template,name:l10n_cn_standard.account_5402
msgid "工程结算"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_1_2
#: model:account.account.template,name:l10n_cn_standard.account_2221_1_2
msgid "已交税金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1405
#: model:account.account.template,name:l10n_cn_standard.account_1405
msgid "库存商品"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_4201
#: model:account.account.template,name:l10n_cn_standard.account_4201
msgid "库存股"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_11
#: model:account.account.template,name:l10n_cn_standard.account_2221_11
msgid "应交个人所得税"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_9
#: model:account.account.template,name:l10n_cn_standard.account_2221_9
msgid "应交土地使用税"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_6
#: model:account.account.template,name:l10n_cn_standard.account_2221_6
msgid "应交土地增值税"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_7
#: model:account.account.template,name:l10n_cn_standard.account_2221_7
msgid "应交城市维护建设税"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_8
#: model:account.account.template,name:l10n_cn_standard.account_2221_8
msgid "应交房产税"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_5
#: model:account.account.template,name:l10n_cn_standard.account_2221_5
msgid "应交所得税"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_3
#: model:account.account.template,name:l10n_cn_standard.account_2221_3
msgid "应交消费税"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221
#: model:account.account.template,name:l10n_cn_standard.account_2221
msgid "应交税费"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_2
#: model:account.account.template,name:l10n_cn_standard.account_2221_2
msgid "应交营业税"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_4
#: model:account.account.template,name:l10n_cn_standard.account_2221_4
msgid "应交资源税"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_10
#: model:account.account.template,name:l10n_cn_standard.account_2221_10
msgid "应交车船使用税"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2251
#: model:account.account.template,name:l10n_cn_standard.account_2251
msgid "应付保单红利"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2502
#: model:account.account.template,name:l10n_cn_standard.account_2502
msgid "应付债券"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2261
#: model:account.account.template,name:l10n_cn_standard.account_2261
msgid "应付分保账款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2231
#: model:account.account.template,name:l10n_cn_standard.account_2231
msgid "应付利息"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2201
#: model:account.account.template,name:l10n_cn_standard.account_2201
msgid "应付票据"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2211
#: model:account.account.template,name:l10n_cn_standard.account_2211
msgid "应付职工薪酬"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2232
#: model:account.account.template,name:l10n_cn_standard.account_2232
msgid "应付股利"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2202
#: model:account.account.template,name:l10n_cn_standard.account_2202
msgid "应付账款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1201
#: model:account.account.template,name:l10n_cn_standard.account_1201
msgid "应收代位追偿款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1212
#: model:account.account.template,name:l10n_cn_standard.account_1212
msgid "应收分保合同准备金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1211
#: model:account.account.template,name:l10n_cn_standard.account_1211
msgid "应收分保账款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1132
#: model:account.account.template,name:l10n_cn_standard.account_1132
msgid "应收利息"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1121
#: model:account.account.template,name:l10n_cn_standard.account_1121
msgid "应收票据"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1131
#: model:account.account.template,name:l10n_cn_standard.account_1131
msgid "应收股利"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1122
#: model:account.account.template,name:l10n_cn_standard.account_1122
msgid "应收账款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1901
#: model:account.account.template,name:l10n_cn_standard.account_1901
msgid "待处理财产损溢"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6801
#: model:account.account.template,name:l10n_cn_standard.account_6801
msgid "所得税费用"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6421
#: model:account.account.template,name:l10n_cn_standard.account_6421
msgid "手续费及佣金支出"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6021
#: model:account.account.template,name:l10n_cn_standard.account_6021
msgid "手续费及佣金收入"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1521
#: model:account.account.template,name:l10n_cn_standard.account_1521
msgid "投资性房地产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6111
#: model:account.account.template,name:l10n_cn_standard.account_6111
msgid "投资收益"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1441
#: model:account.account.template,name:l10n_cn_standard.account_1441
msgid "抵债资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2003
#: model:account.account.template,name:l10n_cn_standard.account_2003
msgid "拆入资金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1302
#: model:account.account.template,name:l10n_cn_standard.account_1302
msgid "拆出资金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1501
#: model:account.account.template,name:l10n_cn_standard.account_1501
msgid "持有至到期投资"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1502
#: model:account.account.template,name:l10n_cn_standard.account_1502
msgid "持有至到期投资减值准备"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1451
#: model:account.account.template,name:l10n_cn_standard.account_1451
msgid "损余物资"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6502
#: model:account.account.template,name:l10n_cn_standard.account_6502
msgid "提取保险责任准备金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6501
#: model:account.account.template,name:l10n_cn_standard.account_6501
msgid "提取未到期责任准备金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6201
#: model:account.account.template,name:l10n_cn_standard.account_6201
msgid "摊回保险责任准备金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6203
#: model:account.account.template,name:l10n_cn_standard.account_6203
msgid "摊回分保费用"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6202
#: model:account.account.template,name:l10n_cn_standard.account_6202
msgid "摊回赔付支出"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1701
#: model:account.account.template,name:l10n_cn_standard.account_1701
msgid "无形资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1703
#: model:account.account.template,name:l10n_cn_standard.account_1703
msgid "无形资产减值准备"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_1_10
#: model:account.account.template,name:l10n_cn_standard.account_2221_1_10
msgid "未交增值税"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2601
#: model:account.account.template,name:l10n_cn_standard.account_2601
msgid "未到期责任准备金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1532
#: model:account.account.template,name:l10n_cn_standard.account_1532
msgid "未实现融资收益"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1611
#: model:account.account.template,name:l10n_cn_standard.account_1611
msgid "未担保余值"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2702
#: model:account.account.template,name:l10n_cn_standard.account_2702
msgid "未确认融资费用"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_4103
#: model:account.account.template,name:l10n_cn_standard.account_4103
msgid "本年利润"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_5403
#: model:account.account.template,name:l10n_cn_standard.account_5403
msgid "机械作业"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1404
#: model:account.account.template,name:l10n_cn_standard.account_1404
msgid "材料成本差异"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1401
#: model:account.account.template,name:l10n_cn_standard.account_1401
msgid "材料采购"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6061
#: model:account.account.template,name:l10n_cn_standard.account_6061
msgid "汇兑损益"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1631
#: model:account.account.template,name:l10n_cn_standard.account_1631
msgid "油气资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1421
#: model:account.account.template,name:l10n_cn_standard.account_1421
msgid "消耗性生物资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_3001
#: model:account.account.template,name:l10n_cn_standard.account_3001
msgid "清算资金往来"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2621
#: model:account.account.template,name:l10n_cn_standard.account_2621
msgid "独立账户负债"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1821
#: model:account.account.template,name:l10n_cn_standard.account_1821
msgid "独立账户资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1621
#: model:account.account.template,name:l10n_cn_standard.account_1621
msgid "生产性生物资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1622
#: model:account.account.template,name:l10n_cn_standard.account_1622
msgid "生产性生物资产累计折旧"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_5001
#: model:account.account.template,name:l10n_cn_standard.account_5001
msgid "生产成本"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_4101
#: model:account.account.template,name:l10n_cn_standard.account_4101
msgid "盈余公积"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2001
#: model:account.account.template,name:l10n_cn_standard.account_2001
msgid "短期借款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_5301
#: model:account.account.template,name:l10n_cn_standard.account_5301
msgid "研发支出"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6041
#: model:account.account.template,name:l10n_cn_standard.account_6041
msgid "租赁收入"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6602
#: model:account.account.template,name:l10n_cn_standard.account_6602
msgid "管理费用"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1602
#: model:account.account.template,name:l10n_cn_standard.account_1602
msgid "累计折旧"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1632
#: model:account.account.template,name:l10n_cn_standard.account_1632
msgid "累计折耗"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1702
#: model:account.account.template,name:l10n_cn_standard.account_1702
msgid "累计摊销"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1021
#: model:account.account.template,name:l10n_cn_standard.account_1021
msgid "结算备付金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6711
#: model:account.account.template,name:l10n_cn_standard.account_6711
msgid "营业外支出"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6301
#: model:account.account.template,name:l10n_cn_standard.account_6301
msgid "营业外收入"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6403
#: model:account.account.template,name:l10n_cn_standard.account_6403
msgid "营业税金及附加"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1461
#: model:account.account.template,name:l10n_cn_standard.account_1461
msgid "融资租赁资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_3101
#: model:account.account.template,name:l10n_cn_standard.account_3101
msgid "衍生工具"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_3202
#: model:account.account.template,name:l10n_cn_standard.account_3202
msgid "被套期项目"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6603
#: model:account.account.template,name:l10n_cn_standard.account_6603
msgid "财务费用"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_3002
#: model:account.account.template,name:l10n_cn_standard.account_3002
msgid "货币兑换"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2021
#: model:account.account.template,name:l10n_cn_standard.account_2021
msgid "贴现负债"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1301
#: model:account.account.template,name:l10n_cn_standard.account_1301
msgid "贴现资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1431
#: model:account.account.template,name:l10n_cn_standard.account_1431
msgid "贵金属"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1303
#: model:account.account.template,name:l10n_cn_standard.account_1303
msgid "贷款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1304
#: model:account.account.template,name:l10n_cn_standard.account_1304
msgid "贷款损失准备"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6701
#: model:account.account.template,name:l10n_cn_standard.account_6701
msgid "资产减值损失"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_4002
#: model:account.account.template,name:l10n_cn_standard.account_4002
msgid "资本公积"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6511
#: model:account.account.template,name:l10n_cn_standard.account_6511
msgid "赔付支出"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_1_9
#: model:account.account.template,name:l10n_cn_standard.account_2221_1_9
msgid "转出多交增值税"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_1_3
#: model:account.account.template,name:l10n_cn_standard.account_2221_1_3
msgid "转出未交增值税"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1003
#: model:account.account.template,name:l10n_cn_standard.account_1003
msgid "转让帐户"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_1_1
#: model:account.account.template,name:l10n_cn_standard.account_2221_1_1
msgid "进项税额"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_1_7
#: model:account.account.template,name:l10n_cn_standard.account_2221_1_7
msgid "进项税额转出"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6531
#: model:account.account.template,name:l10n_cn_standard.account_6531
msgid "退保金"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2901
#: model:account.account.template,name:l10n_cn_standard.account_2901
msgid "递延所得税负债"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1811
#: model:account.account.template,name:l10n_cn_standard.account_1811
msgid "递延所得税资产"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2401
#: model:account.account.template,name:l10n_cn_standard.account_2401
msgid "递延收益"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_6601
#: model:account.account.template,name:l10n_cn_standard.account_6601
msgid "销售费用"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2221_1_5
#: model:account.account.template,name:l10n_cn_standard.account_2221_1_5
msgid "销项税额"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2501
#: model:account.account.template,name:l10n_cn_standard.account_2501
msgid "长期借款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2701
#: model:account.account.template,name:l10n_cn_standard.account_2701
msgid "长期应付款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1531
#: model:account.account.template,name:l10n_cn_standard.account_1531
msgid "长期应收款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1801
#: model:account.account.template,name:l10n_cn_standard.account_1801
msgid "长期待摊费用"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1511
#: model:account.account.template,name:l10n_cn_standard.account_1511
msgid "长期股权投资"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1512
#: model:account.account.template,name:l10n_cn_standard.account_1512
msgid "长期股权投资减值准备"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_1123
#: model:account.account.template,name:l10n_cn_standard.account_1123
msgid "预付账款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2203
#: model:account.account.template,name:l10n_cn_standard.account_2203
msgid "预收账款"
msgstr ""

#. module: l10n_cn_standard
#: model:account.account,name:l10n_cn_standard.1_account_2801
#: model:account.account.template,name:l10n_cn_standard.account_2801
msgid "预计负债"
msgstr ""


```

