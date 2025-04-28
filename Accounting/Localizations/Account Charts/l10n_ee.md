# Odoo Module: l10n_ee

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Estonia - Accounting',
    'version': '1.2',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the base module to manage the accounting chart for Estonia in Odoo.
    """,
    'author': 'Odoo SA',
    'depends': [
        'account',
        'l10n_multilang',
    ],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/l10n_ee_chart_post_data.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_fiscal_position_template_data.xml',
        'data/account.group.template.csv',
        'data/account_chart_template_try_loading.xml',
        'views/account_tax_form.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,name,code,account_type,chart_template_id/id,tag_ids/id,reconcile
l10n_ee_1000,Cash Accounts,1000,asset_cash,l10n_ee.l10nee_chart_template,,False
l10n_ee_1001,Bank Accounts,1001,asset_cash,l10n_ee.l10nee_chart_template,,False
l10n_ee_1008,Transfer Accounts,1008,asset_current,l10n_ee.l10nee_chart_template,,True
l10n_ee_1009,Bank Suspense Account,1009,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1010,Short-Term Financial Investments,1010,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1011,Depreciations on Short-Term Financial Investments,1011,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_10200,Accounts Receivable,10200,asset_receivable,l10n_ee.l10nee_chart_template,,True
l10n_ee_10201,Accounts Receivable (POS),10201,asset_receivable,l10n_ee.l10nee_chart_template,,True
l10n_ee_10202,Doubtful Receivables,10202,asset_receivable,l10n_ee.l10nee_chart_template,,True
l10n_ee_1021,Receivables from Related Parties,1021,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1022,Prepaid and Deferred Taxes,1022,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1023,Loan Receivables,1023,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_10240,Interest Receivable,10240,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_10241,Dividend Receivable,10241,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_10242,Accounts Receivable from Social Insurance Board,10242,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_10243,Netting Account,10243,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_10244,Other Short-Term Receivables,10244,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1025,Prepayments,1025,asset_prepayments,l10n_ee.l10nee_chart_template,,False
l10n_ee_1030,Raw and Other Materials,1030,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1031,Work in Progress,1031,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_10320,Finished Goods from Agricultural Production,10320,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_10321,Other Finished Goods,10321,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1033,Goods for Resale,1033,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1034,Prepayments to Suppliers,1034,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1040,Biological Assets,1040,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1100,Shares and Participations in Subsidiaries,1100,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1101,Shares and Participations in Affiliates,1101,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1110,"Other Shares, Stocks and Bonds",1110,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1111,Other Long-Term Financial Investments,1111,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1120,Long-Term Trade Receivables,1120,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1121,Long-Term Receivables from Related Parties,1121,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1122,Long-Term Prepaid and Deferred Taxes,1122,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1123,Long-Term Loan Receivables,1123,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_1124,Other Long-Term Receivables,1124,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_11250,Constructions in Progress,11250,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_11251,Prepayments for Non-Current Assets,11251,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_11300,Investment Properties,11300,asset_fixed,l10n_ee.l10nee_chart_template,,False
l10n_ee_11301,Depreciations on Investment Properties,11301,asset_fixed,l10n_ee.l10nee_chart_template,,False
l10n_ee_1140,Land,1140,asset_fixed,l10n_ee.l10nee_chart_template,,False
l10n_ee_11410,Buildings and Structures,11410,asset_fixed,l10n_ee.l10nee_chart_template,,False
l10n_ee_11411,Depreciations on Buildings and Structures,11411,asset_fixed,l10n_ee.l10nee_chart_template,,False
l10n_ee_11420,Vehicles,11420,asset_fixed,l10n_ee.l10nee_chart_template,,False
l10n_ee_11421,Depreciations on Vehicles,11421,asset_fixed,l10n_ee.l10nee_chart_template,,False
l10n_ee_11430,Computers and Computer Systems,11430,asset_fixed,l10n_ee.l10nee_chart_template,,False
l10n_ee_11431,Depreciations on Computers and Computer Systems,11431,asset_fixed,l10n_ee.l10nee_chart_template,,False
l10n_ee_11440,Other Machinery and Equipment,11440,asset_fixed,l10n_ee.l10nee_chart_template,,False
l10n_ee_11441,Depreciations on Other Machinery and Equipment,11441,asset_fixed,l10n_ee.l10nee_chart_template,,False
l10n_ee_11450,Other Tangible Non-Current Assets,11450,asset_fixed,l10n_ee.l10nee_chart_template,,False
l10n_ee_11451,Depreciations on Other Tangible Non-Current Assets,11451,asset_fixed,l10n_ee.l10nee_chart_template,,False
l10n_ee_115,Biological Assets,115,asset_fixed,l10n_ee.l10nee_chart_template,,False
l10n_ee_11600,Goodwill,11600,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_11601,Amortizations on Goodwill,11601,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_11610,Development Costs,11610,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_11611,Amortizations on Development Costs,11611,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_11620,Computer Software,11620,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_11621,Amortizations on Computer Software,11621,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_11630,"Patents, Licenses and Trademarks",11630,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_11631,"Amortizations on Patents, Licenses and Trademarks",11631,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_11640,Other Intangible Assets,11640,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_11641,Amortizations on Other Intangible Assets,11641,asset_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2000,Short-Term Bank Loans,2000,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2001,Current Bank Overdrafts,2001,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2002,Short-Term Loans from Owners,2002,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2003,Short-Term Loans from Other Parties,2003,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2004,Current Portion of Long-Term Loan,2004,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2010,Accounts Payable,2010,liability_payable,l10n_ee.l10nee_chart_template,,True
l10n_ee_20110,Salaries and Wages Payable,20110,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20111,Withholdings from Salary,20111,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20112,Vacation Pay Reserve,20112,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20113,Other Employee Payables,20113,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_201200,VAT Current Account,201200,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_201201,VAT Receivable (Input VAT),201201,asset_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_201202,VAT on the Acquisition of Non-Current Assets,201202,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_201203,VAT on the Import at Customs,201203,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_201204,VAT Payable (Output VAT),201204,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20121,Income Tax Payable (Company),20121,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20122,Income Tax Payable (Personal),20122,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20123,Income Tax Payable (Fringe Benefits),20123,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20124,Social Tax Payable,20124,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20125,Unemployment Insurance Premium Payable,20125,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20126,Pension Insurance Payable,20126,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20127,Land Tax Payable,20127,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20128,Excise Tax Payable,20128,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20129,Other Taxes Payable,20129,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20130,Payables to Related Parties,20130,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20131,Dividends Payable,20131,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20132,Interests Payable,20132,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20133,Other Short-Term Payables,20133,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_20140,Prepayments from Customers,20140,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2015,Other Received Prepayments,2015,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2020,Warranty Provisions,2020,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2021,Tax Provisions,2021,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2022,Other Provisions,2022,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_203,Government Grants,203,liability_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2100,Long-Term Bank Loans,2100,liability_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2101,Long-Term Loans from Owners,2101,liability_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2102,Long-Term Portion of Financial Lease,2102,liability_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2110,Long-Term Accounts Payable,2110,liability_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2111,Long-Term Employee Payables,2111,liability_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2112,Long-Term Taxes Payable,2112,liability_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2113,Other Long-Term Payables,2113,liability_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2114,Long-Term Deferred Revenue,2114,liability_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2115,Other Received Long-Term Prepayments,2115,liability_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2120,Long-Term Warranty Provisions,2120,liability_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2121,Long-Term Tax Provisions,2121,liability_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_2122,Other Long-Term Provisions,2122,liability_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_213,Long-Term Government Grants,213,liability_non_current,l10n_ee.l10nee_chart_template,,False
l10n_ee_300,Share Capital (Nominal Value),300,equity,l10n_ee.l10nee_chart_template,,False
l10n_ee_301,Unregistered Share Capital or Equity,301,equity,l10n_ee.l10nee_chart_template,,False
l10n_ee_302,Unpaid Share Capital,302,equity,l10n_ee.l10nee_chart_template,,False
l10n_ee_303,Share Premium,303,equity,l10n_ee.l10nee_chart_template,,False
l10n_ee_304,Own Shares,304,equity,l10n_ee.l10nee_chart_template,,False
l10n_ee_310,Statutory Reserve Capital,310,equity,l10n_ee.l10nee_chart_template,,False
l10n_ee_311,Other Reserves,311,equity,l10n_ee.l10nee_chart_template,,False
l10n_ee_32,Other Equity,32,equity,l10n_ee.l10nee_chart_template,,False
l10n_ee_330,Retained Profit/Loss From Previous Periods,330,equity,l10n_ee.l10nee_chart_template,,False
l10n_ee_331,Profit/Loss for the Financial Year,331,equity,l10n_ee.l10nee_chart_template,,False
l10n_ee_40000,Sales of Goods in Estonia,40000,income,l10n_ee.l10nee_chart_template,,False
l10n_ee_40001,Sales of Goods from Biological Assets in Estonia,40001,income,l10n_ee.l10nee_chart_template,,False
l10n_ee_4001,Sales of Services in Estonia,4001,income,l10n_ee.l10nee_chart_template,,False
l10n_ee_40100,Sales of Goods in the EU,40100,income,l10n_ee.l10nee_chart_template,,False
l10n_ee_40101,Sales of Goods from Biological Assets in the EU,40101,income,l10n_ee.l10nee_chart_template,,False
l10n_ee_4011,Sales of Services in the EU,4011,income,l10n_ee.l10nee_chart_template,,False
l10n_ee_40200,Export of Goods,40200,income,l10n_ee.l10nee_chart_template,,False
l10n_ee_40201,Export of Goods from Biological Assets,40201,income,l10n_ee.l10nee_chart_template,,False
l10n_ee_4021,Export of Services,4021,income,l10n_ee.l10nee_chart_template,,False
l10n_ee_41,Sales of Assets,41,income,l10n_ee.l10nee_chart_template,,False
l10n_ee_420,Cash Rounding Gains,420,income_other,l10n_ee.l10nee_chart_template,,False
l10n_ee_421,Payment Difference Gains,421,income_other,l10n_ee.l10nee_chart_template,,False
l10n_ee_422,Currency Exchange Rate Gains,422,income_other,l10n_ee.l10nee_chart_template,,False
l10n_ee_423,Interest Received,423,income_other,l10n_ee.l10nee_chart_template,,False
l10n_ee_424,Financial Income from Shares in Subsidiaries,424,income_other,l10n_ee.l10nee_chart_template,,False
l10n_ee_425,Financial Income from Shares in Associates,425,income_other,l10n_ee.l10nee_chart_template,,False
l10n_ee_426,Financial Income from Other Financial Investments,426,income_other,l10n_ee.l10nee_chart_template,,False
l10n_ee_427,Other Financial Income,427,income_other,l10n_ee.l10nee_chart_template,,False
l10n_ee_430,Cash Discount Gains,430,income_other,l10n_ee.l10nee_chart_template,,False
l10n_ee_431,Other Income,431,income_other,l10n_ee.l10nee_chart_template,,False
l10n_ee_50,Purchase of Goods for Resale,50,expense_direct_cost,l10n_ee.l10nee_chart_template,,False
l10n_ee_51,Purchase of Raw and Other Materials,51,expense_direct_cost,l10n_ee.l10nee_chart_template,,False
l10n_ee_52,Purchase of Services / Subcontracting,52,expense_direct_cost,l10n_ee.l10nee_chart_template,,False
l10n_ee_53,"Transportation Costs for Goods, Raw Materials and Services",53,expense_direct_cost,l10n_ee.l10nee_chart_template,,False
l10n_ee_54,"Customs Fees for Goods, Raw Materials and Services",54,expense_direct_cost,l10n_ee.l10nee_chart_template,,False
l10n_ee_600,Buildings Rental,600,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6010,Office Rental,6010,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6011,Office Utilities,6011,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6012,Office Security Costs,6012,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6013,Office Maintenance and Repairs,6013,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6020,Workshop Rental,6020,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6021,Workshop Utilities,6021,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6022,Workshop Security Costs,6022,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6023,Workshop Maintance and Repairs,6023,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_603,Property Insurance,603,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_604,Electricity / Gas,604,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_605,Water,605,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_606,Internet,606,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_607,Phone Costs,607,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_610,Equipment Rental,610,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_611,Machinery and Equipment Maintenance and Repairs,611,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_612,"Fuels, Oils and Lubricants",612,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_620,Office Supplies and Postal Expenses,620,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_621,Informational and Educational Materials,621,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_622,Small Tools,622,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_630,IT Services,630,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_631,Software,631,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_632,Consultations and Trainings,632,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_633,Accounting Services,633,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_634,Legal Services,634,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_635,Auditing Services,635,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_636,Costs of Entertaining Guests,636,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_637,Marketing Expenses,637,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_640,Car Rental,640,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_641,Car Insurance,641,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_642,Car Fuel,642,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_643,Car Maintenance and Repairs,643,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_644,Other Car Expenses,644,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_650,Compensation for the Use of a Personal Car,650,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_651,Fringe Benefits to Employees,651,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_652,Salaries and Wages,652,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_653,Social Security Costs,653,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_654,Unemployment Insurance Premium,654,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_655,Pension Expenses,655,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_656,Vacation Pay Reserve,656,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6570,Income Tax on Fringe Benefits,6570,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6571,Social Tax on Fringe Benefits,6571,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6572,Social Tax,6572,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_660,State Fees,660,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_661,Land Tax,661,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_662,Income Tax,662,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_663,Fines and Fines for Delay,663,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_670,Bank Fees,670,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_671,Cash Rounding Losses,671,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_672,Payment Difference Losses,672,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_673,Currency Exchange Losses,673,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_674,Interest Expenses,674,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_675,Financial Expenses from Shares in Subsidiaries,675,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_676,Financial Expenses from Shares in Associates,676,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_677,Financial Expenses from Other Financial Investments,677,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_678,Other Financial Expenses,678,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_680,Capitalized Expenses in the Manufacturing of Fixed Assets for Own Use,680,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6810,Changes in Inventories of Finished Goods and Work in Progress,6810,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6811,Changes in Inventories of Agricultural Production,6811,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_682,Irrecoverable Receivables,682,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6830,Depreciations on Non-Current Assets,6830,expense_depreciation,l10n_ee.l10nee_chart_template,,False
l10n_ee_6831,Amortizations on Intangible Assets,6831,expense_depreciation,l10n_ee.l10nee_chart_template,,False
l10n_ee_6832,Loss on Sales of Biological Assets,6832,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6833,Loss on Sales of Non-Current Assets,6833,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6834,Significant Impairment of Current Assets,6834,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_684,Gifts and Donations,684,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6850,Cash Discount Losses,6850,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_6851,Other Operating Expenses,6851,expense,l10n_ee.l10nee_chart_template,,False
l10n_ee_70,Clearing Account,70,off_balance,l10n_ee.l10nee_chart_template,,False

```

## File: data\account.group.template.csv

```csv
id,code_prefix_start,code_prefix_end,name,chart_template_id/id
ee_group_1,1,,Assets,l10n_ee.l10nee_chart_template
ee_group_10,10,,Current Assets,l10n_ee.l10nee_chart_template
ee_group_100,100,,Cash,l10n_ee.l10nee_chart_template
ee_group_101,101,,Financial Investments,l10n_ee.l10nee_chart_template
ee_group_102,102,,Receivables and Prepayments,l10n_ee.l10nee_chart_template
ee_group_1020,1020,,Trade Receivables,l10n_ee.l10nee_chart_template
ee_group_1021,1021,,Receivables from Related Parties,l10n_ee.l10nee_chart_template
ee_group_1022,1022,,Prepaid and Deferred Taxes,l10n_ee.l10nee_chart_template
ee_group_1023,1023,,Loan Receivables,l10n_ee.l10nee_chart_template
ee_group_1024,1024,,Other Receivables,l10n_ee.l10nee_chart_template
ee_group_1025,1025,,Prepayments,l10n_ee.l10nee_chart_template
ee_group_103,103,,Inventory,l10n_ee.l10nee_chart_template
ee_group_1030,1030,,Raw and Other Materials,l10n_ee.l10nee_chart_template
ee_group_1031,1031,,Work in Progress,l10n_ee.l10nee_chart_template
ee_group_1032,1032,,Finished Goods,l10n_ee.l10nee_chart_template
ee_group_1033,1033,,Goods for Resale,l10n_ee.l10nee_chart_template
ee_group_1034,1034,,Prepayments to Suppliers,l10n_ee.l10nee_chart_template
ee_group_104,104,,Biological Assets,l10n_ee.l10nee_chart_template
ee_group_11,11,,Non-Current Assets,l10n_ee.l10nee_chart_template
ee_group_110,110,,Investments in Subsidiaries and Affiliates,l10n_ee.l10nee_chart_template
ee_group_1100,1100,,Shares and Participations in Subsidiaries,l10n_ee.l10nee_chart_template
ee_group_1101,1101,,Shares and Participations in Affiliates,l10n_ee.l10nee_chart_template
ee_group_111,111,,Financial Investments,l10n_ee.l10nee_chart_template
ee_group_112,112,,Receivables and Prepayments,l10n_ee.l10nee_chart_template
ee_group_1120,1120,,Long-Term Trade Receivables,l10n_ee.l10nee_chart_template
ee_group_1121,1121,,Long-Term Receivables from Related Parties,l10n_ee.l10nee_chart_template
ee_group_1122,1122,,Long-Term Prepaid and Deferred Taxes,l10n_ee.l10nee_chart_template
ee_group_1123,1123,,Long-Term Loan Receivables,l10n_ee.l10nee_chart_template
ee_group_1124,1124,,Other Long-Term Receivables,l10n_ee.l10nee_chart_template
ee_group_1125,1125,,Long-Term Prepayments,l10n_ee.l10nee_chart_template
ee_group_113,113,,Real Estate Investments,l10n_ee.l10nee_chart_template
ee_group_114,114,,Tangible Non-Current Assets,l10n_ee.l10nee_chart_template
ee_group_115,115,,Biological Assets,l10n_ee.l10nee_chart_template
ee_group_116,116,,Intangible Non-Current Assets,l10n_ee.l10nee_chart_template
ee_group_2,2,,Liabilities,l10n_ee.l10nee_chart_template
ee_group_20,20,,Current Liabilities,l10n_ee.l10nee_chart_template
ee_group_200,200,,Loan Liabilities,l10n_ee.l10nee_chart_template
ee_group_201,201,,Payables and Prepayments,l10n_ee.l10nee_chart_template
ee_group_2010,2010,,Accounts Payable,l10n_ee.l10nee_chart_template
ee_group_2011,2011,,Employee Payables,l10n_ee.l10nee_chart_template
ee_group_2012,2012,,Taxes Payable,l10n_ee.l10nee_chart_template
ee_group_2013,2013,,Other Payables,l10n_ee.l10nee_chart_template
ee_group_2014,2014,,Deferred Revenue,l10n_ee.l10nee_chart_template
ee_group_2015,2015,,Other Received Prepayments,l10n_ee.l10nee_chart_template
ee_group_202,202,,Provisions,l10n_ee.l10nee_chart_template
ee_group_2020,2020,,Warranty Provisions,l10n_ee.l10nee_chart_template
ee_group_2021,2021,,Tax Provisions,l10n_ee.l10nee_chart_template
ee_group_2022,2022,,Other Provisions,l10n_ee.l10nee_chart_template
ee_group_203,203,,Government Grants,l10n_ee.l10nee_chart_template
ee_group_21,21,,Non-Current Liabilities,l10n_ee.l10nee_chart_template
ee_group_210,210,,Long-Term Loan Liabilities,l10n_ee.l10nee_chart_template
ee_group_211,211,,Long-Term Payables and Prepayments,l10n_ee.l10nee_chart_template
ee_group_2110,2110,,Long-Term Accounts Payable,l10n_ee.l10nee_chart_template
ee_group_2111,2111,,Long-Term Employee Payables,l10n_ee.l10nee_chart_template
ee_group_2112,2112,,Long-Term Taxes Payable,l10n_ee.l10nee_chart_template
ee_group_2113,2113,,Other Long-Term Payables,l10n_ee.l10nee_chart_template
ee_group_2114,2114,,Long-Term Deferred Revenue,l10n_ee.l10nee_chart_template
ee_group_2115,2115,,Other Received Long-Term Prepayments,l10n_ee.l10nee_chart_template
ee_group_212,212,,Long-Term Provisions,l10n_ee.l10nee_chart_template
ee_group_2120,2120,,Long-Term Warranty Provisions,l10n_ee.l10nee_chart_template
ee_group_2121,2121,,Long-Term Tax Provisions,l10n_ee.l10nee_chart_template
ee_group_2122,2122,,Other Long-Term Provisions,l10n_ee.l10nee_chart_template
ee_group_213,213,,Long-Term Government Grants,l10n_ee.l10nee_chart_template
ee_group_3,3,,Equity,l10n_ee.l10nee_chart_template
ee_group_30,30,,Share Capital (Nominal Value),l10n_ee.l10nee_chart_template
ee_group_31,31,,Unregistered Share Capital or Equity,l10n_ee.l10nee_chart_template
ee_group_32,32,,Unpaid Share Capital,l10n_ee.l10nee_chart_template
ee_group_33,33,,Share Premium,l10n_ee.l10nee_chart_template
ee_group_34,34,,Own Shares,l10n_ee.l10nee_chart_template
ee_group_35,35,,Statutory Reserve Capital,l10n_ee.l10nee_chart_template
ee_group_36,36,,Other Reserves,l10n_ee.l10nee_chart_template
ee_group_37,37,,Other Equity,l10n_ee.l10nee_chart_template
ee_group_38,38,,Retained Profit/Loss From Previous Periods,l10n_ee.l10nee_chart_template
ee_group_39,39,,Profit/Loss for the Financial Year,l10n_ee.l10nee_chart_template
ee_group_4,4,,Income,l10n_ee.l10nee_chart_template
ee_group_40,40,,Sales of Goods and Services,l10n_ee.l10nee_chart_template
ee_group_400,400,,Sales of Goods and Services in Estonia,l10n_ee.l10nee_chart_template
ee_group_401,401,,Sales of Goods and Services in the EU,l10n_ee.l10nee_chart_template
ee_group_402,402,,Export of Goods and Services,l10n_ee.l10nee_chart_template
ee_group_41,41,,Sales of Assets,l10n_ee.l10nee_chart_template
ee_group_42,42,,Financial Income,l10n_ee.l10nee_chart_template
ee_group_43,43,,Other Income,l10n_ee.l10nee_chart_template
ee_group_5,5,,Cost of Goods Sold,l10n_ee.l10nee_chart_template
ee_group_6,6,,Other Expenses,l10n_ee.l10nee_chart_template
ee_group_60,60,,Property Expenses,l10n_ee.l10nee_chart_template
ee_group_600,600,,Buildings Rental,l10n_ee.l10nee_chart_template
ee_group_601,601,,Office Expenses,l10n_ee.l10nee_chart_template
ee_group_602,602,,Workshop Expenses,l10n_ee.l10nee_chart_template
ee_group_603,603,,Property Insurance,l10n_ee.l10nee_chart_template
ee_group_604,604,,Electricity / Gas,l10n_ee.l10nee_chart_template
ee_group_605,605,,Water,l10n_ee.l10nee_chart_template
ee_group_606,606,,Internet,l10n_ee.l10nee_chart_template
ee_group_607,607,,Phone Costs,l10n_ee.l10nee_chart_template
ee_group_61,61,,Equipment Expenses,l10n_ee.l10nee_chart_template
ee_group_62,62,,Product Expenses,l10n_ee.l10nee_chart_template
ee_group_63,63,,Service Expenses,l10n_ee.l10nee_chart_template
ee_group_64,64,,Car Expenses,l10n_ee.l10nee_chart_template
ee_group_65,65,,Employee Expenses,l10n_ee.l10nee_chart_template
ee_group_657,657,,Employee Expenses - Taxes,l10n_ee.l10nee_chart_template
ee_group_66,66,,Fees and Taxes,l10n_ee.l10nee_chart_template
ee_group_67,67,,Financial Expenses,l10n_ee.l10nee_chart_template
ee_group_68,68,,Other Operating Expenses,l10n_ee.l10nee_chart_template
ee_group_70,70,,Other Accounts,l10n_ee.l10nee_chart_template

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10nee_chart_template" model="account.chart.template">
        <field name="name">Estonian Chart of Accounts</field>
        <field name="code_digits">6</field>
        <field name="cash_account_code_prefix">1000</field>
        <field name="bank_account_code_prefix">1001</field>
        <field name="transfer_account_code_prefix">1008</field>
        <field name="currency_id" ref="base.EUR"/>
        <field name="country_id" ref="base.ee"/>
        <field name="spoken_languages" eval="'et_EE'"/>
    </record>
</odoo>

```

## File: data\account_chart_template_try_loading.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_ee.l10nee_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_fiscal_position_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Fiscal Position Templates -->
    <record id="afpt_national" model="account.fiscal.position.template">
        <field name="sequence">1</field>
        <field name="name">National</field>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="country_id" ref="base.ee"/>
    </record>
    <record id="afpt_eu_private" model="account.fiscal.position.template">
        <field name="sequence">2</field>
        <field name="name">EU Private</field>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_group_id" ref="base.europe"/>
    </record>
    <record id="afpt_eu_ic" model="account.fiscal.position.template">
        <field name="sequence">3</field>
        <field name="name">EU Intra-Community</field>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="country_group_id" ref="base.europe"/>
    </record>
    <record id="afpt_imp_exp" model="account.fiscal.position.template">
        <field name="sequence">4</field>
        <field name="name">Import/Export</field>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="auto_apply" eval="True"/>
    </record>

    <!-- Fiscal Position Account Templates-->

    <!-- Sales of Goods in Estonia > Sales of Goods in the EU -->
    <record id="afpat_eu_ic_1" model="account.fiscal.position.account.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="account_src_id" ref="l10n_ee.l10n_ee_40000"/>
        <field name="account_dest_id" ref="l10n_ee.l10n_ee_40100"/>
    </record>

    <!-- Sales of Biological Goods in Estonia > Sales of Biological Goods in the EU -->
    <record id="afpat_eu_ic_2" model="account.fiscal.position.account.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="account_src_id" ref="l10n_ee.l10n_ee_40001"/>
        <field name="account_dest_id" ref="l10n_ee.l10n_ee_40101"/>
    </record>

    <!-- Sales of Services in Estonia > Sales of Services in the EU -->
    <record id="afpat_eu_ic_3" model="account.fiscal.position.account.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="account_src_id" ref="l10n_ee.l10n_ee_4001"/>
        <field name="account_dest_id" ref="l10n_ee.l10n_ee_4011"/>
    </record>

    <!-- Sales of Goods in Estonia > Export of Goods -->
    <record id="afpat_imp_exp_1" model="account.fiscal.position.account.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="account_src_id" ref="l10n_ee.l10n_ee_40000"/>
        <field name="account_dest_id" ref="l10n_ee.l10n_ee_40200"/>
    </record>

    <!-- Sales of Biological Goods in Estonia > Export of Biological Goods -->
    <record id="afpat_imp_exp_2" model="account.fiscal.position.account.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="account_src_id" ref="l10n_ee.l10n_ee_40001"/>
        <field name="account_dest_id" ref="l10n_ee.l10n_ee_40201"/>
    </record>

    <!-- Sales of Services in Estonia > Export of Services -->
    <record id="afpat_imp_exp_3" model="account.fiscal.position.account.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="account_src_id" ref="l10n_ee.l10n_ee_4001"/>
        <field name="account_dest_id" ref="l10n_ee.l10n_ee_4021"/>
    </record>

    <!-- Fiscal Position Tax Templates -->

    <!-- EU IC -->

    <!-- Sales of Goods 20% -> Sales of Goods in EU 0% IC -->
    <record id="afptt_eu_ic_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_20_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_eu_g"/>
    </record>

    <!-- Sales of Goods 22% -> Sales of Goods in EU 0% IC -->
    <record id="afptt_eu_ic_1_22" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_22_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_eu_g"/>
    </record>

    <!-- Sales of Goods 24% -> Sales of Goods in EU 0% IC -->
    <record id="afptt_eu_ic_1_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_24_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_eu_g"/>
    </record>

    <!-- Sales of Services 20% -> Sales of Services in EU 0% IC -->
    <record id="afptt_eu_ic_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_20_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_eu_s"/>
    </record>

    <!-- Sales of Services 22% -> Sales of Services in EU 0% IC -->
    <record id="afptt_eu_ic_2_22" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_22_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_eu_s"/>
    </record>

    <!-- Sales of Services 24% -> Sales of Services in EU 0% IC -->
    <record id="afptt_eu_ic_2_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_24_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_eu_s"/>
    </record>

    <!-- Sales of Services 13% -> Sales of Services in EU 0% IC -->
    <record id="afptt_eu_ic_17" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_13_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_eu_s"/>
    </record>

    <!-- Sales of Goods 9% -> Sales of Goods in EU 0% IC -->
    <record id="afptt_eu_ic_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_9_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_eu_g"/>
    </record>

    <!-- Sales of Services 9% -> Sales of Services in EU 0% IC -->
    <record id="afptt_eu_ic_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_9_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_eu_s"/>
    </record>

    <!-- Sales of Goods 5% -> Sales of Goods in EU 0% IC -->
    <record id="afptt_eu_ic_5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_5_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_eu_g"/>
    </record>

    <!-- Sales of Services 5% -> Sales of Services in EU 0% IC -->
    <record id="afptt_eu_ic_6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_5_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_eu_s"/>
    </record>

    <!-- Sales of Goods 0% -> Sales of Goods in EU 0% IC -->
    <record id="afptt_eu_ic_7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_0_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_eu_g"/>
    </record>

    <!-- Sales of Services 0% -> Sales of Services in EU 0% IC -->
    <record id="afptt_eu_ic_8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_0_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_eu_s"/>
    </record>

    <!-- Purchase of Goods 20% -> Purchase of Goods in EU 0% IC -->
    <record id="afptt_eu_ic_9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_20_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_eu_g_22"/>
    </record>

    <!-- Purchase of Goods 22% -> Purchase of Goods in EU 0% IC -->
    <record id="afptt_eu_ic_19" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_22_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_eu_g_22"/>
    </record>

    <!-- Purchase of Goods 24% -> Purchase of Goods in EU 0% IC -->
    <record id="afptt_eu_ic_20" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_24_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_eu_g_24"/>
    </record>


    <!-- Purchase of Services 20% -> Purchase of Services in EU 0% IC -->
    <record id="afptt_eu_ic_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_20_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_eu_s_22"/>
    </record>

    <!-- Purchase of Services 22% -> Purchase of Services in EU 0% IC -->
    <record id="afptt_eu_ic_10_22" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_22_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_eu_s_22"/>
    </record>

    <!-- Purchase of Services 24% -> Purchase of Services in EU 0% IC -->
    <record id="afptt_eu_ic_10_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_24_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_eu_s_24"/>
    </record>

    <!-- Purchase of Services 13% -> Purchase of Services in EU 0% IC -->
    <record id="afptt_eu_ic_18" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_13_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_eu_s_22"/>
    </record>

    <!-- Purchase of Goods 9% -> Purchase of Goods in EU 0% IC -->
    <record id="afptt_eu_ic_11" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_9_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_eu_g_22"/>
    </record>

    <!-- Purchase of Services 9% -> Purchase of Services in EU 0% IC -->
    <record id="afptt_eu_ic_12" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_9_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_eu_s_22"/>
    </record>

    <!-- Purchase of Goods 5% -> Purchase of Goods in EU 0% IC -->
    <record id="afptt_eu_ic_13" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_5_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_eu_g_22"/>
    </record>

    <!-- Purchase of Services 5% -> Purchase of Services in EU 0% IC -->
    <record id="afptt_eu_ic_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_5_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_eu_s_22"/>
    </record>

    <!-- Purchase of Goods 0% -> Purchase of Goods in EU 0% IC -->
    <record id="afptt_eu_ic_15" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_0_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_eu_g_22"/>
    </record>

    <!-- Purchase of Services 0% -> Purchase of Services in EU 0% IC -->
    <record id="afptt_eu_ic_16" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_eu_ic"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_0_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_eu_s_22"/>
    </record>

    <!-- IMPORT/EXPORT -->

    <!-- Sales of Goods 20% -> Export of Goods 0% -->
    <record id="afptt_imp_exp_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_20_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_exp_g"/>
    </record>

    <!-- Sales of Goods 22% -> Export of Goods 0% -->
    <record id="afptt_imp_exp_1_22" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_22_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_exp_g"/>
    </record>

    <!-- Sales of Goods 24% -> Export of Goods 0% -->
    <record id="afptt_imp_exp_1_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_24_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_exp_g"/>
    </record>

    <!-- Sales of Services 20% -> Export of Services 0% -->
    <record id="afptt_imp_exp_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_20_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_exp_s"/>
    </record>

    <!-- Sales of Services 22% -> Export of Services 0% -->
    <record id="afptt_imp_exp_2_22" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_22_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_exp_s"/>
    </record>

    <!-- Sales of Services 24% -> Export of Services 0% -->
    <record id="afptt_imp_exp_2_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_24_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_exp_s"/>
    </record>

    <!-- Sales of Goods 9% -> Export of Goods 0% -->
    <record id="afptt_imp_exp_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_9_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_exp_g"/>
    </record>

    <!-- Sales of Services 13% -> Export of Services 0% -->
    <record id="afptt_imp_exp_17" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_13_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_exp_s"/>
    </record>

    <!-- Sales of Services 9% -> Export of Services 0% -->
    <record id="afptt_imp_exp_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_9_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_exp_s"/>
    </record>

    <!-- Sales of Goods 5% -> Export of Goods 0% -->
    <record id="afptt_imp_exp_5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_5_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_exp_g"/>
    </record>

    <!-- Sales of Services 5% -> Export of Services 0% -->
    <record id="afptt_imp_exp_6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_5_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_exp_s"/>
    </record>

    <!-- Sales of Goods 0% -> Export of Goods 0% -->
    <record id="afptt_imp_exp_7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_0_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_exp_g"/>
    </record>

    <!-- Sales of Services 0% -> Export of Services 0% -->
    <record id="afptt_imp_exp_8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_out_0_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_out_0_exp_s"/>
    </record>

    <!-- Purchase of Goods 20% -> Import 20% -->
    <record id="afptt_imp_exp_9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_20_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_20_imp_kms_38"/>
    </record>

    <!-- Purchase of Goods 22% -> Import 22% -->
    <record id="afptt_imp_exp_9_22" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_22_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_22_imp_kms_38"/>
    </record>

    <!-- Purchase of Goods 24% -> Import 24% -->
    <record id="afptt_imp_exp_9_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_24_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_24_imp_kms_38"/>
    </record>

    <!-- Purchase of Services 20% -> Import 20% -->
    <record id="afptt_imp_exp_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_20_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_20_imp_kms_38"/>
    </record>

    <!-- Purchase of Services 22% -> Import 22% -->
    <record id="afptt_imp_exp_10_22" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_22_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_22_imp_kms_38"/>
    </record>

    <!-- Purchase of Services 24% -> Import 24% -->
    <record id="afptt_imp_exp_10_24" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_24_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_24_imp_kms_38"/>
    </record>

    <!-- Purchase of Services 13% -> Import 13% -->
    <record id="afptt_imp_exp_18" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_13_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_13_imp_kms_38"/>
    </record>

    <!-- Purchase of Goods 9% -> Import 9% -->
    <record id="afptt_imp_exp_11" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_9_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_9_imp_kms_38"/>
    </record>

    <!-- Purchase of Services 9% -> Import 9% -->
    <record id="afptt_imp_exp_12" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_9_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_9_imp_kms_38"/>
    </record>

    <!-- Purchase of Goods 5% -> Import 5% -->
    <record id="afptt_imp_exp_13" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_5_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_5_imp_kms_38"/>
    </record>

    <!-- Purchase of Services 5% -> Import 5% -->
    <record id="afptt_imp_exp_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_5_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_5_imp_kms_38"/>
    </record>

    <!-- Purchase of Goods 0% -> Import 0% -->
    <record id="afptt_imp_exp_15" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_0_g"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_imp"/>
    </record>

    <!-- Purchase of Services 0% -> Import 0% -->
    <record id="afptt_imp_exp_16" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="afpt_imp_exp"/>
        <field name="tax_src_id" ref="l10n_ee_vat_in_0_s"/>
        <field name="tax_dest_id" ref="l10n_ee_vat_in_0_imp"/>
    </record>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_vat_20" model="account.tax.group">
            <field name="name">VAT 20%</field>
            <field name="country_id" ref="base.ee"/>
        </record>

        <record id="tax_group_vat_22" model="account.tax.group">
            <field name="name">VAT 22%</field>
            <field name="country_id" ref="base.ee"/>
        </record>

        <record id="tax_group_vat_24" model="account.tax.group">
            <field name="name">VAT 24%</field>
            <field name="country_id" ref="base.ee"/>
        </record>

        <record id="tax_group_vat_13" model="account.tax.group">
            <field name="name">VAT 13%</field>
            <field name="country_id" ref="base.ee"/>
        </record>

        <record id="tax_group_vat_9" model="account.tax.group">
            <field name="name">VAT 9%</field>
            <field name="country_id" ref="base.ee"/>
        </record>

        <record id="tax_group_vat_5" model="account.tax.group">
            <field name="name">VAT 5%</field>
            <field name="country_id" ref="base.ee"/>
        </record>

        <record id="tax_group_vat_0" model="account.tax.group">
            <field name="name">VAT 0%</field>
            <field name="country_id" ref="base.ee"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report_vat" model="account.report">
        <field name="name">VAT Report (KMD)</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ee"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_1" model="account.report.line">
                <field name="name">1 - Acts and transactions subject to tax at a rate of 22%</field>
                <field name="code">l10n_ee_vat_1</field>
                <field name="expression_ids">
                    <record id="tax_report_line_1_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">1</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_1_1" model="account.report.line">
                <field name="name">1¹ - Acts and transactions subject to tax at a rate of 20%</field>
                <field name="code">l10n_ee_vat_1_1</field>
                <field name="expression_ids">
                    <record id="tax_report_line_1_1_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">1_1</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_2" model="account.report.line">
                <field name="name">2 - Acts and transactions subject to tax at a rate of 9%</field>
                <field name="code">l10n_ee_vat_2</field>
                <field name="expression_ids">
                    <record id="tax_report_line_2_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">2</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_2_1" model="account.report.line">
                <field name="name">2¹ - Acts and transactions subject to tax at a rate of 5%</field>
                <field name="code">l10n_ee_vat_2_1</field>
                <field name="expression_ids">
                    <record id="tax_report_line_2_1_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">2_1</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_2_2" model="account.report.line">
                <field name="name">2² - Acts and transactions subject to tax at a rate of 13%</field>
                <field name="code">l10n_ee_vat_2_2</field>
                <field name="expression_ids">
                    <record id="tax_report_line_2_2_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">2_2</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_3" model="account.report.line">
                <field name="name">3 - Acts and transactions subject to tax at a rate of 0%, incl.</field>
                <field name="code">l10n_ee_vat_3</field>
                <field name="expression_ids">
                    <record id="tax_report_line_3_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ee_vat_3.tax_tags + l10n_ee_vat_3_1.balance + l10n_ee_vat_3_2.balance</field>
                    </record>
                    <record id="tax_report_line_3_tag" model="account.report.expression">
                        <field name="label">tax_tags</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">3</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_line_3_1" model="account.report.line">
                        <field name="name">3.1 - Intra-Community supply of goods and services provided to a taxable person or taxable person with limited liability of another Member State, total, incl.</field>
                        <field name="code">l10n_ee_vat_3_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_3_1_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ee_vat_3_1.tax_tags + l10n_ee_vat_3_1_1.balance</field>
                            </record>
                            <record id="tax_report_line_3_1_tag" model="account.report.expression">
                                <field name="label">tax_tags</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3_1</field>
                            </record>
                            <!-- Not visible, but used for EC Sales Report -->
                            <record id="tax_report_line_ec_services_tag" model="account.report.expression">
                                <field name="label">ec_services</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3_1_S</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_line_3_1_1" model="account.report.line">
                                <field name="name">3.1.1 - Intra-Community supply of goods</field>
                                <field name="code">l10n_ee_vat_3_1_1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_3_1_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3_1_1</field>
                                    </record>
                                    <!-- Not visible, but used for EC Sales Report -->
                                    <record id="tax_report_line_ec_goods_tag" model="account.report.expression">
                                        <field name="label">ec_goods</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3_1_1_G</field>
                                    </record>
                                    <record id="tax_report_line_ec_triangular_tag" model="account.report.expression">
                                        <field name="label">ec_triangular</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3_1_1_T</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_3_2" model="account.report.line">
                        <field name="name">3.2 - Exportation of goods, incl.</field>
                        <field name="code">l10n_ee_vat_3_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_3_2_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">l10n_ee_vat_3_2.tax_tags + l10n_ee_vat_3_2_1.balance</field>
                            </record>
                            <record id="tax_report_line_3_2_tag" model="account.report.expression">
                                <field name="label">tax_tags</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3_2</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_line_3_2_1" model="account.report.line">
                                <field name="name">3.2.1 - Sale to passengers with return of value added tax</field>
                                <field name="code">l10n_ee_vat_3_2_1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_3_2_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3_2_1</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_4" model="account.report.line">
                <field name="name">4 - Total amount of value added tax</field>
                <field name="code">l10n_ee_vat_4</field>
                <field name="expression_ids">
                    <record id="tax_report_line_4_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ee_vat_1.balance * 0.22 + l10n_ee_vat_1_1.balance * 0.2 + l10n_ee_vat_2.balance * 0.09 + l10n_ee_vat_2_1.balance * 0.05 + l10n_ee_vat_2_2.balance * 0.13</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_4_1" model="account.report.line">
                <field name="name">4¹ - Value added tax payable upon the import of the goods</field>
                <field name="code">l10n_ee_vat_4_1</field>
                <field name="expression_ids">
                    <record id="tax_report_line_4_1_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">4_1</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_5" model="account.report.line">
                <field name="name">5 - Total amount of input VAT subject to deduction pursuant to law, incl.</field>
                <field name="code">l10n_ee_vat_5</field>
                <field name="expression_ids">
                    <record id="tax_report_line_5_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ee_vat_5.tax_tags + l10n_ee_vat_5_1.balance + l10n_ee_vat_5_2.balance + l10n_ee_vat_5_3.balance + l10n_ee_vat_5_4.balance</field>
                    </record>
                    <record id="tax_report_line_5_tag" model="account.report.expression">
                        <field name="label">tax_tags</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">5</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_line_5_1" model="account.report.line">
                        <field name="name">5.1 - VAT paid or payable on import</field>
                        <field name="code">l10n_ee_vat_5_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_5_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5_1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_5_2" model="account.report.line">
                        <field name="name">5.2 - VAT paid or payable on acquisition of fixed assets</field>
                        <field name="code">l10n_ee_vat_5_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_5_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5_2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_5_3" model="account.report.line">
                        <field name="name">5.3 - VAT paid or payable on acquisition of a car used for business purposes (100%), and on acquisition of goods and receipt of services for such car</field>
                        <field name="code">l10n_ee_vat_5_3</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_5_3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5_3</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_line_5_3_cars" model="account.report.line">
                                <field name="name">Number of cars</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_5_3_cars_value" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="subformula">editable;rounding=0</field>
                                        <field name="figure_type">integer</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_5_4" model="account.report.line">
                        <field name="name">5.4 - VAT paid or payable on acquisition of a car used partially for business purposes, and on acquisition of goods and receipt of services for such car</field>
                        <field name="code">l10n_ee_vat_5_4</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_5_4_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5_4</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="tax_report_line_5_4_cars" model="account.report.line">
                                <field name="name">Number of cars</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_5_4_cars_value" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="subformula">editable;rounding=0</field>
                                        <field name="figure_type">integer</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_6" model="account.report.line">
                <field name="name">6 - Intra-Community acquisitions of goods and services received from a taxable person of another Member State, total, incl.</field>
                <field name="code">l10n_ee_vat_6</field>
                <field name="expression_ids">
                    <record id="tax_report_line_6_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ee_vat_6.tax_tags + l10n_ee_vat_6_1.balance</field>
                    </record>
                    <record id="tax_report_line_6_tag" model="account.report.expression">
                        <field name="label">tax_tags</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">6</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_line_6_1" model="account.report.line">
                        <field name="name">6.1 - Intra-Community acquisitions of goods</field>
                        <field name="code">l10n_ee_vat_6_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_6_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6_1</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_7" model="account.report.line">
                <field name="name">7 - Acquisition of other goods and services subject to VAT, incl.</field>
                <field name="code">l10n_ee_vat_7</field>
                <field name="expression_ids">
                    <record id="tax_report_line_7_balance" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ee_vat_7.tax_tags + l10n_ee_vat_7_1.balance</field>
                    </record>
                    <record id="tax_report_line_7_tag" model="account.report.expression">
                        <field name="label">tax_tags</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">7</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_line_7_1" model="account.report.line">
                        <field name="name">7.1 - Acquisition of immovables, scrap metal, precious metal and metal products subject to value added tax under the special arrangements (VAT Act §41¹)</field>
                        <field name="code">l10n_ee_vat_7_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_7_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">7_1</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_8" model="account.report.line">
                <field name="name">8 - Supply exempt from tax</field>
                <field name="code">l10n_ee_vat_8</field>
                <field name="expression_ids">
                    <record id="tax_report_line_8_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">8</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_9" model="account.report.line">
                <field name="name">9 - Supply of immovables, scrap metal, precious metal and metal products subject to value added tax under the special arrangements (VAT Act §41¹) and taxable value of goods to be installed or assembled in another Member State</field>
                <field name="code">l10n_ee_vat_9</field>
                <field name="expression_ids">
                    <record id="tax_report_line_9_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">9</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_10" model="account.report.line">
                <field name="name">10 - Adjustments (+)</field>
                <field name="code">l10n_ee_vat_10</field>
                <field name="expression_ids">
                    <record id="tax_report_line_10_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">10</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_11" model="account.report.line">
                <field name="name">11 - Adjustments (-)</field>
                <field name="code">l10n_ee_vat_11</field>
                <field name="expression_ids">
                    <record id="tax_report_line_11_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">11</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_12" model="account.report.line">
                <field name="name">12 - Value added tax payable</field>
                <field name="expression_ids">
                    <record id="tax_report_line_12_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">l10n_ee_vat_4.balance + l10n_ee_vat_4_1.balance - l10n_ee_vat_5.balance + l10n_ee_vat_10.balance - l10n_ee_vat_11.balance</field>
                        <field name="subformula">if_above(EUR(0))</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_13" model="account.report.line">
                <field name="name">13 - Overpaid value added tax</field>
                <field name="expression_ids">
                    <record id="tax_report_line_13_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">-(l10n_ee_vat_4.balance + l10n_ee_vat_4_1.balance - l10n_ee_vat_5.balance + l10n_ee_vat_10.balance - l10n_ee_vat_11.balance)</field>
                        <field name="subformula">if_above(EUR(0))</field>
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
    <!-- ========== SALES TAXES ========== -->

    <!-- Sales of Goods 20% -->
    <record id="l10n_ee_vat_out_20_g" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="name">20% G</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="description">20%</field>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_1_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_1_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Sales of Goods 22% -->
    <record id="l10n_ee_vat_out_22_g" model="account.tax.template">
        <field name="sequence">11</field>
        <field name="name">22% G</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="description">22%</field>
        <field name="tax_group_id" ref="tax_group_vat_22"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Sales of Goods 24% -->
    <record id="l10n_ee_vat_out_24_g" model="account.tax.template">
        <field name="sequence">12</field>
        <field name="name">24% G</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">24</field>
        <field name="amount_type">percent</field>
        <field name="description">24%</field>
        <field name="tax_group_id" ref="tax_group_vat_24"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Sales of Services 20% -->
    <record id="l10n_ee_vat_out_20_s" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="name">20% S</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="description">20%</field>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_1_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_1_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Sales of Services 22% -->
    <record id="l10n_ee_vat_out_22_s" model="account.tax.template">
        <field name="sequence">21</field>
        <field name="name">22% S</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="description">22%</field>
        <field name="tax_group_id" ref="tax_group_vat_22"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Sales of Services 24% -->
    <record id="l10n_ee_vat_out_24_s" model="account.tax.template">
        <field name="sequence">22</field>
        <field name="name">24% S</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">24</field>
        <field name="amount_type">percent</field>
        <field name="description">24%</field>
        <field name="tax_group_id" ref="tax_group_vat_24"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Sales of Goods 9% -->
    <record id="l10n_ee_vat_out_9_g" model="account.tax.template">
        <field name="sequence">30</field>
        <field name="name">9% G</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="description">9%</field>
        <field name="tax_group_id" ref="tax_group_vat_9"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_2_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_2_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Sales of Services 9% -->
    <record id="l10n_ee_vat_out_9_s" model="account.tax.template">
        <field name="sequence">40</field>
        <field name="name">9% S</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="description">9%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_9"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_2_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_2_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Sales of Goods 5% -->
    <record id="l10n_ee_vat_out_5_g" model="account.tax.template">
        <field name="sequence">50</field>
        <field name="name">5% G</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">5%</field>
        <field name="tax_group_id" ref="tax_group_vat_5"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_2_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_2_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Sales of Services 5% -->
    <record id="l10n_ee_vat_out_5_s" model="account.tax.template">
        <field name="sequence">60</field>
        <field name="name">5% S</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">5%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_5"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_2_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_2_1_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Sales of Services 13% -->
    <record id="l10n_ee_vat_out_13_s" model="account.tax.template">
        <field name="sequence">66</field>
        <field name="name">13% S</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="description">13%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_13"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_2_2_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_2_2_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Sales of Goods 0% -->
    <record id="l10n_ee_vat_out_0_g" model="account.tax.template">
        <field name="sequence">70</field>
        <field name="name">0% G</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">0%</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_3_tag')],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_3_tag')],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
    </record>

    <!-- Sales of Services 0% -->
    <record id="l10n_ee_vat_out_0_s" model="account.tax.template">
        <field name="sequence">80</field>
        <field name="name">0% S</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">0%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_3_tag')],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_3_tag')],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
    </record>

    <!-- Sales of Goods in EU 0% IC -->
    <record id="l10n_ee_vat_out_0_eu_g" model="account.tax.template">
        <field name="sequence">90</field>
        <field name="name">0% EU G</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">0% EU</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [
                    ref('tax_report_line_3_1_1_tag'),
                    ref('tax_report_line_ec_goods_tag'),
                ],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [
                    ref('tax_report_line_3_1_1_tag'),
                    ref('tax_report_line_ec_goods_tag'),
                ],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
    </record>

    <!-- Sales of Goods in EU (Reseller in Triangular Transaction) 0% IC -->
    <record id="l10n_ee_vat_out_0_eu_g_t" model="account.tax.template">
        <field name="sequence">100</field>
        <field name="name">0% EU G Trian</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">0% EU T</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_ec_triangular_tag')],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_ec_triangular_tag')],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
    </record>

    <!-- Sales of Services in EU 0% IC -->
    <record id="l10n_ee_vat_out_0_eu_s" model="account.tax.template">
        <field name="sequence">110</field>
        <field name="name">0% EU S</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">0% EU</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [
                    ref('tax_report_line_3_1_tag'),
                    ref('tax_report_line_ec_services_tag'),
                ],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [
                    ref('tax_report_line_3_1_tag'),
                    ref('tax_report_line_ec_services_tag'),
                ],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
    </record>

    <!-- Export of Goods 0% -->
    <record id="l10n_ee_vat_out_0_exp_g" model="account.tax.template">
        <field name="sequence">120</field>
        <field name="name">0% EX G</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">0%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [
                    ref('tax_report_line_3_2_tag'),
                ],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [
                    ref('tax_report_line_3_2_tag'),
                ],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
    </record>

    <!-- Sales to Passengers 0% -->
    <record id="l10n_ee_vat_out_0_pas" model="account.tax.template">
        <field name="sequence">130</field>
        <field name="name">0% Passengers</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">0%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_3_2_1_tag')],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_3_2_1_tag')],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
    </record>

    <!-- Export of Services 0% -->
    <record id="l10n_ee_vat_out_0_exp_s" model="account.tax.template">
        <field name="sequence">140</field>
        <field name="name">0% EX S</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">0%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_3_tag')],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_3_tag')],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
    </record>

    <!-- Sales exempt from VAT -->
    <record id="l10n_ee_vat_out_exempt" model="account.tax.template">
        <field name="sequence">150</field>
        <field name="name">0% Exempt</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">Exempt</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_8_tag')],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_8_tag')],
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
    </record>

    <!-- Sales 0% under KMS §41¹ -->
    <record id="l10n_ee_vat_out_0_kms_41_1" model="account.tax.template">
        <field name="sequence">160</field>
        <field name="name">0% KMS §41¹</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="description">20% Special</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
        <field name="l10n_ee_kmd_inf_code">2</field>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_9_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_9_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Sales 0% under KMS §41¹ 22% Special-->
    <record id="l10n_ee_vat_out_0_kms_41_2" model="account.tax.template">
        <field name="sequence">161</field>
        <field name="name">0% KMS §41¹</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="description">22% Special</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_22"/>
        <field name="l10n_ee_kmd_inf_code">2</field>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_9_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_9_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- ========== PURCHASE TAXES ========== -->

    <!-- Purchase of Goods 20% -->
    <record id="l10n_ee_vat_in_20_g" model="account.tax.template">
        <field name="sequence">170</field>
        <field name="name">20% G</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="description">20%</field>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
    </record>

    <!-- Purchase of Goods 22% -->
    <record id="l10n_ee_vat_in_22_g" model="account.tax.template">
        <field name="sequence">171</field>
        <field name="name">22% G</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="description">22%</field>
        <field name="tax_group_id" ref="tax_group_vat_22"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
    </record>

    <!-- Purchase of Goods 24% -->
    <record id="l10n_ee_vat_in_24_g" model="account.tax.template">
        <field name="sequence">172</field>
        <field name="name">24% G</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">24</field>
        <field name="amount_type">percent</field>
        <field name="description">24%</field>
        <field name="tax_group_id" ref="tax_group_vat_24"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
    </record>

    <!-- Purchase of Services 20% -->
    <record id="l10n_ee_vat_in_20_s" model="account.tax.template">
        <field name="sequence">180</field>
        <field name="name">20% S</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="description">20%</field>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
    </record>

    <!-- Purchase of Services 22% -->
    <record id="l10n_ee_vat_in_22_s" model="account.tax.template">
        <field name="sequence">181</field>
        <field name="name">22% S</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="description">22%</field>
        <field name="tax_group_id" ref="tax_group_vat_22"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
    </record>

    <!-- Purchase of Services 24% -->
    <record id="l10n_ee_vat_in_24_s" model="account.tax.template">
        <field name="sequence">182</field>
        <field name="name">24% S</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">24</field>
        <field name="amount_type">percent</field>
        <field name="description">24%</field>
        <field name="tax_group_id" ref="tax_group_vat_24"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
    </record>

    <!-- Purchase of Goods 9% -->
    <record id="l10n_ee_vat_in_9_g" model="account.tax.template">
        <field name="sequence">190</field>
        <field name="name">9% G</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="description">9%</field>
        <field name="tax_group_id" ref="tax_group_vat_9"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
    </record>

    <!-- Purchase of Services 9% -->
    <record id="l10n_ee_vat_in_9_s" model="account.tax.template">
        <field name="sequence">200</field>
        <field name="name">9% S</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="description">9%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_9"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
    </record>

    <!-- Purchase of Goods 5% -->
    <record id="l10n_ee_vat_in_5_g" model="account.tax.template">
        <field name="sequence">210</field>
        <field name="name">5% G</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">5%</field>
        <field name="tax_group_id" ref="tax_group_vat_5"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
    </record>

    <!-- Purchase of Services 5% -->
    <record id="l10n_ee_vat_in_5_s" model="account.tax.template">
        <field name="sequence">220</field>
        <field name="name">5% S</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">5%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_5"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
    </record>

    <!-- Purchase of Services 13% -->
    <record id="l10n_ee_vat_in_13_s" model="account.tax.template">
        <field name="sequence">225</field>
        <field name="name">13% S</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="description">13%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_13"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
        ]"/>
    </record>

    <!-- Purchase of Goods 0% -->
    <record id="l10n_ee_vat_in_0_g" model="account.tax.template">
        <field name="sequence">230</field>
        <field name="name">0% G</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">0%</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
    </record>

    <!-- Purchase of Services 0% -->
    <record id="l10n_ee_vat_in_0_s" model="account.tax.template">
        <field name="sequence">240</field>
        <field name="name">0% S</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">0%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
    </record>

    <!-- Purchase of Goods in EU 0% IC -->
    <record id="l10n_ee_vat_in_0_eu_g" model="account.tax.template">
        <field name="sequence">250</field>
        <field name="name">0% EU G 20%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="description">0% EU</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [
                    ref('tax_report_line_1_1_tag'),
                    ref('tax_report_line_6_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [
                    ref('tax_report_line_1_1_tag'),
                    ref('tax_report_line_6_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Purchase of Goods in EU 0% IC 22 %-->
    <record id="l10n_ee_vat_in_0_eu_g_22" model="account.tax.template">
        <field name="sequence">251</field>
        <field name="name">0% EU G 22%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="description">0% EU</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [
                    ref('tax_report_line_1_tag'),
                    ref('tax_report_line_6_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [
                    ref('tax_report_line_1_tag'),
                    ref('tax_report_line_6_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Purchase of Goods in EU 0% IC 24 %-->
    <record id="l10n_ee_vat_in_0_eu_g_24" model="account.tax.template">
        <field name="sequence">252</field>
        <field name="name">0% EU G 24%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">24</field>
        <field name="amount_type">percent</field>
        <field name="description">0% EU</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [
                    ref('tax_report_line_1_tag'),
                    ref('tax_report_line_6_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [
                    ref('tax_report_line_1_tag'),
                    ref('tax_report_line_6_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Purchase of Services in EU 0% IC -->
    <record id="l10n_ee_vat_in_0_eu_s" model="account.tax.template">
        <field name="sequence">260</field>
        <field name="name">0% EU S 20%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="description">0% EU</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [
                    ref('tax_report_line_1_1_tag'),
                    ref('tax_report_line_6_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [
                    ref('tax_report_line_1_1_tag'),
                    ref('tax_report_line_6_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Purchase of Services in EU 0% IC 22 %-->
    <record id="l10n_ee_vat_in_0_eu_s_22" model="account.tax.template">
        <field name="sequence">261</field>
        <field name="name">0% EU S 22%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="description">0% EU</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [
                    ref('tax_report_line_1_tag'),
                    ref('tax_report_line_6_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [
                    ref('tax_report_line_1_tag'),
                    ref('tax_report_line_6_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Purchase of Services in EU 0% IC 24 %-->
    <record id="l10n_ee_vat_in_0_eu_s_24" model="account.tax.template">
        <field name="sequence">262</field>
        <field name="name">0% EU S 24%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">24</field>
        <field name="amount_type">percent</field>
        <field name="description">0% EU</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [
                    ref('tax_report_line_1_tag'),
                    ref('tax_report_line_6_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [
                    ref('tax_report_line_1_tag'),
                    ref('tax_report_line_6_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Purchase 20% - Car -->
    <record id="l10n_ee_vat_in_20_car" model="account.tax.template">
        <field name="sequence">270</field>
        <field name="name">20% Car</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="description">20%</field>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="active" eval="False"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_3_tag'),
                ],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_3_tag'),
                ],
            }),
        ]"/>
    </record>

    <!-- Purchase 20% - Car 50% -->
    <record id="l10n_ee_vat_in_20_car_part" model="account.tax.template">
        <field name="sequence">280</field>
        <field name="name">20% Car 50%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="description">20%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_4_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_4_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <!-- Purchase 22% - Car -->
    <record id="l10n_ee_vat_in_22_car" model="account.tax.template">
        <field name="sequence">271</field>
        <field name="name">22% Car</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="description">22%</field>
        <field name="tax_group_id" ref="tax_group_vat_22"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_3_tag'),
                ],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_3_tag'),
                ],
            }),
        ]"/>
    </record>

    <!-- Purchase 24% - Car -->
    <record id="l10n_ee_vat_in_24_car" model="account.tax.template">
        <field name="sequence">272</field>
        <field name="name">24% Car</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">24</field>
        <field name="amount_type">percent</field>
        <field name="description">24%</field>
        <field name="tax_group_id" ref="tax_group_vat_24"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_3_tag'),
                ],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_3_tag'),
                ],
            }),
        ]"/>
    </record>

    <!-- Purchase 22% - Car 50% -->
    <record id="l10n_ee_vat_in_22_car_part" model="account.tax.template">
        <field name="sequence">281</field>
        <field name="name">22% Car 50%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="description">22%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_22"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_4_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_4_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <!-- Purchase 24% - Car 50% -->
    <record id="l10n_ee_vat_in_24_car_part" model="account.tax.template">
        <field name="sequence">282</field>
        <field name="name">24% Car 50%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">24</field>
        <field name="amount_type">percent</field>
        <field name="description">24%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_24"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_4_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_4_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <!-- Purchase 20% - Fixed Assets -->
    <record id="l10n_ee_vat_in_20_assets" model="account.tax.template">
        <field name="sequence">290</field>
        <field name="name">20% Fixed Assets</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="description">20%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201202'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_2_tag'),
                ],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201202'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_2_tag'),
                ],
            }),
        ]"/>
    </record>

    <!-- Purchase 22% - Fixed Assets -->
    <record id="l10n_ee_vat_in_22_assets" model="account.tax.template">
        <field name="sequence">291</field>
        <field name="name">22% Fixed Assets</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="description">22%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_22"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201202'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_2_tag'),
                ],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201202'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_2_tag'),
                ],
            }),
        ]"/>
    </record>

    <!-- Purchase 24% - Fixed Assets -->
    <record id="l10n_ee_vat_in_24_assets" model="account.tax.template">
        <field name="sequence">292</field>
        <field name="name">24% Fixed Assets</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">24</field>
        <field name="amount_type">percent</field>
        <field name="description">24%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_24"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201202'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_2_tag'),
                ],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201202'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_2_tag'),
                ],
            }),
        ]"/>
    </record>

    <!-- Import VAT at Customs -->
    <record id="l10n_ee_vat_in_imp_cus" model="account.tax.template">
        <field name="sequence">300</field>
        <field name="name">EX VAT Customs</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">Import VAT</field>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_1_tag'),
                ]
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_1_tag'),
                ]
            }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
    </record>

    <!-- Import 20% (KMS §38 - Reverse Charge) -->
    <record id="l10n_ee_vat_in_20_imp_kms_38" model="account.tax.template">
        <field name="sequence">310</field>
        <field name="name">20% EX KMS §38</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="description">20%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
                'minus_report_expression_ids': [ref('tax_report_line_4_1_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
                'plus_report_expression_ids': [ref('tax_report_line_4_1_tag')],
            }),
        ]"/>
    </record>

    <!-- Import 22% (KMS §38 - Reverse Charge) -->
    <record id="l10n_ee_vat_in_22_imp_kms_38" model="account.tax.template">
        <field name="sequence">311</field>
        <field name="name">22% EX KMS §38</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="description">22%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_22"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
                'minus_report_expression_ids': [ref('tax_report_line_4_1_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
                'plus_report_expression_ids': [ref('tax_report_line_4_1_tag')],
            }),
        ]"/>
    </record>

    <!-- Import 24% (KMS §38 - Reverse Charge) -->
    <record id="l10n_ee_vat_in_24_imp_kms_38" model="account.tax.template">
        <field name="sequence">312</field>
        <field name="name">24% EX KMS §38</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">24</field>
        <field name="amount_type">percent</field>
        <field name="description">24%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_24"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
                'minus_report_expression_ids': [ref('tax_report_line_4_1_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
                'plus_report_expression_ids': [ref('tax_report_line_4_1_tag')],
            }),
        ]"/>
    </record>

    <!-- Import 13% (KMS §38 - Reverse Charge) -->
    <record id="l10n_ee_vat_in_13_imp_kms_38" model="account.tax.template">
        <field name="sequence">312</field>
        <field name="name">13% EX KMS §38</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="description">13% EX KMS §38</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_13"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
                'minus_report_expression_ids': [ref('tax_report_line_4_1_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
                'plus_report_expression_ids': [ref('tax_report_line_4_1_tag')],
            }),
        ]"/>
    </record>

    <!-- Import 9% (KMS §38 - Reverse Charge) -->
    <record id="l10n_ee_vat_in_9_imp_kms_38" model="account.tax.template">
        <field name="sequence">330</field>
        <field name="name">9% EX KMS §38</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="description">9%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_9"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
                'minus_report_expression_ids': [ref('tax_report_line_4_1_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
                'plus_report_expression_ids': [ref('tax_report_line_4_1_tag')],
            }),
        ]"/>
    </record>

    <!-- Import 5% (KMS §38 - Reverse Charge) -->
    <record id="l10n_ee_vat_in_5_imp_kms_38" model="account.tax.template">
        <field name="sequence">350</field>
        <field name="name">5% EX KMS §38</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="description">5%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_5"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [
                    ref('tax_report_line_5_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
                'minus_report_expression_ids': [ref('tax_report_line_4_1_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [
                    ref('tax_report_line_5_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
                'plus_report_expression_ids': [ref('tax_report_line_4_1_tag')],
            }),
        ]"/>
    </record>

    <!-- Import 0% -->
    <record id="l10n_ee_vat_in_0_imp" model="account.tax.template">
        <field name="sequence">360</field>
        <field name="name">0% EX</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="description">0%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, { 'repartition_type': 'base' }),
            (0,0, { 'repartition_type': 'tax' }),
        ]"/>
    </record>

    <!-- Purchase 0% under KMS §41¹ -->
    <record id="l10n_ee_vat_in_0_kms_41_1" model="account.tax.template">
        <field name="sequence">370</field>
        <field name="name">0% KMS §41¹</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="description">0%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
        <field name="l10n_ee_kmd_inf_code">12</field>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [
                    ref('tax_report_line_1_1_tag'),
                    ref('tax_report_line_7_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [
                    ref('tax_report_line_1_1_tag'),
                    ref('tax_report_line_7_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>

    <!-- Purchase 0% under KMS §41¹ -->
    <record id="l10n_ee_vat_in_0_kms_41_2" model="account.tax.template">
        <field name="sequence">370</field>
        <field name="name">0% KMS §41¹</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="description">0%</field>
        <field name="active" eval="False"/>
        <field name="tax_group_id" ref="tax_group_vat_22"/>
        <field name="l10n_ee_kmd_inf_code">12</field>
        <field name="chart_template_id" ref="l10nee_chart_template"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [
                    ref('tax_report_line_1_tag'),
                    ref('tax_report_line_7_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'plus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [
                    ref('tax_report_line_1_tag'),
                    ref('tax_report_line_7_1_tag'),
                ],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201201'),
                'minus_report_expression_ids': [ref('tax_report_line_5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_ee_201204'),
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\l10n_ee_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10nee_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="l10n_ee_10200"/>
        <field name="property_account_payable_id" ref="l10n_ee_2010"/>
        <field name="property_account_income_categ_id" ref="l10n_ee_40000"/>
        <field name="property_account_expense_categ_id" ref="l10n_ee_50"/>
        <field name="income_currency_exchange_account_id" ref="l10n_ee_422"/>
        <field name="expense_currency_exchange_account_id" ref="l10n_ee_673"/>
        <field name="property_tax_receivable_account_id" ref="l10n_ee_201200"/>
        <field name="property_tax_payable_account_id" ref="l10n_ee_201200"/>
        <field name="default_pos_receivable_account_id" ref="l10n_ee_10201"/>
        <field name="account_journal_suspense_account_id" ref="l10n_ee_1009"/>
        <field name="default_cash_difference_income_account_id" ref="l10n_ee_420"/>
        <field name="default_cash_difference_expense_account_id" ref="l10n_ee_671"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="l10n_ee_430"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="l10n_ee_6850"/>
    </record>
</odoo>

```

## File: migrations\1.1\post-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo.addons.account.models.chart_template import update_taxes_from_templates


def migrate(cr, version):
    update_taxes_from_templates(cr, 'l10n_ee.l10nee_chart_template')

```

## File: migrations\1.2\post-migrate_update_taxes.py

```python
from odoo.addons.account.models.chart_template import update_taxes_from_templates


def migrate(cr, version):
    update_taxes_from_templates(cr, 'l10n_ee.l10nee_chart_template')

```

## File: models\account_tax.py

```python
from odoo import fields, models


class AccountTax(models.Model):
    _inherit = 'account.tax'

    l10n_ee_kmd_inf_code = fields.Selection(
        selection=[
            ('1', 'Sale KMS §41/42'),
            ('2', 'Sale KMS §41^1'),
            ('11', 'Purchase KMS §29(4)/30/32'),
            ('12', 'Purchase KMS §41^1'),
        ],
        string='KMD INF Code',
        default=False,
        help='This field is used for the comments/special code column in the KMD INF report.'
    )


class AccountTaxTemplate(models.Model):
    _inherit = 'account.tax.template'

    l10n_ee_kmd_inf_code = fields.Selection(
        selection=[
            ('1', 'Sale KMS §41/42'),
            ('2', 'Sale KMS §41^1'),
            ('11', 'Purchase KMS §29(4)/30/32'),
            ('12', 'Purchase KMS §41^1'),
        ],
        string='KMD INF Code',
        default=False,
        help='This field is used for the comments/special code column in the KMD INF report.'
    )

    def _get_tax_vals(self, company, tax_template_to_tax):
        # OVERRIDE
        vals = super()._get_tax_vals(company, tax_template_to_tax)
        vals.update({
            'l10n_ee_kmd_inf_code': self.l10n_ee_kmd_inf_code,
        })
        return vals

```

## File: models\__init__.py

```python
from . import account_tax

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106"><defs><mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse"><path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill:#fff;fill-rule:evenodd"/></mask><mask id="b" x="3.54" y="7.61" width="49.81" height="31.7" maskUnits="userSpaceOnUse"><rect x="4.08" y="7.61" width="48.45" height="31.57" rx="1" style="fill:#fff"/></mask><symbol id="c" viewBox="0 0 106 106"><g style="mask:url(#a)"><path d="M0,0H106V106H0Z" style="fill:#5a5a64;fill-rule:evenodd"/><path d="M6.06,1.51H98.43q6.06,0,7.57,3V0H0V4.54Q1.52,1.51,6.06,1.51Z" style="fill:#fff;fill-opacity:0.382999986410141;fill-rule:evenodd"/><path d="M6.06,104.49H98.43q6.06,0,7.57-4.55V106H0V99.94Q1.52,104.49,6.06,104.49Z" style="fill-opacity:0.382999986410141;fill-rule:evenodd"/><path d="M70.38,104.49H6.06C3,104.49,0,103,0,98.43V61.28L28.77,19.69H59.06a77.33,77.33,0,0,0,21.2,13.87c.07,11.31.07,4.86,0,16.17h3.12l.21,36.82Z" style="fill:#393939;fill-rule:evenodd;isolation:isolate;opacity:0.324000000953674"/><g style="opacity:0.30000000000000004"><path d="M68.77,58.54H76c.76,0,1,.12,1,.46v2.45c0,.31-.24.43-.93.43H61.44c-.66,0-.92-.12-.92-.42,0-.83,0-1.67,0-2.51,0-.29.26-.4.92-.41Z"/><path d="M64.33,77.42c.42.39.76.66,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,4.25,4.25,0,0,1-.48-.47c-.14-.15-.26-.31-.49-.6-.32.37-.54.66-.79.91-.53.53-1.08.58-1.5.15s-.36-.94.15-1.45c.26-.26.54-.5.91-.83-.38-.34-.72-.61-1-.91a.9.9,0,0,1,0-1.36.91.91,0,0,1,1.36,0c.29.28.54.6.93,1A12.1,12.1,0,0,1,64,75.18a.91.91,0,0,1,1.36,0,.87.87,0,0,1,0,1.31C65.07,76.79,64.73,77.06,64.33,77.42Z"/><path d="M62.13,66.9c0-.47,0-.88,0-1.28a.92.92,0,0,1,.92-1,.91.91,0,0,1,1,1c0,.41,0,.81,0,1.3h1.14a1.16,1.16,0,0,1,1.22,1c0,.55-.42.85-1.18.86H64.12c0,.49,0,.91,0,1.34a.94.94,0,1,1-1.88,0c0-.41,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88C61.3,66.89,61.68,66.9,62.13,66.9Z"/><path d="M74.31,76H72.23c-.67,0-1-.34-1-.93a.89.89,0,0,1,1-1q2.18,0,4.35,0a1,1,0,1,1,0,1.91c-.74,0-1.47,0-2.21,0Z"/><path d="M74.28,68.61c-.71,0-1.43,0-2.14,0a.86.86,0,0,1-1-.9.85.85,0,0,1,.92-1c1.5,0,3,0,4.48,0a.93.93,0,0,1,1,1,.91.91,0,0,1-1,.91c-.75,0-1.51,0-2.27,0Z"/><path d="M74.36,78.09c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.57-.38.93-1,.94H72.28c-.75,0-1.09-.32-1.09-.94s.37-1,1.09-1,1.39,0,2.08,0Z"/><path d="M81.29,90.55H56.14a4,4,0,0,1-4-4V53.73a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V86.55A4,4,0,0,1,81.29,90.55ZM56.14,53.73V86.55H81.29V53.73Z"/><path d="M43.49,83.26H31.8V25.71H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V34.8c-4.55-3-16.66-12.11-19.69-13.63H30.29a2.68,2.68,0,0,0-3,3V84.77a2.68,2.68,0,0,0,3,3H48.45V83.26ZM60.57,25.71l15.14,10.6H60.57Z"/></g><path d="M60.57,18.68H30.29a2.68,2.68,0,0,0-3,3V82.28a2.68,2.68,0,0,0,3,3H48.45V80.77H31.8V23.22H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V32.31C75.71,29.28,63.6,20.2,60.57,18.68Zm0,15.14V23.22l15.14,10.6Z" style="fill:#a8a9ab"/><path d="M68.77,55.78H76c.76,0,1,.13,1,.53v2.85c0,.37-.24.5-.93.5q-7.3,0-14.61,0c-.66,0-.92-.14-.92-.48,0-1,0-2,0-2.93,0-.34.26-.47.92-.47Z" style="fill:#a8a9ab"/><path d="M64.33,76.53c.42.38.76.65,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,5.44,5.44,0,0,1-.48-.48c-.14-.14-.26-.31-.49-.59-.32.36-.54.65-.79.91-.53.53-1.08.57-1.5.14s-.36-.94.15-1.45c.26-.26.54-.49.91-.82-.38-.35-.72-.61-1-.92a.9.9,0,0,1,0-1.36.92.92,0,0,1,1.36,0c.29.28.54.61.93,1A13.78,13.78,0,0,1,64,74.28a.91.91,0,0,1,1.36,0,.88.88,0,0,1,0,1.32C65.07,75.89,64.73,76.16,64.33,76.53Z" style="fill:#a8a9ab"/><path d="M62.13,65.88c0-.48,0-.88,0-1.29a1,1,0,1,1,1.91,0c0,.4,0,.81,0,1.3h1.14a1.15,1.15,0,0,1,1.22,1c0,.54-.42.85-1.18.85H64.12c0,.49,0,.92,0,1.34a.94.94,0,1,1-1.88,0c0-.4,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88Z" style="fill:#a8a9ab"/><path d="M74.31,75.11c-.69,0-1.38,0-2.08,0s-1-.35-1-.94a.89.89,0,0,1,1-1q2.18,0,4.35,0a.91.91,0,0,1,1,1,.93.93,0,0,1-1,1c-.74,0-1.47,0-2.21,0Z" style="fill:#a8a9ab"/><path d="M74.28,67.76H72.14a.87.87,0,0,1-1-.9.84.84,0,0,1,.92-1c1.5,0,3,0,4.48,0a.94.94,0,0,1,1,1,.91.91,0,0,1-1,.91H74.28Z" style="fill:#a8a9ab"/><path d="M74.36,77.2c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.56-.38.93-1,.93q-2.12,0-4.23,0c-.75,0-1.09-.32-1.09-.94s.37-.94,1.09-1,1.39,0,2.08,0Z" style="fill:#a8a9ab"/><path d="M81.29,88.06H56.14a4,4,0,0,1-4-4V51.24a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V84.06A4,4,0,0,1,81.29,88.06ZM56.14,51.24V84.06H81.29V51.24Z" style="fill:#a8a9ab"/></g></symbol></defs><rect x="4" y="10.57" width="48.45" height="31.57" rx="1" style="fill:#393939;isolation:isolate;opacity:0.44"/><use width="106" height="106" xlink:href="#c"/><g style="mask:url(#b)"><rect x="3.54" y="7.61" width="49.81" height="31.7" style="fill:#fff"/><rect x="3.54" y="7.61" width="49.81" height="21.13"/><rect x="3.54" y="7.61" width="49.81" height="10.57" style="fill:#0072ce"/></g></svg>
```

## File: views\account_tax_form.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_tax_form_inherit_l10n_ee" model="ir.ui.view">
            <field name="name">account.tax.form</field>
            <field name="model">account.tax</field>
            <field name="inherit_id" ref="account.view_tax_form"/>
            <field name="arch" type="xml">
                <field name="country_id" position="after">
                    <field name="l10n_ee_kmd_inf_code" attrs="{'invisible': [('country_code', '!=', 'EE')]}"/>
                </field>
            </field>
        </record>
    </data>
</odoo>

```

