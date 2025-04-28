# Odoo Module: l10n_mz

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': "Mozambique - Accounting",
    'description': """
        Mozambican Accounting localization
    """,
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'depends': ['base', 'account'],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/account.group.template.csv',
        'data/account_pgcpe_mozambique.xml',
        'data/tax_report.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_fiscal_position_data.xml',
        'data/account_chart_template_configure_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","account_type","chart_template_id/id","reconcile"
"l10n_mz_account_211","Purchased Goods","211","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_2121","Purchased Raw materials","2121","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_2122","Purchased Ancillary materials","2122","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_21231","Purchased Fuel and lubricants","21231","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_21232","Purchased Packaging","21232","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_21233","Purchased Spare parts","21233","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_21239","Other Purchased materials","21239","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_217","Returned purchases","217","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_218","Discounts and rebates on purchases","218","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_222","Goods in transit","222","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_223","Goods held by third parties","223","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_231","Finished and intermediate goods","231","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_232","Finished goods held by third parties","232","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_241","By-products","241","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_242","Waste and scrap","242","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_25","Work in progress","25","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_261","Raw materials","261","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_262","Ancillary materials","262","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_2631","Fuel and lubricants","2631","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_2632","Packaging","2632","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_2633","Spare parts","2633","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_2639","Other materials","2639","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_264","Raw materials and other supplies in transit","264","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_2711","Livestock for production","2711","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_2712","Plants for production","2712","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_2721","Consumable livestock","2721","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_2722","Consumable Plants","2722","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_282","Inventory adjustments - Goods","282","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_283","Inventory adjustments - Finished and intermediate goods","283","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_284","Inventory adjustments - By-products, waste and scrap","284","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_285","Inventory adjustments - Work in progress","285","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_286","Inventory adjustments - Raw materials and other supplies","286","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_287","Inventory adjustments - Biological assets","287","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_292","Net realisable value adjustments - Goods","292","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_293","Net realisable value adjustments - Finished and intermediate goods","293","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_294","Net realisable value adjustments - By-products, waste and scrap","294","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_295","Net realisable value adjustments - Work in progress","295","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_296","Net realisable value adjustments - Raw materials and other supplies","296","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_297","Net realisable value adjustments - Biological assets","297","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_311","Investments in subsidiaries","311","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_312","Investments in associates","312","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_313","Other financial investments","313","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_3211","Industrial buildings","3211","asset_fixed","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_3212","Administrative and commercial offices","3212","asset_fixed","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_3213","Buildings for housing and other social purposes","3213","asset_fixed","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_3216","Transport infrastructures and similar constructions","3216","asset_fixed","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_322","Equipment","322","asset_fixed","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_323","Furniture and fixtures","323","asset_fixed","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_324","Vehicles","324","asset_fixed","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_325","Returnable containers","325","asset_fixed","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_326","Tools and utensils","326","asset_fixed","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_329","Other tangible assets","329","asset_fixed","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_331","Development costs","331","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_332","Intellectual property and other rights","332","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_333","Goodwill","333","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_334","Set-up or expansion costs","334","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_342","Tangible Assets under construction","342","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_343","Intangible Assets under construction","343","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_36","Investment property","36","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_382","Accumulated depreciation and amortisation - Tangible assets","382","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_383","Accumulated depreciation and amortisation - Intangible assets","383","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_386","Accumulated depreciation and amortisation - Investment property","386","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_39","Financial investments adjustments","39","asset_non_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_411","Trade receivables - current account","411","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_412","Securities receivable","412","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_413","Point of sale receivable","413","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_418","Doubtful debts","418","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_419","Advances from clients","419","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_421","Trade payables – current account","421","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_422","Securities payable","422","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_429","Advances to suppliers","429","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4311","Bank loans - Short term","4311","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4312","Bank loans - Medium and long term","4312","liability_non_current","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4411","Income tax - Tax estimate","4411","liability_current","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4412","Income tax - Progress payments","4412","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4413","Income tax - Special progress payments","4413","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4421","Withholding tax - Income from employment","4421","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4422","Withholding tax - Professional income","4422","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4423","Withholding tax - Capital returns","4423","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4424","Withholding tax - Property income","4424","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4425","Withholding tax - Other income","4425","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_44311","Input VAT - Inventories","44311","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_44312","Input VAT - Tangible and intangible assets","44312","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_44313","Input VAT - Other goods and services","44313","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_44321","Deductible VAT - Inventories","44321","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_44322","Deductible VAT - Tangible and intangible assets","44322","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_44323","Deductible VAT - Other goods and services","44323","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_44331","Assessed VAT - General transactions","44331","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_44332","Assessed VAT - Self consumption and gifts","44332","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_44333","Assessed VAT - Special transactions","44333","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_44341","Monthly VAT adjustments in favour of taxable person","44341","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_44342","Monthly VAT adjustments in favour of State","44342","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_44343","Annual VAT adjustments by calculation of final pro rata","44343","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4435","VAT assessment","4435","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4436","VAT assessed by Tax Authorities","4436","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4437","VAT payable","4437","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4438","VAT recoverable","4438","asset_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4439","VAT requested refunds","4439","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4441","Stamp duty","4441","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4442","MunicipaI taxes","4442","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_445","Tax adjustments, contributions and other levies","445","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_449","INSS contributions","449","liability_current","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_4511","Advances to corporate bodies","4511","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4512","Advances to employees","4512","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4518","Other receivable transactions with corporate bodies","4518","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4519","Other receivable transactions with employees","4519","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4521","State and other public entities","4521","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4522","Private entities","4522","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4529","Other entities","4529","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4541","Loans receivable","4541","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4542","Advances on profits","4542","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4543","Distributed profits and losses","4543","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4544","Available profits","4544","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4549","Other receivable transactions","4549","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4551","Grants receivable - State and other public entities","4551","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4552","Grants receivable - Private entities","4552","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_459","Other debtors","459","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4611","Capital expenditure creditors – Current account","4611","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4612","Capital expenditure creditors - Payable securities","4612","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4613","Capital expenditure creditors - Advances","4613","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4614","Capital expenditure creditors - Finance lease","4614","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4619","Capital expenditure creditors - Other","4619","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4621","Remuneration payable to corporate bodies","4621","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4622","Remuneration payable to employees","4622","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4628","Other transactions with corporate bodies","4628","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4629","Other transactions with employees","4629","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_463","Trade unions","463","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_466","Consultants, advisors and intermediaries","466","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4671","Borrowings from partners, shareholders or owners","4671","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4673","Distributed profits","4673","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4674","Available profits","4674","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_469","Other creditors","469","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_471","Accounts receivable adjustments - Trade receivables","471","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_472","Accounts receivable adjustments - Other receivables","472","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_481","Provisions - Outstanding legal matters","481","liability_non_current","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_482","Provisions - Accidents at work and occupational diseases","482","liability_non_current","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_483","Provisions - Taxes","483","liability_current","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_484","Provisions - Business restructuring","484","liability_non_current","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_485","Provisions - Onerous contracts","485","liability_non_current","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_486","Provisions - Warranty obligations","486","liability_non_current","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_487","Provisions - Losses on construction contracts","487","liability_current","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_489","Provisions - Other","489","liability_non_current","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4911","Interest payable","4911","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4912","Remuneration payable","4912","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4919","Other accrued expenses","4919","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4923","Revenue from construction contracts","4923","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4924","Investment grants","4924","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4929","Other deferred income","4929","liability_payable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4931","Interest receivable","4931","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4933","Revenue from construction contracts","4933","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_4939","Other accrued income","4939","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_494","Deferred expenses","494","asset_receivable","l10n_mz.l10n_mz_chart_template","True"
"l10n_mz_account_51","Share capital","51","equity","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_521","Treasury shares - Nominal value","521","equity","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_522","Treasury shares - Discounts and premiums","522","equity","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_53","Supplementary capital","53","equity","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_54","Share premium","54","equity","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_551","Legal reserves","551","equity","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_552","Statutory reserves","552","equity","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_553","Free reserves","553","equity","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_561","Legal revaluations","561","equity","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_562","Other surplus","562","equity","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_58","Other changes in equity","58","equity","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6112","Cost of Goods","6112","expense_direct_cost","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_61161","Cost of Raw materials","61161","expense_direct_cost","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_61162","Cost of Ancillary materials","61162","expense_direct_cost","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_61163","Cost of Other materials","61163","expense_direct_cost","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6117","Cost of Biological assets","6117","expense_direct_cost","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6121","Change in production - Finished and intermediate goods","6121","expense_direct_cost","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6122","Change in production - By-products, waste and scrap","6122","expense_direct_cost","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6123","Change in production - Work in progress","6123","expense_direct_cost","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_621","Remuneration of corporate bodies","621","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_622","Remuneration of employees","622","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_623","Charges on remuneration","623","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6251","Allowances - Taxable","6251","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6252","Allowances - Non taxable","6252","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6261","Indemnities - Insurable risk","6261","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6262","Indemnities - Other","6262","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_627","Insurance covering accidents at work and occupational diseases","627","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_628","Expenses of social nature","628","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_629","Other staff expenses","629","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_631","Subcontracts","631","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63211","Water","63211","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63212","Electricity","63212","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63213","Fuel","63213","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63214","Fast wear and tear tools","63214","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63215","Maintenance and repair material","63215","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63216","Stationary","63216","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63217","Technical books and documentation","63217","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63218","Gifts","63218","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63221","Maintenance and repair","63221","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63222","Freight services","63222","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63223","Staff transport","63223","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63224","Communications","63224","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63225","Fees","63225","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63226","Commissions to intermediaries","63226","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_632271","Advertising – Campaigns","632271","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_632272","Advertising - Other","632272","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_632281","Travel and accommodation – In business","632281","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_632282","Travel and accommodation - Other","632282","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63229","Entertainment expenses","63229","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63231","Litigation and notary expenses","63231","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_632321","Hire and rental charges - Finance lease","632321","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_632331","Life, personal accidents and disease insurance","632331","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63234","Royalties","63234","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63235","Cleaning, hygiene and comfort expenses","63235","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63236","Surveillance and security","63236","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63237","Specialised services","63237","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_63299","Other supplies of goods and services","63299","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_641","Adjustments from inventories to net realisable value","641","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_642","Adjustments from Financial investments","642","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_643","Adjustments from Investment property","643","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6441","Receivables – adjustments within tax limits","6441","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6442","Receivables – adjustments beyond tax limits","6442","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_651","Depreciation and amortisation for the period - Tangible assets","651","expense_depreciation","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_652","Depreciation and amortisation for the period - Intangible assets","652","expense_depreciation","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_653","Depreciation and amortisation for the period - Investment property","653","expense_depreciation","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_661","Provisions for the period - Outstanding legal matters","661","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_662","Provisions for the period - Accidents at work and occupational diseases","662","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_663","Provisions for the period - Taxes","663","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_664","Provisions for the period - Business restructuring","664","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_665","Provisions for the period - Onerous contracts","665","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_666","Provisions for the period - Warranty obligations","666","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_667","Provisions for the period - Losses on construction contracts","667","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_669","Provisions for the period - Other","669","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_681","Research expenses","681","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6821","Custom duties","6821","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6822","Value added tax","6822","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6823","Stamp duty","6823","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6824","Taxes on vehicles","6824","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6825","Municipal taxes","6825","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6831","Disposals","6831","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6832","Retirements","6832","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6833","claims on capital investments","6833","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6841","claims on inventories and biological assets","6841","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6842","Breaks","6842","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6849","Other Losses in inventories and biological assets","6849","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6891","Contributions","6891","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6892","Confidential expenses","6892","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6893","Gifts and inventory samples","6893","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6894","Social responsibility programs","6894","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_68951","Donations to the State","68951","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_68952","Donations - Other patronage","68952","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6896","Fines and penalties","6896","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6899","Other operating expenses","6899","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6911","Bank loans expenses","6911","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6913","Expenses on Loans from partners, shareholders or owners","6913","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6914","Expenses on Other borrowing","6914","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6915","Securities discount","6915","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_69161","Interest of default payments","69161","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_69162","Compensatory interest","69162","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6919","Other interest","6919","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6941","Foreign exchange losses - Realised","6941","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6942","Foreign exchange losses - Unrealised","6942","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_695","Cash discounts granted","695","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6981","Bank charges","6981","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_6989","Other finance costs and losses","6989","expense","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_711","Sales - Goods","711","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_712","Sales - Finished and intermediate goods","712","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_713","Sales - By-products, waste and scrap","713","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_714","Sales - Biological assets","714","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_715","Sales - VAT from sales with tax included","715","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_716","Sales - Return of goods sold","716","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_717","Sales - Discounts and rebates","717","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_721","Services rendered","721","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_722","VAT from services rendered with tax included","722","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_726","Discounts and rebates Services","726","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_731","Work performed by the entity and capitalised - financial investments","731","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_732","Work performed by the entity and capitalised - Tangible assets","732","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_733","Work performed by the entity and capitalised - Intangible assets","733","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_734","Work performed by the entity and capitalised - Assets under construction","734","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7411","Reversals for the period from adjustments of inventories to net realisable value","7411","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7412","Reversals for the period from adjustments of Financial investments","7412","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7413","Reversals for the period from adjustments of Tangible assets","7413","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7414","Reversals for the period from adjustments of Receivables","7414","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7421","Reversals from depreciation - Tangible assets","7421","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7422","Reversals from depreciation - Intangible assets","7422","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7423","Reversals from depreciation - Investment property","7423","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7431","Reversals from provisions - Outstanding legal matters","7431","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7432","Reversals from provisions - Accidents at work and occupational diseases","7432","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7433","Reversals from provisions - Taxes","7433","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7434","Reversals from provisions - Business restructuring","7434","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7435","Reversals from provisions - Onerous contracts","7435","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7436","Reversals from provisions - Warranty obligations","7436","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7437","Reversals from provisions - Losses on construction contracts","7437","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7439","Reversals from provisions - Other provisions","7439","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_75","Supplementary income","75","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7611","Grants for investments from State and other public entities","7611","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7619","Grants for investments from other entities","7619","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7621","Operating grants from State and other public entities","7621","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7629","Operating grants from other entities","7629","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7631","Profit on disposals from capital investments","7631","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7632","Profit on claims from capital investments","7632","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7641","Profit on claims in inventories and biological assets","7641","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7642","Profit on scrap","7642","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7649","Other gains in inventories and biological assets ","7649","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7691","Tax refund","7691","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7692","Benefits from contractual penalties","7692","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7693","Excess of tax estimate","7693","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7699","Other income","7699","income_other","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7811","Interest received - Bank deposits","7811","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7812","Interest received - Loans","7812","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7814","Other cash investments","7814","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7819","Other interest","7819","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_782","Income from investment property","782","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_783","Income from financial investments","783","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7841","Foreign exchange gains - Realised","7841","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_7842","Foreign exchange gains - Unrealised","7842","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_785","Discounts on cash payments","785","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_789","Other financial income and gains","789","income","l10n_mz.l10n_mz_chart_template","False"
"l10n_mz_account_85","Income tax","85","expense","l10n_mz.l10n_mz_chart_template","False"

```

## File: data\account.group.template.csv

```csv
id,code_prefix_start,code_prefix_end,name,chart_template_id/id
"l10n_mz_account_group_1",1,," Financial resources","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_2",2,,"Inventories and biological assets","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_21",21,,"Purchased Inventories","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_212",212,,"Purchased Raw materials and other supplies","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_2123",2123,,"Other Purchased materials","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_22",22,,"Goods","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_23",23,,"Finished and intermediate goods","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_24",24,,"By-products, waste and scrap","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_26",26,,"Raw materials and other supplies","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_263",263,,"Other materials","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_27",27,,"Biological assets","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_271",271,,"Biological assets for production","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_272",272,,"Consumable biological assets","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_28",28,,"Inventory adjustments","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_29",29,,"Net realisable value adjustments","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_3",3,,"Capital investments","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_31",31,,"Financial investments","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_32", 32,,"Tangible assets","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_321",321,,"Buildings","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_33",33,,"Intangible assets","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_34",34,,"Assets under construction","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_38",38,,"Accumulated depreciation and amortisation","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_4",4,,"Accounts receivable, accounts payable, accruals, and deferrals","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_41",41,,"Trade receivables","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_42",42,,"Trade payables","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_43",43,,"Borrowings","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_431",431,,"Bank loans","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_44",44,,"State","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_441",441,,"Income tax","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_442",442,,"Withholding tax","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_443",443,,"Value added tax","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_4431",4431,,"Input VAT","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_4432",4432,,"Deductible VAT","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_4433",4433,,"Assessed VAT","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_4434",4434,,"VAT adjustments","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_444",444,,"Remaining taxes","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_45",45,,"Other receivables","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_451",451,,"Employees receivables","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_452",452,,"Subscribers of Capital","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_454",454,,"Debtors – partners, shareholders or owners","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_455",455,,"Grants receivable","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_46",46,,"Other payables","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_461",461,,"Capital expenditure creditors","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_462",462,,"Employees payables","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_467",467,,"Creditors – partners, shareholders or owners","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_47",47,,"Accounts receivable adjustments","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_48",48,,"Provisions","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_49",49,,"Accruals and deferrals","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_491",491,,"Accrued expenses","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_492",492,,"Deferred income","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_493",493,,"Accrued income","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_5",5,,"Equity","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_52",52,,"Treasury shares","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_55",55,,"Reserves","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_56",56,,"Surplus on revaluation of tangible and intangible assets","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_6",6,,"Expenses and losses","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_61",61,,"Cost of inventories","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_611",611,,"Cost of inventories sold or consumed","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_6116",6116,,"Raw materials and other supplies","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_612",612,,"Change in production","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_62",62,,"Staff expenses","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_625",625,,"Allowances","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_626",626,,"Indemnities","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_63",63,,"Purchased supplies and services","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_632",632,,"Supplies of goods and services","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_63227",63227,,"Advertising","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_63228",63228,,"Travel and accommodation","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_63232",63232,,"Hire and rental charges","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_63233",63233,,"Insurance","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_64",64,,"Adjustments for the period","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_644",644,,"Receivables","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_65",65,,"Depreciation and amortisation for the period","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_66",66,,"Provisions for the period","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_68",68,,"Other operating expenses and losses","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_682",682,,"Taxes and levies","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_683",683,,"Losses in capital investments","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_684",684,,"Losses in inventories and biological assets","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_689",689,,"Other operating expenses","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_6895",6895,,"Donations","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_69",69,,"Finance costs and losses","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_691",691,,"Interest expenses","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_6916",6916,,"Interest of default payments and compensatory","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_694",694,,"Foreign exchange losses","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_698",698,,"Other finance costs and losses","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_7",7,,"Revenue and gains","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_71",71,,"Sales","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_72",72,,"Services rendered","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_73",73,,"Work performed by the entity and capitalised","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_74",74,,"Reversals for the period","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_741",741,,"Reversals From adjustments","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_742",742,,"Reversals From depreciation and amortisation","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_743",743,,"Reversals From provisions","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_76",76,,"Other operating income and gains","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_761",761,,"Grants for investments","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_762",762,,"Operating grants","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_763",763,,"Gains from capital investments","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_764",764,,"Gains in inventories and biological assets","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_769",769,,"Other income non-related to value added","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_78",78,,"Finance income and gains","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_781",781,,"Interest received","l10n_mz.l10n_mz_chart_template"
"l10n_mz_account_group_784",784,,"Foreign exchange gains","l10n_mz.l10n_mz_chart_template"

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_mz.l10n_mz_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="l10n_mz_chart_template" model="account.chart.template">
            <field name="name">Chart of accounts - Small and medium-sized enterprises</field>
            <field name="code_digits">7</field>
            <field name="bank_account_code_prefix">12</field>
            <field name="cash_account_code_prefix">11</field>
            <field name="transfer_account_code_prefix">456</field>
            <field name="currency_id" ref="base.MZN"/>
            <field name="country_id" ref="base.mz"/>
        </record>
    </data>
</odoo>

```

## File: data\account_fiscal_position_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="fiscal_position_template_dom" model="account.fiscal.position.template">
            <field name="name">Domestic regime</field>
            <field name="chart_template_id" ref="l10n_mz_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="country_id" ref="base.mz"/>
        </record>

        <record id="fiscal_position_import_export" model="account.fiscal.position.template">
            <field name="sequence">1</field>
            <field name="name">Import/Export</field>
            <field name="chart_template_id" ref="l10n_mz_chart_template"/>
            <field name="auto_apply" eval="True"/>
        </record>

        <record id="fiscal_position_sale_16_vat" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_import_export"/>
            <field name="tax_src_id" ref="vat_sale_16"/>
            <field name="tax_dest_id" ref="vat_export"/>
        </record>

        <record id="fiscal_position_purchase_inventories_16_vat" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_import_export"/>
            <field name="tax_src_id" ref="vat_purch_16_inventories"/>
            <field name="tax_dest_id" ref="vat_exempt_import"/>
        </record>

        <record id="fiscal_position_purchase_fixed_16_vat" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_import_export"/>
            <field name="tax_src_id" ref="vat_purch_16_fixed"/>
            <field name="tax_dest_id" ref="vat_exempt_import"/>
        </record>

        <record id="fiscal_position_purchase_other_16_vat" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_import_export"/>
            <field name="tax_src_id" ref="vat_purch_16_other"/>
            <field name="tax_dest_id" ref="vat_exempt_import"/>
        </record>

        <record id="fiscal_position_sale_5_vat" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_import_export"/>
            <field name="tax_src_id" ref="vat_sale_5"/>
            <field name="tax_dest_id" ref="vat_export"/>
        </record>

        <record id="fiscal_position_purchase_5_vat" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_import_export"/>
            <field name="tax_src_id" ref="vat_purchase_5"/>
            <field name="tax_dest_id" ref="vat_exempt_import"/>
        </record>
    </data>
</odoo>

```

## File: data\account_pgcpe_mozambique.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="l10n_mz_chart_template" model="account.chart.template">
            <field name="name">General Chart of Accounts = Small and medium-sized enterprises</field>
            <field name="code_digits">7</field>
            <field name="property_account_receivable_id" ref="l10n_mz_account_411"/>
            <field name="property_account_payable_id" ref="l10n_mz_account_421"/>
            <field name="property_account_expense_categ_id" ref="l10n_mz_account_61161"/>
            <field name="property_account_income_categ_id" ref="l10n_mz_account_711"/>
            <field name="expense_currency_exchange_account_id" ref="l10n_mz_account_6941"/>
            <field name="income_currency_exchange_account_id" ref="l10n_mz_account_7841"/>
            <field name="property_tax_payable_account_id" ref="l10n_mz_account_4437"/>
            <field name="property_tax_receivable_account_id" ref="l10n_mz_account_4438"/>
            <field name="default_pos_receivable_account_id" ref="l10n_mz_account_413"/>
            <field name="account_journal_early_pay_discount_loss_account_id" ref="l10n_mz_account_695"/>
            <field name="account_journal_early_pay_discount_gain_account_id" ref="l10n_mz_account_785"/>
        </record>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_vat_0" model="account.tax.group">
            <field name="name">VAT 0%</field>
            <field name="country_id" ref="base.mz"/>
        </record>
        <record id="tax_group_vat_5" model="account.tax.group">
            <field name="name">VAT 5%</field>
            <field name="country_id" ref="base.mz"/>
        </record>
        <record id="tax_group_vat_16" model="account.tax.group">
            <field name="name">VAT 16%</field>
            <field name="country_id" ref="base.mz"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record model="account.tax.template" id="vat_sale_16">
        <field name="name">16%</field>
        <field name="description">16%</field>
        <field name="amount" eval="16"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_mz_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_16"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('l10n_mz_tax_report_1_tag')],
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_mz_account_44331'),
                    'plus_report_expression_ids': [ref('l10n_mz_tax_report_2_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('l10n_mz_tax_report_1_tag')],
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_mz_account_44331'),
                    'minus_report_expression_ids': [ref('l10n_mz_tax_report_2_tag')],
                }),
            ]"/>
    </record>
    <record model="account.tax.template" id="vat_sale_5">
        <field name="name">5% S</field>
        <field name="description">5%</field>
        <field name="amount" eval="5"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_mz_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('l10n_mz_tax_report_1_tag')],
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_mz_account_44331'),
                    'plus_report_expression_ids': [ref('l10n_mz_tax_report_2_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('l10n_mz_tax_report_1_tag')],
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_mz_account_44331'),
                    'minus_report_expression_ids': [ref('l10n_mz_tax_report_2_tag')],
                }),
            ]"/>
    </record>
    <record model="account.tax.template" id="vat_export">
        <field name="name">0% EX</field>
        <field name="description">0%</field>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_mz_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('l10n_mz_tax_report_3_tag')],
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
                    'minus_report_expression_ids': [ref('l10n_mz_tax_report_3_tag')],
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>
    <record model="account.tax.template" id="vat_exempt_sale">
        <field name="name">0%</field>
        <field name="description">0%</field>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_mz_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('l10n_mz_tax_report_4_tag')],
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
                    'minus_report_expression_ids': [ref('l10n_mz_tax_report_4_tag')],
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
    </record>
    <record model="account.tax.template" id="vat_purch_16_fixed">
        <field name="name">16% G Fixed Assets</field>
        <field name="description">16%</field>
        <field name="amount" eval="16"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_mz_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_16"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_mz_account_44322'),
                    'plus_report_expression_ids': [ref('l10n_mz_tax_report_5_tag')],
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
                    'account_id': ref('l10n_mz_account_44322'),
                    'minus_report_expression_ids': [ref('l10n_mz_tax_report_5_tag')],
                }),	
            ]"/>
    </record>
    <record model="account.tax.template" id="vat_purch_16_inventories">
        <field name="name">16% G inventories</field>
        <field name="description">16%</field>
        <field name="amount" eval="16"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_mz_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_16"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_mz_account_44321'),
                    'plus_report_expression_ids': [ref('l10n_mz_tax_report_6_tag')],
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
                    'account_id': ref('l10n_mz_account_44321'),
                    'minus_report_expression_ids': [ref('l10n_mz_tax_report_6_tag')],
                }),	
            ]"/>
    </record>
    <record model="account.tax.template" id="vat_purch_16_other">
        <field name="name">16% GS</field>
        <field name="description">16%</field>
        <field name="amount" eval="16"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_mz_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_16"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_mz_account_44323'),
                    'plus_report_expression_ids': [ref('l10n_mz_tax_report_7_tag')],
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
                    'account_id': ref('l10n_mz_account_44323'),
                    'minus_report_expression_ids': [ref('l10n_mz_tax_report_7_tag')],
                }),	
            ]"/>
    </record>
    <record model="account.tax.template" id="vat_import">
        <field name="name">16% Import</field>
        <field name="description">16%</field>
        <field name="amount" eval="16"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_mz_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_16"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_mz_account_44323'),
                    'plus_report_expression_ids': [ref('l10n_mz_tax_report_8_tag')],
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
                    'account_id': ref('l10n_mz_account_44323'),
                    'minus_report_expression_ids': [ref('l10n_mz_tax_report_8_tag')],
                }),	
            ]"/>
    </record>
    <record model="account.tax.template" id="vat_purchase_5">
        <field name="name">5% S</field>
        <field name="description">5%</field>
        <field name="amount" eval="5"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_mz_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_mz_account_44323'),
                    'plus_report_expression_ids': [ref('l10n_mz_tax_report_7_tag')],
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
                    'account_id': ref('l10n_mz_account_44323'),
                    'minus_report_expression_ids': [ref('l10n_mz_tax_report_7_tag')],
                }),	
            ]"/>
    </record>
    <record model="account.tax.template" id="vat_exempt_import">
        <field name="name">0% Import</field>
        <field name="description">0%</field>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_mz_chart_template"/>
        <field name="type_tax_use">purchase</field>
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
    <record model="account.tax.template" id="vat_exempt_purchase">
        <field name="name">0%</field>
        <field name="description">0%</field>
        <field name="amount" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_mz_chart_template"/>
        <field name="type_tax_use">purchase</field>
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
</odoo>

```

## File: data\tax_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_mz_tax_report" model="account.report">
        <field name="name">Tax report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.mz"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="l10n_mz_tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="l10n_mz_tax_report_frame_5" model="account.report.line">
                <field name="name">Frame 5 - CALCULATION OF THE TAX REGARDING THE PERIOD TO WHICH THE DECLARATION REFERS</field>
                <field name="children_ids">
                    <record id="l10n_mz_tax_report_line_1" model="account.report.line">
                        <field name="name">1. Transfer of goods and/or rendering of services carried out by the taxable person and respective tax assessed</field>
                        <field name="children_ids">
                            <record id="l10n_mz_tax_report_field_1" model="account.report.line">
                                <field name="name">Field 1 - Transfer of goods and/or rendering of services carried out by the taxable person</field>
                                <field name="code">MZ_TR_F_1</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_mz_tax_report_field_2" model="account.report.line">
                                <field name="name">Field 2 - Tax assessed</field>
                                <field name="code">MZ_TR_F_2</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_exempt" model="account.report.line">
                        <field name="name">Exempt</field>
                        <field name="children_ids">
                            <record id="l10n_mz_tax_report_field_3" model="account.report.line">
                                <field name="name">Field 3 -  Operations of paragraph 1, subparagraph b, of Article 18 of the VAT law</field>
                                <field name="code">MZ_TR_F_3</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_mz_tax_report_field_4" model="account.report.line">
                                <field name="name">Field 4 - Operations that do not confer the right to deduction</field>
                                <field name="code">MZ_TR_F_4</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">4</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_line_2" model="account.report.line">
                        <field name="name">Deductible tax relating to transfers of goods and provision of services made to the declarant taxable person</field>
                        <field name="children_ids">
                            <record id="l10n_mz_tax_report_field_5" model="account.report.line">
                                <field name="name">Field 5 - Deductible tax on tangible and intangible assets</field>
                                <field name="code">MZ_TR_F_5</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_mz_tax_report_field_6" model="account.report.line">
                                <field name="name">Field 6 - Deductible tax on Inventories</field>
                                <field name="code">MZ_TR_F_6</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_6_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">6</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_mz_tax_report_field_7" model="account.report.line">
                                <field name="name">Field 7 - Deductible tax on Other goods and services</field>
                                <field name="code">MZ_TR_F_7</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">7</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_line_3" model="account.report.line">
                        <field name="name">3. Deductible tax incurred on imports of goods carried out by the taxable person (Field 08)</field>
                        <field name="code">MZ_TR_F_8</field>
                        <field name="expression_ids">
                            <record id="l10n_mz_tax_report_8_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">8</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_line_4" model="account.report.line">
                        <field name="name">4. Monthly or annual adjustments, with the exception of those communicated by the Tax Administration</field>
                        <field name="children_ids">
                            <record id="l10n_mz_tax_report_field_9" model="account.report.line">
                                <field name="name">Field 9 - Adjustments in favor of taxable person</field>
                                <field name="code">MZ_TR_F_9</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_9_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">9</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_mz_tax_report_field_10" model="account.report.line">
                                <field name="name">Field 10 - Adjustments in favor of the state</field>
                                <field name="code">MZ_TR_F_10</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">10</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_line_sum" model="account.report.line">
                        <field name="name">Settlement Amount</field>
                        <field name="children_ids">
                            <record id="l10n_mz_tax_report_field_11" model="account.report.line">
                                <field name="name">Field 11 - Tax base Total</field>
                                <field name="aggregation_formula">MZ_TR_F_1.balance + MZ_TR_F_3.balance + MZ_TR_F_4.balance</field>
                            </record>
                            <record id="l10n_mz_tax_report_field_12" model="account.report.line">
                                <field name="name">Field 12 - Total tax in favor of the taxable person</field>
                                <field name="code">MZ_TR_F_12</field>
                                <field name="aggregation_formula">MZ_TR_F_5.balance + MZ_TR_F_6.balance + MZ_TR_F_7.balance + MZ_TR_F_8.balance + MZ_TR_F_9.balance</field>
                            </record>
                            <record id="l10n_mz_tax_report_field_13" model="account.report.line">
                                <field name="name">Field 13 - Total tax in favor of the state</field>
                                <field name="code">MZ_TR_F_13</field>
                                <field name="aggregation_formula">MZ_TR_F_2.balance + MZ_TR_F_10.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_line_final" model="account.report.line">
                        <field name="name">Value before the use of the excess to be reported and other credits relating to previous periods</field>
                        <field name="children_ids">
                            <record id="l10n_mz_tax_report_field_14" model="account.report.line">
                                <field name="name">Field 14 - If the value entered in field 13 is greater than that in field 12, enter the difference in field 14 (13-12)</field>
                                <field name="code">MZ_TR_F_14</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_14_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">MZ_TR_F_13.balance - MZ_TR_F_12.balance</field>
                                        <field name="subformula">if_above(MZN(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_mz_tax_report_field_15" model="account.report.line">
                                <field name="name">Field 15 - If the value entered in field 12 is greater than that in field 13, enter the difference in field 15 (12-13)</field>
                                <field name="code">MZ_TR_F_15</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_15_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">MZ_TR_F_12.balance - MZ_TR_F_13.balance</field>
                                        <field name="subformula">if_above(MZN(0))</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_line_credits" model="account.report.line">
                        <field name="name"> Use of credits from previous periods. Important: values ​​can only be entered in fields 16 and 17 if this declaration is presented within the legal deadline.</field>
                        <field name="children_ids">
                            <record id="l10n_mz_tax_report_field_16" model="account.report.line">
                                <field name="name">Field 16 - Excess to be reported from the previous period</field>
                                <field name="code">MZ_TR_F_16</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_16_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                            <record id="l10n_mz_tax_report_field_17" model="account.report.line">
                                <field name="name">Field 17 - Credits communicated by the services</field>
                                <field name="code">MZ_TR_F_17</field>
                                <field name="expression_ids">
                                    <record id="l10n_mz_tax_report_17_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=0</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="l10n_mz_tax_report_frame_6" model="account.report.line">
                <field name="name">Frame 6 - Tax to be delivered to the state (This table should only be completed if field 14 of table 05 has been completed)</field>
                <field name="children_ids">
                    <record id="l10n_mz_tax_report_field_18" model="account.report.line">
                        <field name="name">Field 18 - Payable VAT</field>
                        <field name="code">MZ_TR_F_18</field>
                        <field name="expression_ids">
                            <record id="l10n_mz_tax_report_18_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">MZ_TR_F_14.balance - MZ_TR_F_16.balance - MZ_TR_F_17.balance</field>
                                <field name="subformula">if_above(MZN(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_field_19" model="account.report.line">
                        <field name="name">Field 19 - Late payment fine</field>
                        <field name="code">MZ_TR_F_19</field>
                        <field name="expression_ids">
                            <record id="l10n_mz_tax_report_19_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=0</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_field_20" model="account.report.line">
                        <field name="name">Field 20 - Total payable</field>
                        <field name="code">MZ_TR_F_20</field>
                        <field name="aggregation_formula">MZ_TR_F_18.balance + MZ_TR_F_19.balance</field>
                    </record>
                </field>
            </record>
            <record id="l10n_mz_tax_report_frame_7" model="account.report.line">
                <field name="name">Frame 7 - Tax to be recovered by taxable person</field>
                <field name="children_ids">
                    <record id="l10n_mz_tax_report_field_21" model="account.report.line">
                        <field name="name">Field 21 - Tax credit</field>
                        <field name="code">MZ_TR_F_21</field>
                        <field name="expression_ids">
                            <record id="l10n_mz_tax_report_21_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">MZ_TR_F_15.balance + MZ_TR_F_16.balance + MZ_TR_F_17.balance - MZ_TR_F_14.balance</field>
                                <field name="subformula">if_above(MZN(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_field_22" model="account.report.line">
                        <field name="name">Field 22 - Report for the following period (If this declaration is submitted within the deadline)</field>
                        <field name="code">MZ_TR_F_22</field>
                        <field name="expression_ids">
                            <record id="l10n_mz_tax_report_22_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=0</field>
                            </record>
                        </field>
                    </record>
                    <record id="l10n_mz_tax_report_field_23" model="account.report.line">
                        <field name="name">Field 23 - Refund request (If this declaration is submitted after the deadline)</field>
                        <field name="code">MZ_TR_F_23</field>
                        <field name="expression_ids">
                            <record id="l10n_mz_tax_report_23_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=0</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
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
    <mask id="b" x="4.15" y="0.91" width="48.45" height="45.52" maskUnits="userSpaceOnUse">
      <rect x="4.15" y="7.48" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
    <use width="106" height="106" xlink:href="#c"/>
    <rect x="4.15" y="10.57" width="48.45" height="31.57" rx="1" style="fill: #393939;opacity: 0.44;isolation: isolate"/>
    <g style="mask: url(#b)">
      <image width="300" height="200" transform="translate(6.5 7.5) scale(0.145 0.145)" xlink:href="data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI5MDAiIGhlaWdodD0iNjAwIiB4bWxuczp2PSJodHRwczovL3ZlY3RhLmlvL25hbm8iPg0KCTxwYXRoIGQ9Ik0wIDBoOTAwdjYwMEgweiIgLz4NCgk8cGF0aCBmaWxsPSIjZmZmIiBkPSJNMCAwaDkwMHYyMDYuMjVIMHoiIC8+DQoJPHBhdGggZmlsbD0iIzAwNzE2OCIgZD0iTTAgMGg5MDB2MTg3LjVIMHoiIC8+DQoJPHBhdGggZmlsbD0iI2ZmZiIgZD0iTTAgMzkzLjVoOTAwVjYwMEgweiIgLz4NCgk8cGF0aCBmaWxsPSIjZmNlMTAwIiBkPSJNMCA0MTIuNWg5MDBWNjAwSDB6IiAvPg0KCTxwYXRoIGZpbGw9IiNkMjEwMzQiIGQ9Ik0wIDB2NjAwbDM5My43NS0zMDB6IiAvPg0KCTxwYXRoIGZpbGw9IiNmY2UxMDAiDQoJCWQ9Im0yMDQuMzcgNDEyLjQ4NS03My4xNTQtNTMuNTZMNTguMTYgNDEyLjVsMjguMzMyLTg2LjEyNC03My41MjgtNTIuOTI0IDkwLjY2NC4zMzIgMjcuNjEzLTg2LjI4NCAyNy43IDg2LjMzIDkwLjU5NC0uNDAzLTczLjU0NCA1My4wMjN6IiAvPg0KCTxnIHN0cm9rZT0iIzAwMCIgc3Ryb2tlLWxpbmVqb2luPSJyb3VuZCIgc3Ryb2tlLXdpZHRoPSIxLjU3NSI+DQoJCTxwYXRoIGZpbGw9IiNmZmYiIGZpbGwtcnVsZT0iZXZlbm9kZCINCgkJCWQ9Ik02Ny43MjQgMzUyLjQyN2g1Mi42OTljNC4yNCA0Ljc1NyAxMy43IDYuNjk3IDIyLjcxMS0uMDU2IDE2LjUxOC05LjA0NSA0OC40NzMuMDU2IDQ4LjQ3My4wNTZsNi4yOTItNi42ODUtMTUuMzM4LTUwLjM0LTUuNTA2LTUuODk5cy0xMS43OTctNy4wNzgtMzQuMjE1LTQuNzE5LTMwLjI4My0uNzg2LTMwLjI4My0uNzg2LTE5LjY2MyAyLjM2LTI1LjE3IDUuMTEyYy0uNjA0LjQ5LTYuMjkyIDYuMjkyLTYuMjkyIDYuMjkyeiIgLz4NCgkJPGcgc3Ryb2tlLWxpbmVjYXA9InJvdW5kIj4NCgkJCTxwYXRoIGZpbGw9Im5vbmUiDQoJCQkJZD0iTTc4LjM0MyAzMzkuNDVzNTAuMzM5LTYuMjkzIDY0Ljg5IDEyLjk3N2MtOC4yMTggNS42MjQtMTUuNDUzIDYuMDg3LTIzLjIwMy4zOTUgMS4yMzQtMi4wNTcgMTguMDktMTkuNjY1IDYwLjk1OC0xMy43NjUiIC8+DQoJCQk8cGF0aA0KCQkJCWQ9Im0xMzIuMjIxIDI4OS4xMS0uMzkzIDU1LjQ1M200NS4yMjctNTQuNjY3IDkuNDM5IDQ0LjA0OG0tOTguNTg3LTQ0Ljc0LTUuMjM4IDIyLjcxNiIgLz4NCgkJPC9nPg0KCQk8cGF0aCBmaWxsLXJ1bGU9ImV2ZW5vZGQiDQoJCQlkPSJNMzMuNDk3IDM1OC40NjVsMTIuMzkzIDE0LjUzMWMxLjQ1Ni44NjggMi43NDguODEzIDQuMDQxIDBsMTguMzY3LTIyLjA0IDcuNzE1LTkuNTUxYzEuMTk0LTEuNDE1IDEuNTc1LTIuOTkzIDEuNDY5LTQuNDA4bDE0Ljc0OC0xMy4xMDUgMy4xMzEuMzAxYy0xLjQyNS0uMzY5LTIuNDctMS4wNjMtMS4zNDktMi42MjVsMy4zMDYtMi41NzEgMi41NzEgMy4zMDUtNC4wNCA0Ljc3NmgtNC4wNGwtNy43MTUgNi45OCAzLjM3MiAyLjk4IDUuMDc3IDEzLjkxOCA2LjI0NS00LjQwOS00LjA0LTE0LjMyNSA4LjgxNi05LjU1Mi0zLjMwNy01LjE0MyAyLjIwNS0yLjkzOXMzMC41MTMgMTkuMjEgNDIuMjY4IDE0LjA2OGMuMzE4LjExNS43MS0xMy43LjcxLTEzLjdzLTMxLjU5LTMuMzA2LTMyLjMyNi05LjU1MSA2Ljk4LTYuOTggNi45OC02Ljk4bC0zLjMwNy00Ljc3NS43MzYtMi41NzIgNS41MSA2Ljk4IDEyLjQ5LTEwLjY1NCA3My40NjggODMuNzU1YzQuMDExLTEuNjI2IDQuODY4LTIuNjA3IDUuMTQ0LTYuNjEyLS4xMDQtLjEtNzItODIuNjUzLTcyLTgyLjY1M2w1LjUxLTUuODc2YzEuMDg2LTEuMjI1IDEuNDY4LTEuNzQ1IDEuNDctMy42NzVsOC40NDgtNy4zNDZjMi41NDEuODczIDQuMTYxIDIuMzk2IDUuNTEgNC40MDdsMjMuMjI4LTE5LjY4NWMuNjEyLjYxMiAyLjQ3MiAxLjIyNCAzLjczNC41MzZsMzguNDAyLTM2Ljg2MS00MS44NTQgMjkuNTYyLTEuNDY5LTEuMTAyYzAtMS4yMjUgMS41MTgtMS41MjcgMC0zLjY3My0xLjYyNi0xLjk1Mi00LjA0IDEuODM2LTQuNDA3IDEuODM2cy02LjA1OC0yLjAxLTcuMzA1LTQuNTU4bC0uNDEgNi43NjItMTAuNjUzIDkuOTE5LTguMDgxLS4zNjgtMTEuNzU2IDExLjM4OC0xLjQ2OSA0LjQwOCAxLjgzNyAzLjY3NHMtNi4yNDYgNS41MS02LjI0NiA1LjE0Mi0xLjI2Mi0xLjYyMy0xLjMxNi0xLjc4Nmw1LjM1Ny00LjgyNi43MzUtMy4zMDYtMS43ODgtMi43OTFjLS41NDIuMzk0LTcuMzk2IDcuNTY3LTcuNzY0IDYuODMybC0xOS44MzUtMjIuNDA4IDEuMTAxLTQuMDQtMTIuNDg5LTEzLjU5M2MtNC41NTMtMS41NzItMTEuNzU1LTEuODM2LTEzLjIyNSA4LjA4Mi0xLjE0NCAyLjMzLTEwLjY1My4zNjctMTAuNjUzLjM2N2wtNS4xNDMgMS4xMDItMjkuMDIgNDEuMTQzIDE2LjE2MyAxOS40NjkgMzMuMDYxLTQxLjg3Ny45ODItMTEuODY0IDYuOTM3IDcuNzU3YzIuMzEzLjI5NyA0LjUxNi4zMjMgNi42MTItLjczNWwxOS41ODggMjEuODYzLTMuMjYyIDMuMTgzIDIuOTYzIDMuMjNjMS4xMDItLjczNCAyLjE1NC0xLjYxNSAzLjI1Ni0yLjM1MS4zNjguNDkxLjk4IDEuNDIyIDEuMzQ4IDEuOTEyLTEuNjQxLjg5My0yLjc5MyAyLjA4Mi00LjQzNCAyLjk3Ni0yLjYyNi0xLjcxMi01LjE2Mi0zLjg0NC00Ljk3LTcuMjM2bC0xMS4wMiA5LjE4My0uMzY3IDEuODM3LTMyLjY5NCAyNy4xODMtMi45MzkuMzY4LS43MzQgOC40NDkgMjEuMzA2LTE3LjYzMnYtMi41NzNsMi4yMDQgMS44MzcgMTYuNTMtMTMuMjIzczEuMTAyIDEuNDY5LjczNiAxLjQ2OS0xNC42OTUgMTMuMjI0LTE0LjY5NSAxMy4yMjRsLS4zNjcgMS40NjktMi41NzEgMi4yMDQtMS40Ny0xLjEwMi0xOS44MzcgMTcuNjMyaC0yLjkzOGwtMTEuMDIgMTEuMDIyYy0yLjg0My4yNDctNS4zMDYuNTQ4LTcuNzE1IDIuMjAzeiIgLz4NCgk8L2c+DQo8L3N2Zz4="/>
    </g>
  </g>
</svg>

```

