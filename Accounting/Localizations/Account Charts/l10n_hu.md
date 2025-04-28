# Odoo Module: l10n_hu

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, SUPERUSER_ID

def load_translations(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref('l10n_hu.hungarian_chart_template').process_coa_translations()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Hungary - Accounting',
    'version': '3.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
        Accounting chart and localization for Hungary
    """,
    'depends': [
        'account', 'base_vat', 'l10n_multilang'
    ],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/account.group.template.csv',
        'data/account_tax_group.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_fiscal_position_template.xml',
        'data/account_fiscal_position_tax_template.xml',
        'data/res.bank.csv',
        'data/l10n_hu_chart_data.xml',
        'data/account_chart_template_configure_data.xml',
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
"id","code","name","account_type","reconcile","chart_template_id/id"
"l10n_hu_111","111","Activation value of founding reorganization","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_112","112","Activated value for experimental development","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_113","113","Property rights","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_114","114","Intellectual products","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_115","115","Business or goodwill","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_117","117","Value adjustments in respect of intangible assets","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_118","118","Unscheduled amortization and reversal of intangible assets","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_119","119","Planned amortization of intangible assets","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_121","121","Land","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_122","122","Land, realization","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_123","123","Buildings, ownership shares","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_124","124","Other structures","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_125","125","Real estate and buildings outside the plant","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_126","126","Link to real estate for property rights","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_127","127","Real estate value adjustment","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_128","128","Unplanned depreciation of real estate and its reversal","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_129","129","Planned depreciation of real estate","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_131","131","Production machines, equipment, tools, production tools","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_132","132","Vehicles directly involved in production","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_137","137","Value adjustment of technical equipment, machines, vehicles","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_138","138","Unscheduled depreciation and reversal of technical equipment, machinery and vehicles","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_139","139","Depreciation of technical equipment, machinery and vehicles according to plan","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_141","141","Operating (business) machines, equipment, facilities","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_142","142","Other vehicles","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_143","143","Office, administrative and plant equipment","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_144","144","Off-site equipment, facilities, vehicles","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_145","145","Welfare equipment, fittings and works of art","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_147","147","Value adjustments of other plant, equipment and vehicles","asset_fixed","False","l10n_hu.hungarian_chart_template"
"l10n_hu_148","148","Unscheduled depreciation and reversal of other plant, equipment and vehicles","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_149","149","Depreciation of other plant, equipment and vehicles as planned","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_151","151","Breeding animals","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_152","152","Work animals","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_153","153","Other animals","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_157","157","Value adjustment of breeding animals","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_158","158","Unscheduled depreciation and reversal of breeding animals","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_159","159","Depreciation of breeding animals as planned","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_161","161","Work in progress","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_162","162","Renovations","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_168","168","Investments are unplanned depreciated","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_171","171","Long-term interest in an associate","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_172","172","Permanent significant ownership interest","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_173","173","Other permanent holdings","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_177","177","Value adjustments in respect of shares","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_178","178","Valuation difference on non-current holdings","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_179","179","Impairment and reversal of shares","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_181","181","Government bonds","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_182","182","Securities of affiliated enterprises","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_183","183","Securities of enterprises with a significant shareholding","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_184","184","Securities of other enterprises","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_185","185","Long-term discount securities","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_188","188","Valuation difference on debt securities","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_189","189","Impairment and reversal of securities","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_191","191","Resistant loans to affiliates","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_192","192","Long-term loans granted to other participating enterprises","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_193","193","Other long-term loans","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_194","194","Long-term bank deposits in affiliated enterprises","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_195","195","Long-term bank deposits in other participating enterprises","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_196","196","Other long-term bank deposits","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_197","197","Long-term receivables from finance leases","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_199","199","Impairment and reversal of long-term loans (and bank deposits)","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_211","211","Raw materials","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_221","221","Excipients","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_222","222","Fuels and fuels","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_223","223","Maintenance materials","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_224","224","Building materials","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_225","225","Depreciable assets within one year","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_226","226","Materials reclassified from property, plant and equipment","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_227","227","Other materials","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_228","228","Price difference of materials","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_229","229","Impairment and reversal of materials","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_231","231","Work in progress","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_235","235","Half-done products","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_238","238","Difference in inventories of semi-finished products","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_239","239","Work in progress and impairment of semi-finished products","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_241","241","Adult animals","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_242","242","Fattening animals","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_243","243","Other animals","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_246","246","Hired animals","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_248","248","Animal inventory value difference","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_249","249","Impairment and reversal of animals","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_251","251","Finished products","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_258","258","Difference in inventories of finished goods","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_259","259","Impairment and reversal of finished products","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_261","261","Goods at purchase price","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_262","262","Goods at settlement prices","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_263","263","Price difference of goods","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_264","264","Goods at selling price","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_265","265","Goods margin","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_266","266","Goods stored abroad and handed over to a commission","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_267","267","Goods reclassified from property, plant and equipment","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_269","269","Impairment and reversal of commercial goods","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_271","271","Intermediate services","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_279","279","Impairment and reversal of intermediary services","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_281","281","Own containers","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_282","282","Foreign packaging","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_288","288","Price difference for deposit packages","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_289","289","Impairment and reversal of deposit-bearing packages","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_311","311","Domestic receivables (in huf)","asset_receivable","True","l10n_hu.hungarian_chart_template"
"l10n_hu_312","312","Domestic receivables (in foreign currency)","asset_receivable","True","l10n_hu.hungarian_chart_template"
"l10n_hu_315","315","Impairment and reversal of domestic receivables","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_316","316","Foreign receivables (in huf)","asset_receivable","True","l10n_hu.hungarian_chart_template"
"l10n_hu_317","317","Foreign claims (in foreign currency)","asset_receivable","True","l10n_hu.hungarian_chart_template"
"l10n_hu_318","318","Valuation difference of receivables","asset_cash","False","l10n_hu.hungarian_chart_template"
"l10n_hu_319","319","Impairment and reversal of foreign receivables","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_321","321","Receivables from associates","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_322","322","Receivables from enterprises with significant ownership interest","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_323","323","Subscribed but not yet paid-up capital relationship from business","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_324","324","Subscribed but not yet paid-up capital from a company with a significant shareholding","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_325","325","Impairment and reversal of receivables from associates","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_326","326","Impairment and reversal of receivables from a significant ownership interest","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_331","331","Receivables from other participating interests","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_332","332","Subscribed but not yet paid-up capital from other participating undertakings","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_333","333","Subscribed but not yet paid-up capital from a non-participating company","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_339","339","Impairment and reversal of receivables from other participating interests","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_341","341","Domestic bills","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_345","345","Impairment and reversal of domestic accounts receivable","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_346","346","Foreign exchange receivables","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_349","349","Impairment and reversal of foreign exchange receivables","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_351","351","Advances on intangible assets","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_352","352","Advances on investments","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_353","353","Advances on stocks","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_354","354","Advances on services","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_355","355","Other advances given","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_359","359","Impairment and reversal of specific advances","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_361","361","Claims on employees","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3611","3611","Advances paid to employees","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3612","3612","Prescribed debts","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3613","3613","Other accounts with employees","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_362","362","Budget allocation requirements","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_363","363","Fulfillment of budget allocation needs","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_364","364","Short-term borrowed funds","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_365","365","Receivables purchased and received","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_366","366","Receivables related to shares and securities","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_367","367","Receivables related to futures, options and swaps","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_368","368","Miscellaneous other receivables","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3681","3681","Commission settlements","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3683","3683","Vat on export purchases","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3684","3684","Debtors","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3685","3685","Claims on insurance undertakings","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3686","3686","Barter transaction settlement account","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3687","3687","Exchange differences settlement account","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3688","3688","Intangible assets and tangible assets settlement account","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3689","3689","Other claims not raised","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_369","369","Impairment and reversal of other receivables","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_371","371","Participation in an associate","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3711","3711","Interests in associates purchased for sale","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3719","3719","Impairment and reversal of interests in associates","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_372","372","Significant ownership","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3721","3721","Significant ownership interests purchased for sale","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3729","3729","Impairment and reversal of significant ownership interests","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_373","373","Other shares","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3731","3731","Other shares purchased for sale","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3739","3739","Impairment and reversal of other interests","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_374","374","Own shares, own shares","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3741","3741","Repurchased own shares, business shares","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3749","3749","Impairment of own shares, own shares","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_375","375","Debt securities held for trading","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3751","3751","Debt securities purchased for sale","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_3759","3759","Impairment and reversal of negotiable debt securities","asset_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_378","378","Valuation difference on securities","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_379","379","Securities settlement account","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_381","381","Checkout","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_382","382","Currency office","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_383","383","Checks","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_384","384","Settlement deposit account","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_385","385","Separate deposit accounts","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_386","386","Foreign currency deposit account","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_389","389","Internal relocation","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_391","391","Accruals and deferrals","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_392","392","Accruals and deferrals of costs and expenses","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_393","393","Deferred expenses","asset_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_411","411","Registered capital","equity","False","l10n_hu.hungarian_chart_template"
"l10n_hu_412","412","Capital reserve","equity","False","l10n_hu.hungarian_chart_template"
"l10n_hu_413","413","Profit reserve","equity","False","l10n_hu.hungarian_chart_template"
"l10n_hu_414","414","Reserved reserve","equity","False","l10n_hu.hungarian_chart_template"
"l10n_hu_417","417","Valuation reserve","equity","False","l10n_hu.hungarian_chart_template"
"l10n_hu_4171","4171","Valuation reserve for value adjustments","equity","False","l10n_hu.hungarian_chart_template"
"l10n_hu_4172","4172","Fair value measurement reserve","equity","False","l10n_hu.hungarian_chart_template"
"l10n_hu_421","421","Provision for contingent liabilities","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_422","422","Provision for future expenses","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_429","429","Other provisions","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_431","431","Subordinated liabilities to an associate","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_432","432","Subordinated liabilities to an enterprise that has a significant ownership interest","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_433","433","Subordinated liabilities to other participating interests","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_434","434","Subordinated liabilities to other enterprise","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_441","441","Long-term loans","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_442","442","Convertible bonds","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_443","443","Debt from bond issue","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_444","444","Investment and development loans","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_445","445","Other long-term loans","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_446","446","Long-term liabilities to an associate","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_447","447","Long-term liabilities to a significant shareholder","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_448","448","Non-current liabilities to participating interests","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_449","449","Other long-term liabilities","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_4491","4491","Leases due to finance leases","liability_non_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_451","451","Short-term loans","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_4511","4511","Short-term loans from convertible and convertible bonds","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_452","452","Short-term loans","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_453","453","Advances received from customers","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_454","454","Suppliers","liability_payable","True","l10n_hu.hungarian_chart_template"
"l10n_hu_455","455","Investment suppliers","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_456","456","Accounts payable","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_457","457","Current liabilities to associates","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_458","458","Current liabilities to companies with significant ownership interests","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_459","459","Current liabilities to other participating interests","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_461","461","Corporate tax","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_462","462","Personal income tax accounting","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_463","463","Budget payment obligations","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_464","464","Fulfillment of budget payment obligations","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_465","465","Customs and import vat debts","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_466","466","Vat charged in advance","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_467","467","Value added tax payable","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_468","468","Financial accounting for value added tax","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_469","469","Local taxes settlement account","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_471","471","Subordinated liabilities to other enterprise","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_472","472","Unpaid allowances","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_473","473","Social security contribution obligation","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_474","474","Payment obligations related to segregated funds","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_475","475","Liabilities vis-à-vis asset management organizations","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_476","476","Other current liabilities to employees and members","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_4761","4761","Compensation","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_4762","4762","Judicial disqualification","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_4764","4764","Deducted union fee","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_4765","4765","Private pension fund contribution obligations","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_478","478","Liabilities related to shares and securities","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_479","479","Miscellaneous current liabilities","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_4791","4791","Liabilities to insurance institutions","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_4792","4792","Lenders","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_4796","4796","Valuation difference on liabilities","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_481","481","Accruals and deferrals","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_482","482","Accruals and deferrals of costs and expenses","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_483","483","Deferred income","liability_current","False","l10n_hu.hungarian_chart_template"
"l10n_hu_511","511","Cost of materials purchased","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5111","5111","Raw material costs","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5112","5112","Excipient costs","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5113","5113","Fuel costs","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5114","5114","Cost of consumables, plant, equipment and other assets that are depreciated within one year","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5115","5115","The cost of using work clothes that wear out within a year, protective clothing","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5116","5116","Costs of forms and office supplies","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5117","5117","Fuel costs","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5118","5118","Costs of electricity and water consumption","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5119","5119","Other material use costs","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_512","512","Costs of depreciable assets within one year","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_522","522","Rent cost","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_523","523","Shipping, handling, loading and storage costs","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_524","524","Repair, maintenance costs","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_525","525","Costs of publications on electronic media","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_526","526","Expenses for newspapers and books","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_527","527","Postage, telephone, internet and other telecommunications charges","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_528","528","Laundry, dry cleaning, cleaning costs","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_529","529","Costs of other services used","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5291","5291","Costs of photocopying and reproduction","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5292","5292","District heating costs","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5293","5293","Cost of warranty repairs performed with another company","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5294","5294","Organization of exhibitions, demonstrations, fairs","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5295","5295","Advertising, publicity, propaganda costs","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5296","5296","Expenditure on education and training","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_531","531","Official administration and service fees and charges","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_532","532","Financial and investment service fees","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5321","5321","Bank charges","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_533","533","Insurance fee","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_534","534","Taxes, contributions, product charges to be accounted for as costs","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_539","539","Miscellaneous other costs","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_541","541","Wage cost","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_542","542","Remuneration for the personal contribution of the owner","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_551","551","Personal payments made to employees and members","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5511","5511","Sick leave fee, sick pay charged to the employer, sick pay supplement","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5512","5512","Severance pay","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5513","5513","Reimbursement of other travel expenses","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5514","5514","Daily subsistence allowance","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5515","5515","Remuneration supplement for disabled workers, paid benefits","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5516","5516","Holiday allowance","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5517","5517","Housing subsidy, rent contribution","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5518","5518","Anniversary reward, object reward","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_552","552","Welfare and cultural costs","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_559","559","Other personal payments","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5591","5591","Accident, life and pension insurance premiums paid by the employer","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5592","5592","Employer membership fee contribution paid by an employer to a voluntary fund","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5593","5593","Personal income tax on employers","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5594","5594","Employers' contribution to early retirement","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5595","5595","Invention fee, patent purchase price and utilization fee","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5596","5596","Fees for royalties, author's and other copyrighted works and related contributions","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5597","5597","Scholarships paid","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5598","5598","Entertainment expenses, meal allowance","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_5599","5599","Insurance premiums for employees","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_561","561","Social contribution tax","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_564","564","Vocational training contribution","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_565","565","Rehabilitation contribution","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_571","571","Depreciation according to plan","expense_depreciation","False","l10n_hu.hungarian_chart_template"
"l10n_hu_572","572","Depreciation of small assets (below huf 200 thousand individual purchase value) according to plan","expense_depreciation","False","l10n_hu.hungarian_chart_template"
"l10n_hu_581","581","Change in stocks of own production","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_582","582","Capitalized value of self-produced assets","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_811","811","Material cost","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_812","812","Value of services used","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_813","813","Value of other services","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_814","814","Purchase value of goods sold","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_815","815","Value of services sold (brokered)","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_821","821","Wage cost","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_822","822","Other payments of a personal nature","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_823","823","Wage contributions","expense_direct_cost","False","l10n_hu.hungarian_chart_template"
"l10n_hu_851","851","Sales and distribution costs","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_852","852","Administrative expenses","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_853","853","Other overheads","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_861","861","Realized loss on sale of intangible assets and property, plant and equipment","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_862","862","Book value of receivables sold, transferred (assigned)","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_863","863","Expenses related to the financial year for events occurring before the balance sheet date","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8631","8631","Damage-related payments, amounts payable","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8632","8632","Fines, penalties, late fees, default interest, damages","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_864","864","Subsequent - indirectly related - financially settled discount","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_865","865","Provisioning","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_866","866","Impairment, unplanned depreciation","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8661","8661","Impairment of inventories","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8662","8662","Impairment of receivables","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8663","8663","Unscheduled amortization of intangible assets","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8664","8664","Depreciation of property, plant and equipment is unplanned","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_867","867","Taxes, fees, contributions","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_868","868","Expenses of exceptional size or occurrence","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8681","8681","The book value of assets brought into the company","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8682","8682","Carrying amount of a receivable that is uncollectible","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8683","8683","Contractual amount of debt assumption","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8684","8684","Financially supported aid transferred without obligation to repay","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8685","8685","Amount of aid granted for development purposes","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8686","8686","Book value of assets transferred free of charge","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8687","8687","Cost of services provided free of charge","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_869","869","Miscellaneous other expenses","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8691","8691","Amount written off for bad debt","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8692","8692","Book value of missing, destroyed intangible assets and property, plant and equipment","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8693","8693","Book value of missing, destroyed inventories derecognised","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8694","8694","Loss-weighted inventory difference for commercial goods","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_871","871","Expenses and exchange losses on shares","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8711","8711","Equity costs, exchange losses for associates","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_872","872","Expenses from fixed financial assets (securities, loans), exchange rate losses","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8721","8721","It is derived from the expenses of fixed financial assets (securities, loans) and from the exchange rate losses of affiliated companies.","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_873","873","Interest payable (interest paid) and interest - like expenses","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8731","8731","Interest payable (interest paid) and interest - like expenses to an associate","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_874","874","Impairment of shares, securities, bank deposits","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_875","875","Foreign exchange loss on investment, sale and redemption of investment in current assets","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_876","876","Exchange rate loss at conversion and valuation","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8761","8761","Exchange rate loss on the conversion of foreign exchange and currency inventories into huf","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8762","8762","Foreign exchange losses on assets and liabilities denominated in foreign currencies","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8763","8763","Aggregate foreign exchange loss on the measurement of assets and liabilities denominated in foreign currencies at the balance sheet date","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_877","877","Other exchange losses, option fees","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_878","878","Expenditure on purchased receivables","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_879","879","Other financial expenses","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_8791","8791","Valuation difference other financial expenses","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_891","891","Corporate tax","expense","False","l10n_hu.hungarian_chart_template"
"l10n_hu_911","911","Domestic sales revenue","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_931","931","Export sales revenue","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_961","961","Realized gains on the sale of intangible assets and property, plant and equipment","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_962","962","Recognized value of receivables sold, assigned (assigned)","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_963","963","Other income related to the financial year financially settled up to the balance sheet date","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9631","9631","Proceeds from claims","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9632","9632","Fines, fines, penalties, default interest, damages received","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9633","9633","Amounts received on receivables classified as irrecoverable and written off","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_964","964","Subsequent - indirectly related - financially settled discount","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_965","965","Use of provision (decrease, termination)","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_966","966","Reversal of impairment, unplanned depreciation","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9661","9661","Reversal of impairment, unplanned depreciation","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9662","9662","Reversal of impairment of receivables","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9663","9663","Unscheduled depreciation of intangible assets","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9664","9664","Unscheduled depreciation of property, plant and equipment","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_967","967","Aid received without obligation to repay","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_968","968","Revenues of exceptional size or occurrence","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9681","9681","The value of the assets brought into the company as specified in the memorandum of association","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9682","9682","Value of liabilities waived","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9683","9683","Liability assumed by a third party in the course of a debt assumption","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9684","9684","Funds received and received without obligation to repay","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9685","9685","Funds received definitively for development purposes without a repayment obligation","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9686","9686","Market value of assets received free of charge, received as gifts or bequests","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9687","9687","Market value of services received free of charge","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_969","969","Miscellaneous other revenue","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9691","9691","Amount of compensation confirmed by the insurer","income","False","l10n_hu.hungarian_chart_template"
"l10n_hu_971","971","Dividends and shares received (due)","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9711","9711","Dividends and interests received from associates","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_972","972","Foreign exchange gains on the sale of shares","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9721","9721","Foreign exchange gains on interests sold to associates","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_973","973","Interest and foreign exchange gains on financial assets","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9731","9731","Interest on fixed financial assets, foreign exchange gains from associates","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_974","974","Other interest receivable and similar income","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9741","9741","Other interest receivable and similar income from an associate","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_975","975","Foreign exchange gain on investment, sale and redemption of securities","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_976","976","Exchange rate gains on exchange and valuation","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9761","9761","Foreign exchange gains on the translation of foreign exchange and currency inventories into huf","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9762","9762","Foreign exchange gains and losses on assets and liabilities denominated in foreign currencies","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9763","9763","Aggregate foreign exchange gains on the measurement of assets and liabilities denominated in foreign currencies at the balance sheet date","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_977","977","Other foreign exchange gains, option fee income","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_978","978","Revenue from purchased receivables","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_979","979","Other financial income","income_other","False","l10n_hu.hungarian_chart_template"
"l10n_hu_9791","9791","Valuation difference on other financial income","income_other","False","l10n_hu.hungarian_chart_template"

```

## File: data\account.group.template.csv

```csv
"id","code_prefix_start","code_prefix_end","name","chart_template_id/id"
"l10n_hu_group_1","1","","Fixed assets","l10n_hu.hungarian_chart_template"
"l10n_hu_group_11","11","","Intangible assets","l10n_hu.hungarian_chart_template"
"l10n_hu_group_12","12","","Real estate and associated rights","l10n_hu.hungarian_chart_template"
"l10n_hu_group_13","13","","Technical equipment, machinery, vehicles","l10n_hu.hungarian_chart_template"
"l10n_hu_group_14","14","","Other equipment, equipment and vehicles","l10n_hu.hungarian_chart_template"
"l10n_hu_group_15","15","","Breeding animals","l10n_hu.hungarian_chart_template"
"l10n_hu_group_16","16","","Investments, renovations","l10n_hu.hungarian_chart_template"
"l10n_hu_group_17","17","","Ownership investments (shares)","l10n_hu.hungarian_chart_template"
"l10n_hu_group_18","18","","Debt securities","l10n_hu.hungarian_chart_template"
"l10n_hu_group_19","19","","Long-term loans","l10n_hu.hungarian_chart_template"
"l10n_hu_group_2","2","","Stocks","l10n_hu.hungarian_chart_template"
"l10n_hu_group_21-22","21","22","Materials","l10n_hu.hungarian_chart_template"
"l10n_hu_group_23","23","","Work in progress and semi-finished products","l10n_hu.hungarian_chart_template"
"l10n_hu_group_24","24","","Adult, fattening and other animals","l10n_hu.hungarian_chart_template"
"l10n_hu_group_25","25","","Finished products","l10n_hu.hungarian_chart_template"
"l10n_hu_group_26","26","","Commercial goods","l10n_hu.hungarian_chart_template"
"l10n_hu_group_27","27","","Intermediate services","l10n_hu.hungarian_chart_template"
"l10n_hu_group_28","28","","Deposit feeds","l10n_hu.hungarian_chart_template"
"l10n_hu_group_3","3","","Receivables, securities, cash and current accounts","l10n_hu.hungarian_chart_template"
"l10n_hu_group_31","31","","Receivables from supply of goods and services (customers)","l10n_hu.hungarian_chart_template"
"l10n_hu_group_32","32","","Claims against associated and significant ownership","l10n_hu.hungarian_chart_template"
"l10n_hu_group_33","33","","Claims against other participating undertakings","l10n_hu.hungarian_chart_template"
"l10n_hu_group_34","34","","Receivables","l10n_hu.hungarian_chart_template"
"l10n_hu_group_35","35","","Advances given","l10n_hu.hungarian_chart_template"
"l10n_hu_group_36","36","","Other claims","l10n_hu.hungarian_chart_template"
"l10n_hu_group_37","37","","Securities","l10n_hu.hungarian_chart_template"
"l10n_hu_group_38","38","","Funds","l10n_hu.hungarian_chart_template"
"l10n_hu_group_39","39","","Active definitions","l10n_hu.hungarian_chart_template"
"l10n_hu_group_4","4","","Sources","l10n_hu.hungarian_chart_template"
"l10n_hu_group_41","41","","Equity","l10n_hu.hungarian_chart_template"
"l10n_hu_group_42","42","","Provisions","l10n_hu.hungarian_chart_template"
"l10n_hu_group_43","43","","Subsequent liabilities","l10n_hu.hungarian_chart_template"
"l10n_hu_group_44","44","","Long-term liabilities","l10n_hu.hungarian_chart_template"
"l10n_hu_group_45-47","45","47","Short-term liabilities","l10n_hu.hungarian_chart_template"
"l10n_hu_group_48","48","","Liabilities","l10n_hu.hungarian_chart_template"
"l10n_hu_group_5","5","","Costs","l10n_hu.hungarian_chart_template"
"l10n_hu_group_51","51","","Material cost","l10n_hu.hungarian_chart_template"
"l10n_hu_group_52","52","","Costs of services used","l10n_hu.hungarian_chart_template"
"l10n_hu_group_53","53","","Costs of other services","l10n_hu.hungarian_chart_template"
"l10n_hu_group_54","54","","Wage cost","l10n_hu.hungarian_chart_template"
"l10n_hu_group_55","55","","Other personal payments","l10n_hu.hungarian_chart_template"
"l10n_hu_group_56","56","","Contributions","l10n_hu.hungarian_chart_template"
"l10n_hu_group_57","57","","Description of impairment","l10n_hu.hungarian_chart_template"
"l10n_hu_group_58","58","","Own work capitalized value","l10n_hu.hungarian_chart_template"
"l10n_hu_group_59","59","","Non-cost transfer account","l10n_hu.hungarian_chart_template"
"l10n_hu_group_6","6","","Costs, overall costs","l10n_hu.hungarian_chart_template"
"l10n_hu_group_61","61","","Costs of repair-maintenance plants","l10n_hu.hungarian_chart_template"
"l10n_hu_group_62","62","","Costs of establishments (units)","l10n_hu.hungarian_chart_template"
"l10n_hu_group_63","63","","Machinery costs","l10n_hu.hungarian_chart_template"
"l10n_hu_group_64-65","64","65","Overall costs of operating management","l10n_hu.hungarian_chart_template"
"l10n_hu_group_66","66","","Overall costs of central management","l10n_hu.hungarian_chart_template"
"l10n_hu_group_67","67","","Sales and distribution costs","l10n_hu.hungarian_chart_template"
"l10n_hu_group_68","68","","Allocated other overall costs","l10n_hu.hungarian_chart_template"
"l10n_hu_group_69","69","","Cost positions transfer of cost types","l10n_hu.hungarian_chart_template"
"l10n_hu_group_7","7","","Costs of activities","l10n_hu.hungarian_chart_template"
"l10n_hu_group_71-74","71","74","Costs of activities","l10n_hu.hungarian_chart_template"
"l10n_hu_group_75","75","","Costs of the service","l10n_hu.hungarian_chart_template"
"l10n_hu_group_76","76","","Production costs of centers","l10n_hu.hungarian_chart_template"
"l10n_hu_group_77-78","77","78","Marketing costs","l10n_hu.hungarian_chart_template"
"l10n_hu_group_79","79","","Transfer of costs of activities","l10n_hu.hungarian_chart_template"
"l10n_hu_group_8","8","","Accounted costs of sales and expenses","l10n_hu.hungarian_chart_template"
"l10n_hu_group_81","81","","Material expenses","l10n_hu.hungarian_chart_template"
"l10n_hu_group_82","82","","Personnel expenses","l10n_hu.hungarian_chart_template"
"l10n_hu_group_83","83","","Description of impairment","l10n_hu.hungarian_chart_template"
"l10n_hu_group_85","85","","Indirect costs of sale","l10n_hu.hungarian_chart_template"
"l10n_hu_group_86","86","","Other expenses","l10n_hu.hungarian_chart_template"
"l10n_hu_group_87","87","","Expenditure on financial operations","l10n_hu.hungarian_chart_template"
"l10n_hu_group_89","89","","Taxes on profit","l10n_hu.hungarian_chart_template"
"l10n_hu_group_9","9","","Sales revenue and revenue","l10n_hu.hungarian_chart_template"
"l10n_hu_group_91-92","91","92","Turnover of domestic sales","l10n_hu.hungarian_chart_template"
"l10n_hu_group_93-94","93","94","Export sales revenue","l10n_hu.hungarian_chart_template"
"l10n_hu_group_96","96","","Other revenues","l10n_hu.hungarian_chart_template"
"l10n_hu_group_97","97","","Revenue from financial operations","l10n_hu.hungarian_chart_template"
"l10n_hu_group_0","0","","Registration accounts","l10n_hu.hungarian_chart_template"

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_hu.hungarian_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hungarian_chart_template" model="account.chart.template">
        <field name="name">Hungary - National Chart of Accounts</field>
        <field name="cash_account_code_prefix">381</field>
        <field name="bank_account_code_prefix">384</field>
        <field name="transfer_account_code_prefix">389</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.HUF"/>
        <field name="country_id" ref="base.hu"/>
        <field name="spoken_languages" eval="'hu_HU'"/>
    </record>
</odoo>

```

## File: data\account_fiscal_position_tax_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Exempt taxpayer -->
        <record id="fiscal_position_hu_exempt_tax_F27" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_exempt"/>
            <field name="tax_src_id" ref="F27"/>
            <field name="tax_dest_id" ref="FA"/>
        </record>

        <record id="fiscal_position_hu_exempt_tax_F18" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_exempt"/>
            <field name="tax_src_id" ref="F18"/>
            <field name="tax_dest_id" ref="FA"/>
        </record>

        <record id="fiscal_position_hu_exempt_tax_F5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_exempt"/>
            <field name="tax_src_id" ref="F5"/>
            <field name="tax_dest_id" ref="FA"/>
        </record>

        <record id="fiscal_position_hu_exempt_tax_V27" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_exempt"/>
            <field name="tax_src_id" ref="V27"/>
            <field name="tax_dest_id" ref="VA"/>
        </record>

        <record id="fiscal_position_hu_exempt_tax_V27TE" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_exempt"/>
            <field name="tax_src_id" ref="V27TE"/>
            <field name="tax_dest_id" ref="VA"/>
        </record>

        <record id="fiscal_position_hu_exempt_tax_V18" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_exempt"/>
            <field name="tax_src_id" ref="V18"/>
            <field name="tax_dest_id" ref="VA"/>
        </record>

        <record id="fiscal_position_hu_exempt_tax_V5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_exempt"/>
            <field name="tax_src_id" ref="V5"/>
            <field name="tax_dest_id" ref="VA"/>
        </record>
        <!-- Exempt taxpayer [END] -->

        <!-- EU partner -->
        <record id="fiscal_position_hu_eu_F27" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_eu"/>
            <field name="tax_src_id" ref="F27"/>
            <field name="tax_dest_id" ref="FEUT"/>
        </record>

        <record id="fiscal_position_hu_eu_F18" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_eu"/>
            <field name="tax_src_id" ref="F18"/>
            <field name="tax_dest_id" ref="FEUT"/>
        </record>

        <record id="fiscal_position_hu_eu_F5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_eu"/>
            <field name="tax_src_id" ref="F5"/>
            <field name="tax_dest_id" ref="FEUT"/>
        </record>

        <record id="fiscal_position_hu_eu_V27" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_eu"/>
            <field name="tax_src_id" ref="V27"/>
            <field name="tax_dest_id" ref="VEU27T"/>
        </record>

        <record id="fiscal_position_hu_eu_V27TE" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_eu"/>
            <field name="tax_src_id" ref="V27TE"/>
            <field name="tax_dest_id" ref="VEU27TE"/>
        </record>
        <!-- EU partner [END] -->

        <!-- Partner outside the EU -->
        <record id="fiscal_position_hu_eu_out_F27" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_eu_out"/>
            <field name="tax_src_id" ref="F27"/>
            <field name="tax_dest_id" ref="FEXT"/>
        </record>

        <record id="fiscal_position_hu_eu_out_F18" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_eu_out"/>
            <field name="tax_src_id" ref="F18"/>
            <field name="tax_dest_id" ref="FEUT"/>
        </record>

        <record id="fiscal_position_hu_eu_out_F5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_eu_out"/>
            <field name="tax_src_id" ref="F5"/>
            <field name="tax_dest_id" ref="FEXT"/>
        </record>

        <record id="fiscal_position_hu_eu_out_V27" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_eu_out"/>
            <field name="tax_src_id" ref="V27"/>
            <field name="tax_dest_id" ref="VIMK"/>
        </record>

        <record id="fiscal_position_hu_eu_out_V27TE" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_hu_eu_out"/>
            <field name="tax_src_id" ref="V27TE"/>
            <field name="tax_dest_id" ref="VIMK"/>
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
        <record id="fiscal_position_hu_exempt" model="account.fiscal.position.template">
            <field name="name">Exempt taxpayer</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
        </record>

        <record id="fiscal_position_hu_national" model="account.fiscal.position.template">
            <field name="name">Domestic</field>
            <field name="sequence">10</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_id" ref="base.hu"/>
        </record>

        <record id="fiscal_position_hu_eu_private" model="account.fiscal.position.template">
            <field name="name">EU partner private</field>
            <field name="sequence">20</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>

        <record id="fiscal_position_hu_eu" model="account.fiscal.position.template">
            <field name="name">EU partner</field>
            <field name="sequence">30</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>

        <record id="fiscal_position_hu_eu_out" model="account.fiscal.position.template">
            <field name="name">Partner outside the EU</field>
            <field name="sequence">40</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="auto_apply" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_group.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="tax_group_afa_27" model="account.tax.group">
            <field name="name">27% VAT</field>
            <field name="country_id" ref="base.hu"/>
        </record>

        <record id="tax_group_afa_18" model="account.tax.group">
            <field name="name">18% VAT</field>
            <field name="country_id" ref="base.hu"/>
        </record>

        <record id="tax_group_afa_5" model="account.tax.group">
            <field name="name">5% VAT</field>
            <field name="country_id" ref="base.hu"/>
        </record>

        <record id="tax_group_afa_0" model="account.tax.group">
            <field name="name">0% VAT</field>
            <field name="country_id" ref="base.hu"/>
        </record>

        <record id="tax_group_afa_komp" model="account.tax.group">
            <field name="name">Compensation surcharge</field>
            <field name="country_id" ref="base.hu"/>
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
        <field name="country_id" ref="base.hu"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_alap" model="account.report.line">
                <field name="name">Tax base</field>
                <field name="aggregation_formula">BASE_PAYABLE.balance + BASE_RECOVERABLE.balance</field>
                <field name="children_ids">
                    <record id="tax_report_base_pay" model="account.report.line">
                        <field name="name">Payable</field>
                        <field name="code">BASE_PAYABLE</field>
                        <field name="aggregation_formula">BASE_PAY_EXPORTS.balance + BASE_PAY_INTRA.balance + BASE_PAY_EXEMPT_PROPERTY.balance + BASE_PAY_EXEMPT_TAX.balance + BASE_PAY_EXEMPT.balance + BASE_PAY_27.balance + BASE_PAY_18.balance + BASE_PAY_5.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_alap_fiz_export" model="account.report.line">
                                <field name="name">Exports</field>
                                <field name="code">BASE_PAY_EXPORTS</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_export_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_exports</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_fiz_eu" model="account.report.line">
                                <field name="name">Intra-community</field>
                                <field name="code">BASE_PAY_INTRA</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_eu_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_intra</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_fiz_targyi" model="account.report.line">
                                <field name="name">Exempt from property tax</field>
                                <field name="code">BASE_PAY_EXEMPT_PROPERTY</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_targyi_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_exempt_property</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_fiz_alanyi" model="account.report.line">
                                <field name="name">Exempt from tax</field>
                                <field name="code">BASE_PAY_EXEMPT_TAX</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_alanyi_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_exempt_tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_fiz_koron_kivuli" model="account.report.line">
                                <field name="name">Exempt</field>
                                <field name="code">BASE_PAY_EXEMPT</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_koron_kivuli_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_exempt</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_fiz_afa_27" model="account.report.line">
                                <field name="name">27% VAT</field>
                                <field name="code">BASE_PAY_27</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_afa_27_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_27</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_fiz_afa_18" model="account.report.line">
                                <field name="name">18% VAT</field>
                                <field name="code">BASE_PAY_18</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_afa_18_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_18</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_fiz_afa_5" model="account.report.line">
                                <field name="name">5% VAT</field>
                                <field name="code">BASE_PAY_5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_afa_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_5</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_base_rec" model="account.report.line">
                        <field name="name">Recoverable</field>
                        <field name="code">BASE_RECOVERABLE</field>
                        <field name="aggregation_formula">BASE_REC_IMPORT.balance + BASE_REC_INTRA.balance + BASE_REC_REVERSE.balance + BASE_REC_EXEMPT_MATERIAL.balance + BASE_REC_EXEMPT_TAX.balance + BASE_REC_EXEMPT.balance + BASE_REC_27.balance + BASE_REC_18.balance + BASE_REC_5.balance + BASE_REC_COMPENSATION.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_alap_import" model="account.report.line">
                                <field name="name">Import</field>
                                <field name="code">BASE_REC_IMPORT</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_import_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_import</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_viss" model="account.report.line">
                                <field name="name">Intra-community</field>
                                <field name="code">BASE_REC_INTRA</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_viss_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_intra</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_forditott" model="account.report.line">
                                <field name="name">Reverse</field>
                                <field name="code">BASE_REC_REVERSE</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_forditott_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_reverse</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_viss_targyi" model="account.report.line">
                                <field name="name">Exempt from material tax</field>
                                <field name="code">BASE_REC_EXEMPT_MATERIAL</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_viss_targyi_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_exempt_material</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_viss_alanyi" model="account.report.line">
                                <field name="name">Exempt from tax</field>
                                <field name="code">BASE_REC_EXEMPT_TAX</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_viss_alanyi_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_exempt_tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_viss_koron_kivuli" model="account.report.line">
                                <field name="name">Exempt</field>
                                <field name="code">BASE_REC_EXEMPT</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_viss_koron_kivuli_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_exempt</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_viss_27" model="account.report.line">
                                <field name="name">27% VAT</field>
                                <field name="code">BASE_REC_27</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_viss_27_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_27</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_viss_18" model="account.report.line">
                                <field name="name">18% VAT</field>
                                <field name="code">BASE_REC_18</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_viss_18_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_18</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_viss_5" model="account.report.line">
                                <field name="name">5% VAT</field>
                                <field name="code">BASE_REC_5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_viss_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_komp" model="account.report.line">
                                <field name="name">Compensation Surcharge</field>
                                <field name="code">BASE_REC_COMPENSATION</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_komp_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_compensation</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_fizetndo" model="account.report.line">
                <field name="name">VAT payable/recoverable</field>
                <field name="aggregation_formula">VAT_PAYABLE.balance + VAT_RECOVERABLE.balance</field>
                <field name="children_ids">
                    <record id="tax_report_vat_pay" model="account.report.line">
                        <field name="name">Payable</field>
                        <field name="code">VAT_PAYABLE</field>
                        <field name="aggregation_formula">VAT_PAY_27.balance + VAT_PAY_18.balance + VAT_PAY_5.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_fizetndo_27" model="account.report.line">
                                <field name="name">27% VAT</field>
                                <field name="code">VAT_PAY_27</field>
                                <field name="expression_ids">
                                    <record id="tax_report_fizetndo_27_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vat_pay_27</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_fizetndo_18" model="account.report.line">
                                <field name="name">18% VAT</field>
                                <field name="code">VAT_PAY_18</field>
                                <field name="expression_ids">
                                    <record id="tax_report_fizetndo_18_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vat_pay_18</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_fizetndo_5" model="account.report.line">
                                <field name="name">5% VAT</field>
                                <field name="code">VAT_PAY_5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_fizetndo_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vat_pay_5</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_rec" model="account.report.line">
                        <field name="name">Recoverable</field>
                        <field name="code">VAT_RECOVERABLE</field>
                        <field name="aggregation_formula">VAT_REC_27.balance + VAT_REC_18.balance + VAT_REC_5.balance + VAT_REC_COMPENSATION.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_fizetndo_viss_27" model="account.report.line">
                                <field name="name">27% VAT</field>
                                <field name="code">VAT_REC_27</field>
                                <field name="expression_ids">
                                    <record id="tax_report_fizetndo_viss_27_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vat_rec_27</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_fizetndo_viss_18" model="account.report.line">
                                <field name="name">18% VAT</field>
                                <field name="code">VAT_REC_18</field>
                                <field name="expression_ids">
                                    <record id="tax_report_fizetndo_viss_18_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vat_rec_18</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_fizetndo_viss_5" model="account.report.line">
                                <field name="name">5% VAT</field>
                                <field name="code">VAT_REC_5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_fizetndo_viss_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vat_rec_5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_fizetndo_viss_komp" model="account.report.line">
                                <field name="name">Compensation Surcharge</field>
                                <field name="code">VAT_REC_COMPENSATION</field>
                                <field name="expression_ids">
                                    <record id="tax_report_fizetndo_viss_komp_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vat_rec_compensation</field>
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
    <!-- Sales -->
    <record id="F27" model="account.tax.template">
        <field name="sequence">10</field>
        <field name="name">27%</field>
        <field name="description">27%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">27</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_27"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_fiz_afa_27_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_467'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_27_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_fiz_afa_27_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_467'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_27_tag')],
            }),
        ]"/>
    </record>

    <record id="F18" model="account.tax.template">
        <field name="sequence">20</field>
        <field name="name">18%</field>
        <field name="description">18%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">18</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_18"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_fiz_afa_18_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_467'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_18_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_fiz_afa_18_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_467'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_18_tag')],
            }),
        ]"/>
    </record>

    <record id="F5" model="account.tax.template">
        <field name="sequence">30</field>
        <field name="name">5%</field>
        <field name="description">5%</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_fiz_afa_5_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_467'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_fiz_afa_5_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_467'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_5_tag')],
            }),
        ]"/>
    </record>

    <record id="FA" model="account.tax.template">
        <field name="sequence">40</field>
        <field name="name">0% AAM</field>
        <field name="description">0% AAM</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_fiz_alanyi_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_fiz_alanyi_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="FT" model="account.tax.template">
        <field name="sequence">50</field>
        <field name="name">0% TAM</field>
        <field name="description">0% TAM</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_fiz_targyi_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_fiz_targyi_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="FKKS" model="account.tax.template">
        <field name="sequence">60</field>
        <field name="name">0% S EXEMPT</field>
        <field name="description">0% S EXEMPT</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_fiz_koron_kivuli_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_fiz_koron_kivuli_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="FKKT" model="account.tax.template">
        <field name="sequence">61</field>
        <field name="name">0% G EXEMPT</field>
        <field name="description">0% G EXEMPT</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_fiz_koron_kivuli_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_fiz_koron_kivuli_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="FF" model="account.tax.template">
        <field name="sequence">70</field>
        <field name="name">0% R</field>
        <field name="description">0% R</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_forditott_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_forditott_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="FEUSZ" model="account.tax.template">
        <field name="sequence">80</field>
        <field name="name">0% EU S</field>
        <field name="description">0% EU S</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_fiz_eu_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_fiz_eu_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="FEUT" model="account.tax.template">
        <field name="sequence">81</field>
        <field name="name">0% EU G</field>
        <field name="description">0% EU G</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_fiz_eu_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_fiz_eu_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="FEXS" model="account.tax.template">
        <field name="sequence">90</field>
        <field name="name">0% S EX</field>
        <field name="description">0% S EX</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_fiz_export_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_fiz_export_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="FEXT" model="account.tax.template">
        <field name="sequence">91</field>
        <field name="name">0% G EX</field>
        <field name="description">0% G EX</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_fiz_export_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_fiz_export_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <!-- Sale [END] -->

    <!-- Purchase -->
    <record id="V27" model="account.tax.template">
        <field name="sequence">100</field>
        <field name="name">27%</field>
        <field name="description">27%</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">27</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_27"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_viss_27_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_viss_27_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_viss_27_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_viss_27_tag')]
            }),
        ]"/>
    </record>

    <record id="V27TE" model="account.tax.template">
        <field name="sequence">110</field>
        <field name="name">27% PPE</field>
        <field name="description">27% PPE</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">27</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_27"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_viss_27_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_viss_27_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_viss_27_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_viss_27_tag')]
            }),
        ]"/>
    </record>

    <record id="V18" model="account.tax.template">
        <field name="sequence">120</field>
        <field name="name">18%</field>
        <field name="description">18%</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">18</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_18"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_viss_18_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_viss_18_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_viss_18_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_viss_18_tag')]
            }),
        ]"/>
    </record>

    <record id="V5" model="account.tax.template">
        <field name="sequence">130</field>
        <field name="name">5%</field>
        <field name="description">5%</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_viss_5_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_viss_5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_viss_5_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_viss_5_tag')]
            }),
        ]"/>
    </record>

    <record id="VKOMP7" model="account.tax.template">
        <field name="sequence">140</field>
        <field name="name">7% CS</field>
        <field name="description">7% CS</field>
        <field name="price_include" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="amount">7</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_komp"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_komp_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_viss_komp_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_komp_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_viss_komp_tag')]
            }),
        ]"/>
    </record>

    <record id="VKOMP12" model="account.tax.template">
        <field name="sequence">150</field>
        <field name="name">12% CS</field>
        <field name="description">12% CS</field>
        <field name="price_include" eval="0"/>
        <field name="amount">12</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_komp"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_komp_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_viss_komp_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_komp_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_viss_komp_tag')]
            }),
        ]"/>
    </record>

    <record id="VA" model="account.tax.template">
        <field name="sequence">160</field>
        <field name="name">0% AAM</field>
        <field name="description">0% AAM</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_viss_alanyi_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_viss_alanyi_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="VT" model="account.tax.template">
        <field name="sequence">170</field>
        <field name="name">0% TAM</field>
        <field name="description">0% TAM</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_viss_targyi_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_viss_targyi_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="VKKS" model="account.tax.template">
        <field name="sequence">180</field>
        <field name="name">0% S EXEMPT</field>
        <field name="description">0% S EXEMPT</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_viss_koron_kivuli_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_viss_koron_kivuli_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="VKKT" model="account.tax.template">
        <field name="sequence">181</field>
        <field name="name">0% G EXEMPT</field>
        <field name="description">0% G EXEMPT</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_viss_koron_kivuli_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_viss_koron_kivuli_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="VF" model="account.tax.template">
        <field name="sequence">190</field>
        <field name="name">0% R</field>
        <field name="description">0% R</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_forditott_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_forditott_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="VEU27S" model="account.tax.template">
        <field name="sequence">200</field>
        <field name="name">27% EU S</field>
        <field name="description">27% EU S</field>
        <field name="price_include" eval="0"/>
        <field name="amount">27</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_27"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_viss_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_467'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_27_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_viss_27_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_viss_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_467'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_27_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_viss_27_tag')],
            }),
        ]"/>
    </record>

    <record id="VEU27T" model="account.tax.template">
        <field name="sequence">201</field>
        <field name="name">27% EU G</field>
        <field name="description">27% EU G</field>
        <field name="price_include" eval="0"/>
        <field name="amount">27</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_27"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_viss_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_467'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_27_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_viss_27_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_viss_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_467'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_27_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_viss_27_tag')],
            }),
        ]"/>
    </record>

    <record id="VEU27TE" model="account.tax.template">
        <field name="sequence">210</field>
        <field name="name">27% EU P</field>
        <field name="description">27% EU P</field>
        <field name="price_include" eval="0"/>
        <field name="amount">27</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_27"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_viss_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_467'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_27_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_viss_27_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_viss_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_467'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_27_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_viss_27_tag')],
            }),
        ]"/>
    </record>

    <record id="VEUM" model="account.tax.template">
        <field name="sequence">220</field>
        <field name="name">0% ICA</field>
        <field name="description">0% ICA</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_viss_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_viss_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="VIMS" model="account.tax.template">
        <field name="sequence">230</field>
        <field name="name">27% EX</field>
        <field name="description">27% EX</field>
        <field name="price_include" eval="0"/>
        <field name="amount">27</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_27"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_import_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_467'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_27_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_viss_27_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_import_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_467'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_27_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_viss_27_tag')],
            }),
        ]"/>
    </record>

    <record id="VIMK" model="account.tax.template">
        <field name="sequence">240</field>
        <field name="name">27% EX S</field>
        <field name="description">27% EX S</field>
        <field name="price_include" eval="0"/>
        <field name="amount">27</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_27"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_import_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'plus_report_expression_ids': [ref('tax_report_fizetndo_viss_27_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_import_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('l10n_hu_466'),
                'minus_report_expression_ids': [ref('tax_report_fizetndo_viss_27_tag')],
            }),
        ]"/>
    </record>

    <record id="VIM" model="account.tax.template">
        <field name="sequence">250</field>
        <field name="name">0% EX</field>
        <field name="description">0% EX</field>
        <field name="price_include" eval="0"/>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_alap_import_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_alap_import_tag')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>
    <!-- Purchase [END] -->
</odoo>

```

## File: data\l10n_hu_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hungarian_chart_template" model="account.chart.template">
        <field name="name">Hungary - National Chart of Accounts</field>
        <field name="property_account_receivable_id" ref="l10n_hu_311"/>
        <field name="property_account_payable_id" ref="l10n_hu_454"/>
        <field name="property_account_expense_categ_id" ref="l10n_hu_811"/>
        <field name="property_account_income_categ_id" ref="l10n_hu_911"/>
        <field name="income_currency_exchange_account_id" ref="l10n_hu_976"/>
        <field name="expense_currency_exchange_account_id" ref="l10n_hu_876"/>
        <field name="property_tax_payable_account_id" ref="l10n_hu_468"/>
        <field name="property_tax_receivable_account_id" ref="l10n_hu_468"/>
    </record>
</odoo>

```

## File: data\res.bank.csv

```csv
"id","name","bic","country/id","zip","city","street","email","phone","active"
"BKCHHUHBXXX","Bank of China (Hungária) Hitelintézet Rt.","BKCHHUHBXXX","base.hu",1051,"Budapest","József Nádor tér 7.","service_hu@bank-of-china.com","+3614299200","True"
"BNPAHUHX","BNP Paribas Hungária Bank Rt.","BNPAHUHX","base.hu",1051,"Budapest","Teréz krt. 55-57.","csd_hungary@bnpparibas.com","+3613746300","True"
"BUDAHUHB","Budapest Hitel- és Fejlesztési Bank Rt.","BUDAHUHB","base.hu",1138,"Budapest","Váci út 188."," info@budapestbank.hu","+3614506060","False"
"CIBHHUHB","CIB Közép-Európai Nemzetközi Bank Zrt.","CIBHHUHB","base.hu",1027,"Budapest","Medve utca 4-14","cib@cib.hu","+3614242242","True"
"CITIHUHX","Citibank Rt.","CITIHUHX","base.hu",1051,"Budapest","Szabadság tér 7.",,"+3613745000","True"
"COBAHUHX","Commerzbank Zártkörűen Működő Rt.","COBAHUHX","base.hu",1054,"Budapest","Széchenyi rakpart 8.","info.budapest@commerzbank.com","+3613748100","False"
"DEUTHU2B","Deutsche Bank Zártkörűen Működő Rt.","DEUTHU2B","base.hu",1054,"Budapest","Hold utca 27.","db.hungary@db.com","+3613013700","True"
"FHKBHUHB","FHB Kereskedelmi Bank Zrt.","FHKBHUHB","base.hu",1082,"Budapest","Üllői út 48."," info@fhb.hu","+3614529100","False"
"GIBAHUHB","ERSTE Bank Hungary Zrt.","GIBAHUHB","base.hu",1138,"Budapest","Népfürdő u. 24-26.","erste@erstebank.hu","+3612980222","True"
"GNBAHUHB","Gránit Bank Zrt.","GNBAHUHB","base.hu",1095,"Budapest","Lechner Ödön fasor 8.","info@granitbank.hu","+3615100527","True"
"INCNHUHB","IC Bank Rt.","INCNHUHB","base.hu",1088,"Budapest","Rákóczi út 1-3."," level@bancopopolare.hu ","+3640200515","False"
"INGBHUHB","ING Bank N.V. Magyarországi Fióktelepe","INGBHUHB","base.hu",1068,"Budapest","Dózsa György út 84/b.","communications.hu@ingbank.com","+3612358700","True"
"OKHBHUHB","K&H Bank Zrt.","OKHBHUHB","base.hu",1095,"Budapest","Lechner Ödön fasor 9.","bank@kh.hu","+36203353355","True"
"HBWEHUHB","MagNet Magyar Közösségi Bank Zrt.","HBWEHUHB","base.hu",1062,"Budapest","Andrássy út 98.","info@magnetbank.hu","+3614288888","True"
"MANEHUHB","Magyar Nemzeti Bank","MANEHUHB","base.hu",1054,"Budapest","Szabadság tér 8-9.","info@mnb.hu","+3614282600","True"
"MKKBHUHBXXX","MHB Bank Nyrt.","MKKBHUHBXXX","base.hu",1056,"Budapest","Váci utca 38.","+3612687173","ugyfelszolgalat@mbhbank.hu","True"
"OTPVHUHB","OTP Bank Nyrt.","OTPVHUHB","base.hu",1051,"Budapest","Nádor utca 16.","otpbank@otpbank.hu","+3614735000","True"
"UBRTHUHB","RAIFFEISEN Bank Zrt.","UBRTHUHB","base.hu",1054,"Budapest","Akadémia utca 6.","info@raiffeisen.hu","+3680488588","True"
"MAVOHUHB","Sberbank Magyarország Zrt.","MAVOHUHB","base.hu",1088,"Budapest","Rákóczi út 7.","info@sberbank.hu","+3614114200","False"
"TAKBHUHB","TakarékBank Zrt.","TAKBHUHB","base.hu",1122,"Budapest","Pethényi  köz 10.","info@tbank.hu","False"
"BACXHUHB","UniCredit Bank Hungary Zrt.","BACXHUHB","base.hu",1054,"Budapest","Szabadság tér 5-6.","info@unicreditgroup.hu","+3613253200","True"

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="7.25" width="50.4" height="32.5" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.53" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="255" height="128" transform="translate(4.8 7.25) scale(0.2 0.25)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAP8AAAClCAYAAACTHStbAAAACXBIWXMAADf6AAA3+gH8300lAAACJElEQVR4Xu3WMW0DQRRF0Wy0krvACQYDMQ33xmEOBhAGgWAm3xCmn3tO/dorveN9+Z0vIOd7NQD2JH6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R58/tutoAGzpmZlYjYD9uP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ9R5fz5WG2BD5+v/b7UBNuT2Q5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RB0zs9oAG/oAROcPClSjRlIAAAAASUVORK5CYII="/>
    </g>
  </g>
</svg>

```

