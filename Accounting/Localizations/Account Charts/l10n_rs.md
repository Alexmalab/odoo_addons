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
        "views/res_partner.xml",
        "views/report_invoice.xml",
    ],
    "demo": ["demo/demo_company.xml"],
    "license": "LGPL-3",
}

```

## File: data\account.account.template.csv

```csv
id,name,code,user_type_id/id,chart_template_id/id,tag_ids/id,reconcile
rs_000,"Subscribed shares unpaid",000,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_001,"Subscribed stakes unpaid",001,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_010,"Investment in development",010,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_011,"Concessions, patents, licenses, goods and services brands",011,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_012,"Software and other rights",012,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_013,"Godwill",013,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_014,"Other intangible assets",014,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_015,"Intangible assets under preparation",015,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_016,"Advances for acquisition of intangible assets",016,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_019,"Provisions for acquisition of intangible assets",019,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_020,"Agricultural and other land",020,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_021,"Building land",021,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_022,"Buildings",022,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_023,"Plant and equipment",023,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_024,"Investment property",024,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_025,"Other property, plant and equipment",025,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_026,"Property, plant and equipment under construction",026,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_027,"Investment in other property, plant and equipment",027,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_028,"Advances for property, plant and equipment",028,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_029,"Provisions for property, plant and equipment",029,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_030,"Forests",030,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_031,"Plantations",031,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_032,"Livestock",032,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_037,"Natural assets under construction",037,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_038,"Advances for natural assets",038,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_039,"Provisions of natural assets",039,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_040,"Investments in capital of parent companies and subsidiaries",040,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_041,"Investments in capital of associated legal entities and joint ventures",041,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_042,"Investments in other legal entities and other securities available for sale",042,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_0430,"Long-term domestic loans to parent companies, to subsidiaries",0430,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_0431,"Long-term domestic loans to other associated companies",0431,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_0440,"Long-term foreign loans to parent companies, to subsidiaries",0440,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_0441,"Long-term foreign loans to other associated companies",0441,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_0450,"Long term domestic loans",0450,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_0451,"Long term foreign loans",0451,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_046,"Securities held to maturity",046,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_047,"Own stocks and shares purchased",047,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_048,"Other long term financial investments",048,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_049,"Provisions for long-term financial investments",049,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_050,"Receivables from parent and dependent legal entities",050,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_051,"Receivables from other related parties",051,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_052,"Receivables based on sales on commodity credit",052,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_053,"Receivables for sale under financial lease agreements",053,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_054,"Receivables based on warranty",054,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_055,"Contested and doubtful receivables",055,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_056,"Other long term receivables",056,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_059,"Provisions of long term receivables",059,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_100,"Material purchase costs calculation",100,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_101,"Raw materials",101,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_102,"Spare parts",102,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_103,"Tools and inventories",103,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_104,"Raw materials, spare parts, tools and inventories being processed, finished off and handled",104,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_109,"Provisions for material",109,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_110,"Work in progress",110,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_111,"Services in progress",111,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_120,"Finished products",120,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_130,"Merchandise cost accounting",130,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_131,"Merchandise in warehouse",131,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_132,"Wholesale merchandise",132,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_133,"Merchandise in storehouse, depot and other legal entities' shops",133,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_134,"Merchandise in retail sale",134,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_135,"Merchandise being processed, finished off and handled",135,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_136,"Merchandise in transit",136,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_137,"Merchandise in transport",137,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_139,"Provisions for merchandise",139,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_140,"Intangible assets for trading",140,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_141,"Land held for trading",141,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_142,"Buildings held for trading",142,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_143,"Investment property for trading",143,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_144,"Other property held for trading",144,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_145,"Installations and equipment for trading",145,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_146,"Natural assets for trading",146,account.data_account_type_non_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_147,"Assets of suspended business",147,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_149,"Provisions for assets held for trading",149,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_150,"Domestic advances paid for raw material, spare parts and inventories",150,account.data_account_type_prepayments,l10n_rs.l10n_rs_chart_template,,False
rs_151,"Foreign advances paid for raw material, spare parts and inventories",151,account.data_account_type_prepayments,l10n_rs.l10n_rs_chart_template,,False
rs_152,"Domestic advances paid for merchandise",152,account.data_account_type_prepayments,l10n_rs.l10n_rs_chart_template,,False
rs_153,"Foreign advances paid for merchandise",153,account.data_account_type_prepayments,l10n_rs.l10n_rs_chart_template,,False
rs_154,"Domestic advances paid for services",154,account.data_account_type_prepayments,l10n_rs.l10n_rs_chart_template,,False
rs_155,"Foreign advances paid for services",155,account.data_account_type_prepayments,l10n_rs.l10n_rs_chart_template,,False
rs_159,"Provisions for advances paid",159,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_200,"Domestic trade receivables - parent companies and subsidiaries",200,account.data_account_type_receivable,l10n_rs.l10n_rs_chart_template,,True
rs_201,"Foreign trade receivables - parent companies and subsidiaries",201,account.data_account_type_receivable,l10n_rs.l10n_rs_chart_template,,True
rs_202,"Domestic trade receivables - other associated entities",202,account.data_account_type_receivable,l10n_rs.l10n_rs_chart_template,,True
rs_203,"Foreign trade receivables - other associated entities",203,account.data_account_type_receivable,l10n_rs.l10n_rs_chart_template,,True
rs_204,"Trade receivables - domestic third party",204,account.data_account_type_receivable,l10n_rs.l10n_rs_chart_template,,True
rs_205,"Trade receivables - foreign third party",205,account.data_account_type_receivable,l10n_rs.l10n_rs_chart_template,,True
rs_206,"Other trade receivables",206,account.data_account_type_receivable,l10n_rs.l10n_rs_chart_template,,True
rs_209,"Provisions for trade receivables",209,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_210,"Receivables from exporters",210,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_211,"Receivables due from import on behalf of other entities",211,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_212,"Receivables from sales on commission and consignment",212,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_218,"Other receivables from specific business operations",218,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_219,"Provisions for receivables from specific business operations",219,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_220,"Receivables for interest and dividends",220,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_221,"Receivables from employees",221,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_222,"Receivables from state authorities and organizations",222,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_223,"Receivables for prepaid income tax",223,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_224,"Receivables for other taxes and contributions prepaid",224,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_225,"Receivables for compensation of salaries that are refunded",225,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_226,"Receivables based on damages",226,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_228,"Other short term receivables",228,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_229,"Provisions for other receivables",229,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_230,"Short-term loans and investments in parent companies and subsidiaries",230,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_231,"Short-term loans and investments in other associated companies",231,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_232,"Short term loans - domestic",232,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_233,"Short term loans - foreign",233,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_234,"Current portions of long term loan due within one year",234,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_235,"Current portions of securities held to maturity - a part that reaches one year",235,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_236,"Financial assets that are measured at fair value through the Income statement",236,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_237,"Bought up own shares for trading",237,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_238,"Other short-term investments",238,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_239,"Provisions for short - term investments",239,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_240,"Securities - cash equivalents",240,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_241,"Current (business) accounts",241,account.data_account_type_liquidity,l10n_rs.l10n_rs_chart_template,,False
rs_242,"Cash allocated for payments and open letters of credit",242,account.data_account_type_liquidity,l10n_rs.l10n_rs_chart_template,,False
rs_243,"Cash in hand",243,account.data_account_type_liquidity,l10n_rs.l10n_rs_chart_template,,False
rs_244,"Foreign exchange account",244,account.data_account_type_liquidity,l10n_rs.l10n_rs_chart_template,,False
rs_245,"Foreign currency letters of credit",245,account.data_account_type_liquidity,l10n_rs.l10n_rs_chart_template,,False
rs_246,"Foreign currency - cash in hand",246,account.data_account_type_liquidity,l10n_rs.l10n_rs_chart_template,,False
rs_248,"Other cash and cash equivalents",248,account.data_account_type_liquidity,l10n_rs.l10n_rs_chart_template,,False
rs_249,"Blocked bank accounts",249,account.data_account_type_liquidity,l10n_rs.l10n_rs_chart_template,,False
rs_270,"VAT in incoming invoices - general tax rate (except advances paid)",270,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_271,"VAT in incoming invoices - specific tax rate (except advances paid)",271,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_272,"VAT in advances paid - general tax rate",272,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_273,"VAT in advances paid - specific tax rate",273,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_274,"VAT paid for import of goods - general tax rate",274,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_275,"VAT paid for import of goods - specific tax rate",275,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_276,"VAT on services rendered by foreign suppliers",276,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_277,"VAT returned to foreign customers",277,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_278,"VAT reimbursement paid to farmers",278,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_279,"Receivables for prepaid VAT",279,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_280,"Prepaid costs",280,account.data_account_type_prepayments,l10n_rs.l10n_rs_chart_template,,False
rs_281,"Accrued income (un-invoiced income)",281,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_282,"Deferred costs based on obligations",282,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_288,"Deferred tax assets",288,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_289,"Other accruals",289,account.data_account_type_current_assets,l10n_rs.l10n_rs_chart_template,,False
rs_300,"Share capital",300,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_301,"Stakes in limited liability companies",301,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_302,"Participating interests",302,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_303,"State owned capital",303,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_304,"Socially owned capital",304,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_305,"Stakes in co-operatives",305,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_306,"Share issuing premiums",306,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_309,"Other capital",309,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_310,"Subscribed share capital unpaid",310,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_311,"Subscribed stakes unpaid",311,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_321,"Legal reserves",321,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_322,"Statutory and other reserves",322,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_330,"Effect of restatement of capital based on the revaluation of intangible assets, property, plant and equipment",330,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_331,"Actuarial gains or losses on planned benefit plans",331,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_332,"Gains or losses on investments in equity instruments",332,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_333,"Gains or losses on the basis of the share in other comprehensive income or loss of associates",333,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_334,"Gains or losses arising from the calculations of financial statements of foreign operations",334,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_335,"Gains or losses on instruments for the protection of net investments in foreign operations",335,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_336,"Gains or losses arising from hedging instruments of cash flow",336,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_337,"Gains or losses on securities available for trading",337,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_340,"Retained profit from previous years",340,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_341,"Retained profit from current year",341,account.data_unaffected_earnings,l10n_rs.l10n_rs_chart_template,,False
rs_350,"Previous year's losses",350,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_351,"Current year loss",351,account.data_account_type_equity,l10n_rs.l10n_rs_chart_template,,False
rs_400,"Provisions for costs incurred during the warranty period",400,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_401,"Provisions for the recovery of natural resources",401,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_402,"Provisions for retained deposits and caution money",402,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_403,"Provisions for restructuring costs",403,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_404,"Provisions for contribution and other employee benefits",404,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_405,"Provision for court costs",405,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_409,"Other long-term provisions",409,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_410,"Liabilities which can be converted into capital",410,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_411,"Liabilities to parent companies and subsidiaries",411,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_412,"Liabilities to other associated companies",412,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_413,"Liabilities for long-term securities",413,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_414,"Long-term loans - domestic",414,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_415,"Long-term loans - foreign",415,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_416,"Liabilities based on financial leasing",416,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_419,"Other long-term liabilities",419,account.data_account_type_non_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_420,"Short-term loans from parent companies and subsidiaries",420,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_421,"Short-term loans from other associated companies",421,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_422,"Short term loans - domestic",422,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_423,"Short term loans - foreign",423,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_424,"Current portion of long-term loans",424,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_425,"Current portion of other long-term liabilities",425,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_426,"Liabilities for short-term securities",426,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_427,"Liabilities based on fixed assets and assets of suspended business for trading",427,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_429,"Other short-term financial liabilities",429,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_430,"Received advances, deposits and caution money",430,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_431,"Domestic trade payables - parent companies and subsidiaries",431,account.data_account_type_payable,l10n_rs.l10n_rs_chart_template,,True
rs_432,"Foreign trade payables - parent companies and subsidiaries",432,account.data_account_type_payable,l10n_rs.l10n_rs_chart_template,,True
rs_433,"Domestic trade payables - other associated companies",433,account.data_account_type_payable,l10n_rs.l10n_rs_chart_template,,True
rs_434,"Foreign trade payables - other associated companies",434,account.data_account_type_payable,l10n_rs.l10n_rs_chart_template,,True
rs_435,"Trade payables - domestic",435,account.data_account_type_payable,l10n_rs.l10n_rs_chart_template,,True
rs_436,"Trade payables - foreign",436,account.data_account_type_payable,l10n_rs.l10n_rs_chart_template,,True
rs_439,"Other liabilities from business operations",439,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_440,"Liabilities to importers",440,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_441,"Liabilities due to exporters made on behalf of third parties",441,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_442,"Liabilities for sale on commission and consignment",442,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_449,"Other liabilities from specific business operations",449,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_450,"Liabilities for net salaries and fringe benefits, except refundable fringe benefits",450,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_451,"Liabilities for taxes on salaries and fringe benefits charged to employees",451,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_452,"Liabilities for contributions on salaries and fringe benefits charged to employees",452,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_453,"Liabilities for taxes and contributions on salaries and fringe benefits charged to employer",453,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_454,"Liabilities for refundable net fringe benefits",454,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_455,"Liabilities for taxes and contributions on refundable fringe benefits charged to employees",455,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_456,"Liabilities for taxes and contributions on refundable fringe benefits charged to employer",456,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_460,"Liabilities for interests and finance costs",460,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_461,"Liabilities for dividends",461,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_462,"Liabilities for share in the profit",462,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_463,"Liabilities to employees",463,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_464,"Liabilities to members of Management Board and Supervisory Board",464,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_465,"Liabilities to individuals for contracted fees",465,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_466,"Liabilities for net income of sole traders who obtain advances during the year",466,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_467,"Liabilities for short-term provisions",467,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_469,"Other liabilities",469,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_470,"Liabilities for VAT in outgoing invoices - general tax rate (except advances received)",470,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_471,"Liabilities for VAT in outgoing invoices - specific tax rate (except advances received)",471,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_472,"Liabilities for VAT in advances received - general tax rate",472,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_473,"Liabilities for VAT in advances received - specific tax rate",473,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_474,"Liabilities for VAT on internal consumption - general tax rate",474,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_475,"Liabilities for VAT on internal consumption - specific tax rate",475,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_476,"Liabilities for VAT on sales for cash",476,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_479,"Liabilities for VAT on difference between calculated VAT and previous taxes",479,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_480,"Liabilities for excise duties",480,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_481,"Liabilities for income tax",481,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_482,"Liabilities for taxes, customs, and other duties charged to costs",482,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_483,"Liabilities for contributions charged to costs",483,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_489,"Other liabilities for taxes, contributions and other duties",489,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_490,"Accrued expenses",490,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_491,"Revenue charged in advance",491,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_494,"Accrued direct purchase costs",494,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_495,"Donations received",495,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_496,"Accrued income from receivables",496,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_498,"Deferred tax liabilities",498,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_499,"Other accruals and deferred income",499,account.data_account_type_current_liabilities,l10n_rs.l10n_rs_chart_template,,False
rs_500,"Purchases of merchandises (goods for resale)",500,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_501,"Cost of merchandise sold",501,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_502,"Cost of sold assets held for trading",502,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_503,"Cost of other assets held for trading",503,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_510,"Costs material",510,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_511,"Costs of raw material",511,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_512,"Costs of other material (overhead)",512,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_513,"Costs of fuel and energy",513,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_514,"Costs of spare parts",514,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_515,"Costs of One-time write-off for tools and inventory",515,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_520,"Costs of salaries and fringe benefits (gross)",520,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_521,"Costs of taxes and contributions on salaries and fringe benefits charged to employer",521,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_522,"Costs of remunerations according to temporary service contracts",522,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_523,"Costs of remunerations according to author's contracts",523,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_524,"Costs of remunerations according to temporary and provisional contracts",524,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_525,"Costs of remunerations to individuals according to other contracts",525,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_526,"Costs of remuneration to manager, members of Management Board and Supervisory Board",526,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_529,"Other personal expenses remunerations",529,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_530,"Costs of services used in production process of own costs capitalized",530,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_531,"Transport services costs",531,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_532,"Maintenance costs",532,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_533,"Rental costs",533,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_534,"Fairs exhibit costs",534,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_535,"Advertising costs",535,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_536,"Costs of researching activities",536,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_537,"Costs of development that are not capitalized",537,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_539,"Costs of other services",539,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_540,"Depreciation costs",540,account.data_account_type_depreciation,l10n_rs.l10n_rs_chart_template,,False
rs_541,"Costs of provisions during the warranty period",541,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_542,"Provisions for recovery of natural resources",542,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_543,"Provisions for retained deposits and caution money",543,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_544,"Provisions for restructuring costs",544,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_545,"Provisions for contributions and other benefits of employees",545,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_549,"Other long-term provisions",549,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_550,"Costs of non-production services",550,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_551,"Representation costs",551,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_552,"Costs of insurance premiums",552,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_553,"Costs of payment operations",553,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_554,"Costs of membership fees",554,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_555,"Tax costs",555,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_556,"Contribution costs",556,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_559,"Other non-production costs",559,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_560,"Financial expenses incurred with parent companies and subsidiaries",560,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_561,"Financial expenses incurred with other associated companies",561,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_562,"Costs of interest",562,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_563,"FX losses",563,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_564,"Expenses on the basis of the effects of a currency clause",564,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_565,"Expenses from participation in loss of associated legal entities and joint ventures",565,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_566,"Expenses arising from the hedging effects that do not meet the requirements to qualify under other comprehensive income",566,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_569,"Other financial expenses",569,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_570,"Losses on writing-offs and disposals of Intangible assets and Property, plant and equipment",570,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_571,"Losses on writing-offs and disposals of natural assets",571,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_572,"Losses on disposals of long-term investments",572,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_573,"Losses on disposals of raw material",573,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_574,"Shortages",574,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_575,"Costs from negative hedging effects",575,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_576,"Writing-offs of receivables",576,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_577,"Writing-offs of inventories and merchandise",577,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_579,"Other expenses",579,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_5791,"Cash difference loss",5791,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,FALSE
rs_580,"Impairment of natural assets",580,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_581,"Impairment of intangible assets",581,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_582,"Impairment of property, plant and equipment",582,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_583,"Impairment of long-term investments and other securities available for sale",583,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_584,"Impairment of material and merchandise",584,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_585,"Impairment of receivables and short-term financial investments",585,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_589,"Impairment of other assets",589,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_590,"Losses of suspended business",590,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_591,"Expenses on the Effects of Changes in Accounting Policies",591,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_592,"Expenses based on correction of errors from previous years that are not material",592,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_599,"Transfer of expenses",599,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_600,"Domestic sales of merchandise to parent companies and subsidiaries",600,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_601,"Foreign sales of merchandise to parent companies and subsidiaries",601,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_602,"Domestic sales of merchandise to other associated companies",602,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_603,"Foreign sales of merchandise to other associated companies",603,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_604,"Sales of merchandise to domestic customers",604,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_605,"Sales of merchandise to foreign customers",605,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_610,"Domestic sales of finished goods and services rendered to parent companies and subsidiaries",610,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_611,"Foreign sales of finished goods and services rendered to parent companies and subsidiaries",611,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_612,"Domestic sales of finished goods and services rendered to other associated entities",612,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_613,"Foreign sales of finished goods and services rendered to other associated entities",613,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_614,"Sales of finished goods and services rendered to domestic customers",614,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_615,"Sales of finished goods and services rendered to foreign customers",615,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_620,"Income from the own use of merchandise",620,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_621,"Income from the own use of products and services",621,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_630,"Increase of finished goods, work in progress and services in progress",630,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_631,"Decrease of finished goods, work in progress and services in progress",631,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_640,"Income from premiums, subventions, donations, compensations and tax returns",640,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_641,"Income from donations under specified conditions",641,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_650,"Rental fees income",650,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_651,"Membership fees",651,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_652,"Income from royalties and license fees",652,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_659,"Other operating income",659,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_660,"Financial income incurred with parent companies and subsidiaries",660,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_661,"Financial income incurred with other associated companies",661,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_662,"Income from interest",662,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_663,"FX Gains",663,account.data_account_type_other_income,l10n_rs.l10n_rs_chart_template,,False
rs_664,"Income on the basis of the effects of a currency clause",664,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_665,"Income from participation in loss of associated legal entities and joint ventures",665,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_669,"Other financial income",669,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_670,"Gains on disposals of Intangible assets and Property, plant and equipment",670,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_671,"Gains on disposals of natural assets",671,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_672,"Gains on disposals of long-term investments",672,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_673,"Gains on disposals of raw materials",673,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_674,"Surpluses",674,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_675,"Collected written-off receivables",675,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_676,"Income from positive hedging effects",676,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_677,"Income from reduction of liabilities",677,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_678,"Income from abolishing of long-term and short term provisions",678,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_679,"Other income",679,account.data_account_type_other_income,l10n_rs.l10n_rs_chart_template,,False
rs_6791,"Cash difference gain",6791,account.data_account_type_other_income,l10n_rs.l10n_rs_chart_template,,FALSE
rs_680,"Income from valuation adjustments of natural assets",680,account.data_account_type_other_income,l10n_rs.l10n_rs_chart_template,,False
rs_681,"Income from valuation adjustments of intangible assets",681,account.data_account_type_other_income,l10n_rs.l10n_rs_chart_template,,False
rs_682,"Income from valuation adjustments of property, plant and equipment",682,account.data_account_type_other_income,l10n_rs.l10n_rs_chart_template,,False
rs_683,"Income from valuation adjustments long-term investments and securities available for sale",683,account.data_account_type_other_income,l10n_rs.l10n_rs_chart_template,,False
rs_684,"Income from valuation adjustments of inventories",684,account.data_account_type_other_income,l10n_rs.l10n_rs_chart_template,,False
rs_685,"Income from valuation adjustments of receivables and short- term investments",685,account.data_account_type_other_income,l10n_rs.l10n_rs_chart_template,,False
rs_689,"Income from valuation adjustments of other assets",689,account.data_account_type_other_income,l10n_rs.l10n_rs_chart_template,,False
rs_690,"Profit of suspended business",690,account.data_account_type_other_income,l10n_rs.l10n_rs_chart_template,,False
rs_691,"Income based on the Effects of Changes in Accounting Policies",691,account.data_account_type_other_income,l10n_rs.l10n_rs_chart_template,,False
rs_692,"Income based on correction of errors from previous years that are not material",692,account.data_account_type_other_income,l10n_rs.l10n_rs_chart_template,,False
rs_699,"Transfer of income",699,account.data_account_type_other_income,l10n_rs.l10n_rs_chart_template,,False
rs_700,"Opening account of general ledger",700,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_710,"Account of expenses and income",710,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_711,"Profit and loss of suspended business",711,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_712,"Transfer of total result",712,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_720,"Profit or loss account",720,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_721,"Tax expense of the period",721,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_7220,"Deferred tax expense of the period",7220,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_7221,"Deferred tax income of the period",7221,account.data_account_type_revenue,l10n_rs.l10n_rs_chart_template,,False
rs_723,"Personal earnings of the employer",723,account.data_account_type_expenses,l10n_rs.l10n_rs_chart_template,,False
rs_724,"Transfer of profit or loss",724,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_730,"Closing of the balance sheet's accounts",730,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_880,"Other assets taken into operating lease (rent)",880,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_881,"Acquired products and commodities for joint business",881,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_882,"Goods taken into commission and consignment",882,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_883,"Material and goods received for processing and finishing",883,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_884,"Assigned warranties, guarantees and other rights",884,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_885,"Securities that are out of circulation",885,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_889,"Assets with other entities",889,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_890,"Liabilities for assets taken into operating lease (rent)",890,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_891,"Liabilities for acquired products and commodities for joint business",891,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_892,"Liabilities for goods taken into commission and consignment",892,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_893,"Liabilities for material and goods received for processing and finishing",893,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_894,"Liabilities for assigned warranties, guarantees and other rights",894,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_895,"Liabilities for securities that are out of circulation",895,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_899,"Liabilities for assets with other entities",899,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_900,"Taking over inventories account",900,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_901,"Taking over purchasing of material and merchandise account",901,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_902,"Costs taking over account",902,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_903,"Income taking over account",903,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_910,"Raw material",910,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_911,"Merchandise",911,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_912,"Finished products and merchandise in producer's shops",912,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_950,"Cost units",950,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_951,"Cost units",951,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_952,"Cost units",952,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_953,"Cost units",953,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_954,"Cost units",954,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_955,"Cost units",955,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_956,"Cost units",956,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_957,"Cost units",957,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_958,"Semi-finished products of own production",958,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_959,"Deviation in cost units",959,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_960,"Finished products",960,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_961,"Finished products",961,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_962,"Finished products",962,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_963,"Finished products",963,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_964,"Finished products",964,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_965,"Finished products",965,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_966,"Finished products",966,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_967,"Finished products",967,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_968,"Finished products",968,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_969,"Exceptions in costs of finished products",969,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_980,"Costs of products and services sold",980,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_981,"Cost of merchandise sold",981,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_982,"Costs of the period",982,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_983,"Write-offs, shortages and surpluses of work in progress and finished products",983,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_986,"Income from products and services",986,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_987,"Income from merchandise",987,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_989,"Other operating income",989,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_990,"Operating profit and loss",990,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_991,"Loss and profit on raw materials' sale",991,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_992,"Shortages of raw materials and merchandise",992,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_993,"Write-offs of raw materials and merchandise",993,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_994,"Surpluses of raw materials and merchandise",994,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False
rs_999,"Closing computation of costs and effects",999,account.data_account_off_sheet,l10n_rs.l10n_rs_chart_template,,False

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
    <record id="tax_report_vat" model="account.tax.report">
        <field name="name">VAT Report</field>
        <field name="country_id" ref="base.rs"/>
    </record>

    <record id="tax_report_title_operations" model="account.tax.report.line">
        <field name="name">Amount of fee without VAT</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="sequence">1</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_title_operations_turnover" model="account.tax.report.line">
        <field name="name">I. Trade of goods and services</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations"/>
        <field name="sequence">1</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_line_001" model="account.tax.report.line">
        <field name="name">001 - Turnover of goods and services exempt from VAT with the right to deduct previous tax</field>
        <field name="code">c001</field>
        <field name="tag_name">001</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_turnover"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_line_002" model="account.tax.report.line">
        <field name="name">002 - Turnover of goods and services exempt from VAT without the right to deduct previous tax</field>
        <field name="code">c002</field>
        <field name="tag_name">002</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_turnover"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_line_003" model="account.tax.report.line">
        <field name="name">003 - Turnover of goods and services at the general rate</field>
        <field name="code">c003</field>
        <field name="tag_name">003</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_turnover"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_line_004" model="account.tax.report.line">
        <field name="name">004 - Turnover of goods and services at the special rate</field>
        <field name="code">c004</field>
        <field name="tag_name">004</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_turnover"/>
        <field name="sequence">4</field>
    </record>

    <record id="tax_report_line_005" model="account.tax.report.line">
        <field name="name">005 - Total (001 + 002 + 003 + 004)</field>
        <field name="formula">c001 + c002 + c003 + c004</field>
        <field name="code">c005</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_turnover"/>
        <field name="sequence">5</field>
    </record>

    <record id="tax_report_title_operations_previous_tax" model="account.tax.report.line">
        <field name="name">II. Previous Tax</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations"/>
        <field name="sequence">2</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_line_006" model="account.tax.report.line">
        <field name="name">006 - Previous tax paid upon import</field>
        <field name="code">c006</field>
        <field name="tag_name">006</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_previous_tax"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_line_007" model="account.tax.report.line">
        <field name="name">007 - VAT Compensation paid to the farmer</field>
        <field name="code">c007</field>
        <field name="tag_name">007</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_previous_tax"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_line_008" model="account.tax.report.line">
        <field name="name">008 - Previous tax, except for the previous tax with item no. 6. and 7</field>
        <field name="code">c008</field>
        <field name="tag_name">008</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_previous_tax"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_line_009" model="account.tax.report.line">
        <field name="name">009 - Total (006 + 007 + 008)</field>
        <field name="formula">c006 + c007 + c008</field>
        <field name="code">c009</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_operations_previous_tax"/>
        <field name="sequence">4</field>
    </record>

    <record id="tax_report_title_VAT" model="account.tax.report.line">
        <field name="name">VAT</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="sequence">2</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_title_VAT_turnover" model="account.tax.report.line">
        <field name="name">I. Trade of goods and services</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_VAT"/>
        <field name="sequence">1</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_line_103" model="account.tax.report.line">
        <field name="name">103 - Turnover of goods and services at the general rate</field>
        <field name="code">c103</field>
        <field name="tag_name">103</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_VAT_turnover"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_line_104" model="account.tax.report.line">
        <field name="name">104 - Turnover of goods and services at the special rate</field>
        <field name="code">c104</field>
        <field name="tag_name">104</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_VAT_turnover"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_line_105" model="account.tax.report.line">
        <field name="name">105 - Total (103 + 104)</field>
        <field name="formula">c103 + c104</field>
        <field name="code">c105</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_VAT_turnover"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_title_VAT_previous_tax" model="account.tax.report.line">
        <field name="name">II. Previous Tax</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_VAT"/>
        <field name="sequence">2</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_line_106" model="account.tax.report.line">
        <field name="name">106 - Previous tax paid upon import</field>
        <field name="code">c106</field>
        <field name="tag_name">106</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_VAT_previous_tax"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_line_107" model="account.tax.report.line">
        <field name="name">107 - VAT Compensation paid to the farmer</field>
        <field name="code">c107</field>
        <field name="tag_name">107</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_VAT_previous_tax"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_line_108" model="account.tax.report.line">
        <field name="name">108 - Previous tax, except for the previous tax with item no. 6. and 7</field>
        <field name="code">c108</field>
        <field name="tag_name">108</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_VAT_previous_tax"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_line_109" model="account.tax.report.line">
        <field name="name">109 - Total (106 + 107 + 108)</field>
        <field name="formula">c106 + c107 + c108</field>
        <field name="code">c109</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_VAT_previous_tax"/>
        <field name="sequence">4</field>
    </record>

    <record id="tax_report_title_VAT_liability" model="account.tax.report.line">
        <field name="name">III. Tax liability</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_VAT"/>
        <field name="sequence">3</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_line_110" model="account.tax.report.line">
        <field name="name">110 - Amount of VAT in tax period (105 - 109)</field>
        <field name="formula">c105 - c109</field>
        <field name="code">c110</field>
        <field name="report_id" ref="tax_report_vat"/>
        <field name="parent_id" ref="tax_report_title_VAT_liability"/>
        <field name="sequence">1</field>
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
                'plus_report_line_ids': [ref('tax_report_line_003')],
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_470'),
                'plus_report_line_ids': [ref('tax_report_line_103')],
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
                'minus_report_line_ids': [ref('tax_report_line_103')],
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
                'plus_report_line_ids': [ref('tax_report_line_004')],
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_471'),
                'plus_report_line_ids': [ref('tax_report_line_104')],
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
                'minus_report_line_ids': [ref('tax_report_line_104')],
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
                'plus_report_line_ids': [ref('tax_report_line_001')],
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
                'plus_report_line_ids': [ref('tax_report_line_002')],
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
                'plus_report_line_ids': [ref('tax_report_line_008')],
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_270'),
                'plus_report_line_ids': [ref('tax_report_line_108')],
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
                'minus_report_line_ids': [ref('tax_report_line_108')],
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
                'plus_report_line_ids': [ref('tax_report_line_008')],
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_271'),
                'plus_report_line_ids': [ref('tax_report_line_108')],
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
                'minus_report_line_ids': [ref('tax_report_line_108')],
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
                'plus_report_line_ids': [ref('tax_report_line_006')],
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
                'plus_report_line_ids': [ref('tax_report_line_006')],
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_274'),
                'plus_report_line_ids': [ref('tax_report_line_106')],

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
                'minus_report_line_ids': [ref('tax_report_line_106')],
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
                'plus_report_line_ids': [ref('tax_report_line_006')],
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_275'),
                'plus_report_line_ids': [ref('tax_report_line_106')],

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
                'minus_report_line_ids': [ref('tax_report_line_106')],
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
                'plus_report_line_ids': [ref('tax_report_line_007')],
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('rs_278'),
                'plus_report_line_ids': [ref('tax_report_line_107')],

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
                'minus_report_line_ids': [ref('tax_report_line_107')],
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
                'plus_report_line_ids': [ref('tax_report_line_007')],
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

    l10n_rs_company_registry = fields.Char(string='Company ID', related='partner_id.l10n_rs_company_registry')
    l10n_rs_turnover_date = fields.Date(string='Turnover Date')

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_rs_company_registry = fields.Char(string='Company ID', help='The registry number of the company.')

    _sql_constraints = [
        (
            'company_registry_country_uniq',
            'unique (l10n_rs_company_registry, country_id)',
            'The company registry of the partner must be unique across all partners of a same country !'
         )
    ]

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_move
from . import res_partner

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
            <div t-if="o.company_id.account_fiscal_country_id.code == 'RS' and o.partner_id.l10n_rs_company_registry" class="mt16">
                <p>Company ID: <span t-field="o.partner_id.l10n_rs_company_registry"/></p>
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
            <div t-if="o.company_id.account_fiscal_country_id.code == 'RS' and o.partner_id.l10n_rs_company_registry" class="mt16">
                <p>Company ID: <span t-field="o.partner_id.l10n_rs_company_registry"/></p>
            </div>
        </xpath>
    </template>

</odoo>

```

## File: views\res_partner.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="view_partner_form_inherit" model="ir.ui.view">
        <field name="name">res.partner.form.inherit</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='sales_purchases']/group/group[@name='misc']" position="inside">
                <field name="l10n_rs_company_registry" attrs="{'invisible': [('company_type', '!=', 'company')]}"/>
            </xpath>
        </field>
    </record>

</odoo>

```

