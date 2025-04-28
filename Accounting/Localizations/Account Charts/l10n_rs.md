# Odoo Module: l10n_rs

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    "name": "Serbia - Accounting",
    "version": "1.0",
    "category": "Accounting/Localizations/Account Charts",
    "description": """
        This is the base module of the Serbian localization. It manages chart of accounts and taxes.
    """,
    "author": "Modoolar, Odoo S.A.",
    "depends": ["account", "base_vat"],
    "data": [
        "data/account_chart_template_data.xml",
        "data/account.account.template.csv",
        "data/l10n_rs_chart_data.xml",
        "data/account_tax_group_data.xml",
        'data/account_tax_report_data.xml',
        "data/account_tax_template_data.xml",
        "data/fiscal_position_data.xml",
        "data/account.group.template.csv",
        "data/account_chart_template_configure_data.xml",
        "data/menuitem_data.xml",
        "views/account_move.xml",
        "views/report_invoice.xml",
    ],
    "demo": ["demo/demo_company.xml"],
    "license": "LGPL-3",
}

```

## File: data\account.account.template.csv

```csv
id,name,code,account_type,chart_template_id/id,tag_ids/id,reconcile
rs_000,Subscribed shares unpaid,000,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_001,Subscribed stakes unpaid,001,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_010,Investment in development,010,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_011,"Concessions, patents, licenses, goods and services brands",011,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_012,Software and other rights,012,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_013,Godwill,013,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_014,Other intangible assets,014,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_015,Intangible assets under preparation,015,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_016,Advances for acquisition of intangible assets,016,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_019,Provisions for acquisition of intangible assets,019,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_020,Agricultural and other land,020,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_021,Building land,021,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_022,Buildings,022,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_023,Plant and equipment,023,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_024,Investment property,024,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_025,"Other property, plant and equipment",025,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_026,"Property, plant and equipment under construction",026,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_027,"Investment in other property, plant and equipment",027,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_028,"Advances for property, plant and equipment",028,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_029,"Provisions for property, plant and equipment",029,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_030,Forests,030,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_031,Plantations,031,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_032,Livestock,032,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_037,Natural assets under construction,037,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_038,Advances for natural assets,038,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_039,Provisions of natural assets,039,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_040,Investments in capital of parent companies and subsidiaries,040,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_041,Investments in capital of associated legal entities and joint ventures,041,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_042,Investments in other legal entities and other securities available for sale,042,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_0430,"Long-term domestic loans to parent companies, to subsidiaries",0430,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_0431,Long-term domestic loans to other associated companies,0431,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_0440,"Long-term foreign loans to parent companies, to subsidiaries",0440,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_0441,Long-term foreign loans to other associated companies,0441,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_0450,Long term domestic loans,0450,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_0451,Long term foreign loans,0451,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_046,Securities held to maturity,046,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_047,Own stocks and shares purchased,047,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_048,Other long term financial investments,048,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_049,Provisions for long-term financial investments,049,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_050,Receivables from parent and dependent legal entities,050,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_051,Receivables from other related parties,051,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_052,Receivables based on sales on commodity credit,052,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_053,Receivables for sale under financial lease agreements,053,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_054,Receivables based on warranty,054,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_055,Contested and doubtful receivables,055,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_056,Other long term receivables,056,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_059,Provisions of long term receivables,059,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_100,Material purchase costs calculation,100,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_101,Raw materials,101,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_102,Spare parts,102,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_103,Tools and inventories,103,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_104,"Raw materials, spare parts, tools and inventories being processed, finished off and handled",104,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_109,Provisions for material,109,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_110,Work in progress,110,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_111,Services in progress,111,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_120,Finished products,120,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_130,Merchandise cost accounting,130,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_131,Merchandise in warehouse,131,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_132,Wholesale merchandise,132,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_133,"Merchandise in storehouse, depot and other legal entities' shops",133,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_134,Merchandise in retail sale,134,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_135,"Merchandise being processed, finished off and handled",135,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_136,Merchandise in transit,136,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_137,Merchandise in transport,137,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_139,Provisions for merchandise,139,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_140,Intangible assets for trading,140,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_141,Land held for trading,141,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_142,Buildings held for trading,142,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_143,Investment property for trading,143,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_144,Other property held for trading,144,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_145,Installations and equipment for trading,145,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_146,Natural assets for trading,146,asset_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_147,Assets of suspended business,147,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_149,Provisions for assets held for trading,149,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_150,"Domestic advances paid for raw material, spare parts and inventories",150,asset_prepayments,l10n_rs.l10n_rs_chart_template,,False
rs_151,"Foreign advances paid for raw material, spare parts and inventories",151,asset_prepayments,l10n_rs.l10n_rs_chart_template,,False
rs_152,Domestic advances paid for merchandise,152,asset_prepayments,l10n_rs.l10n_rs_chart_template,,False
rs_153,Foreign advances paid for merchandise,153,asset_prepayments,l10n_rs.l10n_rs_chart_template,,False
rs_154,Domestic advances paid for services,154,asset_prepayments,l10n_rs.l10n_rs_chart_template,,False
rs_155,Foreign advances paid for services,155,asset_prepayments,l10n_rs.l10n_rs_chart_template,,False
rs_159,Provisions for advances paid,159,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_200,Domestic trade receivables - parent companies and subsidiaries,200,asset_receivable,l10n_rs.l10n_rs_chart_template,,True
rs_201,Foreign trade receivables - parent companies and subsidiaries,201,asset_receivable,l10n_rs.l10n_rs_chart_template,,True
rs_202,Domestic trade receivables - other associated entities,202,asset_receivable,l10n_rs.l10n_rs_chart_template,,True
rs_203,Foreign trade receivables - other associated entities,203,asset_receivable,l10n_rs.l10n_rs_chart_template,,True
rs_204,Trade receivables - domestic third party,204,asset_receivable,l10n_rs.l10n_rs_chart_template,,True
rs_205,Trade receivables - foreign third party,205,asset_receivable,l10n_rs.l10n_rs_chart_template,,True
rs_206,Other trade receivables,206,asset_receivable,l10n_rs.l10n_rs_chart_template,,True
rs_209,Provisions for trade receivables,209,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_210,Receivables from exporters,210,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_211,Receivables due from import on behalf of other entities,211,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_212,Receivables from sales on commission and consignment,212,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_218,Other receivables from specific business operations,218,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_219,Provisions for receivables from specific business operations,219,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_220,Receivables for interest and dividends,220,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_221,Receivables from employees,221,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_222,Receivables from state authorities and organizations,222,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_223,Receivables for prepaid income tax,223,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_224,Receivables for other taxes and contributions prepaid,224,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_225,Receivables for compensation of salaries that are refunded,225,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_226,Receivables based on damages,226,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_228,Other short term receivables,228,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_229,Provisions for other receivables,229,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_230,Short-term loans and investments in parent companies and subsidiaries,230,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_231,Short-term loans and investments in other associated companies,231,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_232,Short term loans - domestic,232,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_233,Short term loans - foreign,233,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_234,Current portions of long term loan due within one year,234,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_235,Current portions of securities held to maturity - a part that reaches one year,235,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_236,Financial assets that are measured at fair value through the Income statement,236,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_237,Bought up own shares for trading,237,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_238,Other short-term investments,238,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_239,Provisions for short - term investments,239,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_240,Securities - cash equivalents,240,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_241,Current (business) accounts,241,asset_cash,l10n_rs.l10n_rs_chart_template,,False
rs_242,Cash allocated for payments and open letters of credit,242,asset_cash,l10n_rs.l10n_rs_chart_template,,False
rs_243,Cash in hand,243,asset_cash,l10n_rs.l10n_rs_chart_template,,False
rs_244,Foreign exchange account,244,asset_cash,l10n_rs.l10n_rs_chart_template,,False
rs_245,Foreign currency letters of credit,245,asset_cash,l10n_rs.l10n_rs_chart_template,,False
rs_246,Foreign currency - cash in hand,246,asset_cash,l10n_rs.l10n_rs_chart_template,,False
rs_248,Other cash and cash equivalents,248,asset_cash,l10n_rs.l10n_rs_chart_template,,False
rs_249,Blocked bank accounts,249,asset_cash,l10n_rs.l10n_rs_chart_template,,False
rs_270,VAT in incoming invoices - general tax rate (except advances paid),270,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_271,VAT in incoming invoices - specific tax rate (except advances paid),271,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_272,VAT in advances paid - general tax rate,272,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_273,VAT in advances paid - specific tax rate,273,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_274,VAT paid for import of goods - general tax rate,274,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_275,VAT paid for import of goods - specific tax rate,275,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_276,VAT on services rendered by foreign suppliers,276,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_277,VAT returned to foreign customers,277,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_278,VAT reimbursement paid to farmers,278,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_279,Receivables for prepaid VAT,279,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_280,Prepaid costs,280,asset_prepayments,l10n_rs.l10n_rs_chart_template,,False
rs_281,Accrued income (un-invoiced income),281,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_282,Deferred costs based on obligations,282,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_288,Deferred tax assets,288,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_289,Other accruals,289,asset_current,l10n_rs.l10n_rs_chart_template,,False
rs_300,Share capital,300,equity,l10n_rs.l10n_rs_chart_template,,False
rs_301,Stakes in limited liability companies,301,equity,l10n_rs.l10n_rs_chart_template,,False
rs_302,Participating interests,302,equity,l10n_rs.l10n_rs_chart_template,,False
rs_303,State owned capital,303,equity,l10n_rs.l10n_rs_chart_template,,False
rs_304,Socially owned capital,304,equity,l10n_rs.l10n_rs_chart_template,,False
rs_305,Stakes in co-operatives,305,equity,l10n_rs.l10n_rs_chart_template,,False
rs_306,Share issuing premiums,306,equity,l10n_rs.l10n_rs_chart_template,,False
rs_309,Other capital,309,equity,l10n_rs.l10n_rs_chart_template,,False
rs_310,Subscribed share capital unpaid,310,equity,l10n_rs.l10n_rs_chart_template,,False
rs_311,Subscribed stakes unpaid,311,equity,l10n_rs.l10n_rs_chart_template,,False
rs_321,Legal reserves,321,equity,l10n_rs.l10n_rs_chart_template,,False
rs_322,Statutory and other reserves,322,equity,l10n_rs.l10n_rs_chart_template,,False
rs_330,"Effect of restatement of capital based on the revaluation of intangible assets, property, plant and equipment",330,equity,l10n_rs.l10n_rs_chart_template,,False
rs_331,Actuarial gains or losses on planned benefit plans,331,equity,l10n_rs.l10n_rs_chart_template,,False
rs_332,Gains or losses on investments in equity instruments,332,equity,l10n_rs.l10n_rs_chart_template,,False
rs_333,Gains or losses on the basis of the share in other comprehensive income or loss of associates,333,equity,l10n_rs.l10n_rs_chart_template,,False
rs_334,Gains or losses arising from the calculations of financial statements of foreign operations,334,equity,l10n_rs.l10n_rs_chart_template,,False
rs_335,Gains or losses on instruments for the protection of net investments in foreign operations,335,equity,l10n_rs.l10n_rs_chart_template,,False
rs_336,Gains or losses arising from hedging instruments of cash flow,336,equity,l10n_rs.l10n_rs_chart_template,,False
rs_337,Gains or losses on securities available for trading,337,equity,l10n_rs.l10n_rs_chart_template,,False
rs_340,Retained profit from previous years,340,equity,l10n_rs.l10n_rs_chart_template,,False
rs_341,Retained profit from current year,341,equity_unaffected,l10n_rs.l10n_rs_chart_template,,False
rs_350,Previous year's losses,350,equity,l10n_rs.l10n_rs_chart_template,,False
rs_351,Current year loss,351,equity,l10n_rs.l10n_rs_chart_template,,False
rs_400,Provisions for costs incurred during the warranty period,400,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_401,Provisions for the recovery of natural resources,401,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_402,Provisions for retained deposits and caution money,402,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_403,Provisions for restructuring costs,403,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_404,Provisions for contribution and other employee benefits,404,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_405,Provision for court costs,405,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_409,Other long-term provisions,409,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_410,Liabilities which can be converted into capital,410,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_411,Liabilities to parent companies and subsidiaries,411,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_412,Liabilities to other associated companies,412,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_413,Liabilities for long-term securities,413,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_414,Long-term loans - domestic,414,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_415,Long-term loans - foreign,415,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_416,Liabilities based on financial leasing,416,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_419,Other long-term liabilities,419,liability_non_current,l10n_rs.l10n_rs_chart_template,,False
rs_420,Short-term loans from parent companies and subsidiaries,420,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_421,Short-term loans from other associated companies,421,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_422,Short term loans - domestic,422,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_423,Short term loans - foreign,423,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_424,Current portion of long-term loans,424,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_425,Current portion of other long-term liabilities,425,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_426,Liabilities for short-term securities,426,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_427,Liabilities based on fixed assets and assets of suspended business for trading,427,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_429,Other short-term financial liabilities,429,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_430,"Received advances, deposits and caution money",430,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_431,Domestic trade payables - parent companies and subsidiaries,431,liability_payable,l10n_rs.l10n_rs_chart_template,,True
rs_432,Foreign trade payables - parent companies and subsidiaries,432,liability_payable,l10n_rs.l10n_rs_chart_template,,True
rs_433,Domestic trade payables - other associated companies,433,liability_payable,l10n_rs.l10n_rs_chart_template,,True
rs_434,Foreign trade payables - other associated companies,434,liability_payable,l10n_rs.l10n_rs_chart_template,,True
rs_435,Trade payables - domestic,435,liability_payable,l10n_rs.l10n_rs_chart_template,,True
rs_436,Trade payables - foreign,436,liability_payable,l10n_rs.l10n_rs_chart_template,,True
rs_439,Other liabilities from business operations,439,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_440,Liabilities to importers,440,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_441,Liabilities due to exporters made on behalf of third parties,441,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_442,Liabilities for sale on commission and consignment,442,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_449,Other liabilities from specific business operations,449,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_450,"Liabilities for net salaries and fringe benefits, except refundable fringe benefits",450,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_451,Liabilities for taxes on salaries and fringe benefits charged to employees,451,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_452,Liabilities for contributions on salaries and fringe benefits charged to employees,452,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_453,Liabilities for taxes and contributions on salaries and fringe benefits charged to employer,453,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_454,Liabilities for refundable net fringe benefits,454,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_455,Liabilities for taxes and contributions on refundable fringe benefits charged to employees,455,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_456,Liabilities for taxes and contributions on refundable fringe benefits charged to employer,456,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_460,Liabilities for interests and finance costs,460,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_461,Liabilities for dividends,461,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_462,Liabilities for share in the profit,462,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_463,Liabilities to employees,463,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_464,Liabilities to members of Management Board and Supervisory Board,464,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_465,Liabilities to individuals for contracted fees,465,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_466,Liabilities for net income of sole traders who obtain advances during the year,466,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_467,Liabilities for short-term provisions,467,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_469,Other liabilities,469,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_470,Liabilities for VAT in outgoing invoices - general tax rate (except advances received),470,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_471,Liabilities for VAT in outgoing invoices - specific tax rate (except advances received),471,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_472,Liabilities for VAT in advances received - general tax rate,472,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_473,Liabilities for VAT in advances received - specific tax rate,473,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_474,Liabilities for VAT on internal consumption - general tax rate,474,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_475,Liabilities for VAT on internal consumption - specific tax rate,475,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_476,Liabilities for VAT on sales for cash,476,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_479,Liabilities for VAT on difference between calculated VAT and previous taxes,479,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_480,Liabilities for excise duties,480,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_481,Liabilities for income tax,481,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_482,"Liabilities for taxes, customs, and other duties charged to costs",482,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_483,Liabilities for contributions charged to costs,483,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_489,"Other liabilities for taxes, contributions and other duties",489,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_490,Accrued expenses,490,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_491,Revenue charged in advance,491,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_494,Accrued direct purchase costs,494,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_495,Donations received,495,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_496,Accrued income from receivables,496,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_498,Deferred tax liabilities,498,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_499,Other accruals and deferred income,499,liability_current,l10n_rs.l10n_rs_chart_template,,False
rs_500,Purchases of merchandises (goods for resale),500,expense,l10n_rs.l10n_rs_chart_template,,False
rs_501,Cost of merchandise sold,501,expense,l10n_rs.l10n_rs_chart_template,,False
rs_502,Cost of sold assets held for trading,502,expense,l10n_rs.l10n_rs_chart_template,,False
rs_503,Cost of other assets held for trading,503,expense,l10n_rs.l10n_rs_chart_template,,False
rs_510,Costs material,510,expense,l10n_rs.l10n_rs_chart_template,,False
rs_511,Costs of raw material,511,expense,l10n_rs.l10n_rs_chart_template,,False
rs_512,Costs of other material (overhead),512,expense,l10n_rs.l10n_rs_chart_template,,False
rs_513,Costs of fuel and energy,513,expense,l10n_rs.l10n_rs_chart_template,,False
rs_514,Costs of spare parts,514,expense,l10n_rs.l10n_rs_chart_template,,False
rs_515,Costs of One-time write-off for tools and inventory,515,expense,l10n_rs.l10n_rs_chart_template,,False
rs_520,Costs of salaries and fringe benefits (gross),520,expense,l10n_rs.l10n_rs_chart_template,,False
rs_521,Costs of taxes and contributions on salaries and fringe benefits charged to employer,521,expense,l10n_rs.l10n_rs_chart_template,,False
rs_522,Costs of remunerations according to temporary service contracts,522,expense,l10n_rs.l10n_rs_chart_template,,False
rs_523,Costs of remunerations according to author's contracts,523,expense,l10n_rs.l10n_rs_chart_template,,False
rs_524,Costs of remunerations according to temporary and provisional contracts,524,expense,l10n_rs.l10n_rs_chart_template,,False
rs_525,Costs of remunerations to individuals according to other contracts,525,expense,l10n_rs.l10n_rs_chart_template,,False
rs_526,"Costs of remuneration to manager, members of Management Board and Supervisory Board",526,expense,l10n_rs.l10n_rs_chart_template,,False
rs_529,Other personal expenses remunerations,529,expense,l10n_rs.l10n_rs_chart_template,,False
rs_530,Costs of services used in production process of own costs capitalized,530,expense,l10n_rs.l10n_rs_chart_template,,False
rs_531,Transport services costs,531,expense,l10n_rs.l10n_rs_chart_template,,False
rs_532,Maintenance costs,532,expense,l10n_rs.l10n_rs_chart_template,,False
rs_533,Rental costs,533,expense,l10n_rs.l10n_rs_chart_template,,False
rs_534,Fairs exhibit costs,534,expense,l10n_rs.l10n_rs_chart_template,,False
rs_535,Advertising costs,535,expense,l10n_rs.l10n_rs_chart_template,,False
rs_536,Costs of researching activities,536,expense,l10n_rs.l10n_rs_chart_template,,False
rs_537,Costs of development that are not capitalized,537,expense,l10n_rs.l10n_rs_chart_template,,False
rs_539,Costs of other services,539,expense,l10n_rs.l10n_rs_chart_template,,False
rs_540,Depreciation costs,540,expense_depreciation,l10n_rs.l10n_rs_chart_template,,False
rs_541,Costs of provisions during the warranty period,541,expense,l10n_rs.l10n_rs_chart_template,,False
rs_542,Provisions for recovery of natural resources,542,expense,l10n_rs.l10n_rs_chart_template,,False
rs_543,Provisions for retained deposits and caution money,543,expense,l10n_rs.l10n_rs_chart_template,,False
rs_544,Provisions for restructuring costs,544,expense,l10n_rs.l10n_rs_chart_template,,False
rs_545,Provisions for contributions and other benefits of employees,545,expense,l10n_rs.l10n_rs_chart_template,,False
rs_549,Other long-term provisions,549,expense,l10n_rs.l10n_rs_chart_template,,False
rs_550,Costs of non-production services,550,expense,l10n_rs.l10n_rs_chart_template,,False
rs_551,Representation costs,551,expense,l10n_rs.l10n_rs_chart_template,,False
rs_552,Costs of insurance premiums,552,expense,l10n_rs.l10n_rs_chart_template,,False
rs_553,Costs of payment operations,553,expense,l10n_rs.l10n_rs_chart_template,,False
rs_554,Costs of membership fees,554,expense,l10n_rs.l10n_rs_chart_template,,False
rs_555,Tax costs,555,expense,l10n_rs.l10n_rs_chart_template,,False
rs_556,Contribution costs,556,expense,l10n_rs.l10n_rs_chart_template,,False
rs_559,Other non-production costs,559,expense,l10n_rs.l10n_rs_chart_template,,False
rs_560,Financial expenses incurred with parent companies and subsidiaries,560,expense,l10n_rs.l10n_rs_chart_template,,False
rs_561,Financial expenses incurred with other associated companies,561,expense,l10n_rs.l10n_rs_chart_template,,False
rs_562,Costs of interest,562,expense,l10n_rs.l10n_rs_chart_template,,False
rs_563,FX losses,563,expense,l10n_rs.l10n_rs_chart_template,,False
rs_564,Expenses on the basis of the effects of a currency clause,564,expense,l10n_rs.l10n_rs_chart_template,,False
rs_565,Expenses from participation in loss of associated legal entities and joint ventures,565,expense,l10n_rs.l10n_rs_chart_template,,False
rs_566,Expenses arising from the hedging effects that do not meet the requirements to qualify under other comprehensive income,566,expense,l10n_rs.l10n_rs_chart_template,,False
rs_569,Other financial expenses,569,expense,l10n_rs.l10n_rs_chart_template,,False
rs_570,"Losses on writing-offs and disposals of Intangible assets and Property, plant and equipment",570,expense,l10n_rs.l10n_rs_chart_template,,False
rs_571,Losses on writing-offs and disposals of natural assets,571,expense,l10n_rs.l10n_rs_chart_template,,False
rs_572,Losses on disposals of long-term investments,572,expense,l10n_rs.l10n_rs_chart_template,,False
rs_573,Losses on disposals of raw material,573,expense,l10n_rs.l10n_rs_chart_template,,False
rs_574,Shortages,574,expense,l10n_rs.l10n_rs_chart_template,,False
rs_575,Costs from negative hedging effects,575,expense,l10n_rs.l10n_rs_chart_template,,False
rs_576,Writing-offs of receivables,576,expense,l10n_rs.l10n_rs_chart_template,,False
rs_577,Writing-offs of inventories and merchandise,577,expense,l10n_rs.l10n_rs_chart_template,,False
rs_579,Other expenses,579,expense,l10n_rs.l10n_rs_chart_template,,False
rs_5791,Cash difference loss,5791,expense,l10n_rs.l10n_rs_chart_template,,False
rs_580,Impairment of natural assets,580,expense,l10n_rs.l10n_rs_chart_template,,False
rs_581,Impairment of intangible assets,581,expense,l10n_rs.l10n_rs_chart_template,,False
rs_582,"Impairment of property, plant and equipment",582,expense,l10n_rs.l10n_rs_chart_template,,False
rs_583,Impairment of long-term investments and other securities available for sale,583,expense,l10n_rs.l10n_rs_chart_template,,False
rs_584,Impairment of material and merchandise,584,expense,l10n_rs.l10n_rs_chart_template,,False
rs_585,Impairment of receivables and short-term financial investments,585,expense,l10n_rs.l10n_rs_chart_template,,False
rs_589,Impairment of other assets,589,expense,l10n_rs.l10n_rs_chart_template,,False
rs_590,Losses of suspended business,590,expense,l10n_rs.l10n_rs_chart_template,,False
rs_591,Expenses on the Effects of Changes in Accounting Policies,591,expense,l10n_rs.l10n_rs_chart_template,,False
rs_592,Expenses based on correction of errors from previous years that are not material,592,expense,l10n_rs.l10n_rs_chart_template,,False
rs_599,Transfer of expenses,599,expense,l10n_rs.l10n_rs_chart_template,,False
rs_600,Domestic sales of merchandise to parent companies and subsidiaries,600,income,l10n_rs.l10n_rs_chart_template,,False
rs_601,Foreign sales of merchandise to parent companies and subsidiaries,601,income,l10n_rs.l10n_rs_chart_template,,False
rs_602,Domestic sales of merchandise to other associated companies,602,income,l10n_rs.l10n_rs_chart_template,,False
rs_603,Foreign sales of merchandise to other associated companies,603,income,l10n_rs.l10n_rs_chart_template,,False
rs_604,Sales of merchandise to domestic customers,604,income,l10n_rs.l10n_rs_chart_template,,False
rs_605,Sales of merchandise to foreign customers,605,income,l10n_rs.l10n_rs_chart_template,,False
rs_610,Domestic sales of finished goods and services rendered to parent companies and subsidiaries,610,income,l10n_rs.l10n_rs_chart_template,,False
rs_611,Foreign sales of finished goods and services rendered to parent companies and subsidiaries,611,income,l10n_rs.l10n_rs_chart_template,,False
rs_612,Domestic sales of finished goods and services rendered to other associated entities,612,income,l10n_rs.l10n_rs_chart_template,,False
rs_613,Foreign sales of finished goods and services rendered to other associated entities,613,income,l10n_rs.l10n_rs_chart_template,,False
rs_614,Sales of finished goods and services rendered to domestic customers,614,income,l10n_rs.l10n_rs_chart_template,,False
rs_615,Sales of finished goods and services rendered to foreign customers,615,income,l10n_rs.l10n_rs_chart_template,,False
rs_620,Income from the own use of merchandise,620,income,l10n_rs.l10n_rs_chart_template,,False
rs_621,Income from the own use of products and services,621,income,l10n_rs.l10n_rs_chart_template,,False
rs_630,"Increase of finished goods, work in progress and services in progress",630,income,l10n_rs.l10n_rs_chart_template,,False
rs_631,"Decrease of finished goods, work in progress and services in progress",631,expense,l10n_rs.l10n_rs_chart_template,,False
rs_640,"Income from premiums, subventions, donations, compensations and tax returns",640,income,l10n_rs.l10n_rs_chart_template,,False
rs_641,Income from donations under specified conditions,641,income,l10n_rs.l10n_rs_chart_template,,False
rs_650,Rental fees income,650,income,l10n_rs.l10n_rs_chart_template,,False
rs_651,Membership fees,651,income,l10n_rs.l10n_rs_chart_template,,False
rs_652,Income from royalties and license fees,652,income,l10n_rs.l10n_rs_chart_template,,False
rs_659,Other operating income,659,income,l10n_rs.l10n_rs_chart_template,,False
rs_660,Financial income incurred with parent companies and subsidiaries,660,income,l10n_rs.l10n_rs_chart_template,,False
rs_661,Financial income incurred with other associated companies,661,income,l10n_rs.l10n_rs_chart_template,,False
rs_662,Income from interest,662,income,l10n_rs.l10n_rs_chart_template,,False
rs_663,FX Gains,663,income_other,l10n_rs.l10n_rs_chart_template,,False
rs_664,Income on the basis of the effects of a currency clause,664,income,l10n_rs.l10n_rs_chart_template,,False
rs_665,Income from participation in loss of associated legal entities and joint ventures,665,income,l10n_rs.l10n_rs_chart_template,,False
rs_669,Other financial income,669,income,l10n_rs.l10n_rs_chart_template,,False
rs_670,"Gains on disposals of Intangible assets and Property, plant and equipment",670,income,l10n_rs.l10n_rs_chart_template,,False
rs_671,Gains on disposals of natural assets,671,income,l10n_rs.l10n_rs_chart_template,,False
rs_672,Gains on disposals of long-term investments,672,income,l10n_rs.l10n_rs_chart_template,,False
rs_673,Gains on disposals of raw materials,673,income,l10n_rs.l10n_rs_chart_template,,False
rs_674,Surpluses,674,income,l10n_rs.l10n_rs_chart_template,,False
rs_675,Collected written-off receivables,675,income,l10n_rs.l10n_rs_chart_template,,False
rs_676,Income from positive hedging effects,676,income,l10n_rs.l10n_rs_chart_template,,False
rs_677,Income from reduction of liabilities,677,income,l10n_rs.l10n_rs_chart_template,,False
rs_678,Income from abolishing of long-term and short term provisions,678,income,l10n_rs.l10n_rs_chart_template,,False
rs_679,Other income,679,income_other,l10n_rs.l10n_rs_chart_template,,False
rs_6791,Cash difference gain,6791,income_other,l10n_rs.l10n_rs_chart_template,,False
rs_680,Income from valuation adjustments of natural assets,680,income_other,l10n_rs.l10n_rs_chart_template,,False
rs_681,Income from valuation adjustments of intangible assets,681,income_other,l10n_rs.l10n_rs_chart_template,,False
rs_682,"Income from valuation adjustments of property, plant and equipment",682,income_other,l10n_rs.l10n_rs_chart_template,,False
rs_683,Income from valuation adjustments long-term investments and securities available for sale,683,income_other,l10n_rs.l10n_rs_chart_template,,False
rs_684,Income from valuation adjustments of inventories,684,income_other,l10n_rs.l10n_rs_chart_template,,False
rs_685,Income from valuation adjustments of receivables and short- term investments,685,income_other,l10n_rs.l10n_rs_chart_template,,False
rs_689,Income from valuation adjustments of other assets,689,income_other,l10n_rs.l10n_rs_chart_template,,False
rs_690,Profit of suspended business,690,income_other,l10n_rs.l10n_rs_chart_template,,False
rs_691,Income based on the Effects of Changes in Accounting Policies,691,income_other,l10n_rs.l10n_rs_chart_template,,False
rs_692,Income based on correction of errors from previous years that are not material,692,income_other,l10n_rs.l10n_rs_chart_template,,False
rs_699,Transfer of income,699,income_other,l10n_rs.l10n_rs_chart_template,,False
rs_700,Opening account of general ledger,700,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_710,Account of expenses and income,710,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_711,Profit and loss of suspended business,711,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_712,Transfer of total result,712,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_720,Profit or loss account,720,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_721,Tax expense of the period,721,expense,l10n_rs.l10n_rs_chart_template,,False
rs_7220,Deferred tax expense of the period,7220,expense,l10n_rs.l10n_rs_chart_template,,False
rs_7221,Deferred tax income of the period,7221,income,l10n_rs.l10n_rs_chart_template,,False
rs_723,Personal earnings of the employer,723,expense,l10n_rs.l10n_rs_chart_template,,False
rs_724,Transfer of profit or loss,724,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_730,Closing of the balance sheet's accounts,730,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_880,Other assets taken into operating lease (rent),880,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_881,Acquired products and commodities for joint business,881,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_882,Goods taken into commission and consignment,882,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_883,Material and goods received for processing and finishing,883,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_884,"Assigned warranties, guarantees and other rights",884,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_885,Securities that are out of circulation,885,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_889,Assets with other entities,889,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_890,Liabilities for assets taken into operating lease (rent),890,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_891,Liabilities for acquired products and commodities for joint business,891,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_892,Liabilities for goods taken into commission and consignment,892,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_893,Liabilities for material and goods received for processing and finishing,893,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_894,"Liabilities for assigned warranties, guarantees and other rights",894,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_895,Liabilities for securities that are out of circulation,895,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_899,Liabilities for assets with other entities,899,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_900,Taking over inventories account,900,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_901,Taking over purchasing of material and merchandise account,901,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_902,Costs taking over account,902,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_903,Income taking over account,903,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_910,Raw material,910,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_911,Merchandise,911,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_912,Finished products and merchandise in producer's shops,912,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_950,Cost units,950,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_951,Cost units,951,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_952,Cost units,952,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_953,Cost units,953,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_954,Cost units,954,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_955,Cost units,955,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_956,Cost units,956,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_957,Cost units,957,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_958,Semi-finished products of own production,958,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_959,Deviation in cost units,959,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_960,Finished products,960,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_961,Finished products,961,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_962,Finished products,962,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_963,Finished products,963,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_964,Finished products,964,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_965,Finished products,965,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_966,Finished products,966,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_967,Finished products,967,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_968,Finished products,968,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_969,Exceptions in costs of finished products,969,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_980,Costs of products and services sold,980,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_981,Cost of merchandise sold,981,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_982,Costs of the period,982,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_983,"Write-offs, shortages and surpluses of work in progress and finished products",983,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_986,Income from products and services,986,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_987,Income from merchandise,987,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_989,Other operating income,989,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_990,Operating profit and loss,990,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_991,Loss and profit on raw materials' sale,991,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_992,Shortages of raw materials and merchandise,992,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_993,Write-offs of raw materials and merchandise,993,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_994,Surpluses of raw materials and merchandise,994,off_balance,l10n_rs.l10n_rs_chart_template,,False
rs_999,Closing computation of costs and effects,999,off_balance,l10n_rs.l10n_rs_chart_template,,False

```

## File: data\account.group.template.csv

```csv
id,code_prefix_start,name,chart_template_id/id
rs_group_0,0,"Subscribed capital, unpaid and non-current assets",l10n_rs.l10n_rs_chart_template
rs_group_00,00,"Subscribed capital, unpaid",l10n_rs.l10n_rs_chart_template
rs_group_01,01,Intangibles,l10n_rs.l10n_rs_chart_template
rs_group_02,02,"Property, plant, equipment and natural assets",l10n_rs.l10n_rs_chart_template
rs_group_03,03,Natural assets,l10n_rs.l10n_rs_chart_template
rs_group_04,04,Long term financial investments,l10n_rs.l10n_rs_chart_template
rs_group_043,043,"Long-term domestic loans to parent companies, to subsidiaries and to other associated companies",l10n_rs.l10n_rs_chart_template
rs_group_044,044,"Long-term foreign loans to parent companies, to subsidiaries and to other associated companies",l10n_rs.l10n_rs_chart_template
rs_group_045,045,Long term domestic and foreign loans,l10n_rs.l10n_rs_chart_template
rs_group_05,05,Long-term receivables,l10n_rs.l10n_rs_chart_template
rs_group_1,1,Inventories and assets hold for sale,l10n_rs.l10n_rs_chart_template
rs_group_10,10,Material,l10n_rs.l10n_rs_chart_template
rs_group_11,11,Work in progress and services,l10n_rs.l10n_rs_chart_template
rs_group_12,12,Finished products,l10n_rs.l10n_rs_chart_template
rs_group_13,13,"Merchandise (Goods, purchases for sale)",l10n_rs.l10n_rs_chart_template
rs_group_14,14,Assets held for trading,l10n_rs.l10n_rs_chart_template
rs_group_15,15,Advances paid,l10n_rs.l10n_rs_chart_template
rs_group_2,2,"Short term receivables, investments and cash",l10n_rs.l10n_rs_chart_template
rs_group_20,20,Receivables from sales,l10n_rs.l10n_rs_chart_template
rs_group_21,21,Receivables from specific business operations,l10n_rs.l10n_rs_chart_template
rs_group_22,22,Other receivables,l10n_rs.l10n_rs_chart_template
rs_group_23,23,Short-term financial investments,l10n_rs.l10n_rs_chart_template
rs_group_24,24,Cash and cash equivalents,l10n_rs.l10n_rs_chart_template
rs_group_27,27,Value added tax (VAT),l10n_rs.l10n_rs_chart_template
rs_group_28,28,Prepayments and accrued income,l10n_rs.l10n_rs_chart_template
rs_group_3,3,Equity,l10n_rs.l10n_rs_chart_template
rs_group_30,30,Basic and other capital,l10n_rs.l10n_rs_chart_template
rs_group_31,31,Subscribed capital unpaid,l10n_rs.l10n_rs_chart_template
rs_group_32,32,Reserves,l10n_rs.l10n_rs_chart_template
rs_group_33,33,Effect of restatement of capital and unrealized gains and losses,l10n_rs.l10n_rs_chart_template
rs_group_34,34,Retained profit,l10n_rs.l10n_rs_chart_template
rs_group_35,35,Loss,l10n_rs.l10n_rs_chart_template
rs_group_4,4,Long-term provisions and liabilities,l10n_rs.l10n_rs_chart_template
rs_group_40,40,Long-term provisions,l10n_rs.l10n_rs_chart_template
rs_group_41,41,Long-term liabilities,l10n_rs.l10n_rs_chart_template
rs_group_42,42,Short-term financial liabilities,l10n_rs.l10n_rs_chart_template
rs_group_43,43,Liabilities from business operations,l10n_rs.l10n_rs_chart_template
rs_group_44,44,Liabilities from specific business operations,l10n_rs.l10n_rs_chart_template
rs_group_45,45,Liabilities for salaries and fringe benefits,l10n_rs.l10n_rs_chart_template
rs_group_46,46,Other liabilities,l10n_rs.l10n_rs_chart_template
rs_group_47,47,Liabilities for Value Added Tax,l10n_rs.l10n_rs_chart_template
rs_group_48,48,"Liabilities for other taxes, contributions and other duties",l10n_rs.l10n_rs_chart_template
rs_group_49,49,Accruals and deferred income,l10n_rs.l10n_rs_chart_template
rs_group_5,5,Expenses,l10n_rs.l10n_rs_chart_template
rs_group_50,50,Costs of merchandise sold,l10n_rs.l10n_rs_chart_template
rs_group_51,51,Costs of material and energy,l10n_rs.l10n_rs_chart_template
rs_group_52,52,"Costs of salaries, fringe benefits and other personal expenses",l10n_rs.l10n_rs_chart_template
rs_group_53,53,Costs of production services,l10n_rs.l10n_rs_chart_template
rs_group_54,54,Costs of depreciation and provisions,l10n_rs.l10n_rs_chart_template
rs_group_55,55,Non-production costs,l10n_rs.l10n_rs_chart_template
rs_group_56,56,Financial expenses,l10n_rs.l10n_rs_chart_template
rs_group_57,57,Other Expenses,l10n_rs.l10n_rs_chart_template
rs_group_58,58,Impairment Costs,l10n_rs.l10n_rs_chart_template
rs_group_59,59,"Losses of suspended business, effects of changes in accounting policies, correction of errors of previous periods and transfer of expenditures",l10n_rs.l10n_rs_chart_template
rs_group_6,6,Income,l10n_rs.l10n_rs_chart_template
rs_group_60,60,Income from the sale of merchandise,l10n_rs.l10n_rs_chart_template
rs_group_61,61,Income from sales of products and services rendered,l10n_rs.l10n_rs_chart_template
rs_group_62,62,"Income from the own use of products, services and merchandise",l10n_rs.l10n_rs_chart_template
rs_group_63,63,Change in value of inventories of work in progress and finished products,l10n_rs.l10n_rs_chart_template
rs_group_64,64,"Income from premiums, subventions, donations, etc",l10n_rs.l10n_rs_chart_template
rs_group_65,65,Other operating income,l10n_rs.l10n_rs_chart_template
rs_group_66,66,Financial income,l10n_rs.l10n_rs_chart_template
rs_group_67,67,Other income,l10n_rs.l10n_rs_chart_template
rs_group_68,68,Income from assets valuation adjustments,l10n_rs.l10n_rs_chart_template
rs_group_69,69,"Profit of suspended business, effects of changes in accounting policies, correction of errors of previous periods and transfer of income",l10n_rs.l10n_rs_chart_template
rs_group_7,7,Opening and closing balances of balance sheet's and income statement's accounts,l10n_rs.l10n_rs_chart_template
rs_group_70,70,Opening account of general ledger,l10n_rs.l10n_rs_chart_template
rs_group_71,71,Closing of the Income statement's accounts,l10n_rs.l10n_rs_chart_template
rs_group_72,72,Profit and loss account,l10n_rs.l10n_rs_chart_template
rs_group_722,722,Deferred taxes,l10n_rs.l10n_rs_chart_template
rs_group_73,73,Closing of the Balance Sheet's accounts,l10n_rs.l10n_rs_chart_template
rs_group_74,74,Group free for use,l10n_rs.l10n_rs_chart_template
rs_group_8,8,Off-balance sheet evidence,l10n_rs.l10n_rs_chart_template
rs_group_88,88,Off-Balance sheet assets,l10n_rs.l10n_rs_chart_template
rs_group_89,89,Off-Balance sheet liabilities,l10n_rs.l10n_rs_chart_template
rs_group_9,9,Cost accounting,l10n_rs.l10n_rs_chart_template
rs_group_90,90,Liaison accounts with financial book-keeping,l10n_rs.l10n_rs_chart_template
rs_group_91,91,Raw material and merchandise,l10n_rs.l10n_rs_chart_template
rs_group_92,92,Cost accounts points for purchasing technical management and ancillary activities,l10n_rs.l10n_rs_chart_template
rs_group_93,93,Accounts for main production cost points,l10n_rs.l10n_rs_chart_template
rs_group_94,94,Cost account points for management sales and similar activities,l10n_rs.l10n_rs_chart_template
rs_group_95,95,Cost units,l10n_rs.l10n_rs_chart_template
rs_group_96,96,Finished products,l10n_rs.l10n_rs_chart_template
rs_group_97,97,Group free for use,l10n_rs.l10n_rs_chart_template
rs_group_98,98,Expenses and income,l10n_rs.l10n_rs_chart_template
rs_group_99,99,"Profit, loss and closing accounts",l10n_rs.l10n_rs_chart_template

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_rs.l10n_rs_chart_template')]"/>
    </function>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Chart template -->
    <record id="l10n_rs_chart_template" model="account.chart.template">
        <field name="name">Serbia - Chart of Accounts</field>
        <field name="code_digits">4</field>
        <field name="use_anglo_saxon" eval="True"/>
        <field name="bank_account_code_prefix">241</field>
        <field name="cash_account_code_prefix">243</field>
        <field name="transfer_account_code_prefix">250</field>

        <field name="currency_id" ref="base.RSD"/>
        <field name="country_id" ref="base.rs"/>
    </record>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_vat_20" model="account.tax.group">
            <field name="name">General VAT rate 20%</field>
            <field name="country_id" ref="base.rs"/>
        </record>

        <record id="tax_group_vat_10" model="account.tax.group">
            <field name="name">Specific VAT rate 10%</field>
            <field name="country_id" ref="base.rs"/>
        </record>

        <record id="tax_group_vat_0" model="account.tax.group">
            <field name="name">Exempt VAT rate 0%</field>
            <field name="country_id" ref="base.rs"/>
        </record>

        <record id="tax_group_vat_compensation" model="account.tax.group">
            <field name="name">Compensation</field>
            <field name="country_id" ref="base.rs"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report_vat" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.rs"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_vat_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
                <field name="sequence" eval="1"/>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_title_operations" model="account.report.line">
                <field name="name">Amount of fee without VAT</field>
                <field name="sequence" eval="1"/>
                <field name="children_ids">
                    <record id="tax_report_title_operations_turnover" model="account.report.line">
                        <field name="name">I. Trade of goods and services</field>
                        <field name="sequence" eval="2"/>
                        <field name="children_ids">
                            <record id="tax_report_line_001" model="account.report.line">
                                <field name="name">001 - Turnover of goods and services exempt from VAT with the right to deduct previous tax</field>
                                <field name="code">c001</field>
                                <field name="sequence" eval="3"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_001_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">001</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_002" model="account.report.line">
                                <field name="name">002 - Turnover of goods and services exempt from VAT without the right to deduct previous tax</field>
                                <field name="code">c002</field>
                                <field name="sequence" eval="4"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_002_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">002</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_003" model="account.report.line">
                                <field name="name">003 - Turnover of goods and services at the general rate</field>
                                <field name="code">c003</field>
                                <field name="sequence" eval="5"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_003_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">003</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_004" model="account.report.line">
                                <field name="name">004 - Turnover of goods and services at the special rate</field>
                                <field name="code">c004</field>
                                <field name="sequence" eval="6"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_004_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">004</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_005" model="account.report.line">
                                <field name="name">005 - Total (001 + 002 + 003 + 004)</field>
                                <field name="code">c005</field>
                                <field name="sequence" eval="7"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_005_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c001.balance + c002.balance + c003.balance + c004.balance</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_title_operations_previous_tax" model="account.report.line">
                        <field name="name">II. Previous Tax</field>
                        <field name="sequence" eval="8"/>
                        <field name="children_ids">
                            <record id="tax_report_line_006" model="account.report.line">
                                <field name="name">006 - Previous tax paid upon import</field>
                                <field name="code">c006</field>
                                <field name="sequence" eval="9"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_006_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">006</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_007" model="account.report.line">
                                <field name="name">007 - VAT Compensation paid to the farmer</field>
                                <field name="code">c007</field>
                                <field name="sequence" eval="10"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_007_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">007</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_008" model="account.report.line">
                                <field name="name">008 - Previous tax, except for the previous tax with item no. 6. and 7</field>
                                <field name="code">c008</field>
                                <field name="sequence" eval="11"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_008_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">008</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_009" model="account.report.line">
                                <field name="name">009 - Total (006 + 007 + 008)</field>
                                <field name="code">c009</field>
                                <field name="sequence" eval="12"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_009_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c006.balance + c007.balance + c008.balance</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_VAT" model="account.report.line">
                <field name="name">VAT</field>
                <field name="sequence" eval="13"/>
                <field name="children_ids">
                    <record id="tax_report_title_VAT_turnover" model="account.report.line">
                        <field name="name">I. Trade of goods and services</field>
                        <field name="sequence" eval="14"/>
                        <field name="children_ids">
                            <record id="tax_report_line_103" model="account.report.line">
                                <field name="name">103 - Turnover of goods and services at the general rate</field>
                                <field name="code">c103</field>
                                <field name="sequence" eval="15"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_103_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">103</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_104" model="account.report.line">
                                <field name="name">104 - Turnover of goods and services at the special rate</field>
                                <field name="code">c104</field>
                                <field name="sequence" eval="16"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_104_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">104</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_105" model="account.report.line">
                                <field name="name">105 - Total (103 + 104)</field>
                                <field name="code">c105</field>
                                <field name="sequence" eval="17"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_105_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c103.balance + c104.balance</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_title_VAT_previous_tax" model="account.report.line">
                        <field name="name">II. Previous Tax</field>
                        <field name="sequence" eval="18"/>
                        <field name="children_ids">
                            <record id="tax_report_line_106" model="account.report.line">
                                <field name="name">106 - Previous tax paid upon import</field>
                                <field name="code">c106</field>
                                <field name="sequence" eval="19"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_106_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">106</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_107" model="account.report.line">
                                <field name="name">107 - VAT Compensation paid to the farmer</field>
                                <field name="code">c107</field>
                                <field name="sequence" eval="20"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_107_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">107</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_108" model="account.report.line">
                                <field name="name">108 - Previous tax, except for the previous tax with item no. 6. and 7</field>
                                <field name="code">c108</field>
                                <field name="sequence" eval="21"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_108_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">108</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_109" model="account.report.line">
                                <field name="name">109 - Total (106 + 107 + 108)</field>
                                <field name="code">c109</field>
                                <field name="sequence" eval="22"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_109_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c106.balance + c107.balance + c108.balance</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_title_VAT_liability" model="account.report.line">
                        <field name="name">III. Tax liability</field>
                        <field name="sequence" eval="23"/>
                        <field name="children_ids">
                            <record id="tax_report_line_110" model="account.report.line">
                                <field name="name">110 - Amount of VAT in tax period (105 - 109)</field>
                                <field name="code">c110</field>
                                <field name="sequence" eval="24"/>
                                <field name="expression_ids">
                                    <record id="tax_report_line_110_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">c105.balance - c109.balance</field>
                                    </record>
                                </field>
                            </record>
                        </field>
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
    <record id="rs_sale_vat_20" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="description">VAT 20%</field>
        <field name="name">20%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="True"/>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_003_tag')],
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_470'),
                'plus_report_expression_ids': [ref('tax_report_line_103_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_470'),
                'minus_report_expression_ids': [ref('tax_report_line_103_tag')],
            }),
        ]"/>
    </record>
    <record id="rs_sale_vat_10" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="description">VAT 10%</field>
        <field name="name">10%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="True"/>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_004_tag')],
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_471'),
                'plus_report_expression_ids': [ref('tax_report_line_104_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_471'),
                'minus_report_expression_ids': [ref('tax_report_line_104_tag')],
            }),
        ]"/>
    </record>
    <record id="rs_sale_vat_0" model="account.tax.template">
        <field name="sequence">30</field>
        <field name="description">VAT 0%</field>
        <field name="name">0%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="True"/>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_471'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_471'),
            }),
        ]"/>
    </record>
    <record id="rs_sale_vat_0_deduct_previous_tax" model="account.tax.template">
        <field name="sequence">40</field>
        <field name="description">VAT 0% with the right to deduct previous tax</field>
        <field name="name">0% - prev. tax deductible</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="True"/>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_001_tag')],
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="rs_sale_vat_0_no_deduct_previous_tax" model="account.tax.template">
        <field name="sequence">50</field>
        <field name="description">VAT 0% without the right to deduct previous tax</field>
        <field name="name">0% - prev. tax not deductible</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="True"/>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_002_tag')],
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="rs_purchase_vat_20" model="account.tax.template">
        <field name="sequence">60</field>
        <field name="description">VAT 20%</field>
        <field name="name">20%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="True"/>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_008_tag')],
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_270'),
                'plus_report_expression_ids': [ref('tax_report_line_108_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_270'),
                'minus_report_expression_ids': [ref('tax_report_line_108_tag')],
            }),
        ]"/>
    </record>
    <record id="rs_purchase_vat_10" model="account.tax.template">
        <field name="sequence">70</field>
        <field name="description">VAT 10%</field>
        <field name="name">10%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="True"/>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_008_tag')],
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_271'),
                'plus_report_expression_ids': [ref('tax_report_line_108_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_271'),
                'minus_report_expression_ids': [ref('tax_report_line_108_tag')],
            }),
        ]"/>
    </record>
    <record id="rs_purchase_vat_0" model="account.tax.template">
        <field name="sequence">80</field>
        <field name="description">VAT 0%</field>
        <field name="name">0%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="True"/>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="rs_purchase_import_vat_0" model="account.tax.template">
        <field name="sequence">90</field>
        <field name="description">VAT 0% - import VAT exempt</field>
        <field name="name">0% - import</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="True"/>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_006_tag')],
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="rs_purchase_import_vat_20" model="account.tax.template">
        <field name="sequence">100</field>
        <field name="description">VAT 20% - import general rate</field>
        <field name="name">20% - import</field>
        <field name="price_include" eval="0"/>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="True"/>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_006_tag')],
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_274'),
                'plus_report_expression_ids': [ref('tax_report_line_106_tag')],
             }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_274'),
                'minus_report_expression_ids': [ref('tax_report_line_106_tag')],
            }),
        ]"/>
    </record>
    <record id="rs_purchase_import_vat_10" model="account.tax.template">
        <field name="sequence">110</field>
        <field name="description">VAT 10% - import specific rate</field>
        <field name="name">10% - import</field>
        <field name="price_include" eval="0"/>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="True"/>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_006_tag')],
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_275'),
                'plus_report_expression_ids': [ref('tax_report_line_106_tag')],
             }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_275'),
                'minus_report_expression_ids': [ref('tax_report_line_106_tag')],
            }),
        ]"/>
    </record>
    <record id="rs_purchase_farmer_deductible_vat_8" model="account.tax.template">
        <field name="sequence">120</field>
        <field name="description">VAT 8% - deductible farmers compensation</field>
        <field name="name">8% - farmers deductible</field>
        <field name="price_include" eval="0"/>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="True"/>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_compensation"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_007_tag')],
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_278'),
                'plus_report_expression_ids': [ref('tax_report_line_107_tag')],
             }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_278'),
                'minus_report_expression_ids': [ref('tax_report_line_107_tag')],
            }),
        ]"/>
    </record>
    <record id="rs_purchase_farmer_non_deductible_vat_8" model="account.tax.template">
        <field name="sequence">130</field>
        <field name="description">VAT 8% - non-deductible farmers compensation</field>
        <field name="name">8% - farmers non-deductible</field>
        <field name="price_include" eval="0"/>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="True"/>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="tax_group_id" ref="tax_group_vat_compensation"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_007_tag')],
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\fiscal_position_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="fiscal_position_template_domestic" model="account.fiscal.position.template">
        <field name="sequence">1</field>
        <field name="name">Domestic Customer</field>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.rs"/>
    </record>

    <record id="fiscal_position_template_foreign" model="account.fiscal.position.template">
        <field name="sequence">2</field>
        <field name="name">Foreign Customer</field>
        <field name="chart_template_id" ref="l10n_rs_chart_template"/>
        <field name="auto_apply" eval="True"/>
    </record>

    <record id="account_fiscal_position_purchase_10_0" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="l10n_rs.rs_purchase_vat_10"/>
        <field name="tax_dest_id" ref="l10n_rs.rs_purchase_vat_0"/>
        <field name="position_id" ref="fiscal_position_template_foreign"/>
    </record>

    <record id="account_fiscal_position_purchase_20_0" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="l10n_rs.rs_purchase_vat_20"/>
        <field name="tax_dest_id" ref="l10n_rs.rs_purchase_vat_0"/>
        <field name="position_id" ref="fiscal_position_template_foreign"/>
    </record>

    <record id="account_fiscal_position_sale_10_0" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="l10n_rs.rs_sale_vat_10"/>
        <field name="tax_dest_id" ref="l10n_rs.rs_sale_vat_0"/>
        <field name="position_id" ref="fiscal_position_template_foreign"/>
    </record>

    <record id="account_fiscal_position_sale_20_0" model="account.fiscal.position.tax.template">
        <field name="tax_src_id" ref="l10n_rs.rs_sale_vat_20"/>
        <field name="tax_dest_id" ref="l10n_rs.rs_sale_vat_0"/>
        <field name="position_id" ref="fiscal_position_template_foreign"/>
    </record>

    <!-- Fiscal Position Account Templates -->

    <record id="fiscal_position_account_template_foreign" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_foreign"  />
        <field name="account_src_id" ref="l10n_rs.rs_604" />
        <field name="account_dest_id" ref="l10n_rs.rs_605" />
    </record>
</odoo>

```

## File: data\l10n_rs_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Account Chart Template -->
    <record id="l10n_rs_chart_template" model="account.chart.template">
        <field name="name">Serbia - Chart of Accounts</field>
        <field name="property_account_expense_categ_id" ref="rs_501"/>
        <field name="property_account_income_categ_id" ref="rs_604"/>
        <field name="property_account_payable_id" ref="rs_435"/>
        <field name="property_account_receivable_id" ref="rs_204"/>
        <field name="property_tax_payable_account_id" ref="rs_470"/>
        <field name="property_tax_receivable_account_id" ref="rs_270"/>
        <field name="income_currency_exchange_account_id" ref="rs_663"/>
        <field name="expense_currency_exchange_account_id" ref="rs_563"/>
        <field name="default_cash_difference_expense_account_id" ref="rs_5791" />
        <field name="default_cash_difference_income_account_id" ref="rs_6791" />
    </record>

</odoo>

```

## File: data\menuitem_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem
            id="account_reports_rs_statements_menu"
            name="Serbia"
            parent="account.menu_finance_reports"
            sequence="0"
            groups="account.group_account_readonly"
    />
</odoo>

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_rs_turnover_date = fields.Date(string='Turnover Date')

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
    <mask id="b" x="4.8" y="7.22" width="50.4" height="32.76" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.5" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
    <rect x="6.2" y="10.57" width="48.45" height="31.57" rx="1" style="fill: #393939;opacity: 0.44;isolation: isolate"/>
    <g style="mask: url(#b)">
      <image width="1920" height="1152" transform="translate(4.8 7.22) scale(0.03 0.03)" xlink:href="data:image/png;base64,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"/>
    </g>
  </g>
</svg>

```

## File: views\account_move.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_move_form_inherit" model="ir.ui.view">
        <field name="name">account.move.form.inherit</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='invoice_payment_term_id']/parent::div" position="after">
                <field name="l10n_rs_turnover_date" attrs="{'invisible': [('country_code', '!=', 'RS')]}"/>
            </xpath>
        </field>
    </record>

    <template id="report_invoice_document_inherit" inherit_id="account.report_invoice_document">
        <xpath expr="//address" position="after">
            <div t-if="o.company_id.account_fiscal_country_id.code == 'RS' and o.partner_id.company_registry" class="mt16">
                <p>Company ID: <span t-field="o.partner_id.company_registry"/></p>
            </div>
        </xpath>
    </template>

</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <template id="report_invoice_document_inherit" inherit_id="account.report_invoice_document">
        <xpath expr="//address" position="after">
            <div t-if="o.company_id.account_fiscal_country_id.code == 'RS' and o.partner_id.company_registry" class="mt16">
                <p>Company ID: <span t-field="o.partner_id.company_registry"/></p>
            </div>
        </xpath>
    </template>

</odoo>

```

