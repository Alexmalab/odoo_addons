# Odoo Module: l10n_cn

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2007-2014 Jeff Wang(<http://jeff@osbzr.com>).

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'China - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['cn'],
    'version': '1.8',
    'category': 'Accounting/Localizations/Account Charts',
    'author': 'openerp-china',
    'maintainer': 'jeff@osbzr.com',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
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

    增加大企业会计科目表

We added the option to print a voucher which will also
print the amount in words (special Chinese characters for numbers)
correctly when the cn2an library is installed. (e.g. with pip3 install cn2an)
    """,
    'depends': [
        'base',
        'account',
    ],
    'data': [
        'views/account_move_view.xml',
        'views/account_report.xml',
        'views/report_voucher.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\template\account.account-cn.csv

```csv
"id","name","code","account_type","reconcile","name@zh_CN"
"l10n_cn_1101","Transactional Financial Assets","110100","asset_current","False","交易性金融资产"
"l10n_cn_1012","Other Monetary Funds","101200","asset_current","False","其他货币资金"
"l10n_cn_1121","Bills Receivable","112100","asset_receivable","True","应收票据"
"l10n_cn_1123","Advance Payment","112300","asset_prepayments","False","预付账款"
"l10n_cn_1131","Divident Receivable","113100","asset_receivable","True","应收股利"
"l10n_cn_1132","Interest Receivable","113200","asset_receivable","True","应收利息"
"l10n_cn_1221","Other Receivable","122100","asset_receivable","True","其他应收款"
"l10n_cn_1231","Bad Debt Provisions","123100","asset_current","False","坏账准备"
"l10n_cn_1401","Material Purchasing","140100","asset_current","False","材料采购"
"l10n_cn_1406","Goods shipped in transit","140600","asset_current","False","运输中的货物"
"l10n_cn_1407","Differences between purchasing and selling price","140700","asset_current","False","商品进销差价"
"l10n_cn_1408","Consigned processing materials","140800","asset_current","False","委托加工的材料"
"l10n_cn_1471","Inventory falling price reserves","147100","asset_current","False","存货跌价准备"
"l10n_cn_1531","Long-term receivables","153100","asset_non_current","True","长期应收款"
"l10n_cn_1602","Accumulated depreciation","160200","expense_depreciation","False","累计折旧"
"l10n_cn_1603","Fixed assets depreciation reserves","160300","expense_depreciation","False","固定资产减值准备"
"l10n_cn_1605","Engineering materials","160500","asset_non_current","False","工程物资"
"l10n_cn_1801","Long-term amortized expenses","180100","expense_depreciation","False","长期待摊销费用"
"l10n_cn_2001","Short-term borrowing","200100","liability_payable","True","短期借款"
"l10n_cn_2201","Bills Payable","220100","liability_payable","True","应付票据"
"l10n_cn_2203","Deposit Received","220300","liability_payable","True","预收账款"
"l10n_cn_2211","Payroll payable","221100","liability_payable","True","应付职工薪酬"
"l10n_cn_2221","Tax payable","222100","liability_current","True","应交税金"
"l10n_cn_2231","Interest payable","223100","liability_payable","True","应付利息"
"l10n_cn_2241","Dividents payable","224100","liability_payable","True","应付股利"
"l10n_cn_2501","Other payable","250100","liability_payable","True","其他应付款"
"l10n_cn_2502","Bonds Payable","250200","liability_payable","True","应付债券"
"l10n_cn_2701","Long Term payables","270100","liability_payable","True","长期应付款"
"l10n_cn_2711","Account payable special funds","271100","liability_payable","True","专项应付款"
"l10n_cn_2801","Projected liabilities","280100","liability_payable","True","预计负债"
"l10n_cn_2901","Deferred Tax Liability","290100","liability_payable","True","递延税项负债"
"l10n_cn_4001","Paid in capital","400100","equity","False","实收资本"
"l10n_cn_4002","Capital Surplus","400200","equity","False","资本公积金"
"l10n_cn_4003","Other Comprehensive Income","400300","equity","False","其他综合收益"
"l10n_cn_4101","Surplus Reserve","410100","equity","False","盈余公积"
"l10n_cn_4103","Profit for the year","410300","equity","False","本年利润"
"l10n_cn_4104","Profit distribution","410400","equity","False","利润分配"

```

## File: data\template\account.account-cn_common.csv

```csv
"id","name","code","account_type","reconcile","name@zh_CN"
"l10n_cn_common_100100","Cash on Hand","100100","asset_cash","False","库存现金"
"l10n_cn_common_112200","Accounts Receivable","112200","asset_receivable","True","应收账款"
"l10n_cn_common_112400","Accounts Receivable (PoS)","112400","asset_receivable","True","应收账款 (POS)"
"l10n_cn_common_140200","Materials in Transit","140200","asset_current","False","在途物资"
"l10n_cn_common_140300","Raw Materials","140300","asset_current","False","原材料"
"l10n_cn_common_140400","Material Cost Variance","140400","asset_current","False","材料成本差异"
"l10n_cn_common_140500","Merchandise Inventory","140500","asset_current","False","库存商品"
"l10n_cn_common_150100","Investments held to maturity","150100","asset_current","False","持有至到期投资"
"l10n_cn_common_150200","Provision for impairment of investments held to maturity","150200","asset_current","False","持有至到期投资减值准备"
"l10n_cn_common_150300","Financial Assets Available for Sale","150300","asset_current","False","可供出售金融资产"
"l10n_cn_common_151100","Long-term equity investment","151100","asset_non_current","False","长期股权投资"
"l10n_cn_common_151200","Impairment provision for long-term equity investments","151200","asset_non_current","False","长期股权投资减值准备"
"l10n_cn_common_152100","Investmental real estate","152100","asset_non_current","False","投资性房地产"
"l10n_cn_common_160100","Fixed assets","160100","asset_fixed","False","固定资产"
"l10n_cn_common_160400","Construction in Progress","160400","asset_fixed","False","在建工程"
"l10n_cn_common_160600","Liquidation of Fixed Assets","160600","asset_fixed","False","固定资产清理"
"l10n_cn_common_170100","Intangible Assets","170100","asset_non_current","False","无形资产"
"l10n_cn_common_170200","Accumulated amortization","170200","asset_non_current","False","累计摊销"
"l10n_cn_common_170300","Intangible Assets Depreciation reserves","170300","asset_non_current","False","无形资产减值准备"
"l10n_cn_common_171100","Goodwill","171100","asset_non_current","False","商誉"
"l10n_cn_common_220200","Accounts Payable","220200","liability_payable","True","应付账款"
"l10n_cn_common_500100","Production Costs","500100","expense_direct_cost","False","生产成本"
"l10n_cn_common_510100","Depreciation","510100","expense_direct_cost","False","制造费用"
"l10n_cn_common_520100","Service Cost","520100","expense_direct_cost","False","劳务成本"
"l10n_cn_common_530100","R&D Expenditure","530100","expense_direct_cost","False","研发支出"
"l10n_cn_common_600100","Income","600100","income","False","主营业务收入"
"l10n_cn_common_605100","Other Operating Income","605100","income_other","False","其他业务收入"
"l10n_cn_common_610100","Gains and Losses of fair value change","610100","income_other","False","公允价值变动损益"
"l10n_cn_common_611100","Income from Investments","611100","income_other","False","投资收益"
"l10n_cn_common_630100","Non-Operating Income","630100","income_other","False","营业外收入"
"l10n_cn_common_640100","Main Business Costs","640100","expense_direct_cost","False","主营业务成本"
"l10n_cn_common_640200","Other Operating Costs","640200","expense_direct_cost","False","其它业务成本"
"l10n_cn_common_640300","Operating Taxes and Surcharges","640300","expense","False","营业税金及附加"
"l10n_cn_common_660100","Sales Expense","660100","expense","False","销售费用"
"l10n_cn_common_660200","Managment Expense","660200","expense","False","管理费用"
"l10n_cn_common_660300","Financial Expenses","660300","expense","False","财务费用"
"l10n_cn_common_670100","Asset Impairment loss","670100","expense","False","资产减值损失"
"l10n_cn_common_671100","Non-Operating Expenses","671100","expense","False","营业外支出"
"l10n_cn_common_680100","Income Tax","680100","expense","False","所得税"
"l10n_cn_common_690100","Prior year income adjustment","690100","expense","False","以前年度损益调整"

```

## File: data\template\account.account-cn_large_bis.csv

```csv
"id","name","code","account_type","reconcile","name@zh_CN"
"l10n_cn_large_bis_100200","Bank Savings","100200","asset_cash","False","银行存款"
"l10n_cn_large_bis_100201","Bank Suspense Account","100201","asset_cash","False","银行暂记账户"
"l10n_cn_large_bis_100202","Outstanding Receipts","100202","asset_cash","False","未结收据"
"l10n_cn_large_bis_100203","Outstanding Payments","100203","asset_cash","False","未付款"
"l10n_cn_large_bis_100204","Bank","100204","asset_cash","False","银行"
"l10n_cn_large_bis_100300","Transfer Account","100300","asset_cash","False","转让帐户"
"l10n_cn_large_bis_110100","Tradable Financial Assets","110100","asset_current","False","交易性金融资产"
"l10n_cn_large_bis_101500","Other Monetary Funds","101500","asset_current","False","其他货币资金"
"l10n_cn_large_bis_112100","Notes Receivable","112100","asset_receivable","True","应收票据"
"l10n_cn_large_bis_112300","Prepayments to Supplier","112300","asset_prepayments","False","预付账款"
"l10n_cn_large_bis_113100","Dividend Receivable","113100","asset_receivable","True","应收股利"
"l10n_cn_large_bis_113101","Shares in subsidiary companies","113101","asset_receivable","True","应收子公司股利"
"l10n_cn_large_bis_113200","Interest Receivable","113200","asset_receivable","True","应收利息"
"l10n_cn_large_bis_122100","Other Receivable","122100","asset_receivable","True","其它应收款"
"l10n_cn_large_bis_123100","Provision for Bad Debts","123100","asset_current","False","坏账准备"
"l10n_cn_large_bis_130301","Employees' Loan / Cash Advance","130301","asset_current","False","员工借款"
"l10n_cn_large_bis_130302","Intercompany Loan","130302","asset_current","False","集团内部借款"
"l10n_cn_large_bis_130303","Other Loans","130303","asset_current","False","其他借款"
"l10n_cn_large_bis_140100","Purchase of Material","140100","asset_current","False","材料采购"
"l10n_cn_large_bis_140700","Goods shipped in transit","140700","asset_current","False","发出商品"
"l10n_cn_large_bis_140800","Consigned processing materials","140800","asset_current","False","委托加工物资"
"l10n_cn_large_bis_141100","Working Materials","141100","asset_current","False","周转材料"
"l10n_cn_large_bis_142100","Consumable Biological Assets","142100","asset_current","False","消耗性生物资产"
"l10n_cn_large_bis_143100","Precious Metal","143100","asset_current","False","贵金属"
"l10n_cn_large_bis_144100","Foreclosed Assets","144100","asset_current","False","抵债资产"
"l10n_cn_large_bis_145100","Surplus Materials","145100","asset_current","False","损余物资"
"l10n_cn_large_bis_146100","Provision for Stock Impairement","146100","asset_current","False","存货跌价准备"
"l10n_cn_large_bis_147100","Deferred Expenses","147100","liability_current","False","待摊费用"
"l10n_cn_large_bis_153100","Long-term receivables","153100","asset_non_current","False","长期应收款"
"l10n_cn_large_bis_153200","Unrealised financial income","153200","asset_non_current","False","未实现融资收益"
"l10n_cn_large_bis_154100","Deposited capital guarantee","154100","asset_non_current","False","存出资本保证金"
"l10n_cn_large_bis_160200","Tangible Assets Depreciation","160200","asset_fixed","False","累计折旧"
"l10n_cn_large_bis_160300","Provision for Fixed Assets Impairment","160300","asset_fixed","False","固定资产减值准备"
"l10n_cn_large_bis_160500","Construction Materials","160500","asset_non_current","False","工程物资"
"l10n_cn_large_bis_161100","Unguaranteed residual value","161100","asset_non_current","False","未担保余值"
"l10n_cn_large_bis_162100","Productive Biological Assets","162100","asset_non_current","False","生产性生物资产"
"l10n_cn_large_bis_162200","Accumulated depreciation of productive biological assets","162200","asset_non_current","False","生产性生物资产累计折旧"
"l10n_cn_large_bis_162300","Public Welfare Biological Assets","162300","asset_non_current","False","公益性生物资产"
"l10n_cn_large_bis_163100","Oil and Gas Assets","163100","asset_non_current","False","油气资产"
"l10n_cn_large_bis_163200","Cumulative Depreciation","163200","asset_non_current","False","累计折耗"
"l10n_cn_large_bis_180100","Long-term Deferred and Prepaid Expense","180100","asset_non_current","False","长期待摊费用"
"l10n_cn_large_bis_181100","Deferred Tax Assets","181100","asset_non_current","False","递延所得资产"
"l10n_cn_large_bis_182100","Separate Account Assets","182100","asset_non_current","False","独立账户资产"
"l10n_cn_large_bis_190100","Pending Property Gains / Losses","190100","asset_non_current","False","待处理财产损溢"
"l10n_cn_large_bis_200100","Short-Term Loans","200100","liability_current","False","短期借款"
"l10n_cn_large_bis_200200","Deposit Margin","200200","liability_current","False","存入保证金"
"l10n_cn_large_bis_200300","Borrowing Funds","200300","liability_current","False","拆入资金"
"l10n_cn_large_bis_200400","Borrowing from Central Bank","200400","liability_current","False","向中央银行借款"
"l10n_cn_large_bis_201100","Interbank Deposit","201100","liability_current","False","同业存放"
"l10n_cn_large_bis_201200","Take Deposit","201200","liability_current","False","吸收存款"
"l10n_cn_large_bis_202100","Discounted liabilities","202100","liability_current","False","贴现负债"
"l10n_cn_large_bis_210100","Financial Trading Liabilities","210100","liability_current","False","交易性金融负债"
"l10n_cn_large_bis_211100","Dedicated Repurchase of Financial Assets","211100","liability_current","False","专出回购金融资产款"
"l10n_cn_large_bis_220100","Notes Payable","220100","liability_payable","True","应付票据"
"l10n_cn_large_bis_220300","Advance from Customers","220300","liability_current","False","预收账款"
"l10n_cn_large_bis_221100","Payroll payable","221100","liability_current","False","应付职工薪酬"
"l10n_cn_large_bis_222100","Tax Payable","222100","liability_current","False","应交税费"
"l10n_cn_large_bis_223100","Interest Payable","223100","liability_payable","True","应付利息"
"l10n_cn_large_bis_223200","Dividend Payable","223200","liability_payable","True","应付股利"
"l10n_cn_large_bis_224100","Other Payables","224100","liability_payable","True","其他应付款"
"l10n_cn_large_bis_225100","Protection Bonus Payable","225100","liability_payable","True","应付保护红利"
"l10n_cn_large_bis_226100","Reinsurance Accounts Payable","226100","liability_payable","True","应付分保账款"
"l10n_cn_large_bis_231100","Brokerage for buying and selling securities","231100","liability_current","False","代理买卖证券款"
"l10n_cn_large_bis_231200","Agency Underwriting Securities Payment","231200","liability_current","False","代理承销证券款"
"l10n_cn_large_bis_231300","Agent acting for redemption of security payment redemption","231300","liability_current","False","代理兑付证券款"
"l10n_cn_large_bis_231400","Agency Business Liabilities","231400","liability_current","False","代理业务负债"
"l10n_cn_large_bis_240100","Deferred Income","240100","liability_current","False","递延收益"
"l10n_cn_large_bis_250100","Long-term loan","250100","liability_non_current","False","长期借款"
"l10n_cn_large_bis_250200","Bonds Payable","250200","liability_payable","True","應付債券"
"l10n_cn_large_bis_260100","Reserve for Unearned Liabilities","260100","liability_non_current","False","未到期责任准备金"
"l10n_cn_large_bis_260200","Insurance Liability Reserve","260200","liability_non_current","False","保险责任准备金"
"l10n_cn_large_bis_261100","Policyholder savings","261100","liability_non_current","False","保户储金"
"l10n_cn_large_bis_262100","Separate Account Liabilities","262100","liability_non_current","False","独立帐户负债"
"l10n_cn_large_bis_270100","Long-term bonds","270100","liability_non_current","False","长期债券"
"l10n_cn_large_bis_271100","Special Payables","271100","liability_payable","True","专项应付款"
"l10n_cn_large_bis_280100","Estimated liabilities","280100","liability_non_current","False","预计负债"
"l10n_cn_large_bis_290100","Deferred Tax Liabilities","290100","liability_non_current","False","递延所得税负债"
"l10n_cn_large_bis_300100","Liquidation of Funds","300100","equity","False","清算资金往来"
"l10n_cn_large_bis_300200","Currency Exchange Gain / Loss","300200","equity","False","货币资换"
"l10n_cn_large_bis_310100","Derivatives","310100","equity","False","衍生工具"
"l10n_cn_large_bis_320100","Hedging Instrument","320100","equity","False","套期工具"
"l10n_cn_large_bis_320200","Hedged Item","320200","equity","False","被套期项目"
"l10n_cn_large_bis_540100","Construction Cost","540100","expense_direct_cost","False","工程施工"
"l10n_cn_large_bis_540200","Project Settlement Cost","540200","expense_direct_cost","False","工程结算"
"l10n_cn_large_bis_540300","Cost of Repair","540300","expense_direct_cost","False","机械作业"
"l10n_cn_large_bis_601100","Interest Income","601100","income_other","False","利息收入"
"l10n_cn_large_bis_602100","Fee Income","602100","income_other","False","手续费收入"
"l10n_cn_large_bis_603100","Premium Income","603100","income_other","False","保费收入"
"l10n_cn_large_bis_604100","Rental Income","604100","income_other","False","租赁收入"
"l10n_cn_large_bis_606100","Exchange Gains / Losses","606100","income_other","False","汇兑损益"
"l10n_cn_large_bis_620100","Amortization of Insurance Liability Reserves","620100","income_other","False","摊回保险责任准备金"
"l10n_cn_large_bis_620200","Amortization fo Compensation Expenses","620200","income_other","False","摊回赔付支出"
"l10n_cn_large_bis_620300","Amortization of Reinsurance Expenses","620300","income_other","False","摊回分保费用"
"l10n_cn_large_bis_641100","Interest Expense","641100","expense","False","利息支出"
"l10n_cn_large_bis_642100","Fee Expenditure","642100","expense","False","手续费支出"
"l10n_cn_large_bis_650100","Unexpired Liability Reserves Withdrawal","650100","expense","False","提取未到期责任准备金"
"l10n_cn_large_bis_650200","Insurance Liability Reserve","650200","expense","False","撮保险责任准备金"
"l10n_cn_large_bis_651100","Compensation Expenses","651100","expense","False","赔付支出"
"l10n_cn_large_bis_652100","Policyholder Bonus Payment","652100","expense","False","保户红利支出"
"l10n_cn_large_bis_653100","Surrender","653100","expense","False","退保金"
"l10n_cn_large_bis_654100","Ceded Premium","654100","expense","False","分出保费"
"l10n_cn_large_bis_654200","Reinsurance Costs","654200","expense","False","分保费用"
"l10n_cn_large_bis_660400","Exploration Costs","660400","expense","False","勘探费用"
"l10n_cn_large_bis_999100","Cash Difference Loss","999100","expense","False","现金差额损失"
"l10n_cn_large_bis_999200","Cash Difference Gain","999200","income","False","现金差额收益"
"l10n_cn_large_bis_999900","Undistributed Profits/Losses","999900","equity_unaffected","False","未分配利润/亏损"

```

## File: data\template\account.tax-cn.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","price_include","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","description@zh_CN"
"l10n_cn_sales_included_13","13% INC","VAT 13% (included)","13%","13.0","percent","sale","True","l10n_cn_tax_group_vat_13","base","invoice","","税收13% (含)"
"","","","","","","","","","tax","invoice","l10n_cn_2221",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_2221",""
"l10n_cn_sales_included_9","9% INC","VAT 9% (included)","9%","9.0","percent","sale","True","l10n_cn_tax_group_vat_9","base","invoice","","税收9% (含)"
"","","","","","","","","","tax","invoice","l10n_cn_2221",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_2221",""
"l10n_cn_sales_included_6","6% INC","VAT 6% (included)","6%","6.0","percent","sale","True","l10n_cn_tax_group_vat_6","base","invoice","","税收6% (含)"
"","","","","","","","","","tax","invoice","l10n_cn_2221",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_2221",""
"l10n_cn_sales_excluded_13","13%","VAT 13%","13%","13.0","percent","sale","False","l10n_cn_tax_group_vat_13","base","invoice","","增值税13%"
"","","","","","","","","","tax","invoice","l10n_cn_2221",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_2221",""
"l10n_cn_sales_excluded_9","9%","VAT 9%","9%","9.0","percent","sale","False","l10n_cn_tax_group_vat_9","base","invoice","","增值税9%"
"","","","","","","","","","tax","invoice","l10n_cn_2221",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_2221",""
"l10n_cn_sales_excluded_6","6%","VAT 6%","6%","6.0","percent","sale","False","l10n_cn_tax_group_vat_6","base","invoice","","增值税6%"
"","","","","","","","","","tax","invoice","l10n_cn_2221",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_2221",""
"l10n_cn_purchase_excluded_13","13%","VAT 13%","13%","13.0","percent","purchase","False","l10n_cn_tax_group_vat_13","base","invoice","","增值税13%"
"","","","","","","","","","tax","invoice","l10n_cn_2221",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_2221",""
"l10n_cn_purchase_excluded_9","9%","VAT 9%","9%","9.0","percent","purchase","False","l10n_cn_tax_group_vat_9","base","invoice","","增值税9%"
"","","","","","","","","","tax","invoice","l10n_cn_2221",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_2221",""
"l10n_cn_purchase_excluded_6","6%","VAT 6%","6%","6.0","percent","purchase","False","l10n_cn_tax_group_vat_6","base","invoice","","增值税6%"
"","","","","","","","","","tax","invoice","l10n_cn_2221",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_2221",""

```

## File: data\template\account.tax-cn_large_bis.csv

```csv
"id","name","description","invoice_label","amount","amount_type","type_tax_use","price_include","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","description@zh_CN"
"l10n_cn_tax_large_bis_sales_included_13","13% INC","VAT 13% (included)","13%","13.0","percent","sale","True","l10n_cn_tax_group_vat_13","base","invoice","","税收13%（含)"
"","","","","","","","","","tax","invoice","l10n_cn_large_bis_222100",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_large_bis_222100",""
"l10n_cn_tax_large_bis_sales_included_9","9% INC","VAT 9%(included)","9%","9.0","percent","sale","True","l10n_cn_tax_group_vat_9","base","invoice","","税收9%(含)"
"","","","","","","","","","tax","invoice","l10n_cn_large_bis_222100",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_large_bis_222100",""
"l10n_cn_tax_large_bis_sales_included_6","6% INC","VAT 6%(included)","6%","6.0","percent","sale","True","l10n_cn_tax_group_vat_6","base","invoice","","税收6%(含)"
"","","","","","","","","","tax","invoice","l10n_cn_large_bis_222100",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_large_bis_222100",""
"l10n_cn_tax_large_bis_sales_excluded_13","13%","VAT 13%","13%","13.0","percent","sale","False","l10n_cn_tax_group_vat_13","base","invoice","","增值税13%"
"","","","","","","","","","tax","invoice","l10n_cn_large_bis_222100",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_large_bis_222100",""
"l10n_cn_tax_large_bis_sales_excluded_9","9%","VAT 9%","9%","9.0","percent","sale","False","l10n_cn_tax_group_vat_9","base","invoice","","增值税9%"
"","","","","","","","","","tax","invoice","l10n_cn_large_bis_222100",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_large_bis_222100",""
"l10n_cn_tax_large_bis_sales_excluded_6","6%","VAT 6%","6%","6.0","percent","sale","False","l10n_cn_tax_group_vat_6","base","invoice","","增值税6%"
"","","","","","","","","","tax","invoice","l10n_cn_large_bis_222100",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_large_bis_222100",""
"l10n_cn_tax_large_bis_purchase_excluded_13","13%","VAT 13%","13%","13.0","percent","purchase","False","l10n_cn_tax_group_vat_13","base","invoice","","增值税13%"
"","","","","","","","","","tax","invoice","l10n_cn_large_bis_222100",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_large_bis_222100",""
"l10n_cn_tax_large_bis_purchase_excluded_9","9%","VAT 9%","9%","9.0","percent","purchase","False","l10n_cn_tax_group_vat_9","base","invoice","","增值税9%"
"","","","","","","","","","tax","invoice","l10n_cn_large_bis_222100",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_large_bis_222100",""
"l10n_cn_tax_large_bis_purchase_excluded_6","6%","VAT 6%","6%","6.0","percent","purchase","False","l10n_cn_tax_group_vat_6","base","invoice","","增值税6%"
"","","","","","","","","","tax","invoice","l10n_cn_large_bis_222100",""
"","","","","","","","","","base","refund","",""
"","","","","","","","","","tax","refund","l10n_cn_large_bis_222100",""

```

## File: data\template\account.tax.group-cn_common.csv

```csv
"id","name","country_id","name@zh_CN"
"l10n_cn_tax_group_vat_6","VAT 6%","base.cn","增值税6%"
"l10n_cn_tax_group_vat_9","VAT 9%","base.cn","增值税9%"
"l10n_cn_tax_group_vat_13","VAT 13%","base.cn","增值税13%"

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

## File: models\template_cn.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('cn')
    def _get_cn_template_data(self):
        return {
            'parent': 'cn_common',
        }

    @template('cn', 'res.company')
    def _get_cn_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.cn',
                'bank_account_code_prefix': '1002',
                'cash_account_code_prefix': '1001',
                'transfer_account_code_prefix': '1012',
                'account_default_pos_receivable_account_id': 'l10n_cn_common_112400',
                'income_currency_exchange_account_id': 'l10n_cn_common_605100',
                'expense_currency_exchange_account_id': 'l10n_cn_common_671100',
                'account_sale_tax_id': 'l10n_cn_sales_included_13',
                'account_purchase_tax_id': 'l10n_cn_purchase_excluded_13',
            },
        }

    @template('cn', 'account.journal')
    def _get_cn_account_journal(self):
        return {
            'cash': {
                'name': 'Cash on Hand',
                'default_account_id': 'l10n_cn_common_100100'
            },
        }

```

## File: models\template_cn_common.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('cn_common')
    def _get_cn_common_template_data(self):
        return {
            'name': _('Common'),
            'visible': 0,
            'code_digits': 6,
            'use_storno_accounting': True,
            'property_account_receivable_id': 'l10n_cn_common_112200',
            'property_account_payable_id': 'l10n_cn_common_220200',
            'property_account_expense_categ_id': 'l10n_cn_common_640100',
            'property_account_income_categ_id': 'l10n_cn_common_600100',
        }

    @template('cn_common', 'res.company')
    def _get_cn_common_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.cn',
                'bank_account_code_prefix': '1002',
                'cash_account_code_prefix': '1001',
                'transfer_account_code_prefix': '1012',
                'account_default_pos_receivable_account_id': 'l10n_cn_common_112400',
                'income_currency_exchange_account_id': 'l10n_cn_common_605100',
                'expense_currency_exchange_account_id': 'l10n_cn_common_671100',
            },
        }

```

## File: models\template_cn_large_bis.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, _
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('cn_large_bis')
    def _get_cn_large_bis_template_data(self):
        return {
            'name': _('Large Business'),
            'parent': 'cn_common',
        }

    @template('cn_large_bis', 'res.company')
    def _get_cn_large_bis_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.cn',
                'bank_account_code_prefix': '1002',
                'cash_account_code_prefix': '1001',
                'transfer_account_code_prefix': '1012',
                'account_default_pos_receivable_account_id': 'l10n_cn_common_112400',
                'income_currency_exchange_account_id': 'l10n_cn_common_605100',
                'expense_currency_exchange_account_id': 'l10n_cn_common_671100',
                'account_sale_tax_id': 'l10n_cn_tax_large_bis_sales_included_13',
                'account_purchase_tax_id': 'l10n_cn_tax_large_bis_purchase_excluded_13',
                'account_journal_suspense_account_id': 'l10n_cn_large_bis_100201',
                'account_journal_payment_debit_account_id': 'l10n_cn_large_bis_100202',
                'account_journal_payment_credit_account_id': 'l10n_cn_large_bis_100203',
                'default_cash_difference_income_account_id': 'l10n_cn_large_bis_999200',
                'default_cash_difference_expense_account_id': 'l10n_cn_large_bis_999100',
            },
        }

    @template('cn_large_bis', 'account.journal')
    def _get_cn_large_bis_account_journal(self):
        return {
            'cash': {
                'name': 'Cash on Hand',
                'default_account_id': 'l10n_cn_common_100100',
            },
            'bank': {
                'default_account_id': 'l10n_cn_large_bis_100204',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_cn_common
from . import template_cn
from . import template_cn_large_bis
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
                    <field name="fapiao" invisible="country_code != 'CN' or move_type not in ['out_invoice', 'out_refund', 'in_invoice', 'in_refund']"/>
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
                            <span>Accounting Voucher</span>
                        </h2>
                    </div>

                    <div id="company" class="row col-auto">
                        <span t-field="o.company_id.name"/>
                    </div>
                    <div id="informations" class="row">
                        <!-- offset intentionally for period -->
                        <div class="col-3 offset-3" name="date">
                            <strong>Date:</strong>
                            <span t-field="o.date"/>
                        </div>
                        <div class="col-4" t-if="o.name" name="name">
                            <strong>Reference:</strong>
                            <span t-field="o.name"/>
                        </div>
                        <div class="col-2">
                            <strong>Number of attachments:</strong>
                            <span t-esc="o._count_attachments()"/>
                        </div>
                    </div>

                    <table class="table table-sm o_main_table table-striped" name="entry_line_table">
                        <thead>
                            <tr>
                                <t t-set="colspan" t-value="4"/>
                                <th name="th_description" class="text-center"><span>Balance</span></th>
                                <th name="th_account" class="text-center"><span>Account</span></th>
                                <th name="th_debit" class="text-center"><span>Debit</span></th>
                                <th name="th_credit" class="text-center"><span>Credit</span></th>
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
                                    <span>Total:</span>
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
                            <strong>Validator:</strong>
                        </div>
                        <div class="col-4">
                            <strong>Poster:</strong>
                        </div>
                        <div class="col-4">
                            <strong>Salesperson:</strong>
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

