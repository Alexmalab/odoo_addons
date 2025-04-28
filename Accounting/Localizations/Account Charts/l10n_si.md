# Odoo Module: l10n_si

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, SUPERUSER_ID

def load_translations(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref('l10n_si.gd_chart').process_coa_translations()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    "name": "Slovenian - Accounting",
    "version": "1.1",
    "author": "Odoo S.A.",
    "category": "Accounting/Localizations/Account Charts",
    "description": """
        Chart of accounts and taxes for Slovenia.
    """,
    "depends": [
        "account",
        "base_vat",
        "l10n_multilang"
    ],
    "data": [
        "data/l10n_si_chart_data.xml",
        "data/account.account.template.csv",
        "data/account.group.template.csv",
        "data/account_tax_group.xml",
        "data/account_tax_report_data.xml",
        "data/account_tax_data.xml",
        "data/account_fiscal_position_template.xml",
        "data/account_fiscal_position_account_template.xml",
        "data/account_fiscal_position_tax_template.xml",
        "data/account_chart_template_configure_data.xml",
        "data/account_chart_template_data.xml",
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
"id","code","account_type","name","reconcile","chart_template_id/id"
"gd_acc_000000","000000","asset_non_current","Good name","False","gd_chart"
"gd_acc_001000","001000","asset_non_current","Capitalization costs of investments in foreign tangible fixed assets","False","gd_chart"
"gd_acc_002000","002000","asset_non_current","Deferred development costs","False","gd_chart"
"gd_acc_003000","003000","asset_non_current","Property and other rights","False","gd_chart"
"gd_acc_005000","005000","asset_non_current","Other intangible assets (including allowances)","False","gd_chart"
"gd_acc_007000","007000","asset_non_current","Long-term accrued costs and deferred revenue","False","gd_chart"
"gd_acc_008000","008000","asset_non_current","Value adjustment of intangible assets due to amortization","False","gd_chart"
"gd_acc_009000","009000","asset_non_current","Impairment of intangible assets","False","gd_chart"
"gd_acc_010000","010000","asset_non_current","Investment property valued according to the cost model","False","gd_chart"
"gd_acc_011000","011000","asset_non_current","Investment property valued at fair value model","False","gd_chart"
"gd_acc_015000","015000","asset_non_current","Value adjustment of investment property due to depreciation","False","gd_chart"
"gd_acc_017000","017000","asset_non_current","Investment property under construction","False","gd_chart"
"gd_acc_019000","019000","asset_non_current","Impairment of investment property","False","gd_chart"
"gd_acc_020000","020000","asset_fixed","Land valued according to the cost model","False","gd_chart"
"gd_acc_021000","021000","asset_fixed","Buildings valued according to the cost model","False","gd_chart"
"gd_acc_022000","022000","asset_fixed","Land valued according to the revaluation model","False","gd_chart"
"gd_acc_023000","023000","asset_fixed","Buildings valued according to the revaluation model","False","gd_chart"
"gd_acc_026000","026000","asset_fixed","Investments in foreign-owned real estate","False","gd_chart"
"gd_acc_027000","027000","asset_fixed","Real estate under construction","False","gd_chart"
"gd_acc_031000","031000","asset_non_current","Impairment of land value","False","gd_chart"
"gd_acc_032000","032000","asset_non_current","Value adjustment of land (quarries, landfills) due to depreciation","False","gd_chart"
"gd_acc_035000","035000","asset_non_current","Value adjustment of buildings due to depreciation","False","gd_chart"
"gd_acc_036000","036000","asset_non_current","Value adjustment of foreign-owned real estate investments","False","gd_chart"
"gd_acc_039000","039000","asset_non_current","Impairment of buildings","False","gd_chart"
"gd_acc_040000","040000","asset_fixed","Equipment and spare parts valued according to the cost model","False","gd_chart"
"gd_acc_041000","041000","asset_fixed","Small inventory","False","gd_chart"
"gd_acc_042000","042000","asset_fixed","Equipment and spare parts valued according to the revaluation model","False","gd_chart"
"gd_acc_043000","043000","asset_fixed","Biological resources - perennial crops","False","gd_chart"
"gd_acc_044000","044000","asset_fixed","Biological resources - basic herd","False","gd_chart"
"gd_acc_045000","045000","asset_fixed","Other tangible fixed assets","False","gd_chart"
"gd_acc_046000","046000","asset_fixed","Investments in foreign-owned property, plant and equipment","False","gd_chart"
"gd_acc_047000","047000","asset_fixed","Equipment and other tangible fixed assets under construction","False","gd_chart"
"gd_acc_048000","048000","asset_fixed","Works of art and other objects of cultural and / or historical value that are not depreciated","False","gd_chart"
"gd_acc_050000","050000","asset_non_current","Depreciation of equipment and spare parts due to depreciation","False","gd_chart"
"gd_acc_051000","051000","asset_non_current","Value adjustment of small inventory due to depreciation","False","gd_chart"
"gd_acc_052000","052000","asset_non_current","Impairment of equipment and spare parts","False","gd_chart"
"gd_acc_053000","053000","asset_non_current","Value adjustment of biological assets - perennial crops due to depreciation","False","gd_chart"
"gd_acc_054000","054000","asset_non_current","Value adjustment of biological assets - basic herds due to depreciation","False","gd_chart"
"gd_acc_055000","055000","asset_non_current","Value adjustment of other property, plant and equipment due to depreciation","False","gd_chart"
"gd_acc_056000","056000","asset_non_current","Value adjustment of investments in property, plant and equipment of foreign ownership","False","gd_chart"
"gd_acc_058000","058000","asset_non_current","Impairment of biological assets","False","gd_chart"
"gd_acc_059000","059000","asset_non_current","Impairment of other property, plant and equipment","False","gd_chart"
"gd_acc_060000","060000","asset_non_current","Long-term financial investments in shares and stakes in group organizations, allocated and measured at cost","False","gd_chart"
"gd_acc_061000","061000","asset_non_current","Long-term investments in shares and stakes in group companies, allocated and measured at fair value through profit or loss","False","gd_chart"
"gd_acc_062000","062000","asset_non_current","Long-term investments in shares and stakes in group companies, allocated and measured at fair value through equity","False","gd_chart"
"gd_acc_063000","063000","asset_non_current","Long-term investments in shares and interests of associates and jointly controlled entities, allocated and measured at cost","False","gd_chart"
"gd_acc_064000","064000","asset_non_current","Long-term investments in shares and interests of associates and jointly controlled entities, allocated and measured at fair value through profit or loss","False","gd_chart"
"gd_acc_065000","065000","asset_non_current","Long-term investments in shares and interests of associates and jointly controlled entities, allocated and measured at fair value through equity","False","gd_chart"
"gd_acc_066000","066000","asset_non_current","Other long-term financial investments, allocated and measured at cost","False","gd_chart"
"gd_acc_067000","067000","asset_non_current","Other long-term financial investments allocated and measured at fair value through profit or loss","False","gd_chart"
"gd_acc_068000","068000","asset_non_current","Other long-term financial investments, allocated and measured at fair value through equity or own source of funds","False","gd_chart"
"gd_acc_069000","069000","asset_non_current","Impairment of long-term financial investments","False","gd_chart"
"gd_acc_070000","070000","asset_non_current","Long-term loans granted under loan agreements to group organizations, including long-term finance lease receivables","False","gd_chart"
"gd_acc_071000","071000","asset_non_current","Long-term loans granted under loan agreements to associates and jointly controlled entities, including long-term finance lease receivables","False","gd_chart"
"gd_acc_072000","072000","asset_non_current","Long-term loans to others, including long-term finance lease receivables","False","gd_chart"
"gd_acc_073000","073000","asset_non_current","Long-term loans granted by repurchasing bonds from group organizations","False","gd_chart"
"gd_acc_074000","074000","asset_non_current","Long-term loans granted by repurchase of bonds from associates and jointly controlled entities","False","gd_chart"
"gd_acc_075000","075000","asset_non_current","Long-term loans given by repurchasing bonds from others","False","gd_chart"
"gd_acc_076000","076000","asset_non_current","Long-term receivables for unpaid called - up capital","False","gd_chart"
"gd_acc_077000","077000","asset_non_current","Other long-term investments","False","gd_chart"
"gd_acc_078000","078000","asset_non_current","Long-term deposits given","False","gd_chart"
"gd_acc_079000","079000","asset_non_current","Impairment of long-term loans granted","False","gd_chart"
"gd_acc_080000","080000","asset_non_current","Long-term commodity loans granted in the country","False","gd_chart"
"gd_acc_081000","081000","asset_non_current","Long-term commodity loans granted abroad","False","gd_chart"
"gd_acc_082000","082000","asset_non_current","Long-term consumer loans granted","False","gd_chart"
"gd_acc_083000","083000","asset_non_current","Long-term advances given","False","gd_chart"
"gd_acc_084000","084000","asset_non_current","Long-term securities given","False","gd_chart"
"gd_acc_085000","085000","asset_non_current","Long-term finance lease receivables","False","gd_chart"
"gd_acc_086000","086000","asset_non_current","Other long-term operating receivables","False","gd_chart"
"gd_acc_087000","087000","asset_non_current","Long-term operating receivables from group companies","False","gd_chart"
"gd_acc_089000","089000","asset_non_current","Impairment of long-term operating receivables","False","gd_chart"
"gd_acc_090000","090000","asset_non_current","Deferred tax assets from deductible temporary differences","False","gd_chart"
"gd_acc_091000","091000","asset_non_current","Deferred tax assets from unused tax losses carried forward","False","gd_chart"
"gd_acc_092000","092000","asset_non_current","Deferred tax assets from tax credits carried forward","False","gd_chart"
"gd_acc_100000","100000","asset_cash","Cash on hand, except foreign currency","False","gd_chart"
"gd_acc_101000","101000","asset_cash","Foreign currency at the box office","False","gd_chart"
"gd_acc_102000","102000","asset_cash","Checks issued (deduction item)","False","gd_chart"
"gd_acc_103000","103000","asset_cash","Receipts received","False","gd_chart"
"gd_acc_104000","104000","asset_cash","Risk-free immediately redeemable debt securities","False","gd_chart"
"gd_acc_109000","109000","asset_cash","Money on the go","False","gd_chart"
"gd_acc_110000","110000","asset_cash","Cash on accounts other than foreign currency","False","gd_chart"
"gd_acc_111000","111000","asset_cash","Short-term or callable deposits, except foreign currency deposits","False","gd_chart"
"gd_acc_112000","112000","asset_cash","Foreign currency on accounts","False","gd_chart"
"gd_acc_113000","113000","asset_cash","Short-term foreign currency deposits or callable foreign currency deposits","False","gd_chart"
"gd_acc_114000","114000","asset_cash","Cash on special accounts or for special purposes","False","gd_chart"
"gd_acc_120000","120000","asset_receivable","Short-term receivables from customers in the country","True","gd_chart"
"gd_acc_121000","121000","asset_receivable","Short-term receivables from customers abroad","True","gd_chart"
"gd_acc_122000","122000","asset_current","Short-term commodity loans given to customers in the country","False","gd_chart"
"gd_acc_123000","123000","asset_current","Short-term commodity loans given to customers abroad","False","gd_chart"
"gd_acc_124000","124000","asset_current","Short-term consumer loans given to customers in the country","False","gd_chart"
"gd_acc_125000","125000","asset_current","Short-term receivables from members","False","gd_chart"
"gd_acc_126000","126000","asset_current","Short-term receivables for unbilled goods and services","False","gd_chart"
"gd_acc_127000","127000","asset_current","Short-term operating receivables from group companies","False","gd_chart"
"gd_acc_129000","129000","asset_non_current","Impairment of short-term trade receivables","False","gd_chart"
"gd_acc_130000","130000","asset_current","Short-term advances given for property, plant and equipment","False","gd_chart"
"gd_acc_131000","131000","asset_current","Short-term advances given for intangible assets","False","gd_chart"
"gd_acc_132000","132000","asset_current","Short-term advances paid for inventories of materials and goods and services not yet performed","False","gd_chart"
"gd_acc_133000","133000","asset_current","Other short-term advances and overpayments given","False","gd_chart"
"gd_acc_134000","134000","asset_current","Short-term securities given","False","gd_chart"
"gd_acc_139000","139000","asset_non_current","Impairment of short-term advances, overpayments and securities","False","gd_chart"
"gd_acc_140000","140000","asset_current","Short-term receivables from exporters","False","gd_chart"
"gd_acc_141000","141000","asset_current","Short-term import receivables for foreign account","False","gd_chart"
"gd_acc_142000","142000","asset_current","Short-term receivables from commission and consignment sales","False","gd_chart"
"gd_acc_145000","145000","asset_current","Other short-term operating receivables on behalf of others","False","gd_chart"
"gd_acc_149000","149000","asset_non_current","Impairment of short-term operating receivables on behalf of others","False","gd_chart"
"gd_acc_150000","150000","asset_current","Short-term interest receivables","False","gd_chart"
"gd_acc_151000","151000","asset_current","Short-term dividend receivables","False","gd_chart"
"gd_acc_152000","152000","asset_current","Short-term receivables for other profit shares","False","gd_chart"
"gd_acc_155000","155000","asset_current","Other short-term receivables related to financial income","False","gd_chart"
"gd_acc_159000","159000","asset_non_current","Impairment of short-term receivables related to financial income","False","gd_chart"
"gd_acc_160000","160000","asset_current","Short-term receivables for deductible vat","False","gd_chart"
"gd_acc_160800","160800","asset_current","VAT Receivable","False","gd_chart"
"gd_acc_161000","161000","asset_current","Short-term corporate income tax receivables, including tax paid abroad","False","gd_chart"
"gd_acc_162000","162000","asset_current","Short-term personal income tax receivables from the activities of sole proprietors, including tax paid abroad","False","gd_chart"
"gd_acc_164000","164000","asset_current","Other short-term receivables from state and other institutions","False","gd_chart"
"gd_acc_165000","165000","asset_current","Other short-term receivables","False","gd_chart"
"gd_acc_166000","166000","asset_current","Short-term vat receivables refunded to foreigners","False","gd_chart"
"gd_acc_167000","167000","asset_current","Short-term vat receivables paid abroad","False","gd_chart"
"gd_acc_169000","169000","asset_non_current","Impairment of other short-term receivables","False","gd_chart"
"gd_acc_170000","170000","asset_current","Short-term financial investments in shares and stakes in group organizations, allocated and measured at cost","False","gd_chart"
"gd_acc_171000","171000","asset_current","Short-term investments in shares and stakes in group companies, allocated and measured at fair value through profit or loss","False","gd_chart"
"gd_acc_172000","172000","asset_current","Short-term investments in shares and stakes in group companies, allocated and measured at fair value through equity","False","gd_chart"
"gd_acc_173000","173000","asset_current","Short-term investments in shares and interests of associates and jointly controlled entities, allocated and measured at cost","False","gd_chart"
"gd_acc_176000","176000","asset_current","Other short-term financial investments classified and measured at cost","False","gd_chart"
"gd_acc_177000","177000","asset_current","Other short-term financial investments allocated and measured at fair value through profit or loss","False","gd_chart"
"gd_acc_178000","178000","asset_current","Other short-term financial investments, allocated and measured at fair value through equity or own source of funds","False","gd_chart"
"gd_acc_179000","179000","asset_non_current","Impairment of short-term financial investments","False","gd_chart"
"gd_acc_180000","180000","asset_current","Short-term loans granted on the basis of loan agreements to group organizations","False","gd_chart"
"gd_acc_181000","181000","asset_current","Short-term loans granted on the basis of loan agreements to associated organizations and jointly controlled entities","False","gd_chart"
"gd_acc_182000","182000","asset_current","Short-term loans granted to others, including short-term finance lease receivables","False","gd_chart"
"gd_acc_183000","183000","asset_current","Short-term deposits with banks and other financial institutions","False","gd_chart"
"gd_acc_184000","184000","asset_current","Short-term loans granted by repurchase of bonds","False","gd_chart"
"gd_acc_185000","185000","asset_current","Bills of exchange received","False","gd_chart"
"gd_acc_186000","186000","asset_current","Short-term loans granted by repurchase of other debt securities","False","gd_chart"
"gd_acc_187000","187000","asset_current","Short-term unpaid called-up capital","False","gd_chart"
"gd_acc_189000","189000","asset_non_current","Impairment of short-term loans","False","gd_chart"
"gd_acc_190000","190000","asset_current","Short-term deferred costs or expenses","False","gd_chart"
"gd_acc_191000","191000","asset_current","Short-term accrued income","False","gd_chart"
"gd_acc_192000","192000","asset_current","Securities","False","gd_chart"
"gd_acc_195000","195000","asset_current","Vat on advances received","False","gd_chart"
"gd_acc_210000","210000","liability_current","Liabilities included in disposal groups","False","gd_chart"
"gd_acc_220000","220000","liability_payable","Short-term liabilities (debts) to suppliers in the country","True","gd_chart"
"gd_acc_221000","221000","liability_payable","Short-term liabilities (debts) to suppliers abroad","True","gd_chart"
"gd_acc_222000","222000","liability_current","Short-term commodity loans received in the country","False","gd_chart"
"gd_acc_223000","223000","liability_current","Short-term commodity loans received abroad","False","gd_chart"
"gd_acc_224000","224000","liability_current","Short-term liabilities (debts) for unbilled goods and services","False","gd_chart"
"gd_acc_227000","227000","liability_current","Short-term operating liabilities to group companies","False","gd_chart"
"gd_acc_230000","230000","liability_current","Short-term advances received","False","gd_chart"
"gd_acc_231000","231000","liability_current","Short-term securities received","False","gd_chart"
"gd_acc_240000","240000","liability_current","Short-term export liabilities for foreign account","False","gd_chart"
"gd_acc_241000","241000","liability_current","Short-term liabilities to importers","False","gd_chart"
"gd_acc_242000","242000","liability_current","Short-term liabilities from commission and consignment sales","False","gd_chart"
"gd_acc_245000","245000","liability_current","Other current liabilities for foreign account","False","gd_chart"
"gd_acc_250000","250000","liability_current","Short-term liabilities for accrued and unaccounted salaries","False","gd_chart"
"gd_acc_251000","251000","liability_current","Short-term liabilities for net wages and salaries","False","gd_chart"
"gd_acc_252000","252000","liability_current","Short-term liabilities for social security contributions by type of contribution","False","gd_chart"
"gd_acc_253000","253000","liability_current","Short-term liabilities for contributions from gross wages and salaries","False","gd_chart"
"gd_acc_254000","254000","liability_current","Short-term liabilities for taxes on gross wages and salaries","False","gd_chart"
"gd_acc_255000","255000","liability_current","Short-term liabilities for other employment benefits","False","gd_chart"
"gd_acc_256000","256000","liability_current","Short-term liabilities for contributions from other benefits from employment, which are not calculated together with salaries","False","gd_chart"
"gd_acc_257000","257000","liability_current","Short-term tax liabilities from other employment benefits that are not charged together with salaries","False","gd_chart"
"gd_acc_258000","258000","liability_current","Liabilities for payer 's contributions","False","gd_chart"
"gd_acc_260000","260000","liability_current","Liabilities for vat charged","False","gd_chart"
"gd_acc_260800","260800","liability_current","VAT Payable","False","gd_chart"
"gd_acc_261000","261000","liability_current","Liabilities for vat, customs and other duties on imported goods","False","gd_chart"
"gd_acc_262000","262000","liability_current","Liabilities for contributions","False","gd_chart"
"gd_acc_263000","263000","liability_current","Liabilities for advance payment of personal income tax on income from activities","False","gd_chart"
"gd_acc_264000","264000","liability_current","Corporate income tax liabilities","False","gd_chart"
"gd_acc_265000","265000","liability_current","Withholding tax liabilities","False","gd_chart"
"gd_acc_266000","266000","liability_current","Other short-term liabilities to state and other institutions","False","gd_chart"
"gd_acc_267000","267000","liability_current","Short-term liabilities for social security contributions of sole proprietors","False","gd_chart"
"gd_acc_270000","270000","liability_current","Short-term loans obtained from group organizations","False","gd_chart"
"gd_acc_271000","271000","liability_current","Short-term loans obtained from associates and jointly controlled entities","False","gd_chart"
"gd_acc_272000","272000","liability_current","Short-term loans obtained from banks and organizations in the country","False","gd_chart"
"gd_acc_273000","273000","liability_current","Short-term loans obtained from banks and organizations abroad","False","gd_chart"
"gd_acc_274000","274000","liability_current","Short-term financial liabilities related to bonds","False","gd_chart"
"gd_acc_275000","275000","liability_current","Short-term lease liabilities","False","gd_chart"
"gd_acc_276000","276000","liability_current","Short-term financial liabilities to individuals","False","gd_chart"
"gd_acc_277000","277000","liability_current","Short-term liabilities related to the distribution of profit or loss","False","gd_chart"
"gd_acc_278000","278000","liability_current","Liabilities from the payment of capital until entry in the court register","False","gd_chart"
"gd_acc_279000","279000","liability_current","Other short-term financial liabilities","False","gd_chart"
"gd_acc_280000","280000","liability_current","Short-term interest liabilities","False","gd_chart"
"gd_acc_281000","281000","liability_current","Short-term bill of exchange liabilities","False","gd_chart"
"gd_acc_282000","282000","liability_current","Short-term liabilities related to deductions from wages and salaries","False","gd_chart"
"gd_acc_285000","285000","liability_current","Other short-term operating liabilities","False","gd_chart"
"gd_acc_290000","290000","liability_current","Accrued costs or expenses","False","gd_chart"
"gd_acc_291000","291000","liability_current","Short-term deferred income","False","gd_chart"
"gd_acc_295000","295000","liability_current","Vat on advances given","False","gd_chart"
"gd_acc_300000","300000","asset_current","Value of raw materials and supplies according to suppliers' accounts","False","gd_chart"
"gd_acc_301000","301000","asset_current","Dependent costs of purchasing raw materials and supplies","False","gd_chart"
"gd_acc_302000","302000","asset_current","Customs duties and other import duties on raw materials and supplies","False","gd_chart"
"gd_acc_303000","303000","asset_current","Vat and other taxes on raw materials and supplies","False","gd_chart"
"gd_acc_309000","309000","asset_current","Accounting for the purchase of raw materials and supplies","False","gd_chart"
"gd_acc_310000","310000","asset_current","Stocks of raw materials and supplies in the warehouse","False","gd_chart"
"gd_acc_311000","311000","asset_current","Stocks of raw materials and supplies in a foreign warehouse","False","gd_chart"
"gd_acc_312000","312000","asset_current","Stocks of raw materials along the way","False","gd_chart"
"gd_acc_316000","316000","asset_current","Inventories of raw materials and supplies in finishing and processing","False","gd_chart"
"gd_acc_319000","319000","asset_current","Deviations from constant prices of raw materials and supplies","False","gd_chart"
"gd_acc_320000","320000","asset_current","Stocks of small inventory and packaging in the warehouse","False","gd_chart"
"gd_acc_321000","321000","asset_current","Stocks of small inventory and packaging put into use","False","gd_chart"
"gd_acc_329000","329000","asset_current","Deviations from constant prices of small inventory and packaging","False","gd_chart"
"gd_acc_400000","400000","expense","Material costs","False","gd_chart"
"gd_acc_401000","401000","expense","Costs of auxiliary material","False","gd_chart"
"gd_acc_402000","402000","expense","Energy costs","False","gd_chart"
"gd_acc_403000","403000","expense","Costs of spare parts for fixed assets and materials for the maintenance of fixed assets","False","gd_chart"
"gd_acc_404000","404000","expense","Write-off of small inventory and packaging","False","gd_chart"
"gd_acc_405000","405000","expense","Reconciliation of material costs and small inventory due to identified inventory differences","False","gd_chart"
"gd_acc_406000","406000","expense","Costs of office supplies and professional literature","False","gd_chart"
"gd_acc_407000","407000","expense","Other costs of material","False","gd_chart"
"gd_acc_410000","410000","expense","Costs of services in creating products and providing services","False","gd_chart"
"gd_acc_411000","411000","expense","Costs of transport services","False","gd_chart"
"gd_acc_412000","412000","expense","Costs of maintenance services","False","gd_chart"
"gd_acc_413000","413000","expense","Rents","False","gd_chart"
"gd_acc_414000","414000","expense","Reimbursement of expenses to employees in connection with work","False","gd_chart"
"gd_acc_415000","415000","expense","Payment transaction costs, banking services costs, transaction costs and insurance premiums","False","gd_chart"
"gd_acc_416000","416000","expense","Costs of intellectual and personal services","False","gd_chart"
"gd_acc_417000","417000","expense","Costs of fairs, advertising and representation","False","gd_chart"
"gd_acc_418000","418000","expense","Costs of services of natural persons who do not perform activities, together with duties charged to the organization (costs under employment contracts, copyright contracts, meeting fees for employees and other persons ...)","False","gd_chart"
"gd_acc_419000","419000","expense","Costs of other services","False","gd_chart"
"gd_acc_430000","430000","expense_depreciation","Depreciation of intangible assets","False","gd_chart"
"gd_acc_431000","431000","expense_depreciation","Depreciation of buildings","False","gd_chart"
"gd_acc_432000","432000","expense_depreciation","Depreciation of equipment and spare parts","False","gd_chart"
"gd_acc_433000","433000","expense_depreciation","Depreciation of small inventory","False","gd_chart"
"gd_acc_434000","434000","expense_depreciation","Depreciation of other property, plant and equipment","False","gd_chart"
"gd_acc_435000","435000","expense_depreciation","Depreciation of investment property","False","gd_chart"
"gd_acc_436000","436000","expense_depreciation","Depreciation of biological assets","False","gd_chart"
"gd_acc_440000","440000","expense","Provisions for organizational reorganization costs","False","gd_chart"
"gd_acc_441000","441000","expense","Reservations for warranty days","False","gd_chart"
"gd_acc_442000","442000","expense","Reservations for tricky contracts","False","gd_chart"
"gd_acc_443000","443000","expense","Provisions for pensions, jubilee awards and retirement benefits","False","gd_chart"
"gd_acc_449000","449000","expense","Provisions to cover other liabilities from past operations","False","gd_chart"
"gd_acc_450000","450000","expense","Interest costs","False","gd_chart"
"gd_acc_470000","470000","expense","Employee salaries","False","gd_chart"
"gd_acc_471000","471000","expense","Compensation of employees' salaries","False","gd_chart"
"gd_acc_472000","472000","expense","Costs of supplementary pension insurance for employees","False","gd_chart"
"gd_acc_473000","473000","expense","Annual leave allowance, bonuses, refunds (for transport to and from work, for food, for separate living) and other employee benefits","False","gd_chart"
"gd_acc_474000","474000","expense","Employer's contributions from salaries, wage compensations, bonuses, reimbursements and other employee benefits","False","gd_chart"
"gd_acc_475000","475000","expense","Other employer's benefits from salaries, wage compensations, bonuses, refunds and other employee benefits","False","gd_chart"
"gd_acc_476000","476000","expense","Rewards to apprentices along with levies charged to the organization","False","gd_chart"
"gd_acc_477000","477000","expense","Management costs charged on a basis other than employment","False","gd_chart"
"gd_acc_478000","478000","expense","Other labor costs","False","gd_chart"
"gd_acc_479000","479000","expense","Provisions for pensions, jubilee awards and retirement benefits","False","gd_chart"
"gd_acc_480000","480000","expense","Benefits that do not depend on labor costs or other types of costs","False","gd_chart"
"gd_acc_481000","481000","expense","Expenditure on environmental protection","False","gd_chart"
"gd_acc_482000","482000","expense","Awards to pupils and students on work placements together with benefits","False","gd_chart"
"gd_acc_483000","483000","expense","Scholarships for high school and university students","False","gd_chart"
"gd_acc_484000","484000","expense","Social security contributions of sole proprietors","False","gd_chart"
"gd_acc_486000","486000","expense","Reimbursement of costs to sole proprietors","False","gd_chart"
"gd_acc_488000","488000","expense","Grants to other associations and legal entities","False","gd_chart"
"gd_acc_489000","489000","expense","Other costs","False","gd_chart"
"gd_acc_490000","490000","expense","Transfer of costs to inventories","False","gd_chart"
"gd_acc_491000","491000","expense","Transfer of costs directly to expenses","False","gd_chart"
"gd_acc_600000","600000","asset_current","Work in progress","False","gd_chart"
"gd_acc_601000","601000","asset_current","Incomplete services","False","gd_chart"
"gd_acc_602000","602000","asset_current","Semi-finished products","False","gd_chart"
"gd_acc_604000","604000","asset_current","Production in finishing and processing","False","gd_chart"
"gd_acc_609000","609000","asset_current","Deviations from prices of work in progress and services","False","gd_chart"
"gd_acc_610000","610000","asset_current","Harvested crops at fair value","False","gd_chart"
"gd_acc_618000","618000","asset_current","Harvested crops at cost","False","gd_chart"
"gd_acc_619000","619000","asset_current","Deviations from crop prices","False","gd_chart"
"gd_acc_630000","630000","asset_current","Products in own warehouse","False","gd_chart"
"gd_acc_631000","631000","asset_current","Products in a foreign warehouse","False","gd_chart"
"gd_acc_632000","632000","asset_current","Products on the go","False","gd_chart"
"gd_acc_633000","633000","asset_current","Products in our own store","False","gd_chart"
"gd_acc_634000","634000","asset_current","Included vat on products in the store","False","gd_chart"
"gd_acc_635000","635000","asset_current","Products in finishing and processing","False","gd_chart"
"gd_acc_638000","638000","asset_current","Biological assets that are stocks","False","gd_chart"
"gd_acc_639000","639000","asset_current","Deviations from product prices","False","gd_chart"
"gd_acc_650000","650000","asset_current","Value of goods according to suppliers' accounts","False","gd_chart"
"gd_acc_651000","651000","asset_current","Dependent costs of purchasing goods","False","gd_chart"
"gd_acc_652000","652000","asset_current","Customs and other import duties on goods","False","gd_chart"
"gd_acc_653000","653000","asset_current","Vat and other taxes on goods","False","gd_chart"
"gd_acc_659000","659000","asset_current","Accounting for the purchase of goods","False","gd_chart"
"gd_acc_660000","660000","asset_current","Goods in own warehouse","False","gd_chart"
"gd_acc_661000","661000","asset_current","Goods in a foreign warehouse","False","gd_chart"
"gd_acc_662000","662000","asset_current","Goods on the way","False","gd_chart"
"gd_acc_663000","663000","asset_current","Goods in own store","False","gd_chart"
"gd_acc_664000","664000","asset_current","Vat included in stocks of goods","False","gd_chart"
"gd_acc_669000","669000","asset_current","Included difference in prices of stocks of goods","False","gd_chart"
"gd_acc_670000","670000","asset_non_current","Tangible fixed assets held for sale","False","gd_chart"
"gd_acc_671000","671000","asset_non_current","Investment property valued at cost","False","gd_chart"
"gd_acc_672000","672000","asset_non_current","Other non-current assets held for sale","False","gd_chart"
"gd_acc_673000","673000","asset_non_current","Assets work money-generating units intended for sale","False","gd_chart"
"gd_acc_674000","674000","asset_non_current","Assets of a cash-generating unit held for sale","False","gd_chart"
"gd_acc_700000","700000","expense","Value of business effects sold","False","gd_chart"
"gd_acc_701000","701000","expense","Value of capitalized own products and services","False","gd_chart"
"gd_acc_702000","702000","expense","Cost of goods and goods sold","False","gd_chart"
"gd_acc_703000","703000","expense","Other operating expenses","False","gd_chart"
"gd_acc_704000","704000","expense","Expenditure on evaluation of biological assets and harvesting of agricultural products","False","gd_chart"
"gd_acc_710000","710000","expense","Value of business effects sold","False","gd_chart"
"gd_acc_711000","711000","expense","Cost of goods and goods sold","False","gd_chart"
"gd_acc_712000","712000","expense","Selling costs","False","gd_chart"
"gd_acc_713000","713000","expense","General and administrative expenses (purchasing and administration)","False","gd_chart"
"gd_acc_714000","714000","expense","Other costs not held in stock","False","gd_chart"
"gd_acc_720000","720000","expense","Revaluation operating expenses related to intangible assets, property, plant and equipment and investment property","False","gd_chart"
"gd_acc_721000","721000","expense","Revaluation operating expenses related to inventories","False","gd_chart"
"gd_acc_722000","722000","expense","Revaluation operating expenses as a result of revaluation due to impairment in relation to operating receivables","False","gd_chart"
"gd_acc_723000","723000","expense","Revaluation operating expenses as a result of write-offs related to operating receivables","False","gd_chart"
"gd_acc_724000","724000","expense","Other revaluation operating expenses related to current assets, except financial investments","False","gd_chart"
"gd_acc_740000","740000","expense","Expenses from loans received from group organizations","False","gd_chart"
"gd_acc_741000","741000","expense","Expenses from loans received from banks","False","gd_chart"
"gd_acc_742000","742000","expense","Expenses from issued bonds","False","gd_chart"
"gd_acc_743000","743000","expense","Expenses from other financial liabilities","False","gd_chart"
"gd_acc_744000","744000","expense","Expenses from operating liabilities to group organizations","False","gd_chart"
"gd_acc_745000","745000","expense","Expenses from trade payables and bill of exchange liabilities","False","gd_chart"
"gd_acc_746000","746000","expense","Expenses from other operating liabilities, including interest expenses from the recalculation of severance pay upon retirement","False","gd_chart"
"gd_acc_747000","747000","expense","Expenses from assets allocated at fair value through profit or loss and expenses from the valuation of investment property at fair value","False","gd_chart"
"gd_acc_748000","748000","expense","Expenses from impairment of financial investments","False","gd_chart"
"gd_acc_749000","749000","expense","Expenses from derecognition of financial investments and investment property measured at fair value","False","gd_chart"
"gd_acc_750000","750000","expense","Expenses from valuation of investment property according to the fair value model","False","gd_chart"
"gd_acc_751000","751000","expense","Expenses from disposal of investment property measured at fair value","False","gd_chart"
"gd_acc_752000","752000","expense","Fines not related to business effects","False","gd_chart"
"gd_acc_753000","753000","expense","Compensation not related to business effects","False","gd_chart"
"gd_acc_754000","754000","expense","Donations","False","gd_chart"
"gd_acc_755000","755000","expense","Subsidies, grants and similar expenditure not related to business performance","False","gd_chart"
"gd_acc_758000","758000","expense","Negative euro equalizations","False","gd_chart"
"gd_acc_759000","759000","expense","Other non-operating expenses","False","gd_chart"
"gd_acc_760000","760000","income","Revenues from sales of products and services on the domestic market","False","gd_chart"
"gd_acc_761000","761000","income","Revenues from sales of products and services on foreign markets","False","gd_chart"
"gd_acc_761100","761100","income","Revenues from sales of products and services on the eu market","False","gd_chart"
"gd_acc_761200","761200","income","Revenues from sales of products and services outside the eu market","False","gd_chart"
"gd_acc_762000","762000","income","Revenues from sales of merchandise and materials on the domestic market","False","gd_chart"
"gd_acc_763000","763000","income","Revenues from sales of merchandise and materials on foreign markets","False","gd_chart"
"gd_acc_763100","763100","income","Revenue from the sale of merchandise and materials on the eu market","False","gd_chart"
"gd_acc_763200","763200","income","Revenue from the sale of merchandise and materials outside the eu market","False","gd_chart"
"gd_acc_764000","764000","income","Revenues from valuation of biological assets and harvesting of agricultural products","False","gd_chart"
"gd_acc_765000","765000","income","Rental income","False","gd_chart"
"gd_acc_766000","766000","income","Revenues from the elimination of provisions and accrued costs and deferred revenue at the expense of accrued costs or expenses","False","gd_chart"
"gd_acc_767000","767000","income","Revenues from business combinations (revaluation surplus - bad name)","False","gd_chart"
"gd_acc_768000","768000","income","Other revenues related to business performance (subsidies, grants, resources, compensations, premiums ...)","False","gd_chart"
"gd_acc_769000","769000","income","Revaluation operating revenues","False","gd_chart"
"gd_acc_770000","770000","income","Financial revenues from shares in group organizations","False","gd_chart"
"gd_acc_771000","771000","income","Financial income from interests in associates and jointly controlled entities","False","gd_chart"
"gd_acc_772000","772000","income","Financial income from shares in other organizations","False","gd_chart"
"gd_acc_773000","773000","income","Financial income from other investments","False","gd_chart"
"gd_acc_774000","774000","income","Financial income from loans granted to group organizations","False","gd_chart"
"gd_acc_775000","775000","income","Financial income from loans to others (including deposits)","False","gd_chart"
"gd_acc_776000","776000","income","Financial revenues from operating receivables from group organizations","False","gd_chart"
"gd_acc_777000","777000","income","Financial revenues from operating receivables from others","False","gd_chart"
"gd_acc_778000","778000","income","Financial income from financial assets allocated at fair value through profit or loss","False","gd_chart"
"gd_acc_779000","779000","income","Financial income from the valuation of investment property at fair value and income from the disposal of investment property measured at fair value","False","gd_chart"
"gd_acc_780000","780000","income_other","Revenue from the valuation of investment property at fair value","False","gd_chart"
"gd_acc_781000","781000","income_other","Income from disposal of investment property measured at fair value","False","gd_chart"
"gd_acc_784000","784000","income_other","Donations","False","gd_chart"
"gd_acc_785000","785000","income_other","Subsidies, grants and similar revenues not related to business performance","False","gd_chart"
"gd_acc_786000","786000","income_other","Compensation not related to business effects","False","gd_chart"
"gd_acc_787000","787000","income_other","Penalties not related to business effects","False","gd_chart"
"gd_acc_788000","788000","income_other","Positive euro equalizations","False","gd_chart"
"gd_acc_789000","789000","income_other","Other non-operating income","False","gd_chart"
"gd_acc_810000","810000","expense","Corporate income tax","False","gd_chart"
"gd_acc_812000","812000","expense","Other taxes not included in other items","False","gd_chart"
"gd_acc_813000","813000","expense","Deferred tax income (expenses)","False","gd_chart"
"gd_acc_815000","815000","equity_unaffected","Net profit for the financial year or net surplus of revenues","False","gd_chart"
"gd_acc_900000","900000","equity","Share capital - shares","False","gd_chart"
"gd_acc_901000","901000","equity","Share capital - capital shares or capital contribution","False","gd_chart"
"gd_acc_902000","902000","equity","Initial capital of sole proprietors","False","gd_chart"
"gd_acc_903000","903000","equity","Share capital - capital deposit","False","gd_chart"
"gd_acc_905000","905000","equity","Inseparable cooperative capital","False","gd_chart"
"gd_acc_906000","906000","equity","Shares of cooperative members","False","gd_chart"
"gd_acc_907000","907000","equity","Founding and subsequent roles","False","gd_chart"
"gd_acc_909000","909000","equity","Uncalled capital (deductible item)","False","gd_chart"
"gd_acc_910000","910000","equity","Payments above the minimum issue amounts of shares or stakes (paid-in capital surplus)","False","gd_chart"
"gd_acc_911000","911000","equity","Payments over the book value in case of disposal of temporarily repurchased own shares or stakes","False","gd_chart"
"gd_acc_912000","912000","equity","Payments above the minimum issue amount of capital obtained by issuing convertible bonds and bonds with a share option","False","gd_chart"
"gd_acc_913000","913000","equity","Payments for the acquisition of additional rights from shares or stakes","False","gd_chart"
"gd_acc_914000","914000","equity","Other capital payments based on the articles of association","False","gd_chart"
"gd_acc_915000","915000","equity","Amounts from the simplified reduction of share capital and amounts of reduction of share capital by withdrawal of shares or stakes","False","gd_chart"
"gd_acc_916000","916000","equity","General capital revaluation adjustment and amounts transferred from the revaluation reserve","False","gd_chart"
"gd_acc_917000","917000","equity","Amounts from the effects of confirmed compulsory settlement","False","gd_chart"
"gd_acc_918000","918000","equity","Transfers of tangible assets in the course of business","False","gd_chart"
"gd_acc_919000","919000","equity","Inflows and outflows between enterprise and household","False","gd_chart"
"gd_acc_920000","920000","equity","Legal reserves","False","gd_chart"
"gd_acc_921000","921000","equity","Reserves for own shares or own business shares","False","gd_chart"
"gd_acc_922000","922000","equity","Statutory reserves","False","gd_chart"
"gd_acc_923000","923000","equity","Other profit reserves","False","gd_chart"
"gd_acc_924000","924000","equity","Voluntary cooperative reserves","False","gd_chart"
"gd_acc_925000","925000","equity","Voluntary cooperative funds","False","gd_chart"
"gd_acc_926000","926000","equity","Part of the net surplus of revenue for a specific purpose (fund)","False","gd_chart"
"gd_acc_929000","929000","equity","Acquired treasury shares or treasury shares (deductible item)","False","gd_chart"
"gd_acc_930000","930000","equity","Retained earnings from previous years","False","gd_chart"
"gd_acc_931000","931000","equity","Carried forward net loss from previous years","False","gd_chart"
"gd_acc_933000","933000","equity","Net loss for the financial year or net surplus of expenses","False","gd_chart"
"gd_acc_934000","934000","equity","Transfer from revaluation surplus","False","gd_chart"
"gd_acc_935000","935000","equity","Income of sole proprietors","False","gd_chart"
"gd_acc_937000","937000","equity","Negative profit of sole proprietors","False","gd_chart"
"gd_acc_940000","940000","equity","Revaluation reserves from land revaluation","False","gd_chart"
"gd_acc_941000","941000","equity","Revaluation reserves from revaluation of buildings","False","gd_chart"
"gd_acc_949000","949000","equity","Value adjustment of revaluation reserves for deferred tax liabilities","False","gd_chart"
"gd_acc_954000","954000","equity","Reserves arising from the valuation of long-term financial investments at fair value","False","gd_chart"
"gd_acc_955000","955000","equity","Reserves arising from the valuation of short-term financial investments at fair value","False","gd_chart"
"gd_acc_956000","956000","equity","Amounts of proven profit or loss from change in the fair value of available-for-sale financial assets that are not part of a hedging relationship","False","gd_chart"
"gd_acc_957000","957000","equity","Actuarial gains or losses on certain earnings","False","gd_chart"
"gd_acc_959000","959000","equity","Value adjustment of reserves arising from the measurement at fair value of deferred tax liabilities","False","gd_chart"
"gd_acc_960000","960000","liability_non_current","Provisions for organizational reorganization costs","False","gd_chart"
"gd_acc_961000","961000","liability_non_current","Provisions to cover future costs or expenses due to decommissioning and restoration and other similar provisions","False","gd_chart"
"gd_acc_962000","962000","liability_non_current","Reservations for tricky contracts","False","gd_chart"
"gd_acc_963000","963000","liability_non_current","Provisions for pensions, jubilee awards and retirement benefits","False","gd_chart"
"gd_acc_964000","964000","liability_non_current","Reservations for warranty days","False","gd_chart"
"gd_acc_965000","965000","liability_non_current","Other provisions for long-term accrued expenses","False","gd_chart"
"gd_acc_966000","966000","liability_non_current","State aid received","False","gd_chart"
"gd_acc_967000","967000","liability_non_current","Donations received","False","gd_chart"
"gd_acc_968000","968000","liability_non_current","Other long-term accrued costs and deferred revenue","False","gd_chart"
"gd_acc_970000","970000","liability_non_current","Long-term loans obtained from group organizations","False","gd_chart"
"gd_acc_971000","971000","liability_non_current","Long-term loans obtained from associated organizations","False","gd_chart"
"gd_acc_972000","972000","liability_non_current","Long-term loans obtained from banks and organizations in the country","False","gd_chart"
"gd_acc_973000","973000","liability_non_current","Long-term loans obtained from banks and organizations abroad","False","gd_chart"
"gd_acc_974000","974000","liability_non_current","Long-term financial liabilities related to bonds and bills of exchange","False","gd_chart"
"gd_acc_975000","975000","liability_non_current","Long-term lease debts","False","gd_chart"
"gd_acc_976000","976000","liability_non_current","Long-term financial liabilities to individuals","False","gd_chart"
"gd_acc_979000","979000","liability_non_current","Other long-term financial liabilities","False","gd_chart"
"gd_acc_980000","980000","liability_non_current","Long-term loans obtained on the basis of credit agreements from group organizations","False","gd_chart"
"gd_acc_981000","981000","liability_non_current","Long-term loans obtained under credit agreements from associates and jointly controlled entities","False","gd_chart"
"gd_acc_982000","982000","liability_non_current","Long-term loans obtained from other domestic suppliers","False","gd_chart"
"gd_acc_983000","983000","liability_non_current","Long-term loans obtained from other foreign suppliers","False","gd_chart"
"gd_acc_984000","984000","liability_non_current","Long-term financial lease debts","False","gd_chart"
"gd_acc_985000","985000","liability_non_current","Long-term bill of exchange liabilities","False","gd_chart"
"gd_acc_986000","986000","liability_non_current","Long-term advances and securities received","False","gd_chart"
"gd_acc_988000","988000","liability_non_current","Deferred tax liabilities","False","gd_chart"
"gd_acc_989000","989000","liability_non_current","Other long-term operating liabilities","False","gd_chart"
"gd_acc_990000","990000","off_balance","Leased, borrowed and leased (foreign) assets","False","gd_chart"
"gd_acc_991000","991000","off_balance","Bills of exchange and other securities received to secure payments","False","gd_chart"
"gd_acc_992000","992000","off_balance","Goods received for commission and consignment sale","False","gd_chart"
"gd_acc_993000","993000","off_balance","Securities issued for accounting within the organization","False","gd_chart"
"gd_acc_994000","994000","off_balance","Other active off-balance sheet accounts","False","gd_chart"
"gd_acc_995000","995000","off_balance","Owners of leased, borrowed and leased assets","False","gd_chart"
"gd_acc_996000","996000","off_balance","Debtors who secured payments with bills of exchange and other securities","False","gd_chart"
"gd_acc_997000","997000","off_balance","Liabilities from goods received for commission and consignment sales","False","gd_chart"
"gd_acc_998000","998000","off_balance","Nominal value of securities issued for accounting within the organization","False","gd_chart"
"gd_acc_999000","999000","off_balance","Other passive off-balance sheet accounts","False","gd_chart"

```

## File: data\account.group.template.csv

```csv
"id","code_prefix_start","name","chart_template_id/id"
"gd_acc_group_0","0","Long-term assets","gd_chart"
"gd_acc_group_00","00","Intangible assets and long-term accrued costs and deferred revenue","gd_chart"
"gd_acc_group_01","01","Investment property","gd_chart"
"gd_acc_group_02","02","Real estate","gd_chart"
"gd_acc_group_03","03","Impairment and impairment of real estate","gd_chart"
"gd_acc_group_04","04","Equipment and other tangible fixed assets","gd_chart"
"gd_acc_group_05","05","Adjustment and impairment of equipment and other property, plant and equipment","gd_chart"
"gd_acc_group_06","06","Long-term financial investments, except loans","gd_chart"
"gd_acc_group_07","07","Long-term loans and receivables given for unpaid called-up capital","gd_chart"
"gd_acc_group_08","08","Long-term operating receivables","gd_chart"
"gd_acc_group_09","09","Deferred tax assets","gd_chart"
"gd_acc_group_1","1","Short-term assets, except inventories, and short-term accrued and deferred income","gd_chart"
"gd_acc_group_10","10","Cash on hand and immediately realizable securities","gd_chart"
"gd_acc_group_11","11","Balances with banks and other financial institutions","gd_chart"
"gd_acc_group_12","12","Short-term trade receivables","gd_chart"
"gd_acc_group_13","13","Short-term advances and securities given","gd_chart"
"gd_acc_group_14","14","Short-term operating receivables for a foreign account","gd_chart"
"gd_acc_group_15","15","Short-term receivables related to financial income","gd_chart"
"gd_acc_group_16","16","Other short-term receivables","gd_chart"
"gd_acc_group_17","17","Short-term financial investments, except loans","gd_chart"
"gd_acc_group_18","18","Short-term loans and short-term receivables for unpaid capital","gd_chart"
"gd_acc_group_19","19","Short-term accrued costs and deferred revenue","gd_chart"
"gd_acc_group_2","2","Short-term liabilities (debt) and short-term accrued and deferred income","gd_chart"
"gd_acc_group_21","21","Liabilities included in disposal groups","gd_chart"
"gd_acc_group_22","22","Short-term liabilities (debts) to suppliers","gd_chart"
"gd_acc_group_23","23","Short-term advances and securities received","gd_chart"
"gd_acc_group_24","24","Short-term operating liabilities for a foreign account","gd_chart"
"gd_acc_group_25","25","Short-term wage liabilities","gd_chart"
"gd_acc_group_26","26","Liabilities to state and other institutions","gd_chart"
"gd_acc_group_27","27","Short-term financial liabilities","gd_chart"
"gd_acc_group_28","28","Other current liabilities","gd_chart"
"gd_acc_group_29","29","Shorthand passive time relations","gd_chart"
"gd_acc_group_3","3","Stocks of raw materials","gd_chart"
"gd_acc_group_30","30","Accounting for the purchase of raw materials and supplies (including small inventory and packaging)","gd_chart"
"gd_acc_group_31","31","Inventories of raw materials and supplies","gd_chart"
"gd_acc_group_32","32","Stocks of small inventory and packaging","gd_chart"
"gd_acc_group_4","4","Costs","gd_chart"
"gd_acc_group_40","40","Material costs","gd_chart"
"gd_acc_group_41","41","Cost of service","gd_chart"
"gd_acc_group_43","43","Depreciation","gd_chart"
"gd_acc_group_44","44","Reservations","gd_chart"
"gd_acc_group_45","45","Interest costs","gd_chart"
"gd_acc_group_47","47","Labor costs","gd_chart"
"gd_acc_group_48","48","Other costs","gd_chart"
"gd_acc_group_49","49","Cost transfer","gd_chart"
"gd_acc_group_6","6","Inventories of products, services, goods and non-current assets (disposal groups) for sale","gd_chart"
"gd_acc_group_60","60","Work in progress and services","gd_chart"
"gd_acc_group_61","61","Stocks of crops (harvested) from biological assets","gd_chart"
"gd_acc_group_63","63","Products","gd_chart"
"gd_acc_group_65","65","Accounting for the purchase of goods","gd_chart"
"gd_acc_group_66","66","Stocks of goods","gd_chart"
"gd_acc_group_67","67","Non-current assets (disposal groups) for sale","gd_chart"
"gd_acc_group_7","7","Expenses and revenues","gd_chart"
"gd_acc_group_70","70","Operating expenses (I. version of the income statement)","gd_chart"
"gd_acc_group_71","71","Operating expenses (II. Version of the income statement)","gd_chart"
"gd_acc_group_72","72","Revaluation operating expenses","gd_chart"
"gd_acc_group_74","74","Financial expenses","gd_chart"
"gd_acc_group_75","75","Other expenses","gd_chart"
"gd_acc_group_76","76","Business income","gd_chart"
"gd_acc_group_77","77","Financial income","gd_chart"
"gd_acc_group_78","78","Other incomes","gd_chart"
"gd_acc_group_79","79","Capitalize own products and own services","gd_chart"
"gd_acc_group_8","8","Profit or loss","gd_chart"
"gd_acc_group_80","80","Profit or loss before tax","gd_chart"
"gd_acc_group_81","81","Distribution of profit and / or total surplus revenue","gd_chart"
"gd_acc_group_82","82","Allocation of net profit for the financial year or net surplus of revenues","gd_chart"
"gd_acc_group_89","89","Net loss or net surplus of expenses and transfer of net loss or net surplus of expenses","gd_chart"
"gd_acc_group_9","9","Capital, long-term liabilities (debt) and long-term provisions","gd_chart"
"gd_acc_group_90","90","Called-up and initial capital and founding deposits","gd_chart"
"gd_acc_group_91","91","Capital reserves and transfers of funds","gd_chart"
"gd_acc_group_92","92","Profit reserves or allocated net surplus of revenues","gd_chart"
"gd_acc_group_93","93","Net profit or net loss or unallocated net surplus of revenues or net surplus of expenses","gd_chart"
"gd_acc_group_94","94","Revaluation reserves","gd_chart"
"gd_acc_group_95","95","Reserves arising from fair value measurement","gd_chart"
"gd_acc_group_96","96","Provisions and long-term accrued costs and deferred revenue","gd_chart"
"gd_acc_group_97","97","Long-term financial liabilities","gd_chart"
"gd_acc_group_98","98","Long-term operating liabilities","gd_chart"
"gd_acc_group_99","99","Off-balance sheet accounts","gd_chart"

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="gd_chart" model="account.chart.template">
        <field name="property_account_receivable_id" ref="gd_acc_120000"/>
        <field name="property_account_payable_id" ref="gd_acc_220000"/>
        <field name="property_account_expense_categ_id" ref="gd_acc_702000"/>
        <field name="property_account_income_categ_id" ref="gd_acc_762000"/>
        <field name="expense_currency_exchange_account_id" ref="gd_acc_484000"/>
        <field name="income_currency_exchange_account_id" ref="gd_acc_777000"/>
        <field name="default_pos_receivable_account_id" ref="gd_acc_125000"/>
        <field name="property_tax_payable_account_id" ref="gd_acc_260800"/>
        <field name="property_tax_receivable_account_id" ref="gd_acc_160800"/>
    </record>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_si.gd_chart')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_fiscal_position_account_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="gd_fp_eu_acc1" model="account.fiscal.position.account.template">
        <field name="position_id" ref="gd_fp_eu"/>
        <field name="account_src_id" ref="gd_acc_120000"/>
        <field name="account_dest_id" ref="gd_acc_121000"/>
    </record>

    <record id="gd_fp_eu_acc2" model="account.fiscal.position.account.template">
        <field name="position_id" ref="gd_fp_eu"/>
        <field name="account_src_id" ref="gd_acc_220000"/>
        <field name="account_dest_id" ref="gd_acc_221000"/>
    </record>

    <record id="gd_fp_eu_acc3" model="account.fiscal.position.account.template">
        <field name="position_id" ref="gd_fp_eu"/>
        <field name="account_src_id" ref="gd_acc_760000"/>
        <field name="account_dest_id" ref="gd_acc_761000"/>
    </record>

    <record id="gd_fp_eu_acc4" model="account.fiscal.position.account.template">
        <field name="position_id" ref="gd_fp_eu"/>
        <field name="account_src_id" ref="gd_acc_762000"/>
        <field name="account_dest_id" ref="gd_acc_763000"/>
    </record>

    <record id="gd_fp_ne_acc1" model="account.fiscal.position.account.template">
        <field name="position_id" ref="gd_fp_ne"/>
        <field name="account_src_id" ref="gd_acc_120000"/>
        <field name="account_dest_id" ref="gd_acc_121000"/>
    </record>

    <record id="gd_fp_ne_acc2" model="account.fiscal.position.account.template">
        <field name="position_id" ref="gd_fp_ne"/>
        <field name="account_src_id" ref="gd_acc_220000"/>
        <field name="account_dest_id" ref="gd_acc_221000"/>
    </record>

    <record id="gd_fp_ne_acc3" model="account.fiscal.position.account.template">
        <field name="position_id" ref="gd_fp_ne"/>
        <field name="account_src_id" ref="gd_acc_760000"/>
        <field name="account_dest_id" ref="gd_acc_761000"/>
    </record>

    <record id="gd_fp_ne_acc4" model="account.fiscal.position.account.template">
        <field name="position_id" ref="gd_fp_ne"/>
        <field name="account_src_id" ref="gd_acc_762000"/>
        <field name="account_dest_id" ref="gd_acc_763000"/>
    </record>
</odoo>

```

## File: data\account_fiscal_position_tax_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- EU partner -->
        <record id="gd_fp_eu_sale_vat_22" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="gd_fp_eu"/>
            <field name="tax_src_id" ref="gd_taxr_3"/>
            <field name="tax_dest_id" ref="gd_taxr_1"/>
        </record>

        <record id="gd_fp_eu_sale_vat_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="gd_fp_eu"/>
            <field name="tax_src_id" ref="gd_taxr_2"/>
            <field name="tax_dest_id" ref="gd_taxr_1"/>
        </record>

        <record id="gd_fp_eu_sale_vat_5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="gd_fp_eu"/>
            <field name="tax_src_id" ref="l10n_si_vat_5_sale"/>
            <field name="tax_dest_id" ref="gd_taxr_1"/>
        </record>

        <record id="gd_fp_eu_purchase_vat_22" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="gd_fp_eu"/>
            <field name="tax_src_id" ref="gd_taxp_3"/>
            <field name="tax_dest_id" ref="gd_taxp_st_2"/>
        </record>

        <record id="gd_fp_eu_purchase_vat_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="gd_fp_eu"/>
            <field name="tax_src_id" ref="gd_taxp_2"/>
            <field name="tax_dest_id" ref="gd_taxp_st_1"/>
        </record>

        <record id="gd_fp_eu_purchase_vat_5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="gd_fp_eu"/>
            <field name="tax_src_id" ref="l10n_si_vat_5_purchase"/>
            <field name="tax_dest_id" ref="l10n_si_vat_5_purchase_goods_eu"/>
        </record>
        <!-- EU partner [END] -->

        <!-- Partner outside the EU -->
        <record id="gd_fp_ne_purchase_vat_22" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="gd_fp_ne"/>
            <field name="tax_src_id" ref="gd_taxp_3"/>
            <field name="tax_dest_id" ref="gd_taxp_1"/>
        </record>

        <record id="gd_fp_ne_purchase_vat_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="gd_fp_ne"/>
            <field name="tax_src_id" ref="gd_taxp_2"/>
            <field name="tax_dest_id" ref="gd_taxp_1"/>
        </record>

        <record id="gd_fp_ne_purchase_vat_5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="gd_fp_ne"/>
            <field name="tax_src_id" ref="l10n_si_vat_5_purchase"/>
            <field name="tax_dest_id" ref="gd_taxp_1"/>
        </record>
        <!-- Partner outside the EU [END] -->
    </data>
</odoo>

```

## File: data\account_fiscal_position_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="gd_fp_exempt" model="account.fiscal.position.template">
            <field name="name">Exempt taxpayer</field>
            <field name="chart_template_id" ref="gd_chart"/>
        </record>

        <record id="gd_fp_do" model="account.fiscal.position.template">
            <field name="name">Domestic</field>
            <field name="sequence">10</field>
            <field name="chart_template_id" ref="gd_chart"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_id" ref="base.si"/>
        </record>

        <record id="gd_fp_do1" model="account.fiscal.position.template">
            <field name="name">EU partner private</field>
            <field name="sequence">20</field>
            <field name="chart_template_id" ref="gd_chart"/>
            <field name="auto_apply" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>

        <record id="gd_fp_eu" model="account.fiscal.position.template">
            <field name="name">EU partner</field>
            <field name="sequence">30</field>
            <field name="chart_template_id" ref="gd_chart"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>

        <record id="gd_fp_ne" model="account.fiscal.position.template">
            <field name="name">Partner outside the EU</field>
            <field name="sequence">40</field>
            <field name="chart_template_id" ref="gd_chart"/>
            <field name="auto_apply" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Sales -->
    <record id="gd_taxr_3" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="name">22% VAT</field>
        <field name="description">VAT charged at a rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_11_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_21_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_11_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_21_tag')],
            }),
        ]"/>
    </record>
    <record id="gd_taxr_2" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="name">9,5% VAT</field>
        <field name="description">VAT charged at a rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_11_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_22_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_11_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_22_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_5_sale" model="account.tax.template">
        <field name="sequence">30</field>
        <field name="name">5% VAT</field>
        <field name="description">VAT charged at a rate of 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_11_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_22a_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_11_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_22a_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_recipient_22_sale" model="account.tax.template">
        <field name="sequence">40</field>
        <field name="name">22% VAT recipient</field>
        <field name="description">VAT charged by the recipient at a rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_11a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_11a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_recipient_9_sale" model="account.tax.template">
        <field name="sequence">50</field>
        <field name="name">9,5% VAT recipient</field>
        <field name="description">VAT charged by the recipient at a rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_11a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_11a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_recipient_5_sale" model="account.tax.template">
        <field name="sequence">60</field>
        <field name="name">5% VAT recipient</field>
        <field name="description">VAT charged by the recipient at a rate of 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_11a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_11a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="gd_taxr_1" model="account.tax.template">
        <field name="sequence">70</field>
        <field name="name">0% VAT EU</field>
        <field name="description">VAT charged from acquisitions of goods and services from other EU Member States at a rate of 0%</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_12_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_12_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_22_sale_distance" model="account.tax.template">
        <field name="sequence">80</field>
        <field name="name">22% VAT distance</field>
        <field name="description">Distance selling goods at a rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_13_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_23_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_13_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_23_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_9_sale_distance" model="account.tax.template">
        <field name="sequence">90</field>
        <field name="name">9,5% VAT distance</field>
        <field name="description">Distance selling goods at a rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_13_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_24_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_13_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_24_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_5_sale_distance" model="account.tax.template">
        <field name="sequence">100</field>
        <field name="name">5% VAT distance</field>
        <field name="description">Distance selling goods at a rate of 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_13_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_24b_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_13_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_24b_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_22_sale_installation_eu" model="account.tax.template">
        <field name="sequence">110</field>
        <field name="name">22% VAT EU installation</field>
        <field name="description">VAT charged from assembly and installation of goods in other EU Member States at a rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_14_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_23_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_14_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_23_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_9_sale_installation_eu" model="account.tax.template">
        <field name="sequence">120</field>
        <field name="name">9,5% VAT EU installation</field>
        <field name="description">VAT charged from assembly and installation of goods in other EU Member States at a rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_14_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_24_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_14_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_24_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_5_sale_installation_eu" model="account.tax.template">
        <field name="sequence">130</field>
        <field name="name">5% VAT EU installation</field>
        <field name="description">VAT charged from assembly and installation of goods in other EU Member States at a rate of 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_14_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_24b_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_14_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_24b_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_0_sale_no_deduction" model="account.tax.template">
        <field name="sequence">140</field>
        <field name="name">0% VAT without deduction</field>
        <field name="description">VAT charged at a rate of 0% without deduction</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_15_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_15_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_self_22_sale" model="account.tax.template">
        <field name="sequence">150</field>
        <field name="name">22% VAT self-assessment</field>
        <field name="description">VAT charged on the basis of self-assessment as a recipient of goods and services at a rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_25_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_25_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_self_9_sale" model="account.tax.template">
        <field name="sequence">160</field>
        <field name="name">9,5% VAT self-assessment</field>
        <field name="description">VAT charged on the basis of self-assessment as a recipient of goods and services at a rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_25a_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_25a_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_self_5_sale" model="account.tax.template">
        <field name="sequence">170</field>
        <field name="name">5% VAT self-assessment</field>
        <field name="description">VAT charged on the basis of self-assessment as a recipient of goods and services at a rate of 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_25b_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_25b_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_self_22_sale_76a" model="account.tax.template">
        <field name="sequence">180</field>
        <field name="name">22% VAT 76.a</field>
        <field name="description">VAT charged on the basis of self-assessment as a recipient of goods and services specified in Article 76.a at a rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_25_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_25_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_self_9_sale_76a" model="account.tax.template">
        <field name="sequence">190</field>
        <field name="name">9,5% VAT 76.a</field>
        <field name="description">VAT charged on the basis of self-assessment as a recipient of goods and services specified in Article 76.a at a rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_25a_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_25a_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_self_22_sale_imports" model="account.tax.template">
        <field name="sequence">200</field>
        <field name="name">22% VAT imports self-assessment</field>
        <field name="description">VAT charged on the basis of self-assessment of imports at a rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_26_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_26_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_self_9_sale_imports" model="account.tax.template">
        <field name="sequence">210</field>
        <field name="name">9,5% VAT imports self-assessment</field>
        <field name="description">VAT charged on the basis of self-assessment of imports at a rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_26_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_26_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_self_5_sale_imports" model="account.tax.template">
        <field name="sequence">220</field>
        <field name="name">5% VAT imports self-assessment</field>
        <field name="description">VAT charged on the basis of self-assessment of imports at a rate of 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'plus_report_expression_ids': [ref('tax_report_26_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260000'),
                'minus_report_expression_ids': [ref('tax_report_26_tag')],
            }),
        ]"/>
    </record>
    <!-- Sales [END] -->
    <!-- Purchases -->
    <record id="gd_taxp_3" model="account.tax.template">
        <field name="sequence">230</field>
        <field name="name">22% VAT</field>
        <field name="description">VAT deduction at a rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_41_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_41_tag')],
            }),
        ]"/>
    </record>
    <record id="gd_taxp_2" model="account.tax.template">
        <field name="sequence">240</field>
        <field name="name">9,5% VAT</field>
        <field name="description">VAT deduction at a rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_42_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_42_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_5_purchase" model="account.tax.template">
        <field name="sequence">250</field>
        <field name="name">5% VAT</field>
        <field name="description">VAT deduction at a rate of 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_42a_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_42a_tag')],
            }),
        ]"/>
    </record>
    <record id="gd_taxp_nr_2" model="account.tax.template">
        <field name="sequence">260</field>
        <field name="name">22% VAT non-deductible</field>
        <field name="description">VAT non-deductible at a rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
        ]"/>
    </record>
    <record id="gd_taxp_nr_1" model="account.tax.template">
        <field name="sequence">270</field>
        <field name="name">9,5% VAT non-deductible</field>
        <field name="description">VAT non-deductible at a rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_5_purchase_non_deductible" model="account.tax.template">
        <field name="sequence">280</field>
        <field name="name">5% VAT non-deductible</field>
        <field name="description">VAT non-deductible at a rate of 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_expression_ids': [ref('tax_report_31_tag')],
            }),
        ]"/>
    </record>
    <record id="gd_taxp_st_2" model="account.tax.template">
        <field name="sequence">290</field>
        <field name="name">22% VAT EU goods</field>
        <field name="description">VAT deduction on acquisitions of goods from other EU Member States at the rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_32_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_23_tag'), ref('tax_report_41_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_32_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_23_tag'), ref('tax_report_41_tag')],
            }),
        ]"/>
    </record>
    <record id="gd_taxp_st_1" model="account.tax.template">
        <field name="sequence">300</field>
        <field name="name">9,5% VAT EU goods</field>
        <field name="description">VAT deduction on acquisitions of goods from other EU Member States at the rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_32_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_24_tag'), ref('tax_report_42_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_32_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_24_tag'), ref('tax_report_42_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_5_purchase_goods_eu" model="account.tax.template">
        <field name="sequence">310</field>
        <field name="name">5% VAT EU goods</field>
        <field name="description">VAT deduction on acquisitions of goods from other EU Member States at the rate of 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_32_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_24b_tag'), ref('tax_report_42a_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_32_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_24b_tag'), ref('tax_report_42a_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_22_purchase_services_eu" model="account.tax.template">
        <field name="sequence">320</field>
        <field name="name">22% VAT EU services</field>
        <field name="description">VAT deduction on acquisitions of services from other EU Member States at the rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_32a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_23a_tag'), ref('tax_report_41_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_32a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_23a_tag'), ref('tax_report_41_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_9_purchase_services_eu" model="account.tax.template">
        <field name="sequence">330</field>
        <field name="name">9,5% VAT EU services</field>
        <field name="description">VAT deduction on acquisitions of services from other EU Member States at the rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_32a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_24a_tag'), ref('tax_report_42_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_32a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_24a_tag'), ref('tax_report_42_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_5_purchase_services_eu" model="account.tax.template">
        <field name="sequence">340</field>
        <field name="name">5% VAT EU services</field>
        <field name="description">VAT deduction on acquisitions of services from other EU Member States at the rate of 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_32a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_24c_tag'), ref('tax_report_42a_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_32a_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_24c_tag'), ref('tax_report_42a_tag')],
            }),
        ]"/>
    </record>
    <record id="gd_taxp_1" model="account.tax.template">
        <field name="sequence">350</field>
        <field name="name">0% VAT</field>
        <field name="description">VAT deduction from purchases of goods and services, acquisition of goods and services received from other EU Member States and from imports at a rate of 0%</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_33_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_33_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_22_purchase_estate" model="account.tax.template">
        <field name="sequence">360</field>
        <field name="name">22% VAT real estate</field>
        <field name="description">VAT deduction from the purchase of real estate at the rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag'), ref('tax_report_34_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_41_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag'), ref('tax_report_34_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_41_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_9_purchase_estate" model="account.tax.template">
        <field name="sequence">370</field>
        <field name="name">9,5% VAT real estate</field>
        <field name="description">VAT deduction from the purchase of real estate at the rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag'), ref('tax_report_34_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_42_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag'), ref('tax_report_34_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_42_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_5_purchase_estate" model="account.tax.template">
        <field name="sequence">380</field>
        <field name="name">5% VAT real estate</field>
        <field name="description">VAT deduction from the purchase of real estate at the rate of 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag'), ref('tax_report_34_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_42a_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag'), ref('tax_report_34_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_42a_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_22_purchase_estate_76a" model="account.tax.template">
        <field name="sequence">390</field>
        <field name="name">22% VAT 76.a</field>
        <field name="description">VAT deduction from the purchase of real estate specified in Article 76.a at a rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31a_tag'), ref('tax_report_34_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_41_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31a_tag'), ref('tax_report_34_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_41_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_9_purchase_estate_76a" model="account.tax.template">
        <field name="sequence">400</field>
        <field name="name">9,5% VAT 76.a</field>
        <field name="description">VAT deduction from the purchase of real estate specified in Article 76.a at a rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31a_tag'), ref('tax_report_34_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_42_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31a_tag'), ref('tax_report_34_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_42_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_5_purchase_estate_76a" model="account.tax.template">
        <field name="sequence">410</field>
        <field name="name">5% VAT 76.a</field>
        <field name="description">VAT deduction from the purchase of real estate specified in Article 76.a at a rate of 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31a_tag'), ref('tax_report_34_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_42a_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31a_tag'), ref('tax_report_34_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_42a_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_22_purchase_fixed" model="account.tax.template">
        <field name="sequence">420</field>
        <field name="name">22% VAT fixed assets</field>
        <field name="description">VAT deduction from the purchase of other fixed assets at the rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag'), ref('tax_report_35_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_41_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag'), ref('tax_report_35_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_41_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_9_purchase_fixed" model="account.tax.template">
        <field name="sequence">430</field>
        <field name="name">9,5% VAT fixed assets</field>
        <field name="description">VAT deduction from the purchase of other fixed assets at the rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag'), ref('tax_report_35_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_42_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag'), ref('tax_report_35_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_42_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_5_purchase_fixed" model="account.tax.template">
        <field name="sequence">440</field>
        <field name="name">5% VAT fixed assets</field>
        <field name="description">VAT deduction from the purchase of other fixed assets at the rate of 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_31_tag'), ref('tax_report_35_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_42a_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_31_tag'), ref('tax_report_35_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_42a_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_0_purchase_fixed" model="account.tax.template">
        <field name="sequence">450</field>
        <field name="name">0% VAT fixed assets</field>
        <field name="description">VAT deduction from the purchase of other fixed assets at the rate of 0%</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_33_tag'), ref('tax_report_35_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_33_tag'), ref('tax_report_35_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_22_purchase_fixed_eu" model="account.tax.template">
        <field name="sequence">460</field>
        <field name="name">22% VAT EU fixed assets</field>
        <field name="description">VAT deduction from the purchase of other fixed assets from other EU Member States at the rate of 22%</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_32_tag'), ref('tax_report_35_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_41_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_32_tag'), ref('tax_report_35_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_41_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_9_purchase_fixed_eu" model="account.tax.template">
        <field name="sequence">470</field>
        <field name="name">9,5% VAT EU fixed assets</field>
        <field name="description">VAT deduction from the purchase of other fixed assets from other EU Member States at the rate of 9,5%</field>
        <field name="amount">9.5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_32_tag'), ref('tax_report_35_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_42_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_32_tag'), ref('tax_report_35_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_42_tag')],
            }),
        ]"/>
    </record>
    <record id="l10n_si_vat_5_purchase_fixed_eu" model="account.tax.template">
        <field name="sequence">480</field>
        <field name="name">5% VAT EU fixed assets</field>
        <field name="description">VAT deduction from the purchase of other fixed assets from other EU Member States at the rate of 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="active" eval="False"/>
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_32_tag'), ref('tax_report_35_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'plus_report_expression_ids': [ref('tax_report_42a_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_32_tag'), ref('tax_report_35_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160000'),
                'minus_report_expression_ids': [ref('tax_report_42a_tag')],
            }),
        ]"/>
    </record>
    <!-- Purchases [END] -->
</odoo>

```

## File: data\account_tax_group.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="tax_group_22" model="account.tax.group">
            <field name="name">22% VAT</field>
            <field name="country_id" ref="base.si"/>
        </record>

        <record id="tax_group_95" model="account.tax.group">
            <field name="name">9,5% VAT</field>
            <field name="country_id" ref="base.si"/>
        </record>

        <record id="tax_group_5" model="account.tax.group">
            <field name="name">5% VAT</field>
            <field name="country_id" ref="base.si"/>
        </record>

        <record id="tax_group_0" model="account.tax.group">
            <field name="name">0% VAT</field>
            <field name="country_id" ref="base.si"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.si"/>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_I" model="account.report.line">
                <field name="name">I. Supplies of goods and services (values excluding VAT)</field>
                <field name="sequence">1000</field>
                <field name="expression_ids">
                    <record id="tax_report_I_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">c11.balance + c11a.balance + c12.balance + c13.balance + c14.balance + c15.balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_11" model="account.report.line">
                        <field name="name">11. Supplies of goods and services</field>
                        <field name="code">c11</field>
                        <field name="sequence">1110</field>
                        <field name="expression_ids">
                            <record id="tax_report_11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_11a" model="account.report.line">
                        <field name="name">11a. Supplies of goods and services in Slovenia, of which VAT is charged by the recipient</field>
                        <field name="code">c11a</field>
                        <field name="sequence">1111</field>
                        <field name="expression_ids">
                            <record id="tax_report_11a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_12" model="account.report.line">
                        <field name="name">12. Deliveries of goods and services to other EU Member States</field>
                        <field name="code">c12</field>
                        <field name="sequence">1120</field>
                        <field name="expression_ids">
                            <record id="tax_report_12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">12</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_13" model="account.report.line">
                        <field name="name">13. Sale of goods at a distance</field>
                        <field name="code">c13</field>
                        <field name="sequence">1130</field>
                        <field name="expression_ids">
                            <record id="tax_report_13_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">13</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_14" model="account.report.line">
                        <field name="name">14. Assembly and installation of goods in another Member State</field>
                        <field name="code">c14</field>
                        <field name="sequence">1140</field>
                        <field name="expression_ids">
                            <record id="tax_report_14_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">14</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_15" model="account.report.line">
                        <field name="name">15. Exempt supplies without the right to deduct VAT</field>
                        <field name="code">c15</field>
                        <field name="sequence">1150</field>
                        <field name="expression_ids">
                            <record id="tax_report_15_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">15</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_II" model="account.report.line">
                <field name="name">II. VAT charged</field>
                <field name="sequence">2000</field>
                <field name="expression_ids">
                    <record id="tax_report_II_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">c21.balance + c22.balance + c22a.balance + c23.balance + c23a.balance + c24.balance + c24a.balance + c24b.balance + c24c.balance + c25.balance + c25a.balance + c25b.balance + c26.balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_21" model="account.report.line">
                        <field name="name">21. At a rate of 22%</field>
                        <field name="code">c21</field>
                        <field name="sequence">2210</field>
                        <field name="expression_ids">
                            <record id="tax_report_21_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">21</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_22" model="account.report.line">
                        <field name="name">22. At a rate of 9,5%</field>
                        <field name="code">c22</field>
                        <field name="sequence">2220</field>
                        <field name="expression_ids">
                            <record id="tax_report_22_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">22</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_22a" model="account.report.line">
                        <field name="name">22a. At a rate of 5%</field>
                        <field name="code">c22a</field>
                        <field name="sequence">2221</field>
                        <field name="expression_ids">
                            <record id="tax_report_22a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">22a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_23" model="account.report.line">
                        <field name="name">23. 22% of acquisitions of goods from other EU Member States</field>
                        <field name="code">c23</field>
                        <field name="sequence">2230</field>
                        <field name="expression_ids">
                            <record id="tax_report_23_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">23</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_23a" model="account.report.line">
                        <field name="name">23a. Of the services received from other EU Member States at a rate of 22%</field>
                        <field name="code">c23a</field>
                        <field name="sequence">2231</field>
                        <field name="expression_ids">
                            <record id="tax_report_23a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">23a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_24" model="account.report.line">
                        <field name="name">24. 9,5% of acquisitions of goods from other EU Member States</field>
                        <field name="code">c24</field>
                        <field name="sequence">2240</field>
                        <field name="expression_ids">
                            <record id="tax_report_24_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">24</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_24a" model="account.report.line">
                        <field name="name">24a. Of the services received from other EU Member States at the rate of 9,5%</field>
                        <field name="code">c24a</field>
                        <field name="sequence">2241</field>
                        <field name="expression_ids">
                            <record id="tax_report_24a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">24a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_24b" model="account.report.line">
                        <field name="name">24b. Acquisitions of goods from other EU Member States at the rate of 5%</field>
                        <field name="code">c24b</field>
                        <field name="sequence">2242</field>
                        <field name="expression_ids">
                            <record id="tax_report_24b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">24b</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_24c" model="account.report.line">
                        <field name="name">24c. Of the services received from other EU Member States at the rate of 5%</field>
                        <field name="code">c24c</field>
                        <field name="sequence">2243</field>
                        <field name="expression_ids">
                            <record id="tax_report_24c_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">24c</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_25" model="account.report.line">
                        <field name="name">25. On the basis of self-assessment as a recipient of goods and services at a rate of 22%</field>
                        <field name="code">c25</field>
                        <field name="sequence">2250</field>
                        <field name="expression_ids">
                            <record id="tax_report_25_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">25</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_25a" model="account.report.line">
                        <field name="name">25a. On the basis of self-assessment as a recipient of goods and services at a rate of 9,5%</field>
                        <field name="code">c25a</field>
                        <field name="sequence">2251</field>
                        <field name="expression_ids">
                            <record id="tax_report_25a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">25a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_25b" model="account.report.line">
                        <field name="name">25b. On the basis of self-assessment as a recipient of goods and services at a rate of 5%</field>
                        <field name="code">c25b</field>
                        <field name="sequence">2252</field>
                        <field name="expression_ids">
                            <record id="tax_report_25b_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">25b</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_26" model="account.report.line">
                        <field name="name">26. On the basis of self-assessment of imports</field>
                        <field name="code">c26</field>
                        <field name="sequence">2260</field>
                        <field name="expression_ids">
                            <record id="tax_report_26_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">26</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_III" model="account.report.line">
                <field name="name">III. Purchases of goods and services (values excluding VAT)</field>
                <field name="sequence">3000</field>
                <field name="expression_ids">
                    <record id="tax_report_III_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">c31.balance + c31a.balance + c32.balance + c32a.balance + c33.balance + c34.balance + c35.balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_31" model="account.report.line">
                        <field name="name">31. Purchases of goods and services</field>
                        <field name="code">c31</field>
                        <field name="sequence">3310</field>
                        <field name="expression_ids">
                            <record id="tax_report_31_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">31</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_31a" model="account.report.line">
                        <field name="name">31a. Purchases of goods and services in Slovenia, of which the recipient charges VAT</field>
                        <field name="code">c31a</field>
                        <field name="sequence">3311</field>
                        <field name="expression_ids">
                            <record id="tax_report_31a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">31a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_32" model="account.report.line">
                        <field name="name">32. Acquisitions of goods from other EU Member States</field>
                        <field name="code">c32</field>
                        <field name="sequence">3320</field>
                        <field name="expression_ids">
                            <record id="tax_report_32_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">32</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_32a" model="account.report.line">
                        <field name="name">32a. Services received from other EU Member States</field>
                        <field name="code">c32a</field>
                        <field name="sequence">3321</field>
                        <field name="expression_ids">
                            <record id="tax_report_32a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">32a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_33" model="account.report.line">
                        <field name="name">33. Exempt purchases of goods and services and exempt acquisitions of goods</field>
                        <field name="code">c33</field>
                        <field name="sequence">3330</field>
                        <field name="expression_ids">
                            <record id="tax_report_33_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">33</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_34" model="account.report.line">
                        <field name="name">34. Purchase value of real estate</field>
                        <field name="code">c34</field>
                        <field name="sequence">3340</field>
                        <field name="expression_ids">
                            <record id="tax_report_34_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">34</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_35" model="account.report.line">
                        <field name="name">35. Cost of other fixed assets</field>
                        <field name="code">c35</field>
                        <field name="sequence">3350</field>
                        <field name="expression_ids">
                            <record id="tax_report_35_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">35</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_IV" model="account.report.line">
                <field name="name">IV. VAT deduction</field>
                <field name="sequence">4000</field>
                <field name="expression_ids">
                    <record id="tax_report_IV_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">c41.balance + c42.balance + c42a.balance + c43.balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="tax_report_41" model="account.report.line">
                        <field name="name">41. From purchases of goods and services, acquisition of goods and services received from other EU Member States and from imports at a rate of 22%</field>
                        <field name="code">c41</field>
                        <field name="sequence">4410</field>
                        <field name="expression_ids">
                            <record id="tax_report_41_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">41</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_42" model="account.report.line">
                        <field name="name">42. From purchases of goods and services, acquisition of goods and services received from other EU Member States and from imports at a rate of 9,5%</field>
                        <field name="code">c42</field>
                        <field name="sequence">4420</field>
                        <field name="expression_ids">
                            <record id="tax_report_42_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">42</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_42a" model="account.report.line">
                        <field name="name">42a. From purchases of goods and services, acquisition of goods and services received from other EU Member States and from imports at a rate of 5%</field>
                        <field name="code">c42a</field>
                        <field name="sequence">4421</field>
                        <field name="expression_ids">
                            <record id="tax_report_42a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">42a</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_43" model="account.report.line">
                        <field name="name">43. Of the flat-rate compensation at the rate of 8%</field>
                        <field name="code">c43</field>
                        <field name="sequence">4430</field>
                        <field name="expression_ids">
                            <record id="tax_report_43_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">43</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_51" model="account.report.line">
                <field name="name">51. VAT liability</field>
                <field name="sequence">5100</field>
                <field name="expression_ids">
                    <record id="tax_report_51_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">c21.balance + c22.balance + c22a.balance + c23.balance + c23a.balance + c24.balance + c24a.balance + c24b.balance + c25.balance + c25a.balance + c25b.balance + c26.balance - c41.balance - c42.balance - c42a.balance - c43.balance</field>
                        <field name="subformula">if_above(EUR(0))</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_52" model="account.report.line">
                <field name="name">52. VAT surplus</field>
                <field name="sequence">5200</field>
                <field name="expression_ids">
                    <record id="tax_report_52_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">c41.balance + c42.balance + c42a.balance + c43.balance - c21.balance - c22.balance - c22a.balance - c23.balance - c23a.balance - c24.balance - c24a.balance - c24b.balance - c25.balance - c25a.balance - c25b.balance - c26.balance</field>
                        <field name="subformula">if_above(EUR(0))</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_si_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="gd_chart" model="account.chart.template">
        <field name="name">Slovenian Chart of Accounts</field>
        <field name="bank_account_code_prefix">110</field>
        <field name="cash_account_code_prefix">100</field>
        <field name="transfer_account_code_prefix">109</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.EUR"/>
        <field name="country_id" ref="base.si"/>
        <field name="spoken_languages" eval="'sl_SI'"/>
        <field name="use_storno_accounting" eval="True"/>
    </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="3.6" y="4.92" width="64.8" height="35.1" maskUnits="userSpaceOnUse">
      <rect x="5.09" y="7.8" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="1200" height="600" transform="translate(3.6 4.92) scale(0.05 0.06)" xlink:href="data:image/png;base64,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JmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJA2CaWEqQPwbk8AAAAASUVORK5CYII="/>
    </g>
  </g>
</svg>

```

