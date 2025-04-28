# Odoo Module: l10n_lu

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, SUPERUSER_ID

from . import models

def _post_init_hook(cr, registry):
    _preserve_tag_on_taxes(cr, registry)
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref('l10n_lu.lu_2011_chart_1').process_coa_translations()

def _preserve_tag_on_taxes(cr, registry):
    from odoo.addons.account.models.chart_template import preserve_existing_tags_on_taxes
    preserve_existing_tags_on_taxes(cr, registry, 'l10n_lu')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2011 Thamini S.à.R.L (<http://www.thamini.com>)
# Copyright (C) 2011 ADN Consultants S.à.R.L (<http://www.adn-luxembourg.com>)
#    Copyright (C) 2014 ACSONE SA/NV (<http://acsone.eu>)

{
    'name': 'Luxembourg - Accounting',
    'version': '2.0',
    'category': 'Localization',
    'description': """
This is the base module to manage the accounting chart for Luxembourg.
======================================================================

    * the Luxembourg Official Chart of Accounts (law of June 2009 + 2015 chart and Taxes),
    * the Tax Code Chart for Luxembourg
    * the main taxes used in Luxembourg
    * default fiscal position for local, intracom, extracom

Notes:
    * the 2015 chart of taxes is implemented to a large extent,
      see the first sheet of tax.xls for details of coverage
    * to update the chart of tax template, update tax.xls and run tax2csv.py
""",
    'author': 'OpenERP SA, ADN, ACSONE SA/NV',
    'depends': [
        'account',
        'base_iban',
        'base_vat',
        'l10n_multilang',
    ],
    'data': [
        # basic accounting data
        'data/l10n_lu_chart_data.xml',
        'data/account.account.template.csv',
        'data/account_tax_report_line.xml',
        'data/account.tax.group.csv',
        'data/account_tax_template_2015.xml',
        'data/account.fiscal.position.template-2011.csv',
        'data/account.fiscal.position.tax.template-2015.csv',
        'data/account_reconcile_model_template_data.xml',
        # configuration wizard, views, reports...
        'data/account.chart.template.csv',
        'data/account_chart_template_data.xml',
    ],
    'post_init_hook': '_post_init_hook',
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,code,name,user_type_id/id,reconcile,chart_template_id:id
lu_2011_account_101,101,Subscribed capital,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_102,102,Subscribed capital not called,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_103,103,Subscribed capital called but unpaid,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_104,104,"Capital of individual companies, corporate partnerships and similar",account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_105,105,Endowment of branches,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_10611,10611,Cash withdrawals (daily life),account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_10612,10612,"Withdrawals of merchandise, finished products and services (at cost)",account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_10613,10613,Private share of medical services expenses,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106141,106141,Life insurance,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106142,106142,Accident insurance,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106143,106143,Fire insurance,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106144,106144,Third-party insurance,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106145,106145,Full coverage insurance,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106148,106148,Other private insurance premiums,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106151,106151,Social Security,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106152,106152,Child benefit office,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106153,106153,Health insurance funds,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106154,106154,Death and other health insurance funds,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106158,106158,Other contributions,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106161,106161,Wages,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106162,106162,Rent,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106163,106163,"Heating, gas, electricity",account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106164,106164,Water,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106165,106165,Telephone,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106166,106166,Car,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106168,106168,Other in kind withdrawals,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106171,106171,Private furniture,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106172,106172,Private car,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106173,106173,Private held securities,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106174,106174,Private buildings,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106178,106178,Other acquisitions,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106181,106181,Income tax paid,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106183,106183,Municipal business tax - payment in arrears,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106188,106188,Other taxes,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106191,106191,Repairs to private buildings,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106192,106192,Deposits on private financial accounts,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106193,106193,Refund of private debts,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106194,106194,Gifts and allowance to children,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106195,106195,Inheritance taxes and mutation tax due to death,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106198,106198,Other special private withdrawals,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_10621,10621,Inheritance or donation,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_10622,10622,Personal holdings,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_10623,10623,Private loans,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106241,106241,Private furniture,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106242,106242,Private car,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106243,106243,Private shares / bonds,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106244,106244,Private buildings,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106248,106248,Other disposals,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_10625,10625,Received rents,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_10626,10626,Received wages or pensions,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_10627,10627,Received child benefit,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106281,106281,Income tax,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106284,106284,Municipal business tax (MBT),account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_106288,106288,Other tax refunds,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_10629,10629,Business share in private expenses,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_111,111,Share premium,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_112,112,Merger premium,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_113,113,Contribution premium,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_114,114,Premiums on conversion of bonds into shares,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_115,115,Capital contribution without issue of shares,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_122,122,Reserves in application of the equity method,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_123,123,Temporarily not taxable currency translation adjustments,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_128,128,Other revaluation reserves,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_131,131,Legal reserve,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_132,132,Reserves for own shares or own corporate units,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_133,133,Reserves provided for by the articles of association,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_1381,1381,Other reserves available for distribution,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2020_account_13821,13821,Reserve for net wealth tax (NWT),account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2020_account_13822,13822,Reserves in application of fair value,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2020_account_138231,138231,Temporarily not taxable capital gains to reinvest,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2020_account_138232,138232,Temporarily not taxable capital gains reinvested,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2020_account_13828,13828,Reserves not available for distribution not mentioned above,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2020_account_1411,1411,Results brought forward in the process of assignment,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2020_account_1412,1412,Results brought forward (assigned),account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_142,142,Result for the financial year,account.data_unaffected_earnings,FALSE,lu_2011_chart_1
lu_2011_account_15,15,Interim dividends,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2020_account_1611,1611,Development costs,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2020_account_16121,16121,Acquired against payment (except Goodwill),account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2020_account_16122,16122,Created by the undertaking itself,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2020_account_1613,1613,Goodwill acquired for consideration,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2020_account_1621,1621,"Subsidies on land, fitting-outs and buildings",account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2020_account_1622,1622,Subsidies on plant and machinery,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2020_account_1623,1623,"Subsidies on other fixtures, fittings, tools and equipment (including rolling stock)",account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_168,168,Other capital investment subsidies,account.data_account_type_equity,FALSE,lu_2011_chart_1
lu_2011_account_181,181,Provisions for pensions and similar obligations,account.data_account_type_non_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_182,182,Provisions for taxation,account.data_account_type_non_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_183,183,Deferred tax provisions,account.data_account_type_non_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_1881,1881,Operating provisions,account.data_account_type_non_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_1882,1882,Financial provisions,account.data_account_type_non_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_1921,1921,Due and payable within one year,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_1922,1922,Due and payable after more than one year,account.data_account_type_non_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_1931,1931,Due and payable within one year,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_1932,1932,Due and payable after more than one year,account.data_account_type_non_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_1941,1941,Due and payable within one year,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_1942,1942,Due and payable after more than one year,account.data_account_type_non_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_201,201,Set-up and start-up costs,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_203,203,"Expenses for increases in capital and for various operations (merger, demerger, change of legal form)",account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_204,204,Loan issuances expenses,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_208,208,Other similar expenses,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_211,211,Development costs,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_21211,21211,Concessions,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_21212,21212,Patents,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_21213,21213,Software licences,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_21214,21214,Trademarks and franchises,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_212151,212151,Copyrights and reproduction rights,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_212152,212152,Greenhouse gas and similar emission quotas,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_212158,212158,Other similar rights and assets acquired for consideration,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_21221,21221,Concessions,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_21222,21222,Patents,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_21223,21223,Software licences,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_21224,21224,Trademarks and franchises,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_212251,212251,Copyrights and reproduction rights,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_212258,212258,Other similar rights and assets created by the undertaking itself,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_213,213,Goodwill acquired for consideration,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_214,214,Down payments and intangible fixed assets under development,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_221111,221111,Developed land,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_221112,221112,Property rights and similar,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_221118,221118,Other land,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_22112,22112,Land in foreign countries,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_22121,22121,Fixtures and fitting-outs of land in Luxembourg,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_22122,22122,Fixtures and fitting-outs of land in foreign countries,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_221311,221311,Residential buildings,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_221312,221312,Non-residential buildings,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_221313,221313,Mixed-use buildings,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_221318,221318,Other buildings,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_22132,22132,Buildings in foreign countries,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_22141,22141,Fixtures and fitting-outs of buildings in Luxembourg,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_22142,22142,Fixtures and fitting-outs of buildings in foreign countries,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_22151,22151,Investment properties in Luxembourg,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_22152,22152,Investment properties in foreign countries,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_2221,2221,Plant,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_2222,2222,Machinery,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_2231,2231,Transportation and handling equipment,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_2232,2232,Motor vehicles,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_2233,2233,Tools,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_2234,2234,Furniture,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_2235,2235,Computer equipment,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_2236,2236,Livestock,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_2237,2237,Returnable packaging,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_2238,2238,Other fixtures,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_22411,22411,"Land, fitting-outs and buildings in Luxembourg",account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_22412,22412,"Land, fitting-outs and buildings in foreign countries",account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_2242,2242,Plant and machinery,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_2243,2243,"Other fixtures and fittings, tools and equipment (including rolling stock)",account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_231,231,Shares in affiliated undertakings,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_232,232,Amounts owed by affiliated undertakings,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_233,233,Participating interests,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_234,234,Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_235111,235111,Listed shares,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_235112,235112,Unlisted shares,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_23518,23518,Other securities held as fixed assets (equity right),account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_23521,23521,Debentures,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_23528,23528,Other securities held as fixed assets (creditor's right),account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_2353,2353,Shares of collective investment funds,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_2358,2358,Other securities held as fixed assets,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_2361,2361,Loans,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2020_account_2362,2362,Deposits and guarantees paid,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_2363,2363,Long-term receivables,account.data_account_type_fixed_assets,FALSE,lu_2011_chart_1
lu_2011_account_301,301,Inventories of raw materials,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_303,303,Inventories of consumable materials and supplies,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_304,304,Inventories of packaging,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_311,311,Inventories of work in progress,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_312,312,Contracts in progress - goods,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_313,313,Contracts in progress - services,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_314,314,Buildings under construction,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_321,321,Inventories of finished goods,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_322,322,Inventories of semi-finished goods,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_323,323,"Inventories of residual goods (waste, rejected and recuperable material)",account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_361,361,Inventories of merchandise,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_3621,3621,Inventories of land for resale in Luxembourg,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_3622,3622,Inventories of land for resale in foreign countries,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_3631,3631,Inventories of buildings for resale in Luxembourg,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_3632,3632,Inventories of buildings for resale in foreign countries,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_37,37,Down payments on account on inventories,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_4011,4011,Customers,account.data_account_type_receivable,TRUE,lu_2011_chart_1
lu_2011_account_40111,40111,Customers (PoS),account.data_account_type_receivable,TRUE,lu_2011_chart_1
lu_2011_account_4012,4012,Customers - Receivable bills of exchange,account.data_account_type_current_assets,TRUE,lu_2011_chart_1
lu_2011_account_4013,4013,Doubtful or disputed customers,account.data_account_type_current_assets,TRUE,lu_2011_chart_1
lu_2011_account_4014,4014,Customers - Unbilled sales,account.data_account_type_current_assets,TRUE,lu_2011_chart_1
lu_2011_account_4015,4015,Customers with a credit balance,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2011_account_4019,4019,Value adjustments,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_4021,4021,Customers,account.data_account_type_receivable,TRUE,lu_2011_chart_1
lu_2011_account_4025,4025,Customers with creditor balance,account.data_account_type_current_assets,TRUE,lu_2011_chart_1
lu_2011_account_4029,4029,Value adjustments,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41111,41111,Trade receivables,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41112,41112,Loans and advances,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41118,41118,Other receivables,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41119,41119,Value adjustments,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41121,41121,Trade receivables,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41122,41122,Loans and advances,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41128,41128,Other receivables,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41129,41129,Value adjustments,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41211,41211,Trade receivables,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41212,41212,Loans and advances,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41218,41218,Other receivables,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41219,41219,Value adjustments,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41221,41221,Trade receivables,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41222,41222,Loans and advances,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41228,41228,Other receivables,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_41229,41229,Value adjustments,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42111,42111,Advances and down payments,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42119,42119,Value adjustments,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_4212,4212,Amounts owed by partners and shareholders (others than from affiliated undertakings),account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42131,42131,Investment subsidies,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42132,42132,Operating subsidies,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42138,42138,Other subsidies,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_42141,42141,Corporate income tax,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_42142,42142,Municipal business tax,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_42143,42143,Net wealth tax,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_42144,42144,Withholding tax on wages and salaries,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_42145,42145,Withholding tax on financial investment income,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_42146,42146,Withholding tax on director's fees,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_42148,42148,ACD - Other amounts receivable,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_4215,4215,Customs and Excise Authority (ADA),account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_421611,421611,VAT paid and recoverable,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_421612,421612,VAT receivable,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_421613,421613,VAT down payments made,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_421618,421618,VAT - Other receivables,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_421621,421621,Registration duties,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_421622,421622,Subscription tax,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_421628,421628,Other indirect taxes,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42168,42168,Other receivables,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42171,42171,Social Security office (CCSS),account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42172,42172,Foreign social security offices,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42178,42178,Other social bodies,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_421811,421811,Foreign VAT,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_421818,421818,Other foreign taxes,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_42187,42187,Derivative financial instruments,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42188,42188,Other miscellaneous receivables,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42189,42189,Value adjustments,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_4221,4221,Staff - advances and down payments,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_4222,4222,Amounts owed by partners and shareholders (others than from affiliated undertakings),account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42231,42231,Investment subsidies,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42232,42232,Operating subsidies,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42238,42238,Other subsidies,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_42287,42287,Derivative financial instruments,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42288,42288,Other miscellaneous receivables,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_42289,42289,Value adjustments,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_431,431,Down payments received within one year,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_432,432,Down payments received after more than one year,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_44111,44111,Suppliers,account.data_account_type_payable,TRUE,lu_2011_chart_1
lu_2011_account_44112,44112,Suppliers - invoices not yet received,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2020_account_44113,44113,Suppliers with a debit balance,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2011_account_44121,44121,Suppliers,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2020_account_44123,44123,Suppliers with a debit balance,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2011_account_4421,4421,Bills of exchange payable within one year,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_4422,4422,Bills of exchange payable after more than one year,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_45111,45111,Purchases and services,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_45112,45112,Loans and advances,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_45118,45118,Other payables,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_45121,45121,Purchases and services,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_45122,45122,Loans and advances,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_45128,45128,Other payables,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_45211,45211,Purchases and services,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_45212,45212,Loans and advances,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_45218,45218,Other payables,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_45221,45221,Purchases and services,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_45222,45222,Loans and advances,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_45228,45228,Other payables,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_4611,4611,Municipal authorities,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2011_account_461211,461211,Corporate income tax - Tax accrual,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_461212,461212,CIT - Tax payable,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_461221,461221,MBT - Tax accrual,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_461222,461222,MBT - Tax payable,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_461231,461231,NWT - Tax accrual,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_461232,461232,NWT - Tax payable,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_46124,46124,Withholding tax on wages and salaries,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_46125,46125,Withholding tax on financial investment income,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_46126,46126,Withholding tax on director's fees,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_46128,46128,ACD - Other amounts payable,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_4613,4613,Customs and Excise Authority (ADA),account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2020_account_461411,461411,VAT received,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2011_account_461412,461412,VAT payable,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_461413,461413,VAT down payments received,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_461418,461418,VAT - Other payables,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_461421,461421,Registration duties,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_461422,461422,Subscription tax,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_461428,461428,Other indirect taxes,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_46148,46148,AED - Other debts,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_46151,46151,Foreign VAT,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_46158,46158,Other foreign taxes,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_4621,4621,Social Security office (CCSS),account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_4622,4622,Foreign Social Security offices,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_4628,4628,Other social bodies,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_4711,4711,Received deposits and guarantees,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2020_account_4712,4712,Amounts payable to partners and shareholders (others than from affiliated undertakings),account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2011_account_4713,4713,"Amounts payable to directors, managers, statutory auditors and similar",account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_4714,4714,Amounts payable to staff,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2011_account_4715,4715,State - Greenhous gas and similar emission quotas to be returned or acquired,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_47161,47161,Other loans,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2020_account_47162,47162,Lease debts,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2020_account_47163,47163,Life annuities,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2020_account_47164,47164,Other similar debts,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2020_account_4717,4717,Derivative financial instruments,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2011_account_4718,4718,Other miscellaneous debts,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_4721,4721,Received deposits and guarantees,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2020_account_4722,4722,Amounts payable to partners and shareholders (others than from affiliated undertakings),account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2011_account_4723,4723,"Amounts payable to directors, managers, statutory auditors and similar",account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2020_account_4724,4724,Amounts payable to staff,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2020_account_47261,47261,Other loans,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2020_account_47262,47262,Lease debts,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2020_account_47263,47263,Life annuities,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2020_account_47264,47264,Other similar debts,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2020_account_4727,4727,Derivative financial instruments,account.data_account_type_current_liabilities,TRUE,lu_2011_chart_1
lu_2011_account_4728,4728,Other miscellaneous debts,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_481,481,Deferred charges (on one or more financial years),account.data_account_type_prepayments,FALSE,lu_2011_chart_1
lu_2011_account_482,482,Deferred income (on one or more financial years),account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_483,483,State - Greenhouse gas and similar emission quotas received,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_484,484,Transitory or suspense accounts - Assets,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_485,485,Transitory or suspense accounts - Liabilities,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_486,486,Linking accounts (branches) - Assets,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_487,487,Linking accounts (branches) - Liabilities,account.data_account_type_current_liabilities,FALSE,lu_2011_chart_1
lu_2011_account_501,501,Shares in affiliated undertakings,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_502,502,Own shares or own corporate units,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_503,503,Shares in undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_5081,5081,Shares - listed securities,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_5082,5082,Shares - unlisted securities,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_5083,5083,Debenture loans and other notes issued and repurchased by the company,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_5084,5084,Listed debenture loans,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_5085,5085,Unlisted debenture loans,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2011_account_5088,5088,Other miscellaneous transferable securities,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_5131,5131,Banks and CCP : available balance,account.data_account_type_liquidity,TRUE,lu_2011_chart_1
lu_2020_account_5132,5132,Banks and CCP : overdraft,account.data_account_type_liquidity,TRUE,lu_2011_chart_1
lu_2020_account_516,516,Cash in hand,account.data_account_type_liquidity,TRUE,lu_2011_chart_1
lu_2020_account_5171,5171,Internal transfers : debit balance,account.data_account_type_liquidity,TRUE,lu_2011_chart_1
lu_2020_account_5172,5172,Internal transfers : credit balance,account.data_account_type_liquidity,TRUE,lu_2011_chart_1
lu_2011_account_518,518,Other cash amounts,account.data_account_type_current_assets,FALSE,lu_2011_chart_1
lu_2020_account_601,601,Purchases of raw materials,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_60311,60311,Solid fuels,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_60312,60312,Liquid fuels,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_60313,60313,Gas,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_60314,60314,Water and sewage,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_60315,60315,Electricity,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6032,6032,Maintenance supplies,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6033,6033,"Workshop, factory and store supplies and small equipment",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6034,6034,Work clothes,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6035,6035,Office and administrative supplies,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6036,6036,Motor fuels,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6037,6037,Lubricants,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6038,6038,Other consumable supplies,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_604,604,Purchases of packaging,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6061,6061,Purchases of merchandise,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6062,6062,Purchases of land for resale,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6063,6063,Purchases of buildings for resale,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6071,6071,Changes in inventory of raw materials,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6073,6073,Changes in inventory of consumable materials and supplies,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6074,6074,Changes in inventory of packaging,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_60761,60761,Merchandise,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_60762,60762,Land for resale,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_60763,60763,Buildings for resale,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_60811,60811,Tailoring,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_60812,60812,Research and development,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_60813,60813,Architects' and engineers' fees,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_60814,60814,Outsourcing included in the production of goods and services,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6082,6082,Other purchases of material included in the production of goods and services,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6083,6083,Purchase of greenhouse gas and similar emission quotas,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6088,6088,Other purchases included in the production of goods and services,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6091,6091,RDR on purchases of raw materials,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6093,6093,RDR on purchases of consumable materials and supplies,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6094,6094,RDR on purchases of packaging,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6096,6096,RDR on purchases of merchandise and other goods for resale,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6098,6098,RDR on purchases included in the production of goods and services,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6099,6099,Unallocated RDR,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61111,61111,Land,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61112,61112,Buildings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61123,61123,Rolling stock,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_61128,61128,Other,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6113,6113,Service charges and co-ownership expenses,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6114,6114,Financial leasing on real property,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61153,61153,Rolling stock,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_61158,61158,Other,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6121,6121,General subcontracting (not included in the production of goods and services),account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61221,61221,Buildings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61223,61223,Rolling stock,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_61228,61228,Other,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6131,6131,Commissions and brokerage fees,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6132,6132,IT services,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61332,61332,Loans' issuance expenses,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61333,61333,Bank account charges and bank commissions (included custody fees on securities),account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61334,61334,Charges for electronic means of payment,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61336,61336,Factoring services,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61338,61338,Other banking and similar services (except interest and similar expenses),account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61341,61341,"Legal, litigation and similar fees",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61342,61342,"Accounting, tax consulting, auditing and similar fees",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61348,61348,Other professional fees,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6135,6135,Notarial and similar fees,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6138,6138,Other remuneration of intermediaries and professional fees,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61411,61411,Buildings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61412,61412,Rolling stock,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61418,61418,Other,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6142,6142,Insurance on rented assets,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6143,6143,Transport insurance,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6144,6144,Business risk insurance,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6145,6145,Customers credit insurance,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6146,6146,Third-party insurance,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6148,6148,Other insurances,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61511,61511,Press advertising,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61512,61512,Samples,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61513,61513,Fairs and exhibitions,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61514,61514,Gifts to customers,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61515,61515,"Catalogues, printed materials and publications",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61516,61516,Donations,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61517,61517,Sponsorship,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61518,61518,Other purchases of advertising services,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_615211,615211,Management (respectively owner and partner),account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_615212,615212,Staff,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61522,61522,Relocation expenses,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61523,61523,Business assignments,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61524,61524,Receptions and entertainment costs,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61531,61531,Postal charges,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_61532,61532,Telecommunication costs,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6161,6161,Transportation of purchased goods,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6162,6162,Transportation of sold goods,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6165,6165,Collective staff transportation,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6168,6168,Other transportation,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6171,6171,Temporary staff,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6172,6172,External staff on secondment,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6181,6181,Documentation,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6182,6182,"Costs of training, symposiums, seminars, conferences",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6183,6183,Industrial and non-industrial waste treatment,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_61841,61841,Solid fuels,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_61842,61842,"Liquid fuels (oil, motor fuel, etc.)",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_61843,61843,Gas,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_61844,61844,Water and waste water,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_61845,61845,Electricity,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_61851,61851,Office supplies,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_61852,61852,Small equipment,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_61853,61853,Work clothes,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_61854,61854,Maintenance supplies,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_61858,61858,Other,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6186,6186,Surveillance and security charges,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6187,6187,Contributions to professional associations,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6188,6188,Other miscellaneous external charges,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_619,619,"Rebates, discounts and refunds received on other external charges",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_62111,62111,Base wages,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_621121,621121,Sunday,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_621122,621122,Public holidays,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_621123,621123,Overtime,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_621128,621128,Other supplements,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_62114,62114,"Incentives, bonuses and commissions",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_62115,62115,Benefits in kind,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_62116,62116,Severance pay,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_62117,62117,Survivor's pay,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6218,6218,Other benefits,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6219,6219,Refunds on wages paid,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6221,6221,Students,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6222,6222,Casual workers,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6228,6228,Other,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6231,6231,Social security on pensions,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6232,6232,"Other social security costs (including illness, accidents, a.s.o.)",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_62411,62411,Premiums for external pensions funds,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_62412,62412,Changes to provisions for complementary pensions,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_62413,62413,Withholding tax on complementary pensions,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_62414,62414,Insolvency insurance premiums,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_62415,62415,Complementary pensions paid by the employer,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6248,6248,Other staff expenses not mentioned above,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6311,6311,AVA on set-up and start-up costs,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6313,6313,"AVA on expenses for capital increases and various operations (mergers, demergers, changes of legal form)",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6314,6314,AVA on loan-issuance expenses,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6318,6318,AVA on other similar expenses,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6321,6321,AVA on development costs,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6322,6322,"AVA on concessions, patents, licences, trademarks and similar rights and assets",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6323,6323,AVA on goodwill acquired for consideration,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6324,6324,AVA on down payments and intangible fixed assets under development,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_63311,63311,AVA on land,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_63312,63312,AVA on fixtures and fittings-out of land,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_63313,63313,AVA on buildings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_63314,63314,AVA on fixtures and fittings-out of buildings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_63315,63315,FVA on investment properties,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6332,6332,AVA on plant and machinery,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6333,6333,"AVA on other fixtures and fittings, tools and equipment (including rolling stock)",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6334,6334,AVA on down payments and tangible fixed assets under development,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6341,6341,AVA on inventories of raw materials and consumables,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6342,6342,AVA on inventories of work and contracts in progress,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6343,6343,AVA on inventories of goods,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6344,6344,AVA on inventories of merchandise and other goods for resale,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6345,6345,AVA on down payments on inventories,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6351,6351,AVA on trade receivables,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6352,6352,AVA on amounts owed by affiliated undertakings and undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6353,6353,AVA on other receivables,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6354,6354,FVA on receivables from current assets,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6411,6411,Concessions,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6412,6412,Patents,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6413,6413,Software licences,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6414,6414,Trademarks and franchise,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_64151,64151,Copyrights and reproduction rights,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_64158,64158,Other similar rights and assets,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_642,642,"Indemnities, damages and interest",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6431,6431,Attendance fees,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6432,6432,"Director's fees",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6438,6438,Other similar remuneration,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_64411,64411,Book value of yielded intangible fixed assets,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_64412,64412,Disposal proceeds of intangible fixed assets,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_64421,64421,Book value of yielded tangible fixed assets,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_64422,64422,Disposal proceeds of tangible fixed assets,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6451,6451,Trade receivables,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6452,6452,Amounts owed by affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6453,6453,Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6454,6454,Other receivables,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6461,6461,Real property tax,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6462,6462,Non-refundable VAT,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6463,6463,Duties on imported merchandise,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6464,6464,Excise duties on production and tax on consumption,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_64651,64651,Registration fees,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_64658,64658,"Other registration fees, stamp duties and mortgage duties",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6466,6466,Motor-vehicle taxes,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6467,6467,Bar licence tax,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6468,6468,Other duties and taxes,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_647,647,Allocations to tax-exempt capital gains,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6481,6481,"Fines, sanctions and penalties",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6488,6488,Miscellaneous operating charges,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6491,6491,Allocations to tax provisions,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6492,6492,Allocations to operating provisions,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_65111,65111,AVA on shares in affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_65112,65112,AVA on amounts owed by affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_65113,65113,AVA on participating interests,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_65114,65114,AVA on amounts owed by undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_65115,65115,AVA on securities held as fixed assets,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_65116,65116,"AVA on loans, deposits and claims held as fixed assets",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6512,6512,FVA on financial fixed assets,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65211,65211,Shares in affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65212,65212,Amounts owed by affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65213,65213,Participating interests,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65214,65214,Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65215,65215,Securities held as fixed assets,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65216,65216,"Loans, deposits and claims held as fixed assets",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_652211,652211,Book value of yielded shares in affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_652212,652212,Disposal proceeds of shares in affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_652221,652221,Book value of yielded amounts owed by affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_652222,652222,Disposal proceeds of amounts owed by affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_652231,652231,Book value of yielded participating interests,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_652232,652232,Disposal proceeds of participating interests,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_652241,652241,Book value of yielded amounts owed by undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_652242,652242,Disposal proceeds of amounts owed by undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_652251,652251,Book value of yielded securities held as fixed assets,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_652252,652252,Disposal proceeds of securities held as fixed assets,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_652261,652261,"Book value of yielded loans, deposits and claims held as fixed assets",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_652262,652262,"Disposal proceeds of loans, deposits and claims held as fixed assets",account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_65311,65311,AVA on shares in affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_65312,65312,AVA on own shares or own corporate units,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_65313,65313,AVA on shares in undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_65318,65318,AVA on other transferable securities,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6532,6532,FVA on transferable securities,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65411,65411,From affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65412,65412,From undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65413,65413,From other receivables from current assets,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65421,65421,Shares in affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65422,65422,Own shares or corporate units,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65423,65423,Shares in undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65428,65428,Other transferable securities,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_65511,65511,Interest on debenture loans - affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_65512,65512,Interest on debenture loans - other,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_65521,65521,Banking interest on current accounts,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_65522,65522,Banking interest on financing operations,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_655231,655231,Interest on financial leases - affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_655232,655232,Interest on financial leases - other,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6553,6553,Interest on trade payables,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65541,65541,Interest payable to affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65542,65542,Interest payable to undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65551,65551,Discounts and charges on bills of exchange - affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65552,65552,Discounts and charges on bills of exchange - other,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65561,65561,Granted discounts - affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65562,65562,Granted discounts - other,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_657,657,Share in the losses of undertakings accounted for under the equity method,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65581,65581,Interest payable on other loans and debts - affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_65582,65582,Interest payable on other loans and debts - other,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6561,6561,Foreign currency exchange losses - affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6562,6562,Foreign currency exchange losses - other,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6581,6581,Other financial charges - affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6582,6582,Other financial charges - other,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6591,6591,Allocations to financial provisions - affiliated undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_6592,6592,Allocations to financial provisions - other,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6711,6711,CIT - current financial year,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6712,6712,CIT - previous financial years,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6721,6721,MBT - current financial year,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6722,6722,MBT - previous financial years,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6731,6731,Withholding taxes,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_67321,67321,Current financial year,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_67322,67322,Previous financial years,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6733,6733,Taxes levied on non-resident undertakings,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6738,6738,Other foreign income taxes,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_679,679,Allocations to provisions for deferred taxes,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6811,6811,NWT - current financial year,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_6812,6812,NWT - previous financial years,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_682,682,Subscription tax,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_683,683,Foreign taxes,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2011_account_688,688,Other taxes,account.data_account_type_expenses,FALSE,lu_2011_chart_1
lu_2020_account_7021,7021,Sales of finished goods,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7022,7022,Sales of semi-finished goods,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7023,7023,Sales of residual products,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7029,7029,Sales of work in progress,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_703001,703001,Sale of Services,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_70311,70311,Concessions,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_70312,70312,Patents,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_70313,70313,Software licences,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_70314,70314,Trademarks and franchises,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_703151,703151,Copyrights and reproduction rights,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_703158,703158,Other similar rights and assets,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_70321,70321,Rental income from real property,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_70322,70322,Rental income from movable property,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7033,7033,Sales of services not mentioned above,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7039,7039,Sales of services in the course of completion,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_704,704,Sales of packaging,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_705,705,Commissions and brokerage fees,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7061,7061,Sales of merchandise,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7062,7062,Sales of land resale,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7063,7063,Sales of buildings for resale,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_708,708,Other components of turnover,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7092,7092,RDR on sales of goods,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7093,7093,RDR on sales of services,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7094,7094,RDR on sales of packages,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7095,7095,RDR on commissions and brokerage fees,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7096,7096,RDR on sales of merchandise and other goods for resale,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7098,7098,RDR on other components of turnover,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7099,7099,"Not allocated rebates, discounts and refunds",account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7111,7111,Change in inventories of work in progress,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7112,7112,Change in inventories: contracts in progress - goods,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7113,7113,Change in inventories: contracts in progress - services,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7114,7114,Change in inventories: buildings under construction,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7121,7121,Change in inventories of finished goods,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7122,7122,Change in inventories of semi-finished goods,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7123,7123,Change in inventories of residual goods,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7211,7211,Development costs,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_72121,72121,Concessions,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_72122,72122,Patents,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_72123,72123,Software licences,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_72124,72124,Trademarks and franchises,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_721251,721251,Copyrights and reproduction rights,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_721258,721258,Other similar rights and assets,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7221,7221,"Land, fittings and buildings",account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7222,7222,Plant and machinery,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7223,7223,"Other fixtures and fittings, tools and equipment (included motor vehicles)",account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7321,7321,RVA on development costs,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7322,7322,"RVA on concessions, patents, licences, trademarks and similar rights and assets",account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7324,7324,RVA on down payments and intangible fixed assets under development,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_73311,73311,RVA on land,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_73312,73312,RVA on fixtures and fittings-out of land,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_73313,73313,RVA on buildings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_73314,73314,RVA on fixtures and fittings-out of buildings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_73315,73315,FVA on investment properties,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7332,7332,RVA on plant and machinery,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7333,7333,"Other fixtures and fittings, tools and equipment (included motor vehicles)",account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7334,7334,RVA on down payments and tangible fixed assets under development,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7341,7341,RVA on inventories of raw materials and consumables,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7342,7342,RVA on inventories of work and contracts in progress,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7343,7343,RVA on inventories of goods,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7344,7344,RVA on inventories of merchandise and other goods for resale,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7345,7345,RVA on down payments on inventories,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7351,7351,RVA on trade receivables,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7352,7352,RVA on amounts owed by affiliated undertakings and undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7353,7353,RVA on other receivables,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7354,7354,FVA on receivables from current assets,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7411,7411,Concessions,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2011_account_7412,7412,Patents,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2011_account_7413,7413,Software licences,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2011_account_7414,7414,Trademarks and franchises,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2011_account_74151,74151,Copyrights and reproduction rights,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2011_account_74158,74158,Other similar rights and assets,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_7421,7421,Rental income on real property,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_7422,7422,Rental income on movable property,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2011_account_743,743,"Attendance fees, director's fees and similar remunerations",account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_74411,74411,Book value of yielded intangible fixed assets,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_74412,74412,Disposal proceeds of intangible fixed assets,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_74421,74421,Book value of yielded tangible fixed assets,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_74422,74422,Disposal proceeds of tangible fixed assets,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_7451,7451,Product subsidies,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_7452,7452,Interest subsidies,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_7453,7453,Compensatory allowances,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_7454,7454,Subsidies in favour of employment development,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_7458,7458,Other subsidies for operating activities,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2011_account_746,746,Benefits in kind,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2011_account_7471,7471,Temporarily not taxable capital gains not reinvested,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2011_account_7472,7472,Temporarily not taxable capital gains reinvested,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2011_account_7473,7473,Capital investment subsidies,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_7481,7481,Insurance indemnities,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_7488,7488,Miscellaneous operating income,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_7491,7491,Reversals of provisions for taxes,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2020_account_7492,7492,Reversals of operating provisions,account.data_account_type_other_income,FALSE,lu_2011_chart_1
lu_2011_account_75111,75111,Shares in affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_75112,75112,Amounts owed by affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_75113,75113,Participating interests,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_75114,75114,Amounts owed by undertakings with which the company is linked by virtue of participating interests,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_75115,75115,Securities held as fixed assets,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_75116,75116,"Loans, deposits and claims held as fixed assets",account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7512,7512,FVA on financial fixed assets,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75211,75211,Shares in affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75212,75212,Amounts owed by affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75213,75213,Participating interests,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75214,75214,Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75215,75215,Securities held as fixed assets,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75216,75216,"Loans, deposits and claims held as fixed assets",account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_752211,752211,Book value of yielded shares in affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_752212,752212,Disposal proceeds of shares in affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_752221,752221,Book value of yielded amounts owed by affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_752222,752222,Disposal proceeds of amounts owed by affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_752231,752231,Book value of yielded participating interests,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_752232,752232,Disposal proceeds of participating interests,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_752241,752241,Book value of yielded amounts owed by undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_752242,752242,Disposal proceeds of amounts owed by undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_752251,752251,Book value of yielded securities held as fixed assets,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_752252,752252,Disposal proceeds of securities held as fixed assets,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_752261,752261,"Book value of yielded loans, deposits and claims held as fixed assets",account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_752262,752262,"Disposal proceed of loans, deposits and claims held as fixed assets",account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75311,75311,Shares in affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75312,75312,Own shares or corporate units,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75313,75313,Shares in undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75318,75318,Other transferable securities,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7532,7532,Fair value adjustments on transferable securities,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_75411,75411,On affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_75412,75412,On undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_75413,75413,On other current receivables,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75421,75421,Shares in affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75422,75422,Own shares or corporate units,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75423,75423,Shares in undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75428,75428,Other transferable securities,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_75481,75481,Shares in affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_75482,75482,Own shares or corporate units,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_75483,75483,Shares in undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_75488,75488,Other transferable securities,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_75521,75521,Interest on bank accounts,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_755231,755231,From affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_755232,755232,From other,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_7553,7553,Interest on trade receivables,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75541,75541,Interest on amounts owed by affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75542,75542,Interest on amounts owed by undertakings with which the undertaking is linked by virtue of participating interests,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75551,75551,Discounts on bills of exchange - affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75552,75552,Discounts on bills of exchange - other,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75561,75561,Discounts received - affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75562,75562,Discounts received - other,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75581,75581,Interest on other amounts receivable - affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_75582,75582,Interest on other amounts receivable - other,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7561,7561,Foreign currency exchange gains - affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7562,7562,Foreign currency exchange gains - other,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_757,757,Share of profit from undertakings accounted for under the equity method,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7581,7581,Other financial income - affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7582,7582,Other financial income - other,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7591,7591,Reversals of financial provisions - affiliated undertakings,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_7592,7592,Reversals of financial provisions - other,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_771,771,Adjustments of corporate income tax (CIT),account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_772,772,Adjustments of municipal business tax (MBT),account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_773,773,Adjustments of foreign income taxes,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2020_account_779,779,Reversals of provisions for deferred taxes,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_781,781,Adjustments of net wealth tax (NWT),account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_782,782,Adjustments of subscription tax,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_783,783,Adjustments of foreign taxes,account.data_account_type_revenue,FALSE,lu_2011_chart_1
lu_2011_account_788,788,Adjustments of other taxes,account.data_account_type_revenue,FALSE,lu_2011_chart_1

```

## File: data\account.chart.template.csv

```csv
id,property_account_receivable_id/id,property_account_payable_id/id,property_account_expense_categ_id/id,property_account_income_categ_id/id,income_currency_exchange_account_id/id,expense_currency_exchange_account_id/id,default_pos_receivable_account_id/id,property_stock_account_input_categ_id/id,property_stock_account_output_categ_id/id,property_stock_valuation_account_id/id,property_tax_payable_account_id:id,property_tax_receivable_account_id:id,property_advance_tax_payment_account_id:id
lu_2011_chart_1,lu_2011_account_4011,lu_2011_account_44111,lu_2011_account_6061,lu_2020_account_703001,lu_2020_account_7561,lu_2020_account_6561,lu_2011_account_40111,lu_2011_account_321,lu_2011_account_321,lu_2020_account_60761,lu_2011_account_461412,lu_2011_account_421612,lu_2011_account_421613

```

## File: data\account.fiscal.position.tax.template-2015.csv

```csv
id,position_id:id,tax_src_id:id,tax_dest_id:id
fiscal_position_tax_template_LU_85,account_fiscal_position_template_LU_IC,lu_2015_tax_VB-PA-17,lu_2011_tax_VB-IC-0
fiscal_position_tax_template_LU_86,account_fiscal_position_template_LU_IC,lu_2015_tax_VB-PA-14,lu_2011_tax_VB-IC-0
fiscal_position_tax_template_LU_87,account_fiscal_position_template_LU_IC,lu_2015_tax_VB-PA-8,lu_2011_tax_VB-IC-0
fiscal_position_tax_template_LU_4,account_fiscal_position_template_LU_IC,lu_2011_tax_VB-PA-3,lu_2011_tax_VB-IC-0
fiscal_position_tax_template_LU_5,account_fiscal_position_template_LU_IC,lu_2011_tax_VB-PA-0,lu_2011_tax_VB-IC-0
fiscal_position_tax_template_LU_88,account_fiscal_position_template_LU_IC,lu_2015_tax_VP-PA-17,lu_2011_tax_VP-IC-0
fiscal_position_tax_template_LU_89,account_fiscal_position_template_LU_IC,lu_2015_tax_VP-PA-14,lu_2011_tax_VP-IC-0
fiscal_position_tax_template_LU_90,account_fiscal_position_template_LU_IC,lu_2015_tax_VP-PA-8,lu_2011_tax_VP-IC-0
fiscal_position_tax_template_LU_9,account_fiscal_position_template_LU_IC,lu_2011_tax_VP-PA-3,lu_2011_tax_VP-IC-0
fiscal_position_tax_template_LU_91,account_fiscal_position_template_LU_IC,lu_2015_tax_FB-PA-17,lu_2015_tax_FB-IC-17
fiscal_position_tax_template_LU_92,account_fiscal_position_template_LU_IC,lu_2015_tax_FB-PA-14,lu_2015_tax_FB-IC-14
fiscal_position_tax_template_LU_93,account_fiscal_position_template_LU_IC,lu_2015_tax_FB-PA-8,lu_2015_tax_FB-IC-8
fiscal_position_tax_template_LU_14,account_fiscal_position_template_LU_IC,lu_2011_tax_FB-PA-3,lu_2011_tax_FB-IC-3
fiscal_position_tax_template_LU_15,account_fiscal_position_template_LU_IC,lu_2011_tax_FB-PA-0,lu_2011_tax_FB-IC-0
fiscal_position_tax_template_LU_94,account_fiscal_position_template_LU_IC,lu_2015_tax_FP-PA-17,lu_2015_tax_FP-IC-17
fiscal_position_tax_template_LU_95,account_fiscal_position_template_LU_IC,lu_2015_tax_FP-PA-14,lu_2015_tax_FP-IC-14
fiscal_position_tax_template_LU_96,account_fiscal_position_template_LU_IC,lu_2015_tax_FP-PA-8,lu_2015_tax_FP-IC-8
fiscal_position_tax_template_LU_19,account_fiscal_position_template_LU_IC,lu_2011_tax_FP-PA-3,lu_2011_tax_FP-IC-3
fiscal_position_tax_template_LU_20,account_fiscal_position_template_LU_IC,lu_2011_tax_FP-PA-0,lu_2011_tax_FP-IC-0
fiscal_position_tax_template_LU_97,account_fiscal_position_template_LU_IC,lu_2015_tax_IB-PA-17,lu_2015_tax_IB-IC-17
fiscal_position_tax_template_LU_98,account_fiscal_position_template_LU_IC,lu_2015_tax_IB-PA-14,lu_2015_tax_IB-IC-14
fiscal_position_tax_template_LU_99,account_fiscal_position_template_LU_IC,lu_2015_tax_IB-PA-8,lu_2015_tax_IB-IC-8
fiscal_position_tax_template_LU_24,account_fiscal_position_template_LU_IC,lu_2011_tax_IB-PA-3,lu_2011_tax_IB-IC-3
fiscal_position_tax_template_LU_25,account_fiscal_position_template_LU_IC,lu_2011_tax_IB-PA-0,lu_2011_tax_IB-IC-0
fiscal_position_tax_template_LU_100,account_fiscal_position_template_LU_IC,lu_2015_tax_IP-PA-17,lu_2015_tax_IP-IC-17
fiscal_position_tax_template_LU_101,account_fiscal_position_template_LU_IC,lu_2015_tax_IP-PA-14,lu_2015_tax_IP-IC-14
fiscal_position_tax_template_LU_102,account_fiscal_position_template_LU_IC,lu_2015_tax_IP-PA-8,lu_2015_tax_IP-IC-8
fiscal_position_tax_template_LU_29,account_fiscal_position_template_LU_IC,lu_2011_tax_IP-PA-3,lu_2011_tax_IP-IC-3
fiscal_position_tax_template_LU_30,account_fiscal_position_template_LU_IC,lu_2011_tax_IP-PA-0,lu_2011_tax_IP-IC-0
fiscal_position_tax_template_LU_103,account_fiscal_position_template_LU_IC,lu_2015_tax_AB-PA-17,lu_2015_tax_AB-IC-17
fiscal_position_tax_template_LU_104,account_fiscal_position_template_LU_IC,lu_2015_tax_AB-PA-14,lu_2015_tax_AB-IC-14
fiscal_position_tax_template_LU_105,account_fiscal_position_template_LU_IC,lu_2015_tax_AB-PA-8,lu_2015_tax_AB-IC-8
fiscal_position_tax_template_LU_35,account_fiscal_position_template_LU_IC,lu_2011_tax_AB-PA-3,lu_2011_tax_AB-IC-3
fiscal_position_tax_template_LU_36,account_fiscal_position_template_LU_IC,lu_2011_tax_AB-PA-0,lu_2011_tax_AB-IC-0
fiscal_position_tax_template_LU_106,account_fiscal_position_template_LU_IC,lu_2015_tax_AP-PA-17,lu_2015_tax_AP-IC-17
fiscal_position_tax_template_LU_107,account_fiscal_position_template_LU_IC,lu_2015_tax_AP-PA-14,lu_2015_tax_AP-IC-14
fiscal_position_tax_template_LU_108,account_fiscal_position_template_LU_IC,lu_2015_tax_AP-PA-8,lu_2015_tax_AP-IC-8
fiscal_position_tax_template_LU_41,account_fiscal_position_template_LU_IC,lu_2011_tax_AP-PA-3,lu_2011_tax_AP-IC-3
fiscal_position_tax_template_LU_42,account_fiscal_position_template_LU_IC,lu_2011_tax_AP-PA-0,lu_2011_tax_AP-IC-0
fiscal_position_tax_template_LU_109,account_fiscal_position_template_LU_EC,lu_2015_tax_VB-PA-17,lu_2011_tax_VB-EC-0
fiscal_position_tax_template_LU_110,account_fiscal_position_template_LU_EC,lu_2015_tax_VB-PA-14,lu_2011_tax_VB-EC-0
fiscal_position_tax_template_LU_111,account_fiscal_position_template_LU_EC,lu_2015_tax_VB-PA-8,lu_2011_tax_VB-EC-0
fiscal_position_tax_template_LU_46,account_fiscal_position_template_LU_EC,lu_2011_tax_VB-PA-3,lu_2011_tax_VB-EC-0
fiscal_position_tax_template_LU_47,account_fiscal_position_template_LU_EC,lu_2011_tax_VB-PA-0,lu_2011_tax_VB-EC-0
fiscal_position_tax_template_LU_112,account_fiscal_position_template_LU_EC,lu_2015_tax_VP-PA-17,lu_2011_tax_VP-EC-0
fiscal_position_tax_template_LU_113,account_fiscal_position_template_LU_EC,lu_2015_tax_VP-PA-14,lu_2011_tax_VP-EC-0
fiscal_position_tax_template_LU_114,account_fiscal_position_template_LU_EC,lu_2015_tax_VP-PA-8,lu_2011_tax_VP-EC-0
fiscal_position_tax_template_LU_51,account_fiscal_position_template_LU_EC,lu_2011_tax_VP-PA-3,lu_2011_tax_VP-EC-0
fiscal_position_tax_template_LU_52,account_fiscal_position_template_LU_EC,lu_2011_tax_VP-PA-0,lu_2011_tax_VP-EC-0
fiscal_position_tax_template_LU_115,account_fiscal_position_template_LU_EC,lu_2015_tax_FB-PA-17,lu_2015_tax_FB-EC-17
fiscal_position_tax_template_LU_116,account_fiscal_position_template_LU_EC,lu_2015_tax_FB-PA-14,lu_2015_tax_FB-EC-14
fiscal_position_tax_template_LU_117,account_fiscal_position_template_LU_EC,lu_2015_tax_FB-PA-8,lu_2015_tax_FB-EC-8
fiscal_position_tax_template_LU_56,account_fiscal_position_template_LU_EC,lu_2011_tax_FB-PA-3,lu_2011_tax_FB-EC-3
fiscal_position_tax_template_LU_57,account_fiscal_position_template_LU_EC,lu_2011_tax_FB-PA-0,lu_2011_tax_FB-EC-0
fiscal_position_tax_template_LU_118,account_fiscal_position_template_LU_EC,lu_2015_tax_FP-PA-17,lu_2015_tax_FP-EC-17
fiscal_position_tax_template_LU_119,account_fiscal_position_template_LU_EC,lu_2015_tax_FP-PA-14,lu_2015_tax_FP-EC-14
fiscal_position_tax_template_LU_120,account_fiscal_position_template_LU_EC,lu_2015_tax_FP-PA-8,lu_2015_tax_FP-EC-8
fiscal_position_tax_template_LU_61,account_fiscal_position_template_LU_EC,lu_2011_tax_FP-PA-3,lu_2011_tax_FP-EC-3
fiscal_position_tax_template_LU_62,account_fiscal_position_template_LU_EC,lu_2011_tax_FP-PA-0,lu_2011_tax_FP-EC-0
fiscal_position_tax_template_LU_121,account_fiscal_position_template_LU_EC,lu_2015_tax_IB-PA-17,lu_2015_tax_IB-EC-17
fiscal_position_tax_template_LU_122,account_fiscal_position_template_LU_EC,lu_2015_tax_IB-PA-14,lu_2015_tax_IB-EC-14
fiscal_position_tax_template_LU_123,account_fiscal_position_template_LU_EC,lu_2015_tax_IB-PA-8,lu_2015_tax_IB-EC-8
fiscal_position_tax_template_LU_66,account_fiscal_position_template_LU_EC,lu_2011_tax_IB-PA-3,lu_2011_tax_IB-EC-3
fiscal_position_tax_template_LU_67,account_fiscal_position_template_LU_EC,lu_2011_tax_IB-PA-0,lu_2011_tax_IB-EC-0
fiscal_position_tax_template_LU_124,account_fiscal_position_template_LU_EC,lu_2015_tax_IP-PA-17,lu_2015_tax_IP-EC-17
fiscal_position_tax_template_LU_125,account_fiscal_position_template_LU_EC,lu_2015_tax_IP-PA-14,lu_2015_tax_IP-EC-14
fiscal_position_tax_template_LU_126,account_fiscal_position_template_LU_EC,lu_2015_tax_IP-PA-8,lu_2015_tax_IP-EC-8
fiscal_position_tax_template_LU_71,account_fiscal_position_template_LU_EC,lu_2011_tax_IP-PA-3,lu_2011_tax_IP-EC-3
fiscal_position_tax_template_LU_72,account_fiscal_position_template_LU_EC,lu_2011_tax_IP-PA-0,lu_2011_tax_IP-EC-0
fiscal_position_tax_template_LU_127,account_fiscal_position_template_LU_EC,lu_2015_tax_AB-PA-17,lu_2015_tax_AB-EC-17
fiscal_position_tax_template_LU_128,account_fiscal_position_template_LU_EC,lu_2015_tax_AB-PA-14,lu_2015_tax_AB-EC-14
fiscal_position_tax_template_LU_129,account_fiscal_position_template_LU_EC,lu_2015_tax_AB-PA-8,lu_2015_tax_AB-EC-8
fiscal_position_tax_template_LU_77,account_fiscal_position_template_LU_EC,lu_2011_tax_AB-PA-3,lu_2011_tax_AB-EC-3
fiscal_position_tax_template_LU_78,account_fiscal_position_template_LU_EC,lu_2011_tax_AB-PA-0,lu_2011_tax_AB-EC-0
fiscal_position_tax_template_LU_130,account_fiscal_position_template_LU_EC,lu_2015_tax_AP-PA-17,lu_2015_tax_AP-EC-17
fiscal_position_tax_template_LU_131,account_fiscal_position_template_LU_EC,lu_2015_tax_AP-PA-14,lu_2015_tax_AP-EC-14
fiscal_position_tax_template_LU_132,account_fiscal_position_template_LU_EC,lu_2015_tax_AP-PA-8,lu_2015_tax_AP-EC-8
fiscal_position_tax_template_LU_83,account_fiscal_position_template_LU_EC,lu_2011_tax_AP-PA-3,lu_2011_tax_AP-EC-3
fiscal_position_tax_template_LU_84,account_fiscal_position_template_LU_EC,lu_2011_tax_AP-PA-0,lu_2011_tax_AP-EC-0

```

## File: data\account.fiscal.position.template-2011.csv

```csv
id,name,chart_template_id:id,sequence,auto_apply,vat_required,country_id:id,country_group_id:id
account_fiscal_position_template_LU_NO,Not liable to VAT,lu_2011_chart_1,,,,,
account_fiscal_position_template_LU_LU,Luxembourgish Taxable Person,lu_2011_chart_1,1,1,1,base.lu,
account_fiscal_position_template_private_LU_IC,EU private,lu_2011_chart_1,2,1,,,base.europe
account_fiscal_position_template_LU_IC,Intra-Community Taxable Person,lu_2011_chart_1,3,1,1,,base.europe
account_fiscal_position_template_LU_EC,Extra-Community Taxable Person,lu_2011_chart_1,4,1,,,

```

## File: data\account.tax.group.csv

```csv
id,name
tax_group_0,TVA 0%
tax_group_3,TVA 3%
tax_group_6,TVA 6%
tax_group_8,TVA 8%
tax_group_10,TVA 10%
tax_group_12,TVA 12%
tax_group_14,TVA 14%
tax_group_15,TVA 15%
tax_group_17,TVA 17%

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_lu.lu_2011_chart_1')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_reconcile_model_template_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="bank_fees_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="name">Bank Fees</field>
        <field name="account_id" ref="lu_2011_account_61333"/>
        <field name="amount_type">percentage</field>
        <field name="amount">100</field>
        <field name="label">Bank Fees</field>
        <field name="rule_type">writeoff_button</field>
    </record>
    <record id="cash_discount_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="name">Cash Discount</field>
        <field name="account_id" ref="lu_2020_account_65562"/>
        <field name="amount_type">percentage</field>
        <field name="amount">100</field>
        <field name="label">Cash Discount</field>
        <field name="rule_type">writeoff_button</field>
    </record>
</odoo>

```

## File: data\account_tax_report_line.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>
    <record id="account_tax_report_line_2_assesment_of_tax_due" model="account.tax.report.line">
      <field name="name">II. ASSESSMENT OF TAX DUE (output tax)</field>
      <field name="sequence">2</field>
      <field name="country_id" ref="base.lu"/>
      <field name="formula">None</field>
    </record>

    <record id="account_tax_report_line_2b_intra_community_acqui_of_goods_base" model="account.tax.report.line">
      <field name="name">II.B. Intra-Community acquisitions of goods – base (051)</field>
      <field name="sequence">3</field>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2b_base_3" model="account.tax.report.line">
      <field name="name">II.B. base 3% (049)</field>
      <field name="tag_name">II.B. base 3% (049)</field>
      <field name="sequence">9</field>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2b_base_exempt" model="account.tax.report.line">
      <field name="name">II.B. base exempt (194)</field>
      <field name="tag_name">II.B. base exempt (194)</field>
      <field name="sequence">10</field>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2b_base_17" model="account.tax.report.line">
      <field name="name">II.B. base 17% (711)</field>
      <field name="tag_name">II.B. base 17% (711)</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2b_base_14" model="account.tax.report.line">
      <field name="name">II.B. base 14% (713)</field>
      <field name="tag_name">II.B. base 14% (713)</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2b_base_8" model="account.tax.report.line">
      <field name="name">II.B. base 8% (715)</field>
      <field name="tag_name">II.B. base 8% (715)</field>
      <field name="sequence">8</field>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2b_manufactured_tobacco" model="account.tax.report.line">
      <field name="name">II.B. 719 - of manufactured tobacco (VAT is collected at the exit of the tax warehouse with excise duties)</field>
      <field name="tag_name">719</field>
      <field name="sequence">10</field>
      <field name="code">LUTAX_719</field>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_importation_of_goods_base" model="account.tax.report.line">
      <field name="name">II.D. Importation of goods – base (065)</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_2_base_exempt" model="account.tax.report.line">
      <field name="name">II.D.2. for non-business purposes: base exempt (196)</field>
      <field name="tag_name">II.D.2. for non-business purposes: base exempt (196)</field>
      <field name="sequence">12</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_1_base_17" model="account.tax.report.line">
      <field name="name">II.D.1. for business purposes: base 17% (721)</field>
      <field name="tag_name">II.D.1. for business purposes: base 17% (721)</field>
      <field name="sequence">3</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_1_base_14" model="account.tax.report.line">
      <field name="name">II.D.1. for business purposes: base 14% (723)</field>
      <field name="tag_name">II.D.1. for business purposes: base 14% (723)</field>
      <field name="sequence">4</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_1_base_8" model="account.tax.report.line">
      <field name="name">II.D.1. for business purposes: base 8% (725)</field>
      <field name="tag_name">II.D.1. for business purposes: base 8% (725)</field>
      <field name="sequence">5</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_1_manufactured_tobacco" model="account.tax.report.line">
      <field name="name">II.D. 729 - of manufactured tobacco (VAT is collected at the exit of the tax warehouse with excise duties)</field>
      <field name="tag_name">729</field>
      <field name="sequence">8</field>
      <field name="code">LUTAX_729</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_2_base_17" model="account.tax.report.line">
      <field name="name">II.D.2. for non-business purposes: base 17% (731)</field>
      <field name="tag_name">II.D.2. for non-business purposes: base 17% (731)</field>
      <field name="sequence">8</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_2_base_14" model="account.tax.report.line">
      <field name="name">II.D.2. for non-business purposes: base 14% (733)</field>
      <field name="tag_name">II.D.2. for non-business purposes: base 14% (733)</field>
      <field name="sequence">9</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_2_base_8" model="account.tax.report.line">
      <field name="name">II.D.2. for non-business purposes: base 8% (735)</field>
      <field name="tag_name">II.D.2. for non-business purposes: base 8% (735)</field>
      <field name="sequence">10</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_1_base_3" model="account.tax.report.line">
      <field name="name">II.D.1. for business purposes: base 3% (059)</field>
      <field name="tag_name">II.D.1. for business purposes: base 3% (059)</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_2_base_3" model="account.tax.report.line">
      <field name="name">II.D.2. for non-business purposes: base 3% (063)</field>
      <field name="tag_name">II.D.2. for non-business purposes: base 3% (063)</field>
      <field name="sequence">11</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_1_base_exempt" model="account.tax.report.line">
      <field name="name">II.D.1. for business purposes: base exempt (195)</field>
      <field name="tag_name">II.D.1. for business purposes: base exempt (195)</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2c_acquisitions_triangular_transactions_base" model="account.tax.report.line">
      <field name="name">II.C. Acquisitions, in the context of triangular transactions – base (152)</field>
      <field name="sequence">5</field>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_supply_of_service_for_customer" model="account.tax.report.line">
      <field name="name">II.E. Supply of services for which the customer is liable for the payment of VAT – base (409)</field>
      <field name="sequence">8</field>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_1_base" model="account.tax.report.line">
      <field name="name">II.E.1. base (436)</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_2e_supply_of_service_for_customer"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_base_3" model="account.tax.report.line">
      <field name="name">II.E.1.a) not exempt within the territory: base 3% (431)</field>
      <field name="tag_name">II.E.1.a) not exempt within the territory: base 3% (431)</field>
      <field name="sequence">9</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_1_b_exempt" model="account.tax.report.line">
      <field name="name">II.E.1.b) exempt within the territory: exempt (435)</field>
      <field name="tag_name">II.E.1.b) exempt within the territory: exempt (435)</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_2e_supply_of_service_for_customer"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_base_17" model="account.tax.report.line">
      <field name="name">II.E.1.a) not exempt within the territory: base 17% (741)</field>
      <field name="tag_name">II.E.1.a) not exempt within the territory: base 17% (741)</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_base_14" model="account.tax.report.line">
      <field name="name">II.E.1.a) not exempt within the territory: base 14% (743)</field>
      <field name="tag_name">II.E.1.a) not exempt within the territory: base 14% (743)</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_base_8" model="account.tax.report.line">
      <field name="name">II.E.1.a) not exempt within the territory: base 8% (745)</field>
      <field name="tag_name">II.E.1.a) not exempt within the territory: base 8% (745)</field>
      <field name="sequence">8</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_2_base" model="account.tax.report.line">
      <field name="name">II.E.2. base (463)</field>
      <field name="sequence">3</field>
      <field name="parent_id" ref="account_tax_report_line_2e_supply_of_service_for_customer"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_2_base_3" model="account.tax.report.line">
      <field name="name">II.E.2. not established or residing within the Community: base 3% (441)</field>
      <field name="tag_name">II.E.2. not established or residing within the Community: base 3% (441)</field>
      <field name="sequence">9</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_2_exempt" model="account.tax.report.line">
      <field name="name">II.E.2. not established or residing within the Community: exempt (445)</field>
      <field name="tag_name">II.E.2. not established or residing within the Community: exempt (445)</field>
      <field name="sequence">10</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_2_base_17" model="account.tax.report.line">
      <field name="name">II.E.2. not established or residing within the Community: base 17% (751)</field>
      <field name="tag_name">II.E.2. not established or residing within the Community: base 17% (751)</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_2_base_14" model="account.tax.report.line">
      <field name="name">II.E.2. not established or residing within the Community: base 14% (753)</field>
      <field name="tag_name">II.E.2. not established or residing within the Community: base 14% (753)</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_2_base_8" model="account.tax.report.line">
      <field name="name">II.E.2. not established or residing within the Community: base 8% (755)</field>
      <field name="tag_name">II.E.2. not established or residing within the Community: base 8% (755)</field>
      <field name="sequence">8</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_3_base" model="account.tax.report.line">
      <field name="name">II.E.3. base (765)</field>
      <field name="sequence">4</field>
      <field name="parent_id" ref="account_tax_report_line_2e_supply_of_service_for_customer"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_3_base_17" model="account.tax.report.line">
      <field name="name">II.E.3. suppliers established within the territory: base 17% (761)</field>
      <field name="tag_name">II.E.3. suppliers established within the territory: base 17% (761)</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_2e_3_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_1_assessment_taxable_turnover" model="account.tax.report.line">
      <field name="name">I. ASSESSMENT OF TAXABLE TURNOVER</field>
      <field name="sequence">1</field>
      <field name="country_id" ref="base.lu"/>
      <field name="formula">None</field>
    </record>

    <record id="account_tax_report_line_1a_overall_turnover" model="account.tax.report.line">
      <field name="name">I.A. Overall turnover (012)</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_1_assessment_taxable_turnover"/>
      <field name="country_id" ref="base.lu"/>
      <field name="formula">LUTAX_471 + LUTAX_021 + LUTAX_037 + LUTAX_455</field>
    </record>

    <record id="account_tax_report_line_1a_vat_acc_scheme" model="account.tax.report.line">
        <field name="name">I.A.1. VAT accounting scheme</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_1a_overall_turnover"/>
        <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_1a_total_sale" model="account.tax.report.line">
        <field name="name">I.A.2. Total Sales / Receipts (454)</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_1a_overall_turnover"/>
        <field name="country_id" ref="base.lu"/>
        <field name="formula">LUTAX_471 + LUTAX_021 + LUTAX_037 - LUTAX_456</field>
    </record>

    <!-- 471 not managed, only 472 is filled in -->
    <record id="account_tax_report_line_1a_telecom_service" model="account.tax.report.line">
        <field name="name">I.A.2.a). Telecommunications services, radio and television broadcasting services... (471)</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_1a_total_sale"/>
        <field name="country_id" ref="base.lu"/>
        <field name="code">LUTAX_471</field>
    </record>

    <record id="account_tax_report_line_1a_other_sales" model="account.tax.report.line">
        <field name="name">I.A.2.b). Other sales / receipts (472)</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_1a_total_sale"/>
        <field name="country_id" ref="base.lu"/>
        <field name="formula">LUTAX_021 + LUTAX_037 - LUTAX_456 - LUTAX_455 - LUTAX_471</field>
    </record>

    <record id="account_tax_report_line_1a_app_goods_non_bus" model="account.tax.report.line">
        <field name="name">I.A.3. Application of goods for non-business use and for business purposes (Art.13) (455)</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_1a_overall_turnover"/>
        <field name="country_id" ref="base.lu"/>
        <field name="code">LUTAX_455</field>
    </record>

    <record id="account_tax_report_line_1a_non_bus_gs" model="account.tax.report.line">
        <field name="name">I.A.4. Non-business use of goods and supply of services free of charge (Art.16) (456)</field>
        <field name="tag_name">I.A.4. Non-business use of goods and supply of services free of charge (Art.16) (456)</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_1a_overall_turnover"/>
        <field name="country_id" ref="base.lu"/>
        <field name="code">LUTAX_456</field>
    </record>

    <record id="account_tax_report_line_1b_exemptions_deductible_amounts" model="account.tax.report.line">
      <field name="name">I.B. Exemptions and deductible amounts (021)</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_1_assessment_taxable_turnover"/>
      <field name="country_id" ref="base.lu"/>
      <field name="code">LUTAX_021</field>
    </record>

    <record id="account_tax_report_line_1b_2_export" model="account.tax.report.line">
      <field name="name">I.B.2. Exports (Art.43(1)(a) and (b)) (014)</field>
      <field name="tag_name">I.B.2. Exports (Art.43(1)(a) and (b)) (014)</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_1b_1_intra_community_goods_pi_vat" model="account.tax.report.line">
      <field name="name">I.B.1. Intra-Community supply of goods to persons identified for VAT purposes in another Member State (MS) (Art.43(1)(d),(e) and (f)) (3) (457)</field>
      <field name="tag_name">I.B.1. Intra-Community supply of goods to persons identified for VAT purposes in another Member State (MS) (Art.43(1)(d),(e) and (f)) (3) (457)</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_1b_3_other_exemptions_art_43" model="account.tax.report.line">
      <field name="name">I.B.3. Other exemptions (Art.43 and 60bis) (015)</field>
      <field name="tag_name">I.B.3. Other exemptions (art.43 et 60bis) (015)</field>
      <field name="sequence">3</field>
      <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_1b_4_other_exemptions_art_44_et_56quater" model="account.tax.report.line">
      <field name="name">I.B.4. Other exemptions (Art.44 and 56quater) (016)</field>
      <field name="tag_name">I.B.4. Other exemptions (Art.44 and 56quater) (016)</field>
      <field name="sequence">4</field>
      <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_1b_5_manufactured_tobacco_vat_collected" model="account.tax.report.line">
      <field name="name">I.B.5. Manufactured tobacco whose VAT was collected at the source or at the exit of the tax... (017)</field>
      <field name="tag_name">I.B.5. Manufactured tobacco whose VAT was collected at the source or at the exit of the tax... (017)</field>
      <field name="sequence">5</field>
      <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_1b_6_a_subsequent_to_intra_community" model="account.tax.report.line">
      <field name="name">I.B.6.a) Supply, subsequent to intra-Community acquisitions of goods, in the context of triangular transactions, when the customer identified,... (018)</field>
      <field name="tag_name">I.B.6.a) Supply, subsequent to intra-Community acquisitions of goods, in the context of triangular transactions, when the customer identified,... (018)</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_1b_6_b1_non_exempt_customer_vat" model="account.tax.report.line">
      <field name="name">I.B.6.b)1) not exempt in the MS where the customer is liable for payment of VAT (Art.17(1)(b)) (5) (423)</field>
      <field name="tag_name">I.B.6.b)1) not exempt in the MS where the customer is liable for payment of VAT (Art.17(1)(b)) (5) (423)</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_1b_6_b2_exempt_ms_customer" model="account.tax.report.line">
      <field name="name">I.B.6.b)2) exempt in the MS where the customer is identified (Art.17(1)(b)) (424)</field>
      <field name="tag_name">I.B.6.b)2) exempt in the MS where the customer is identified (Art.17(1)(b)) (424)</field>
      <field name="sequence">8</field>
      <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_1b_6_c_supplies_scope_special_arrangement" model="account.tax.report.line">
      <field name="name">I.B.6.c) Supplies carried out within the scope of the special arrangement of art. 56sexies (226)</field>
      <field name="tag_name">I.B.6.c) Supplies carried out within the scope of the special arrangement of art. 56sexies (226)</field>
      <field name="sequence">9</field>
      <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_1b_6_d_supplies_other_referred" model="account.tax.report.line">
      <field name="name">I.B.6.d) Supplies other than referred to in (6)(a) and (6)(b) (019)</field>
      <field name="tag_name">I.B.6.d) Supplies other than referred to in (6)(a) and (6)(b) (019)</field>
      <field name="sequence">10</field>
      <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_1b_7_inland_supplies_for_customer" model="account.tax.report.line">
      <field name="name">I.B.7. Inland supplies for which the customer is liable for the payment of VAT (419)</field>
      <field name="tag_name">I.B.7. Inland supplies for which the customer is liable for the payment of VAT (419)</field>
      <field name="sequence">11</field>
      <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_1c_taxable_turnover" model="account.tax.report.line">
      <field name="name">I.C. Taxable turnover (022)</field>
      <field name="sequence">3</field>
      <field name="parent_id" ref="account_tax_report_line_1_assessment_taxable_turnover"/>
      <field name="country_id" ref="base.lu"/>
      <field name="code">LUTAX_022</field>
      <field name="formula">LUTAX_037</field>
    </record>

    <record id="account_tax_report_line_2a_breakdown_taxable_turnover_base" model="account.tax.report.line">
      <field name="name">II.A. Breakdown of taxable turnover – base (037)</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="country_id" ref="base.lu"/>
      <field name="code">LUTAX_037</field>
    </record>

    <record id="account_tax_report_line_2a_base_3" model="account.tax.report.line">
      <field name="name">II.A. base 3% (031)</field>
      <field name="tag_name">II.A. base 3% (031)</field>
      <field name="sequence">9</field>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2a_base_0" model="account.tax.report.line">
      <field name="name">II.A. base 0% (033)</field>
      <field name="tag_name">II.A. base 0% (033)</field>
      <field name="sequence">10</field>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_base"/>
      <field name="code">LUTAX_033</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2a_base_17" model="account.tax.report.line">
      <field name="name">II.A. base 17% (701)</field>
      <field name="tag_name">II.A. base 17% (701)</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2a_base_14" model="account.tax.report.line">
      <field name="name">II.A. base 14% (703)</field>
      <field name="tag_name">II.A. base 14% (703)</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2a_base_8" model="account.tax.report.line">
      <field name="name">II.A. base 8% (705)</field>
      <field name="tag_name">II.A. base 8% (705)</field>
      <field name="sequence">8</field>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_4_tax_tobe_paid_or_reclaimed" model="account.tax.report.line">
      <field name="name">IV. TAX TO BE PAID OR TO BE RECLAIMED</field>
      <field name="sequence">4</field>
      <field name="country_id" ref="base.lu"/>
      <field name="formula">None</field>
    </record>

    <record id="account_tax_report_line_4c_exceeding_amount" model="account.tax.report.line">
      <field name="name">IV.C. Exceeding amount (105)</field>
      <field name="sequence">3</field>
      <field name="parent_id" ref="account_tax_report_line_4_tax_tobe_paid_or_reclaimed"/>
      <field name="formula">(LUTAX_702+LUTAX_704+LUTAX_706+LUTAX_040+LUTAX_042+LUTAX_712+LUTAX_714+LUTAX_716+LUTAX_054+LUTAX_722+LUTAX_724+LUTAX_726+LUTAX_068+LUTAX_732+LUTAX_734+LUTAX_736+LUTAX_073+LUTAX_742+LUTAX_744+LUTAX_746+LUTAX_432+LUTAX_752+LUTAX_754+LUTAX_756+LUTAX_442+LUTAX_762+LUTAX_764+LUTAX_227)-(LUTAX_458+LUTAX_459+LUTAX_460+LUTAX_090+LUTAX_461+LUTAX_092+LUTAX_228+LUTAX_094+LUTAX_095)</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <!-- Note: this should have same formula as of section `II.H. Total tax due (076)` here above -->
    <record id="account_tax_report_line_4a_total_tax_due" model="account.tax.report.line">
      <field name="name">IV.A. Total tax due (103)</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_4_tax_tobe_paid_or_reclaimed"/>
      <field name="formula">LUTAX_702+LUTAX_704+LUTAX_706+LUTAX_040+LUTAX_042+LUTAX_712+LUTAX_714+LUTAX_716+LUTAX_054+LUTAX_722+LUTAX_724+LUTAX_726+LUTAX_068+LUTAX_732+LUTAX_734+LUTAX_736+LUTAX_073+LUTAX_742+LUTAX_744+LUTAX_746+LUTAX_432+LUTAX_752+LUTAX_754+LUTAX_756+LUTAX_442+LUTAX_762+LUTAX_764+LUTAX_227</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2f_supply_goods_base" model="account.tax.report.line">
      <field name="name">II.F. Supply of goods for which the purchaser is liable for the payment of VAT - base (767)</field>
      <field name="sequence">10</field>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2f_supply_goods_base_8" model="account.tax.report.line">
      <field name="name">II.F. base 8% (763)</field>
      <field name="tag_name">II.F. base 8% (763)</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_2f_supply_goods_base"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2f_supply_goods_tax" model="account.tax.report.line">
      <field name="name">II.F. Supply of goods for which the purchaser is liable for the payment of VAT - tax (768)</field>
      <field name="sequence">11</field>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2f_supply_goods_tax_8" model="account.tax.report.line">
      <field name="name">II.F. tax 8% (764)</field>
      <field name="tag_name">II.F. tax 8% (764)</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_2f_supply_goods_tax"/>
      <field name="code">LUTAX_764</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2g_special_arrangement" model="account.tax.report.line">
      <field name="name">II.G. Special arrangement for tax suspension: adjustment (Art.60bis, (5) and (8)) (227)</field>
      <field name="sequence">12</field>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="code">LUTAX_227</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <!-- Note: any change in formula here should be reflected in section `IV.A. Total tax due (103)` below -->
    <record id="account_tax_report_line_2h_total_tax_due" model="account.tax.report.line">
      <field name="name">II.H. Total tax due (076)</field>
      <field name="sequence">13</field>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="formula">LUTAX_702+LUTAX_704+LUTAX_706+LUTAX_040+LUTAX_042+LUTAX_712+LUTAX_714+LUTAX_716+LUTAX_054+LUTAX_722+LUTAX_724+LUTAX_726+LUTAX_068+LUTAX_732+LUTAX_734+LUTAX_736+LUTAX_073+LUTAX_742+LUTAX_744+LUTAX_746+LUTAX_432+LUTAX_752+LUTAX_754+LUTAX_756+LUTAX_442+LUTAX_762+LUTAX_764+LUTAX_227</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2a_breakdown_taxable_turnover_tax" model="account.tax.report.line">
      <field name="name">II.A. Breakdown of taxable turnover – tax (046)</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2a_tax_3" model="account.tax.report.line">
      <field name="name">II.A. tax 3% (040)</field>
      <field name="tag_name">II.A. tax 3% (040)</field>
      <field name="sequence">9</field>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_tax"/>
      <field name="code">LUTAX_040</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2a_tax_0" model="account.tax.report.line">
      <field name="name">II.A. tax 0% (042)</field>
      <field name="tag_name">II.A. tax 0% (042)</field>
      <field name="sequence">10</field>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_tax"/>
      <field name="code">LUTAX_042</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2a_tax_17" model="account.tax.report.line">
      <field name="name">II.A. tax 17% (702)</field>
      <field name="tag_name">II.A. tax 17% (702)</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_tax"/>
      <field name="code">LUTAX_702</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2a_tax_14" model="account.tax.report.line">
      <field name="name">II.A. tax 14% (704)</field>
      <field name="tag_name">II.A. tax 14% (704)</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_tax"/>
      <field name="code">LUTAX_704</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2a_tax_8" model="account.tax.report.line">
      <field name="name">II.A. tax 8% (706)</field>
      <field name="tag_name">II.A. tax 8% (706)</field>
      <field name="sequence">8</field>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_tax"/>
      <field name="code">LUTAX_706</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2b_intra_community_acquisitions_goods_tax" model="account.tax.report.line">
      <field name="name">II.B. Intra-Community acquisitions of goods – tax (056)</field>
      <field name="sequence">4</field>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2b_tax_3" model="account.tax.report.line">
      <field name="name">II.B. tax 3% (054)</field>
      <field name="tag_name">II.B. tax 3% (054)</field>
      <field name="sequence">8</field>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acquisitions_goods_tax"/>
      <field name="code">LUTAX_054</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2b_tax_17" model="account.tax.report.line">
      <field name="name">II.B. tax 17% (712)</field>
      <field name="tag_name">II.B. tax 17% (712)</field>
      <field name="sequence">5</field>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acquisitions_goods_tax"/>
      <field name="code">LUTAX_712</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2b_tax_14" model="account.tax.report.line">
      <field name="name">II.B. tax 14% (714)</field>
      <field name="tag_name">II.B. tax 14% (714)</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acquisitions_goods_tax"/>
      <field name="code">LUTAX_714</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2b_tax_8" model="account.tax.report.line">
      <field name="name">II.B. tax 8% (716)</field>
      <field name="tag_name">II.B. tax 8% (716)</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acquisitions_goods_tax"/>
      <field name="code">LUTAX_716</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_importation_of_goods_tax" model="account.tax.report.line">
      <field name="name">II.D. Importation of goods – tax (407)</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_1_tax_14" model="account.tax.report.line">
      <field name="name">II.D.1. for business purposes: tax 14% (724)</field>
      <field name="tag_name">II.D.1. for business purposes: tax 14% (724)</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_724</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_1_tax_8" model="account.tax.report.line">
      <field name="name">II.D.1. for business purposes: tax 8% (726)</field>
      <field name="tag_name">II.D.1. for business purposes: tax 8% (726)</field>
      <field name="sequence">3</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_726</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_2_tax_17" model="account.tax.report.line">
      <field name="name">II.D.2. for non-business purposes: tax 17% (732)</field>
      <field name="tag_name">II.D.2. for non-business purposes: tax 17% (732)</field>
      <field name="sequence">5</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_732</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_2_tax_14" model="account.tax.report.line">
      <field name="name">II.D.2. for non-business purposes: tax 14% (734)</field>
      <field name="tag_name">II.D.2. for non-business purposes: tax 14% (734)</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_734</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_2_tax_8" model="account.tax.report.line">
      <field name="name">II.D.2. for non-business purposes: tax 8% (736)</field>
      <field name="tag_name">II.D.2. for non-business purposes: tax 8% (736)</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_736</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_1_tax_3" model="account.tax.report.line">
      <field name="name">II.D.1. for business purposes: tax 3% (068)</field>
      <field name="tag_name">II.D.1. for business purposes: tax 3% (068)</field>
      <field name="sequence">4</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_068</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_2_tax_3" model="account.tax.report.line">
      <field name="name">II.D.2. for non-business purposes: tax 3% (073)</field>
      <field name="tag_name">II.D.2. for non-business purposes: tax 3% (073)</field>
      <field name="sequence">8</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_073</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2d_1_tax_17" model="account.tax.report.line">
      <field name="name">II.D.1. for business purposes: tax 17% (722)</field>
      <field name="tag_name">II.D.1. for business purposes: tax 17% (722)</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_722</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_supply_of_service_for_customer_liable_for_payment_tax" model="account.tax.report.line">
      <field name="name">II.E. Supply of services for which the customer is liable for the payment of VAT – tax (410)</field>
      <field name="sequence">9</field>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_tax" model="account.tax.report.line">
      <field name="name">II.E.1.a) tax (462)</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_2e_supply_of_service_for_customer_liable_for_payment_tax"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_tax_3" model="account.tax.report.line">
      <field name="name">II.E.1.a) not exempt within the territory: tax 3% (432)</field>
      <field name="tag_name">II.E.1.a) not exempt within the territory: tax 3% (432)</field>
      <field name="sequence">8</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_a_tax"/>
      <field name="code">LUTAX_432</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_tax_17" model="account.tax.report.line">
      <field name="name">II.E.1.a) not exempt within the territory: tax 17% (742)</field>
      <field name="tag_name">II.E.1.a) not exempt within the territory: tax 17% (742)</field>
      <field name="sequence">5</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_a_tax"/>
      <field name="code">LUTAX_742</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_tax_14" model="account.tax.report.line">
      <field name="name">II.E.1.a) not exempt within the territory: tax 14% (744)</field>
      <field name="tag_name">II.E.1.a) not exempt within the territory: tax 14% (744)</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_a_tax"/>
      <field name="code">LUTAX_744</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_tax_8" model="account.tax.report.line">
      <field name="name">II.E.1.a) not exempt within the territory: tax 8% (746)</field>
      <field name="tag_name">II.E.1.a) not exempt within the territory: tax 8% (746)</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_a_tax"/>
      <field name="code">LUTAX_746</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_2_tax" model="account.tax.report.line">
      <field name="name">II.E.2. tax (464)</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_2e_supply_of_service_for_customer_liable_for_payment_tax"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_2_tax_3" model="account.tax.report.line">
      <field name="name">II.E.2. not established or residing within the Community: tax 3% (442)</field>
      <field name="tag_name">II.E.2. not established or residing within the Community: tax 3% (442)</field>
      <field name="sequence">8</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_tax"/>
      <field name="code">LUTAX_442</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_2_tax_17" model="account.tax.report.line">
      <field name="name">II.E.2. not established or residing within the Community: tax 17% (752)</field>
      <field name="tag_name">II.E.2. not established or residing within the Community: tax 17% (752)</field>
      <field name="sequence">5</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_tax"/>
      <field name="code">LUTAX_752</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_2_tax_14" model="account.tax.report.line">
      <field name="name">II.E.2. not established or residing within the Community: tax 14% (754)</field>
      <field name="tag_name">II.E.2. not established or residing within the Community: tax 14% (754)</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_tax"/>
      <field name="code">LUTAX_754</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_2_tax_8" model="account.tax.report.line">
      <field name="name">II.E.2. not established or residing within the Community: tax 8% (756)</field>
      <field name="tag_name">II.E.2. not established or residing within the Community: tax 8% (756)</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_tax"/>
      <field name="code">LUTAX_756</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_3_tax" model="account.tax.report.line">
      <field name="name">II.E.3. tax (766)</field>
      <field name="sequence">3</field>
      <field name="parent_id" ref="account_tax_report_line_2e_supply_of_service_for_customer_liable_for_payment_tax"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2e_3_tax_17" model="account.tax.report.line">
      <field name="name">II.E.3. suppliers established within the territory: tax 17% (762)</field>
      <field name="tag_name">II.E.3. suppliers established within the territory: tax 17% (762)</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_2e_3_tax"/>
      <field name="code">LUTAX_762</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <!-- Note: this should have same formula as of section `III.C. Total input tax deductible (102)` here below -->
    <record id="account_tax_report_line_4a_total_input_tax_deductible" model="account.tax.report.line">
      <field name="name">IV.B. Total input tax deductible (104)</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_4_tax_tobe_paid_or_reclaimed"/>
      <field name="formula">LUTAX_458+LUTAX_459+LUTAX_460+LUTAX_090+LUTAX_461+LUTAX_092+LUTAX_228+LUTAX_094+LUTAX_095</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_3_assessment_deducible_tax" model="account.tax.report.line">
      <field name="name">III. ASSESSMENT OF DEDUCTIBLE TAX (input tax)</field>
      <field name="sequence">3</field>
      <field name="country_id" ref="base.lu"/>
      <field name="formula">None</field>
    </record>

    <record id="account_tax_report_line_3a_total_input_tax" model="account.tax.report.line">
      <field name="name">III.A. Total input tax (093)</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_3_assessment_deducible_tax"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_3b_total_input_tax_nd" model="account.tax.report.line">
      <field name="name">III.B. Total input tax non-deductible (097)</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_3_assessment_deducible_tax"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_3b1_rel_trans" model="account.tax.report.line">
      <field name="name">III.B.1. relating to transactions which are exempt pursuant to articles 44 and 56quater (094)</field>
      <field name="tag_name">094</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_3b_total_input_tax_nd"/>
      <field name="code">LUTAX_094</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_3b2_ded_prop" model="account.tax.report.line">
      <field name="name">III.B.2. where the deductible proportion determined in accordance to article 50 is applied (095)</field>
      <field name="tag_name">095</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_3b_total_input_tax_nd"/>
      <field name="code">LUTAX_095</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_3b2_input_tax_margin" model="account.tax.report.line">
      <field name="name">III.B.3 096 - Non recoverable input tax in accordance with Art. 56ter-1(7) and 56ter-2(7) (when applying the margin scheme)</field>
      <field name="tag_name">096</field>
      <field name="sequence">3</field>
      <field name="code">LUTAX_096</field>
      <field name="parent_id" ref="account_tax_report_line_3b_total_input_tax_nd"/>
      <field name="country_id" ref="base.lu"/>
    </record>

    <!-- Note: any change in formula here should be reflected in section `IV.B. Total input tax deductible (104)` above -->
    <record id="account_tax_report_line_3c_total_input_tax_deductible" model="account.tax.report.line">
      <field name="name">III.C. Total input tax deductible (102)</field>
      <field name="sequence">3</field>
      <field name="parent_id" ref="account_tax_report_line_3_assessment_deducible_tax"/>
      <field name="formula">LUTAX_458+LUTAX_459+LUTAX_460+LUTAX_090+LUTAX_461+LUTAX_092+LUTAX_228+LUTAX_094+LUTAX_095</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_3a_4_due_respect_application_goods" model="account.tax.report.line">
      <field name="name">III.A.4. Due in respect of the application of goods for business purposes (Art.48(1)(d)) (090)</field>
      <field name="tag_name">III.A.4. Due in respect of the application of goods for business purposes (Art.48(1)(d)) (090)</field>
      <field name="sequence">4</field>
      <field name="parent_id" ref="account_tax_report_line_3a_total_input_tax"/>
      <field name="code">LUTAX_090</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_3a_6_paid_joint_several_guarantee" model="account.tax.report.line">
      <field name="name">III.A.6. Paid as joint and several guarantee (092)</field>
      <field name="tag_name">III.A.6. Paid as joint and several guarantee (092)</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_3a_total_input_tax"/>
      <field name="code">LUTAX_092</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_3a_7_adjusted_tax_special_arrangement" model="account.tax.report.line">
      <field name="name">III.A.7. Adjusted tax - special arrangement for tax suspension (Art.60bis(9), subpar. 2) (228)</field>
      <field name="tag_name">III.A.7. Adjusted tax - special arrangement for tax suspension (Art.60bis(9), subpar. 2) (228)</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_3a_total_input_tax"/>
      <field name="code">LUTAX_228</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_3a_1_invoiced_by_other_taxable_person" model="account.tax.report.line">
      <field name="name">III.A.1. Invoiced by other taxable persons for goods or services supplied (Art.48(1)(a)) (458)</field>
      <field name="tag_name">III.A.1. Invoiced by other taxable persons for goods or services supplied (Art.48(1)(a)) (458)</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_3a_total_input_tax"/>
      <field name="code">LUTAX_458</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_3a_2_due_respect_intra_comm_goods" model="account.tax.report.line">
      <field name="name">III.A.2. Due in respect of intra-Community acquisitions of goods (Art.48(1)(b)) (459)</field>
      <field name="tag_name">III.A.2. Due in respect of intra-Community acquisitions of goods (Art.48(1)(b)) (459)</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_3a_total_input_tax"/>
      <field name="code">LUTAX_459</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_3a_3_due_paid_respect_importation_goods" model="account.tax.report.line">
      <field name="name">III.A.3. Due or paid in respect of importation of goods (Art.48(1)(c)) (460)</field>
      <field name="tag_name">III.A.3. Due or paid in respect of importation of goods (Art.48(1)(c)) (460)</field>
      <field name="sequence">3</field>
      <field name="parent_id" ref="account_tax_report_line_3a_total_input_tax"/>
      <field name="code">LUTAX_460</field>
      <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_3a_5_due_under_reverse_charge" model="account.tax.report.line">
      <field name="name">III.A.5. Due under the reverse charge (see points II.E and F) (461)</field>
      <field name="tag_name">III.A.5. Due under the reverse charge (see points II.E and F) (461)</field>
      <field name="sequence">5</field>
      <field name="parent_id" ref="account_tax_report_line_3a_total_input_tax"/>
      <field name="code">LUTAX_461</field>
      <field name="country_id" ref="base.lu"/>
    </record>

</odoo>

```

## File: data\account_tax_template_2015.xml

```xml
<odoo>
	<record id="lu_2011_tax_AB-EC-0" model="account.tax.template">
		<field name="sequence">171</field>
        <field name="description">0%</field>
        <field name="name">EX-EC-P-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax'
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax'
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-EC-14" model="account.tax.template">
		<field name="sequence">105</field>
        <field name="description">14%</field>
        <field name="name">14-EC-P-G</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_14')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-EC-17" model="account.tax.template">
		<field name="sequence">111</field>
        <field name="description">17%</field>
        <field name="name">17-EC-P-G</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_17')],
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_AB-EC-3" model="account.tax.template">
		<field name="sequence">114</field>
        <field name="description">3%</field>
        <field name="name">3-EC-P-G</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')]
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_3')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-EC-8" model="account.tax.template">
		<field name="sequence">120</field>
        <field name="description">8%</field>
        <field name="name">8-EC-P-G</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_8')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-ECP-0" model="account.tax.template">
		<field name="sequence">123</field>
        <field name="description">0%</field>
        <field name="name">EX-EC(P)-P-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax'
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax'
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-ECP-14" model="account.tax.template">
		<field name="sequence">127</field>
        <field name="description">14%</field>
        <field name="name">14-EC(P)-P-G</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_14')]
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_14')]
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_14')]
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')]
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_14')]
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-ECP-17" model="account.tax.template">
		<field name="sequence">133</field>
        <field name="description">17%</field>
        <field name="name">17-EC(P)-P-G</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')]
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_17')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-ECP-3" model="account.tax.template">
		<field name="sequence">136</field>
        <field name="description">3%</field>
        <field name="name">3-EC(P)-P-G</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_3')]
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')]
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_3')]
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_3')]
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')]
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_3')]
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-ECP-8" model="account.tax.template">
		<field name="sequence">142</field>
        <field name="description">8%</field>
        <field name="name">8-EC(P)-P-G</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')]
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_8')]
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')]
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_8')]
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_AB-IC-0" model="account.tax.template">
		<field name="sequence">145</field>
        <field name="description">0%</field>
        <field name="name">EX-IC-P-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax'
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax'
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-IC-14" model="account.tax.template">
		<field name="sequence">149</field>
        <field name="description">14%</field>
        <field name="name">14-IC-P-G</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')]
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_14')]
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')]
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_14')]
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-IC-17" model="account.tax.template">
		<field name="sequence">155</field>
        <field name="description">17%</field>
        <field name="name">17-IC-P-G</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')]
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_17')]
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')]
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_17')]
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_AB-IC-3" model="account.tax.template">
		<field name="sequence">158</field>
        <field name="description">3%</field>
        <field name="name">3-IC-P-G</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_3')]
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')]
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_3')]
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_3')]
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')]
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_3')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-IC-8" model="account.tax.template">
		<field name="sequence">164</field>
        <field name="description">8%</field>
        <field name="name">8-IC-P-G</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_8')],
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_AB-PA-0" model="account.tax.template">
		<field name="sequence">167</field>
        <field name="description">0%</field>
        <field name="name">0-P-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
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
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-PA-14" model="account.tax.template">
		<field name="sequence">169</field>
        <field name="description">14%</field>
        <field name="name">14-P-G</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-PA-17" model="account.tax.template">
		<field name="sequence">101</field>
        <field name="description">17%</field>
        <field name="name">17-P-G</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>

		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_AB-PA-3" model="account.tax.template">
		<field name="sequence">172</field>
        <field name="description">3%</field>
        <field name="name">3-P-G</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-PA-8" model="account.tax.template">
		<field name="sequence">174</field>
        <field name="description">8%</field>
        <field name="name">8-P-G</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_AP-EC-0" model="account.tax.template">
		<field name="sequence">175</field>
        <field name="description">0%</field>
        <field name="name">EX-EC-P-S</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_exempt')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AP-EC-14" model="account.tax.template">
		<field name="sequence">179</field>
        <field name="description">14%</field>
        <field name="name">14-EC-P-S</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_14')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AP-EC-17" model="account.tax.template">
		<field name="sequence">185</field>
        <field name="description">17%</field>
        <field name="name">17-EC-P-S</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_17')],
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_AP-EC-3" model="account.tax.template">
		<field name="sequence">188</field>
        <field name="description">3%</field>
        <field name="name">3-EC-P-S</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_3')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AP-EC-8" model="account.tax.template">
		<field name="sequence">194</field>
        <field name="description">8%</field>
        <field name="name">8-EC-P-S</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_8')],
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_AP-IC-0" model="account.tax.template">
		<field name="sequence">197</field>
        <field name="description">0%</field>
        <field name="name">EX-IC-P-S</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_b_exempt')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_b_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AP-IC-14" model="account.tax.template">
		<field name="sequence">201</field>
        <field name="description">14%</field>
        <field name="name">14-IC-P-S</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_14')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AP-IC-17" model="account.tax.template">
		<field name="sequence">207</field>
        <field name="description">17%</field>
        <field name="name">17-IC-P-S</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_17')],
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_AP-IC-3" model="account.tax.template">
		<field name="sequence">210</field>
        <field name="description">3%</field>
        <field name="name">3-IC-P-S</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_3')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AP-IC-8" model="account.tax.template">
		<field name="sequence">216</field>
        <field name="description">8%</field>
        <field name="name">8-IC-P-S</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_8')],
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_AP-PA-0" model="account.tax.template">
		<field name="sequence">219</field>
        <field name="description">0%</field>
        <field name="name">0-P-S</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
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
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AP-PA-14" model="account.tax.template">
		<field name="sequence">221</field>
        <field name="description">14%</field>
        <field name="name">14-P-S</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AP-PA-17" model="account.tax.template">
		<field name="sequence">223</field>
        <field name="description">17%</field>
        <field name="name">17-P-S</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_AP-PA-3" model="account.tax.template">
		<field name="sequence">224</field>
        <field name="description">3%</field>
        <field name="name">3-P-S</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AP-PA-8" model="account.tax.template">
		<field name="sequence">226</field>
        <field name="description">8%</field>
        <field name="name">8-P-S</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_FB-EC-0" model="account.tax.template">
		<field name="sequence">227</field>
        <field name="description">0%</field>
        <field name="name">EX-EC-E-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_exempt')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
        <field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-EC-14" model="account.tax.template">
		<field name="sequence">231</field>
        <field name="description">14%</field>
        <field name="name">14-EC-E-G</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_14')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-EC-17" model="account.tax.template">
		<field name="sequence">237</field>
        <field name="description">17%</field>
        <field name="name">17-EC-E-G</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_17')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_FB-EC-3" model="account.tax.template">
		<field name="sequence">240</field>
        <field name="description">3%</field>
        <field name="name">3-EC-E-G</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_3')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-EC-8" model="account.tax.template">
		<field name="sequence">246</field>
        <field name="description">8%</field>
        <field name="name">8-EC-E-G</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_8')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-ECP-0" model="account.tax.template">
		<field name="sequence">249</field>
        <field name="description">0%</field>
        <field name="name">EX-EC(P)-E-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_exempt')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-ECP-14" model="account.tax.template">
		<field name="sequence">253</field>
        <field name="description">14%</field>
        <field name="name">14-EC(P)-E-G</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_14')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-ECP-17" model="account.tax.template">
		<field name="sequence">259</field>
        <field name="description">17%</field>
        <field name="name">17-EC(P)-E-G</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_17')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-ECP-3" model="account.tax.template">
		<field name="sequence">262</field>
        <field name="description">3%</field>
        <field name="name">3-EC(P)-E-G</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_3')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-ECP-8" model="account.tax.template">
		<field name="sequence">268</field>
        <field name="description">8%</field>
        <field name="name">8-EC(P)-E-G</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_8')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_FB-IC-0" model="account.tax.template">
		<field name="sequence">271</field>
        <field name="description">0%</field>
        <field name="name">EX-IC-E-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_exempt')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-IC-14" model="account.tax.template">
		<field name="sequence">275</field>
        <field name="description">14%</field>
        <field name="name">14-IC-E-G</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_14')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-IC-17" model="account.tax.template">
		<field name="sequence">281</field>
        <field name="description">17%</field>
        <field name="name">17-IC-E-G</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_17')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_FB-IC-3" model="account.tax.template">
		<field name="sequence">284</field>
        <field name="description">3%</field>
        <field name="name">3-IC-E-G</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_3')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-IC-8" model="account.tax.template">
		<field name="sequence">290</field>
        <field name="description">8%</field>
        <field name="name">8-IC-E-G</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_8')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_FB-PA-0" model="account.tax.template">
		<field name="sequence">293</field>
        <field name="description">0%</field>
        <field name="name">0-E-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
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
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-PA-14" model="account.tax.template">
		<field name="sequence">295</field>
        <field name="description">14%</field>
        <field name="name">14-E-G</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-PA-17" model="account.tax.template">
		<field name="sequence">297</field>
        <field name="description">17%</field>
        <field name="name">17-E-G</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_FB-PA-3" model="account.tax.template">
		<field name="sequence">298</field>
        <field name="description">3%</field>
        <field name="name">3-E-G</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-PA-8" model="account.tax.template">
		<field name="sequence">300</field>
        <field name="description">8%</field>
        <field name="name">8-E-G</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_FP-EC-0" model="account.tax.template">
		<field name="sequence">301</field>
        <field name="description">0%</field>
        <field name="name">0-EC-E-S</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_exempt')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FP-EC-14" model="account.tax.template">
		<field name="sequence">305</field>
        <field name="description">14%</field>
        <field name="name">14-EC-E-S</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_14')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FP-EC-17" model="account.tax.template">
		<field name="sequence">311</field>
        <field name="description">17%</field>
        <field name="name">17-EC-E-S</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_17')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_FP-EC-3" model="account.tax.template">
		<field name="sequence">314</field>
        <field name="description">3%</field>
        <field name="name">3-EC-E-S</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_3')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FP-EC-8" model="account.tax.template">
		<field name="sequence">320</field>
        <field name="description">8%</field>
        <field name="name">8-EC-E-S</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_8')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

    <record id="lu_2011_tax_FP-IC-0" model="account.tax.template">
		<field name="sequence">323</field>
        <field name="description">0%</field>
        <field name="name">EX-IC-E-S</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_b_exempt')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_b_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FP-IC-14" model="account.tax.template">
		<field name="sequence">327</field>
        <field name="description">14%</field>
        <field name="name">14-IC-E-S</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_14')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FP-IC-17" model="account.tax.template">
		<field name="sequence">333</field>
        <field name="description">17%</field>
        <field name="name">17-IC-E-S</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_17')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_FP-IC-3" model="account.tax.template">
		<field name="sequence">336</field>
        <field name="description">3%</field>
        <field name="name">3-IC-E-S</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_3')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FP-IC-8" model="account.tax.template">
		<field name="sequence">342</field>
        <field name="description">8%</field>
        <field name="name">8-IC-E-S</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_8')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_FP-PA-0" model="account.tax.template">
		<field name="sequence">345</field>
        <field name="description">0%</field>
        <field name="name">0-E-S</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
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
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FP-PA-14" model="account.tax.template">
		<field name="sequence">347</field>
        <field name="description">14%</field>
        <field name="name">14-E-S</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FP-PA-17" model="account.tax.template">
		<field name="sequence">349</field>
        <field name="description">17%</field>
        <field name="name">17-E-S</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_FP-PA-3" model="account.tax.template">
		<field name="sequence">350</field>
        <field name="description">3%</field>
        <field name="name">3-E-S</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FP-PA-8" model="account.tax.template">
		<field name="sequence">352</field>
        <field name="description">8%</field>
        <field name="name">8-E-S</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_IB-EC-0" model="account.tax.template">
		<field name="sequence">353</field>
        <field name="description">0%</field>
        <field name="name">0-EC-IG</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_exempt')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-EC-14" model="account.tax.template">
		<field name="sequence">357</field>
        <field name="description">14%</field>
        <field name="name">14-EC-IG</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_14')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-EC-17" model="account.tax.template">
		<field name="sequence">363</field>
        <field name="description">17%</field>
        <field name="name">17-EC-IG</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_17')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_IB-EC-3" model="account.tax.template">
		<field name="sequence">366</field>
        <field name="description">3%</field>
        <field name="name">3-EC-IG</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_3')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-EC-8" model="account.tax.template">
		<field name="sequence">372</field>
        <field name="description">8%</field>
        <field name="name">8-EC-IG</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_8')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-ECP-0" model="account.tax.template">
		<field name="sequence">375</field>
        <field name="description">5%</field>
        <field name="name">0-EC(P)-IG</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_exempt')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-ECP-14" model="account.tax.template">
		<field name="sequence">379</field>
        <field name="description">14%</field>
        <field name="name">14-EC(P)-IG</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_14')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-ECP-17" model="account.tax.template">
		<field name="sequence">385</field>
        <field name="description">17%</field>
        <field name="name">17-EC(P)-IG</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_17')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-ECP-3" model="account.tax.template">
		<field name="sequence">388</field>
        <field name="description">3%</field>
        <field name="name">3-EC(P)-IG</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_3')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-ECP-8" model="account.tax.template">
		<field name="sequence">394</field>
        <field name="description">8%</field>
        <field name="name">8-EC(P)-IG</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_3_due_paid_respect_importation_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_8')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_IB-IC-0" model="account.tax.template">
		<field name="sequence">397</field>
        <field name="description">0%</field>
        <field name="name">0-IC-IG</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_exempt')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-IC-14" model="account.tax.template">
		<field name="sequence">401</field>
        <field name="description">14%</field>
        <field name="name">14-IC-IG</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_14')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-IC-17" model="account.tax.template">
		<field name="sequence">407</field>
        <field name="description">17%</field>
        <field name="name">17-IC-IG</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_17')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_IB-IC-3" model="account.tax.template">
		<field name="sequence">410</field>
        <field name="description">3%</field>
        <field name="name">3-IC-IG</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_3')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-IC-8" model="account.tax.template">
		<field name="sequence">416</field>
        <field name="description">8%</field>
        <field name="name">8-IC-IG</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_2_due_respect_intra_comm_goods')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_8')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_IB-PA-0" model="account.tax.template">
		<field name="sequence">419</field>
        <field name="description">0%</field>
        <field name="name">0-IG</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
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
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-PA-14" model="account.tax.template">
		<field name="sequence">421</field>
        <field name="description">14%</field>
        <field name="name">14-IG</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-PA-17" model="account.tax.template">
		<field name="sequence">423</field>
        <field name="description">17%</field>
        <field name="name">17-IG</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_IB-PA-3" model="account.tax.template">
		<field name="sequence">424</field>
        <field name="description">3%</field>
        <field name="name">3-IG</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-PA-8" model="account.tax.template">
		<field name="sequence">426</field>
        <field name="description">8%</field>
        <field name="name">8-IG</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_IP-EC-0" model="account.tax.template">
		<field name="sequence">427</field>
        <field name="description">0%</field>
        <field name="name">0-EC-IS</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_exempt')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IP-EC-14" model="account.tax.template">
		<field name="sequence">431</field>
        <field name="description">14%</field>
        <field name="name">14-EC-IS</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_14')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IP-EC-17" model="account.tax.template">
		<field name="sequence">437</field>
        <field name="description">17%</field>
        <field name="name">17-EC-IS</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_17')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_IP-EC-3" model="account.tax.template">
		<field name="sequence">440</field>
        <field name="description">3%</field>
        <field name="name">3-EC-IS</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_3')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IP-EC-8" model="account.tax.template">
		<field name="sequence">446</field>
        <field name="description">8%</field>
        <field name="name">8-EC-IS</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_8')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_IP-IC-0" model="account.tax.template">
		<field name="sequence">449</field>
        <field name="description">0%</field>
        <field name="name">0-IC-IS</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_b_exempt')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_b_exempt')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IP-IC-14" model="account.tax.template">
		<field name="sequence">453</field>
        <field name="description">14%</field>
        <field name="name">14-IC-IS</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_14')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IP-IC-17" model="account.tax.template">
		<field name="sequence">459</field>
        <field name="description">17%</field>
        <field name="name">17-IC-IS</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_17')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_IP-IC-3" model="account.tax.template">
		<field name="sequence">462</field>
        <field name="description">3%</field>
        <field name="name">3-IC-IS</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_3')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IP-IC-8" model="account.tax.template">
		<field name="sequence">468</field>
        <field name="description">8%</field>
        <field name="name">8-IC-IS</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': -100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_5_due_under_reverse_charge')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_8')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_IP-PA-0" model="account.tax.template">
		<field name="sequence">471</field>
        <field name="description">0%</field>
        <field name="name">0-IS</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
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
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IP-PA-14" model="account.tax.template">
		<field name="sequence">473</field>
        <field name="description">14%</field>
        <field name="name">14-IS</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IP-PA-17" model="account.tax.template">
		<field name="sequence">475</field>
        <field name="description">17%</field>
        <field name="name">17-IS</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_IP-PA-3" model="account.tax.template">
		<field name="sequence">476</field>
        <field name="description">3%</field>
        <field name="name">3-IS</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IP-PA-8" model="account.tax.template">
		<field name="sequence">478</field>
        <field name="description">8%</field>
        <field name="name">8-IS</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_421611'),
		        'plus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
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
		        'account_id': ref('lu_2020_account_421611'),
		        'minus_report_line_ids': [ref('account_tax_report_line_3a_1_invoiced_by_other_taxable_person')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_V-ART-43_60b" model="account.tax.template">
		<field name="sequence">489</field>
        <field name="description">0%</field>
        <field name="name">0-E-Art.43&amp;60b</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_1b_3_other_exemptions_art_43')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_1b_3_other_exemptions_art_43')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_V-ART-44_56q" model="account.tax.template">
		<field name="sequence">480</field>
        <field name="description">0%</field>
        <field name="name">0-E-Art.44&amp;56q</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_1b_4_other_exemptions_art_44_et_56quater')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_1b_4_other_exemptions_art_44_et_56quater')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_VB-EC-0" model="account.tax.template">
		<field name="sequence">481</field>
        <field name="description">0%</field>
        <field name="name">0-EC-S-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_1b_2_export')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_1b_2_export')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_VB-EC-Tab" model="account.tax.template">
		<field name="sequence">482</field>
        <field name="description">0%</field>
        <field name="name">0-EC-ST-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_1b_5_manufactured_tobacco_vat_collected')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_1b_5_manufactured_tobacco_vat_collected')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_VB-IC-0" model="account.tax.template">
		<field name="sequence">483</field>
        <field name="description">0%</field>
        <field name="name">0-IC-S-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_1b_1_intra_community_goods_pi_vat')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_1b_1_intra_community_goods_pi_vat')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_VB-IC-Tab" model="account.tax.template">
		<field name="sequence">484</field>
        <field name="description">0%</field>
        <field name="name">0-IC-ST-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_1b_5_manufactured_tobacco_vat_collected')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_1b_5_manufactured_tobacco_vat_collected')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_VB-PA-0" model="account.tax.template">
		<field name="sequence">485</field>
        <field name="description">0%</field>
        <field name="name">0-S-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_0')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_0')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_VB-PA-14" model="account.tax.template">
		<field name="sequence">487</field>
        <field name="description">14%</field>
        <field name="name">14-S-G</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_14')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_VB-PA-17" model="account.tax.template">
		<field name="sequence">502</field>
        <field name="description">17%</field>
        <field name="name">17-S-G</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_17')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_VB-PA-3" model="account.tax.template">
		<field name="sequence">490</field>
        <field name="description">3%</field>
        <field name="name">3-S-G</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_3')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_VB-PA-8" model="account.tax.template">
		<field name="sequence">492</field>
        <field name="description">8%</field>
        <field name="name">8-S-G</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_8')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_VB-PA-Tab" model="account.tax.template">
		<field name="sequence">493</field>
        <field name="description">0%</field>
        <field name="name">0-ST-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_1b_5_manufactured_tobacco_vat_collected')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_1b_5_manufactured_tobacco_vat_collected')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_VB-TR-0" model="account.tax.template">
		<field name="sequence">494</field>
        <field name="description">0%</field>
        <field name="name">0-ICT-S-G</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_1b_6_a_subsequent_to_intra_community')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_1b_6_a_subsequent_to_intra_community')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_VP-EC-0" model="account.tax.template">
		<field name="sequence">495</field>
        <field name="description">0%</field>
        <field name="name">0-EC-S-S</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_1b_6_d_supplies_other_referred')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_1b_6_d_supplies_other_referred')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_VP-IC-0" model="account.tax.template">
		<field name="sequence">496</field>
        <field name="description">0%</field>
        <field name="name">0-IC-S-S</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_1b_6_b1_non_exempt_customer_vat')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_1b_6_b1_non_exempt_customer_vat')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_VP-IC-EX" model="account.tax.template">
		<field name="sequence">497</field>
        <field name="description">0%</field>
        <field name="name"> EX-IC-S-S</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_1b_6_b2_exempt_ms_customer')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_1b_6_b2_exempt_ms_customer')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_VP-PA-0" model="account.tax.template">
		<field name="sequence">498</field>
        <field name="description">0%</field>
        <field name="name">0-S-S</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_0')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_0')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_VP-PA-14" model="account.tax.template">
		<field name="sequence">500</field>
        <field name="description">14%</field>
        <field name="name">14-S-S</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_14"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_14')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_14')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_14')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_VP-PA-17" model="account.tax.template">
		<field name="sequence">479</field>
        <field name="description">17%</field>
        <field name="name">17-S-S</field>
        <field name="amount">17</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_17')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_17')],
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_VP-PA-3" model="account.tax.template">
		<field name="sequence">503</field>
        <field name="description">3%</field>
        <field name="name">3-S-S</field>
        <field name="amount">3</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_3"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_3')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_3')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_3')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_VP-PA-8" model="account.tax.template">
		<field name="sequence">505</field>
        <field name="description">8%</field>
        <field name="name">8-S-S</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_8"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_8')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_8')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_8')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_SANS" model="account.tax.template">
		<field name="sequence">506</field>
        <field name="description">0%</field>
        <field name="name">0-P-Tax-Free</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
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
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_SANS_sale" model="account.tax.template">
		<field name="sequence">507</field>
        <field name="description">0%</field>
        <field name="name">0-S-Tax-Free</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
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
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_ATN_sale" model="account.tax.template">
		<field name="sequence">510</field>
		<field name="description">17%</field>
		<field name="name">17-ATN</field>
		<field name="amount">17</field>
		<field name="amount_type">percent</field>
		<field name="type_tax_use">sale</field>
		<field name="chart_template_id" ref="lu_2011_chart_1"/>
		<field name="tax_group_id" ref="tax_group_17"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [
		            ref('account_tax_report_line_1a_non_bus_gs'),
		            ref('account_tax_report_line_2a_base_17'),
		        ],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_17')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [
		            ref('account_tax_report_line_1a_non_bus_gs'),
		            ref('account_tax_report_line_2a_base_17'),
		        ],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_17')],
		    }),
		]"/>
	</record>
</odoo>

```

## File: data\l10n_lu_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <menuitem id="account_reports_lu_statements_menu" name="Luxembourg" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_user"/>

        <record id="lu_2011_chart_1" model="account.chart.template">
            <field name="name">PCMN Luxembourg</field>
            <field name="bank_account_code_prefix">513</field>
            <field name="cash_account_code_prefix">516</field>
            <field name="transfer_account_code_prefix">517</field>
            <field name="code_digits">6</field>
            <field name="currency_id" ref="base.EUR"/>
            <field name="spoken_languages" eval="'fr_FR;fr_BE;de_DE'"/>
        </record>
</odoo>

```

## File: i18n_extra\l10n_lu.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_lu
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 13.0+e\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2020-06-19 09:05+0000\n"
"PO-Revision-Date: 2020-06-19 09:05+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_VP-IC-EX
msgid " EX-IC-S-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_V-ART-43_60b
msgid "0-E-Art.43&60b"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_V-ART-44_56q
msgid "0-E-Art.44&56q"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_FB-PA-0
msgid "0-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_FP-PA-0
msgid "0-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IB-ECP-0
msgid "0-EC(P)-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_FP-EC-0
msgid "0-EC-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_IB-EC-0
msgid "0-EC-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_IP-EC-0
msgid "0-EC-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_VB-EC-0
msgid "0-EC-S-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_VP-EC-0
msgid "0-EC-S-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_VB-EC-Tab
msgid "0-EC-ST-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_IB-IC-0
msgid "0-IC-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_IP-IC-0
msgid "0-IC-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_VB-IC-0
msgid "0-IC-S-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_VP-IC-0
msgid "0-IC-S-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_VB-IC-Tab
msgid "0-IC-ST-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_VB-TR-0
msgid "0-ICT-S-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_IB-PA-0
msgid "0-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_IP-PA-0
msgid "0-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_AB-PA-0
msgid "0-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_AP-PA-0
msgid "0-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_SANS
msgid "0-P-Tax-Free"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_VB-PA-0
msgid "0-S-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_VP-PA-0
msgid "0-S-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_SANS_sale
msgid "0-S-Tax-Free"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_VB-PA-Tab
msgid "0-ST-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_AB-EC-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_AB-ECP-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_AB-IC-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_AB-PA-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_AP-EC-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_AP-IC-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_AP-PA-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_FB-EC-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_FB-ECP-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_FB-IC-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_FB-PA-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_FP-EC-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_FP-IC-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_FP-PA-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_IB-EC-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_IB-ECP-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_IB-IC-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_IB-PA-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_IP-EC-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_IP-IC-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_IP-PA-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_VB-PA-14
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_VP-PA-14
msgid "14%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FB-PA-14
msgid "14-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FP-PA-14
msgid "14-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FB-ECP-14
msgid "14-EC(P)-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IB-ECP-14
msgid "14-EC(P)-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AB-ECP-14
msgid "14-EC(P)-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FB-EC-14
msgid "14-EC-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FP-EC-14
msgid "14-EC-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IB-EC-14
msgid "14-EC-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IP-EC-14
msgid "14-EC-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AB-EC-14
msgid "14-EC-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AP-EC-14
msgid "14-EC-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FB-IC-14
msgid "14-IC-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FP-IC-14
msgid "14-IC-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IB-IC-14
msgid "14-IC-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IP-IC-14
msgid "14-IC-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AB-IC-14
msgid "14-IC-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AP-IC-14
msgid "14-IC-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IB-PA-14
msgid "14-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IP-PA-14
msgid "14-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AB-PA-14
msgid "14-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AP-PA-14
msgid "14-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_VB-PA-14
msgid "14-S-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_VP-PA-14
msgid "14-S-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_AB-EC-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_AB-ECP-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_AB-IC-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_AB-PA-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_AP-EC-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_AP-IC-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_AP-PA-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_FB-EC-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_FB-ECP-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_FB-IC-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_FB-PA-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_FP-EC-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_FP-IC-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_FP-PA-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_IB-EC-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_IB-ECP-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_IB-IC-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_IB-PA-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_IP-EC-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_IP-IC-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_IP-PA-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_VB-PA-17
#: model:account.tax.template,description:l10n_lu.lu_2015_tax_VP-PA-17
msgid "17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FB-PA-17
msgid "17-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FP-PA-17
msgid "17-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FB-ECP-17
msgid "17-EC(P)-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IB-ECP-17
msgid "17-EC(P)-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AB-ECP-17
msgid "17-EC(P)-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FB-EC-17
msgid "17-EC-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FP-EC-17
msgid "17-EC-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IB-EC-17
msgid "17-EC-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IP-EC-17
msgid "17-EC-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AB-EC-17
msgid "17-EC-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AP-EC-17
msgid "17-EC-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FB-IC-17
msgid "17-IC-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FP-IC-17
msgid "17-IC-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IB-IC-17
msgid "17-IC-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IP-IC-17
msgid "17-IC-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AB-IC-17
msgid "17-IC-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AP-IC-17
msgid "17-IC-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IB-PA-17
msgid "17-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IP-PA-17
msgid "17-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AB-PA-17
msgid "17-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AP-PA-17
msgid "17-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_VB-PA-17
msgid "17-S-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_VP-PA-17
msgid "17-S-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_FB-PA-3
msgid "3-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_FP-PA-3
msgid "3-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FB-ECP-3
msgid "3-EC(P)-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IB-ECP-3
msgid "3-EC(P)-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AB-ECP-3
msgid "3-EC(P)-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_FB-EC-3
msgid "3-EC-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_FP-EC-3
msgid "3-EC-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_IB-EC-3
msgid "3-EC-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_IP-EC-3
msgid "3-EC-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_AB-EC-3
msgid "3-EC-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_AP-EC-3
msgid "3-EC-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_FB-IC-3
msgid "3-IC-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_FP-IC-3
msgid "3-IC-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_IB-IC-3
msgid "3-IC-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_IP-IC-3
msgid "3-IC-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_AB-IC-3
msgid "3-IC-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_AP-IC-3
msgid "3-IC-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_IB-PA-3
msgid "3-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_IP-PA-3
msgid "3-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_AB-PA-3
msgid "3-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_AP-PA-3
msgid "3-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_VB-PA-3
msgid "3-S-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_VP-PA-3
msgid "3-S-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FB-PA-8
msgid "8-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FP-PA-8
msgid "8-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FB-ECP-8
msgid "8-EC(P)-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IB-ECP-8
msgid "8-EC(P)-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AB-ECP-8
msgid "8-EC(P)-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FB-EC-8
msgid "8-EC-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FP-EC-8
msgid "8-EC-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IB-EC-8
msgid "8-EC-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IP-EC-8
msgid "8-EC-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AB-EC-8
msgid "8-EC-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AP-EC-8
msgid "8-EC-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FB-IC-8
msgid "8-IC-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FP-IC-8
msgid "8-IC-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IB-IC-8
msgid "8-IC-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IP-IC-8
msgid "8-IC-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AB-IC-8
msgid "8-IC-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AP-IC-8
msgid "8-IC-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IB-PA-8
msgid "8-IG"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_IP-PA-8
msgid "8-IS"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AB-PA-8
msgid "8-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AP-PA-8
msgid "8-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_VB-PA-8
msgid "8-S-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_VP-PA-8
msgid "8-S-S"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_46128
msgid "ACD - Other amounts payable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_42148
msgid "ACD - Other amounts receivable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_46148
msgid "AED - Other debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65112
msgid "AVA on amounts owed by affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6352
msgid ""
"AVA on amounts owed by affiliated undertakings and undertakings with which "
"the undertaking is linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65114
msgid ""
"AVA on amounts owed by undertakings with which the undertaking is linked by "
"virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_63313
msgid "AVA on buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6322
msgid ""
"AVA on concessions, patents, licences, trademarks and similar rights and "
"assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6321
msgid "AVA on development costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6324
msgid "AVA on down payments and intangible fixed assets under development"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6334
msgid "AVA on down payments and tangible fixed assets under development"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6345
msgid "AVA on down payments on inventories"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6313
msgid ""
"AVA on expenses for capital increases and various operations (mergers, "
"demergers, changes of legal form)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_63314
msgid "AVA on fixtures and fittings-out of buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_63312
msgid "AVA on fixtures and fittings-out of land"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6323
msgid "AVA on goodwill acquired for consideration"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6343
msgid "AVA on inventories of goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6344
msgid "AVA on inventories of merchandise and other goods for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6341
msgid "AVA on inventories of raw materials and consumables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6342
msgid "AVA on inventories of work and contracts in progress"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_63311
msgid "AVA on land"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6314
msgid "AVA on loan-issuance expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65116
msgid "AVA on loans, deposits and claims held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6333
msgid ""
"AVA on other fixtures and fittings, tools and equipment (including rolling "
"stock)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6353
msgid "AVA on other receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6318
msgid "AVA on other similar expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65318
msgid "AVA on other transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65312
msgid "AVA on own shares or own corporate units"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65113
msgid "AVA on participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6332
msgid "AVA on plant and machinery"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65115
msgid "AVA on securities held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6311
msgid "AVA on set-up and start-up costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65111
#: model:account.account.template,name:l10n_lu.lu_2011_account_65311
msgid "AVA on shares in affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65313
msgid ""
"AVA on shares in undertakings with which the undertaking is linked by virtue"
" of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6351
msgid "AVA on trade receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106142
msgid "Accident insurance"
msgstr ""

#. module: l10n_lu
#: model:ir.model,name:l10n_lu.model_account_chart_template
msgid "Account Chart Template"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61342
msgid "Accounting, tax consulting, auditing and similar fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_16121
msgid "Acquired against payment (except Goodwill)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_771
msgid "Adjustments of corporate income tax (CIT)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_773
msgid "Adjustments of foreign income taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_783
msgid "Adjustments of foreign taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_772
msgid "Adjustments of municipal business tax (MBT)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_781
msgid "Adjustments of net wealth tax (NWT)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_788
msgid "Adjustments of other taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_782
msgid "Adjustments of subscription tax"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42111
msgid "Advances and down payments"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6591
msgid "Allocations to financial provisions - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6592
msgid "Allocations to financial provisions - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6492
msgid "Allocations to operating provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_679
msgid "Allocations to provisions for deferred taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6491
msgid "Allocations to tax provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_647
msgid "Allocations to tax-exempt capital gains"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_232
#: model:account.account.template,name:l10n_lu.lu_2011_account_6452
#: model:account.account.template,name:l10n_lu.lu_2011_account_75112
#: model:account.account.template,name:l10n_lu.lu_2020_account_65212
#: model:account.account.template,name:l10n_lu.lu_2020_account_75212
msgid "Amounts owed by affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_4212
#: model:account.account.template,name:l10n_lu.lu_2020_account_4222
msgid ""
"Amounts owed by partners and shareholders (others than from affiliated "
"undertakings)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_75114
msgid ""
"Amounts owed by undertakings with which the company is linked by virtue of "
"participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_234
#: model:account.account.template,name:l10n_lu.lu_2011_account_6453
#: model:account.account.template,name:l10n_lu.lu_2020_account_65214
#: model:account.account.template,name:l10n_lu.lu_2020_account_75214
msgid ""
"Amounts owed by undertakings with which the undertaking is linked by virtue "
"of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4713
#: model:account.account.template,name:l10n_lu.lu_2011_account_4723
msgid "Amounts payable to directors, managers, statutory auditors and similar"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_4712
#: model:account.account.template,name:l10n_lu.lu_2020_account_4722
msgid ""
"Amounts payable to partners and shareholders (others than from affiliated "
"undertakings)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_4714
#: model:account.account.template,name:l10n_lu.lu_2020_account_4724
msgid "Amounts payable to staff"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_60813
msgid "Architects' and engineers' fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6431
msgid "Attendance fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_743
msgid "Attendance fees, director's fees and similar remunerations"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61333
msgid ""
"Bank account charges and bank commissions (included custody fees on "
"securities)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65521
msgid "Banking interest on current accounts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65522
msgid "Banking interest on financing operations"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_5131
msgid "Banks and CCP : available balance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_5132
msgid "Banks and CCP : overdraft"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6467
msgid "Bar licence tax"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_62111
msgid "Base wages"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_62115
#: model:account.account.template,name:l10n_lu.lu_2011_account_746
msgid "Benefits in kind"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4422
msgid "Bills of exchange payable after more than one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4421
msgid "Bills of exchange payable within one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652221
#: model:account.account.template,name:l10n_lu.lu_2020_account_752221
msgid "Book value of yielded amounts owed by affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652241
#: model:account.account.template,name:l10n_lu.lu_2020_account_752241
msgid ""
"Book value of yielded amounts owed by undertakings with which the "
"undertaking is linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_64411
#: model:account.account.template,name:l10n_lu.lu_2020_account_74411
msgid "Book value of yielded intangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652261
#: model:account.account.template,name:l10n_lu.lu_2020_account_752261
msgid "Book value of yielded loans, deposits and claims held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652231
#: model:account.account.template,name:l10n_lu.lu_2020_account_752231
msgid "Book value of yielded participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652251
#: model:account.account.template,name:l10n_lu.lu_2020_account_752251
msgid "Book value of yielded securities held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652211
#: model:account.account.template,name:l10n_lu.lu_2020_account_752211
msgid "Book value of yielded shares in affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_64421
#: model:account.account.template,name:l10n_lu.lu_2020_account_74421
msgid "Book value of yielded tangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61112
#: model:account.account.template,name:l10n_lu.lu_2011_account_61221
#: model:account.account.template,name:l10n_lu.lu_2011_account_61411
msgid "Buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_60763
msgid "Buildings for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_22132
msgid "Buildings in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_314
msgid "Buildings under construction"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61523
msgid "Business assignments"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6144
msgid "Business risk insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10629
msgid "Business share in private expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461212
msgid "CIT - Tax payable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6711
msgid "CIT - current financial year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6712
msgid "CIT - previous financial years"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_115
msgid "Capital contribution without issue of shares"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7473
msgid "Capital investment subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_104
msgid "Capital of individual companies, corporate partnerships and similar"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106166
msgid "Car"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_516
msgid "Cash in hand"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10611
msgid "Cash withdrawals (daily life)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6222
msgid "Casual workers"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61515
msgid "Catalogues, printed materials and publications"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7121
msgid "Change in inventories of finished goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7123
msgid "Change in inventories of residual goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7122
msgid "Change in inventories of semi-finished goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7111
msgid "Change in inventories of work in progress"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7114
msgid "Change in inventories: buildings under construction"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7112
msgid "Change in inventories: contracts in progress - goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7113
msgid "Change in inventories: contracts in progress - services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6073
msgid "Changes in inventory of consumable materials and supplies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6074
msgid "Changes in inventory of packaging"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6071
msgid "Changes in inventory of raw materials"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_62412
msgid "Changes to provisions for complementary pensions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61334
msgid "Charges for electronic means of payment"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106152
msgid "Child benefit office"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6165
msgid "Collective staff transportation"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6131
#: model:account.account.template,name:l10n_lu.lu_2020_account_705
msgid "Commissions and brokerage fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7453
msgid "Compensatory allowances"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_62415
msgid "Complementary pensions paid by the employer"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2235
msgid "Computer equipment"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_21211
#: model:account.account.template,name:l10n_lu.lu_2011_account_21221
#: model:account.account.template,name:l10n_lu.lu_2011_account_6411
#: model:account.account.template,name:l10n_lu.lu_2011_account_72121
#: model:account.account.template,name:l10n_lu.lu_2011_account_7411
#: model:account.account.template,name:l10n_lu.lu_2020_account_70311
msgid "Concessions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_312
msgid "Contracts in progress - goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_313
msgid "Contracts in progress - services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_113
msgid "Contribution premium"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6187
msgid "Contributions to professional associations"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_212151
#: model:account.account.template,name:l10n_lu.lu_2011_account_212251
#: model:account.account.template,name:l10n_lu.lu_2011_account_64151
#: model:account.account.template,name:l10n_lu.lu_2011_account_721251
#: model:account.account.template,name:l10n_lu.lu_2011_account_74151
#: model:account.account.template,name:l10n_lu.lu_2020_account_703151
msgid "Copyrights and reproduction rights"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_42141
msgid "Corporate income tax"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461211
msgid "Corporate income tax - Tax accrual"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6182
msgid "Costs of training, symposiums, seminars, conferences"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_16122
msgid "Created by the undertaking itself"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_67321
msgid "Current financial year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4011
#: model:account.account.template,name:l10n_lu.lu_2011_account_4021
msgid "Customers"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_40111
msgid "Customers (PoS)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4012
msgid "Customers - Receivable bills of exchange"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4014
msgid "Customers - Unbilled sales"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6145
msgid "Customers credit insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4015
msgid "Customers with a credit balance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4025
msgid "Customers with creditor balance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4215
#: model:account.account.template,name:l10n_lu.lu_2020_account_4613
msgid "Customs and Excise Authority (ADA)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106154
msgid "Death and other health insurance funds"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_5083
msgid "Debenture loans and other notes issued and repurchased by the company"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_23521
msgid "Debentures"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_481
msgid "Deferred charges (on one or more financial years)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_482
msgid "Deferred income (on one or more financial years)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_183
msgid "Deferred tax provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_2362
msgid "Deposits and guarantees paid"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106192
msgid "Deposits on private financial accounts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_42187
#: model:account.account.template,name:l10n_lu.lu_2020_account_42287
#: model:account.account.template,name:l10n_lu.lu_2020_account_4717
#: model:account.account.template,name:l10n_lu.lu_2020_account_4727
msgid "Derivative financial instruments"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_221111
msgid "Developed land"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_211
#: model:account.account.template,name:l10n_lu.lu_2011_account_7211
#: model:account.account.template,name:l10n_lu.lu_2020_account_1611
msgid "Development costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6432
msgid "Director's fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65551
msgid "Discounts and charges on bills of exchange - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65552
msgid "Discounts and charges on bills of exchange - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75551
msgid "Discounts on bills of exchange - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75552
msgid "Discounts on bills of exchange - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75561
msgid "Discounts received - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75562
msgid "Discounts received - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_752262
msgid "Disposal proceed of loans, deposits and claims held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652222
#: model:account.account.template,name:l10n_lu.lu_2020_account_752222
msgid "Disposal proceeds of amounts owed by affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652242
#: model:account.account.template,name:l10n_lu.lu_2020_account_752242
msgid ""
"Disposal proceeds of amounts owed by undertakings with which the undertaking"
" is linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_64412
#: model:account.account.template,name:l10n_lu.lu_2020_account_74412
msgid "Disposal proceeds of intangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652262
msgid "Disposal proceeds of loans, deposits and claims held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652232
#: model:account.account.template,name:l10n_lu.lu_2020_account_752232
msgid "Disposal proceeds of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652252
#: model:account.account.template,name:l10n_lu.lu_2020_account_752252
msgid "Disposal proceeds of securities held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652212
#: model:account.account.template,name:l10n_lu.lu_2020_account_752212
msgid "Disposal proceeds of shares in affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_64422
#: model:account.account.template,name:l10n_lu.lu_2020_account_74422
msgid "Disposal proceeds of tangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6181
msgid "Documentation"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61516
msgid "Donations"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4013
msgid "Doubtful or disputed customers"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_214
msgid "Down payments and intangible fixed assets under development"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_37
msgid "Down payments on account on inventories"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_432
msgid "Down payments received after more than one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_431
msgid "Down payments received within one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_1922
#: model:account.account.template,name:l10n_lu.lu_2020_account_1932
#: model:account.account.template,name:l10n_lu.lu_2020_account_1942
msgid "Due and payable after more than one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_1921
#: model:account.account.template,name:l10n_lu.lu_2020_account_1931
#: model:account.account.template,name:l10n_lu.lu_2020_account_1941
msgid "Due and payable within one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6463
msgid "Duties on imported merchandise"
msgstr ""

#. module: l10n_lu
#: model:account.fiscal.position,name:l10n_lu.1_account_fiscal_position_template_private_LU_IC
#: model:account.fiscal.position.template,name:l10n_lu.account_fiscal_position_template_private_LU_IC
msgid "EU private"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_FB-ECP-0
msgid "EX-EC(P)-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2015_tax_AB-ECP-0
msgid "EX-EC(P)-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_FB-EC-0
msgid "EX-EC-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_AB-EC-0
msgid "EX-EC-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_AP-EC-0
msgid "EX-EC-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_FB-IC-0
msgid "EX-IC-E-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_FP-IC-0
msgid "EX-IC-E-S"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_AB-IC-0
msgid "EX-IC-P-G"
msgstr ""

#. module: l10n_lu
#: model:account.tax.template,name:l10n_lu.lu_2011_tax_AP-IC-0
msgid "EX-IC-P-S"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_60315
#: model:account.account.template,name:l10n_lu.lu_2020_account_61845
msgid "Electricity"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_105
msgid "Endowment of branches"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6464
msgid "Excise duties on production and tax on consumption"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_203
msgid ""
"Expenses for increases in capital and for various operations (merger, "
"demerger, change of legal form)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6172
msgid "External staff on secondment"
msgstr ""

#. module: l10n_lu
#: model:account.fiscal.position,name:l10n_lu.1_account_fiscal_position_template_LU_EC
#: model:account.fiscal.position.template,name:l10n_lu.account_fiscal_position_template_LU_EC
msgid "Extra-Community Taxable Person"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6512
#: model:account.account.template,name:l10n_lu.lu_2011_account_7512
msgid "FVA on financial fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_63315
#: model:account.account.template,name:l10n_lu.lu_2020_account_73315
msgid "FVA on investment properties"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6354
#: model:account.account.template,name:l10n_lu.lu_2020_account_7354
msgid "FVA on receivables from current assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6532
msgid "FVA on transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61336
msgid "Factoring services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7532
msgid "Fair value adjustments on transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61513
msgid "Fairs and exhibitions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6114
msgid "Financial leasing on real property"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_1882
msgid "Financial provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6481
msgid "Fines, sanctions and penalties"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106143
msgid "Fire insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_22141
msgid "Fixtures and fitting-outs of buildings in Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_22142
msgid "Fixtures and fitting-outs of buildings in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_22121
msgid "Fixtures and fitting-outs of land in Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_22122
msgid "Fixtures and fitting-outs of land in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4622
msgid "Foreign Social Security offices"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421811
#: model:account.account.template,name:l10n_lu.lu_2020_account_46151
msgid "Foreign VAT"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7561
msgid "Foreign currency exchange gains - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7562
msgid "Foreign currency exchange gains - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6561
msgid "Foreign currency exchange losses - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6562
msgid "Foreign currency exchange losses - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42172
msgid "Foreign social security offices"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_683
msgid "Foreign taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65411
#: model:account.account.template,name:l10n_lu.lu_2020_account_755231
msgid "From affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_755232
msgid "From other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65413
msgid "From other receivables from current assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65412
msgid ""
"From undertakings with which the undertaking is linked by virtue of "
"participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106145
msgid "Full coverage insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2234
msgid "Furniture"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_60313
#: model:account.account.template,name:l10n_lu.lu_2020_account_61843
msgid "Gas"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6121
msgid ""
"General subcontracting (not included in the production of goods and "
"services)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106194
msgid "Gifts and allowance to children"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61514
msgid "Gifts to customers"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_213
#: model:account.account.template,name:l10n_lu.lu_2020_account_1613
msgid "Goodwill acquired for consideration"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65561
msgid "Granted discounts - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65562
msgid "Granted discounts - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_212152
msgid "Greenhouse gas and similar emission quotas"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106153
msgid "Health insurance funds"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106163
msgid "Heating, gas, electricity"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1_assessment_taxable_turnover
msgid "I. ASSESSMENT OF TAXABLE TURNOVER"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1a_overall_turnover
msgid "I.A. Overall turnover (012)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1a_vat_acc_scheme
msgid "I.A.1. VAT accounting scheme"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1a_total_sale
msgid "I.A.2. Total Sales / Receipts (454)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1a_telecom_service
msgid ""
"I.A.2.a). Telecommunications services, radio and television broadcasting "
"services... (471)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1a_other_sales
msgid "I.A.2.b). Other sales / receipts (472)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1a_app_goods_non_bus
msgid ""
"I.A.3. Application of goods for non-business use and for business purposes "
"(Art.13) (455)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1a_non_bus_gs
msgid ""
"I.A.4. Non-business use of goods and supply of services free of charge "
"(Art.16) (456)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_exemptions_deductible_amounts
msgid "I.B. Exemptions and deductible amounts (021)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_1_intra_community_goods_pi_vat
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_1_intra_community_goods_pi_vat
msgid ""
"I.B.1. Intra-Community supply of goods to persons identified for VAT "
"purposes in another Member State (MS) (Art.43(1)(d),(e) and (f)) (3) (457)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_2_export
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_2_export
msgid "I.B.2. Exports (Art.43(1)(a) and (b)) (014)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_3_other_exemptions_art_43
msgid "I.B.3. Other exemptions (Art.43 and 60bis) (015)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_3_other_exemptions_art_43
msgid "I.B.3. Other exemptions (art.43 et 60bis) (015)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_4_other_exemptions_art_44_et_56quater
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_4_other_exemptions_art_44_et_56quater
msgid "I.B.4. Other exemptions (Art.44 and 56quater) (016)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_5_manufactured_tobacco_vat_collected
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_5_manufactured_tobacco_vat_collected
msgid ""
"I.B.5. Manufactured tobacco whose VAT was collected at the source or at the "
"exit of the tax... (017)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_6_a_subsequent_to_intra_community
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_6_a_subsequent_to_intra_community
msgid ""
"I.B.6.a) Supply, subsequent to intra-Community acquisitions of goods, in the"
" context of triangular transactions, when the customer identified,... (018)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_6_b1_non_exempt_customer_vat
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_6_b1_non_exempt_customer_vat
msgid ""
"I.B.6.b)1) not exempt in the MS where the customer is liable for payment of "
"VAT (Art.17(1)(b)) (5) (423)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_6_b2_exempt_ms_customer
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_6_b2_exempt_ms_customer
msgid ""
"I.B.6.b)2) exempt in the MS where the customer is identified (Art.17(1)(b)) "
"(424)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_6_c_supplies_scope_special_arrangement
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_6_c_supplies_scope_special_arrangement
msgid ""
"I.B.6.c) Supplies carried out within the scope of the special arrangement of"
" art. 56sexies (226)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_6_d_supplies_other_referred
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_6_d_supplies_other_referred
msgid "I.B.6.d) Supplies other than referred to in (6)(a) and (6)(b) (019)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_7_inland_supplies_for_customer
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_7_inland_supplies_for_customer
msgid ""
"I.B.7. Inland supplies for which the customer is liable for the payment of "
"VAT (419)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1c_taxable_turnover
msgid "I.C. Taxable turnover (022)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2_assesment_of_tax_due
msgid "II. ASSESSMENT OF TAX DUE (output tax)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_breakdown_taxable_turnover_base
msgid "II.A. Breakdown of taxable turnover – base (037)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_breakdown_taxable_turnover_tax
msgid "II.A. Breakdown of taxable turnover – tax (046)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_base_0
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_base_0
msgid "II.A. base 0%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_base_14
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_base_14
msgid "II.A. base 14% (703)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_base_17
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_base_17
msgid "II.A. base 17% (701)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_base_3
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_base_3
msgid "II.A. base 3% (031)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_base_8
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_base_8
msgid "II.A. base 8% (705)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_tax_0
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_tax_0
msgid "II.A. tax 0%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_tax_14
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_tax_14
msgid "II.A. tax 14% (704)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_tax_17
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_tax_17
msgid "II.A. tax 17% (702)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_tax_3
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_tax_3
msgid "II.A. tax 3% (040)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_tax_8
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_tax_8
msgid "II.A. tax 8% (706)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_intra_community_acqui_of_goods_base
msgid "II.B. Intra-Community acquisitions of goods – base (051)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_intra_community_acquisitions_goods_tax
msgid "II.B. Intra-Community acquisitions of goods – tax (056)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_base_14
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_base_14
msgid "II.B. base 14% (713)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_base_17
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_base_17
msgid "II.B. base 17% (711)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_base_3
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_base_3
msgid "II.B. base 3% (049)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_base_8
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_base_8
msgid "II.B. base 8% (715)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_base_exempt
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_base_exempt
msgid "II.B. base exempt (194)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_tax_14
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_tax_14
msgid "II.B. tax 14% (714)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_tax_17
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_tax_17
msgid "II.B. tax 17% (712)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_tax_3
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_tax_3
msgid "II.B. tax 3% (054)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_tax_8
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_tax_8
msgid "II.B. tax 8% (716)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2c_acquisitions_triangular_transactions_base
msgid ""
"II.C. Acquisitions, in the context of triangular transactions – base (152)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_importation_of_goods_base
msgid "II.D. Importation of goods – base (065)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_importation_of_goods_tax
msgid "II.D. Importation of goods – tax (407)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_14
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_base_14
msgid "II.D.1. for business purposes: base 14% (723)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_17
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_base_17
msgid "II.D.1. for business purposes: base 17% (721)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_3
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_base_3
msgid "II.D.1. for business purposes: base 3% (059)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_8
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_base_8
msgid "II.D.1. for business purposes: base 8% (725)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_exempt
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_base_exempt
msgid "II.D.1. for business purposes: base exempt (195)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_14
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_tax_14
msgid "II.D.1. for business purposes: tax 14% (724)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_17
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_tax_17
msgid "II.D.1. for business purposes: tax 17% (722)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_3
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_tax_3
msgid "II.D.1. for business purposes: tax 3% (068)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_8
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_tax_8
msgid "II.D.1. for business purposes: tax 8% (726)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_14
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_base_14
msgid "II.D.2. for non-business purposes: base 14% (733)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_17
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_base_17
msgid "II.D.2. for non-business purposes: base 17% (731)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_3
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_base_3
msgid "II.D.2. for non-business purposes: base 3% (063)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_8
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_base_8
msgid "II.D.2. for non-business purposes: base 8% (735)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_exempt
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_base_exempt
msgid "II.D.2. for non-business purposes: base exempt (196)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_14
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_tax_14
msgid "II.D.2. for non-business purposes: tax 14% (734)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_17
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_tax_17
msgid "II.D.2. for non-business purposes: tax 17% (732)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_3
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_tax_3
msgid "II.D.2. for non-business purposes: tax 3% (073)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_8
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_tax_8
msgid "II.D.2. for non-business purposes: tax 8% (736)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_supply_of_service_for_customer
msgid ""
"II.E. Supply of services for which the customer is liable for the payment of"
" VAT – base (409)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_supply_of_service_for_customer_liable_for_payment_tax
msgid ""
"II.E. Supply of services for which the customer is liable for the payment of"
" VAT – tax (410)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_base
msgid "II.E.1. base (436)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_14
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_base_14
msgid "II.E.1.a) not exempt within the territory: base 14% (743)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_17
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_base_17
msgid "II.E.1.a) not exempt within the territory: base 17% (741)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_3
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_base_3
msgid "II.E.1.a) not exempt within the territory: base 3% (431)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_8
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_base_8
msgid "II.E.1.a) not exempt within the territory: base 8% (745)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_14
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_tax_14
msgid "II.E.1.a) not exempt within the territory: tax 14% (744)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_17
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_tax_17
msgid "II.E.1.a) not exempt within the territory: tax 17% (742)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_3
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_tax_3
msgid "II.E.1.a) not exempt within the territory: tax 3% (432)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_8
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_tax_8
msgid "II.E.1.a) not exempt within the territory: tax 8% (746)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax
msgid "II.E.1.a) tax (462)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_b_exempt
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_b_exempt
msgid "II.E.1.b) exempt within the territory: exempt (435)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_base
msgid "II.E.2. base (463)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_14
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_base_14
msgid ""
"II.E.2. not established or residing within the Community: base 14% (753)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_17
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_base_17
msgid ""
"II.E.2. not established or residing within the Community: base 17% (751)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_3
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_base_3
msgid ""
"II.E.2. not established or residing within the Community: base 3% (441)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_8
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_base_8
msgid ""
"II.E.2. not established or residing within the Community: base 8% (755)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_exempt
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_exempt
msgid "II.E.2. not established or residing within the Community: exempt (445)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_14
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_tax_14
msgid ""
"II.E.2. not established or residing within the Community: tax 14% (754)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_17
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_tax_17
msgid ""
"II.E.2. not established or residing within the Community: tax 17% (752)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_3
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_tax_3
msgid "II.E.2. not established or residing within the Community: tax 3% (442)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_8
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_tax_8
msgid "II.E.2. not established or residing within the Community: tax 8% (756)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax
msgid "II.E.2. tax (464)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_3_base
msgid "II.E.3. base (765)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_3_base_17
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_3_base_17
msgid "II.E.3. suppliers established within the territory: base 17% (761)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_3_tax_17
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_3_tax_17
msgid "II.E.3. suppliers established within the territory: tax 17% (762)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_3_tax
msgid "II.E.3. tax (766)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_base
msgid ""
"II.F. Supply of goods for which the purchaser is liable for the payment of "
"VAT - base (767)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_tax
msgid ""
"II.F. Supply of goods for which the purchaser is liable for the payment of "
"VAT - tax (768)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_base_8
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2f_supply_goods_base_8
msgid "II.F. base 8% (763)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_tax_8
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2f_supply_goods_tax_8
msgid "II.F. tax 8% (764)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2g_special_arrangement
msgid ""
"II.G. Special arrangement for tax suspension: adjustment (Art.60bis, (5) and"
" (8)) (227)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2h_total_tax_due
msgid "II.H. Total tax due (076)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3_assessment_deducible_tax
msgid "III. ASSESSMENT OF DEDUCTIBLE TAX (input tax)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_total_input_tax
msgid "III.A. Total input tax (093)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_1_invoiced_by_other_taxable_person
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3a_1_invoiced_by_other_taxable_person
msgid ""
"III.A.1. Invoiced by other taxable persons for goods or services supplied "
"(Art.48(1)(a)) (458)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_2_due_respect_intra_comm_goods
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3a_2_due_respect_intra_comm_goods
msgid ""
"III.A.2. Due in respect of intra-Community acquisitions of goods "
"(Art.48(1)(b)) (459)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_3_due_paid_respect_importation_goods
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3a_3_due_paid_respect_importation_goods
msgid ""
"III.A.3. Due or paid in respect of importation of goods (Art.48(1)(c)) (460)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_4_due_respect_application_goods
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3a_4_due_respect_application_goods
msgid ""
"III.A.4. Due in respect of the application of goods for business purposes "
"(Art.48(1)(d)) (090)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_5_due_under_reverse_charge
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3a_5_due_under_reverse_charge
msgid "III.A.5. Due under the reverse charge (see points II.E and F) (461)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_6_paid_joint_several_guarantee
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3a_6_paid_joint_several_guarantee
msgid "III.A.6. Paid as joint and several guarantee (092)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_7_adjusted_tax_special_arrangement
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3a_7_adjusted_tax_special_arrangement
msgid ""
"III.A.7. Adjusted tax - special arrangement for tax suspension "
"(Art.60bis(9), subpar. 2) (228)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3b_total_input_tax_nd
msgid "III.B. Total input tax non-deductible (097)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3b1_rel_trans
msgid ""
"III.B.1. relating to transactions which are exempt pursuant to articles 44 "
"and 56quater (094)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3b2_ded_prop
msgid ""
"III.B.2. where the deductible proportion determined in accordance to article"
" 50 is applied (095)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3c_total_input_tax_deductible
msgid "III.C. Total input tax deductible (102)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6132
msgid "IT services"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_4_tax_tobe_paid_or_reclaimed
msgid "IV. TAX TO BE PAID OR TO BE RECLAIMED"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_4a_total_tax_due
msgid "IV.A. Total tax due (103)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_4a_total_input_tax_deductible
msgid "IV.B. Total input tax deductible (104)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_4c_exceeding_amount
msgid "IV.C. Exceeding amount (105)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_62114
msgid "Incentives, bonuses and commissions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106281
msgid "Income tax"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106181
msgid "Income tax paid"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_642
msgid "Indemnities, damages and interest"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6183
msgid "Industrial and non-industrial waste treatment"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10621
msgid "Inheritance or donation"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106195
msgid "Inheritance taxes and mutation tax due to death"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_62414
msgid "Insolvency insurance premiums"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7481
msgid "Insurance indemnities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6142
msgid "Insurance on rented assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75541
msgid "Interest on amounts owed by affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75542
msgid ""
"Interest on amounts owed by undertakings with which the undertaking is "
"linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_75521
msgid "Interest on bank accounts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65511
msgid "Interest on debenture loans - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65512
msgid "Interest on debenture loans - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_655231
msgid "Interest on financial leases - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_655232
msgid "Interest on financial leases - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75581
msgid "Interest on other amounts receivable - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75582
msgid "Interest on other amounts receivable - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6553
msgid "Interest on trade payables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7553
msgid "Interest on trade receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65581
msgid "Interest payable on other loans and debts - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65582
msgid "Interest payable on other loans and debts - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65541
msgid "Interest payable to affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65542
msgid ""
"Interest payable to undertakings with which the undertaking is linked by "
"virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7452
msgid "Interest subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_15
msgid "Interim dividends"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_5172
msgid "Internal transfers : credit balance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_5171
msgid "Internal transfers : debit balance"
msgstr ""

#. module: l10n_lu
#: model:account.fiscal.position,name:l10n_lu.1_account_fiscal_position_template_LU_IC
#: model:account.fiscal.position.template,name:l10n_lu.account_fiscal_position_template_LU_IC
msgid "Intra-Community Taxable Person"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_3631
msgid "Inventories of buildings for resale in Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_3632
msgid "Inventories of buildings for resale in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_303
msgid "Inventories of consumable materials and supplies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_321
msgid "Inventories of finished goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_3621
msgid "Inventories of land for resale in Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_3622
msgid "Inventories of land for resale in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_361
msgid "Inventories of merchandise"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_304
msgid "Inventories of packaging"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_301
msgid "Inventories of raw materials"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_323
msgid ""
"Inventories of residual goods (waste, rejected and recuperable material)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_322
msgid "Inventories of semi-finished goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_311
msgid "Inventories of work in progress"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_22151
msgid "Investment properties in Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_22152
msgid "Investment properties in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42131
#: model:account.account.template,name:l10n_lu.lu_2011_account_42231
msgid "Investment subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61111
msgid "Land"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_60762
msgid "Land for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_22112
msgid "Land in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_22411
msgid "Land, fitting-outs and buildings in Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_22412
msgid "Land, fitting-outs and buildings in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7221
msgid "Land, fittings and buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_47162
#: model:account.account.template,name:l10n_lu.lu_2020_account_47262
msgid "Lease debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_131
msgid "Legal reserve"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61341
msgid "Legal, litigation and similar fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_47163
#: model:account.account.template,name:l10n_lu.lu_2020_account_47263
msgid "Life annuities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106141
msgid "Life insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_486
msgid "Linking accounts (branches) - Assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_487
msgid "Linking accounts (branches) - Liabilities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_60312
msgid "Liquid fuels"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_61842
msgid "Liquid fuels (oil, motor fuel, etc.)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_chart_1_liquidity_transfer
msgid "Liquidity Transfer"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_5084
msgid "Listed debenture loans"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_235111
msgid "Listed shares"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2236
msgid "Livestock"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_204
msgid "Loan issuances expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_2361
msgid "Loans"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_41112
#: model:account.account.template,name:l10n_lu.lu_2011_account_41122
#: model:account.account.template,name:l10n_lu.lu_2011_account_41212
#: model:account.account.template,name:l10n_lu.lu_2011_account_41222
#: model:account.account.template,name:l10n_lu.lu_2011_account_45112
#: model:account.account.template,name:l10n_lu.lu_2011_account_45122
#: model:account.account.template,name:l10n_lu.lu_2011_account_45212
#: model:account.account.template,name:l10n_lu.lu_2011_account_45222
msgid "Loans and advances"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61332
msgid "Loans' issuance expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_75116
#: model:account.account.template,name:l10n_lu.lu_2020_account_65216
#: model:account.account.template,name:l10n_lu.lu_2020_account_75216
msgid "Loans, deposits and claims held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2363
msgid "Long-term receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6037
msgid "Lubricants"
msgstr ""

#. module: l10n_lu
#: model:ir.ui.menu,name:l10n_lu.account_reports_lu_statements_menu
msgid "Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.fiscal.position,name:l10n_lu.1_account_fiscal_position_template_LU_LU
#: model:account.fiscal.position.template,name:l10n_lu.account_fiscal_position_template_LU_LU
msgid "Luxembourgish Taxable Person"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461221
msgid "MBT - Tax accrual"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461222
msgid "MBT - Tax payable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6721
msgid "MBT - current financial year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6722
msgid "MBT - previous financial years"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2222
msgid "Machinery"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6032
#: model:account.account.template,name:l10n_lu.lu_2020_account_61854
msgid "Maintenance supplies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_615211
msgid "Management (respectively owner and partner)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_60761
msgid "Merchandise"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_112
msgid "Merger premium"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6488
msgid "Miscellaneous operating charges"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7488
msgid "Miscellaneous operating income"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_221313
msgid "Mixed-use buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6036
msgid "Motor fuels"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2232
msgid "Motor vehicles"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6466
msgid "Motor-vehicle taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_4611
msgid "Municipal authorities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_42142
msgid "Municipal business tax"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106284
msgid "Municipal business tax (MBT)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106183
msgid "Municipal business tax - payment in arrears"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461231
msgid "NWT - Tax accrual"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461232
msgid "NWT - Tax payable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6811
msgid "NWT - current financial year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6812
msgid "NWT - previous financial years"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_42143
msgid "Net wealth tax"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6462
msgid "Non-refundable VAT"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_221312
msgid "Non-residential buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7099
msgid "Not allocated rebates, discounts and refunds"
msgstr ""

#. module: l10n_lu
#: model:account.fiscal.position,name:l10n_lu.1_account_fiscal_position_template_LU_NO
#: model:account.fiscal.position.template,name:l10n_lu.account_fiscal_position_template_LU_NO
msgid "Not liable to VAT"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6135
msgid "Notarial and similar fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6035
msgid "Office and administrative supplies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_61851
msgid "Office supplies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_75411
msgid "On affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_75413
msgid "On other current receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_75412
msgid ""
"On undertakings with which the undertaking is linked by virtue of "
"participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_1881
msgid "Operating provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42132
#: model:account.account.template,name:l10n_lu.lu_2011_account_42232
msgid "Operating subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61418
#: model:account.account.template,name:l10n_lu.lu_2011_account_6228
#: model:account.account.template,name:l10n_lu.lu_2020_account_61128
#: model:account.account.template,name:l10n_lu.lu_2020_account_61158
#: model:account.account.template,name:l10n_lu.lu_2020_account_61228
#: model:account.account.template,name:l10n_lu.lu_2020_account_61858
msgid "Other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106178
msgid "Other acquisitions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61338
msgid ""
"Other banking and similar services (except interest and similar expenses)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6218
msgid "Other benefits"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_221318
msgid "Other buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_168
msgid "Other capital investment subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_518
msgid "Other cash amounts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_708
msgid "Other components of turnover"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6038
msgid "Other consumable supplies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106158
msgid "Other contributions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106248
msgid "Other disposals"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6468
msgid "Other duties and taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6581
msgid "Other financial charges - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6582
msgid "Other financial charges - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7581
msgid "Other financial income - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7582
msgid "Other financial income - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2238
msgid "Other fixtures"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7223
#: model:account.account.template,name:l10n_lu.lu_2011_account_7333
msgid ""
"Other fixtures and fittings, tools and equipment (included motor vehicles)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2243
msgid ""
"Other fixtures and fittings, tools and equipment (including rolling stock)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6738
msgid "Other foreign income taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421818
#: model:account.account.template,name:l10n_lu.lu_2020_account_46158
msgid "Other foreign taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106168
msgid "Other in kind withdrawals"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421628
#: model:account.account.template,name:l10n_lu.lu_2011_account_461428
msgid "Other indirect taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6148
msgid "Other insurances"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_221118
msgid "Other land"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_47161
#: model:account.account.template,name:l10n_lu.lu_2020_account_47261
msgid "Other loans"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4718
#: model:account.account.template,name:l10n_lu.lu_2011_account_4728
msgid "Other miscellaneous debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6188
msgid "Other miscellaneous external charges"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42188
#: model:account.account.template,name:l10n_lu.lu_2011_account_42288
msgid "Other miscellaneous receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_5088
msgid "Other miscellaneous transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_45118
#: model:account.account.template,name:l10n_lu.lu_2011_account_45128
#: model:account.account.template,name:l10n_lu.lu_2011_account_45218
#: model:account.account.template,name:l10n_lu.lu_2011_account_45228
msgid "Other payables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106148
msgid "Other private insurance premiums"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61348
msgid "Other professional fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6088
msgid "Other purchases included in the production of goods and services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61518
msgid "Other purchases of advertising services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6082
msgid ""
"Other purchases of material included in the production of goods and services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_41118
#: model:account.account.template,name:l10n_lu.lu_2011_account_41128
#: model:account.account.template,name:l10n_lu.lu_2011_account_41218
#: model:account.account.template,name:l10n_lu.lu_2011_account_41228
#: model:account.account.template,name:l10n_lu.lu_2011_account_42168
#: model:account.account.template,name:l10n_lu.lu_2020_account_6454
msgid "Other receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_64658
msgid "Other registration fees, stamp duties and mortgage duties"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6138
msgid "Other remuneration of intermediaries and professional fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_1381
msgid "Other reserves available for distribution"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_128
msgid "Other revaluation reserves"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2358
msgid "Other securities held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_23528
msgid "Other securities held as fixed assets (creditor's right)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_23518
msgid "Other securities held as fixed assets (equity right)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_47164
#: model:account.account.template,name:l10n_lu.lu_2020_account_47264
msgid "Other similar debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_208
msgid "Other similar expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6438
msgid "Other similar remuneration"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_64158
#: model:account.account.template,name:l10n_lu.lu_2011_account_721258
#: model:account.account.template,name:l10n_lu.lu_2011_account_74158
#: model:account.account.template,name:l10n_lu.lu_2020_account_703158
msgid "Other similar rights and assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_212158
msgid "Other similar rights and assets acquired for consideration"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_212258
msgid "Other similar rights and assets created by the undertaking itself"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42178
#: model:account.account.template,name:l10n_lu.lu_2011_account_4628
msgid "Other social bodies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6232
msgid "Other social security costs (including illness, accidents, a.s.o.)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106198
msgid "Other special private withdrawals"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6248
msgid "Other staff expenses not mentioned above"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42138
#: model:account.account.template,name:l10n_lu.lu_2011_account_42238
msgid "Other subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7458
msgid "Other subsidies for operating activities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_621128
msgid "Other supplements"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106288
msgid "Other tax refunds"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106188
#: model:account.account.template,name:l10n_lu.lu_2011_account_688
msgid "Other taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_75488
#: model:account.account.template,name:l10n_lu.lu_2020_account_65428
#: model:account.account.template,name:l10n_lu.lu_2020_account_75318
#: model:account.account.template,name:l10n_lu.lu_2020_account_75428
msgid "Other transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6168
msgid "Other transportation"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_60814
msgid "Outsourcing included in the production of goods and services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_621123
msgid "Overtime"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_75482
#: model:account.account.template,name:l10n_lu.lu_2020_account_65422
#: model:account.account.template,name:l10n_lu.lu_2020_account_75312
#: model:account.account.template,name:l10n_lu.lu_2020_account_75422
msgid "Own shares or corporate units"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_502
msgid "Own shares or own corporate units"
msgstr ""

#. module: l10n_lu
#: model:account.chart.template,name:l10n_lu.lu_2011_chart_1
msgid "PCMN Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_233
#: model:account.account.template,name:l10n_lu.lu_2011_account_75113
#: model:account.account.template,name:l10n_lu.lu_2020_account_65213
#: model:account.account.template,name:l10n_lu.lu_2020_account_75213
msgid "Participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_21212
#: model:account.account.template,name:l10n_lu.lu_2011_account_21222
#: model:account.account.template,name:l10n_lu.lu_2011_account_6412
#: model:account.account.template,name:l10n_lu.lu_2011_account_72122
#: model:account.account.template,name:l10n_lu.lu_2011_account_7412
#: model:account.account.template,name:l10n_lu.lu_2020_account_70312
msgid "Patents"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10622
msgid "Personal holdings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2221
msgid "Plant"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2242
#: model:account.account.template,name:l10n_lu.lu_2011_account_7222
msgid "Plant and machinery"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61531
msgid "Postal charges"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_62411
msgid "Premiums for external pensions funds"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_114
msgid "Premiums on conversion of bonds into shares"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61511
msgid "Press advertising"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_67322
msgid "Previous financial years"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106174
#: model:account.account.template,name:l10n_lu.lu_2011_account_106244
msgid "Private buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106172
#: model:account.account.template,name:l10n_lu.lu_2011_account_106242
msgid "Private car"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106171
#: model:account.account.template,name:l10n_lu.lu_2011_account_106241
msgid "Private furniture"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106173
msgid "Private held securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10623
msgid "Private loans"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10613
msgid "Private share of medical services expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106243
msgid "Private shares / bonds"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7451
msgid "Product subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_221112
msgid "Property rights and similar"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_181
msgid "Provisions for pensions and similar obligations"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_182
msgid "Provisions for taxation"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_621122
msgid "Public holidays"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6083
msgid "Purchase of greenhouse gas and similar emission quotas"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_45111
#: model:account.account.template,name:l10n_lu.lu_2011_account_45121
#: model:account.account.template,name:l10n_lu.lu_2011_account_45211
#: model:account.account.template,name:l10n_lu.lu_2011_account_45221
msgid "Purchases and services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6063
msgid "Purchases of buildings for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6062
msgid "Purchases of land for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6061
msgid "Purchases of merchandise"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_604
msgid "Purchases of packaging"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_601
msgid "Purchases of raw materials"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7095
msgid "RDR on commissions and brokerage fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7098
msgid "RDR on other components of turnover"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6098
msgid "RDR on purchases included in the production of goods and services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6093
msgid "RDR on purchases of consumable materials and supplies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6096
msgid "RDR on purchases of merchandise and other goods for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6094
msgid "RDR on purchases of packaging"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6091
msgid "RDR on purchases of raw materials"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7092
msgid "RDR on sales of goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7096
msgid "RDR on sales of merchandise and other goods for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7094
msgid "RDR on sales of packages"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7093
msgid "RDR on sales of services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7352
msgid ""
"RVA on amounts owed by affiliated undertakings and undertakings with which "
"the undertaking is linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_73313
msgid "RVA on buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7322
msgid ""
"RVA on concessions, patents, licences, trademarks and similar rights and "
"assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7321
msgid "RVA on development costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7324
msgid "RVA on down payments and intangible fixed assets under development"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7334
msgid "RVA on down payments and tangible fixed assets under development"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7345
msgid "RVA on down payments on inventories"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_73314
msgid "RVA on fixtures and fittings-out of buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_73312
msgid "RVA on fixtures and fittings-out of land"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7343
msgid "RVA on inventories of goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7344
msgid "RVA on inventories of merchandise and other goods for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7341
msgid "RVA on inventories of raw materials and consumables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7342
msgid "RVA on inventories of work and contracts in progress"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_73311
msgid "RVA on land"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7353
msgid "RVA on other receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7332
msgid "RVA on plant and machinery"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7351
msgid "RVA on trade receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6461
msgid "Real property tax"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_619
msgid "Rebates, discounts and refunds received on other external charges"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10627
msgid "Received child benefit"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_4711
#: model:account.account.template,name:l10n_lu.lu_2020_account_4721
msgid "Received deposits and guarantees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10625
msgid "Received rents"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10626
msgid "Received wages or pensions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61524
msgid "Receptions and entertainment costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106193
msgid "Refund of private debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6219
msgid "Refunds on wages paid"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421621
#: model:account.account.template,name:l10n_lu.lu_2011_account_461421
msgid "Registration duties"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_64651
msgid "Registration fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61522
msgid "Relocation expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106162
msgid "Rent"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_70322
msgid "Rental income from movable property"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_70321
msgid "Rental income from real property"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7422
msgid "Rental income on movable property"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7421
msgid "Rental income on real property"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106191
msgid "Repairs to private buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_60812
msgid "Research and development"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_13821
msgid "Reserve for net wealth tax (NWT)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_132
msgid "Reserves for own shares or own corporate units"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_13822
msgid "Reserves in application of fair value"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_122
msgid "Reserves in application of the equity method"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_13828
msgid "Reserves not available for distribution not mentioned above"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_133
msgid "Reserves provided for by the articles of association"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_221311
msgid "Residential buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_142
msgid "Result for the financial year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_1412
msgid "Results brought forward (assigned)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_1411
msgid "Results brought forward in the process of assignment"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2237
msgid "Returnable packaging"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7591
msgid "Reversals of financial provisions - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7592
msgid "Reversals of financial provisions - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7492
msgid "Reversals of operating provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_779
msgid "Reversals of provisions for deferred taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7491
msgid "Reversals of provisions for taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61123
#: model:account.account.template,name:l10n_lu.lu_2011_account_61153
#: model:account.account.template,name:l10n_lu.lu_2011_account_61223
#: model:account.account.template,name:l10n_lu.lu_2011_account_61412
msgid "Rolling stock"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_703001
msgid "Sale of Services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7063
msgid "Sales of buildings for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7021
msgid "Sales of finished goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7062
msgid "Sales of land resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7061
msgid "Sales of merchandise"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_704
msgid "Sales of packaging"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7023
msgid "Sales of residual products"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7022
msgid "Sales of semi-finished goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7039
msgid "Sales of services in the course of completion"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7033
msgid "Sales of services not mentioned above"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7029
msgid "Sales of work in progress"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61512
msgid "Samples"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_75115
#: model:account.account.template,name:l10n_lu.lu_2020_account_65215
#: model:account.account.template,name:l10n_lu.lu_2020_account_75215
msgid "Securities held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6113
msgid "Service charges and co-ownership expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_201
msgid "Set-up and start-up costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_62116
msgid "Severance pay"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_657
msgid ""
"Share in the losses of undertakings accounted for under the equity method"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_757
msgid ""
"Share of profit from undertakings accounted for under the equity method"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_111
msgid "Share premium"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_5081
msgid "Shares - listed securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_5082
msgid "Shares - unlisted securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_231
#: model:account.account.template,name:l10n_lu.lu_2011_account_501
#: model:account.account.template,name:l10n_lu.lu_2011_account_75111
#: model:account.account.template,name:l10n_lu.lu_2011_account_75481
#: model:account.account.template,name:l10n_lu.lu_2020_account_65211
#: model:account.account.template,name:l10n_lu.lu_2020_account_65421
#: model:account.account.template,name:l10n_lu.lu_2020_account_75211
#: model:account.account.template,name:l10n_lu.lu_2020_account_75311
#: model:account.account.template,name:l10n_lu.lu_2020_account_75421
msgid "Shares in affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_503
#: model:account.account.template,name:l10n_lu.lu_2011_account_75483
#: model:account.account.template,name:l10n_lu.lu_2020_account_65423
#: model:account.account.template,name:l10n_lu.lu_2020_account_75313
#: model:account.account.template,name:l10n_lu.lu_2020_account_75423
msgid ""
"Shares in undertakings with which the undertaking is linked by virtue of "
"participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_2353
msgid "Shares of collective investment funds"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_61852
msgid "Small equipment"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106151
msgid "Social Security"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42171
#: model:account.account.template,name:l10n_lu.lu_2011_account_4621
msgid "Social Security office (CCSS)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6231
msgid "Social security on pensions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_21213
#: model:account.account.template,name:l10n_lu.lu_2011_account_21223
#: model:account.account.template,name:l10n_lu.lu_2011_account_6413
#: model:account.account.template,name:l10n_lu.lu_2011_account_72123
#: model:account.account.template,name:l10n_lu.lu_2011_account_7413
#: model:account.account.template,name:l10n_lu.lu_2020_account_70313
msgid "Software licences"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_60311
#: model:account.account.template,name:l10n_lu.lu_2020_account_61841
msgid "Solid fuels"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61517
msgid "Sponsorship"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_615212
msgid "Staff"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_4221
msgid "Staff - advances and down payments"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4715
msgid ""
"State - Greenhous gas and similar emission quotas to be returned or acquired"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_483
msgid "State - Greenhouse gas and similar emission quotas received"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6221
msgid "Students"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_101
msgid "Subscribed capital"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_103
msgid "Subscribed capital called but unpaid"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_102
msgid "Subscribed capital not called"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421622
#: model:account.account.template,name:l10n_lu.lu_2011_account_461422
#: model:account.account.template,name:l10n_lu.lu_2011_account_682
msgid "Subscription tax"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7454
msgid "Subsidies in favour of employment development"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_1621
msgid "Subsidies on land, fitting-outs and buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_1623
msgid ""
"Subsidies on other fixtures, fittings, tools and equipment (including "
"rolling stock)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_1622
msgid "Subsidies on plant and machinery"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_621121
msgid "Sunday"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_44111
#: model:account.account.template,name:l10n_lu.lu_2011_account_44121
msgid "Suppliers"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_44112
msgid "Suppliers - invoices not yet received"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_44113
#: model:account.account.template,name:l10n_lu.lu_2020_account_44123
msgid "Suppliers with a debit balance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6186
msgid "Surveillance and security charges"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_62117
msgid "Survivor's pay"
msgstr ""

#. module: l10n_lu
#: model:account.tax.group,name:l10n_lu.tax_group_0
msgid "TVA 0%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.group,name:l10n_lu.tax_group_10
msgid "TVA 10%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.group,name:l10n_lu.tax_group_12
msgid "TVA 12%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.group,name:l10n_lu.tax_group_14
msgid "TVA 14%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.group,name:l10n_lu.tax_group_15
msgid "TVA 15%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.group,name:l10n_lu.tax_group_17
msgid "TVA 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.group,name:l10n_lu.tax_group_3
msgid "TVA 3%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.group,name:l10n_lu.tax_group_6
msgid "TVA 6%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.group,name:l10n_lu.tax_group_8
msgid "TVA 8%"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_60811
msgid "Tailoring"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6733
msgid "Taxes levied on non-resident undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61532
msgid "Telecommunication costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106165
msgid "Telephone"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7471
msgid "Temporarily not taxable capital gains not reinvested"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7472
#: model:account.account.template,name:l10n_lu.lu_2020_account_138232
msgid "Temporarily not taxable capital gains reinvested"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_138231
msgid "Temporarily not taxable capital gains to reinvest"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_123
msgid "Temporarily not taxable currency translation adjustments"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6171
msgid "Temporary staff"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106144
#: model:account.account.template,name:l10n_lu.lu_2011_account_6146
msgid "Third-party insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2233
msgid "Tools"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_41111
#: model:account.account.template,name:l10n_lu.lu_2011_account_41121
#: model:account.account.template,name:l10n_lu.lu_2011_account_41211
#: model:account.account.template,name:l10n_lu.lu_2011_account_41221
#: model:account.account.template,name:l10n_lu.lu_2011_account_6451
msgid "Trade receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6414
msgid "Trademarks and franchise"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_21214
#: model:account.account.template,name:l10n_lu.lu_2011_account_21224
#: model:account.account.template,name:l10n_lu.lu_2011_account_72124
#: model:account.account.template,name:l10n_lu.lu_2011_account_7414
#: model:account.account.template,name:l10n_lu.lu_2020_account_70314
msgid "Trademarks and franchises"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_484
msgid "Transitory or suspense accounts - Assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_485
msgid "Transitory or suspense accounts - Liabilities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6143
msgid "Transport insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2231
msgid "Transportation and handling equipment"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6161
msgid "Transportation of purchased goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6162
msgid "Transportation of sold goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6099
msgid "Unallocated RDR"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_5085
msgid "Unlisted debenture loans"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_235112
msgid "Unlisted shares"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461418
msgid "VAT - Other payables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421618
msgid "VAT - Other receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421613
msgid "VAT down payments made"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461413
msgid "VAT down payments received"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_421611
msgid "VAT paid and recoverable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461412
msgid "VAT payable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421612
msgid "VAT receivable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_461411
msgid "VAT received"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4019
#: model:account.account.template,name:l10n_lu.lu_2011_account_4029
#: model:account.account.template,name:l10n_lu.lu_2011_account_41119
#: model:account.account.template,name:l10n_lu.lu_2011_account_41129
#: model:account.account.template,name:l10n_lu.lu_2011_account_41219
#: model:account.account.template,name:l10n_lu.lu_2011_account_41229
#: model:account.account.template,name:l10n_lu.lu_2011_account_42119
#: model:account.account.template,name:l10n_lu.lu_2011_account_42189
#: model:account.account.template,name:l10n_lu.lu_2011_account_42289
msgid "Value adjustments"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106161
msgid "Wages"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106164
msgid "Water"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_60314
msgid "Water and sewage"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_61844
msgid "Water and waste water"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10612
msgid "Withdrawals of merchandise, finished products and services (at cost)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_62413
msgid "Withholding tax on complementary pensions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_46126
#: model:account.account.template,name:l10n_lu.lu_2020_account_42146
msgid "Withholding tax on director's fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_46125
#: model:account.account.template,name:l10n_lu.lu_2020_account_42145
msgid "Withholding tax on financial investment income"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_46124
#: model:account.account.template,name:l10n_lu.lu_2020_account_42144
msgid "Withholding tax on wages and salaries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6731
msgid "Withholding taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6034
#: model:account.account.template,name:l10n_lu.lu_2020_account_61853
msgid "Work clothes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6033
msgid "Workshop, factory and store supplies and small equipment"
msgstr ""

```

## File: models\account_chart_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    def get_countries_posting_at_bank_rec(self):
        rslt = super(AccountChartTemplate, self).get_countries_posting_at_bank_rec()
        rslt.append('LU')
        return rslt

    @api.model
    def _prepare_all_journals(self, acc_template_ref, company, journals_dict=None):
        journal_data = super(AccountChartTemplate, self)._prepare_all_journals(
            acc_template_ref, company, journals_dict)
        for journal in journal_data:
            if journal['type'] in ('sale', 'purchase') and company.country_id == self.env.ref('base.lu'):
                journal.update({'refund_sequence': True})
        return journal_data

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_chart_template
```

## File: scripts\tax2csv.py

```python
from collections import OrderedDict

import xlrd
from odoo.tools import pycompat

def _is_true(s):
    return s not in ('F', 'False', 0, '', None, False)


class LuxTaxGenerator:

    def __init__(self, filename):
        self.workbook = xlrd.open_workbook('tax.xls')
        self.sheet_info = \
            self.workbook.sheet_by_name('INFO')
        self.sheet_taxes = \
            self.workbook.sheet_by_name('TAXES')
        self.sheet_tax_codes = \
            self.workbook.sheet_by_name('TAX.CODES')
        self.sheet_fiscal_pos_map = \
            self.workbook.sheet_by_name('FISCAL.POSITION.MAPPINGS')
        self.suffix = self.sheet_info.cell_value(4, 2)

    def iter_tax_codes(self):
        keys = [c.value for c in self.sheet_tax_codes.row(0)]
        yield keys
        for i in range(1, self.sheet_tax_codes.nrows):
            row = (c.value for c in self.sheet_tax_codes.row(i))
            d = OrderedDict(zip(keys, row))
            d['sign'] = int(d['sign'])
            d['sequence'] = int(d['sequence'])
            yield d

    def iter_taxes(self):
        keys = [c.value for c in self.sheet_taxes.row(0)]
        yield keys
        for i in range(1, self.sheet_taxes.nrows):
            row = (c.value for c in self.sheet_taxes.row(i))
            yield OrderedDict(zip(keys, row))

    def iter_fiscal_pos_map(self):
        keys = [c.value for c in self.sheet_fiscal_pos_map.row(0)]
        yield keys
        for i in range(1, self.sheet_fiscal_pos_map.nrows):
            row = (c.value for c in self.sheet_fiscal_pos_map.row(i))
            yield OrderedDict(zip(keys, row))

    def tax_codes_to_csv(self):
        writer = pycompat.csv_writer(open('account.tax.code.template-%s.csv' %
                                 self.suffix, 'wb'))
        tax_codes_iterator = self.iter_tax_codes()
        keys = next(tax_codes_iterator)
        writer.writerow(keys)

        # write structure tax codes
        tax_codes = {}  # code: id
        for row in tax_codes_iterator:
            tax_code = row['code']
            if tax_code in tax_codes:
                raise RuntimeError('duplicate tax code %s' % tax_code)
            tax_codes[tax_code] = row['id']
            writer.writerow([pycompat.to_text(v) for v in row.values()])

        # read taxes and add leaf tax codes
        new_tax_codes = {}  # id: parent_code

        def add_new_tax_code(tax_code_id, new_name, new_parent_code):
            if not tax_code_id:
                return
            name, parent_code = new_tax_codes.get(tax_code_id, (None, None))
            if parent_code and parent_code != new_parent_code:
                raise RuntimeError('tax code "%s" already exist with '
                                   'parent %s while trying to add it with '
                                   'parent %s' %
                                   (tax_code_id, parent_code, new_parent_code))
            else:
                new_tax_codes[tax_code_id] = (new_name, new_parent_code)

        taxes_iterator = self.iter_taxes()
        next(taxes_iterator)
        for row in taxes_iterator:
            if not _is_true(row['active']):
                continue
            if row['child_depend'] and row['amount'] != 1:
                raise RuntimeError('amount must be one if child_depend '
                                   'for %s' % row['id'])
            # base parent
            base_code = row['BASE_CODE']
            if not base_code or base_code == '/':
                base_code = 'NA'
            if base_code not in tax_codes:
                raise RuntimeError('undefined tax code %s' % base_code)
            if base_code != 'NA':
                if row['child_depend']:
                    raise RuntimeError('base code specified '
                                       'with child_depend for %s' % row['id'])
            if not row['child_depend']:
                # ... in lux, we have the same code for invoice and refund
                if base_code != 'NA':
                    assert row['base_code_id:id'], 'missing base_code_id for %s' % row['id']
                assert row['ref_base_code_id:id'] == row['base_code_id:id']
                add_new_tax_code(row['base_code_id:id'],
                                 'Base - ' + row['name'],
                                 base_code)
            # tax parent
            tax_code = row['TAX_CODE']
            if not tax_code or tax_code == '/':
                tax_code = 'NA'
            if tax_code not in tax_codes:
                raise RuntimeError('undefined tax code %s' % tax_code)
            if tax_code == 'NA':
                if row['amount'] and not row['child_depend']:
                    raise RuntimeError('TAX_CODE not specified '
                                       'for non-zero tax %s' % row['id'])
                if row['tax_code_id:id']:
                    raise RuntimeError('tax_code_id specified '
                                       'for tax %s' % row['id'])
            else:
                if row['child_depend']:
                    raise RuntimeError('TAX_CODE specified '
                                       'with child_depend for %s' % row['id'])
                if not row['amount']:
                    raise RuntimeError('TAX_CODE specified '
                                       'for zero tax %s' % row['id'])
                if not row['tax_code_id:id']:
                    raise RuntimeError('tax_code_id not specified '
                                       'for tax %s' % row['id'])
            if not row['child_depend'] and row['amount']:
                # ... in lux, we have the same code for invoice and refund
                assert row['tax_code_id:id'], 'missing tax_code_id for %s' % row['id']
                assert row['ref_tax_code_id:id'] == row['tax_code_id:id']
                add_new_tax_code(row['tax_code_id:id'],
                                 'Taxe - ' + row['name'],
                                 tax_code)

        for tax_code_id in sorted(new_tax_codes):
            name, parent_code = new_tax_codes[tax_code_id]
            writer.writerow([
                tax_code_id,
                u'lu_tct_m' + parent_code,
                tax_code_id.replace('lu_tax_code_template_', u''),
                u'1',
                u'',
                pycompat.to_text(name),
                u''
            ])

    def taxes_to_csv(self):
        writer = pycompat.csv_writer(open('account.tax.template-%s.csv' %
                                     self.suffix, 'wb'))
        taxes_iterator = self.iter_taxes()
        keys = next(taxes_iterator)
        writer.writerow(keys[3:] + ['sequence'])
        seq = 100
        for row in sorted(taxes_iterator, key=lambda r: r['description']):
            if not _is_true(row['active']):
                continue
            seq += 1
            if row['parent_id:id']:
                cur_seq = seq + 1000
            else:
                cur_seq = seq
            writer.writerow([
                pycompat.to_text(v)
                for v in list(row.values())[3:]
            ] + [cur_seq])

    def fiscal_pos_map_to_csv(self):
        writer = pycompat.csv_writer(open('account.fiscal.'
                                     'position.tax.template-%s.csv' %
                                     self.suffix, 'wb'))
        fiscal_pos_map_iterator = self.iter_fiscal_pos_map()
        keys = next(fiscal_pos_map_iterator)
        writer.writerow(keys)
        for row in fiscal_pos_map_iterator:
            writer.writerow([pycompat.to_text(s) for s in row.values()])


if __name__ == '__main__':
    o = LuxTaxGenerator('tax.xls')
    o.tax_codes_to_csv()
    o.taxes_to_csv()
    o.fiscal_pos_map_to_csv()

```

