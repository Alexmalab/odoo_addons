# Odoo Module: l10n_bg

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, SUPERUSER_ID

def load_translations(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref('l10n_bg.l10n_bg_chart_template').process_coa_translations()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Bulgaria - Accounting',
    'icon': '/l10n_bg/static/description/icon.png',
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'author': 'Odoo S.A.',
    'description': """
        Chart accounting and taxes for Bulgaria
    """,
    'depends': [
        'account', 'base_vat', 'l10n_multilang',
    ],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/account.group.template.csv',
        'data/l10n_bg_chart_data.xml',
        'data/tax_report.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_template_data.xml',
        "data/account_fiscal_position_template.xml",
        'data/account_chart_template_configure_data.xml',
        'data/menuitem.xml',
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
"id","name","code","user_type_id/id","chart_template_id/id","tag_ids/id","reconcile"
"l10n_bg_101","Fixed capital required for registration","101","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_102","Fixed capital that does not require registration","102","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_103","Liquidation capital in case of insolvency and liquidation","103","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_104","Capital of non-profit enterprises","104","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_105","Capital premiums","105","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_106","Deductions (discount) related to capital","106","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_111","General reserves","111","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_112","Reserves from subsequent valuation of fixed assets","112","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_113","Reserves from subsequent valuation of current assets","113","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_114","Reserves from subsequent valuation of financial instruments","114","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_115","Reserves from share issue","115","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_116","Reserve related to repurchased own shares","116","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_117","Reserve according to constitutive act","117","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_119","Other reserves","119","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_121","Uncovered loss from previous years","121","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_122","Retained earnings from previous years","122","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_123","Profit and loss from the current year","123","account.data_unaffected_earnings","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_124","Result in bankruptcy and liquidation","124","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_125","Result of the activity of non-profit enterprises","125","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_131","Life insurance reserves","131","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_132","Transmission-premium reserves","132","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_133","Reserves for forthcoming payments","133","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_134","Reserves for reserve fund","134","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_135","Reinsurance reserves","135","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_136","Other reserves under insurance contracts","136","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_137","Reserves of investment companies","137","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_138","Reserves of pension companies","138","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_141","Banknotes for circulation","141","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_142","Coins in circulation","142","account.data_account_type_equity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_151","Received short-term loans","151","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_152","Long-term loans received","152","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_153","Debt instruments","153","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_159","Other loans and debts","159","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_201","Lands (terrains)","201","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_202","Land improvements","202","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_203","Buildings and structures","203","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_204","Computer equipment","204","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_205","Facilities","205","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_206","Machinery and equipment","206","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_207","Vehicles","207","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_208","Office furniture","208","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_209","Other tangible fixed assets","209","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_211","Development products","211","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_212","Software products","212","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_213","Intellectual property rights","213","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_214","Industrial property rights","214","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_219","Other intangible fixed assets","219","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_221","Investments in subsidiaries","221","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_222","Investments in associates","222","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_223","Investments in joint ventures","223","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_224","Investment property","224","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_225","Financial assets held to maturity","225","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_226","Financial assets put up for sale","226","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_227","Financial assets pledged as compensation","227","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_228","Government securities","228","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_229","Other long-term financial assets","229","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_231","Positive commercial reputation","231","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_232","Negative commercial reputation","232","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_241","Depreciation of tangible fixed assets","241","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_2413","Depreciation of buildings and structures","2413","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_2414","Depreciation of computer equipment","2414","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_2415","Depreciation of equipment","2415","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_2416","Depreciation of machinery and equipment","2416","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_2417","Depreciation of vehicles","2417","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_2418","Depreciation of office furniture","2418","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_2419","Depreciation of other tangible assets","2419","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_242","Amortization of intangible fixed assets","242","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_2421","Depreciation of development products","2421","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_2422","Depreciation of software products","2422","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_2423","Depreciation of intellectual property rights","2423","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_2424","Depreciation of industrial property rights","2424","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_2429","Depreciation of other intangible fixed assets","2429","account.data_account_type_fixed_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_261","Long-term receivables and loans to banks and other financial institutions","261","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_262","Long-term receivables and loans to non-financial corporations and other customers","262","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_263","Overdue long-term receivables and loans granted","263","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_264","Long-term receivables and loans granted as collateral","264","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_269","Other long-term receivables and loans","269","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_271","Burning","271","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_272","Permanent plantations - fruitful","272","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_273","Permanent plantations - infertile","273","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_274","Animals in major herds","274","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_279","Other fixed biological assets","279","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_301","Delivery","301","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_302","Materials","302","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_303","Products","303","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_304","Goods","304","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_305","Work in progress","305","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_311","Small productive animals","311","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_312","Bird-main flocks","312","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_313","Bee families","313","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_314","Young animals","314","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_315","Animals for fattening","315","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_316","Animals for experimental purposes","316","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_319","Other current biological assets","319","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_401","Providers","401","account.data_account_type_payable","l10n_bg.l10n_bg_chart_template","","True"
"l10n_bg_402","Advance providers","402","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_403","Trade credit providers","403","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_404","Suppliers of supplies under certain conditions","404","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_405","Suppliers of related party supplies","405","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_406","Settlements with related parties for purchases","406","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_409","Other settlements with suppliers","409","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_411","Customers","411","account.data_account_type_receivable","l10n_bg.l10n_bg_chart_template","","True"
"l10n_bg_412","Clients on advances","412","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_413","Clients on trade credits","413","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_414","Sales customers under certain conditions","414","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_415","Sales customers with related parties","415","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_416","Settlements with related parties for sales","416","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_419","Other settlements with clients","419","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_421","Staff","421","account.data_account_type_payable","l10n_bg.l10n_bg_chart_template","","True"
"l10n_bg_422","Accountable persons","422","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_423","Liabilities for unused leave","423","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_424","Receivables from participations","424","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_425","Obligations for participation","425","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_426","Receivables from subscribed share contributions","426","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_429","Other settlements with staff and partners","429","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_430","Internal calculations","430","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_431","Settlements by interbank operations","431","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_432","Internal settlements on interbank operations","432","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_433","Internal settlements on intrabank operations","433","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_439","Other internal estimates","439","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_441","Receivables on claims","441","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_442","Receivables for losses and deductions","442","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_443","Price differences by shortages and readings","443","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_444","Claims in litigation","444","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_445","Awarded receivables","445","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_451","Settlements with municipalities","451","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_452","Profit tax estimates","452","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_453","Estimates of value added tax","453","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_4531","Vat purchases","4531","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_4532","Vat sales","4532","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_4534","Vat sales outside the country","4534","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_4538","Vat recovery","4538","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_4539","Vat to pay","4539","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_454","Estimates of personal income taxes","454","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_455","Settlements with ministries","455","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_456","Excise estimates","456","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_457","Settlements with customs","457","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_459","Other estimates with the budget and with departments","459","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_461","Settlements with the national social security institute","461","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_4611","Estimates for the social security fund","4611","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_4612","Estimates for smps fund","4612","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_4613","Estimates for the upf fund","4613","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_4614","Estimates for the gvrs fund","4614","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_4615","Estimates for the tzpb fund","4615","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_462","Voluntary social security estimates","462","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_463","Health insurance estimates","463","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_464","Estimates for one-time benefits and child allowances","464","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_469","Other settlements with insurers","469","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_471","Settlements with the international monetary fund","471","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_472","Settlements with the world bank","472","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_473","Settlements with other international financial institutions","473","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_474","Estimates under intergovernmental agreements","474","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_481","Estimates of upcoming payments","481","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_482","Reinsurance and co-insurance estimates","482","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_483","Settlements with reinsurers","483","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_484","Settlements with sedants","484","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_489","Other estimates","489","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_491","Trustees","491","account.data_account_type_non_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_492","Guarantee estimates","492","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_493","Settlements with owners","493","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_494","Insurance estimates","494","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_495","Interest calculations","495","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_496","Deferred tax estimates","496","account.data_account_type_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_497","Provisions recognized as liabilities","497","account.data_account_type_non_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_4971","Provisions for pensions and other similar liabilities","4971","account.data_account_type_non_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_4972","Other provisions and similar liabilities","4972","account.data_account_type_non_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_498","Other debtors","498","account.data_account_type_payable","l10n_bg.l10n_bg_chart_template","","True"
"l10n_bg_499","Other creditors","499","account.data_account_type_receivable","l10n_bg.l10n_bg_chart_template","","True"
"l10n_bg_501","Cash register in bgn","501","account.data_account_type_liquidity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_502","Cash in foreign currency","502","account.data_account_type_liquidity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_503","Current account in bgn","503","account.data_account_type_liquidity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_504","Current account in foreign currency","504","account.data_account_type_liquidity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_505","Letters of credit in bgn","505","account.data_account_type_liquidity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_506","Letters of credit in foreign currency","506","account.data_account_type_liquidity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_507","Deposits provided","507","account.data_account_type_liquidity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_508","Payment checks","508","account.data_account_type_liquidity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_509","Other cash","509","account.data_account_type_liquidity","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_511","Financial assets held for trading","511","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_512","Repurchased own shares","512","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_513","Financial assets pledged as collateral","513","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_514","Repurchased own liabilities","514","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_515","Government securities","515","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_516","Precious metals and precious stones","516","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_517","Shares and interests in group companies","517","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_518","Financing","518","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_519","Other short-term financial assets","519","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_521","Short-term receivables and loans from banks and other financial institutions","521","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_522","Short-term receivables and loans from non-financial corporations and other customers","522","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_523","Short-term receivables and loans pledged as collateral","523","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_524","Overdue short-term receivables and loans","524","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_529","Other current receivables and loans","529","account.data_account_type_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_531","Prepaid expenses","531","account.data_account_type_non_current_assets","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_532","Income for future periods","532","account.data_account_type_non_current_liabilities","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_601","Material costs","601","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_602","Costs for external services","602","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_603","Depreciation costs","603","account.data_account_type_depreciation","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_604","Wage costs (remuneration)","604","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_6041","Compensation costs","6041","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_6042","Expenses for additional remuneration","6042","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_6043","Paid leave expenses","6043","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_6044","Hospital costs","6044","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_6045","Expenses for remuneration of self-insured persons","6045","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_605","Insurance costs","605","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_6051","Expenditures for the social security fund","6051","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_6052","Expenditures for smps fund","6052","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_6053","Expenditures for the upf fund","6053","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_6054","Expenditures for the gvrs fund","6054","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_6055","Expenditures for tzpb fund","6055","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_6056","Expenditures for the health fund","6056","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_606","Expenses for taxes, fees and other similar payments","606","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_606451","Expenditures for settlements with municipalities","606451","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_606452","Costs of income tax estimates","606452","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_606454","Expenses for estimates of personal income taxes","606454","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_606459","Expenditures for other estimates with the budget and with departments","606459","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_607","Provision costs","607","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_608","Expenses from subsequent valuations of assets","608","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_609","Other expenses","609","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_611","Operating expenses","611","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_612","Ancillary activity costs","612","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_613","Expenses for acquisition of fixed assets","613","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_614","Administrative costs","614","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_615","Sales costs","615","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_616","Expenses for liquidation of tangible fixed assets","616","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_617","Expenses on activities in non-profit enterprises","617","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_618","Liquidation and insolvency costs","618","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_621","Interest expenses","621","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_622","Provisioning costs for risky assets","622","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_623","Expenses on operations with financial assets and instruments","623","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_624","Expenses on foreign exchange operations","624","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_625","Expenses from subsequent valuations of financial assets and instruments","625","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_629","Other financial expenses","629","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_631","Impairment loss (expense)","631","account.data_account_type_depreciation","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_632","Impairment losses on current assets","632","account.data_account_type_depreciation","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_651","Non-financial expenses for future periods","651","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_652","Financial expenses for future periods","652","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_661","Expenses for insurance amounts","661","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_662","Costs for participation in the result","662","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_663","Liquidation costs","663","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_664","Insurance commission expenses","664","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_665","Insurance reserve costs","665","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_669","Other direct insurance costs","669","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_671","Expenses for reinsurers' premiums","671","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_672","Expenses for released reserves for passive reinsurance","672","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_679","Other passive reinsurance costs","679","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_681","Expenses for insurance benefits","681","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_682","Costs for participation in the result","682","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_683","Expenses for participation in the liquidation","683","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_684","Expenses for assigned reinsurance commissions and fees","684","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_685","Expenses for set aside reserves for active reinsurance","685","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_689","Other active reinsurance costs","689","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_691","Exceptional costs","691","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_701","Revenues from sales of products","701","account.data_account_type_revenue","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_702","Revenues from sales of goods","702","account.data_account_type_revenue","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_703","Revenues from sales of services","703","account.data_account_type_revenue","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_704","Revenues from financing","704","account.data_account_type_revenue","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_705","Income from sales of fixed assets","705","account.data_account_type_revenue","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_706","Revenues from sales of materials","706","account.data_account_type_revenue","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_707","Proceeds from liquidation and insolvency","707","account.data_account_type_revenue","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_709","Other operating income","709","account.data_account_type_revenue","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_711","Revenues from regulated activities","711","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_712","Revenues from membership fees","712","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_713","Revenues from program financing","713","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_714","Revenues from government donations","714","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_715","Revenues from other donations","715","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_719","Other incomes","719","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_721","Interest income","721","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_722","Income from participations","722","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_723","Income from operations with financial assets and instruments","723","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_724","Income from foreign exchange operations","724","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_725","Income from subsequent valuations of financial assets and instruments","725","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_726","Revenues from reintegrated provisions","726","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_729","Other financial income","729","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_731","Income from recovered impairment losses","731","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_751","Non-financial income for future periods","751","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_752","Financial income for future periods","752","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_753","Financing for fixed assets","753","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_754","Financing the current activity","754","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_761","Income from insurance premiums","761","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_762","Revenues from commissions and fees","762","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_763","Insurance income from previous years","763","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_764","Revenue from recourses","764","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_765","Revenues from released reserves","765","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_769","Other insurance income","769","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_771","Income from indemnities received from reinsurers","771","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_772","Income from participation in the result of reinsurers","772","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_773","Revenues from commissions from reinsurers","773","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_774","Passive reinsurance income from previous years","774","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_775","Income from reinsurance reserves","775","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_779","Other income from passive reinsurance","779","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_781","Reinsurance premium income","781","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_782","Income from active reinsurance from previous years","782","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_783","Income from active reinsurance recourses","783","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_784","Income from active reinsurance reserves","784","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_789","Other income from active reinsurance","789","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_791","Extraordinary income","791","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_911","Foreign tangible and intangible assets provided as collateral","911","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_912","Leased foreign assets","912","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_913","Foreign tangible assets received under a consignment agreement","913","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_914","Documents under special reporting","914","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_915","Non-financial assets accepted for safekeeping","915","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_921","Foreign financial assets provided as collateral","921","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_922","Pledged policies","922","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_923","Miscellaneous emissions for circulation","923","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_924","Foreign financial assets held on behalf of customers","924","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_925","Commitments under contracts","925","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_931","Receivables from loans granted under interstate agreements","931","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_932","Outstanding receivables written off","932","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_933","Debtors by collection operations","933","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_934","Receivables from derivative transactions","934","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_941","Receivables from spot transactions","941","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_942","Write-off of outstanding receivables in banks","942","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_943","Debtors on collection operations in banks","943","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_945","Contingent receivables from other banking operations","945","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_951","Guarantees and other similar contingent liabilities","951","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_952","Irrevocable commitments","952","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_953","Cancellable commitments","953","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_954","Liabilities under spot operations","954","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_955","Liabilities under derivative transactions","955","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_956","Contingent liabilities on other banking operations","956","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_961","Reserve fund","961","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_962","Assets in use, reported","962","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_971","Issue reserve","971","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_972","Own issues out of circulation with expired term for exchange","972","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_981","Reserve account","981","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_982","Accounts receivable","982","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_983","Commitments on authorized loans","983","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_984","Various statistical accounts","984","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_989","Other contingent asset accounts","985","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_991","Other contingent liability accounts","991","account.data_account_off_sheet","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_791001","Cash Difference Gain","791001","account.data_account_type_other_income","l10n_bg.l10n_bg_chart_template","","False"
"l10n_bg_691001","Cash Difference Loss","691001","account.data_account_type_expenses","l10n_bg.l10n_bg_chart_template","","False"

```

## File: data\account.group.template.csv

```csv
"id","code_prefix_start","name","chart_template_id/id"
"l10n_bg_group_1","1","Capital accounts","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_10","10","Capital","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_11","11","Capital reserves","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_12","12","Financial results","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_13","13","Specific non-capital reserves","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_14","14","Cash issues","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_15","15","Loans received","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_16","16","Attracted funds on deposit accounts","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_17","17","Borrowed funds on current and other accounts","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_18","18","Funds raised on accounts of participants in insurance funds","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_19","19","Premiums and discounts on financial instruments","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_2","2","Accounts for fixed assets","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_20","20","Tangible fixed assets","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_21","21","Tangible fixed assets","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_22","22","Long-term financial assets","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_23","23","Goodwill","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_24","24","Depreciation","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_26","26","Long-term receivables and loans","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_27","27","Long-term biological assets","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_3","3","Accounts for inventories","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_30","30","Materials, products and goods","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_31","31","Current biological assets","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_4","4","Accounts for settlements","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_40","40","Providers and related accounts","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_41","41","Customers and related accounts","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_42","42","Staff and partners","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_43","43","Translation estimates and internal estimates","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_44","44","Receivables from absences, deductions and litigation","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_45","45","Estimates from the budget and departments","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_46","46","Settlements with insurers","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_47","47","Settlements with international financial institutions","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_48","48","Insurance and co-insurance estimates","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_49","49","Miscellaneous debtors and creditors","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_5","5","Accounts for financial resources","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_50","50","Cash","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_51","51","Current financial assets","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_52","52","Current receivables and loans","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_6","6","Expense accounts","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_60","60","Costs by economic elements","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_61","61","Activity costs","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_62","62","Financial costs","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_63","63","Impairment loss (expense)","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_65","65","Prepaid expenses","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_66","66","Direct insurance costs","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_67","67","Passive reinsurance expenses","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_68","68","Active reinsurance expenses","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_69","69","Exceptional costs","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_7","7","Income accounts","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_70","70","Sales revenue","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_71","71","Revenues in non-profit enterprises","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_72","72","Financial income","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_73","73","Income from recovered impairment losses","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_75","75","Deferred income and financing","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_76","76","Income from direct insurance","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_77","77","Passive reinsurance income","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_78","78","Income from active reinsurance","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_79","79","Extraordinary income","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_9","9","Conditional liabilities and contingent assets accounts","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_91","91","Foreign tangible and intangible assets","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_92","92","Foreign financial assets","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_93","93","Debtors on contingent receivables","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_94","94","Contingent assets","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_95","95","Contingent creditors","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_96","96","Own assets not included in economic turnover","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_97","97","Own liabilities not included in economic turnover","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_98","98","Miscellaneous contingent asset accounts","l10n_bg.l10n_bg_chart_template"
"l10n_bg_group_99","99","Various contingent liability accounts","l10n_bg.l10n_bg_chart_template"

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_bg.l10n_bg_chart_template')]"/>
    </function>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_bg_chart_template" model="account.chart.template">
        <field name="name">Bulgaria - National Chart of Accounts</field>
        <field name="bank_account_code_prefix">503</field>
        <field name="cash_account_code_prefix">501</field>
        <field name="transfer_account_code_prefix">430</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.BGN"/>
        <field name="country_id" ref="base.bg"/>
        <field name="spoken_languages" eval="'bg_BG'"/>
    </record>
</odoo>

```

## File: data\account_fiscal_position_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Fiscal positions -->
        <record id="fiscal_position_template_dom" model="account.fiscal.position.template">
            <field name="sequence">10</field>
            <field name="name">Domestic</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_id" ref="base.bg"/>
        </record>

        <record id="fiscal_position_bg_private_eu" model="account.fiscal.position.template">
            <field name="name">EU B2C</field>
            <field name="sequence">20</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>

        <record id="fiscal_position_template_in_eu" model="account.fiscal.position.template">
            <field name="sequence">30</field>
            <field name="name">EU B2B</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>

        <record id="fiscal_position_template_out_eu" model="account.fiscal.position.template">
            <field name="sequence">40</field>
            <field name="name">Outside EU</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="auto_apply" eval="True"/>
        </record>
        <!-- Fiscal positions [END] -->

        <!-- Sales -->
            <!-- 20% VAT -->
            <record id="fp_in_eu_sale_20_vat" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_in_eu"/>
                <field name="tax_src_id" ref="l10n_bg_sale_vat_20"/>
                <field name="tax_dest_id" ref="l10n_bg_sale_vat_0_icd"/>
            </record>

            <record id="fp_out_eu_sale_20_vat" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_out_eu"/>
                <field name="tax_src_id" ref="l10n_bg_sale_vat_20"/>
                <field name="tax_dest_id" ref="l10n_bg_sale_vat_0_export"/>
            </record>
            <!-- 20% VAT [END] -->

            <!-- 20% Personal use -->
            <record id="fp_in_eu_sale_20_personal" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_in_eu"/>
                <field name="tax_src_id" ref="l10n_bg_sale_vat_20_personal"/>
                <field name="tax_dest_id" ref="l10n_bg_sale_vat_0_icd"/>
            </record>

            <record id="fp_out_eu_sale_20_personal" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_out_eu"/>
                <field name="tax_src_id" ref="l10n_bg_sale_vat_20_personal"/>
                <field name="tax_dest_id" ref="l10n_bg_sale_vat_0_export"/>
            </record>
            <!-- 20% Personal use [END] -->

            <!-- 9% VAT -->
            <record id="fp_in_eu_sale_9_vat" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_in_eu"/>
                <field name="tax_src_id" ref="l10n_bg_sale_vat_9"/>
                <field name="tax_dest_id" ref="l10n_bg_sale_vat_0_icd"/>
            </record>

            <record id="fp_out_eu_sale_9_vat" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_out_eu"/>
                <field name="tax_src_id" ref="l10n_bg_sale_vat_9"/>
                <field name="tax_dest_id" ref="l10n_bg_sale_vat_0_export"/>
            </record>
            <!-- 9% VAT [END] -->

            <!-- 9% Personal use -->
            <record id="fp_in_eu_sale_9_personal" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_in_eu"/>
                <field name="tax_src_id" ref="l10n_bg_sale_vat_9_personal"/>
                <field name="tax_dest_id" ref="l10n_bg_sale_vat_0_icd"/>
            </record>

            <record id="fp_out_eu_sale_9_personal" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_out_eu"/>
                <field name="tax_src_id" ref="l10n_bg_sale_vat_9_personal"/>
                <field name="tax_dest_id" ref="l10n_bg_sale_vat_0_export"/>
            </record>
            <!-- 9% Personal use [END] -->

            <!-- 0% Exempt -->
            <record id="fp_in_eu_sale_0_exempt" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_in_eu"/>
                <field name="tax_src_id" ref="l10n_bg_sale_vat_0_exempt"/>
                <field name="tax_dest_id" ref="l10n_bg_sale_vat_0_icd"/>
            </record>

            <record id="fp_out_eu_sale_9_personal" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_out_eu"/>
                <field name="tax_src_id" ref="l10n_bg_sale_vat_0_exempt"/>
                <field name="tax_dest_id" ref="l10n_bg_sale_vat_0_export"/>
            </record>
            <!-- 0% Exempt [END] -->
        <!-- Sales [END] -->

        <!-- Purchase -->
            <!-- 20% Other Tax Credit -->
            <record id="fp_in_eu_purchase_20_otc" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_in_eu"/>
                <field name="tax_src_id" ref="l10n_bg_purchase_vat_20_otc"/>
                <field name="tax_dest_id" ref="l10n_bg_purchase_vat_0_otc_ica"/>
            </record>

            <record id="fp_out_eu_purchase_20_otc" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_out_eu"/>
                <field name="tax_src_id" ref="l10n_bg_purchase_vat_20_otc"/>
                <field name="tax_dest_id" ref="l10n_bg_purchase_vat_0_otc_exempt"/>
            </record>
            <!-- 20% Other Tax Credit [END] -->

            <!-- 20% Foreign Tax Credit -->
            <record id="fp_in_eu_purchase_20_ftc" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_in_eu"/>
                <field name="tax_src_id" ref="l10n_bg_purchase_vat_20_ftc"/>
                <field name="tax_dest_id" ref="l10n_bg_purchase_vat_20_ftc_ica"/>
            </record>

            <record id="fp_out_eu_purchase_20_ftc" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_out_eu"/>
                <field name="tax_src_id" ref="l10n_bg_purchase_vat_20_ftc"/>
                <field name="tax_dest_id" ref="l10n_bg_purchase_vat_20_ftc_exempt"/>
            </record>
            <!-- 20% Foreign Tax Credit [END] -->

            <!-- 20% Payable Tax Credit -->
            <record id="fp_in_eu_purchase_20_ptc" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_in_eu"/>
                <field name="tax_src_id" ref="l10n_bg_purchase_vat_20_ptc"/>
                <field name="tax_dest_id" ref="l10n_bg_purchase_vat_20_ptc_ica"/>
            </record>

            <record id="fp_out_eu_purchase_20_ptc" model="account.fiscal.position.tax.template">
                <field name="position_id" ref="fiscal_position_template_out_eu"/>
                <field name="tax_src_id" ref="l10n_bg_purchase_vat_20_ptc"/>
                <field name="tax_dest_id" ref="l10n_bg_purchase_vat_20_ptc_exempt"/>
            </record>
            <!-- 20% Payable Tax Credit [END] -->
        <!-- Purchase [END] -->
    </data>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="tax_group_vat_20" model="account.tax.group">
            <field name="name">20% VAT</field>
            <field name="country_id" ref="base.bg"/>
        </record>

        <record id="tax_group_vat_9" model="account.tax.group">
            <field name="name">9% VAT</field>
            <field name="country_id" ref="base.bg"/>
        </record>

        <record id="tax_group_vat_0" model="account.tax.group">
            <field name="name">0% VAT</field>
            <field name="country_id" ref="base.bg"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Sales -->
        <record id="l10n_bg_sale_vat_20" model="account.tax.template">
            <field name="sequence">101</field>
            <field name="name">20% VAT</field>
            <field name="description">20% VAT</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_11')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_21')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_11')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_21')]
                }),
            ]"/>
        </record>

        <record id="l10n_bg_sale_vat_20_remote" model="account.tax.template">
            <field name="sequence">102</field>
            <field name="name">20% Remote</field>
            <field name="description">20% Remote</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_11'), ref('l10n_bg_tax_report_18')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_21')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_11'), ref('l10n_bg_tax_report_18')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_21')]
                }),
            ]"/>
            <field name="active" eval="False"/>
        </record>

        <record id="l10n_bg_sale_vat_20_personal" model="account.tax.template">
            <field name="sequence">103</field>
            <field name="name">20% Personal use</field>
            <field name="description">20% Personal use</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_11')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_23')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_11')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_23')]
                }),
            ]"/>
            <field name="active" eval="False"/>
        </record>

        <record id="l10n_bg_sale_vat_9" model="account.tax.template">
            <field name="sequence">111</field>
            <field name="name">9% VAT</field>
            <field name="description">9% VAT</field>
            <field name="price_include" eval="0"/>
            <field name="amount">9</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_9"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_13')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_24')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_13')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_24')]
                }),
            ]"/>
        </record>

        <record id="l10n_bg_sale_vat_9_personal" model="account.tax.template">
            <field name="sequence">112</field>
            <field name="name">9% Personal use</field>
            <field name="description">9% Personal use</field>
            <field name="price_include" eval="0"/>
            <field name="amount">9</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_9"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_13')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_23')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_13')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_23')]
                }),
            ]"/>
            <field name="active" eval="False"/>
        </record>

        <record id="l10n_bg_sale_vat_0_export" model="account.tax.template">
            <field name="sequence">121</field>
            <field name="name">0% Export</field>
            <field name="description">0% Export</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_14')],
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
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_14')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="l10n_bg_sale_vat_0_icd" model="account.tax.template">
            <field name="sequence">122</field>
            <field name="name">0% ICD</field>
            <field name="description">0% Intra-Community Deliveries</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_15')],
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
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_15')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="l10n_bg_sale_vat_0_140" model="account.tax.template">
            <field name="sequence">123</field>
            <field name="name">0% Art. 140, 146, 173</field>
            <field name="description">0% Art. 140, 146, 173</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_16')],
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
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_16')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="active" eval="False"/>
        </record>

        <record id="l10n_bg_sale_vat_0_21" model="account.tax.template">
            <field name="sequence">124</field>
            <field name="name">0% Art. 21</field>
            <field name="description">0% Art. 21</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_17')],
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
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_17')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="active" eval="False"/>
        </record>

        <record id="l10n_bg_sale_vat_0_exempt" model="account.tax.template">
            <field name="sequence">125</field>
            <field name="name">0% Exempt</field>
            <field name="description">0% Exempt</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_19')],
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
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_19')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="l10n_bg_sale_vat_0_tri" model="account.tax.template">
            <field name="sequence">126</field>
            <field name="name">0% Tripartite</field>
            <field name="description">0% Tripartite</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_18')],
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
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_18')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="active" eval="False"/>
        </record>
    <!-- Sales [END] -->

    <!-- Purchases -->
        <record id="l10n_bg_purchase_vat_20_ftc" model="account.tax.template">
            <field name="sequence">201</field>
            <field name="name">20% FTC</field>
            <field name="description">20% Foreign Tax Credit</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_31')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_41')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_31')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_41')]
                }),
            ]"/>
        </record>

        <record id="l10n_bg_purchase_vat_20_ptc" model="account.tax.template">
            <field name="sequence">202</field>
            <field name="name">20% PTC</field>
            <field name="description">20% Payable Tax Credit</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_32')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_42')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_32')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_42')]
                }),
            ]"/>
        </record>

        <record id="l10n_bg_purchase_vat_20_otc" model="account.tax.template">
            <field name="sequence">203</field>
            <field name="name">20% OTC</field>
            <field name="description">20% Other Tax Credit</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_30')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_30')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_30')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_30')]
                }),
            ]"/>
        </record>

        <record id="l10n_bg_purchase_vat_20_ftc_exempt" model="account.tax.template">
            <field name="sequence">204</field>
            <field name="name">20% FTC (Exempt)</field>
            <field name="description">20% Foreign Tax Credit – Exempt</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_19'), ref('l10n_bg_tax_report_31')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_41')]
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_22')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_19'), ref('l10n_bg_tax_report_31')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_41')]
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_22')]
                }),
            ]"/>
        </record>

        <record id="l10n_bg_purchase_vat_20_ftc_ica" model="account.tax.template">
            <field name="sequence">204</field>
            <field name="name">20% FTC (ICA)</field>
            <field name="description">20% Foreign Tax Credit – Intra-Community Acquisition</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_12_1'), ref('l10n_bg_tax_report_31')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_41')]
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_22')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_12_1'), ref('l10n_bg_tax_report_31')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_41')]
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_22')]
                }),
            ]"/>
        </record>

        <record id="l10n_bg_purchase_vat_20_ftc_82" model="account.tax.template">
            <field name="sequence">205</field>
            <field name="name">20% FTC (Art. 82)</field>
            <field name="description">20% Foreign Tax Credit – Art. 82</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_12_2'), ref('l10n_bg_tax_report_31')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_41')]
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_22')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_12_2'), ref('l10n_bg_tax_report_31')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_41')]
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_22')]
                }),
            ]"/>
            <field name="active" eval="False"/>
        </record>

        <record id="l10n_bg_purchase_vat_20_ptc_exempt" model="account.tax.template">
            <field name="sequence">206</field>
            <field name="name">20% PTC (Exempt)</field>
            <field name="description">20% Payable Tax Credit – Exempt</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_19'), ref('l10n_bg_tax_report_32')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_42')]
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_22')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_19'), ref('l10n_bg_tax_report_32')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_42')]
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_22')]
                }),
            ]"/>
        </record>

        <record id="l10n_bg_purchase_vat_20_ptc_ica" model="account.tax.template">
            <field name="sequence">206</field>
            <field name="name">20% PTC (ICA)</field>
            <field name="description">20% Payable Tax Credit – Intra-Community Acquisition</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_12_1'), ref('l10n_bg_tax_report_32')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_42')]
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_22')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_12_1'), ref('l10n_bg_tax_report_32')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_42')]
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_22')]
                }),
            ]"/>
        </record>

        <record id="l10n_bg_purchase_vat_20_ptc_82" model="account.tax.template">
            <field name="sequence">207</field>
            <field name="name">20% PTC (Art. 82)</field>
            <field name="description">20% Payable Tax Credit - Art. 82</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_12_2'), ref('l10n_bg_tax_report_32')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_42')]
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_22')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_12_2'), ref('l10n_bg_tax_report_32')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_42')]
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4532'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_22')]
                }),
            ]"/>
            <field name="active" eval="False"/>
        </record>

        <record id="l10n_bg_purchase_vat_9_ftc" model="account.tax.template">
            <field name="sequence">211</field>
            <field name="name">9% FTC</field>
            <field name="description">9% Foreign Tax Credit</field>
            <field name="price_include" eval="0"/>
            <field name="amount">9</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_9"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_31')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_41')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_31')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_41')]
                }),
            ]"/>
        </record>

        <record id="l10n_bg_purchase_vat_9_otc" model="account.tax.template">
            <field name="sequence">212</field>
            <field name="name">9% OTC</field>
            <field name="description">9% Other Tax Credit</field>
            <field name="price_include" eval="0"/>
            <field name="amount">9</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_9"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_30')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_30')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_30')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_bg_4531'),
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_30')]
                }),
            ]"/>
        </record>

        <record id="l10n_bg_purchase_vat_0_otc_ica" model="account.tax.template">
            <field name="sequence">231</field>
            <field name="name">0% OTC (ICA)</field>
            <field name="description">0% Other Tax Credit – Intra-Community Acquisition</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_19'), ref('l10n_bg_tax_report_30')],
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
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_19'), ref('l10n_bg_tax_report_30')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="l10n_bg_purchase_vat_0_otc_exempt" model="account.tax.template">
            <field name="sequence">232</field>
            <field name="name">0% OTC (Exempt)</field>
            <field name="description">0% Other Tax Credit – Exempt</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_30')],
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
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_30')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="l10n_bg_purchase_vat_0_tri" model="account.tax.template">
            <field name="sequence">233</field>
            <field name="name">0% Tripartite</field>
            <field name="description">0% Tripartite</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10n_bg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('l10n_bg_tax_report_18')],
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
                    'minus_report_line_ids': [ref('l10n_bg_tax_report_18')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="active" eval="False"/>
        </record>
    <!-- Purchase [END] -->
</odoo>

```

## File: data\l10n_bg_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_bg_chart_template" model="account.chart.template">
        <field name="name">Bulgaria - National Chart of Accounts</field>
        <field name="property_account_receivable_id" ref="l10n_bg_411"/>
        <field name="property_account_payable_id" ref="l10n_bg_401"/>
        <field name="property_account_expense_categ_id" ref="l10n_bg_601"/>
        <field name="property_account_income_categ_id" ref="l10n_bg_701"/>
        <field name="expense_currency_exchange_account_id" ref="l10n_bg_624"/>
        <field name="income_currency_exchange_account_id" ref="l10n_bg_624"/>
        <field name="default_cash_difference_income_account_id" ref="l10n_bg_791001"/>
        <field name="default_cash_difference_expense_account_id" ref="l10n_bg_691001"/>
    </record>
</odoo>

```

## File: data\menuitem.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="bg_reports_menu" name="Bulgaria" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>
</odoo>

```

## File: data\tax_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_bg_tax_report" model="account.tax.report">
        <field name="name">Tax report</field>
        <field name="country_id" ref="base.bg"/>
    </record>

    <record id="l10n_bg_tax_report_a" model="account.tax.report.line">
        <field name="name">Section A: Data on value added tax charged</field>
        <field name="report_id" ref="l10n_bg_tax_report"/>
        <field name="sequence">100000</field>
    </record>

        <record id="l10n_bg_tax_report_01" model="account.tax.report.line">
            <field name="name">[01] Total amount of the tax bases for VAT taxation (amount from class 11 to class 16)</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_a"/>
            <field name="sequence">101000</field>
            <field name="code">BG_TR_01</field>
            <field name="formula">BG_TR_11 + BG_TR_12 + BG_TR_13 + BG_TR_14 + BG_TR_15 + BG_TR_16</field>
        </record>

        <record id="l10n_bg_tax_report_a_1" model="account.tax.report.line">
            <field name="name">Tax base subject to taxation at a rate of 20%</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_a"/>
            <field name="sequence">102000</field>
        </record>

            <record id="l10n_bg_tax_report_11" model="account.tax.report.line">
                <field name="name">[11] - tax base of taxable supplies, incl. deliveries under the conditions of distance sales with a place of performance on the territory of the country</field>
                <field name="report_id" ref="l10n_bg_tax_report"/>
                <field name="parent_id" ref="l10n_bg_tax_report_a_1"/>
                <field name="sequence">102100</field>
                <field name="code">BG_TR_11</field>
                <field name="tag_name">11</field>
            </record>

            <record id="l10n_bg_tax_report_12" model="account.tax.report.line">
                <field name="name">[12] - tax base of VAT and tax base of received supplies under Article 82, paragraphs 2-6 of the VAT Act</field>
                <field name="report_id" ref="l10n_bg_tax_report"/>
                <field name="parent_id" ref="l10n_bg_tax_report_a_1"/>
                <field name="sequence">102200</field>
                <field name="code">BG_TR_12</field>
                <field name="formula">BG_TR_12_1 + BG_TR_12_2</field>
            </record>

                <record id="l10n_bg_tax_report_12_1" model="account.tax.report.line">
                    <field name="name">Intra-community acquisitions</field>
                    <field name="report_id" ref="l10n_bg_tax_report"/>
                    <field name="parent_id" ref="l10n_bg_tax_report_12"/>
                    <field name="sequence">102210</field>
                    <field name="code">BG_TR_12_1</field>
                    <field name="tag_name">12_1</field>
                </record>

                <record id="l10n_bg_tax_report_12_2" model="account.tax.report.line">
                    <field name="name">Deliveries under Art. 82, para. 2-6</field>
                    <field name="report_id" ref="l10n_bg_tax_report"/>
                    <field name="parent_id" ref="l10n_bg_tax_report_12"/>
                    <field name="sequence">102220</field>
                    <field name="code">BG_TR_12_2</field>
                    <field name="tag_name">12_2</field>
                </record>

        <record id="l10n_bg_tax_report_13" model="account.tax.report.line">
            <field name="name">[13] Tax base of taxable supplies at a rate of 9%</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_a"/>
            <field name="sequence">103000</field>
            <field name="code">BG_TR_13</field>
            <field name="tag_name">13</field>
        </record>

        <record id="l10n_bg_tax_report_a_2" model="account.tax.report.line">
            <field name="name">Tax base subject to 0% tax</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_a"/>
            <field name="sequence">104000</field>
        </record>

            <record id="l10n_bg_tax_report_14" model="account.tax.report.line">
                <field name="name">[14] Tax base for supplies under Chapter Three of the VAT Act</field>
                <field name="report_id" ref="l10n_bg_tax_report"/>
                <field name="parent_id" ref="l10n_bg_tax_report_a_2"/>
                <field name="sequence">104100</field>
                <field name="code">BG_TR_14</field>
                <field name="tag_name">14</field>
            </record>

            <record id="l10n_bg_tax_report_15" model="account.tax.report.line">
                <field name="name">[15] Tax base of AEO of goods</field>
                <field name="report_id" ref="l10n_bg_tax_report"/>
                <field name="parent_id" ref="l10n_bg_tax_report_a_2"/>
                <field name="sequence">104200</field>
                <field name="code">BG_TR_15</field>
                <field name="tag_name">15</field>
            </record>

            <record id="l10n_bg_tax_report_16" model="account.tax.report.line">
                <field name="name">[16] Tax base of supplies under Articles 140, 146 and 173 of the VAT Act </field>
                <field name="report_id" ref="l10n_bg_tax_report"/>
                <field name="parent_id" ref="l10n_bg_tax_report_a_2"/>
                <field name="sequence">104300</field>
                <field name="code">BG_TR_16</field>
                <field name="tag_name">16</field>
            </record>

        <record id="l10n_bg_tax_report_17" model="account.tax.report.line">
            <field name="name">[17] Tax base for supplies of services under Article 21, paragraph 2 with a place of performance on the territory of another member state</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_a"/>
            <field name="sequence">105000</field>
            <field name="code">BG_TR_17</field>
            <field name="tag_name">17</field>
        </record>

        <record id="l10n_bg_tax_report_18" model="account.tax.report.line">
            <field name="name">[18] Tax base of supplies under Article 69, paragraph 2 of the VAT Act, incl. deliveries on the basis of distance selling with a place of performance in the territory of another Member State, as well as deliveries as an intermediary in a tripartite</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_a"/>
            <field name="sequence">106000</field>
            <field name="code">BG_TR_18</field>
            <field name="tag_name">18</field>
        </record>

        <record id="l10n_bg_tax_report_19" model="account.tax.report.line">
            <field name="name">[19] Tax base of exempt supplies and exempt VOP</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_a"/>
            <field name="sequence">107000</field>
            <field name="code">BG_TR_19</field>
            <field name="tag_name">19</field>
        </record>

        <record id="l10n_bg_tax_report_20" model="account.tax.report.line">
            <field name="name">[20] All VAT charged (amount from class 21 to class 24)</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_a"/>
            <field name="sequence">108000</field>
            <field name="code">BG_TR_20</field>
            <field name="formula">BG_TR_21 + BG_TR_22 + BG_TR_23 + BG_TR_24</field>
        </record>

        <record id="l10n_bg_tax_report_21" model="account.tax.report.line">
            <field name="name">[21] VAT charged</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_a"/>
            <field name="sequence">109000</field>
            <field name="code">BG_TR_21</field>
            <field name="tag_name">21</field>
        </record>

        <record id="l10n_bg_tax_report_22" model="account.tax.report.line">
            <field name="name">[22] VAT charged for VAT and for received deliveries under Art. 82, para 2-6</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_a"/>
            <field name="sequence">110000</field>
            <field name="code">BG_TR_22</field>
            <field name="tag_name">22</field>
        </record>

        <record id="l10n_bg_tax_report_23" model="account.tax.report.line">
            <field name="name">[23] Tax charged on supplies of goods and services for personal use</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_a"/>
            <field name="sequence">111000</field>
            <field name="code">BG_TR_23</field>
            <field name="tag_name">23</field>
        </record>

        <record id="l10n_bg_tax_report_24" model="account.tax.report.line">
            <field name="name">[24] VAT charged (9%)</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_a"/>
            <field name="sequence">112000</field>
            <field name="code">BG_TR_24</field>
            <field name="tag_name">24</field>
        </record>

    <record id="l10n_bg_tax_report_b" model="account.tax.report.line">
        <field name="name">Section B: Data on the exercised right to a tax credit</field>
        <field name="report_id" ref="l10n_bg_tax_report"/>
        <field name="sequence">200000</field>
    </record>

        <record id="l10n_bg_tax_report_30" model="account.tax.report.line">
            <field name="name">[30] Tax base and tax on the received deliveries, VAT, the received deliveries under art. 82, para. 2-6 of the VAT Act and imports without the right to a tax credit or without tax</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_b"/>
            <field name="sequence">201000</field>
            <field name="code">BG_TR_30</field>
            <field name="tag_name">30</field>
        </record>

        <record id="l10n_bg_tax_report_b_2" model="account.tax.report.line">
            <field name="name">Tax base of the received deliveries, VAT, the received deliveries under art. 82, para 2-6 of the VAT Act, the import, as well as the tax base of the received deliveries, used for making deliveries under art. 69, para 2 of the VAT Act</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_b"/>
            <field name="sequence">202000</field>
        </record>

            <record id="l10n_bg_tax_report_31" model="account.tax.report.line">
                <field name="name">[31] - entitled to a full tax credit</field>
                <field name="report_id" ref="l10n_bg_tax_report"/>
                <field name="parent_id" ref="l10n_bg_tax_report_b_2"/>
                <field name="sequence">202100</field>
                <field name="code">BG_TR_31</field>
                <field name="tag_name">31</field>
            </record>

            <record id="l10n_bg_tax_report_32" model="account.tax.report.line">
                <field name="name">[32] - with the right to a partial tax credit</field>
                <field name="report_id" ref="l10n_bg_tax_report"/>
                <field name="parent_id" ref="l10n_bg_tax_report_b_2"/>
                <field name="sequence">202200</field>
                <field name="code">BG_TR_32</field>
                <field name="tag_name">32</field>
            </record>

        <record id="l10n_bg_tax_report_33" model="account.tax.report.line">
            <field name="name">[33] Coefficient under Article 73, paragraph 5 of the VAT Act</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_b"/>
            <field name="sequence">203000</field>
            <field name="code">BG_TR_33</field>
            <field name="formula">(BG_TR_01 + BG_TR_17 + BG_TR_18 + BG_TR_19 != 0) and (BG_TR_11 + BG_TR_12_1 + BG_TR_13)/(BG_TR_01 + BG_TR_17 + BG_TR_18 + BG_TR_19)</field>
        </record>

        <record id="l10n_bg_tax_report_41" model="account.tax.report.line">
            <field name="name">[41] VAT eligible for a full tax credit</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_b"/>
            <field name="sequence">204000</field>
            <field name="code">BG_TR_41</field>
            <field name="tag_name">41</field>
        </record>

        <record id="l10n_bg_tax_report_42" model="account.tax.report.line">
            <field name="name">[42] VAT with the right to a partial tax credit</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_b"/>
            <field name="sequence">205000</field>
            <field name="code">BG_TR_42</field>
            <field name="tag_name">42</field>
        </record>

        <record id="l10n_bg_tax_report_43" model="account.tax.report.line">
            <field name="name">[43] Annual adjustment under Article 73, paragraph 8 (+/-)</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_b"/>
            <field name="sequence">206000</field>
            <field name="code">BG_TR_43</field>
            <field name="formula">BG_TR_42 * BG_TR_33</field>
        </record>

        <record id="l10n_bg_tax_report_40" model="account.tax.report.line">
            <field name="name">[40] Total tax credit (41 + 42 x class 33 + 43)</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_b"/>
            <field name="sequence">207000</field>
            <field name="code">BG_TR_40</field>
            <field name="formula">BG_TR_41 + (BG_TR_42 * BG_TR_33) + BG_TR_43</field>
        </record>

    <record id="l10n_bg_tax_report_c" model="account.tax.report.line">
        <field name="name">Section C: Result for the period</field>
        <field name="report_id" ref="l10n_bg_tax_report"/>
        <field name="sequence">300000</field>
    </record>

        <record id="l10n_bg_tax_report_50" model="account.tax.report.line">
            <field name="name">[50] VAT to be paid (class 20 - class 40) >= 0</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_c"/>
            <field name="sequence">301000</field>
            <field name="code">BG_TR_50</field>
            <field name="formula">max(BG_TR_20 - BG_TR_40, 0)</field>
        </record>

        <record id="l10n_bg_tax_report_60" model="account.tax.report.line">
            <field name="name">[60] VAT for refund (class 20 - class 40) &lt; 0</field>
            <field name="report_id" ref="l10n_bg_tax_report"/>
            <field name="parent_id" ref="l10n_bg_tax_report_c"/>
            <field name="sequence">302000</field>
            <field name="code">BG_TR_60</field>
            <field name="formula">min(BG_TR_20 - BG_TR_40, 0)</field>
        </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106"><defs><mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse"><path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill:#fff;fill-rule:evenodd"/></mask><mask id="b" x="4.75" y="6.67" width="52.3" height="32.38" maskUnits="userSpaceOnUse"><rect x="6.19" y="7.38" width="48.45" height="31.57" rx="1" style="fill:#fff"/></mask><symbol id="c" viewBox="0 0 106 106"><g style="mask:url(#a)"><path d="M0,0H106V106H0Z" style="fill:#5a5a64;fill-rule:evenodd"/><path d="M6.06,1.51H98.43q6.06,0,7.57,3V0H0V4.54Q1.52,1.51,6.06,1.51Z" style="fill:#fff;fill-opacity:0.382999986410141;fill-rule:evenodd"/><path d="M6.06,104.49H98.43q6.06,0,7.57-4.55V106H0V99.94Q1.52,104.49,6.06,104.49Z" style="fill-opacity:0.382999986410141;fill-rule:evenodd"/><path d="M70.38,104.49H6.06C3,104.49,0,103,0,98.43V61.28L28.77,19.69H59.06a77.33,77.33,0,0,0,21.2,13.87c.07,11.31.07,4.86,0,16.17h3.12l.21,36.82Z" style="fill:#393939;fill-rule:evenodd;isolation:isolate;opacity:0.324000000953674"/><g style="opacity:0.30000000000000004"><path d="M68.77,58.54H76c.76,0,1,.12,1,.46v2.45c0,.31-.24.43-.93.43H61.44c-.66,0-.92-.12-.92-.42,0-.83,0-1.67,0-2.51,0-.29.26-.4.92-.41Z"/><path d="M64.33,77.42c.42.39.76.66,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,4.25,4.25,0,0,1-.48-.47c-.14-.15-.26-.31-.49-.6-.32.37-.54.66-.79.91-.53.53-1.08.58-1.5.15s-.36-.94.15-1.45c.26-.26.54-.5.91-.83-.38-.34-.72-.61-1-.91a.9.9,0,0,1,0-1.36.91.91,0,0,1,1.36,0c.29.28.54.6.93,1A12.1,12.1,0,0,1,64,75.18a.91.91,0,0,1,1.36,0,.87.87,0,0,1,0,1.31C65.07,76.79,64.73,77.06,64.33,77.42Z"/><path d="M62.13,66.9c0-.47,0-.88,0-1.28a.92.92,0,0,1,.92-1,.91.91,0,0,1,1,1c0,.41,0,.81,0,1.3h1.14a1.16,1.16,0,0,1,1.22,1c0,.55-.42.85-1.18.86H64.12c0,.49,0,.91,0,1.34a.94.94,0,1,1-1.88,0c0-.41,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88C61.3,66.89,61.68,66.9,62.13,66.9Z"/><path d="M74.31,76H72.23c-.67,0-1-.34-1-.93a.89.89,0,0,1,1-1q2.18,0,4.35,0a1,1,0,1,1,0,1.91c-.74,0-1.47,0-2.21,0Z"/><path d="M74.28,68.61c-.71,0-1.43,0-2.14,0a.86.86,0,0,1-1-.9.85.85,0,0,1,.92-1c1.5,0,3,0,4.48,0a.93.93,0,0,1,1,1,.91.91,0,0,1-1,.91c-.75,0-1.51,0-2.27,0Z"/><path d="M74.36,78.09c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.57-.38.93-1,.94H72.28c-.75,0-1.09-.32-1.09-.94s.37-1,1.09-1,1.39,0,2.08,0Z"/><path d="M81.29,90.55H56.14a4,4,0,0,1-4-4V53.73a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V86.55A4,4,0,0,1,81.29,90.55ZM56.14,53.73V86.55H81.29V53.73Z"/><path d="M43.49,83.26H31.8V25.71H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V34.8c-4.55-3-16.66-12.11-19.69-13.63H30.29a2.68,2.68,0,0,0-3,3V84.77a2.68,2.68,0,0,0,3,3H48.45V83.26ZM60.57,25.71l15.14,10.6H60.57Z"/></g><path d="M60.57,18.68H30.29a2.68,2.68,0,0,0-3,3V82.28a2.68,2.68,0,0,0,3,3H48.45V80.77H31.8V23.22H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V32.31C75.71,29.28,63.6,20.2,60.57,18.68Zm0,15.14V23.22l15.14,10.6Z" style="fill:#a8a9ab"/><path d="M68.77,55.78H76c.76,0,1,.13,1,.53v2.85c0,.37-.24.5-.93.5q-7.3,0-14.61,0c-.66,0-.92-.14-.92-.48,0-1,0-2,0-2.93,0-.34.26-.47.92-.47Z" style="fill:#a8a9ab"/><path d="M64.33,76.53c.42.38.76.65,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,5.44,5.44,0,0,1-.48-.48c-.14-.14-.26-.31-.49-.59-.32.36-.54.65-.79.91-.53.53-1.08.57-1.5.14s-.36-.94.15-1.45c.26-.26.54-.49.91-.82-.38-.35-.72-.61-1-.92a.9.9,0,0,1,0-1.36.92.92,0,0,1,1.36,0c.29.28.54.61.93,1A13.78,13.78,0,0,1,64,74.28a.91.91,0,0,1,1.36,0,.88.88,0,0,1,0,1.32C65.07,75.89,64.73,76.16,64.33,76.53Z" style="fill:#a8a9ab"/><path d="M62.13,65.88c0-.48,0-.88,0-1.29a1,1,0,1,1,1.91,0c0,.4,0,.81,0,1.3h1.14a1.15,1.15,0,0,1,1.22,1c0,.54-.42.85-1.18.85H64.12c0,.49,0,.92,0,1.34a.94.94,0,1,1-1.88,0c0-.4,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88Z" style="fill:#a8a9ab"/><path d="M74.31,75.11c-.69,0-1.38,0-2.08,0s-1-.35-1-.94a.89.89,0,0,1,1-1q2.18,0,4.35,0a.91.91,0,0,1,1,1,.93.93,0,0,1-1,1c-.74,0-1.47,0-2.21,0Z" style="fill:#a8a9ab"/><path d="M74.28,67.76H72.14a.87.87,0,0,1-1-.9.84.84,0,0,1,.92-1c1.5,0,3,0,4.48,0a.94.94,0,0,1,1,1,.91.91,0,0,1-1,.91H74.28Z" style="fill:#a8a9ab"/><path d="M74.36,77.2c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.56-.38.93-1,.93q-2.12,0-4.23,0c-.75,0-1.09-.32-1.09-.94s.37-.94,1.09-1,1.39,0,2.08,0Z" style="fill:#a8a9ab"/><path d="M81.29,88.06H56.14a4,4,0,0,1-4-4V51.24a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V84.06A4,4,0,0,1,81.29,88.06ZM56.14,51.24V84.06H81.29V51.24Z" style="fill:#a8a9ab"/></g></symbol></defs><use width="106" height="106" transform="translate(-0.07 0)" xlink:href="#c"/><rect x="6.2" y="10.57" width="48.45" height="31.57" rx="1" style="fill:#393939;isolation:isolate;opacity:0.44"/><g style="mask:url(#b)"><rect x="6.15" y="6.67" width="50.9" height="30.54" style="fill:#fff"/><rect x="4.75" y="18.13" width="52.3" height="20.92" style="fill:#00966e"/><rect x="4.75" y="28.59" width="52.3" height="10.46" style="fill:#d62612"/></g></svg>
```

