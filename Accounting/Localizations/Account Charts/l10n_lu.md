# Odoo Module: l10n_lu

Category: Accounting/Localizations/Account Charts

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
    'version': '2.1',
    'category': 'Accounting/Localizations/Account Charts',
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
        'data/account.group.template.csv',
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
    'demo': [
        'demo/demo_company.xml',
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
id,property_account_receivable_id/id,property_account_payable_id/id,property_account_expense_categ_id/id,property_account_income_categ_id/id,income_currency_exchange_account_id/id,expense_currency_exchange_account_id/id,default_pos_receivable_account_id/id,property_stock_account_input_categ_id/id,property_stock_account_output_categ_id/id,property_stock_valuation_account_id/id,property_tax_payable_account_id:id,property_tax_receivable_account_id:id,property_advance_tax_payment_account_id:id,account_journal_suspense_account_id:id
lu_2011_chart_1,lu_2011_account_4011,lu_2011_account_44111,lu_2011_account_6061,lu_2020_account_703001,lu_2020_account_7561,lu_2020_account_6561,lu_2011_account_40111,lu_2011_account_321,lu_2011_account_321,lu_2020_account_60761,lu_2011_account_461412,lu_2011_account_421612,lu_2011_account_421613,lu_2011_account_485

```

## File: data\account.fiscal.position.tax.template-2015.csv

```csv
id,position_id:id,tax_src_id:id,tax_dest_id:id
fiscal_position_tax_template_LU_85,account_fiscal_position_template_LU_IC,lu_2015_tax_VB-PA-17,lu_2011_tax_VB-IC-0
fiscal_position_tax_template_LU_169,account_fiscal_position_template_LU_IC,lu_2015_tax_VB-PA-16,lu_2011_tax_VB-IC-0
fiscal_position_tax_template_LU_86,account_fiscal_position_template_LU_IC,lu_2015_tax_VB-PA-14,lu_2011_tax_VB-IC-0
fiscal_position_tax_template_LU_170,account_fiscal_position_template_LU_IC,lu_2015_tax_VB-PA-13,lu_2011_tax_VB-IC-0
fiscal_position_tax_template_LU_87,account_fiscal_position_template_LU_IC,lu_2015_tax_VB-PA-8,lu_2011_tax_VB-IC-0
fiscal_position_tax_template_LU_171,account_fiscal_position_template_LU_IC,lu_2015_tax_VB-PA-7,lu_2011_tax_VB-IC-0
fiscal_position_tax_template_LU_4,account_fiscal_position_template_LU_IC,lu_2011_tax_VB-PA-3,lu_2011_tax_VB-IC-0
fiscal_position_tax_template_LU_5,account_fiscal_position_template_LU_IC,lu_2011_tax_VB-PA-0,lu_2011_tax_VB-IC-0
fiscal_position_tax_template_LU_88,account_fiscal_position_template_LU_IC,lu_2015_tax_VP-PA-17,lu_2011_tax_VP-IC-0
fiscal_position_tax_template_LU_172,account_fiscal_position_template_LU_IC,lu_2015_tax_VP-PA-16,lu_2011_tax_VP-IC-0
fiscal_position_tax_template_LU_89,account_fiscal_position_template_LU_IC,lu_2015_tax_VP-PA-14,lu_2011_tax_VP-IC-0
fiscal_position_tax_template_LU_173,account_fiscal_position_template_LU_IC,lu_2015_tax_VP-PA-13,lu_2011_tax_VP-IC-0
fiscal_position_tax_template_LU_90,account_fiscal_position_template_LU_IC,lu_2015_tax_VP-PA-8,lu_2011_tax_VP-IC-0
fiscal_position_tax_template_LU_174,account_fiscal_position_template_LU_IC,lu_2015_tax_VP-PA-7,lu_2011_tax_VP-IC-0
fiscal_position_tax_template_LU_9,account_fiscal_position_template_LU_IC,lu_2011_tax_VP-PA-3,lu_2011_tax_VP-IC-0
fiscal_position_tax_template_LU_91,account_fiscal_position_template_LU_IC,lu_2015_tax_FB-PA-17,lu_2015_tax_FB-IC-17
fiscal_position_tax_template_LU_178,account_fiscal_position_template_LU_IC,lu_2015_tax_FB-PA-16,lu_2015_tax_FB-IC-16
fiscal_position_tax_template_LU_92,account_fiscal_position_template_LU_IC,lu_2015_tax_FB-PA-14,lu_2015_tax_FB-IC-14
fiscal_position_tax_template_LU_179,account_fiscal_position_template_LU_IC,lu_2015_tax_FB-PA-13,lu_2015_tax_FB-IC-13
fiscal_position_tax_template_LU_93,account_fiscal_position_template_LU_IC,lu_2015_tax_FB-PA-8,lu_2015_tax_FB-IC-8
fiscal_position_tax_template_LU_180,account_fiscal_position_template_LU_IC,lu_2015_tax_FB-PA-7,lu_2015_tax_FB-IC-7
fiscal_position_tax_template_LU_14,account_fiscal_position_template_LU_IC,lu_2011_tax_FB-PA-3,lu_2011_tax_FB-IC-3
fiscal_position_tax_template_LU_15,account_fiscal_position_template_LU_IC,lu_2011_tax_FB-PA-0,lu_2011_tax_FB-IC-0
fiscal_position_tax_template_LU_94,account_fiscal_position_template_LU_IC,lu_2015_tax_FP-PA-17,lu_2015_tax_FP-IC-17
fiscal_position_tax_template_LU_151,account_fiscal_position_template_LU_IC,lu_2015_tax_FP-PA-16,lu_2015_tax_FP-IC-16
fiscal_position_tax_template_LU_95,account_fiscal_position_template_LU_IC,lu_2015_tax_FP-PA-14,lu_2015_tax_FP-IC-14
fiscal_position_tax_template_LU_152,account_fiscal_position_template_LU_IC,lu_2015_tax_FP-PA-13,lu_2015_tax_FP-IC-13
fiscal_position_tax_template_LU_96,account_fiscal_position_template_LU_IC,lu_2015_tax_FP-PA-8,lu_2015_tax_FP-IC-8
fiscal_position_tax_template_LU_153,account_fiscal_position_template_LU_IC,lu_2015_tax_FP-PA-7,lu_2015_tax_FP-IC-7
fiscal_position_tax_template_LU_19,account_fiscal_position_template_LU_IC,lu_2011_tax_FP-PA-3,lu_2011_tax_FP-IC-3
fiscal_position_tax_template_LU_20,account_fiscal_position_template_LU_IC,lu_2011_tax_FP-PA-0,lu_2011_tax_FP-IC-0
fiscal_position_tax_template_LU_97,account_fiscal_position_template_LU_IC,lu_2015_tax_IB-PA-17,lu_2015_tax_IB-IC-17
fiscal_position_tax_template_LU_154,account_fiscal_position_template_LU_IC,lu_2015_tax_IB-PA-16,lu_2015_tax_IB-IC-16
fiscal_position_tax_template_LU_98,account_fiscal_position_template_LU_IC,lu_2015_tax_IB-PA-14,lu_2015_tax_IB-IC-14
fiscal_position_tax_template_LU_155,account_fiscal_position_template_LU_IC,lu_2015_tax_IB-PA-13,lu_2015_tax_IB-IC-13
fiscal_position_tax_template_LU_99,account_fiscal_position_template_LU_IC,lu_2015_tax_IB-PA-8,lu_2015_tax_IB-IC-8
fiscal_position_tax_template_LU_156,account_fiscal_position_template_LU_IC,lu_2015_tax_IB-PA-7,lu_2015_tax_IB-IC-7
fiscal_position_tax_template_LU_24,account_fiscal_position_template_LU_IC,lu_2011_tax_IB-PA-3,lu_2011_tax_IB-IC-3
fiscal_position_tax_template_LU_25,account_fiscal_position_template_LU_IC,lu_2011_tax_IB-PA-0,lu_2011_tax_IB-IC-0
fiscal_position_tax_template_LU_100,account_fiscal_position_template_LU_IC,lu_2015_tax_IP-PA-17,lu_2015_tax_IP-IC-17
fiscal_position_tax_template_LU_163,account_fiscal_position_template_LU_IC,lu_2015_tax_IP-PA-16,lu_2015_tax_IP-IC-16
fiscal_position_tax_template_LU_101,account_fiscal_position_template_LU_IC,lu_2015_tax_IP-PA-14,lu_2015_tax_IP-IC-14
fiscal_position_tax_template_LU_164,account_fiscal_position_template_LU_IC,lu_2015_tax_IP-PA-13,lu_2015_tax_IP-IC-13
fiscal_position_tax_template_LU_102,account_fiscal_position_template_LU_IC,lu_2015_tax_IP-PA-8,lu_2015_tax_IP-IC-8
fiscal_position_tax_template_LU_165,account_fiscal_position_template_LU_IC,lu_2015_tax_IP-PA-7,lu_2015_tax_IP-IC-7
fiscal_position_tax_template_LU_29,account_fiscal_position_template_LU_IC,lu_2011_tax_IP-PA-3,lu_2011_tax_IP-IC-3
fiscal_position_tax_template_LU_30,account_fiscal_position_template_LU_IC,lu_2011_tax_IP-PA-0,lu_2011_tax_IP-IC-0
fiscal_position_tax_template_LU_103,account_fiscal_position_template_LU_IC,lu_2015_tax_AB-PA-17,lu_2015_tax_AB-IC-17
fiscal_position_tax_template_LU_133,account_fiscal_position_template_LU_IC,lu_2015_tax_AB-PA-16,lu_2015_tax_AB-IC-16
fiscal_position_tax_template_LU_104,account_fiscal_position_template_LU_IC,lu_2015_tax_AB-PA-14,lu_2015_tax_AB-IC-14
fiscal_position_tax_template_LU_134,account_fiscal_position_template_LU_IC,lu_2015_tax_AB-PA-13,lu_2015_tax_AB-IC-13
fiscal_position_tax_template_LU_105,account_fiscal_position_template_LU_IC,lu_2015_tax_AB-PA-8,lu_2015_tax_AB-IC-8
fiscal_position_tax_template_LU_135,account_fiscal_position_template_LU_IC,lu_2015_tax_AB-PA-7,lu_2015_tax_AB-IC-7
fiscal_position_tax_template_LU_35,account_fiscal_position_template_LU_IC,lu_2011_tax_AB-PA-3,lu_2011_tax_AB-IC-3
fiscal_position_tax_template_LU_36,account_fiscal_position_template_LU_IC,lu_2011_tax_AB-PA-0,lu_2011_tax_AB-IC-0
fiscal_position_tax_template_LU_106,account_fiscal_position_template_LU_IC,lu_2015_tax_AP-PA-17,lu_2015_tax_AP-IC-17
fiscal_position_tax_template_LU_142,account_fiscal_position_template_LU_IC,lu_2015_tax_AP-PA-16,lu_2015_tax_AP-IC-16
fiscal_position_tax_template_LU_107,account_fiscal_position_template_LU_IC,lu_2015_tax_AP-PA-14,lu_2015_tax_AP-IC-14
fiscal_position_tax_template_LU_143,account_fiscal_position_template_LU_IC,lu_2015_tax_AP-PA-13,lu_2015_tax_AP-IC-13
fiscal_position_tax_template_LU_108,account_fiscal_position_template_LU_IC,lu_2015_tax_AP-PA-8,lu_2015_tax_AP-IC-8
fiscal_position_tax_template_LU_144,account_fiscal_position_template_LU_IC,lu_2015_tax_AP-PA-7,lu_2015_tax_AP-IC-7
fiscal_position_tax_template_LU_41,account_fiscal_position_template_LU_IC,lu_2011_tax_AP-PA-3,lu_2011_tax_AP-IC-3
fiscal_position_tax_template_LU_42,account_fiscal_position_template_LU_IC,lu_2011_tax_AP-PA-0,lu_2011_tax_AP-IC-0
fiscal_position_tax_template_LU_109,account_fiscal_position_template_LU_EC,lu_2015_tax_VB-PA-17,lu_2011_tax_VB-EC-0
fiscal_position_tax_template_LU_166,account_fiscal_position_template_LU_EC,lu_2015_tax_VB-PA-16,lu_2011_tax_VB-EC-0
fiscal_position_tax_template_LU_110,account_fiscal_position_template_LU_EC,lu_2015_tax_VB-PA-14,lu_2011_tax_VB-EC-0
fiscal_position_tax_template_LU_167,account_fiscal_position_template_LU_EC,lu_2015_tax_VB-PA-13,lu_2011_tax_VB-EC-0
fiscal_position_tax_template_LU_111,account_fiscal_position_template_LU_EC,lu_2015_tax_VB-PA-8,lu_2011_tax_VB-EC-0
fiscal_position_tax_template_LU_168,account_fiscal_position_template_LU_EC,lu_2015_tax_VB-PA-7,lu_2011_tax_VB-EC-0
fiscal_position_tax_template_LU_46,account_fiscal_position_template_LU_EC,lu_2011_tax_VB-PA-3,lu_2011_tax_VB-EC-0
fiscal_position_tax_template_LU_47,account_fiscal_position_template_LU_EC,lu_2011_tax_VB-PA-0,lu_2011_tax_VB-EC-0
fiscal_position_tax_template_LU_112,account_fiscal_position_template_LU_EC,lu_2015_tax_VP-PA-17,lu_2011_tax_VP-EC-0
fiscal_position_tax_template_LU_175,account_fiscal_position_template_LU_EC,lu_2015_tax_VP-PA-16,lu_2011_tax_VP-EC-0
fiscal_position_tax_template_LU_113,account_fiscal_position_template_LU_EC,lu_2015_tax_VP-PA-14,lu_2011_tax_VP-EC-0
fiscal_position_tax_template_LU_176,account_fiscal_position_template_LU_EC,lu_2015_tax_VP-PA-13,lu_2011_tax_VP-EC-0
fiscal_position_tax_template_LU_114,account_fiscal_position_template_LU_EC,lu_2015_tax_VP-PA-8,lu_2011_tax_VP-EC-0
fiscal_position_tax_template_LU_177,account_fiscal_position_template_LU_EC,lu_2015_tax_VP-PA-7,lu_2011_tax_VP-EC-0
fiscal_position_tax_template_LU_51,account_fiscal_position_template_LU_EC,lu_2011_tax_VP-PA-3,lu_2011_tax_VP-EC-0
fiscal_position_tax_template_LU_52,account_fiscal_position_template_LU_EC,lu_2011_tax_VP-PA-0,lu_2011_tax_VP-EC-0
fiscal_position_tax_template_LU_115,account_fiscal_position_template_LU_EC,lu_2015_tax_FB-PA-17,lu_2015_tax_FB-EC-17
fiscal_position_tax_template_LU_145,account_fiscal_position_template_LU_EC,lu_2015_tax_FB-PA-16,lu_2015_tax_FB-EC-16
fiscal_position_tax_template_LU_116,account_fiscal_position_template_LU_EC,lu_2015_tax_FB-PA-14,lu_2015_tax_FB-EC-14
fiscal_position_tax_template_LU_146,account_fiscal_position_template_LU_EC,lu_2015_tax_FB-PA-13,lu_2015_tax_FB-EC-13
fiscal_position_tax_template_LU_117,account_fiscal_position_template_LU_EC,lu_2015_tax_FB-PA-8,lu_2015_tax_FB-EC-8
fiscal_position_tax_template_LU_147,account_fiscal_position_template_LU_EC,lu_2015_tax_FB-PA-7,lu_2015_tax_FB-EC-7
fiscal_position_tax_template_LU_56,account_fiscal_position_template_LU_EC,lu_2011_tax_FB-PA-3,lu_2011_tax_FB-EC-3
fiscal_position_tax_template_LU_57,account_fiscal_position_template_LU_EC,lu_2011_tax_FB-PA-0,lu_2011_tax_FB-EC-0
fiscal_position_tax_template_LU_118,account_fiscal_position_template_LU_EC,lu_2015_tax_FP-PA-17,lu_2015_tax_FP-EC-17
fiscal_position_tax_template_LU_148,account_fiscal_position_template_LU_EC,lu_2015_tax_FP-PA-16,lu_2015_tax_FP-EC-16
fiscal_position_tax_template_LU_119,account_fiscal_position_template_LU_EC,lu_2015_tax_FP-PA-14,lu_2015_tax_FP-EC-14
fiscal_position_tax_template_LU_149,account_fiscal_position_template_LU_EC,lu_2015_tax_FP-PA-13,lu_2015_tax_FP-EC-13
fiscal_position_tax_template_LU_120,account_fiscal_position_template_LU_EC,lu_2015_tax_FP-PA-8,lu_2015_tax_FP-EC-8
fiscal_position_tax_template_LU_150,account_fiscal_position_template_LU_EC,lu_2015_tax_FP-PA-7,lu_2015_tax_FP-EC-7
fiscal_position_tax_template_LU_61,account_fiscal_position_template_LU_EC,lu_2011_tax_FP-PA-3,lu_2011_tax_FP-EC-3
fiscal_position_tax_template_LU_62,account_fiscal_position_template_LU_EC,lu_2011_tax_FP-PA-0,lu_2011_tax_FP-EC-0
fiscal_position_tax_template_LU_121,account_fiscal_position_template_LU_EC,lu_2015_tax_IB-PA-17,lu_2015_tax_IB-EC-17
fiscal_position_tax_template_LU_157,account_fiscal_position_template_LU_EC,lu_2015_tax_IB-PA-16,lu_2015_tax_IB-EC-16
fiscal_position_tax_template_LU_122,account_fiscal_position_template_LU_EC,lu_2015_tax_IB-PA-14,lu_2015_tax_IB-EC-14
fiscal_position_tax_template_LU_158,account_fiscal_position_template_LU_EC,lu_2015_tax_IB-PA-13,lu_2015_tax_IB-EC-13
fiscal_position_tax_template_LU_123,account_fiscal_position_template_LU_EC,lu_2015_tax_IB-PA-8,lu_2015_tax_IB-EC-8
fiscal_position_tax_template_LU_159,account_fiscal_position_template_LU_EC,lu_2015_tax_IB-PA-7,lu_2015_tax_IB-EC-7
fiscal_position_tax_template_LU_66,account_fiscal_position_template_LU_EC,lu_2011_tax_IB-PA-3,lu_2011_tax_IB-EC-3
fiscal_position_tax_template_LU_67,account_fiscal_position_template_LU_EC,lu_2011_tax_IB-PA-0,lu_2011_tax_IB-EC-0
fiscal_position_tax_template_LU_124,account_fiscal_position_template_LU_EC,lu_2015_tax_IP-PA-17,lu_2015_tax_IP-EC-17
fiscal_position_tax_template_LU_160,account_fiscal_position_template_LU_EC,lu_2015_tax_IP-PA-16,lu_2015_tax_IP-EC-16
fiscal_position_tax_template_LU_125,account_fiscal_position_template_LU_EC,lu_2015_tax_IP-PA-14,lu_2015_tax_IP-EC-14
fiscal_position_tax_template_LU_161,account_fiscal_position_template_LU_EC,lu_2015_tax_IP-PA-13,lu_2015_tax_IP-EC-13
fiscal_position_tax_template_LU_126,account_fiscal_position_template_LU_EC,lu_2015_tax_IP-PA-8,lu_2015_tax_IP-EC-8
fiscal_position_tax_template_LU_162,account_fiscal_position_template_LU_EC,lu_2015_tax_IP-PA-7,lu_2015_tax_IP-EC-7
fiscal_position_tax_template_LU_71,account_fiscal_position_template_LU_EC,lu_2011_tax_IP-PA-3,lu_2011_tax_IP-EC-3
fiscal_position_tax_template_LU_72,account_fiscal_position_template_LU_EC,lu_2011_tax_IP-PA-0,lu_2011_tax_IP-EC-0
fiscal_position_tax_template_LU_127,account_fiscal_position_template_LU_EC,lu_2015_tax_AB-PA-17,lu_2015_tax_AB-EC-17
fiscal_position_tax_template_LU_136,account_fiscal_position_template_LU_EC,lu_2015_tax_AB-PA-16,lu_2015_tax_AB-EC-16
fiscal_position_tax_template_LU_128,account_fiscal_position_template_LU_EC,lu_2015_tax_AB-PA-14,lu_2015_tax_AB-EC-14
fiscal_position_tax_template_LU_137,account_fiscal_position_template_LU_EC,lu_2015_tax_AB-PA-13,lu_2015_tax_AB-EC-13
fiscal_position_tax_template_LU_129,account_fiscal_position_template_LU_EC,lu_2015_tax_AB-PA-8,lu_2015_tax_AB-EC-8
fiscal_position_tax_template_LU_138,account_fiscal_position_template_LU_EC,lu_2015_tax_AB-PA-7,lu_2015_tax_AB-EC-7
fiscal_position_tax_template_LU_77,account_fiscal_position_template_LU_EC,lu_2011_tax_AB-PA-3,lu_2011_tax_AB-EC-3
fiscal_position_tax_template_LU_78,account_fiscal_position_template_LU_EC,lu_2011_tax_AB-PA-0,lu_2011_tax_AB-EC-0
fiscal_position_tax_template_LU_130,account_fiscal_position_template_LU_EC,lu_2015_tax_AP-PA-17,lu_2015_tax_AP-EC-17
fiscal_position_tax_template_LU_139,account_fiscal_position_template_LU_EC,lu_2015_tax_AP-PA-16,lu_2015_tax_AP-EC-16
fiscal_position_tax_template_LU_131,account_fiscal_position_template_LU_EC,lu_2015_tax_AP-PA-14,lu_2015_tax_AP-EC-14
fiscal_position_tax_template_LU_140,account_fiscal_position_template_LU_EC,lu_2015_tax_AP-PA-13,lu_2015_tax_AP-EC-13
fiscal_position_tax_template_LU_132,account_fiscal_position_template_LU_EC,lu_2015_tax_AP-PA-8,lu_2015_tax_AP-EC-8
fiscal_position_tax_template_LU_141,account_fiscal_position_template_LU_EC,lu_2015_tax_AP-PA-7,lu_2015_tax_AP-EC-7
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

## File: data\account.group.template.csv

```csv
id,code_prefix_start,code_prefix_end,name,chart_template_id/id
account_group_1,1,,"EQUITY, PROVISIONS AND FINANCIAL LIABILITIES ACCOUNTS",l10n_lu.lu_2011_chart_1
account_group_10,10,,"Subscribed capital or branches' assigned capital and owner's account",l10n_lu.lu_2011_chart_1
account_group_101,101,,"Subscribed capital",l10n_lu.lu_2011_chart_1
account_group_102,102,,"Subscribed capital not called",l10n_lu.lu_2011_chart_1
account_group_103,103,,"Subscribed capital called but unpaid",l10n_lu.lu_2011_chart_1
account_group_104,104,,"Capital of individual companies, corporate partnerships and similar",l10n_lu.lu_2011_chart_1
account_group_105,105,,"Endowment of branches",l10n_lu.lu_2011_chart_1
account_group_106,106,,"Account of the owner or the co-owners",l10n_lu.lu_2011_chart_1
account_group_11,11,,"Share premium and similar premiums",l10n_lu.lu_2011_chart_1
account_group_111,111,,"Share premium",l10n_lu.lu_2011_chart_1
account_group_112,112,,"Merger premium",l10n_lu.lu_2011_chart_1
account_group_113,113,,"Contribution premium",l10n_lu.lu_2011_chart_1
account_group_114,114,,"Premiums on conversion of bonds into shares",l10n_lu.lu_2011_chart_1
account_group_115,115,,"Capital contribution without issue of shares",l10n_lu.lu_2011_chart_1
account_group_12,12,,"Revaluation reserves",l10n_lu.lu_2011_chart_1
account_group_122,122,,"Reserves in application of the equity method",l10n_lu.lu_2011_chart_1
account_group_123,123,,"Temporarily not taxable currency translation adjustments",l10n_lu.lu_2011_chart_1
account_group_128,128,,"Other revaluation reserves",l10n_lu.lu_2011_chart_1
account_group_13,13,,"Reserves",l10n_lu.lu_2011_chart_1
account_group_131,131,,"Legal reserve",l10n_lu.lu_2011_chart_1
account_group_132,132,,"Reserves for own shares or own corporate units",l10n_lu.lu_2011_chart_1
account_group_133,133,,"Reserves provided for by the articles of association",l10n_lu.lu_2011_chart_1
account_group_138,138,,"Other reserves, including fair-value reserve",l10n_lu.lu_2011_chart_1
account_group_1381,1381,,"Other reserves available for distribution",l10n_lu.lu_2011_chart_1
account_group_1382,1382,,"Other reserves not available for distribution",l10n_lu.lu_2011_chart_1
account_group_13821,13821,,"Reserve for net wealth tax (NWT)",l10n_lu.lu_2011_chart_1
account_group_13822,13822,,"Reserves in application of fair value",l10n_lu.lu_2011_chart_1
account_group_13823,13823,,"Temporarily not taxable capital gains",l10n_lu.lu_2011_chart_1
account_group_138231,138231,,"Temporarily not taxable capital gains to reinvest",l10n_lu.lu_2011_chart_1
account_group_138232,138232,,"Temporarily not taxable capital gains reinvested",l10n_lu.lu_2011_chart_1
account_group_13828,13828,,"Reserves not available for distribution not mentioned above",l10n_lu.lu_2011_chart_1
account_group_14,14,,"Result for the financial year and results brought forward",l10n_lu.lu_2011_chart_1
account_group_141,141,,"Results brought forward",l10n_lu.lu_2011_chart_1
account_group_1411,1411,,"Results brought forward in the process of assignment",l10n_lu.lu_2011_chart_1
account_group_1412,1412,,"Results brought forward (assigned)",l10n_lu.lu_2011_chart_1
account_group_142,142,,"Result for the financial year",l10n_lu.lu_2011_chart_1
account_group_15,15,,"Interim dividends",l10n_lu.lu_2011_chart_1
account_group_16,16,,"Capital investment subsidies",l10n_lu.lu_2011_chart_1
account_group_161,161,,"Subsidies on intangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_1611,1611,,"Development costs",l10n_lu.lu_2011_chart_1
account_group_1612,1612,,"Concessions, patents, licences, trademarks and similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_16121,16121,,"acquired against payment (except Goodwill)",l10n_lu.lu_2011_chart_1
account_group_16122,16122,,"created by the undertaking itself",l10n_lu.lu_2011_chart_1
account_group_1613,1613,,"Goodwill acquired for consideration",l10n_lu.lu_2011_chart_1
account_group_162,162,,"Subsidies on tangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_1621,1621,,"Subsidies on land, fitting-outs and buildings",l10n_lu.lu_2011_chart_1
account_group_1622,1622,,"Subsidies on plant and machinery",l10n_lu.lu_2011_chart_1
account_group_1623,1623,,"Subsidies on other fixtures, fittings, tools and equipment (including rolling stock)",l10n_lu.lu_2011_chart_1
account_group_168,168,,"Other capital investment subsidies",l10n_lu.lu_2011_chart_1
account_group_18,18,,"Provisions",l10n_lu.lu_2011_chart_1
account_group_181,181,,"Provisions for pensions and similar obligations",l10n_lu.lu_2011_chart_1
account_group_182,182,,"Provisions for taxation",l10n_lu.lu_2011_chart_1
account_group_183,183,,"Deferred tax provisions",l10n_lu.lu_2011_chart_1
account_group_188,188,,"Other provisions",l10n_lu.lu_2011_chart_1
account_group_1881,1881,,"Operating provisions",l10n_lu.lu_2011_chart_1
account_group_1882,1882,,"Financial provisions",l10n_lu.lu_2011_chart_1
account_group_19,19,,"Debenture loans and amounts owed to credit institutions",l10n_lu.lu_2011_chart_1
account_group_192,192,,"Convertible debenture loans",l10n_lu.lu_2011_chart_1
account_group_1921,1921,,"due and payable within one year",l10n_lu.lu_2011_chart_1
account_group_1922,1922,,"due and payable after more than one year",l10n_lu.lu_2011_chart_1
account_group_193,193,,"Non-convertible debenture loans",l10n_lu.lu_2011_chart_1
account_group_1931,1931,,"due and payable within one year",l10n_lu.lu_2011_chart_1
account_group_1932,1932,,"due and payable after more than one year",l10n_lu.lu_2011_chart_1
account_group_194,194,,"Amounts owed to credit institutions",l10n_lu.lu_2011_chart_1
account_group_1941,1941,,"due and payable within one year",l10n_lu.lu_2011_chart_1
account_group_1942,1942,,"due and payable after more than one year",l10n_lu.lu_2011_chart_1
account_group_2,2,,"FORMATION EXPENSES AND FIXED ASSETS ACCOUNTS",l10n_lu.lu_2011_chart_1
account_group_20,20,,"Formation expenses and similar expenses",l10n_lu.lu_2011_chart_1
account_group_201,201,,"Set-up and start-up costs",l10n_lu.lu_2011_chart_1
account_group_203,203,,"Expenses for increases in capital and for various operations (merger, demerger, change of legal form)",l10n_lu.lu_2011_chart_1
account_group_204,204,,"Loan issuances expenses",l10n_lu.lu_2011_chart_1
account_group_208,208,,"Other similar expenses",l10n_lu.lu_2011_chart_1
account_group_21,21,,"Intangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_211,211,,"Development costs",l10n_lu.lu_2011_chart_1
account_group_212,212,,"Concessions, patents, licences, trademarks and similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_2121,2121,,"acquired for consideration (except Goodwill)",l10n_lu.lu_2011_chart_1
account_group_21211,21211,,"Concessions",l10n_lu.lu_2011_chart_1
account_group_21212,21212,,"Patents",l10n_lu.lu_2011_chart_1
account_group_21213,21213,,"Software licences",l10n_lu.lu_2011_chart_1
account_group_21214,21214,,"Trademarks and franchises",l10n_lu.lu_2011_chart_1
account_group_21215,21215,,"Similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_212151,212151,,"Copyrights and reproduction rights",l10n_lu.lu_2011_chart_1
account_group_212152,212152,,"Greenhous gas and similar emission quotas",l10n_lu.lu_2011_chart_1
account_group_212158,212158,,"Other similar rights and assets acquired for consideration",l10n_lu.lu_2011_chart_1
account_group_2122,2122,,"created by the undertaking itself",l10n_lu.lu_2011_chart_1
account_group_21221,21221,,"Concessions",l10n_lu.lu_2011_chart_1
account_group_21222,21222,,"Patents",l10n_lu.lu_2011_chart_1
account_group_21223,21223,,"Software licences",l10n_lu.lu_2011_chart_1
account_group_21224,21224,,"Trademarks and franchises",l10n_lu.lu_2011_chart_1
account_group_21225,21225,,"Similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_212251,212251,,"Copyrights and reproduction rights",l10n_lu.lu_2011_chart_1
account_group_212258,212258,,"Other similar rights and assets created by the undertaking itself",l10n_lu.lu_2011_chart_1
account_group_213,213,,"Goodwill acquired for consideration",l10n_lu.lu_2011_chart_1
account_group_214,214,,"Down payments and intangible fixed assets under development",l10n_lu.lu_2011_chart_1
account_group_22,22,,"Tangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_221,221,,"Land, fixtures and fitting-outs and buildings",l10n_lu.lu_2011_chart_1
account_group_2211,2211,,"Land",l10n_lu.lu_2011_chart_1
account_group_22111,22111,,"Land in Luxembourg",l10n_lu.lu_2011_chart_1
account_group_221111,221111,,"Developed land",l10n_lu.lu_2011_chart_1
account_group_221112,221112,,"Property rights and similar",l10n_lu.lu_2011_chart_1
account_group_221118,221118,,"Other land",l10n_lu.lu_2011_chart_1
account_group_22112,22112,,"Land in foreign countries",l10n_lu.lu_2011_chart_1
account_group_2212,2212,,"Fixtures and fittings-out of land",l10n_lu.lu_2011_chart_1
account_group_22121,22121,,"Fixtures and fitting-outs of land in Luxembourg",l10n_lu.lu_2011_chart_1
account_group_22122,22122,,"Fixtures and fitting-outs of land in foreign countries",l10n_lu.lu_2011_chart_1
account_group_2213,2213,,"Buildings",l10n_lu.lu_2011_chart_1
account_group_22131,22131,,"Buildings in Luxembourg",l10n_lu.lu_2011_chart_1
account_group_221311,221311,,"Residential buildings",l10n_lu.lu_2011_chart_1
account_group_221312,221312,,"Non-residential buildings",l10n_lu.lu_2011_chart_1
account_group_221313,221313,,"Mixed-use buildings",l10n_lu.lu_2011_chart_1
account_group_221318,221318,,"Other buildings",l10n_lu.lu_2011_chart_1
account_group_22132,22132,,"Buildings in foreign countries",l10n_lu.lu_2011_chart_1
account_group_2214,2214,,"Fixtures and fitting-outs of buildings",l10n_lu.lu_2011_chart_1
account_group_22141,22141,,"Fixtures and fitting-outs of buildings in Luxembourg",l10n_lu.lu_2011_chart_1
account_group_22142,22142,,"Fixtures and fitting-outs of buildings in foreign countries",l10n_lu.lu_2011_chart_1
account_group_2215,2215,,"Investment properties",l10n_lu.lu_2011_chart_1
account_group_22151,22151,,"Investment properties in Luxembourg",l10n_lu.lu_2011_chart_1
account_group_22152,22152,,"Investment properties in foreign countries",l10n_lu.lu_2011_chart_1
account_group_222,222,,"Plant and machinery",l10n_lu.lu_2011_chart_1
account_group_2221,2221,,"Plant",l10n_lu.lu_2011_chart_1
account_group_2222,2222,,"Machinery",l10n_lu.lu_2011_chart_1
account_group_223,223,,"Other fixtures and fittings, tools and equipment (including rolling stock)",l10n_lu.lu_2011_chart_1
account_group_2231,2231,,"Transportation and handling equipment",l10n_lu.lu_2011_chart_1
account_group_2232,2232,,"Motor vehicles",l10n_lu.lu_2011_chart_1
account_group_2233,2233,,"Tools",l10n_lu.lu_2011_chart_1
account_group_2234,2234,,"Furniture",l10n_lu.lu_2011_chart_1
account_group_2235,2235,,"Computer equipment",l10n_lu.lu_2011_chart_1
account_group_2236,2236,,"Livestock",l10n_lu.lu_2011_chart_1
account_group_2237,2237,,"Returnable packaging",l10n_lu.lu_2011_chart_1
account_group_2238,2238,,"Other fixtures",l10n_lu.lu_2011_chart_1
account_group_224,224,,"Down payments and tangible fixed assets under development",l10n_lu.lu_2011_chart_1
account_group_2241,2241,,"Land, fitting-outs and buildings",l10n_lu.lu_2011_chart_1
account_group_22411,22411,,"Land, fitting-outs and buildings in Luxembourg",l10n_lu.lu_2011_chart_1
account_group_22412,22412,,"Land, fitting-outs and buildings in foreign countries",l10n_lu.lu_2011_chart_1
account_group_2242,2242,,"Plant and machinery",l10n_lu.lu_2011_chart_1
account_group_2243,2243,,"Other fixtures and fittings, tools and equipment (including rolling stock)",l10n_lu.lu_2011_chart_1
account_group_23,23,,"Financial fixed assets",l10n_lu.lu_2011_chart_1
account_group_231,231,,"Shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_232,232,,"Amounts owed by affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_233,233,,"Participating interests",l10n_lu.lu_2011_chart_1
account_group_234,234,,"Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_235,235,,"Securities held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_2351,2351,,"Securities held as fixed assets (equity right)",l10n_lu.lu_2011_chart_1
account_group_23511,23511,,"Shares or corporate units",l10n_lu.lu_2011_chart_1
account_group_235111,235111,,"Listed shares",l10n_lu.lu_2011_chart_1
account_group_235112,235112,,"Unlisted shares",l10n_lu.lu_2011_chart_1
account_group_23518,23518,,"Other securities held as fixed assets (equity right)",l10n_lu.lu_2011_chart_1
account_group_2352,2352,,"Securities held as fixed assets (creditor's right)",l10n_lu.lu_2011_chart_1
account_group_23521,23521,,"Debentures",l10n_lu.lu_2011_chart_1
account_group_23528,23528,,"Other securities held as fixed assets (creditor's right)",l10n_lu.lu_2011_chart_1
account_group_2353,2353,,"Shares of collective investment funds",l10n_lu.lu_2011_chart_1
account_group_2358,2358,,"Other securities held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_236,236,,"Loans, deposits and claims held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_2361,2361,,"Loans",l10n_lu.lu_2011_chart_1
account_group_2362,2362,,"Deposits and guarantees paid",l10n_lu.lu_2011_chart_1
account_group_2363,2363,,"Long-term receivables",l10n_lu.lu_2011_chart_1
account_group_3,3,,"INVENTORIES ACCOUNTS",l10n_lu.lu_2011_chart_1
account_group_30,30,,"Inventories of raw materials and consumables",l10n_lu.lu_2011_chart_1
account_group_301,301,,"Inventories of raw materials",l10n_lu.lu_2011_chart_1
account_group_303,303,,"Inventories of consumable materials and supplies",l10n_lu.lu_2011_chart_1
account_group_304,304,,"Inventories of packaging",l10n_lu.lu_2011_chart_1
account_group_31,31,,"Inventories of work and contracts in progress",l10n_lu.lu_2011_chart_1
account_group_311,311,,"Inventories of work in progress",l10n_lu.lu_2011_chart_1
account_group_312,312,,"Contracts in progress - goods",l10n_lu.lu_2011_chart_1
account_group_313,313,,"Contracts in progress - services",l10n_lu.lu_2011_chart_1
account_group_314,314,,"Buildings under construction",l10n_lu.lu_2011_chart_1
account_group_315,315,,"Down payments received on inventories of work and on contracts in progress",l10n_lu.lu_2011_chart_1
account_group_32,32,,"Inventories of goods",l10n_lu.lu_2011_chart_1
account_group_321,321,,"Inventories of finished goods",l10n_lu.lu_2011_chart_1
account_group_322,322,,"Inventories of semi-finished goods",l10n_lu.lu_2011_chart_1
account_group_323,323,,"Inventories of residual goods (waste, rejected and recuperable material)",l10n_lu.lu_2011_chart_1
account_group_36,36,,"Inventories of merchandises and other goods for resale",l10n_lu.lu_2011_chart_1
account_group_361,361,,"Inventories of merchandise",l10n_lu.lu_2011_chart_1
account_group_362,362,,"Inventories of land for resale",l10n_lu.lu_2011_chart_1
account_group_3621,3621,,"Inventories of land for resale in Luxembourg",l10n_lu.lu_2011_chart_1
account_group_3622,3622,,"Inventories of land for resale in foreign countries",l10n_lu.lu_2011_chart_1
account_group_363,363,,"Inventories of buildings for resale",l10n_lu.lu_2011_chart_1
account_group_3631,3631,,"Inventories of buildings for resale in Luxembourg",l10n_lu.lu_2011_chart_1
account_group_3632,3632,,"Inventories of buildings for resale in foreign countries",l10n_lu.lu_2011_chart_1
account_group_37,37,,"Down payments on account on inventories",l10n_lu.lu_2011_chart_1
account_group_4,4,,"DEBTORS AND CREDITORS",l10n_lu.lu_2011_chart_1
account_group_40,40,,"Trade receivables (Receivables from sales and rendering of services)",l10n_lu.lu_2011_chart_1
account_group_401,401,,"Trade receivables due and payable within one year",l10n_lu.lu_2011_chart_1
account_group_4011,4011,,"Customers",l10n_lu.lu_2011_chart_1
account_group_4012,4012,,"Customers - Receivable bills of exchange",l10n_lu.lu_2011_chart_1
account_group_4013,4013,,"Doubtful or disputed customers",l10n_lu.lu_2011_chart_1
account_group_4014,4014,,"Customers - Unbilled sales",l10n_lu.lu_2011_chart_1
account_group_4015,4015,,"Customers with a credit balance",l10n_lu.lu_2011_chart_1
account_group_4019,4019,,"Value adjustments",l10n_lu.lu_2011_chart_1
account_group_402,402,,"Trade receivables due and payable after more than one year",l10n_lu.lu_2011_chart_1
account_group_4021,4021,,"Customers",l10n_lu.lu_2011_chart_1
account_group_4025,4025,,"Customers with creditor balance",l10n_lu.lu_2011_chart_1
account_group_4029,4029,,"Value adjustments",l10n_lu.lu_2011_chart_1
account_group_41,41,,"Amounts owed by affiliated undertakings and by undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_411,411,,"Amounts owed by affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_4111,4111,,"Amounts owed by affiliated undertakings receivable within one year",l10n_lu.lu_2011_chart_1
account_group_41111,41111,,"Trade receivables",l10n_lu.lu_2011_chart_1
account_group_41112,41112,,"Loans and advances",l10n_lu.lu_2011_chart_1
account_group_41118,41118,,"Other receivables",l10n_lu.lu_2011_chart_1
account_group_41119,41119,,"Value adjustments",l10n_lu.lu_2011_chart_1
account_group_4112,4112,,"Amounts owed by affiliated undertakings receivable after more than one year",l10n_lu.lu_2011_chart_1
account_group_41121,41121,,"Trade receivables",l10n_lu.lu_2011_chart_1
account_group_41122,41122,,"Loans and advances",l10n_lu.lu_2011_chart_1
account_group_41128,41128,,"Other receivables",l10n_lu.lu_2011_chart_1
account_group_41129,41129,,"Value adjustments",l10n_lu.lu_2011_chart_1
account_group_412,412,,"Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_4121,4121,,"Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests receivable within one year",l10n_lu.lu_2011_chart_1
account_group_41211,41211,,"Trade receivables",l10n_lu.lu_2011_chart_1
account_group_41212,41212,,"Loans and advances",l10n_lu.lu_2011_chart_1
account_group_41218,41218,,"Other receivables",l10n_lu.lu_2011_chart_1
account_group_41219,41219,,"Value adjustments",l10n_lu.lu_2011_chart_1
account_group_4122,4122,,"Amounts receivable after more than one year",l10n_lu.lu_2011_chart_1
account_group_41221,41221,,"Trade receivables",l10n_lu.lu_2011_chart_1
account_group_41222,41222,,"Loans and advances",l10n_lu.lu_2011_chart_1
account_group_41228,41228,,"Other receivables",l10n_lu.lu_2011_chart_1
account_group_41229,41229,,"Value adjustments",l10n_lu.lu_2011_chart_1
account_group_42,42,,"Other receivables",l10n_lu.lu_2011_chart_1
account_group_421,421,,"Other receivables within one year",l10n_lu.lu_2011_chart_1
account_group_4211,4211,,"Staff - Advances and down payments",l10n_lu.lu_2011_chart_1
account_group_42111,42111,,"Advances and down payments",l10n_lu.lu_2011_chart_1
account_group_42119,42119,,"Value adjustments",l10n_lu.lu_2011_chart_1
account_group_4212,4212,,"Amounts owed by partners and shareholders (others than from affiliated undertakings)",l10n_lu.lu_2011_chart_1
account_group_4213,4213,,"State - Subsidies to be received",l10n_lu.lu_2011_chart_1
account_group_42131,42131,,"Investment subsidies",l10n_lu.lu_2011_chart_1
account_group_42132,42132,,"Operating subsidies",l10n_lu.lu_2011_chart_1
account_group_42138,42138,,"Other subsidies",l10n_lu.lu_2011_chart_1
account_group_4214,4214,,"Direct Tax Authority (ACD)",l10n_lu.lu_2011_chart_1
account_group_42141,42141,,"Corporate income tax",l10n_lu.lu_2011_chart_1
account_group_42142,42142,,"Municipal business tax",l10n_lu.lu_2011_chart_1
account_group_42143,42143,,"Net wealth tax",l10n_lu.lu_2011_chart_1
account_group_42144,42144,,"Withholding tax on wages and salaries",l10n_lu.lu_2011_chart_1
account_group_42145,42145,,"Withholding tax on financial investment income",l10n_lu.lu_2011_chart_1
account_group_42146,42146,,"Withholding tax on director's fees",l10n_lu.lu_2011_chart_1
account_group_42148,42148,,"ACD - Other amounts receivable",l10n_lu.lu_2011_chart_1
account_group_4215,4215,,"Customs and Excise Authority (ADA)",l10n_lu.lu_2011_chart_1
account_group_4216,4216,,"Indirect Tax Authority (AED)",l10n_lu.lu_2011_chart_1
account_group_42161,42161,,"Value-added tax (VAT)",l10n_lu.lu_2011_chart_1
account_group_421611,421611,,"VAT paid and recoverable",l10n_lu.lu_2011_chart_1
account_group_421612,421612,,"VAT receivable",l10n_lu.lu_2011_chart_1
account_group_421613,421613,,"VAT down payments made",l10n_lu.lu_2011_chart_1
account_group_421618,421618,,"VAT - Other receivables",l10n_lu.lu_2011_chart_1
account_group_42162,42162,,"Indirect taxes",l10n_lu.lu_2011_chart_1
account_group_421621,421621,,"Registration duties",l10n_lu.lu_2011_chart_1
account_group_421622,421622,,"Subscription tax",l10n_lu.lu_2011_chart_1
account_group_421628,421628,,"Other indirect taxes",l10n_lu.lu_2011_chart_1
account_group_42168,42168,,"Other receivables",l10n_lu.lu_2011_chart_1
account_group_4217,4217,,"Amounts owed by the Social Security and other social bodies",l10n_lu.lu_2011_chart_1
account_group_42171,42171,,"Joint Social Security Centre (CCSS)",l10n_lu.lu_2011_chart_1
account_group_42172,42172,,"Foreign social security offices",l10n_lu.lu_2011_chart_1
account_group_42178,42178,,"Other social bodies",l10n_lu.lu_2011_chart_1
account_group_4218,4218,,"Miscellaneous receivables",l10n_lu.lu_2011_chart_1
account_group_42181,42181,,"Foreign taxes",l10n_lu.lu_2011_chart_1
account_group_421811,421811,,"Foreign VAT",l10n_lu.lu_2011_chart_1
account_group_421818,421818,,"Other foreign taxes",l10n_lu.lu_2011_chart_1
account_group_42187,42187,,"Derivative financial instruments",l10n_lu.lu_2011_chart_1
account_group_42188,42188,,"Other miscellaneous receivables",l10n_lu.lu_2011_chart_1
account_group_42189,42189,,"Value adjustments",l10n_lu.lu_2011_chart_1
account_group_422,422,,"Other receivables after one year",l10n_lu.lu_2011_chart_1
account_group_4221,4221,,"Staff - advances and down payments",l10n_lu.lu_2011_chart_1
account_group_4222,4222,,"Amounts owed by partners and shareholders (others than from affiliated undertakings)",l10n_lu.lu_2011_chart_1
account_group_4223,4223,,"State - Subsidies to be received",l10n_lu.lu_2011_chart_1
account_group_42231,42231,,"Investment subsidies",l10n_lu.lu_2011_chart_1
account_group_42232,42232,,"Operating subsidies",l10n_lu.lu_2011_chart_1
account_group_42238,42238,,"Other subsidies",l10n_lu.lu_2011_chart_1
account_group_4228,4228,,"Miscellaneous receivables",l10n_lu.lu_2011_chart_1
account_group_42287,42287,,"Derivative financial instruments",l10n_lu.lu_2011_chart_1
account_group_42288,42288,,"Other miscellaneous receivables",l10n_lu.lu_2011_chart_1
account_group_42289,42289,,"Value adjustments",l10n_lu.lu_2011_chart_1
account_group_43,43,,"Down payments received on orders as far as they are not deducted distinctly from inventories",l10n_lu.lu_2011_chart_1
account_group_431,431,,"Down payments received within one year",l10n_lu.lu_2011_chart_1
account_group_4311,4311,,"Down payments received on orders",l10n_lu.lu_2011_chart_1
account_group_4312,4312,,"Inventories of work and contracts in progress less down payments received",l10n_lu.lu_2011_chart_1
account_group_432,432,,"Down payments received after more than one year",l10n_lu.lu_2011_chart_1
account_group_4321,4321,,"Down payments received on orders",l10n_lu.lu_2011_chart_1
account_group_4322,4322,,"Inventories of work and contracts in progress less down payments received",l10n_lu.lu_2011_chart_1
account_group_44,44,,"Trade payables and bills of exchange",l10n_lu.lu_2011_chart_1
account_group_441,441,,"Trade payables",l10n_lu.lu_2011_chart_1
account_group_4411,4411,,"Trade payables within one year",l10n_lu.lu_2011_chart_1
account_group_44111,44111,,"Suppliers",l10n_lu.lu_2011_chart_1
account_group_44112,44112,,"Suppliers - invoices not yet received",l10n_lu.lu_2011_chart_1
account_group_44113,44113,,"Suppliers with a debit balance",l10n_lu.lu_2011_chart_1
account_group_4412,4412,,"Trade payables after more than one year",l10n_lu.lu_2011_chart_1
account_group_44121,44121,,"Suppliers",l10n_lu.lu_2011_chart_1
account_group_44123,44123,,"Suppliers with a debit balance",l10n_lu.lu_2011_chart_1
account_group_442,442,,"Bills of exchange payable",l10n_lu.lu_2011_chart_1
account_group_4421,4421,,"Bills of exchange payable within one year",l10n_lu.lu_2011_chart_1
account_group_4422,4422,,"Bills of exchange payable after more than one year",l10n_lu.lu_2011_chart_1
account_group_45,45,,"Amounts payable to affiliated undertakings and to undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_451,451,,"Amounts payable to affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_4511,4511,,"Amounts payable to affiliated undertakings within one year",l10n_lu.lu_2011_chart_1
account_group_45111,45111,,"Purchases and services",l10n_lu.lu_2011_chart_1
account_group_45112,45112,,"Loans and advances",l10n_lu.lu_2011_chart_1
account_group_45118,45118,,"Other payables",l10n_lu.lu_2011_chart_1
account_group_4512,4512,,"Amounts payable to affiliated undertakings after more than one year",l10n_lu.lu_2011_chart_1
account_group_45121,45121,,"Purchases and services",l10n_lu.lu_2011_chart_1
account_group_45122,45122,,"Loans and advances",l10n_lu.lu_2011_chart_1
account_group_45128,45128,,"Other payables",l10n_lu.lu_2011_chart_1
account_group_452,452,,"Amounts payable to undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_4521,4521,,"Amounts payable to undertakings with which the undertaking is linked by virtue of participating interests within one year",l10n_lu.lu_2011_chart_1
account_group_45211,45211,,"Purchases and services",l10n_lu.lu_2011_chart_1
account_group_45212,45212,,"Loans and advances",l10n_lu.lu_2011_chart_1
account_group_45218,45218,,"Other payables",l10n_lu.lu_2011_chart_1
account_group_4522,4522,,"Amounts payable to undertakings with which the undertaking is linked by virtue of participating interests payable after more than one year",l10n_lu.lu_2011_chart_1
account_group_45221,45221,,"Purchases and services",l10n_lu.lu_2011_chart_1
account_group_45222,45222,,"Loans and advances",l10n_lu.lu_2011_chart_1
account_group_45228,45228,,"Other payables",l10n_lu.lu_2011_chart_1
account_group_46,46,,"Tax and social security debts",l10n_lu.lu_2011_chart_1
account_group_461,461,,"Tax debts",l10n_lu.lu_2011_chart_1
account_group_4611,4611,,"Municipal authorities",l10n_lu.lu_2011_chart_1
account_group_4612,4612,,"Direct Tax Authority (ACD)",l10n_lu.lu_2011_chart_1
account_group_46121,46121,,"Corporate income tax (CIT)",l10n_lu.lu_2011_chart_1
account_group_461211,461211,,"Corporate income tax - Tax accrual",l10n_lu.lu_2011_chart_1
account_group_461212,461212,,"CIT - Tax payable",l10n_lu.lu_2011_chart_1
account_group_46122,46122,,"Municipal business tax (MBT)",l10n_lu.lu_2011_chart_1
account_group_461221,461221,,"MBT - Tax accrual",l10n_lu.lu_2011_chart_1
account_group_461222,461222,,"MBT - Tax payable",l10n_lu.lu_2011_chart_1
account_group_46123,46123,,"Net wealth tax (NWT)",l10n_lu.lu_2011_chart_1
account_group_461231,461231,,"NWT - Tax accrual",l10n_lu.lu_2011_chart_1
account_group_461232,461232,,"NWT - Tax payable",l10n_lu.lu_2011_chart_1
account_group_46124,46124,,"Withholding tax on wages and salaries",l10n_lu.lu_2011_chart_1
account_group_46125,46125,,"Withholding tax on financial investment income",l10n_lu.lu_2011_chart_1
account_group_46126,46126,,"Withholding tax on director's fees",l10n_lu.lu_2011_chart_1
account_group_46128,46128,,"ACD - Other amounts payable",l10n_lu.lu_2011_chart_1
account_group_4613,4613,,"Customs and Excise Authority (ADA)",l10n_lu.lu_2011_chart_1
account_group_4614,4614,,"Indirect tax authorities (AED)",l10n_lu.lu_2011_chart_1
account_group_46141,46141,,"Value-added tax (VAT)",l10n_lu.lu_2011_chart_1
account_group_461411,461411,,"VAT received",l10n_lu.lu_2011_chart_1
account_group_461412,461412,,"VAT payable",l10n_lu.lu_2011_chart_1
account_group_461413,461413,,"VAT down payments received",l10n_lu.lu_2011_chart_1
account_group_461418,461418,,"VAT - Other payables",l10n_lu.lu_2011_chart_1
account_group_46142,46142,,"Indirect taxes",l10n_lu.lu_2011_chart_1
account_group_461421,461421,,"Registration duties",l10n_lu.lu_2011_chart_1
account_group_461422,461422,,"Subscription tax",l10n_lu.lu_2011_chart_1
account_group_461428,461428,,"Other indirect taxes",l10n_lu.lu_2011_chart_1
account_group_46148,46148,,"AED - Other debts",l10n_lu.lu_2011_chart_1
account_group_4615,4615,,"Foreign tax authorities",l10n_lu.lu_2011_chart_1
account_group_46151,46151,,"Foreign VAT",l10n_lu.lu_2011_chart_1
account_group_46158,46158,,"Other foreign taxes",l10n_lu.lu_2011_chart_1
account_group_462,462,,"Social security debts and other social securities offices",l10n_lu.lu_2011_chart_1
account_group_4621,4621,,"Joint Social Security Centre (CCSS)",l10n_lu.lu_2011_chart_1
account_group_4622,4622,,"Foreign Social Security offices",l10n_lu.lu_2011_chart_1
account_group_4628,4628,,"Other social bodies",l10n_lu.lu_2011_chart_1
account_group_47,47,,"Other debts",l10n_lu.lu_2011_chart_1
account_group_471,471,,"Other debts payable within one year",l10n_lu.lu_2011_chart_1
account_group_4711,4711,,"Received deposits and guarantees",l10n_lu.lu_2011_chart_1
account_group_4712,4712,,"Amounts payable to partners and shareholders (others than from affiliated undertakings)",l10n_lu.lu_2011_chart_1
account_group_4713,4713,,"Amounts payable to directors, managers, statutory auditors and similar",l10n_lu.lu_2011_chart_1
account_group_4714,4714,,"Amounts payable to staff",l10n_lu.lu_2011_chart_1
account_group_4715,4715,,"State - Greenhous gas and similar emission quotas to be returned or acquired",l10n_lu.lu_2011_chart_1
account_group_4716,4716,,"Loans and similar debts",l10n_lu.lu_2011_chart_1
account_group_47161,47161,,"Other loans",l10n_lu.lu_2011_chart_1
account_group_47162,47162,,"Lease debts",l10n_lu.lu_2011_chart_1
account_group_47163,47163,,"Life annuities",l10n_lu.lu_2011_chart_1
account_group_47168,47168,,"Other similar debts",l10n_lu.lu_2011_chart_1
account_group_4717,4717,,"Derivative financial instruments",l10n_lu.lu_2011_chart_1
account_group_4718,4718,,"Other miscellaneous debts",l10n_lu.lu_2011_chart_1
account_group_472,472,,"Other debts payable after more than one year",l10n_lu.lu_2011_chart_1
account_group_4721,4721,,"Received deposits and guarantees",l10n_lu.lu_2011_chart_1
account_group_4722,4722,,"Amounts payable to partners and shareholders (others than from affiliated undertakings)",l10n_lu.lu_2011_chart_1
account_group_4723,4723,,"Amounts payable to directors, managers, statutory auditors and similar",l10n_lu.lu_2011_chart_1
account_group_4724,4724,,"Amounts payable to staff",l10n_lu.lu_2011_chart_1
account_group_4726,4726,,"Loans and similar debts",l10n_lu.lu_2011_chart_1
account_group_47261,47261,,"Other loans",l10n_lu.lu_2011_chart_1
account_group_47262,47262,,"Lease debts",l10n_lu.lu_2011_chart_1
account_group_47263,47263,,"Life annuities",l10n_lu.lu_2011_chart_1
account_group_47268,47268,,"Other similar debts",l10n_lu.lu_2011_chart_1
account_group_4727,4727,,"Derivative financial instruments",l10n_lu.lu_2011_chart_1
account_group_4728,4728,,"Other miscellaneous debts",l10n_lu.lu_2011_chart_1
account_group_48,48,,"Deferred charges and income",l10n_lu.lu_2011_chart_1
account_group_481,481,,"Deferred charges (on one or more financial years)",l10n_lu.lu_2011_chart_1
account_group_482,482,,"Deferred income (on one or more financial years)",l10n_lu.lu_2011_chart_1
account_group_483,483,,"State - Greenhous gas and similar emission quotas received",l10n_lu.lu_2011_chart_1
account_group_484,484,,"Transitory or suspense accounts - Assets",l10n_lu.lu_2011_chart_1
account_group_485,485,,"Transitory or suspense accounts - Liabilities",l10n_lu.lu_2011_chart_1
account_group_486,486,,"Linking accounts (branches) - Assets",l10n_lu.lu_2011_chart_1
account_group_487,487,,"Linking accounts (branches) - Liabilities",l10n_lu.lu_2011_chart_1
account_group_5,5,,"FINANCIAL ACCOUNTS",l10n_lu.lu_2011_chart_1
account_group_50,50,,"Transferable securities",l10n_lu.lu_2011_chart_1
account_group_501,501,,"Shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_502,502,,"Own shares or own corporate units",l10n_lu.lu_2011_chart_1
account_group_503,503,,"Shares in undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_508,508,,"Other transferable securities",l10n_lu.lu_2011_chart_1
account_group_5081,5081,,"Shares - listed securities",l10n_lu.lu_2011_chart_1
account_group_5082,5082,,"Shares - unlisted securities",l10n_lu.lu_2011_chart_1
account_group_5083,5083,,"Debenture loans and other notes issued and repurchased by the company",l10n_lu.lu_2011_chart_1
account_group_5084,5084,,"Listed debenture loans",l10n_lu.lu_2011_chart_1
account_group_5085,5085,,"Unlisted debenture loans",l10n_lu.lu_2011_chart_1
account_group_5088,5088,,"Other miscellaneous transferable securities",l10n_lu.lu_2011_chart_1
account_group_51,51,,"Cash at bank, in postal cheques accounts, cheques and in hand",l10n_lu.lu_2011_chart_1
account_group_513,513,,"Banks and postal cheques accounts (CCP)",l10n_lu.lu_2011_chart_1
account_group_5131,5131,,"Banks and CCP : available balance",l10n_lu.lu_2011_chart_1
account_group_5132,5132,,"Banks and CCP : overdraft",l10n_lu.lu_2011_chart_1
account_group_516,516,,"Cash in hand",l10n_lu.lu_2011_chart_1
account_group_517,517,,"Internal transfers",l10n_lu.lu_2011_chart_1
account_group_5171,5171,,"Internal transfers : debit balance",l10n_lu.lu_2011_chart_1
account_group_5172,5172,,"Internal transfers : credit balance",l10n_lu.lu_2011_chart_1
account_group_518,518,,"Other cash amounts",l10n_lu.lu_2011_chart_1
account_group_6,6,,"CHARGES ACCOUNTS",l10n_lu.lu_2011_chart_1
account_group_60,60,,"Use of merchandise, raw and consumable materials",l10n_lu.lu_2011_chart_1
account_group_601,601,,"Purchases of raw materials",l10n_lu.lu_2011_chart_1
account_group_603,603,,"Purchases of consumable materials and supplies",l10n_lu.lu_2011_chart_1
account_group_6031,6031,,"Fuels, gas, water and electricity",l10n_lu.lu_2011_chart_1
account_group_60311,60311,,"Solid fuels",l10n_lu.lu_2011_chart_1
account_group_60312,60312,,"Liquid fuels",l10n_lu.lu_2011_chart_1
account_group_60313,60313,,"Gas",l10n_lu.lu_2011_chart_1
account_group_60314,60314,,"Water and sewage",l10n_lu.lu_2011_chart_1
account_group_60315,60315,,"Electricity",l10n_lu.lu_2011_chart_1
account_group_6032,6032,,"Maintenance supplies",l10n_lu.lu_2011_chart_1
account_group_6033,6033,,"Workshop, factory and store supplies and small equipment",l10n_lu.lu_2011_chart_1
account_group_6034,6034,,"Work clothes",l10n_lu.lu_2011_chart_1
account_group_6035,6035,,"Office and administrative supplies",l10n_lu.lu_2011_chart_1
account_group_6036,6036,,"Motor fuels",l10n_lu.lu_2011_chart_1
account_group_6037,6037,,"Lubricants",l10n_lu.lu_2011_chart_1
account_group_6038,6038,,"Other consumable supplies",l10n_lu.lu_2011_chart_1
account_group_604,604,,"Purchases of packaging",l10n_lu.lu_2011_chart_1
account_group_606,606,,"Purchases of merchandise and other goods for resale",l10n_lu.lu_2011_chart_1
account_group_6061,6061,,"Purchases of merchandise",l10n_lu.lu_2011_chart_1
account_group_6062,6062,,"Purchases of land for resale",l10n_lu.lu_2011_chart_1
account_group_6063,6063,,"Purchases of buildings for resale",l10n_lu.lu_2011_chart_1
account_group_607,607,,"Changes in inventory",l10n_lu.lu_2011_chart_1
account_group_6071,6071,,"Changes in inventory of raw materials",l10n_lu.lu_2011_chart_1
account_group_6073,6073,,"Changes in inventory of consumable materials and supplies",l10n_lu.lu_2011_chart_1
account_group_6074,6074,,"Changes in inventory of packaging",l10n_lu.lu_2011_chart_1
account_group_6076,6076,,"Changes in inventory of merchandise and other goods for resale",l10n_lu.lu_2011_chart_1
account_group_60761,60761,,"Merchandise",l10n_lu.lu_2011_chart_1
account_group_60762,60762,,"Land for resale",l10n_lu.lu_2011_chart_1
account_group_60763,60763,,"Buildings for resale",l10n_lu.lu_2011_chart_1
account_group_608,608,,"Purchases of items included in the production of goods and services",l10n_lu.lu_2011_chart_1
account_group_6081,6081,,"Services included in the production of goods and services",l10n_lu.lu_2011_chart_1
account_group_60811,60811,,"Tailoring",l10n_lu.lu_2011_chart_1
account_group_60812,60812,,"Research and development",l10n_lu.lu_2011_chart_1
account_group_60813,60813,,"Architects' and engineers' fees",l10n_lu.lu_2011_chart_1
account_group_60814,60814,,"Outsourcing included in the production of goods and services",l10n_lu.lu_2011_chart_1
account_group_6082,6082,,"Other purchases of material included in the production of goods and services",l10n_lu.lu_2011_chart_1
account_group_6083,6083,,"Purchase of greenhous gas and similar emission quotas",l10n_lu.lu_2011_chart_1
account_group_6088,6088,,"Other purchases included in the production of goods and services",l10n_lu.lu_2011_chart_1
account_group_609,609,,"Rebates, discounts and refunds (RDR) received and not directly deducted from purchases",l10n_lu.lu_2011_chart_1
account_group_6091,6091,,"RDR on purchases of raw materials",l10n_lu.lu_2011_chart_1
account_group_6093,6093,,"RDR on purchases of consumable materials and supplies",l10n_lu.lu_2011_chart_1
account_group_6094,6094,,"RDR on purchases of packaging",l10n_lu.lu_2011_chart_1
account_group_6096,6096,,"RDR on purchases of merchandise and other goods for resale",l10n_lu.lu_2011_chart_1
account_group_6098,6098,,"RDR on purchases included in the production of goods and services",l10n_lu.lu_2011_chart_1
account_group_6099,6099,,"Unallocated RDR",l10n_lu.lu_2011_chart_1
account_group_61,61,,"Other external charges",l10n_lu.lu_2011_chart_1
account_group_611,611,,"Rents and service charges",l10n_lu.lu_2011_chart_1
account_group_6111,6111,,"Rents and operationnal leasing for real property",l10n_lu.lu_2011_chart_1
account_group_61111,61111,,"Land",l10n_lu.lu_2011_chart_1
account_group_61112,61112,,"Buildings",l10n_lu.lu_2011_chart_1
account_group_6112,6112,,"Rents and operational leasing on movable property",l10n_lu.lu_2011_chart_1
account_group_61123,61123,,"Rolling stock",l10n_lu.lu_2011_chart_1
account_group_61128,61128,,"Other",l10n_lu.lu_2011_chart_1
account_group_6113,6113,,"Service charges and co-ownership expenses",l10n_lu.lu_2011_chart_1
account_group_6114,6114,,"Financial leasing on real property",l10n_lu.lu_2011_chart_1
account_group_6115,6115,,"Financial leasing on movable property",l10n_lu.lu_2011_chart_1
account_group_61153,61153,,"Rolling stock",l10n_lu.lu_2011_chart_1
account_group_61158,61158,,"Other",l10n_lu.lu_2011_chart_1
account_group_612,612,,"Subcontracting, servicing, repairs and maintenance",l10n_lu.lu_2011_chart_1
account_group_6121,6121,,"General subcontracting (not included in the production of goods and services)",l10n_lu.lu_2011_chart_1
account_group_6122,6122,,"Servicing, repairs and maintenance",l10n_lu.lu_2011_chart_1
account_group_61221,61221,,"Buildings",l10n_lu.lu_2011_chart_1
account_group_61223,61223,,"Rolling stock",l10n_lu.lu_2011_chart_1
account_group_61228,61228,,"Other",l10n_lu.lu_2011_chart_1
account_group_613,613,,"Remuneration of intermediaries and professional fees",l10n_lu.lu_2011_chart_1
account_group_6131,6131,,"Commissions and brokerage fees",l10n_lu.lu_2011_chart_1
account_group_6132,6132,,"IT services",l10n_lu.lu_2011_chart_1
account_group_6133,6133,,"Banking and similar services",l10n_lu.lu_2011_chart_1
account_group_61332,61332,,"Loans' issuance expenses",l10n_lu.lu_2011_chart_1
account_group_61333,61333,,"Bank account charges and bank commissions (included custody fees on securities)",l10n_lu.lu_2011_chart_1
account_group_61334,61334,,"Charges for electronic means of paiment",l10n_lu.lu_2011_chart_1
account_group_61336,61336,,"Factoring services",l10n_lu.lu_2011_chart_1
account_group_61338,61338,,"Other banking and similar services (except interest and similar expenses)",l10n_lu.lu_2011_chart_1
account_group_6134,6134,,"Professional fees",l10n_lu.lu_2011_chart_1
account_group_61341,61341,,"Legal, litigation and similar fees",l10n_lu.lu_2011_chart_1
account_group_61342,61342,,"Accounting, tax consulting, auditing and similar fees",l10n_lu.lu_2011_chart_1
account_group_61348,61348,,"Other professional fees",l10n_lu.lu_2011_chart_1
account_group_6135,6135,,"Notarial and similar fees",l10n_lu.lu_2011_chart_1
account_group_6138,6138,,"Other remuneration of intermediaries and professional fees",l10n_lu.lu_2011_chart_1
account_group_614,614,,"Insurance premiums",l10n_lu.lu_2011_chart_1
account_group_6141,6141,,"Insurance for assets",l10n_lu.lu_2011_chart_1
account_group_61411,61411,,"Buildings",l10n_lu.lu_2011_chart_1
account_group_61412,61412,,"Rolling stock",l10n_lu.lu_2011_chart_1
account_group_61418,61418,,"Other",l10n_lu.lu_2011_chart_1
account_group_6142,6142,,"Insurance on rented assets",l10n_lu.lu_2011_chart_1
account_group_6143,6143,,"Transport insurance",l10n_lu.lu_2011_chart_1
account_group_6144,6144,,"Business risk insurance",l10n_lu.lu_2011_chart_1
account_group_6145,6145,,"Customers credit insurance",l10n_lu.lu_2011_chart_1
account_group_6146,6146,,"Third-party insurance",l10n_lu.lu_2011_chart_1
account_group_6148,6148,,"Other insurances",l10n_lu.lu_2011_chart_1
account_group_615,615,,"Marketing and communication costs",l10n_lu.lu_2011_chart_1
account_group_6151,6151,,"Marketing and advertising costs",l10n_lu.lu_2011_chart_1
account_group_61511,61511,,"Press advertising",l10n_lu.lu_2011_chart_1
account_group_61512,61512,,"Samples",l10n_lu.lu_2011_chart_1
account_group_61513,61513,,"Fairs and exhibitions",l10n_lu.lu_2011_chart_1
account_group_61514,61514,,"Gifts to customers",l10n_lu.lu_2011_chart_1
account_group_61515,61515,,"Catalogues, printed materials and publications",l10n_lu.lu_2011_chart_1
account_group_61516,61516,,"Donations",l10n_lu.lu_2011_chart_1
account_group_61517,61517,,"Sponsorship",l10n_lu.lu_2011_chart_1
account_group_61518,61518,,"Other purchases of advertising services",l10n_lu.lu_2011_chart_1
account_group_6152,6152,,"Travel and entertainment expenses",l10n_lu.lu_2011_chart_1
account_group_61521,61521,,"Travel expenses",l10n_lu.lu_2011_chart_1
account_group_615211,615211,,"Management (if appropriate owner and partner)",l10n_lu.lu_2011_chart_1
account_group_615212,615212,,"Staff",l10n_lu.lu_2011_chart_1
account_group_61522,61522,,"Relocation expenses",l10n_lu.lu_2011_chart_1
account_group_61523,61523,,"Business assignments",l10n_lu.lu_2011_chart_1
account_group_61524,61524,,"Receptions and entertainment costs",l10n_lu.lu_2011_chart_1
account_group_6153,6153,,"Postal charges and telecommunication costs",l10n_lu.lu_2011_chart_1
account_group_61531,61531,,"Postal charges",l10n_lu.lu_2011_chart_1
account_group_61532,61532,,"Telecommunication costs",l10n_lu.lu_2011_chart_1
account_group_616,616,,"Transportation of goods and collective staff transportation",l10n_lu.lu_2011_chart_1
account_group_6161,6161,,"Transportation of purchased goods",l10n_lu.lu_2011_chart_1
account_group_6162,6162,,"Transportation of sold goods",l10n_lu.lu_2011_chart_1
account_group_6165,6165,,"Collective staff transportation",l10n_lu.lu_2011_chart_1
account_group_6168,6168,,"Other transportation",l10n_lu.lu_2011_chart_1
account_group_617,617,,"External staff of the company",l10n_lu.lu_2011_chart_1
account_group_6171,6171,,"Temporary staff",l10n_lu.lu_2011_chart_1
account_group_6172,6172,,"External staff on secondment",l10n_lu.lu_2011_chart_1
account_group_618,618,,"Miscellaneous external charges",l10n_lu.lu_2011_chart_1
account_group_6181,6181,,"Documentation",l10n_lu.lu_2011_chart_1
account_group_6182,6182,,"Costs of training, symposiums, seminars, conferences",l10n_lu.lu_2011_chart_1
account_group_6183,6183,,"Industrial and non-industrial waste treatment",l10n_lu.lu_2011_chart_1
account_group_6184,6184,,"Fuels, gas, water and electricity (not included in the production of goods and services)",l10n_lu.lu_2011_chart_1
account_group_61841,61841,,"Solid fuels",l10n_lu.lu_2011_chart_1
account_group_61842,61842,,"Liquid fuels (oil, motor fuel, etc.)",l10n_lu.lu_2011_chart_1
account_group_61843,61843,,"Gas",l10n_lu.lu_2011_chart_1
account_group_61844,61844,,"Water and waste water",l10n_lu.lu_2011_chart_1
account_group_61845,61845,,"Electricity",l10n_lu.lu_2011_chart_1
account_group_6185,6185,,"Supplies and small equipment",l10n_lu.lu_2011_chart_1
account_group_61851,61851,,"Office supplies",l10n_lu.lu_2011_chart_1
account_group_61852,61852,,"Small equipment",l10n_lu.lu_2011_chart_1
account_group_61853,61853,,"Work clothes",l10n_lu.lu_2011_chart_1
account_group_61854,61854,,"Maintenance supplies",l10n_lu.lu_2011_chart_1
account_group_61858,61858,,"Other",l10n_lu.lu_2011_chart_1
account_group_6186,6186,,"Surveillance and security charges",l10n_lu.lu_2011_chart_1
account_group_6187,6187,,"Contributions to professional associations",l10n_lu.lu_2011_chart_1
account_group_6188,6188,,"Other miscellaneous external charges",l10n_lu.lu_2011_chart_1
account_group_619,619,,"Rebates, discounts and refunds received on other external charges",l10n_lu.lu_2011_chart_1
account_group_62,62,,"Staff expenses",l10n_lu.lu_2011_chart_1
account_group_621,621,,"Staff remuneration",l10n_lu.lu_2011_chart_1
account_group_6211,6211,,"Gross wages",l10n_lu.lu_2011_chart_1
account_group_62111,62111,,"Base wages",l10n_lu.lu_2011_chart_1
account_group_62112,62112,,"Wage supplements",l10n_lu.lu_2011_chart_1
account_group_621121,621121,,"Sunday",l10n_lu.lu_2011_chart_1
account_group_621122,621122,,"Public holidays",l10n_lu.lu_2011_chart_1
account_group_621123,621123,,"Overtime",l10n_lu.lu_2011_chart_1
account_group_621128,621128,,"Other supplements",l10n_lu.lu_2011_chart_1
account_group_62114,62114,,"Incentives, bonuses and commissions",l10n_lu.lu_2011_chart_1
account_group_62115,62115,,"Benefits in kind",l10n_lu.lu_2011_chart_1
account_group_62116,62116,,"Severance pay",l10n_lu.lu_2011_chart_1
account_group_62117,62117,,"Survivor's pay",l10n_lu.lu_2011_chart_1
account_group_6218,6218,,"Other benefits",l10n_lu.lu_2011_chart_1
account_group_6219,6219,,"Refunds on wages paid",l10n_lu.lu_2011_chart_1
account_group_622,622,,"Other staff remuneration",l10n_lu.lu_2011_chart_1
account_group_6221,6221,,"Students",l10n_lu.lu_2011_chart_1
account_group_6222,6222,,"Casual workers",l10n_lu.lu_2011_chart_1
account_group_6228,6228,,"Other",l10n_lu.lu_2011_chart_1
account_group_623,623,,"Social security costs (employer's share)",l10n_lu.lu_2011_chart_1
account_group_6231,6231,,"Social security on pensions",l10n_lu.lu_2011_chart_1
account_group_6232,6232,,"Other social security costs (including illness, accidents, a.s.o.)",l10n_lu.lu_2011_chart_1
account_group_624,624,,"Other staff expenses",l10n_lu.lu_2011_chart_1
account_group_6241,6241,,"Complementary pensions",l10n_lu.lu_2011_chart_1
account_group_62411,62411,,"Premiums for external pensions funds",l10n_lu.lu_2011_chart_1
account_group_62412,62412,,"Changes to provisions for complementary pensions",l10n_lu.lu_2011_chart_1
account_group_62413,62413,,"Withholding tax on complementary pensions",l10n_lu.lu_2011_chart_1
account_group_62414,62414,,"Insolvency insurance premiums",l10n_lu.lu_2011_chart_1
account_group_62415,62415,,"Complementary pensions paid by the employer",l10n_lu.lu_2011_chart_1
account_group_6248,6248,,"Other staff expenses not mentioned above",l10n_lu.lu_2011_chart_1
account_group_63,63,,"Allocations to value adjustments (AVA) and fair value adjustments (FVA) on formation expenses, intangible, tangible and current assets (except transferable securities)",l10n_lu.lu_2011_chart_1
account_group_631,631,,"AVA on formation expenses and similar expenses",l10n_lu.lu_2011_chart_1
account_group_6311,6311,,"AVA on set-up and start-up costs",l10n_lu.lu_2011_chart_1
account_group_6313,6313,,"AVA on expenses for capital increases and various operations (mergers, demergers, changes of legal form)",l10n_lu.lu_2011_chart_1
account_group_6314,6314,,"AVA on loan-issuance expenses",l10n_lu.lu_2011_chart_1
account_group_6318,6318,,"AVA on other similar expenses",l10n_lu.lu_2011_chart_1
account_group_632,632,,"AVA on intangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_6321,6321,,"AVA on development costs",l10n_lu.lu_2011_chart_1
account_group_6322,6322,,"AVA on concessions, patents, licences, trademarks and similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_6323,6323,,"AVA on goodwill acquired for consideration",l10n_lu.lu_2011_chart_1
account_group_6324,6324,,"AVA on down payments and intangible fixed assets under development",l10n_lu.lu_2011_chart_1
account_group_633,633,,"AVA on tangible fixed assets and fair value adjustments (FVA) on investment properties",l10n_lu.lu_2011_chart_1
account_group_6331,6331,,"AVA on land, fittings-out and buildings and FVA on investment properties",l10n_lu.lu_2011_chart_1
account_group_63311,63311,,"AVA on land",l10n_lu.lu_2011_chart_1
account_group_63312,63312,,"AVA on fixtures and fittings-out of land",l10n_lu.lu_2011_chart_1
account_group_63313,63313,,"AVA on buildings",l10n_lu.lu_2011_chart_1
account_group_63314,63314,,"AVA on fixtures and fittings-out of buildings",l10n_lu.lu_2011_chart_1
account_group_63315,63315,,"FVA on investment properties",l10n_lu.lu_2011_chart_1
account_group_6332,6332,,"AVA on plant and machinery",l10n_lu.lu_2011_chart_1
account_group_6333,6333,,"AVA on other fixtures and fittings, tools and equipment (including rolling stock)",l10n_lu.lu_2011_chart_1
account_group_6334,6334,,"AVA on down payments and tangible fixed assets under development",l10n_lu.lu_2011_chart_1
account_group_634,634,,"AVA on inventories",l10n_lu.lu_2011_chart_1
account_group_6341,6341,,"AVA on inventories of raw materials and consumables",l10n_lu.lu_2011_chart_1
account_group_6342,6342,,"AVA on inventories of work and contracts in progress",l10n_lu.lu_2011_chart_1
account_group_6343,6343,,"AVA on inventories of goods",l10n_lu.lu_2011_chart_1
account_group_6344,6344,,"AVA on inventories of merchandise and other goods for resale",l10n_lu.lu_2011_chart_1
account_group_6345,6345,,"AVA on down payments on inventories",l10n_lu.lu_2011_chart_1
account_group_635,635,,"AVA and FVA on receivables from current assets",l10n_lu.lu_2011_chart_1
account_group_6351,6351,,"AVA on trade receivables",l10n_lu.lu_2011_chart_1
account_group_6352,6352,,"AVA on amounts owed by affiliated undertakings and undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_6353,6353,,"AVA on other receivables",l10n_lu.lu_2011_chart_1
account_group_6354,6354,,"FVA on receivables from current assets",l10n_lu.lu_2011_chart_1
account_group_64,64,,"Other operating charges",l10n_lu.lu_2011_chart_1
account_group_641,641,,"Fees and royalties for concessions, patents, licences, trademarks and similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_6411,6411,,"Concessions",l10n_lu.lu_2011_chart_1
account_group_6412,6412,,"Patents",l10n_lu.lu_2011_chart_1
account_group_6413,6413,,"Software licences",l10n_lu.lu_2011_chart_1
account_group_6414,6414,,"Trademarks and franchise",l10n_lu.lu_2011_chart_1
account_group_6415,6415,,"Similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_64151,64151,,"Copyrights and reproduction rights",l10n_lu.lu_2011_chart_1
account_group_64158,64158,,"Other similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_642,642,,"Indemnities, damages and interest",l10n_lu.lu_2011_chart_1
account_group_643,643,,"Attendance fees, director's fees and similar remuneration",l10n_lu.lu_2011_chart_1
account_group_6431,6431,,"Attendance fees",l10n_lu.lu_2011_chart_1
account_group_6432,6432,,"Director's fees",l10n_lu.lu_2011_chart_1
account_group_6438,6438,,"Other similar remuneration",l10n_lu.lu_2011_chart_1
account_group_644,644,,"Loss on disposal of intangible and tangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_6441,6441,,"Loss on disposal of intangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_64411,64411,,"Book value of yielded intangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_64412,64412,,"Disposal proceeds of intangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_6442,6442,,"Loss on disposal of tangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_64421,64421,,"Book value of yielded tangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_64422,64422,,"Disposal proceeds of tangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_645,645,,"Losses on bad debts",l10n_lu.lu_2011_chart_1
account_group_6451,6451,,"Trade receivables",l10n_lu.lu_2011_chart_1
account_group_6452,6452,,"Amounts owed by affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_6453,6453,,"Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_6454,6454,,"Other receivables",l10n_lu.lu_2011_chart_1
account_group_646,646,,"Taxes, duties and similar expenses",l10n_lu.lu_2011_chart_1
account_group_6461,6461,,"Real property tax",l10n_lu.lu_2011_chart_1
account_group_6462,6462,,"Non-refundable VAT",l10n_lu.lu_2011_chart_1
account_group_6463,6463,,"Duties on imported merchandise",l10n_lu.lu_2011_chart_1
account_group_6464,6464,,"Excise duties on production and tax on consumption",l10n_lu.lu_2011_chart_1
account_group_6465,6465,,"Registration fees, stamp duties and mortgage duties",l10n_lu.lu_2011_chart_1
account_group_64651,64651,,"Registration fees",l10n_lu.lu_2011_chart_1
account_group_64658,64658,,"Other registration fees, stamp duties and mortgage duties",l10n_lu.lu_2011_chart_1
account_group_6466,6466,,"Motor-vehicle taxes",l10n_lu.lu_2011_chart_1
account_group_6467,6467,,"Bar licence tax",l10n_lu.lu_2011_chart_1
account_group_6468,6468,,"Other duties and taxes",l10n_lu.lu_2011_chart_1
account_group_647,647,,"Allocations to tax-exempt capital gains",l10n_lu.lu_2011_chart_1
account_group_648,648,,"Other miscellaneous operating charges",l10n_lu.lu_2011_chart_1
account_group_6481,6481,,"Fines, sanctions and penalties",l10n_lu.lu_2011_chart_1
account_group_6488,6488,,"Miscellaneous operating charges",l10n_lu.lu_2011_chart_1
account_group_649,649,,"Allocations to provisions",l10n_lu.lu_2011_chart_1
account_group_6491,6491,,"Allocations to tax provisions",l10n_lu.lu_2011_chart_1
account_group_6492,6492,,"Allocations to operating provisions",l10n_lu.lu_2011_chart_1
account_group_65,65,,"Financial charges",l10n_lu.lu_2011_chart_1
account_group_651,651,,"Allocations to value adjustments (AVA) and fair-value adjustments (FVA) of financial fixed assets",l10n_lu.lu_2011_chart_1
account_group_6511,6511,,"AVA on financial fixed assets",l10n_lu.lu_2011_chart_1
account_group_65111,65111,,"AVA on shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_65112,65112,,"AVA on amounts owed by affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_65113,65113,,"AVA on participating interests",l10n_lu.lu_2011_chart_1
account_group_65114,65114,,"AVA on amounts owed by undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_65115,65115,,"AVA on securities held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_65116,65116,,"AVA on loans, deposits and claims held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_6512,6512,,"FVA on financial fixed assets",l10n_lu.lu_2011_chart_1
account_group_652,652,,"Charges and loss of disposal of financial fixed assets",l10n_lu.lu_2011_chart_1
account_group_6521,6521,,"Charges of financial fixed assets",l10n_lu.lu_2011_chart_1
account_group_65211,65211,,"Shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_65212,65212,,"Amounts owed by affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_65213,65213,,"Participating interests",l10n_lu.lu_2011_chart_1
account_group_65214,65214,,"Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_65215,65215,,"Securities held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_65216,65216,,"Loans, deposits and claims held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_6522,6522,,"Loss on disposal of financial fixed assets",l10n_lu.lu_2011_chart_1
account_group_65221,65221,,"Loss on disposal of shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_652211,652211,,"Book value of yielded shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_652212,652212,,"Disposal proceeds of shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_65222,65222,,"Loss on disposal of amounts owed by affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_652221,652221,,"Book value of yielded amounts owed by affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_652222,652222,,"Disposal proceeds of amounts owed by affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_65223,65223,,"Loss on disposal of participating interests",l10n_lu.lu_2011_chart_1
account_group_652231,652231,,"Book value of yielded participating interests",l10n_lu.lu_2011_chart_1
account_group_652232,652232,,"Disposal proceeds of participating interests",l10n_lu.lu_2011_chart_1
account_group_65224,65224,,"Loss on disposal of amounts owed by undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_652241,652241,,"Book value of yielded amounts owed by undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_652242,652242,,"Disposal proceeds of amounts owed by undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_65225,65225,,"Loss on disposal of securities held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_652251,652251,,"Book value of yielded securities held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_652252,652252,,"Disposal proceeds of securities held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_65226,65226,,"Loss on disposal of loans, deposits and claims held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_652261,652261,,"Book value of yielded loans, deposits and claims held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_652262,652262,,"Disposal proceeds of loans, deposits and claims held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_653,653,,"Allocations to value adjustment (AVA) and fair-value adjustments (FVA) on transferable securities",l10n_lu.lu_2011_chart_1
account_group_6531,6531,,"AVA on transferable securities",l10n_lu.lu_2011_chart_1
account_group_65311,65311,,"AVA on shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_65312,65312,,"AVA on own shares or own corporate units",l10n_lu.lu_2011_chart_1
account_group_65313,65313,,"AVA on shares in undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_65318,65318,,"AVA on other transferable securities",l10n_lu.lu_2011_chart_1
account_group_6532,6532,,"FVA on transferable securities",l10n_lu.lu_2011_chart_1
account_group_654,654,,"Loss on disposal of receivables and transferable securities from current assets",l10n_lu.lu_2011_chart_1
account_group_6541,6541,,"Loss on disposal of receivables from current assets",l10n_lu.lu_2011_chart_1
account_group_65411,65411,,"From affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_65412,65412,,"From undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_65413,65413,,"from other receivables from current assets",l10n_lu.lu_2011_chart_1
account_group_6542,6542,,"Loss on disposal of transferable securities",l10n_lu.lu_2011_chart_1
account_group_65421,65421,,"Shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_65422,65422,,"Own shares or corporate units",l10n_lu.lu_2011_chart_1
account_group_65423,65423,,"Shares in in undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_65428,65428,,"Other transferable securities",l10n_lu.lu_2011_chart_1
account_group_655,655,,"Interest and discounts",l10n_lu.lu_2011_chart_1
account_group_6551,6551,,"Interest on debenture loans",l10n_lu.lu_2011_chart_1
account_group_65511,65511,,"Interest on debenture loans - affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_65512,65512,,"Interest on debenture loans - other",l10n_lu.lu_2011_chart_1
account_group_6552,6552,,"Banking and similar interest",l10n_lu.lu_2011_chart_1
account_group_65521,65521,,"Banking interest on current accounts",l10n_lu.lu_2011_chart_1
account_group_65522,65522,,"Banking interest on financing operations",l10n_lu.lu_2011_chart_1
account_group_65523,65523,,"Interest on financial leases",l10n_lu.lu_2011_chart_1
account_group_655231,655231,,"Interest on financial leases - affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_655232,655232,,"Interest on financial leases - other",l10n_lu.lu_2011_chart_1
account_group_6553,6553,,"Interest on trade payables",l10n_lu.lu_2011_chart_1
account_group_6554,6554,,"Interest payable to affiliated undertakings and undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_65541,65541,,"Interest payable to affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_65542,65542,,"Interest payable to undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_6555,6555,,"Discounts and charges on bills of exchange",l10n_lu.lu_2011_chart_1
account_group_65551,65551,,"Discounts and charges on bills of exchange - affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_65552,65552,,"Discounts and charges on bills of exchange - other",l10n_lu.lu_2011_chart_1
account_group_6556,6556,,"Granted discounts",l10n_lu.lu_2011_chart_1
account_group_65561,65561,,"Granted discounts - affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_65562,65562,,"Granted discounts - other",l10n_lu.lu_2011_chart_1
account_group_6558,6558,,"Interest payable on other loans and debts",l10n_lu.lu_2011_chart_1
account_group_65581,65581,,"Interest payable on other loans and debts - affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_65582,65582,,"Interest payable on other loans and debts - other",l10n_lu.lu_2011_chart_1
account_group_656,656,,"Foreign currency exchange losses",l10n_lu.lu_2011_chart_1
account_group_6561,6561,,"Foreign currency exchange losses - affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_6562,6562,,"Foreign currency exchange losses - other",l10n_lu.lu_2011_chart_1
account_group_657,657,,"Share in the losses of undertakings accounted for under the equity method",l10n_lu.lu_2011_chart_1
account_group_658,658,,"Other financial charges",l10n_lu.lu_2011_chart_1
account_group_6581,6581,,"Other financial charges - affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_6582,6582,,"Other financial charges - other",l10n_lu.lu_2011_chart_1
account_group_659,659,,"Allocations to financial provisions",l10n_lu.lu_2011_chart_1
account_group_6591,6591,,"Allocations to financial provisions - affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_6592,6592,,"Allocations to financial provisions - other",l10n_lu.lu_2011_chart_1
account_group_67,67,,"Income taxes",l10n_lu.lu_2011_chart_1
account_group_671,671,,"Corporate income tax (CIT)",l10n_lu.lu_2011_chart_1
account_group_6711,6711,,"CIT - current financial year",l10n_lu.lu_2011_chart_1
account_group_6712,6712,,"CIT - previous financial years",l10n_lu.lu_2011_chart_1
account_group_672,672,,"Municipal business tax",l10n_lu.lu_2011_chart_1
account_group_6721,6721,,"MBT - current financial year",l10n_lu.lu_2011_chart_1
account_group_6722,6722,,"MBT - previous financial years",l10n_lu.lu_2011_chart_1
account_group_673,673,,"Foreign income taxes",l10n_lu.lu_2011_chart_1
account_group_6731,6731,,"Withholding taxes",l10n_lu.lu_2011_chart_1
account_group_6732,6732,,"Taxes levied on permanent establishments",l10n_lu.lu_2011_chart_1
account_group_67321,67321,,"Current financial year",l10n_lu.lu_2011_chart_1
account_group_67322,67322,,"Previous financial years",l10n_lu.lu_2011_chart_1
account_group_6733,6733,,"Taxes levied on non-resident undertakings",l10n_lu.lu_2011_chart_1
account_group_6738,6738,,"Other foreign income taxes",l10n_lu.lu_2011_chart_1
account_group_679,679,,"Allocations to provisions for deferred taxes",l10n_lu.lu_2011_chart_1
account_group_68,68,,"Other taxes not included in the previous caption",l10n_lu.lu_2011_chart_1
account_group_681,681,,"Net wealth tax (NWT)",l10n_lu.lu_2011_chart_1
account_group_6811,6811,,"NWT - current financial year",l10n_lu.lu_2011_chart_1
account_group_6812,6812,,"NWT - previous financial years",l10n_lu.lu_2011_chart_1
account_group_682,682,,"Subscription tax",l10n_lu.lu_2011_chart_1
account_group_683,683,,"Foreign taxes",l10n_lu.lu_2011_chart_1
account_group_688,688,,"Other taxes",l10n_lu.lu_2011_chart_1
account_group_7,7,,"INCOME ACCOUNTS",l10n_lu.lu_2011_chart_1
account_group_70,70,,"Net turnover",l10n_lu.lu_2011_chart_1
account_group_702,702,,"Sales of goods",l10n_lu.lu_2011_chart_1
account_group_7021,7021,,"Sales of finished goods",l10n_lu.lu_2011_chart_1
account_group_7022,7022,,"Sales of semi-finished goods",l10n_lu.lu_2011_chart_1
account_group_7023,7023,,"Sales of residual products",l10n_lu.lu_2011_chart_1
account_group_7029,7029,,"Sales of work in progress",l10n_lu.lu_2011_chart_1
account_group_703,703,,"Sales of services",l10n_lu.lu_2011_chart_1
account_group_7031,7031,,"Fees and royalties for concessions, patents, licences, trademarks and similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_70311,70311,,"Concessions",l10n_lu.lu_2011_chart_1
account_group_70312,70312,,"Patents",l10n_lu.lu_2011_chart_1
account_group_70313,70313,,"Software licences",l10n_lu.lu_2011_chart_1
account_group_70314,70314,,"Trademarks and franchises",l10n_lu.lu_2011_chart_1
account_group_70315,70315,,"Similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_703151,703151,,"Copyrights and reproduction rights",l10n_lu.lu_2011_chart_1
account_group_703158,703158,,"Other similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_7032,7032,,"Rental income",l10n_lu.lu_2011_chart_1
account_group_70321,70321,,"Rental income from real property",l10n_lu.lu_2011_chart_1
account_group_70322,70322,,"Rental income from movable property",l10n_lu.lu_2011_chart_1
account_group_7033,7033,,"Sales of services not mentioned above",l10n_lu.lu_2011_chart_1
account_group_7039,7039,,"Sales of services in the course of completion",l10n_lu.lu_2011_chart_1
account_group_704,704,,"Sales of packaging",l10n_lu.lu_2011_chart_1
account_group_705,705,,"Commissions and brokerage fees",l10n_lu.lu_2011_chart_1
account_group_706,706,,"Sales of merchandise and other goods for resale",l10n_lu.lu_2011_chart_1
account_group_7061,7061,,"Sales of merchandise",l10n_lu.lu_2011_chart_1
account_group_7062,7062,,"Sales of land resale",l10n_lu.lu_2011_chart_1
account_group_7063,7063,,"Sales of buildings for resale",l10n_lu.lu_2011_chart_1
account_group_708,708,,"Other components of turnover",l10n_lu.lu_2011_chart_1
account_group_709,709,,"Rebates, discounts and refunds (RDR) granted and not immediately deducted from sales",l10n_lu.lu_2011_chart_1
account_group_7092,7092,,"RDR on sales of goods",l10n_lu.lu_2011_chart_1
account_group_7093,7093,,"RDR on sales of services",l10n_lu.lu_2011_chart_1
account_group_7094,7094,,"RDR on sales of packages",l10n_lu.lu_2011_chart_1
account_group_7095,7095,,"RDR on commissions and brokerage fees",l10n_lu.lu_2011_chart_1
account_group_7096,7096,,"RDR on sales of merchandise and other goods for resale",l10n_lu.lu_2011_chart_1
account_group_7098,7098,,"RDR on other components of turnover",l10n_lu.lu_2011_chart_1
account_group_7099,7099,,"Not allocated rebates, discounts and refunds",l10n_lu.lu_2011_chart_1
account_group_71,71,,"Change in inventories of goods and of work in progress",l10n_lu.lu_2011_chart_1
account_group_711,711,,"Change in inventories of work and contracts in progress",l10n_lu.lu_2011_chart_1
account_group_7111,7111,,"Change in inventories of work in progress",l10n_lu.lu_2011_chart_1
account_group_7112,7112,,"Change in inventories: contracts in progress - goods",l10n_lu.lu_2011_chart_1
account_group_7113,7113,,"Change in inventories: contracts in progress - services",l10n_lu.lu_2011_chart_1
account_group_7114,7114,,"Change in inventories: buildings under construction",l10n_lu.lu_2011_chart_1
account_group_712,712,,"Change in inventories of goods",l10n_lu.lu_2011_chart_1
account_group_7121,7121,,"Change in inventories of finished goods",l10n_lu.lu_2011_chart_1
account_group_7122,7122,,"Change in inventories of semi-finished goods",l10n_lu.lu_2011_chart_1
account_group_7123,7123,,"Change in inventories of residual goods",l10n_lu.lu_2011_chart_1
account_group_72,72,,"Capitalised production",l10n_lu.lu_2011_chart_1
account_group_721,721,,"Intangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_7211,7211,,"Development costs",l10n_lu.lu_2011_chart_1
account_group_7212,7212,,"Concessions, patents, licences, trademarks and similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_72121,72121,,"Concessions",l10n_lu.lu_2011_chart_1
account_group_72122,72122,,"Patents",l10n_lu.lu_2011_chart_1
account_group_72123,72123,,"Software licences",l10n_lu.lu_2011_chart_1
account_group_72124,72124,,"Trademarks and franchises",l10n_lu.lu_2011_chart_1
account_group_72125,72125,,"Similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_721251,721251,,"Copyrights and reproduction rights",l10n_lu.lu_2011_chart_1
account_group_721258,721258,,"Other similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_722,722,,"Tangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_7221,7221,,"Land, fittings and buildings",l10n_lu.lu_2011_chart_1
account_group_7222,7222,,"Plant and machinery",l10n_lu.lu_2011_chart_1
account_group_7223,7223,,"Other fixtures and fittings, tools and equipment (included motor vehicles)",l10n_lu.lu_2011_chart_1
account_group_73,73,,"Reversals of value adjustments (RVA) on intangible, tangible and current assets (except transferable securities)",l10n_lu.lu_2011_chart_1
account_group_732,732,,"RVA on intangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_7321,7321,,"RVA on development costs",l10n_lu.lu_2011_chart_1
account_group_7322,7322,,"RVA on concessions, patents, licences, trademarks and similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_7324,7324,,"RVA on down payments and intangible fixed assets under development",l10n_lu.lu_2011_chart_1
account_group_733,733,,"RVA on tangible fixed assets and fair value adjustments (FVA) on investment properties",l10n_lu.lu_2011_chart_1
account_group_7331,7331,,"RVA on land, fixtures and fittings-out and buildings and FVA on investment properties",l10n_lu.lu_2011_chart_1
account_group_73311,73311,,"RVA on land",l10n_lu.lu_2011_chart_1
account_group_73312,73312,,"RVA on fixtures and fittings-out of land",l10n_lu.lu_2011_chart_1
account_group_73313,73313,,"RVA on buildings",l10n_lu.lu_2011_chart_1
account_group_73314,73314,,"RVA on fixtures and fittings-out of buildings",l10n_lu.lu_2011_chart_1
account_group_73315,73315,,"FVA on investment properties",l10n_lu.lu_2011_chart_1
account_group_7332,7332,,"RVA on plant and machinery",l10n_lu.lu_2011_chart_1
account_group_7333,7333,,"Other fixtures and fittings, tools and equipment (included motor vehicles)",l10n_lu.lu_2011_chart_1
account_group_7334,7334,,"RVA on down payments and tangible fixed assets under development",l10n_lu.lu_2011_chart_1
account_group_734,734,,"RVA on inventories",l10n_lu.lu_2011_chart_1
account_group_7341,7341,,"RVA on inventories of raw materials and consumables",l10n_lu.lu_2011_chart_1
account_group_7342,7342,,"RVA on inventories of work and contracts in progress",l10n_lu.lu_2011_chart_1
account_group_7343,7343,,"RVA on inventories of goods",l10n_lu.lu_2011_chart_1
account_group_7344,7344,,"RVA on inventories of merchandise and other goods for resale",l10n_lu.lu_2011_chart_1
account_group_7345,7345,,"RVA on down payments on inventories",l10n_lu.lu_2011_chart_1
account_group_735,735,,"RVA and FVA on receivables from current assets",l10n_lu.lu_2011_chart_1
account_group_7351,7351,,"RVA on trade receivables",l10n_lu.lu_2011_chart_1
account_group_7352,7352,,"RVA on amounts owed by affiliated undertakings and undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_7353,7353,,"RVA on other receivables",l10n_lu.lu_2011_chart_1
account_group_7354,7354,,"FVA on receivables from current assets",l10n_lu.lu_2011_chart_1
account_group_74,74,,"Other operating income",l10n_lu.lu_2011_chart_1
account_group_741,741,,"Fees and royalties for concessions, patents, licences, trademarks and similar rights and assets from ancillary activities",l10n_lu.lu_2011_chart_1
account_group_7411,7411,,"Concessions",l10n_lu.lu_2011_chart_1
account_group_7412,7412,,"Patents",l10n_lu.lu_2011_chart_1
account_group_7413,7413,,"Software licences",l10n_lu.lu_2011_chart_1
account_group_7414,7414,,"Trademarks and franchises",l10n_lu.lu_2011_chart_1
account_group_7415,7415,,"Similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_74151,74151,,"Copyrights and reproduction rights",l10n_lu.lu_2011_chart_1
account_group_74158,74158,,"Other similar rights and assets",l10n_lu.lu_2011_chart_1
account_group_742,742,,"Rental income from ancillary activities",l10n_lu.lu_2011_chart_1
account_group_7421,7421,,"Rental income on real property",l10n_lu.lu_2011_chart_1
account_group_7422,7422,,"Rental income on movable property",l10n_lu.lu_2011_chart_1
account_group_743,743,,"Attendance fees, director's fees and similar remunerations",l10n_lu.lu_2011_chart_1
account_group_744,744,,"Gain on disposal of intangible and tangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_7441,7441,,"Gain on disposal of intangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_74411,74411,,"Book value of yielded intangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_74412,74412,,"Disposal proceeds of intangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_7442,7442,,"Income of yielded tangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_74421,74421,,"Book value of yielded tangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_74422,74422,,"Disposal proceeds of tangible fixed assets",l10n_lu.lu_2011_chart_1
account_group_745,745,,"Subsidies for operating activities",l10n_lu.lu_2011_chart_1
account_group_7451,7451,,"Product subsidies",l10n_lu.lu_2011_chart_1
account_group_7452,7452,,"Interest subsidies",l10n_lu.lu_2011_chart_1
account_group_7453,7453,,"Compensatory allowances",l10n_lu.lu_2011_chart_1
account_group_7454,7454,,"Subsidies in favour of employment development",l10n_lu.lu_2011_chart_1
account_group_7458,7458,,"Other subsidies for operating activities",l10n_lu.lu_2011_chart_1
account_group_746,746,,"Benefits in kind",l10n_lu.lu_2011_chart_1
account_group_747,747,,"Reversals of temporarily not taxable capital gains and of investment subsidies",l10n_lu.lu_2011_chart_1
account_group_7471,7471,,"Temporarily not taxable capital gains not reinvested",l10n_lu.lu_2011_chart_1
account_group_7472,7472,,"Temporarily not taxable capital gains reinvested",l10n_lu.lu_2011_chart_1
account_group_7473,7473,,"Capital investment subsidies",l10n_lu.lu_2011_chart_1
account_group_748,748,,"Other miscellaneous operating income",l10n_lu.lu_2011_chart_1
account_group_7481,7481,,"Insurance indemnities",l10n_lu.lu_2011_chart_1
account_group_7488,7488,,"Miscellaneous operating income",l10n_lu.lu_2011_chart_1
account_group_749,749,,"Reversals of provisions",l10n_lu.lu_2011_chart_1
account_group_7491,7491,,"Reversals of provisions for taxes",l10n_lu.lu_2011_chart_1
account_group_7492,7492,,"Reversals of operating provisions",l10n_lu.lu_2011_chart_1
account_group_75,75,,"Financial income",l10n_lu.lu_2011_chart_1
account_group_751,751,,"Reversals of value adjustments (RVA) and fair-value adjustments (FVA) on financial fixed assets",l10n_lu.lu_2011_chart_1
account_group_7511,7511,,"RVA on financial fixed assets",l10n_lu.lu_2011_chart_1
account_group_75111,75111,,"RVA on shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_75112,75112,,"RVA on amounts owed by affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_75113,75113,,"RVA on participating interests",l10n_lu.lu_2011_chart_1
account_group_75114,75114,,"RVA on amounts owed by undertakings with which the company is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_75115,75115,,"RVA on securities held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_75116,75116,,"RVA on loans, deposits and claims held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_7512,7512,,"FVA on financial fixed assets",l10n_lu.lu_2011_chart_1
account_group_752,752,,"Income and gain on disposal of financial fixed assets",l10n_lu.lu_2011_chart_1
account_group_7521,7521,,"Income from financial fixed assets",l10n_lu.lu_2011_chart_1
account_group_75211,75211,,"Shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_75212,75212,,"Amounts owed by affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_75213,75213,,"Participating interests",l10n_lu.lu_2011_chart_1
account_group_75214,75214,,"Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_75215,75215,,"Securities held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_75216,75216,,"Loans, deposits and claims held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_7522,7522,,"Gain on disposal of financial fixed assets",l10n_lu.lu_2011_chart_1
account_group_75221,75221,,"Shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_752211,752211,,"Book value of yielded shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_752212,752212,,"Disposal proceeds of shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_75222,75222,,"Amounts owed by affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_752221,752221,,"Book value of yielded amounts owed by affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_752222,752222,,"Disposal proceeds of amounts owed by affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_75223,75223,,"Participating interests",l10n_lu.lu_2011_chart_1
account_group_752231,752231,,"Book value of yielded participating interests",l10n_lu.lu_2011_chart_1
account_group_752232,752232,,"Disposal proceeds of participating interests",l10n_lu.lu_2011_chart_1
account_group_75224,75224,,"Amounts owed by undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_752241,752241,,"Book value of yielded amounts owed by undertakings with which the undertaking is linked by virtue of participating  interests",l10n_lu.lu_2011_chart_1
account_group_752242,752242,,"Disposal proceeds of amounts owed by undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_75225,75225,,"Securities held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_752251,752251,,"Book value of yielded securities held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_752252,752252,,"Disposal proceeds of securities held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_75226,75226,,"Loans, deposits and claims held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_752261,752261,,"Book value of yielded loans, deposits and claims held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_752262,752262,,"Disposal proceed of loans, deposits and claims held as fixed assets",l10n_lu.lu_2011_chart_1
account_group_753,753,,"Reversals of value adjustments (RVA) and fair-value adjustments (FVA) on transferable securities",l10n_lu.lu_2011_chart_1
account_group_7531,7531,,"RVA on transferable securities",l10n_lu.lu_2011_chart_1
account_group_75311,75311,,"RVA on shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_75312,75312,,"RVA on own shares or corporate units",l10n_lu.lu_2011_chart_1
account_group_75313,75313,,"RVA on shares in undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_75318,75318,,"RVA on other transferable securities",l10n_lu.lu_2011_chart_1
account_group_7532,7532,,"FVA on transferable securities",l10n_lu.lu_2011_chart_1
account_group_754,754,,"Gain on disposal and other income from current receivables and transferable securities of current assets",l10n_lu.lu_2011_chart_1
account_group_7541,7541,,"Gain on disposal of receivables from current assets",l10n_lu.lu_2011_chart_1
account_group_75411,75411,,"on affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_75412,75412,,"on undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_75413,75413,,"on other current receivables",l10n_lu.lu_2011_chart_1
account_group_7542,7542,,"Gain on disposal of transferable securities",l10n_lu.lu_2011_chart_1
account_group_75421,75421,,"Shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_75422,75422,,"Own shares or corporate units",l10n_lu.lu_2011_chart_1
account_group_75423,75423,,"Shares in undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_75428,75428,,"Other transferable securities",l10n_lu.lu_2011_chart_1
account_group_7548,7548,,"Other income from transferable securities",l10n_lu.lu_2011_chart_1
account_group_75481,75481,,"Shares in affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_75482,75482,,"Own shares or corporate units",l10n_lu.lu_2011_chart_1
account_group_75483,75483,,"Shares in undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_75488,75488,,"Other transferable securities",l10n_lu.lu_2011_chart_1
account_group_755,755,,"Other interest income from current assets and discounts",l10n_lu.lu_2011_chart_1
account_group_7552,7552,,"Bank and similar interest",l10n_lu.lu_2011_chart_1
account_group_75521,75521,,"Interest on bank accounts",l10n_lu.lu_2011_chart_1
account_group_75523,75523,,"Interest on financial leases",l10n_lu.lu_2011_chart_1
account_group_755231,755231,,"from affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_755232,755232,,"from other",l10n_lu.lu_2011_chart_1
account_group_7553,7553,,"Interest on trade receivables",l10n_lu.lu_2011_chart_1
account_group_7554,7554,,"Interest on amounts owed by affiliated undertakings and undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_75541,75541,,"Interest on amounts owed by affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_75542,75542,,"Interest on amounts owed by undertakings with which the undertaking is linked by virtue of participating interests",l10n_lu.lu_2011_chart_1
account_group_7555,7555,,"Discounts on bills of exchange",l10n_lu.lu_2011_chart_1
account_group_75551,75551,,"Discounts on bills of exchange - affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_75552,75552,,"Discounts on bills of exchange - other",l10n_lu.lu_2011_chart_1
account_group_7556,7556,,"Discounts received",l10n_lu.lu_2011_chart_1
account_group_75561,75561,,"Discounts received - affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_75562,75562,,"Discounts received - other",l10n_lu.lu_2011_chart_1
account_group_7558,7558,,"Interest on other amounts receivable",l10n_lu.lu_2011_chart_1
account_group_75581,75581,,"Interest on other amounts receivable - affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_75582,75582,,"Interest on other amounts receivable - other",l10n_lu.lu_2011_chart_1
account_group_756,756,,"Foreign currency exchange gains",l10n_lu.lu_2011_chart_1
account_group_7561,7561,,"Foreign currency exchange gains - affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_7562,7562,,"Foreign currency exchange gains - other",l10n_lu.lu_2011_chart_1
account_group_757,757,,"Share of profit from undertakings accounted for under the equity method",l10n_lu.lu_2011_chart_1
account_group_758,758,,"Other financial income",l10n_lu.lu_2011_chart_1
account_group_7581,7581,,"Other financial income - affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_7582,7582,,"Other financial income - other",l10n_lu.lu_2011_chart_1
account_group_759,759,,"Reversals of financial provisions",l10n_lu.lu_2011_chart_1
account_group_7591,7591,,"Reversals of financial provisions - affiliated undertakings",l10n_lu.lu_2011_chart_1
account_group_7592,7592,,"Reversals of financial provisions - other",l10n_lu.lu_2011_chart_1
account_group_77,77,,"Adjustments of income taxes",l10n_lu.lu_2011_chart_1
account_group_771,771,,"Adjustments of corporate income tax (CIT)",l10n_lu.lu_2011_chart_1
account_group_772,772,,"Adjustments of municipal business tax (MBT)",l10n_lu.lu_2011_chart_1
account_group_773,773,,"Adjustments of foreign income taxes",l10n_lu.lu_2011_chart_1
account_group_779,779,,"Reversals of provisions for deferred taxes",l10n_lu.lu_2011_chart_1
account_group_78,78,,"Adjustments of other taxes not included in the previous caption",l10n_lu.lu_2011_chart_1
account_group_781,781,,"Adjustments of net wealth tax (NWT)",l10n_lu.lu_2011_chart_1
account_group_782,782,,"Adjustments of subscription tax",l10n_lu.lu_2011_chart_1
account_group_783,783,,"Adjustments of foreign taxes",l10n_lu.lu_2011_chart_1
account_group_788,788,,"Adjustments of other taxes",l10n_lu.lu_2011_chart_1

```

## File: data\account.tax.group.csv

```csv
id,name
tax_group_0,TVA 0%
tax_group_3,TVA 3%
tax_group_6,TVA 6%
tax_group_7,TVA 7%
tax_group_8,TVA 8%
tax_group_10,TVA 10%
tax_group_12,TVA 12%
tax_group_13,TVA 13%
tax_group_14,TVA 14%
tax_group_15,TVA 15%
tax_group_16,TVA 16%
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
        <field name="rule_type">writeoff_button</field>
    </record>
    <record id="bank_fees_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="l10n_lu.bank_fees_template"/>
        <field name="account_id" ref="lu_2011_account_61333"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Bank Fees</field>
    </record>
    <record id="cash_discount_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="name">Cash Discount</field>
        <field name="rule_type">writeoff_button</field>
    </record>
    <record id="cash_discount_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="l10n_lu.cash_discount_template"/>
        <field name="account_id" ref="lu_2020_account_65562"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Cash Discount</field>
    </record>
</odoo>

```

## File: data\account_tax_report_line.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.lu"/>
    </record>

    <record id="account_tax_report_line_2_assesment_of_tax_due" model="account.tax.report.line">
      <field name="name">II. ASSESSMENT OF TAX DUE (output tax)</field>
      <field name="sequence">2</field>
      <field name="report_id" ref="tax_report"/>
      <field name="formula">None</field>
    </record>

    <record id="account_tax_report_line_2b_intra_community_acqui_of_goods_base" model="account.tax.report.line">
      <field name="name">051 - Intra-Community acquisitions of goods – base</field>
      <field name="sequence">3</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
    </record>

    <record id="account_tax_report_line_2b_base_17" model="account.tax.report.line">
      <field name="name">711 - base 17%</field>
      <field name="tag_name">711</field>
      <field name="sequence">6</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2b_base_16" model="account.tax.report.line">
      <field name="name">911 - base 16%</field>
      <field name="tag_name">911</field>
      <field name="sequence">7</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2b_base_14" model="account.tax.report.line">
      <field name="name">713 - base 14%</field>
      <field name="tag_name">713</field>
      <field name="sequence">8</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2b_base_13" model="account.tax.report.line">
      <field name="name">913 - base 13%</field>
      <field name="tag_name">913</field>
      <field name="sequence">9</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2b_base_8" model="account.tax.report.line">
      <field name="name">715 - base 8%</field>
      <field name="tag_name">715</field>
      <field name="sequence">10</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2b_base_7" model="account.tax.report.line">
      <field name="name">915 - base 7%</field>
      <field name="tag_name">915</field>
      <field name="sequence">11</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2b_base_3" model="account.tax.report.line">
      <field name="name">049 - base 3%</field>
      <field name="tag_name">049</field>
      <field name="sequence">12</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2b_base_exempt" model="account.tax.report.line">
      <field name="name">194 - base exempt</field>
      <field name="tag_name">194</field>
      <field name="sequence">13</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2b_manufactured_tobacco" model="account.tax.report.line">
      <field name="name">719 - of manufactured tobacco (VAT is collected at the exit of the tax warehouse with excise duties)</field>
      <field name="tag_name">719</field>
      <field name="sequence">14</field>
      <field name="code">LUTAX_719</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acqui_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_importation_of_goods_base" model="account.tax.report.line">
      <field name="name">065 - Importation of goods – base</field>
      <field name="sequence">6</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
    </record>

    <record id="account_tax_report_line_2d_1_base_17" model="account.tax.report.line">
      <field name="name">721 - for business purposes: base 17%</field>
      <field name="tag_name">721</field>
      <field name="sequence">3</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_1_base_16" model="account.tax.report.line">
      <field name="name">921 - for business purposes: base 16%</field>
      <field name="tag_name">921</field>
      <field name="sequence">4</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_1_base_14" model="account.tax.report.line">
      <field name="name">723 - for business purposes: base 14%</field>
      <field name="tag_name">723</field>
      <field name="sequence">5</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_1_base_13" model="account.tax.report.line">
      <field name="name">923 - for business purposes: base 13%</field>
      <field name="tag_name">923</field>
      <field name="sequence">6</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_1_base_8" model="account.tax.report.line">
      <field name="name">725 - for business purposes: base 8%</field>
      <field name="tag_name">725</field>
      <field name="sequence">7</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_1_base_7" model="account.tax.report.line">
      <field name="name">925 - for business purposes: base 7%</field>
      <field name="tag_name">925</field>
      <field name="sequence">8</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_1_base_3" model="account.tax.report.line">
      <field name="name">059 - for business purposes: base 3%</field>
      <field name="tag_name">059</field>
      <field name="sequence">9</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_1_base_exempt" model="account.tax.report.line">
      <field name="name">195 - for business purposes: base exempt</field>
      <field name="tag_name">195</field>
      <field name="sequence">10</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_1_manufactured_tobacco" model="account.tax.report.line">
      <field name="name">729 - of manufactured tobacco (VAT is collected at the exit of the tax warehouse with excise duties)</field>
      <field name="tag_name">729</field>
      <field name="sequence">11</field>
      <field name="code">LUTAX_729</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_2_base_17" model="account.tax.report.line">
      <field name="name">731 - for non-business purposes: base 17%</field>
      <field name="tag_name">731</field>
      <field name="sequence">12</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_2_base_16" model="account.tax.report.line">
      <field name="name">931 - for non-business purposes: base 16%</field>
      <field name="tag_name">931</field>
      <field name="sequence">13</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_2_base_14" model="account.tax.report.line">
      <field name="name">733 - for non-business purposes: base 14%</field>
      <field name="tag_name">733</field>
      <field name="sequence">14</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_2_base_13" model="account.tax.report.line">
      <field name="name">933 - for non-business purposes: base 13%</field>
      <field name="tag_name">933</field>
      <field name="sequence">15</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_2_base_8" model="account.tax.report.line">
      <field name="name">735 - for non-business purposes: base 8%</field>
      <field name="tag_name">735</field>
      <field name="sequence">16</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_2_base_7" model="account.tax.report.line">
      <field name="name">935 - for non-business purposes: base 7%</field>
      <field name="tag_name">935</field>
      <field name="sequence">17</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_2_base_3" model="account.tax.report.line">
      <field name="name">063 - for non-business purposes: base 3%</field>
      <field name="tag_name">063</field>
      <field name="sequence">18</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2d_2_base_exempt" model="account.tax.report.line">
      <field name="name">196 - for non-business purposes: base exempt</field>
      <field name="tag_name">196</field>
      <field name="sequence">19</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_base"/>
    </record>

    <record id="account_tax_report_line_2c_acquisitions_triangular_transactions_base" model="account.tax.report.line">
      <field name="name">152 - Acquisitions, in the context of triangular transactions – base</field>
      <field name="tag_name">152</field>
      <field name="sequence">5</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
    </record>

    <record id="account_tax_report_line_2e_supply_of_service_for_customer" model="account.tax.report.line">
      <field name="name">409 - Supply of services for which the customer is liable for the payment of VAT – base</field>
      <field name="sequence">8</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
    </record>

    <record id="account_tax_report_line_2e_1_base" model="account.tax.report.line">
      <field name="name">436 - base</field>
      <field name="sequence">1</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_supply_of_service_for_customer"/>
    </record>

    <record id="account_tax_report_line_2e_1_b_exempt" model="account.tax.report.line">
      <field name="name">435 - exempt within the territory: exempt</field>
      <field name="tag_name">435</field>
      <field name="sequence">2</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2e_supply_of_service_for_customer"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_base_17" model="account.tax.report.line">
      <field name="name">741 - not exempt within the territory: base 17%</field>
      <field name="tag_name">741</field>
      <field name="sequence">6</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_1_base"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_base_16" model="account.tax.report.line">
      <field name="name">941 - not exempt within the territory: base 16%</field>
      <field name="tag_name">941</field>
      <field name="sequence">7</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_1_base"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_base_14" model="account.tax.report.line">
      <field name="name">743 - not exempt within the territory: base 14%</field>
      <field name="tag_name">743</field>
      <field name="sequence">8</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_1_base"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_base_13" model="account.tax.report.line">
      <field name="name">943 - not exempt within the territory: base 13%</field>
      <field name="tag_name">943</field>
      <field name="sequence">9</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_1_base"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_base_8" model="account.tax.report.line">
      <field name="name">745 - not exempt within the territory: base 8%</field>
      <field name="tag_name">745</field>
      <field name="sequence">10</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_1_base"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_base_7" model="account.tax.report.line">
      <field name="name">945 - not exempt within the territory: base 7%</field>
      <field name="tag_name">945</field>
      <field name="sequence">11</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_1_base"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_base_3" model="account.tax.report.line">
      <field name="name">431 - not exempt within the territory: base 3%</field>
      <field name="tag_name">431</field>
      <field name="sequence">12</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_1_base"/>
    </record>

    <record id="account_tax_report_line_2e_2_base" model="account.tax.report.line">
      <field name="name">463 - base</field>
      <field name="sequence">3</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2e_supply_of_service_for_customer"/>
    </record>

    <record id="account_tax_report_line_2e_2_base_17" model="account.tax.report.line">
      <field name="name">751 - not established or residing within the Community: base 17%</field>
      <field name="tag_name">751</field>
      <field name="sequence">6</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_2_base"/>
    </record>

    <record id="account_tax_report_line_2e_2_base_16" model="account.tax.report.line">
      <field name="name">951 - not established or residing within the Community: base 16%</field>
      <field name="tag_name">951</field>
      <field name="sequence">7</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_2_base"/>
    </record>

    <record id="account_tax_report_line_2e_2_base_14" model="account.tax.report.line">
      <field name="name">753 - not established or residing within the Community: base 14%</field>
      <field name="tag_name">753</field>
      <field name="sequence">8</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_2_base"/>
    </record>

    <record id="account_tax_report_line_2e_2_base_13" model="account.tax.report.line">
      <field name="name">953 - not established or residing within the Community: base 13%</field>
      <field name="tag_name">953</field>
      <field name="sequence">9</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_2_base"/>
    </record>

    <record id="account_tax_report_line_2e_2_base_8" model="account.tax.report.line">
      <field name="name">755 - not established or residing within the Community: base 8%</field>
      <field name="tag_name">755</field>
      <field name="sequence">10</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_2_base"/>
    </record>

    <record id="account_tax_report_line_2e_2_base_7" model="account.tax.report.line">
      <field name="name">955 - not established or residing within the Community: base 7%</field>
      <field name="tag_name">955</field>
      <field name="sequence">11</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_2_base"/>
    </record>

    <record id="account_tax_report_line_2e_2_base_3" model="account.tax.report.line">
      <field name="name">441 - not established or residing within the Community: base 3%</field>
      <field name="tag_name">441</field>
      <field name="sequence">12</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_2_base"/>
    </record>

    <record id="account_tax_report_line_2e_2_exempt" model="account.tax.report.line">
      <field name="name">445 - not established or residing within the Community: exempt</field>
      <field name="tag_name">445</field>
      <field name="sequence">13</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_2_base"/>
    </record>

    <record id="account_tax_report_line_2e_3_base" model="account.tax.report.line">
      <field name="name">765 - base</field>
      <field name="sequence">4</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2e_supply_of_service_for_customer"/>
    </record>

    <record id="account_tax_report_line_2e_3_base_17" model="account.tax.report.line">
      <field name="name">761 - suppliers established within the territory: base 17%</field>
      <field name="tag_name">761</field>
      <field name="sequence">1</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_3_base"/>
    </record>

    <record id="account_tax_report_line_2e_3_base_16" model="account.tax.report.line">
      <field name="name">961 - suppliers established within the territory: base 16%</field>
      <field name="tag_name">961</field>
      <field name="sequence">2</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2e_3_base"/>
    </record>

    <record id="account_tax_report_line_1_assessment_taxable_turnover" model="account.tax.report.line">
      <field name="name">I. ASSESSMENT OF TAXABLE TURNOVER</field>
      <field name="sequence">1</field>
      <field name="report_id" ref="tax_report"/>
      <field name="formula">None</field>
    </record>

    <record id="account_tax_report_line_1a_overall_turnover" model="account.tax.report.line">
      <field name="name">012 - Overall turnover</field>
      <field name="sequence">1</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_1_assessment_taxable_turnover"/>
      <field name="formula">LUTAX_454 + LUTAX_455 + LUTAX_456</field>
    </record>

    <record id="account_tax_report_line_1a_total_sale" model="account.tax.report.line">
        <field name="name">454 - Total Sales / Receipts</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_1a_overall_turnover"/>
        <field name="report_id" ref="tax_report"/>
        <field name="code">LUTAX_454</field>
        <field name="formula">LUTAX_471 + LUTAX_472</field>
    </record>

    <!-- 471 not managed, only 472 is filled in -->
    <record id="account_tax_report_line_1a_telecom_service" model="account.tax.report.line">
        <field name="name">471 - Telecommunications services, radio and television broadcasting services...</field>
        <field name="tag_name">471</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_1a_total_sale"/>
        <field name="report_id" ref="tax_report"/>
        <field name="code">LUTAX_471</field>
    </record>

    <record id="account_tax_report_line_1a_other_sales" model="account.tax.report.line">
        <field name="name">472 - Other sales / receipts</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_1a_total_sale"/>
        <field name="report_id" ref="tax_report"/>
        <field name="code">LUTAX_472</field>
        <field name="formula">LUTAX_021 + LUTAX_037 - LUTAX_456 - LUTAX_455 - LUTAX_471</field>
    </record>

    <record id="account_tax_report_line_1a_app_goods_non_bus" model="account.tax.report.line">
        <field name="name">455 - Application of goods for non-business use and for business purposes</field>
        <field name="tag_name">455</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_1a_overall_turnover"/>
        <field name="report_id" ref="tax_report"/>
        <field name="code">LUTAX_455</field>
    </record>

    <record id="account_tax_report_line_1a_non_bus_gs" model="account.tax.report.line">
        <field name="name">456 - Non-business use of goods and supply of services free of charge</field>
        <field name="tag_name">456</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_1a_overall_turnover"/>
        <field name="report_id" ref="tax_report"/>
        <field name="code">LUTAX_456</field>
    </record>

    <record id="account_tax_report_line_1b_exemptions_deductible_amounts" model="account.tax.report.line">
      <field name="name">021 - Exemptions and deductible amounts</field>
      <field name="sequence">2</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_1_assessment_taxable_turnover"/>
      <field name="code">LUTAX_021</field>
    </record>

    <record id="account_tax_report_line_1b_2_export" model="account.tax.report.line">
      <field name="name">014 - Exports</field>
      <field name="tag_name">014</field>
      <field name="sequence">2</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
    </record>

    <record id="account_tax_report_line_1b_1_intra_community_goods_pi_vat" model="account.tax.report.line">
      <field name="name">457 - Intra-Community supply of goods to persons identified for VAT purposes in another Member State (MS)</field>
      <field name="tag_name">457</field>
      <field name="sequence">1</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
    </record>

    <record id="account_tax_report_line_1b_3_other_exemptions_art_43" model="account.tax.report.line">
      <field name="name">015 - Other exemptions</field>
      <field name="tag_name">015</field>
      <field name="sequence">3</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
    </record>

    <record id="account_tax_report_line_1b_4_other_exemptions_art_44_et_56quater" model="account.tax.report.line">
      <field name="name">016 - Other exemptions</field>
      <field name="tag_name">016</field>
      <field name="sequence">4</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
    </record>

    <record id="account_tax_report_line_1b_5_manufactured_tobacco_vat_collected" model="account.tax.report.line">
      <field name="name">017 - Manufactured tobacco whose VAT was collected at the source or at the exit of the tax...</field>
      <field name="tag_name">017</field>
      <field name="sequence">5</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
    </record>

    <record id="account_tax_report_line_1b_6_a_subsequent_to_intra_community" model="account.tax.report.line">
      <field name="name">018 - Supply, subsequent to intra-Community acquisitions of goods, in the context of triangular transactions, when the customer identified,...</field>
      <field name="tag_name">018</field>
      <field name="sequence">6</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
    </record>

    <record id="account_tax_report_line_1b_6_b1_non_exempt_customer_vat" model="account.tax.report.line">
      <field name="name">423 - not exempt in the MS where the customer is liable for payment of VAT</field>
      <field name="tag_name">423</field>
      <field name="sequence">7</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
    </record>

    <record id="account_tax_report_line_1b_6_b2_exempt_ms_customer" model="account.tax.report.line">
      <field name="name">424 - exempt in the MS where the customer is identified</field>
      <field name="tag_name">424</field>
      <field name="sequence">8</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
    </record>

    <record id="account_tax_report_line_1b_6_c_supplies_scope_special_arrangement" model="account.tax.report.line">
      <field name="name">226 - Supplies carried out within the scope of the special arrangement of art. 56sexies</field>
      <field name="tag_name">226</field>
      <field name="sequence">9</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
    </record>

    <record id="account_tax_report_line_1b_6_d_supplies_other_referred" model="account.tax.report.line">
      <field name="name">019 - Supplies other than referred to in 018 and 423 or 424</field>
      <field name="tag_name">019</field>
      <field name="sequence">10</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
    </record>

    <record id="account_tax_report_line_1b_7_inland_supplies_for_customer" model="account.tax.report.line">
      <field name="name">419 - Inland supplies for which the customer is liable for the payment of VAT</field>
      <field name="tag_name">419</field>
      <field name="sequence">11</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_1b_exemptions_deductible_amounts"/>
    </record>

    <record id="account_tax_report_line_1c_taxable_turnover" model="account.tax.report.line">
      <field name="name">022 - Taxable turnover</field>
      <field name="sequence">3</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_1_assessment_taxable_turnover"/>
      <field name="code">LUTAX_022</field>
      <field name="formula">LUTAX_037</field>
    </record>

    <record id="account_tax_report_line_2a_breakdown_taxable_turnover_base" model="account.tax.report.line">
      <field name="name">037 - Breakdown of taxable turnover – base</field>
      <field name="sequence">1</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="code">LUTAX_037</field>
    </record>

    <record id="account_tax_report_line_2a_base_17" model="account.tax.report.line">
      <field name="name">701 - base 17%</field>
      <field name="tag_name">701</field>
      <field name="sequence">6</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_base"/>
    </record>

    <record id="account_tax_report_line_2a_base_16" model="account.tax.report.line">
      <field name="name">901 - base 16%</field>
      <field name="tag_name">901</field>
      <field name="sequence">7</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_base"/>
    </record>

    <record id="account_tax_report_line_2a_base_14" model="account.tax.report.line">
      <field name="name">703 - base 14%</field>
      <field name="tag_name">703</field>
      <field name="sequence">8</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_base"/>
    </record>

    <record id="account_tax_report_line_2a_base_13" model="account.tax.report.line">
      <field name="name">903 - base 13%</field>
      <field name="tag_name">903</field>
      <field name="sequence">9</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_base"/>
    </record>

    <record id="account_tax_report_line_2a_base_8" model="account.tax.report.line">
      <field name="name">705 - base 8%</field>
      <field name="tag_name">705</field>
      <field name="sequence">10</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_base"/>
    </record>

    <record id="account_tax_report_line_2a_base_7" model="account.tax.report.line">
      <field name="name">905 - base 7%</field>
      <field name="tag_name">905</field>
      <field name="sequence">11</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_base"/>
    </record>

    <record id="account_tax_report_line_2a_base_3" model="account.tax.report.line">
      <field name="name">031 - base 3%</field>
      <field name="tag_name">031</field>
      <field name="sequence">12</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_base"/>
    </record>

    <record id="account_tax_report_line_2a_base_0" model="account.tax.report.line">
      <field name="name">033 - base 0%</field>
      <field name="tag_name">033</field>
      <field name="sequence">13</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_base"/>
      <field name="code">LUTAX_033</field>
    </record>

    <record id="account_tax_report_line_4_tax_tobe_paid_or_reclaimed" model="account.tax.report.line">
      <field name="name">IV. TAX TO BE PAID OR TO BE RECLAIMED</field>
      <field name="sequence">4</field>
      <field name="report_id" ref="tax_report"/>
      <field name="formula">None</field>
    </record>

    <record id="account_tax_report_line_4c_exceeding_amount" model="account.tax.report.line">
      <field name="name">105 - Exceeding amount</field>
      <field name="sequence">3</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_4_tax_tobe_paid_or_reclaimed"/>
      <field name="formula">LUTAX_103 - LUTAX_102</field>
    </record>

    <record id="account_tax_report_line_4a_total_tax_due" model="account.tax.report.line">
      <field name="name">103 - Total tax due</field>
      <field name="sequence">1</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_4_tax_tobe_paid_or_reclaimed"/>
      <field name="code">LUTAX_103</field>
      <field name="formula">LUTAX_046 + LUTAX_056 + LUTAX_407 + LUTAX_410 + LUTAX_768 + LUTAX_227</field>
    </record>

    <record id="account_tax_report_line_2f_supply_goods_base" model="account.tax.report.line">
      <field name="name">767 - Supply of goods for which the purchaser is liable for the payment of VAT - base</field>
      <field name="sequence">10</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
    </record>

    <record id="account_tax_report_line_2f_supply_goods_base_8" model="account.tax.report.line">
      <field name="name">763 - base 8%</field>
      <field name="tag_name">763</field>
      <field name="sequence">1</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2f_supply_goods_base"/>
    </record>

    <record id="account_tax_report_line_2f_supply_goods_base_7" model="account.tax.report.line">
      <field name="name">963 - base 7%</field>
      <field name="tag_name">963</field>
      <field name="sequence">2</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2f_supply_goods_base"/>
    </record>

    <record id="account_tax_report_line_2f_supply_goods_tax" model="account.tax.report.line">
      <field name="name">768 - Supply of goods for which the purchaser is liable for the payment of VAT - tax</field>
      <field name="sequence">11</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="code">LUTAX_768</field>
    </record>

    <record id="account_tax_report_line_2f_supply_goods_tax_8" model="account.tax.report.line">
      <field name="name">764 - tax 8%</field>
      <field name="tag_name">764</field>
      <field name="sequence">1</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2f_supply_goods_tax"/>
      <field name="code">LUTAX_764</field>
    </record>

    <record id="account_tax_report_line_2f_supply_goods_tax_7" model="account.tax.report.line">
      <field name="name">964 - tax 7%</field>
      <field name="tag_name">964</field>
      <field name="sequence">1</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2f_supply_goods_tax"/>
      <field name="code">LUTAX_964</field>
    </record>

    <record id="account_tax_report_line_2g_special_arrangement" model="account.tax.report.line">
      <field name="name">227 - Special arrangement for tax suspension: adjustment</field>
      <field name="tag_name">227</field>
      <field name="sequence">12</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="code">LUTAX_227</field>
    </record>

    <record id="account_tax_report_line_2h_total_tax_due" model="account.tax.report.line">
      <field name="name">076 - Total tax due</field>
      <field name="sequence">13</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="formula">LUTAX_103</field>
    </record>

    <record id="account_tax_report_line_2a_breakdown_taxable_turnover_tax" model="account.tax.report.line">
      <field name="name">046 - Breakdown of taxable turnover – tax</field>
      <field name="sequence">2</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="code">LUTAX_046</field>
    </record>

    <record id="account_tax_report_line_2a_tax_17" model="account.tax.report.line">
      <field name="name">702 - tax 17%</field>
      <field name="tag_name">702</field>
      <field name="sequence">6</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_tax"/>
      <field name="code">LUTAX_702</field>
    </record>

    <record id="account_tax_report_line_2a_tax_16" model="account.tax.report.line">
      <field name="name">902 - tax 16%</field>
      <field name="tag_name">902</field>
      <field name="sequence">7</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_tax"/>
      <field name="code">LUTAX_902</field>
    </record>

    <record id="account_tax_report_line_2a_tax_14" model="account.tax.report.line">
      <field name="name">704 - tax 14%</field>
      <field name="tag_name">704</field>
      <field name="sequence">8</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_tax"/>
      <field name="code">LUTAX_704</field>
    </record>

    <record id="account_tax_report_line_2a_tax_13" model="account.tax.report.line">
      <field name="name">904 - tax 13%</field>
      <field name="tag_name">904</field>
      <field name="sequence">9</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_tax"/>
      <field name="code">LUTAX_904</field>
    </record>

    <record id="account_tax_report_line_2a_tax_8" model="account.tax.report.line">
      <field name="name">706 - tax 8%</field>
      <field name="tag_name">706</field>
      <field name="sequence">10</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_tax"/>
      <field name="code">LUTAX_706</field>
    </record>

    <record id="account_tax_report_line_2a_tax_7" model="account.tax.report.line">
      <field name="name">906 - tax 7%</field>
      <field name="tag_name">906</field>
      <field name="sequence">11</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_tax"/>
      <field name="code">LUTAX_906</field>
    </record>

    <record id="account_tax_report_line_2a_tax_3" model="account.tax.report.line">
      <field name="name">040 - tax 3%</field>
      <field name="tag_name">040</field>
      <field name="sequence">12</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2a_breakdown_taxable_turnover_tax"/>
      <field name="code">LUTAX_040</field>
    </record>

    <record id="account_tax_report_line_2b_intra_community_acquisitions_goods_tax" model="account.tax.report.line">
      <field name="name">056 - Intra-Community acquisitions of goods – tax</field>
      <field name="sequence">4</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="code">LUTAX_056</field>
    </record>

    <record id="account_tax_report_line_2b_tax_17" model="account.tax.report.line">
      <field name="name">712 - tax 17%</field>
      <field name="tag_name">712</field>
      <field name="sequence">5</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acquisitions_goods_tax"/>
      <field name="code">LUTAX_712</field>
    </record>

    <record id="account_tax_report_line_2b_tax_16" model="account.tax.report.line">
      <field name="name">912 - tax 16%</field>
      <field name="tag_name">912</field>
      <field name="sequence">6</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acquisitions_goods_tax"/>
      <field name="code">LUTAX_912</field>
    </record>

    <record id="account_tax_report_line_2b_tax_14" model="account.tax.report.line">
      <field name="name">714 - tax 14%</field>
      <field name="tag_name">714</field>
      <field name="sequence">7</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acquisitions_goods_tax"/>
      <field name="code">LUTAX_714</field>
    </record>

    <record id="account_tax_report_line_2b_tax_13" model="account.tax.report.line">
      <field name="name">914 - tax 13%</field>
      <field name="tag_name">914</field>
      <field name="sequence">8</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acquisitions_goods_tax"/>
      <field name="code">LUTAX_914</field>
    </record>

    <record id="account_tax_report_line_2b_tax_8" model="account.tax.report.line">
      <field name="name">716 - tax 8%</field>
      <field name="tag_name">716</field>
      <field name="sequence">9</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acquisitions_goods_tax"/>
      <field name="code">LUTAX_716</field>
    </record>

    <record id="account_tax_report_line_2b_tax_7" model="account.tax.report.line">
      <field name="name">916 - tax 7%</field>
      <field name="tag_name">916</field>
      <field name="sequence">10</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acquisitions_goods_tax"/>
      <field name="code">LUTAX_916</field>
    </record>

    <record id="account_tax_report_line_2b_tax_3" model="account.tax.report.line">
      <field name="name">054 - tax 3%</field>
      <field name="tag_name">054</field>
      <field name="sequence">11</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2b_intra_community_acquisitions_goods_tax"/>
      <field name="code">LUTAX_054</field>
    </record>

    <record id="account_tax_report_line_2d_importation_of_goods_tax" model="account.tax.report.line">
      <field name="name">407 - Importation of goods – tax</field>
      <field name="sequence">7</field>
      <field name="report_id" ref="tax_report"/>
        <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="code">LUTAX_407</field>
    </record>

    <record id="account_tax_report_line_2d_1_tax_17" model="account.tax.report.line">
      <field name="name">722 - for business purposes: tax 17%</field>
      <field name="tag_name">722</field>
      <field name="sequence">1</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_722</field>
    </record>

    <record id="account_tax_report_line_2d_1_tax_16" model="account.tax.report.line">
      <field name="name">922 - for business purposes: tax 16%</field>
      <field name="tag_name">922</field>
      <field name="sequence">2</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_922</field>
    </record>

    <record id="account_tax_report_line_2d_1_tax_14" model="account.tax.report.line">
      <field name="name">724 - for business purposes: tax 14%</field>
      <field name="tag_name">724</field>
      <field name="sequence">3</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_724</field>
    </record>

    <record id="account_tax_report_line_2d_1_tax_13" model="account.tax.report.line">
      <field name="name">924 - for business purposes: tax 13%</field>
      <field name="tag_name">924</field>
      <field name="sequence">4</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_924</field>
    </record>

    <record id="account_tax_report_line_2d_1_tax_8" model="account.tax.report.line">
      <field name="name">726 - for business purposes: tax 8%</field>
      <field name="tag_name">726</field>
      <field name="sequence">5</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_726</field>
    </record>

    <record id="account_tax_report_line_2d_1_tax_7" model="account.tax.report.line">
      <field name="name">926 - for business purposes: tax 7%</field>
      <field name="tag_name">926</field>
      <field name="sequence">6</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_926</field>
    </record>

    <record id="account_tax_report_line_2d_1_tax_3" model="account.tax.report.line">
      <field name="name">068 - for business purposes: tax 3%</field>
      <field name="tag_name">068</field>
      <field name="sequence">7</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_068</field>
    </record>

    <record id="account_tax_report_line_2d_2_tax_17" model="account.tax.report.line">
      <field name="name">732 - for non-business purposes: tax 17%</field>
      <field name="tag_name">732</field>
      <field name="sequence">8</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_732</field>
    </record>

    <record id="account_tax_report_line_2d_2_tax_16" model="account.tax.report.line">
      <field name="name">932 - for non-business purposes: tax 16%</field>
      <field name="tag_name">932</field>
      <field name="sequence">9</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_932</field>
    </record>

    <record id="account_tax_report_line_2d_2_tax_14" model="account.tax.report.line">
      <field name="name">734 - for non-business purposes: tax 14%</field>
      <field name="tag_name">734</field>
      <field name="sequence">10</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_734</field>
    </record>

    <record id="account_tax_report_line_2d_2_tax_13" model="account.tax.report.line">
      <field name="name">934 - for non-business purposes: tax 13%</field>
      <field name="tag_name">934</field>
      <field name="sequence">11</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_934</field>
    </record>

    <record id="account_tax_report_line_2d_2_tax_8" model="account.tax.report.line">
      <field name="name">736 - for non-business purposes: tax 8%</field>
      <field name="tag_name">736</field>
      <field name="sequence">12</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_736</field>
    </record>

    <record id="account_tax_report_line_2d_2_tax_7" model="account.tax.report.line">
      <field name="name">936 - for non-business purposes: tax 7%</field>
      <field name="tag_name">936</field>
      <field name="sequence">13</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_936</field>
    </record>

    <record id="account_tax_report_line_2d_2_tax_3" model="account.tax.report.line">
      <field name="name">073 - for non-business purposes: tax 3%</field>
      <field name="tag_name">073</field>
      <field name="sequence">14</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2d_importation_of_goods_tax"/>
      <field name="code">LUTAX_073</field>
    </record>

    <record id="account_tax_report_line_2e_supply_of_service_for_customer_liable_for_payment_tax" model="account.tax.report.line">
      <field name="name">410 - Supply of services for which the customer is liable for the payment of VAT – tax</field>
      <field name="sequence">9</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2_assesment_of_tax_due"/>
      <field name="code">LUTAX_410</field>
    </record>

    <record id="account_tax_report_line_2e_1_a_tax" model="account.tax.report.line">
      <field name="name">462 - tax</field>
      <field name="sequence">1</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2e_supply_of_service_for_customer_liable_for_payment_tax"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_tax_17" model="account.tax.report.line">
      <field name="name">742 - not exempt within the territory: tax 17%</field>
      <field name="tag_name">742</field>
      <field name="sequence">5</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_a_tax"/>
      <field name="code">LUTAX_742</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_tax_16" model="account.tax.report.line">
      <field name="name">942 - not exempt within the territory: tax 16%</field>
      <field name="tag_name">942</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_a_tax"/>
      <field name="code">LUTAX_942</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_tax_14" model="account.tax.report.line">
      <field name="name">744 - not exempt within the territory: tax 14%</field>
      <field name="tag_name">744</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_a_tax"/>
      <field name="code">LUTAX_744</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_tax_13" model="account.tax.report.line">
      <field name="name">944 - not exempt within the territory: tax 13%</field>
      <field name="tag_name">944</field>
      <field name="sequence">8</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_a_tax"/>
      <field name="code">LUTAX_944</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_tax_8" model="account.tax.report.line">
      <field name="name">746 - not exempt within the territory: tax 8%</field>
      <field name="tag_name">746</field>
      <field name="sequence">9</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_a_tax"/>
      <field name="code">LUTAX_746</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_tax_7" model="account.tax.report.line">
      <field name="name">946 - not exempt within the territory: tax 7%</field>
      <field name="tag_name">946</field>
      <field name="sequence">10</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_a_tax"/>
      <field name="code">LUTAX_946</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_1_a_tax_3" model="account.tax.report.line">
      <field name="name">432 - not exempt within the territory: tax 3%</field>
      <field name="tag_name">432</field>
      <field name="sequence">11</field>
      <field name="parent_id" ref="account_tax_report_line_2e_1_a_tax"/>
      <field name="code">LUTAX_432</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_2_tax" model="account.tax.report.line">
      <field name="name">464 - tax</field>
      <field name="sequence">2</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2e_supply_of_service_for_customer_liable_for_payment_tax"/>
    </record>

    <record id="account_tax_report_line_2e_2_tax_17" model="account.tax.report.line">
      <field name="name">752 - not established or residing within the Community: tax 17%</field>
      <field name="tag_name">752</field>
      <field name="sequence">5</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_tax"/>
      <field name="code">LUTAX_752</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_2_tax_16" model="account.tax.report.line">
      <field name="name">952 - not established or residing within the Community: tax 16%</field>
      <field name="tag_name">952</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_tax"/>
      <field name="code">LUTAX_952</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_2_tax_14" model="account.tax.report.line">
      <field name="name">754 - not established or residing within the Community: tax 14%</field>
      <field name="tag_name">754</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_tax"/>
      <field name="code">LUTAX_754</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_2_tax_13" model="account.tax.report.line">
      <field name="name">954 - not established or residing within the Community: tax 13%</field>
      <field name="tag_name">954</field>
      <field name="sequence">8</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_tax"/>
      <field name="code">LUTAX_954</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_2_tax_8" model="account.tax.report.line">
      <field name="name">756 - not established or residing within the Community: tax 8%</field>
      <field name="tag_name">756</field>
      <field name="sequence">9</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_tax"/>
      <field name="code">LUTAX_756</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_2_tax_7" model="account.tax.report.line">
      <field name="name">956 - not established or residing within the Community: tax 7%</field>
      <field name="tag_name">956</field>
      <field name="sequence">10</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_tax"/>
      <field name="code">LUTAX_956</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_2_tax_3" model="account.tax.report.line">
      <field name="name">442 - not established or residing within the Community: tax 3%</field>
      <field name="tag_name">442</field>
      <field name="sequence">11</field>
      <field name="parent_id" ref="account_tax_report_line_2e_2_tax"/>
      <field name="code">LUTAX_442</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_3_tax" model="account.tax.report.line">
      <field name="name">766 - tax</field>
      <field name="sequence">3</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_2e_supply_of_service_for_customer_liable_for_payment_tax"/>
    </record>

    <record id="account_tax_report_line_2e_3_tax_17" model="account.tax.report.line">
      <field name="name">762 - suppliers established within the territory: tax 17%</field>
      <field name="tag_name">762</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_2e_3_tax"/>
      <field name="code">LUTAX_762</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_2e_3_tax_16" model="account.tax.report.line">
      <field name="name">962 - suppliers established within the territory: tax 16%</field>
      <field name="tag_name">962</field>
      <field name="sequence">3</field>
      <field name="parent_id" ref="account_tax_report_line_2e_3_tax"/>
      <field name="code">LUTAX_962</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_4a_total_input_tax_deductible" model="account.tax.report.line">
      <field name="name">104 - Total input tax deductible</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_4_tax_tobe_paid_or_reclaimed"/>
      <field name="formula">LUTAX_102</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_3_assessment_deducible_tax" model="account.tax.report.line">
      <field name="name">III. ASSESSMENT OF DEDUCTIBLE TAX (input tax)</field>
      <field name="sequence">3</field>
      <field name="report_id" ref="tax_report"/>
      <field name="formula">None</field>
    </record>

    <record id="account_tax_report_line_3a_total_input_tax" model="account.tax.report.line">
      <field name="name">093 - Total input tax</field>
      <field name="sequence">1</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_3_assessment_deducible_tax"/>
    </record>

    <record id="account_tax_report_line_3b_total_input_tax_nd" model="account.tax.report.line">
      <field name="name">097 - Total input tax non-deductible</field>
      <field name="sequence">2</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_3_assessment_deducible_tax"/>
    </record>

    <record id="account_tax_report_line_3b1_rel_trans" model="account.tax.report.line">
      <field name="name">094 - relating to transactions which are exempt pursuant to articles 44 and 56quater</field>
      <field name="tag_name">094</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_3b_total_input_tax_nd"/>
      <field name="code">LUTAX_094</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_3b2_ded_prop" model="account.tax.report.line">
      <field name="name">095 - where the deductible proportion determined in accordance to article 50 is applied</field>
      <field name="tag_name">095</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_3b_total_input_tax_nd"/>
      <field name="code">LUTAX_095</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_3b2_input_tax_margin" model="account.tax.report.line">
      <field name="name">096 - Non recoverable input tax in accordance with Art. 56ter-1(7) and 56ter-2(7) (when applying the margin scheme)</field>
      <field name="tag_name">096</field>
      <field name="sequence">3</field>
      <field name="code">LUTAX_096</field>
      <field name="report_id" ref="tax_report"/>
      <field name="parent_id" ref="account_tax_report_line_3b_total_input_tax_nd"/>
    </record>

    <!-- Note: any change in formula here should be reflected in section `IV.B. Total input tax deductible (104)` above -->
    <record id="account_tax_report_line_3c_total_input_tax_deductible" model="account.tax.report.line">
      <field name="name">102 - Total input tax deductible</field>
      <field name="sequence">3</field>
      <field name="parent_id" ref="account_tax_report_line_3_assessment_deducible_tax"/>
      <field name="code">LUTAX_102</field>
      <field name="formula">LUTAX_458+LUTAX_459+LUTAX_460+LUTAX_090+LUTAX_461+LUTAX_092+LUTAX_228+LUTAX_094+LUTAX_095</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_3a_4_due_respect_application_goods" model="account.tax.report.line">
      <field name="name">090 - Due in respect of the application of goods for business purposes</field>
      <field name="tag_name">090</field>
      <field name="sequence">4</field>
      <field name="parent_id" ref="account_tax_report_line_3a_total_input_tax"/>
      <field name="code">LUTAX_090</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_3a_6_paid_joint_several_guarantee" model="account.tax.report.line">
      <field name="name">092 - Paid as joint and several guarantee</field>
      <field name="tag_name">092</field>
      <field name="sequence">6</field>
      <field name="parent_id" ref="account_tax_report_line_3a_total_input_tax"/>
      <field name="code">LUTAX_092</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_3a_7_adjusted_tax_special_arrangement" model="account.tax.report.line">
      <field name="name">228 - Adjusted tax - special arrangement for tax suspension</field>
      <field name="tag_name">228</field>
      <field name="sequence">7</field>
      <field name="parent_id" ref="account_tax_report_line_3a_total_input_tax"/>
      <field name="code">LUTAX_228</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_3a_1_invoiced_by_other_taxable_person" model="account.tax.report.line">
      <field name="name">458 - Invoiced by other taxable persons for goods or services supplied</field>
      <field name="tag_name">458</field>
      <field name="sequence">1</field>
      <field name="parent_id" ref="account_tax_report_line_3a_total_input_tax"/>
      <field name="code">LUTAX_458</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_3a_2_due_respect_intra_comm_goods" model="account.tax.report.line">
      <field name="name">459 - Due in respect of intra-Community acquisitions of goods</field>
      <field name="tag_name">459</field>
      <field name="sequence">2</field>
      <field name="parent_id" ref="account_tax_report_line_3a_total_input_tax"/>
      <field name="code">LUTAX_459</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_3a_3_due_paid_respect_importation_goods" model="account.tax.report.line">
      <field name="name">460 - Due or paid in respect of importation of goods</field>
      <field name="tag_name">460</field>
      <field name="sequence">3</field>
      <field name="parent_id" ref="account_tax_report_line_3a_total_input_tax"/>
      <field name="code">LUTAX_460</field>
      <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_3a_5_due_under_reverse_charge" model="account.tax.report.line">
      <field name="name">461 - Due under the reverse charge (see points II.E and F)</field>
      <field name="tag_name">461</field>
      <field name="sequence">5</field>
      <field name="parent_id" ref="account_tax_report_line_3a_total_input_tax"/>
      <field name="code">LUTAX_461</field>
      <field name="report_id" ref="tax_report"/>
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

	<record id="lu_2015_tax_AB-EC-16" model="account.tax.template">
		<field name="sequence">112</field>
        <field name="description">16%</field>
        <field name="name">16-EC-P-G</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_16')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-EC-13" model="account.tax.template">
		<field name="sequence">113</field>
        <field name="description">13%</field>
        <field name="name">13-EC-P-G</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_13')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-EC-7" model="account.tax.template">
		<field name="sequence">114</field>
        <field name="description">7%</field>
        <field name="name">7-EC-P-G</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_7')],
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

	<record id="lu_2015_tax_AB-ECP-16" model="account.tax.template">
		<field name="sequence">134</field>
        <field name="description">16%</field>
        <field name="name">16-EC(P)-P-G</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_16')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-ECP-13" model="account.tax.template">
		<field name="sequence">135</field>
        <field name="description">13%</field>
        <field name="name">13-EC(P)-P-G</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_13')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-ECP-7" model="account.tax.template">
		<field name="sequence">136</field>
        <field name="description">7%</field>
        <field name="name">7-EC(P)-P-G</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_7')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-ECP-3" model="account.tax.template">
		<field name="sequence">137</field>
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

	<record id="lu_2015_tax_AB-IC-16" model="account.tax.template">
		<field name="sequence">155</field>
        <field name="description">16%</field>
        <field name="name">16-IC-P-G</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_16')]
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_16')]
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-IC-13" model="account.tax.template">
		<field name="sequence">156</field>
        <field name="description">13%</field>
        <field name="name">13-IC-P-G</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_13')]
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_13')]
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AB-IC-7" model="account.tax.template">
		<field name="sequence">157</field>
        <field name="description">7%</field>
        <field name="name">7-IC-P-G</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_7')]
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_7')]
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

	<record id="lu_2015_tax_AB-PA-16" model="account.tax.template">
		<field name="sequence">102</field>
        <field name="description">16%</field>
        <field name="name">16-P-G</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>

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

	<record id="lu_2015_tax_AB-PA-13" model="account.tax.template">
		<field name="sequence">103</field>
        <field name="description">13%</field>
        <field name="name">13-P-G</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>

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

	<record id="lu_2015_tax_AB-PA-7" model="account.tax.template">
		<field name="sequence">104</field>
        <field name="description">7%</field>
        <field name="name">7-P-G</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>

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

	<record id="lu_2015_tax_AP-EC-16" model="account.tax.template">
		<field name="sequence">185</field>
        <field name="description">16%</field>
        <field name="name">16-EC-P-S</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_16')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AP-EC-13" model="account.tax.template">
		<field name="sequence">186</field>
        <field name="description">13%</field>
        <field name="name">13-EC-P-S</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_13')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AP-EC-7" model="account.tax.template">
		<field name="sequence">187</field>
        <field name="description">7%</field>
        <field name="name">7-EC-P-S</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_7')],
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

	<record id="lu_2015_tax_AP-IC-16" model="account.tax.template">
		<field name="sequence">208</field>
        <field name="description">16%</field>
        <field name="name">16-IC-P-S</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_16')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AP-IC-13" model="account.tax.template">
		<field name="sequence">209</field>
        <field name="description">13%</field>
        <field name="name">13-IC-P-S</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_13')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_AP-IC-7" model="account.tax.template">
		<field name="sequence">210</field>
        <field name="description">7%</field>
        <field name="name">7-IC-P-S</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_7')],
		    }),
		]"/>
	</record>

	<record id="lu_2011_tax_AP-IC-3" model="account.tax.template">
		<field name="sequence">211</field>
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

	<record id="lu_2015_tax_AP-PA-16" model="account.tax.template">
		<field name="sequence">223</field>
        <field name="description">16%</field>
        <field name="name">16-P-S</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
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

	<record id="lu_2015_tax_AP-PA-13" model="account.tax.template">
		<field name="sequence">223</field>
        <field name="description">13%</field>
        <field name="name">13-P-S</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
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

	<record id="lu_2015_tax_AP-PA-7" model="account.tax.template">
		<field name="sequence">223</field>
        <field name="description">7%</field>
        <field name="name">7-P-S</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
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

	<record id="lu_2015_tax_FB-EC-16" model="account.tax.template">
		<field name="sequence">238</field>
        <field name="description">16%</field>
        <field name="name">16-EC-E-G</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_16')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-EC-13" model="account.tax.template">
		<field name="sequence">239</field>
        <field name="description">13%</field>
        <field name="name">13-EC-E-G</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_13')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-EC-7" model="account.tax.template">
		<field name="sequence">240</field>
        <field name="description">7%</field>
        <field name="name">7-EC-E-G</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_7')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_FB-EC-3" model="account.tax.template">
		<field name="sequence">241</field>
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

	<record id="lu_2015_tax_FB-ECP-16" model="account.tax.template">
		<field name="sequence">260</field>
        <field name="description">16%</field>
        <field name="name">16-EC(P)-E-G</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_16')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-ECP-13" model="account.tax.template">
		<field name="sequence">261</field>
        <field name="description">13%</field>
        <field name="name">13-EC(P)-E-G</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_13')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-ECP-7" model="account.tax.template">
		<field name="sequence">262</field>
        <field name="description">7%</field>
        <field name="name">7-EC(P)-E-G</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_7')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-ECP-3" model="account.tax.template">
		<field name="sequence">263</field>
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

	<record id="lu_2015_tax_FB-IC-16" model="account.tax.template">
		<field name="sequence">282</field>
        <field name="description">16%</field>
        <field name="name">16-IC-E-G</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_16')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-IC-13" model="account.tax.template">
		<field name="sequence">283</field>
        <field name="description">13%</field>
        <field name="name">13-IC-E-G</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_13')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FB-IC-7" model="account.tax.template">
		<field name="sequence">284</field>
        <field name="description">7%</field>
        <field name="name">7-IC-E-G</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_7')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_FB-IC-3" model="account.tax.template">
		<field name="sequence">285</field>
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

	<record id="lu_2015_tax_FB-PA-16" model="account.tax.template">
		<field name="sequence">298</field>
        <field name="description">16%</field>
        <field name="name">16-E-G</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
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

	<record id="lu_2015_tax_FB-PA-13" model="account.tax.template">
		<field name="sequence">299</field>
        <field name="description">13%</field>
        <field name="name">13-E-G</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
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

	<record id="lu_2015_tax_FB-PA-7" model="account.tax.template">
		<field name="sequence">300</field>
        <field name="description">7%</field>
        <field name="name">7-E-G</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
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
		<field name="sequence">301</field>
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
		<field name="sequence">302</field>
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
		<field name="sequence">303</field>
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

	<record id="lu_2015_tax_FP-EC-16" model="account.tax.template">
		<field name="sequence">312</field>
        <field name="description">16%</field>
        <field name="name">16-EC-E-S</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_16')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FP-EC-13" model="account.tax.template">
		<field name="sequence">313</field>
        <field name="description">13%</field>
        <field name="name">13-EC-E-S</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_13')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FP-EC-7" model="account.tax.template">
		<field name="sequence">314</field>
        <field name="description">7%</field>
        <field name="name">7-EC-E-S</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_7')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_FP-EC-3" model="account.tax.template">
		<field name="sequence">315</field>
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

	<record id="lu_2015_tax_FP-IC-16" model="account.tax.template">
		<field name="sequence">334</field>
        <field name="description">16%</field>
        <field name="name">16-IC-E-S</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_16')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FP-IC-13" model="account.tax.template">
		<field name="sequence">335</field>
        <field name="description">13%</field>
        <field name="name">13-IC-E-S</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_13')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_FP-IC-7" model="account.tax.template">
		<field name="sequence">336</field>
        <field name="description">7%</field>
        <field name="name">7-IC-E-S</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_7')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_FP-IC-3" model="account.tax.template">
		<field name="sequence">337</field>
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

	<record id="lu_2015_tax_FP-PA-16" model="account.tax.template">
		<field name="sequence">350</field>
        <field name="description">16%</field>
        <field name="name">16-E-S</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
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

	<record id="lu_2015_tax_FP-PA-13" model="account.tax.template">
		<field name="sequence">351</field>
        <field name="description">13%</field>
        <field name="name">13-E-S</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
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

	<record id="lu_2015_tax_FP-PA-7" model="account.tax.template">
		<field name="sequence">352</field>
        <field name="description">7%</field>
        <field name="name">7-E-S</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
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
		<field name="sequence">353</field>
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
		<field name="sequence">354</field>
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
		<field name="sequence">355</field>
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

	<record id="lu_2015_tax_IB-EC-16" model="account.tax.template">
		<field name="sequence">364</field>
        <field name="description">16%</field>
        <field name="name">16-EC-IG</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_16')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-EC-13" model="account.tax.template">
		<field name="sequence">365</field>
        <field name="description">13%</field>
        <field name="name">13-EC-IG</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_13')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-EC-7" model="account.tax.template">
		<field name="sequence">366</field>
        <field name="description">7%</field>
        <field name="name">7-EC-IG</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_1_tax_7')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_IB-EC-3" model="account.tax.template">
		<field name="sequence">367</field>
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

	<record id="lu_2015_tax_IB-ECP-16" model="account.tax.template">
		<field name="sequence">386</field>
        <field name="description">16%</field>
        <field name="name">16-EC(P)-IG</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_16')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-ECP-13" model="account.tax.template">
		<field name="sequence">387</field>
        <field name="description">13%</field>
        <field name="name">13-EC(P)-IG</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_13')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-ECP-7" model="account.tax.template">
		<field name="sequence">388</field>
        <field name="description">7%</field>
        <field name="name">7-EC(P)-IG</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2d_2_tax_7')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-ECP-3" model="account.tax.template">
		<field name="sequence">389</field>
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

	<record id="lu_2015_tax_IB-IC-16" model="account.tax.template">
		<field name="sequence">407</field>
        <field name="description">16%</field>
        <field name="name">16-IC-IG</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_16')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-IC-13" model="account.tax.template">
		<field name="sequence">408</field>
        <field name="description">13%</field>
        <field name="name">13-IC-IG</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_13')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IB-IC-7" model="account.tax.template">
		<field name="sequence">409</field>
        <field name="description">7%</field>
        <field name="name">7-IC-IG</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2b_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2b_tax_7')],
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

	<record id="lu_2015_tax_IB-PA-16" model="account.tax.template">
		<field name="sequence">424</field>
        <field name="description">16%</field>
        <field name="name">16-IG</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
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

	<record id="lu_2015_tax_IB-PA-13" model="account.tax.template">
		<field name="sequence">425</field>
        <field name="description">13%</field>
        <field name="name">13-IG</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
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

	<record id="lu_2015_tax_IB-PA-7" model="account.tax.template">
		<field name="sequence">426</field>
        <field name="description">7%</field>
        <field name="name">7-IG</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
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
		<field name="sequence">427</field>
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
		<field name="sequence">428</field>
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
		<field name="sequence">429</field>
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

	<record id="lu_2015_tax_IP-EC-16" model="account.tax.template">
		<field name="sequence">438</field>
        <field name="description">16%</field>
        <field name="name">16-EC-IS</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_16')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IP-EC-13" model="account.tax.template">
		<field name="sequence">439</field>
        <field name="description">13%</field>
        <field name="name">13-EC-IS</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_13')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IP-EC-7" model="account.tax.template">
		<field name="sequence">440</field>
        <field name="description">7%</field>
        <field name="name">7-EC-IS</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_2_tax_7')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2011_tax_IP-EC-3" model="account.tax.template">
		<field name="sequence">441</field>
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

	<record id="lu_2015_tax_IP-IC-16" model="account.tax.template">
		<field name="sequence">459</field>
        <field name="description">16%</field>
        <field name="name">16-IC-IS</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_16')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_16')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_16')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IP-IC-13" model="account.tax.template">
		<field name="sequence">460</field>
        <field name="description">13%</field>
        <field name="name">13-IC-IS</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_13')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_13')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_13')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_IP-IC-7" model="account.tax.template">
		<field name="sequence">461</field>
        <field name="description">7%</field>
        <field name="name">7-IC-IS</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_7')],
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
		        'plus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_base_7')],
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
		        'minus_report_line_ids': [ref('account_tax_report_line_2e_1_a_tax_7')],
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

	<record id="lu_2015_tax_IP-PA-16" model="account.tax.template">
		<field name="sequence">476</field>
        <field name="description">16%</field>
        <field name="name">16-IS</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
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

	<record id="lu_2015_tax_IP-PA-13" model="account.tax.template">
		<field name="sequence">477</field>
        <field name="description">13%</field>
        <field name="name">13-IS</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
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

	<record id="lu_2015_tax_IP-PA-7" model="account.tax.template">
		<field name="sequence">478</field>
        <field name="description">7%</field>
        <field name="name">7-IS</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
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
		<field name="sequence">479</field>
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
		<field name="sequence">480</field>
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

	<record id="lu_2015_tax_VB-PA-16" model="account.tax.template">
		<field name="sequence">503</field>
        <field name="description">16%</field>
        <field name="name">16-S-G</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_16')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_16')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_16')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_VB-PA-13" model="account.tax.template">
		<field name="sequence">504</field>
        <field name="description">13%</field>
        <field name="name">13-S-G</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_13')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_13')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_13')],
		    }),
		]"/>
		<field name="active" eval="False"/>
	</record>

	<record id="lu_2015_tax_VB-PA-7" model="account.tax.template">
		<field name="sequence">505</field>
        <field name="description">7%</field>
        <field name="name">7-S-G</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_7')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_7')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_7')],
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

	<record id="lu_2015_tax_VP-PA-16" model="account.tax.template">
		<field name="sequence">480</field>
        <field name="description">16%</field>
        <field name="name">16-S-S</field>
        <field name="amount">16</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_16')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_16')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_16')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_VP-PA-13" model="account.tax.template">
		<field name="sequence">481</field>
        <field name="description">13%</field>
        <field name="name">13-S-S</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_13"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_13')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_13')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_13')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_13')],
		    }),
		]"/>
	</record>

	<record id="lu_2015_tax_VP-PA-7" model="account.tax.template">
		<field name="sequence">482</field>
        <field name="description">7%</field>
        <field name="name">7-S-S</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="lu_2011_chart_1"/>
        <field name="tax_group_id" ref="tax_group_7"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_base_7')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_7')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_base_7')],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_7')],
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

	<record id="lu_2015_tax_ATN_sale_16" model="account.tax.template">
		<field name="sequence">511</field>
		<field name="description">16%</field>
		<field name="name">16-ATN</field>
		<field name="amount">16</field>
		<field name="amount_type">percent</field>
		<field name="type_tax_use">sale</field>
		<field name="chart_template_id" ref="lu_2011_chart_1"/>
		<field name="tax_group_id" ref="tax_group_16"/>
		<field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'plus_report_line_ids': [
		            ref('account_tax_report_line_1a_non_bus_gs'),
		            ref('account_tax_report_line_2a_base_16'),
		        ],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'plus_report_line_ids': [ref('account_tax_report_line_2a_tax_16')],
		    }),
		]"/>
		<field name="refund_repartition_line_ids" eval="[(5, 0, 0),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'base',
		        'minus_report_line_ids': [
		            ref('account_tax_report_line_1a_non_bus_gs'),
		            ref('account_tax_report_line_2a_base_16'),
		        ],
		    }),
		    (0,0, {
		        'factor_percent': 100,
		        'repartition_type': 'tax',
		        'account_id': ref('lu_2020_account_461411'),
		        'minus_report_line_ids': [ref('account_tax_report_line_2a_tax_16')],
		    }),
		]"/>
	</record>
</odoo>

```

## File: data\l10n_lu_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <menuitem id="account_reports_lu_statements_menu" name="Luxembourg" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>

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
"Project-Id-Version: Odoo Server saas~13.3\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2020-06-19 13:53+0000\n"
"PO-Revision-Date: 2020-06-19 13:53+0000\n"
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
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1a_overall_turnover
msgid "012 - Overall turnover"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_2_export
msgid "014"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_2_export
msgid "014 - Exports"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_3_other_exemptions_art_43
msgid "015"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_3_other_exemptions_art_43
msgid "015 - Other exemptions"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_4_other_exemptions_art_44_et_56quater
msgid "016"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_4_other_exemptions_art_44_et_56quater
msgid "016 - Other exemptions"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_5_manufactured_tobacco_vat_collected
msgid "017"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_5_manufactured_tobacco_vat_collected
msgid ""
"017 - Manufactured tobacco whose VAT was collected at the source or at the "
"exit of the tax..."
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_6_a_subsequent_to_intra_community
msgid "018"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_6_a_subsequent_to_intra_community
msgid ""
"018 - Supply, subsequent to intra-Community acquisitions of goods, in the "
"context of triangular transactions, when the customer identified,..."
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_6_d_supplies_other_referred
msgid "019"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_6_d_supplies_other_referred
msgid "019 - Supplies other than referred to in 018 and 423 or 424"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_exemptions_deductible_amounts
msgid "021 - Exemptions and deductible amounts"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1c_taxable_turnover
msgid "022 - Taxable turnover"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_base_3
msgid "031"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_base_3
msgid "031 - base 3%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_base_0
msgid "033"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_base_0
msgid "033 - base 0%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_breakdown_taxable_turnover_base
msgid "037 - Breakdown of taxable turnover – base"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_tax_3
msgid "040"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_tax_3
msgid "040 - tax 3%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_breakdown_taxable_turnover_tax
msgid "046 - Breakdown of taxable turnover – tax"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_base_3
msgid "049"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_base_3
msgid "049 - base 3%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_intra_community_acqui_of_goods_base
msgid "051 - Intra-Community acquisitions of goods – base"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_tax_3
msgid "054"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_tax_3
msgid "054 - tax 3%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_intra_community_acquisitions_goods_tax
msgid "056 - Intra-Community acquisitions of goods – tax"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_base_3
msgid "059"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_3
msgid "059 - for business purposes: base 3%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_base_3
msgid "063"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_3
msgid "063 - for non-business purposes: base 3%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_importation_of_goods_base
msgid "065 - Importation of goods – base"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_tax_3
msgid "068"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_3
msgid "068 - for business purposes: tax 3%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_tax_3
msgid "073"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_3
msgid "073 - for non-business purposes: tax 3%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2h_total_tax_due
msgid "076 - Total tax due"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3a_4_due_respect_application_goods
msgid "090"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_4_due_respect_application_goods
msgid "090 - Due in respect of the application of goods for business purposes"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3a_6_paid_joint_several_guarantee
msgid "092"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_6_paid_joint_several_guarantee
msgid "092 - Paid as joint and several guarantee"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_total_input_tax
msgid "093 - Total input tax"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3b1_rel_trans
msgid ""
"094 - relating to transactions which are exempt pursuant to articles 44 and "
"56quater"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3b2_ded_prop
msgid ""
"095 - where the deductible proportion determined in accordance to article 50"
" is applied"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3b2_input_tax_margin
msgid "096"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3b2_input_tax_margin
msgid ""
"096 - Non recoverable input tax in accordance with Art. 56ter-1(7) and "
"56ter-2(7) (when applying the margin scheme)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3b_total_input_tax_nd
msgid "097 - Total input tax non-deductible"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3c_total_input_tax_deductible
msgid "102 - Total input tax deductible"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_4a_total_tax_due
msgid "103 - Total tax due"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_4a_total_input_tax_deductible
msgid "104 - Total input tax deductible"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_4c_exceeding_amount
msgid "105 - Exceeding amount"
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
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2c_acquisitions_triangular_transactions_base
msgid "152"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2c_acquisitions_triangular_transactions_base
msgid "152 - Acquisitions, in the context of triangular transactions – base"
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
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_base_exempt
msgid "194"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_base_exempt
msgid "194 - base exempt"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_base_exempt
msgid "195"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_exempt
msgid "195 - for business purposes: base exempt"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_base_exempt
msgid "196"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_exempt
msgid "196 - for non-business purposes: base exempt"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_6_c_supplies_scope_special_arrangement
msgid "226"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_6_c_supplies_scope_special_arrangement
msgid ""
"226 - Supplies carried out within the scope of the special arrangement of "
"art. 56sexies"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2g_special_arrangement
msgid "227"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2g_special_arrangement
msgid "227 - Special arrangement for tax suspension: adjustment"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3a_7_adjusted_tax_special_arrangement
msgid "228"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_7_adjusted_tax_special_arrangement
msgid "228 - Adjusted tax - special arrangement for tax suspension"
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
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_importation_of_goods_tax
msgid "407 - Importation of goods – tax"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_supply_of_service_for_customer
msgid ""
"409 - Supply of services for which the customer is liable for the payment of"
" VAT – base"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_supply_of_service_for_customer_liable_for_payment_tax
msgid ""
"410 - Supply of services for which the customer is liable for the payment of"
" VAT – tax"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_7_inland_supplies_for_customer
msgid "419"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_7_inland_supplies_for_customer
msgid ""
"419 - Inland supplies for which the customer is liable for the payment of "
"VAT"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_6_b1_non_exempt_customer_vat
msgid "423"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_6_b1_non_exempt_customer_vat
msgid ""
"423 - not exempt in the MS where the customer is liable for payment of VAT"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_6_b2_exempt_ms_customer
msgid "424"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_6_b2_exempt_ms_customer
msgid "424 - exempt in the MS where the customer is identified"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_base_3
msgid "431"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_3
msgid "431 - not exempt within the territory: base 3%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_tax_3
msgid "432"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_3
msgid "432 - not exempt within the territory: tax 3%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_b_exempt
msgid "435"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_b_exempt
msgid "435 - exempt within the territory: exempt"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_base
msgid "436 - base"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_base_3
msgid "441"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_3
msgid "441 - not established or residing within the Community: base 3%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_tax_3
msgid "442"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_3
msgid "442 - not established or residing within the Community: tax 3%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_exempt
msgid "445"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_exempt
msgid "445 - not established or residing within the Community: exempt"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1a_total_sale
msgid "454 - Total Sales / Receipts"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1a_app_goods_non_bus
msgid "455"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1a_app_goods_non_bus
msgid ""
"455 - Application of goods for non-business use and for business purposes"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1a_non_bus_gs
msgid "456"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1a_non_bus_gs
msgid "456 - Non-business use of goods and supply of services free of charge"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1b_1_intra_community_goods_pi_vat
msgid "457"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1b_1_intra_community_goods_pi_vat
msgid ""
"457 - Intra-Community supply of goods to persons identified for VAT purposes"
" in another Member State (MS)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3a_1_invoiced_by_other_taxable_person
msgid "458"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_1_invoiced_by_other_taxable_person
msgid "458 - Invoiced by other taxable persons for goods or services supplied"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3a_2_due_respect_intra_comm_goods
msgid "459"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_2_due_respect_intra_comm_goods
msgid "459 - Due in respect of intra-Community acquisitions of goods"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3a_3_due_paid_respect_importation_goods
msgid "460"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_3_due_paid_respect_importation_goods
msgid "460 - Due or paid in respect of importation of goods"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_3a_5_due_under_reverse_charge
msgid "461"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3a_5_due_under_reverse_charge
msgid "461 - Due under the reverse charge (see points II.E and F)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax
msgid "462 - tax"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_base
msgid "463 - base"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax
msgid "464 - tax"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_1a_telecom_service
msgid "471"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1a_telecom_service
msgid ""
"471 - Telecommunications services, radio and television broadcasting "
"services..."
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_1a_other_sales
msgid "472 - Other sales / receipts"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_base_17
msgid "701"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_base_17
msgid "701 - base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_tax_17
msgid "702"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_tax_17
msgid "702 - tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_base_14
msgid "703"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_base_14
msgid "703 - base 14%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_tax_14
msgid "704"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_tax_14
msgid "704 - tax 14%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_base_8
msgid "705"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_base_8
msgid "705 - base 8%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2a_tax_8
msgid "706"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_tax_8
msgid "706 - tax 8%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_base_17
msgid "711"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_base_17
msgid "711 - base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_tax_17
msgid "712"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_tax_17
msgid "712 - tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_base_14
msgid "713"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_base_14
msgid "713 - base 14%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_tax_14
msgid "714"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_tax_14
msgid "714 - tax 14%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_base_8
msgid "715"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_base_8
msgid "715 - base 8%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_tax_8
msgid "716"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_tax_8
msgid "716 - tax 8%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2b_manufactured_tobacco
msgid "719"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_manufactured_tobacco
msgid ""
"719 - of manufactured tobacco (VAT is collected at the exit of the tax "
"warehouse with excise duties)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_base_17
msgid "721"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_17
msgid "721 - for business purposes: base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_tax_17
msgid "722"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_17
msgid "722 - for business purposes: tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_base_14
msgid "723"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_14
msgid "723 - for business purposes: base 14%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_tax_14
msgid "724"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_14
msgid "724 - for business purposes: tax 14%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_base_8
msgid "725"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_8
msgid "725 - for business purposes: base 8%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_tax_8
msgid "726"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_8
msgid "726 - for business purposes: tax 8%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_1_manufactured_tobacco
msgid "729"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_manufactured_tobacco
msgid ""
"729 - of manufactured tobacco (VAT is collected at the exit of the tax "
"warehouse with excise duties)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_base_17
msgid "731"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_17
msgid "731 - for non-business purposes: base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_tax_17
msgid "732"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_17
msgid "732 - for non-business purposes: tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_base_14
msgid "733"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_14
msgid "733 - for non-business purposes: base 14%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_tax_14
msgid "734"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_14
msgid "734 - for non-business purposes: tax 14%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_base_8
msgid "735"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_8
msgid "735 - for non-business purposes: base 8%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2d_2_tax_8
msgid "736"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_8
msgid "736 - for non-business purposes: tax 8%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_base_17
msgid "741"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_17
msgid "741 - not exempt within the territory: base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_tax_17
msgid "742"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_17
msgid "742 - not exempt within the territory: tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_base_14
msgid "743"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_14
msgid "743 - not exempt within the territory: base 14%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_tax_14
msgid "744"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_14
msgid "744 - not exempt within the territory: tax 14%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_base_8
msgid "745"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_8
msgid "745 - not exempt within the territory: base 8%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_1_a_tax_8
msgid "746"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_8
msgid "746 - not exempt within the territory: tax 8%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_base_17
msgid "751"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_17
msgid "751 - not established or residing within the Community: base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_tax_17
msgid "752"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_17
msgid "752 - not established or residing within the Community: tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_base_14
msgid "753"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_14
msgid "753 - not established or residing within the Community: base 14%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_tax_14
msgid "754"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_14
msgid "754 - not established or residing within the Community: tax 14%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_base_8
msgid "755"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_8
msgid "755 - not established or residing within the Community: base 8%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_2_tax_8
msgid "756"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_8
msgid "756 - not established or residing within the Community: tax 8%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_3_base_17
msgid "761"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_3_base_17
msgid "761 - suppliers established within the territory: base 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2e_3_tax_17
msgid "762"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_3_tax_17
msgid "762 - suppliers established within the territory: tax 17%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2f_supply_goods_base_8
msgid "763"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_base_8
msgid "763 - base 8%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,tag_name:l10n_lu.account_tax_report_line_2f_supply_goods_tax_8
msgid "764"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_tax_8
msgid "764 - tax 8%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_3_base
msgid "765 - base"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_3_tax
msgid "766 - tax"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_base
msgid ""
"767 - Supply of goods for which the purchaser is liable for the payment of "
"VAT - base"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_tax
msgid ""
"768 - Supply of goods for which the purchaser is liable for the payment of "
"VAT - tax"
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
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_16
msgid "921 - for business purposes: base 16%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_13
msgid "923 - for business purposes: base 13%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_base_7
msgid "925 - for business purposes: base 7%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_16
msgid "931 - for non-business purposes: base 16%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_13
msgid "933 - for non-business purposes: base 13%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_base_7
msgid "935 - for non-business purposes: base 7%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_16
msgid "941 - not exempt within the territory: base 16%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_13
msgid "943 - not exempt within the territory: base 13%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_base_7
msgid "945 - not exempt within the territory: base 7%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_16
msgid "951 - not established or residing within the Community: base 16%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_13
msgid "953 - not established or residing within the Community: base 13%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_base_7
msgid "955 - not established or residing within the Community: base 7%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_3_base_16
msgid "961 - suppliers established within the territory: base 16%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_base_7
msgid "963 - base 7%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2f_supply_goods_tax_7
msgid "964 - tax 7%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_base_16
msgid "901 - base 16%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_base_13
msgid "903 - base 13%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_base_7
msgid "905 - base 7%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_tax_16
msgid "902 - tax 16%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_tax_13
msgid "904 - tax 13%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2a_tax_7
msgid "906 - tax 7%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_base_16
msgid "911 - base 16%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_base_13
msgid "913 - base 13%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_base_7
msgid "915 - base 7%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_tax_16
msgid "912 - tax 16%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_tax_13
msgid "914 - tax 13%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2b_tax_7
msgid "916 - tax 7%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_16
msgid "922 - for business purposes: tax 16%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_13
msgid "924 - for business purposes: tax 13%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_1_tax_7
msgid "926 - for business purposes: tax 7%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_16
msgid "932 - for non-business purposes: tax 16%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_13
msgid "934 - for non-business purposes: tax 13%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2d_2_tax_7
msgid "936 - for non-business purposes: tax 7%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_16
msgid "942 - not exempt within the territory: tax 16%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_13
msgid "944 - not exempt within the territory: tax 13%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_1_a_tax_7
msgid "946 - not exempt within the territory: tax 7%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_16
msgid "952 - not established or residing within the Community: tax 16%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_13
msgid "954 - not established or residing within the Community: tax 13%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_2_tax_7
msgid "956 - not established or residing within the Community: tax 7%"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2e_3_tax_16
msgid "962 - suppliers established within the territory: tax 16%"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_46128
#: model:account.group.template,name:l10n_lu.account_group_46128
msgid "ACD - Other amounts payable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_42148
#: model:account.group.template,name:l10n_lu.account_group_42148
msgid "ACD - Other amounts receivable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_46148
#: model:account.group.template,name:l10n_lu.account_group_46148
msgid "AED - Other debts"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_635
msgid "AVA and FVA on receivables from current assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65112
#: model:account.group.template,name:l10n_lu.account_group_65112
msgid "AVA on amounts owed by affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6352
#: model:account.group.template,name:l10n_lu.account_group_6352
msgid ""
"AVA on amounts owed by affiliated undertakings and undertakings with which "
"the undertaking is linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65114
#: model:account.group.template,name:l10n_lu.account_group_65114
msgid ""
"AVA on amounts owed by undertakings with which the undertaking is linked by "
"virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_63313
#: model:account.group.template,name:l10n_lu.account_group_63313
msgid "AVA on buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6322
#: model:account.group.template,name:l10n_lu.account_group_6322
msgid ""
"AVA on concessions, patents, licences, trademarks and similar rights and "
"assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6321
#: model:account.group.template,name:l10n_lu.account_group_6321
msgid "AVA on development costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6324
#: model:account.group.template,name:l10n_lu.account_group_6324
msgid "AVA on down payments and intangible fixed assets under development"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6334
#: model:account.group.template,name:l10n_lu.account_group_6334
msgid "AVA on down payments and tangible fixed assets under development"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6345
#: model:account.group.template,name:l10n_lu.account_group_6345
msgid "AVA on down payments on inventories"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6313
#: model:account.group.template,name:l10n_lu.account_group_6313
msgid ""
"AVA on expenses for capital increases and various operations (mergers, "
"demergers, changes of legal form)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6511
msgid "AVA on financial fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_63314
#: model:account.group.template,name:l10n_lu.account_group_63314
msgid "AVA on fixtures and fittings-out of buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_63312
#: model:account.group.template,name:l10n_lu.account_group_63312
msgid "AVA on fixtures and fittings-out of land"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_631
msgid "AVA on formation expenses and similar expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6323
#: model:account.group.template,name:l10n_lu.account_group_6323
msgid "AVA on goodwill acquired for consideration"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_632
msgid "AVA on intangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_634
msgid "AVA on inventories"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6343
#: model:account.group.template,name:l10n_lu.account_group_6343
msgid "AVA on inventories of goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6344
#: model:account.group.template,name:l10n_lu.account_group_6344
msgid "AVA on inventories of merchandise and other goods for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6341
#: model:account.group.template,name:l10n_lu.account_group_6341
msgid "AVA on inventories of raw materials and consumables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6342
#: model:account.group.template,name:l10n_lu.account_group_6342
msgid "AVA on inventories of work and contracts in progress"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_63311
#: model:account.group.template,name:l10n_lu.account_group_63311
msgid "AVA on land"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6331
msgid ""
"AVA on land, fittings-out and buildings and FVA on investment properties"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6314
#: model:account.group.template,name:l10n_lu.account_group_6314
msgid "AVA on loan-issuance expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65116
#: model:account.group.template,name:l10n_lu.account_group_65116
msgid "AVA on loans, deposits and claims held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6333
#: model:account.group.template,name:l10n_lu.account_group_6333
msgid ""
"AVA on other fixtures and fittings, tools and equipment (including rolling "
"stock)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6353
#: model:account.group.template,name:l10n_lu.account_group_6353
msgid "AVA on other receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6318
#: model:account.group.template,name:l10n_lu.account_group_6318
msgid "AVA on other similar expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65318
#: model:account.group.template,name:l10n_lu.account_group_65318
msgid "AVA on other transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65312
#: model:account.group.template,name:l10n_lu.account_group_65312
msgid "AVA on own shares or own corporate units"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65113
#: model:account.group.template,name:l10n_lu.account_group_65113
msgid "AVA on participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6332
#: model:account.group.template,name:l10n_lu.account_group_6332
msgid "AVA on plant and machinery"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65115
#: model:account.group.template,name:l10n_lu.account_group_65115
msgid "AVA on securities held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6311
#: model:account.group.template,name:l10n_lu.account_group_6311
msgid "AVA on set-up and start-up costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65111
#: model:account.account.template,name:l10n_lu.lu_2011_account_65311
#: model:account.group.template,name:l10n_lu.account_group_65111
#: model:account.group.template,name:l10n_lu.account_group_65311
msgid "AVA on shares in affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65313
#: model:account.group.template,name:l10n_lu.account_group_65313
msgid ""
"AVA on shares in undertakings with which the undertaking is linked by virtue"
" of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_633
msgid ""
"AVA on tangible fixed assets and fair value adjustments (FVA) on investment "
"properties"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6351
#: model:account.group.template,name:l10n_lu.account_group_6351
msgid "AVA on trade receivables"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6531
msgid "AVA on transferable securities"
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
#: model:account.group.template,name:l10n_lu.account_group_106
msgid "Account of the owner or the co-owners"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61342
#: model:account.group.template,name:l10n_lu.account_group_61342
msgid "Accounting, tax consulting, auditing and similar fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_16121
msgid "Acquired against payment (except Goodwill)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_771
#: model:account.group.template,name:l10n_lu.account_group_771
msgid "Adjustments of corporate income tax (CIT)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_773
#: model:account.group.template,name:l10n_lu.account_group_773
msgid "Adjustments of foreign income taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_783
#: model:account.group.template,name:l10n_lu.account_group_783
msgid "Adjustments of foreign taxes"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_77
msgid "Adjustments of income taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_772
#: model:account.group.template,name:l10n_lu.account_group_772
msgid "Adjustments of municipal business tax (MBT)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_781
#: model:account.group.template,name:l10n_lu.account_group_781
msgid "Adjustments of net wealth tax (NWT)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_788
#: model:account.group.template,name:l10n_lu.account_group_788
msgid "Adjustments of other taxes"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_78
msgid "Adjustments of other taxes not included in the previous caption"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_782
#: model:account.group.template,name:l10n_lu.account_group_782
msgid "Adjustments of subscription tax"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42111
#: model:account.group.template,name:l10n_lu.account_group_42111
msgid "Advances and down payments"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_659
msgid "Allocations to financial provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6591
#: model:account.group.template,name:l10n_lu.account_group_6591
msgid "Allocations to financial provisions - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6592
#: model:account.group.template,name:l10n_lu.account_group_6592
msgid "Allocations to financial provisions - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6492
#: model:account.group.template,name:l10n_lu.account_group_6492
msgid "Allocations to operating provisions"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_649
msgid "Allocations to provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_679
#: model:account.group.template,name:l10n_lu.account_group_679
msgid "Allocations to provisions for deferred taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6491
#: model:account.group.template,name:l10n_lu.account_group_6491
msgid "Allocations to tax provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_647
#: model:account.group.template,name:l10n_lu.account_group_647
msgid "Allocations to tax-exempt capital gains"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_653
msgid ""
"Allocations to value adjustment (AVA) and fair-value adjustments (FVA) on "
"transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_63
msgid ""
"Allocations to value adjustments (AVA) and fair value adjustments (FVA) on "
"formation expenses, intangible, tangible and current assets (except "
"transferable securities)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_651
msgid ""
"Allocations to value adjustments (AVA) and fair-value adjustments (FVA) of "
"financial fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_232
#: model:account.account.template,name:l10n_lu.lu_2011_account_6452
#: model:account.account.template,name:l10n_lu.lu_2011_account_75112
#: model:account.account.template,name:l10n_lu.lu_2020_account_65212
#: model:account.account.template,name:l10n_lu.lu_2020_account_75212
#: model:account.group.template,name:l10n_lu.account_group_232
#: model:account.group.template,name:l10n_lu.account_group_411
#: model:account.group.template,name:l10n_lu.account_group_6452
#: model:account.group.template,name:l10n_lu.account_group_65212
#: model:account.group.template,name:l10n_lu.account_group_75212
#: model:account.group.template,name:l10n_lu.account_group_75222
msgid "Amounts owed by affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_41
msgid ""
"Amounts owed by affiliated undertakings and by undertakings with which the "
"undertaking is linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4112
msgid ""
"Amounts owed by affiliated undertakings receivable after more than one year"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4111
msgid "Amounts owed by affiliated undertakings receivable within one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_4212
#: model:account.account.template,name:l10n_lu.lu_2020_account_4222
#: model:account.group.template,name:l10n_lu.account_group_4212
#: model:account.group.template,name:l10n_lu.account_group_4222
msgid ""
"Amounts owed by partners and shareholders (others than from affiliated "
"undertakings)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4217
msgid "Amounts owed by the Social Security and other social bodies"
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
#: model:account.group.template,name:l10n_lu.account_group_234
#: model:account.group.template,name:l10n_lu.account_group_412
#: model:account.group.template,name:l10n_lu.account_group_6453
#: model:account.group.template,name:l10n_lu.account_group_65214
#: model:account.group.template,name:l10n_lu.account_group_75214
#: model:account.group.template,name:l10n_lu.account_group_75224
msgid ""
"Amounts owed by undertakings with which the undertaking is linked by virtue "
"of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4121
msgid ""
"Amounts owed by undertakings with which the undertaking is linked by virtue "
"of participating interests receivable within one year"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_194
msgid "Amounts owed to credit institutions"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_451
msgid "Amounts payable to affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4512
msgid "Amounts payable to affiliated undertakings after more than one year"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_45
msgid ""
"Amounts payable to affiliated undertakings and to undertakings with which "
"the undertaking is linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4511
msgid "Amounts payable to affiliated undertakings within one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4713
#: model:account.account.template,name:l10n_lu.lu_2011_account_4723
#: model:account.group.template,name:l10n_lu.account_group_4713
#: model:account.group.template,name:l10n_lu.account_group_4723
msgid "Amounts payable to directors, managers, statutory auditors and similar"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_4712
#: model:account.account.template,name:l10n_lu.lu_2020_account_4722
#: model:account.group.template,name:l10n_lu.account_group_4712
#: model:account.group.template,name:l10n_lu.account_group_4722
msgid ""
"Amounts payable to partners and shareholders (others than from affiliated "
"undertakings)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_4714
#: model:account.account.template,name:l10n_lu.lu_2020_account_4724
#: model:account.group.template,name:l10n_lu.account_group_4714
#: model:account.group.template,name:l10n_lu.account_group_4724
msgid "Amounts payable to staff"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_452
msgid ""
"Amounts payable to undertakings with which the undertaking is linked by "
"virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4522
msgid ""
"Amounts payable to undertakings with which the undertaking is linked by "
"virtue of participating interests payable after more than one year"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4521
msgid ""
"Amounts payable to undertakings with which the undertaking is linked by "
"virtue of participating interests within one year"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4122
msgid "Amounts receivable after more than one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_60813
#: model:account.group.template,name:l10n_lu.account_group_60813
msgid "Architects' and engineers' fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6431
#: model:account.group.template,name:l10n_lu.account_group_6431
msgid "Attendance fees"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_643
msgid "Attendance fees, director's fees and similar remuneration"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_743
#: model:account.group.template,name:l10n_lu.account_group_743
msgid "Attendance fees, director's fees and similar remunerations"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61333
#: model:account.group.template,name:l10n_lu.account_group_61333
msgid ""
"Bank account charges and bank commissions (included custody fees on "
"securities)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7552
msgid "Bank and similar interest"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6552
msgid "Banking and similar interest"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6133
msgid "Banking and similar services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65521
#: model:account.group.template,name:l10n_lu.account_group_65521
msgid "Banking interest on current accounts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65522
#: model:account.group.template,name:l10n_lu.account_group_65522
msgid "Banking interest on financing operations"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_5131
#: model:account.group.template,name:l10n_lu.account_group_5131
msgid "Banks and CCP : available balance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_5132
#: model:account.group.template,name:l10n_lu.account_group_5132
msgid "Banks and CCP : overdraft"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_513
msgid "Banks and postal cheques accounts (CCP)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6467
#: model:account.group.template,name:l10n_lu.account_group_6467
msgid "Bar licence tax"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_62111
#: model:account.group.template,name:l10n_lu.account_group_62111
msgid "Base wages"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_62115
#: model:account.account.template,name:l10n_lu.lu_2011_account_746
#: model:account.group.template,name:l10n_lu.account_group_62115
#: model:account.group.template,name:l10n_lu.account_group_746
msgid "Benefits in kind"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_442
msgid "Bills of exchange payable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4422
#: model:account.group.template,name:l10n_lu.account_group_4422
msgid "Bills of exchange payable after more than one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4421
#: model:account.group.template,name:l10n_lu.account_group_4421
msgid "Bills of exchange payable within one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652221
#: model:account.account.template,name:l10n_lu.lu_2020_account_752221
#: model:account.group.template,name:l10n_lu.account_group_652221
#: model:account.group.template,name:l10n_lu.account_group_752221
msgid "Book value of yielded amounts owed by affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_752241
msgid ""
"Book value of yielded amounts owed by undertakings with which the "
"undertaking is linked by virtue of participating  interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652241
#: model:account.account.template,name:l10n_lu.lu_2020_account_752241
#: model:account.group.template,name:l10n_lu.account_group_652241
msgid ""
"Book value of yielded amounts owed by undertakings with which the "
"undertaking is linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_64411
#: model:account.account.template,name:l10n_lu.lu_2020_account_74411
#: model:account.group.template,name:l10n_lu.account_group_64411
#: model:account.group.template,name:l10n_lu.account_group_74411
msgid "Book value of yielded intangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652261
#: model:account.account.template,name:l10n_lu.lu_2020_account_752261
#: model:account.group.template,name:l10n_lu.account_group_652261
#: model:account.group.template,name:l10n_lu.account_group_752261
msgid "Book value of yielded loans, deposits and claims held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652231
#: model:account.account.template,name:l10n_lu.lu_2020_account_752231
#: model:account.group.template,name:l10n_lu.account_group_652231
#: model:account.group.template,name:l10n_lu.account_group_752231
msgid "Book value of yielded participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652251
#: model:account.account.template,name:l10n_lu.lu_2020_account_752251
#: model:account.group.template,name:l10n_lu.account_group_652251
#: model:account.group.template,name:l10n_lu.account_group_752251
msgid "Book value of yielded securities held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652211
#: model:account.account.template,name:l10n_lu.lu_2020_account_752211
#: model:account.group.template,name:l10n_lu.account_group_652211
#: model:account.group.template,name:l10n_lu.account_group_752211
msgid "Book value of yielded shares in affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_64421
#: model:account.account.template,name:l10n_lu.lu_2020_account_74421
#: model:account.group.template,name:l10n_lu.account_group_64421
#: model:account.group.template,name:l10n_lu.account_group_74421
msgid "Book value of yielded tangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61112
#: model:account.account.template,name:l10n_lu.lu_2011_account_61221
#: model:account.account.template,name:l10n_lu.lu_2011_account_61411
#: model:account.group.template,name:l10n_lu.account_group_2213
#: model:account.group.template,name:l10n_lu.account_group_61112
#: model:account.group.template,name:l10n_lu.account_group_61221
#: model:account.group.template,name:l10n_lu.account_group_61411
msgid "Buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_60763
#: model:account.group.template,name:l10n_lu.account_group_60763
msgid "Buildings for resale"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_22131
msgid "Buildings in Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_22132
#: model:account.group.template,name:l10n_lu.account_group_22132
msgid "Buildings in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_314
#: model:account.group.template,name:l10n_lu.account_group_314
msgid "Buildings under construction"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61523
#: model:account.group.template,name:l10n_lu.account_group_61523
msgid "Business assignments"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6144
#: model:account.group.template,name:l10n_lu.account_group_6144
msgid "Business risk insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10629
msgid "Business share in private expenses"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6
msgid "CHARGES ACCOUNTS"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461212
#: model:account.group.template,name:l10n_lu.account_group_461212
msgid "CIT - Tax payable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6711
#: model:account.group.template,name:l10n_lu.account_group_6711
msgid "CIT - current financial year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6712
#: model:account.group.template,name:l10n_lu.account_group_6712
msgid "CIT - previous financial years"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_115
#: model:account.group.template,name:l10n_lu.account_group_115
msgid "Capital contribution without issue of shares"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7473
#: model:account.group.template,name:l10n_lu.account_group_16
#: model:account.group.template,name:l10n_lu.account_group_7473
msgid "Capital investment subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_104
#: model:account.group.template,name:l10n_lu.account_group_104
msgid "Capital of individual companies, corporate partnerships and similar"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_72
msgid "Capitalised production"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106166
msgid "Car"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_51
msgid "Cash at bank, in postal cheques accounts, cheques and in hand"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_516
#: model:account.group.template,name:l10n_lu.account_group_516
msgid "Cash in hand"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10611
msgid "Cash withdrawals (daily life)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6222
#: model:account.group.template,name:l10n_lu.account_group_6222
msgid "Casual workers"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61515
#: model:account.group.template,name:l10n_lu.account_group_61515
msgid "Catalogues, printed materials and publications"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7121
#: model:account.group.template,name:l10n_lu.account_group_7121
msgid "Change in inventories of finished goods"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_712
msgid "Change in inventories of goods"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_71
msgid "Change in inventories of goods and of work in progress"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7123
#: model:account.group.template,name:l10n_lu.account_group_7123
msgid "Change in inventories of residual goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7122
#: model:account.group.template,name:l10n_lu.account_group_7122
msgid "Change in inventories of semi-finished goods"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_711
msgid "Change in inventories of work and contracts in progress"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7111
#: model:account.group.template,name:l10n_lu.account_group_7111
msgid "Change in inventories of work in progress"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7114
#: model:account.group.template,name:l10n_lu.account_group_7114
msgid "Change in inventories: buildings under construction"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7112
#: model:account.group.template,name:l10n_lu.account_group_7112
msgid "Change in inventories: contracts in progress - goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7113
#: model:account.group.template,name:l10n_lu.account_group_7113
msgid "Change in inventories: contracts in progress - services"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_607
msgid "Changes in inventory"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6073
#: model:account.group.template,name:l10n_lu.account_group_6073
msgid "Changes in inventory of consumable materials and supplies"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6076
msgid "Changes in inventory of merchandise and other goods for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6074
#: model:account.group.template,name:l10n_lu.account_group_6074
msgid "Changes in inventory of packaging"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6071
#: model:account.group.template,name:l10n_lu.account_group_6071
msgid "Changes in inventory of raw materials"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_62412
#: model:account.group.template,name:l10n_lu.account_group_62412
msgid "Changes to provisions for complementary pensions"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_652
msgid "Charges and loss of disposal of financial fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_61334
msgid "Charges for electronic means of paiment"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61334
msgid "Charges for electronic means of payment"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6521
msgid "Charges of financial fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106152
msgid "Child benefit office"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6165
#: model:account.group.template,name:l10n_lu.account_group_6165
msgid "Collective staff transportation"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6131
#: model:account.account.template,name:l10n_lu.lu_2020_account_705
#: model:account.group.template,name:l10n_lu.account_group_6131
#: model:account.group.template,name:l10n_lu.account_group_705
msgid "Commissions and brokerage fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7453
#: model:account.group.template,name:l10n_lu.account_group_7453
msgid "Compensatory allowances"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6241
msgid "Complementary pensions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_62415
#: model:account.group.template,name:l10n_lu.account_group_62415
msgid "Complementary pensions paid by the employer"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2235
#: model:account.group.template,name:l10n_lu.account_group_2235
msgid "Computer equipment"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_21211
#: model:account.account.template,name:l10n_lu.lu_2011_account_21221
#: model:account.account.template,name:l10n_lu.lu_2011_account_6411
#: model:account.account.template,name:l10n_lu.lu_2011_account_72121
#: model:account.account.template,name:l10n_lu.lu_2011_account_7411
#: model:account.account.template,name:l10n_lu.lu_2020_account_70311
#: model:account.group.template,name:l10n_lu.account_group_21211
#: model:account.group.template,name:l10n_lu.account_group_21221
#: model:account.group.template,name:l10n_lu.account_group_6411
#: model:account.group.template,name:l10n_lu.account_group_70311
#: model:account.group.template,name:l10n_lu.account_group_72121
#: model:account.group.template,name:l10n_lu.account_group_7411
msgid "Concessions"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_1612
#: model:account.group.template,name:l10n_lu.account_group_212
#: model:account.group.template,name:l10n_lu.account_group_7212
msgid ""
"Concessions, patents, licences, trademarks and similar rights and assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_312
#: model:account.group.template,name:l10n_lu.account_group_312
msgid "Contracts in progress - goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_313
#: model:account.group.template,name:l10n_lu.account_group_313
msgid "Contracts in progress - services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_113
#: model:account.group.template,name:l10n_lu.account_group_113
msgid "Contribution premium"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6187
#: model:account.group.template,name:l10n_lu.account_group_6187
msgid "Contributions to professional associations"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_192
msgid "Convertible debenture loans"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_212151
#: model:account.account.template,name:l10n_lu.lu_2011_account_212251
#: model:account.account.template,name:l10n_lu.lu_2011_account_64151
#: model:account.account.template,name:l10n_lu.lu_2011_account_721251
#: model:account.account.template,name:l10n_lu.lu_2011_account_74151
#: model:account.account.template,name:l10n_lu.lu_2020_account_703151
#: model:account.group.template,name:l10n_lu.account_group_212151
#: model:account.group.template,name:l10n_lu.account_group_212251
#: model:account.group.template,name:l10n_lu.account_group_64151
#: model:account.group.template,name:l10n_lu.account_group_703151
#: model:account.group.template,name:l10n_lu.account_group_721251
#: model:account.group.template,name:l10n_lu.account_group_74151
msgid "Copyrights and reproduction rights"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_42141
#: model:account.group.template,name:l10n_lu.account_group_42141
msgid "Corporate income tax"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_46121
#: model:account.group.template,name:l10n_lu.account_group_671
msgid "Corporate income tax (CIT)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461211
#: model:account.group.template,name:l10n_lu.account_group_461211
msgid "Corporate income tax - Tax accrual"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6182
#: model:account.group.template,name:l10n_lu.account_group_6182
msgid "Costs of training, symposiums, seminars, conferences"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_16122
msgid "Created by the undertaking itself"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_67321
#: model:account.group.template,name:l10n_lu.account_group_67321
msgid "Current financial year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4011
#: model:account.account.template,name:l10n_lu.lu_2011_account_4021
#: model:account.group.template,name:l10n_lu.account_group_4011
#: model:account.group.template,name:l10n_lu.account_group_4021
msgid "Customers"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_40111
msgid "Customers (PoS)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4012
#: model:account.group.template,name:l10n_lu.account_group_4012
msgid "Customers - Receivable bills of exchange"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4014
#: model:account.group.template,name:l10n_lu.account_group_4014
msgid "Customers - Unbilled sales"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6145
#: model:account.group.template,name:l10n_lu.account_group_6145
msgid "Customers credit insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4015
#: model:account.group.template,name:l10n_lu.account_group_4015
msgid "Customers with a credit balance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4025
#: model:account.group.template,name:l10n_lu.account_group_4025
msgid "Customers with creditor balance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4215
#: model:account.account.template,name:l10n_lu.lu_2020_account_4613
#: model:account.group.template,name:l10n_lu.account_group_4215
#: model:account.group.template,name:l10n_lu.account_group_4613
msgid "Customs and Excise Authority (ADA)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4
msgid "DEBTORS AND CREDITORS"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106154
msgid "Death and other health insurance funds"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_19
msgid "Debenture loans and amounts owed to credit institutions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_5083
#: model:account.group.template,name:l10n_lu.account_group_5083
msgid "Debenture loans and other notes issued and repurchased by the company"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_23521
#: model:account.group.template,name:l10n_lu.account_group_23521
msgid "Debentures"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_481
#: model:account.group.template,name:l10n_lu.account_group_481
msgid "Deferred charges (on one or more financial years)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_48
msgid "Deferred charges and income"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_482
#: model:account.group.template,name:l10n_lu.account_group_482
msgid "Deferred income (on one or more financial years)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_183
#: model:account.group.template,name:l10n_lu.account_group_183
msgid "Deferred tax provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_2362
#: model:account.group.template,name:l10n_lu.account_group_2362
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
#: model:account.group.template,name:l10n_lu.account_group_42187
#: model:account.group.template,name:l10n_lu.account_group_42287
#: model:account.group.template,name:l10n_lu.account_group_4717
#: model:account.group.template,name:l10n_lu.account_group_4727
msgid "Derivative financial instruments"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_221111
#: model:account.group.template,name:l10n_lu.account_group_221111
msgid "Developed land"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_211
#: model:account.account.template,name:l10n_lu.lu_2011_account_7211
#: model:account.account.template,name:l10n_lu.lu_2020_account_1611
#: model:account.group.template,name:l10n_lu.account_group_1611
#: model:account.group.template,name:l10n_lu.account_group_211
#: model:account.group.template,name:l10n_lu.account_group_7211
msgid "Development costs"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4214
#: model:account.group.template,name:l10n_lu.account_group_4612
msgid "Direct Tax Authority (ACD)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6432
#: model:account.group.template,name:l10n_lu.account_group_6432
msgid "Director's fees"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6555
msgid "Discounts and charges on bills of exchange"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65551
#: model:account.group.template,name:l10n_lu.account_group_65551
msgid "Discounts and charges on bills of exchange - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65552
#: model:account.group.template,name:l10n_lu.account_group_65552
msgid "Discounts and charges on bills of exchange - other"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7555
msgid "Discounts on bills of exchange"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75551
#: model:account.group.template,name:l10n_lu.account_group_75551
msgid "Discounts on bills of exchange - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75552
#: model:account.group.template,name:l10n_lu.account_group_75552
msgid "Discounts on bills of exchange - other"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7556
msgid "Discounts received"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75561
#: model:account.group.template,name:l10n_lu.account_group_75561
msgid "Discounts received - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75562
#: model:account.group.template,name:l10n_lu.account_group_75562
msgid "Discounts received - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_752262
#: model:account.group.template,name:l10n_lu.account_group_752262
msgid "Disposal proceed of loans, deposits and claims held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652222
#: model:account.account.template,name:l10n_lu.lu_2020_account_752222
#: model:account.group.template,name:l10n_lu.account_group_652222
#: model:account.group.template,name:l10n_lu.account_group_752222
msgid "Disposal proceeds of amounts owed by affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652242
#: model:account.account.template,name:l10n_lu.lu_2020_account_752242
#: model:account.group.template,name:l10n_lu.account_group_652242
#: model:account.group.template,name:l10n_lu.account_group_752242
msgid ""
"Disposal proceeds of amounts owed by undertakings with which the undertaking"
" is linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_64412
#: model:account.account.template,name:l10n_lu.lu_2020_account_74412
#: model:account.group.template,name:l10n_lu.account_group_64412
#: model:account.group.template,name:l10n_lu.account_group_74412
msgid "Disposal proceeds of intangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652262
#: model:account.group.template,name:l10n_lu.account_group_652262
msgid "Disposal proceeds of loans, deposits and claims held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652232
#: model:account.account.template,name:l10n_lu.lu_2020_account_752232
#: model:account.group.template,name:l10n_lu.account_group_652232
#: model:account.group.template,name:l10n_lu.account_group_752232
msgid "Disposal proceeds of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652252
#: model:account.account.template,name:l10n_lu.lu_2020_account_752252
#: model:account.group.template,name:l10n_lu.account_group_652252
#: model:account.group.template,name:l10n_lu.account_group_752252
msgid "Disposal proceeds of securities held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_652212
#: model:account.account.template,name:l10n_lu.lu_2020_account_752212
#: model:account.group.template,name:l10n_lu.account_group_652212
#: model:account.group.template,name:l10n_lu.account_group_752212
msgid "Disposal proceeds of shares in affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_64422
#: model:account.account.template,name:l10n_lu.lu_2020_account_74422
#: model:account.group.template,name:l10n_lu.account_group_64422
#: model:account.group.template,name:l10n_lu.account_group_74422
msgid "Disposal proceeds of tangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6181
#: model:account.group.template,name:l10n_lu.account_group_6181
msgid "Documentation"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61516
#: model:account.group.template,name:l10n_lu.account_group_61516
msgid "Donations"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4013
#: model:account.group.template,name:l10n_lu.account_group_4013
msgid "Doubtful or disputed customers"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_214
#: model:account.group.template,name:l10n_lu.account_group_214
msgid "Down payments and intangible fixed assets under development"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_224
msgid "Down payments and tangible fixed assets under development"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_37
#: model:account.group.template,name:l10n_lu.account_group_37
msgid "Down payments on account on inventories"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_432
#: model:account.group.template,name:l10n_lu.account_group_432
msgid "Down payments received after more than one year"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_315
msgid ""
"Down payments received on inventories of work and on contracts in progress"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4311
#: model:account.group.template,name:l10n_lu.account_group_4321
msgid "Down payments received on orders"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_43
msgid ""
"Down payments received on orders as far as they are not deducted distinctly "
"from inventories"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_431
#: model:account.group.template,name:l10n_lu.account_group_431
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
#: model:account.group.template,name:l10n_lu.account_group_6463
msgid "Duties on imported merchandise"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_1
msgid "EQUITY, PROVISIONS AND FINANCIAL LIABILITIES ACCOUNTS"
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
#: model:account.group.template,name:l10n_lu.account_group_60315
#: model:account.group.template,name:l10n_lu.account_group_61845
msgid "Electricity"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_105
#: model:account.group.template,name:l10n_lu.account_group_105
msgid "Endowment of branches"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6464
#: model:account.group.template,name:l10n_lu.account_group_6464
msgid "Excise duties on production and tax on consumption"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_203
#: model:account.group.template,name:l10n_lu.account_group_203
msgid ""
"Expenses for increases in capital and for various operations (merger, "
"demerger, change of legal form)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_617
msgid "External staff of the company"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6172
#: model:account.group.template,name:l10n_lu.account_group_6172
msgid "External staff on secondment"
msgstr ""

#. module: l10n_lu
#: model:account.fiscal.position,name:l10n_lu.1_account_fiscal_position_template_LU_EC
#: model:account.fiscal.position.template,name:l10n_lu.account_fiscal_position_template_LU_EC
msgid "Extra-Community Taxable Person"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_5
msgid "FINANCIAL ACCOUNTS"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_2
msgid "FORMATION EXPENSES AND FIXED ASSETS ACCOUNTS"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6512
#: model:account.account.template,name:l10n_lu.lu_2011_account_7512
#: model:account.group.template,name:l10n_lu.account_group_6512
#: model:account.group.template,name:l10n_lu.account_group_7512
msgid "FVA on financial fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_63315
#: model:account.account.template,name:l10n_lu.lu_2020_account_73315
#: model:account.group.template,name:l10n_lu.account_group_63315
#: model:account.group.template,name:l10n_lu.account_group_73315
msgid "FVA on investment properties"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6354
#: model:account.account.template,name:l10n_lu.lu_2020_account_7354
#: model:account.group.template,name:l10n_lu.account_group_6354
#: model:account.group.template,name:l10n_lu.account_group_7354
msgid "FVA on receivables from current assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6532
#: model:account.group.template,name:l10n_lu.account_group_6532
#: model:account.group.template,name:l10n_lu.account_group_7532
msgid "FVA on transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61336
#: model:account.group.template,name:l10n_lu.account_group_61336
msgid "Factoring services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7532
msgid "Fair value adjustments on transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61513
#: model:account.group.template,name:l10n_lu.account_group_61513
msgid "Fairs and exhibitions"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_641
#: model:account.group.template,name:l10n_lu.account_group_7031
msgid ""
"Fees and royalties for concessions, patents, licences, trademarks and "
"similar rights and assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_741
msgid ""
"Fees and royalties for concessions, patents, licences, trademarks and "
"similar rights and assets from ancillary activities"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_65
msgid "Financial charges"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_23
msgid "Financial fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_75
msgid "Financial income"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6115
msgid "Financial leasing on movable property"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6114
#: model:account.group.template,name:l10n_lu.account_group_6114
msgid "Financial leasing on real property"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_1882
#: model:account.group.template,name:l10n_lu.account_group_1882
msgid "Financial provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6481
#: model:account.group.template,name:l10n_lu.account_group_6481
msgid "Fines, sanctions and penalties"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106143
msgid "Fire insurance"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_2214
msgid "Fixtures and fitting-outs of buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_22141
#: model:account.group.template,name:l10n_lu.account_group_22141
msgid "Fixtures and fitting-outs of buildings in Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_22142
#: model:account.group.template,name:l10n_lu.account_group_22142
msgid "Fixtures and fitting-outs of buildings in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_22121
#: model:account.group.template,name:l10n_lu.account_group_22121
msgid "Fixtures and fitting-outs of land in Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_22122
#: model:account.group.template,name:l10n_lu.account_group_22122
msgid "Fixtures and fitting-outs of land in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_2212
msgid "Fixtures and fittings-out of land"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4622
#: model:account.group.template,name:l10n_lu.account_group_4622
msgid "Foreign Social Security offices"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421811
#: model:account.account.template,name:l10n_lu.lu_2020_account_46151
#: model:account.group.template,name:l10n_lu.account_group_421811
#: model:account.group.template,name:l10n_lu.account_group_46151
msgid "Foreign VAT"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_756
msgid "Foreign currency exchange gains"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7561
#: model:account.group.template,name:l10n_lu.account_group_7561
msgid "Foreign currency exchange gains - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7562
#: model:account.group.template,name:l10n_lu.account_group_7562
msgid "Foreign currency exchange gains - other"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_656
msgid "Foreign currency exchange losses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6561
#: model:account.group.template,name:l10n_lu.account_group_6561
msgid "Foreign currency exchange losses - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6562
#: model:account.group.template,name:l10n_lu.account_group_6562
msgid "Foreign currency exchange losses - other"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_673
msgid "Foreign income taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42172
#: model:account.group.template,name:l10n_lu.account_group_42172
msgid "Foreign social security offices"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4615
msgid "Foreign tax authorities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_683
#: model:account.group.template,name:l10n_lu.account_group_42181
#: model:account.group.template,name:l10n_lu.account_group_683
msgid "Foreign taxes"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_20
msgid "Formation expenses and similar expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65411
#: model:account.account.template,name:l10n_lu.lu_2020_account_755231
#: model:account.group.template,name:l10n_lu.account_group_65411
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
#: model:account.group.template,name:l10n_lu.account_group_65412
msgid ""
"From undertakings with which the undertaking is linked by virtue of "
"participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6031
msgid "Fuels, gas, water and electricity"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6184
msgid ""
"Fuels, gas, water and electricity (not included in the production of goods "
"and services)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106145
msgid "Full coverage insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2234
#: model:account.group.template,name:l10n_lu.account_group_2234
msgid "Furniture"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_754
msgid ""
"Gain on disposal and other income from current receivables and transferable "
"securities of current assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7522
msgid "Gain on disposal of financial fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_744
msgid "Gain on disposal of intangible and tangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7441
msgid "Gain on disposal of intangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7541
msgid "Gain on disposal of receivables from current assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7542
msgid "Gain on disposal of transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_60313
#: model:account.account.template,name:l10n_lu.lu_2020_account_61843
#: model:account.group.template,name:l10n_lu.account_group_60313
#: model:account.group.template,name:l10n_lu.account_group_61843
msgid "Gas"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6121
#: model:account.group.template,name:l10n_lu.account_group_6121
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
#: model:account.group.template,name:l10n_lu.account_group_61514
msgid "Gifts to customers"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_213
#: model:account.account.template,name:l10n_lu.lu_2020_account_1613
#: model:account.group.template,name:l10n_lu.account_group_1613
#: model:account.group.template,name:l10n_lu.account_group_213
msgid "Goodwill acquired for consideration"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6556
msgid "Granted discounts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65561
#: model:account.group.template,name:l10n_lu.account_group_65561
msgid "Granted discounts - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65562
#: model:account.group.template,name:l10n_lu.account_group_65562
msgid "Granted discounts - other"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_212152
msgid "Greenhous gas and similar emission quotas"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_212152
msgid "Greenhouse gas and similar emission quotas"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6211
msgid "Gross wages"
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
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_2_assesment_of_tax_due
msgid "II. ASSESSMENT OF TAX DUE (output tax)"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_3_assessment_deducible_tax
msgid "III. ASSESSMENT OF DEDUCTIBLE TAX (input tax)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7
msgid "INCOME ACCOUNTS"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_3
msgid "INVENTORIES ACCOUNTS"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6132
#: model:account.group.template,name:l10n_lu.account_group_6132
msgid "IT services"
msgstr ""

#. module: l10n_lu
#: model:account.tax.report.line,name:l10n_lu.account_tax_report_line_4_tax_tobe_paid_or_reclaimed
msgid "IV. TAX TO BE PAID OR TO BE RECLAIMED"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_62114
#: model:account.group.template,name:l10n_lu.account_group_62114
msgid "Incentives, bonuses and commissions"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_752
msgid "Income and gain on disposal of financial fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7521
msgid "Income from financial fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7442
msgid "Income of yielded tangible fixed assets"
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
#: model:account.group.template,name:l10n_lu.account_group_67
msgid "Income taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_642
#: model:account.group.template,name:l10n_lu.account_group_642
msgid "Indemnities, damages and interest"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4216
msgid "Indirect Tax Authority (AED)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4614
msgid "Indirect tax authorities (AED)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_42162
#: model:account.group.template,name:l10n_lu.account_group_46142
msgid "Indirect taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6183
#: model:account.group.template,name:l10n_lu.account_group_6183
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
#: model:account.group.template,name:l10n_lu.account_group_62414
msgid "Insolvency insurance premiums"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6141
msgid "Insurance for assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7481
#: model:account.group.template,name:l10n_lu.account_group_7481
msgid "Insurance indemnities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6142
#: model:account.group.template,name:l10n_lu.account_group_6142
msgid "Insurance on rented assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_614
msgid "Insurance premiums"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_21
#: model:account.group.template,name:l10n_lu.account_group_721
msgid "Intangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_655
msgid "Interest and discounts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75541
#: model:account.group.template,name:l10n_lu.account_group_75541
msgid "Interest on amounts owed by affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7554
msgid ""
"Interest on amounts owed by affiliated undertakings and undertakings with "
"which the undertaking is linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75542
#: model:account.group.template,name:l10n_lu.account_group_75542
msgid ""
"Interest on amounts owed by undertakings with which the undertaking is "
"linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_75521
#: model:account.group.template,name:l10n_lu.account_group_75521
msgid "Interest on bank accounts"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6551
msgid "Interest on debenture loans"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65511
#: model:account.group.template,name:l10n_lu.account_group_65511
msgid "Interest on debenture loans - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_65512
#: model:account.group.template,name:l10n_lu.account_group_65512
msgid "Interest on debenture loans - other"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_65523
#: model:account.group.template,name:l10n_lu.account_group_75523
msgid "Interest on financial leases"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_655231
#: model:account.group.template,name:l10n_lu.account_group_655231
msgid "Interest on financial leases - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_655232
#: model:account.group.template,name:l10n_lu.account_group_655232
msgid "Interest on financial leases - other"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7558
msgid "Interest on other amounts receivable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75581
#: model:account.group.template,name:l10n_lu.account_group_75581
msgid "Interest on other amounts receivable - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_75582
#: model:account.group.template,name:l10n_lu.account_group_75582
msgid "Interest on other amounts receivable - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6553
#: model:account.group.template,name:l10n_lu.account_group_6553
msgid "Interest on trade payables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7553
#: model:account.group.template,name:l10n_lu.account_group_7553
msgid "Interest on trade receivables"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6558
msgid "Interest payable on other loans and debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65581
#: model:account.group.template,name:l10n_lu.account_group_65581
msgid "Interest payable on other loans and debts - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65582
#: model:account.group.template,name:l10n_lu.account_group_65582
msgid "Interest payable on other loans and debts - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65541
#: model:account.group.template,name:l10n_lu.account_group_65541
msgid "Interest payable to affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6554
msgid ""
"Interest payable to affiliated undertakings and undertakings with which the "
"undertaking is linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_65542
#: model:account.group.template,name:l10n_lu.account_group_65542
msgid ""
"Interest payable to undertakings with which the undertaking is linked by "
"virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7452
#: model:account.group.template,name:l10n_lu.account_group_7452
msgid "Interest subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_15
#: model:account.group.template,name:l10n_lu.account_group_15
msgid "Interim dividends"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_517
msgid "Internal transfers"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_5172
#: model:account.group.template,name:l10n_lu.account_group_5172
msgid "Internal transfers : credit balance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_5171
#: model:account.group.template,name:l10n_lu.account_group_5171
msgid "Internal transfers : debit balance"
msgstr ""

#. module: l10n_lu
#: model:account.fiscal.position,name:l10n_lu.1_account_fiscal_position_template_LU_IC
#: model:account.fiscal.position.template,name:l10n_lu.account_fiscal_position_template_LU_IC
msgid "Intra-Community Taxable Person"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_363
msgid "Inventories of buildings for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_3631
#: model:account.group.template,name:l10n_lu.account_group_3631
msgid "Inventories of buildings for resale in Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_3632
#: model:account.group.template,name:l10n_lu.account_group_3632
msgid "Inventories of buildings for resale in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_303
#: model:account.group.template,name:l10n_lu.account_group_303
msgid "Inventories of consumable materials and supplies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_321
#: model:account.group.template,name:l10n_lu.account_group_321
msgid "Inventories of finished goods"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_32
msgid "Inventories of goods"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_362
msgid "Inventories of land for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_3621
#: model:account.group.template,name:l10n_lu.account_group_3621
msgid "Inventories of land for resale in Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_3622
#: model:account.group.template,name:l10n_lu.account_group_3622
msgid "Inventories of land for resale in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_361
#: model:account.group.template,name:l10n_lu.account_group_361
msgid "Inventories of merchandise"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_36
msgid "Inventories of merchandises and other goods for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_304
#: model:account.group.template,name:l10n_lu.account_group_304
msgid "Inventories of packaging"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_301
#: model:account.group.template,name:l10n_lu.account_group_301
msgid "Inventories of raw materials"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_30
msgid "Inventories of raw materials and consumables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_323
#: model:account.group.template,name:l10n_lu.account_group_323
msgid ""
"Inventories of residual goods (waste, rejected and recuperable material)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_322
#: model:account.group.template,name:l10n_lu.account_group_322
msgid "Inventories of semi-finished goods"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_31
msgid "Inventories of work and contracts in progress"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4312
#: model:account.group.template,name:l10n_lu.account_group_4322
msgid ""
"Inventories of work and contracts in progress less down payments received"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_311
#: model:account.group.template,name:l10n_lu.account_group_311
msgid "Inventories of work in progress"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_2215
msgid "Investment properties"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_22151
#: model:account.group.template,name:l10n_lu.account_group_22151
msgid "Investment properties in Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_22152
#: model:account.group.template,name:l10n_lu.account_group_22152
msgid "Investment properties in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42131
#: model:account.account.template,name:l10n_lu.lu_2011_account_42231
#: model:account.group.template,name:l10n_lu.account_group_42131
#: model:account.group.template,name:l10n_lu.account_group_42231
msgid "Investment subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_42171
#: model:account.group.template,name:l10n_lu.account_group_4621
msgid "Joint Social Security Centre (CCSS)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61111
#: model:account.group.template,name:l10n_lu.account_group_2211
#: model:account.group.template,name:l10n_lu.account_group_61111
msgid "Land"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_60762
#: model:account.group.template,name:l10n_lu.account_group_60762
msgid "Land for resale"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_22111
msgid "Land in Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_22112
#: model:account.group.template,name:l10n_lu.account_group_22112
msgid "Land in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_2241
msgid "Land, fitting-outs and buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_22411
#: model:account.group.template,name:l10n_lu.account_group_22411
msgid "Land, fitting-outs and buildings in Luxembourg"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_22412
#: model:account.group.template,name:l10n_lu.account_group_22412
msgid "Land, fitting-outs and buildings in foreign countries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7221
#: model:account.group.template,name:l10n_lu.account_group_7221
msgid "Land, fittings and buildings"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_221
msgid "Land, fixtures and fitting-outs and buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_47162
#: model:account.account.template,name:l10n_lu.lu_2020_account_47262
#: model:account.group.template,name:l10n_lu.account_group_47162
#: model:account.group.template,name:l10n_lu.account_group_47262
msgid "Lease debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_131
#: model:account.group.template,name:l10n_lu.account_group_131
msgid "Legal reserve"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61341
#: model:account.group.template,name:l10n_lu.account_group_61341
msgid "Legal, litigation and similar fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_47163
#: model:account.account.template,name:l10n_lu.lu_2020_account_47263
#: model:account.group.template,name:l10n_lu.account_group_47163
#: model:account.group.template,name:l10n_lu.account_group_47263
msgid "Life annuities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106141
msgid "Life insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_486
#: model:account.group.template,name:l10n_lu.account_group_486
msgid "Linking accounts (branches) - Assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_487
#: model:account.group.template,name:l10n_lu.account_group_487
msgid "Linking accounts (branches) - Liabilities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_60312
#: model:account.group.template,name:l10n_lu.account_group_60312
msgid "Liquid fuels"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_61842
#: model:account.group.template,name:l10n_lu.account_group_61842
msgid "Liquid fuels (oil, motor fuel, etc.)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_chart_1_liquidity_transfer
msgid "Liquidity Transfer"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_5084
#: model:account.group.template,name:l10n_lu.account_group_5084
msgid "Listed debenture loans"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_235111
#: model:account.group.template,name:l10n_lu.account_group_235111
msgid "Listed shares"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2236
#: model:account.group.template,name:l10n_lu.account_group_2236
msgid "Livestock"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_204
#: model:account.group.template,name:l10n_lu.account_group_204
msgid "Loan issuances expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_2361
#: model:account.group.template,name:l10n_lu.account_group_2361
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
#: model:account.group.template,name:l10n_lu.account_group_41112
#: model:account.group.template,name:l10n_lu.account_group_41122
#: model:account.group.template,name:l10n_lu.account_group_41212
#: model:account.group.template,name:l10n_lu.account_group_41222
#: model:account.group.template,name:l10n_lu.account_group_45112
#: model:account.group.template,name:l10n_lu.account_group_45122
#: model:account.group.template,name:l10n_lu.account_group_45212
#: model:account.group.template,name:l10n_lu.account_group_45222
msgid "Loans and advances"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4716
#: model:account.group.template,name:l10n_lu.account_group_4726
msgid "Loans and similar debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61332
#: model:account.group.template,name:l10n_lu.account_group_61332
msgid "Loans' issuance expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_75116
#: model:account.account.template,name:l10n_lu.lu_2020_account_65216
#: model:account.account.template,name:l10n_lu.lu_2020_account_75216
#: model:account.group.template,name:l10n_lu.account_group_236
#: model:account.group.template,name:l10n_lu.account_group_65216
#: model:account.group.template,name:l10n_lu.account_group_75216
#: model:account.group.template,name:l10n_lu.account_group_75226
msgid "Loans, deposits and claims held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2363
#: model:account.group.template,name:l10n_lu.account_group_2363
msgid "Long-term receivables"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_65222
msgid "Loss on disposal of amounts owed by affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_65224
msgid ""
"Loss on disposal of amounts owed by undertakings with which the undertaking "
"is linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6522
msgid "Loss on disposal of financial fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_644
msgid "Loss on disposal of intangible and tangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6441
msgid "Loss on disposal of intangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_65226
msgid "Loss on disposal of loans, deposits and claims held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_65223
msgid "Loss on disposal of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_654
msgid ""
"Loss on disposal of receivables and transferable securities from current "
"assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6541
msgid "Loss on disposal of receivables from current assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_65225
msgid "Loss on disposal of securities held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_65221
msgid "Loss on disposal of shares in affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6442
msgid "Loss on disposal of tangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6542
msgid "Loss on disposal of transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_645
msgid "Losses on bad debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6037
#: model:account.group.template,name:l10n_lu.account_group_6037
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
#: model:account.group.template,name:l10n_lu.account_group_461221
msgid "MBT - Tax accrual"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461222
#: model:account.group.template,name:l10n_lu.account_group_461222
msgid "MBT - Tax payable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6721
#: model:account.group.template,name:l10n_lu.account_group_6721
msgid "MBT - current financial year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6722
#: model:account.group.template,name:l10n_lu.account_group_6722
msgid "MBT - previous financial years"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2222
#: model:account.group.template,name:l10n_lu.account_group_2222
msgid "Machinery"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6032
#: model:account.account.template,name:l10n_lu.lu_2020_account_61854
#: model:account.group.template,name:l10n_lu.account_group_6032
#: model:account.group.template,name:l10n_lu.account_group_61854
msgid "Maintenance supplies"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_615211
msgid "Management (if appropriate owner and partner)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_615211
msgid "Management (respectively owner and partner)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6151
msgid "Marketing and advertising costs"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_615
msgid "Marketing and communication costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_60761
#: model:account.group.template,name:l10n_lu.account_group_60761
msgid "Merchandise"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_112
#: model:account.group.template,name:l10n_lu.account_group_112
msgid "Merger premium"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_618
msgid "Miscellaneous external charges"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6488
#: model:account.group.template,name:l10n_lu.account_group_6488
msgid "Miscellaneous operating charges"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7488
#: model:account.group.template,name:l10n_lu.account_group_7488
msgid "Miscellaneous operating income"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4218
#: model:account.group.template,name:l10n_lu.account_group_4228
msgid "Miscellaneous receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_221313
#: model:account.group.template,name:l10n_lu.account_group_221313
msgid "Mixed-use buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6036
#: model:account.group.template,name:l10n_lu.account_group_6036
msgid "Motor fuels"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2232
#: model:account.group.template,name:l10n_lu.account_group_2232
msgid "Motor vehicles"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6466
#: model:account.group.template,name:l10n_lu.account_group_6466
msgid "Motor-vehicle taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_4611
#: model:account.group.template,name:l10n_lu.account_group_4611
msgid "Municipal authorities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_42142
#: model:account.group.template,name:l10n_lu.account_group_42142
#: model:account.group.template,name:l10n_lu.account_group_672
msgid "Municipal business tax"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106284
#: model:account.group.template,name:l10n_lu.account_group_46122
msgid "Municipal business tax (MBT)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106183
msgid "Municipal business tax - payment in arrears"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461231
#: model:account.group.template,name:l10n_lu.account_group_461231
msgid "NWT - Tax accrual"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461232
#: model:account.group.template,name:l10n_lu.account_group_461232
msgid "NWT - Tax payable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6811
#: model:account.group.template,name:l10n_lu.account_group_6811
msgid "NWT - current financial year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6812
#: model:account.group.template,name:l10n_lu.account_group_6812
msgid "NWT - previous financial years"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_70
msgid "Net turnover"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_42143
#: model:account.group.template,name:l10n_lu.account_group_42143
msgid "Net wealth tax"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_46123
#: model:account.group.template,name:l10n_lu.account_group_681
msgid "Net wealth tax (NWT)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_193
msgid "Non-convertible debenture loans"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6462
#: model:account.group.template,name:l10n_lu.account_group_6462
msgid "Non-refundable VAT"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_221312
#: model:account.group.template,name:l10n_lu.account_group_221312
msgid "Non-residential buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7099
#: model:account.group.template,name:l10n_lu.account_group_7099
msgid "Not allocated rebates, discounts and refunds"
msgstr ""

#. module: l10n_lu
#: model:account.fiscal.position,name:l10n_lu.1_account_fiscal_position_template_LU_NO
#: model:account.fiscal.position.template,name:l10n_lu.account_fiscal_position_template_LU_NO
msgid "Not liable to VAT"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6135
#: model:account.group.template,name:l10n_lu.account_group_6135
msgid "Notarial and similar fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6035
#: model:account.group.template,name:l10n_lu.account_group_6035
msgid "Office and administrative supplies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_61851
#: model:account.group.template,name:l10n_lu.account_group_61851
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
#: model:account.group.template,name:l10n_lu.account_group_1881
msgid "Operating provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42132
#: model:account.account.template,name:l10n_lu.lu_2011_account_42232
#: model:account.group.template,name:l10n_lu.account_group_42132
#: model:account.group.template,name:l10n_lu.account_group_42232
msgid "Operating subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61418
#: model:account.account.template,name:l10n_lu.lu_2011_account_6228
#: model:account.account.template,name:l10n_lu.lu_2020_account_61128
#: model:account.account.template,name:l10n_lu.lu_2020_account_61158
#: model:account.account.template,name:l10n_lu.lu_2020_account_61228
#: model:account.account.template,name:l10n_lu.lu_2020_account_61858
#: model:account.group.template,name:l10n_lu.account_group_61128
#: model:account.group.template,name:l10n_lu.account_group_61158
#: model:account.group.template,name:l10n_lu.account_group_61228
#: model:account.group.template,name:l10n_lu.account_group_61418
#: model:account.group.template,name:l10n_lu.account_group_61858
#: model:account.group.template,name:l10n_lu.account_group_6228
msgid "Other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106178
msgid "Other acquisitions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61338
#: model:account.group.template,name:l10n_lu.account_group_61338
msgid ""
"Other banking and similar services (except interest and similar expenses)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6218
#: model:account.group.template,name:l10n_lu.account_group_6218
msgid "Other benefits"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_221318
#: model:account.group.template,name:l10n_lu.account_group_221318
msgid "Other buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_168
#: model:account.group.template,name:l10n_lu.account_group_168
msgid "Other capital investment subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_518
#: model:account.group.template,name:l10n_lu.account_group_518
msgid "Other cash amounts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_708
#: model:account.group.template,name:l10n_lu.account_group_708
msgid "Other components of turnover"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6038
#: model:account.group.template,name:l10n_lu.account_group_6038
msgid "Other consumable supplies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106158
msgid "Other contributions"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_47
msgid "Other debts"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_472
msgid "Other debts payable after more than one year"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_471
msgid "Other debts payable within one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106248
msgid "Other disposals"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6468
#: model:account.group.template,name:l10n_lu.account_group_6468
msgid "Other duties and taxes"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_61
msgid "Other external charges"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_658
msgid "Other financial charges"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6581
#: model:account.group.template,name:l10n_lu.account_group_6581
msgid "Other financial charges - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6582
#: model:account.group.template,name:l10n_lu.account_group_6582
msgid "Other financial charges - other"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_758
msgid "Other financial income"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7581
#: model:account.group.template,name:l10n_lu.account_group_7581
msgid "Other financial income - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7582
#: model:account.group.template,name:l10n_lu.account_group_7582
msgid "Other financial income - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2238
#: model:account.group.template,name:l10n_lu.account_group_2238
msgid "Other fixtures"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7223
#: model:account.account.template,name:l10n_lu.lu_2011_account_7333
#: model:account.group.template,name:l10n_lu.account_group_7223
#: model:account.group.template,name:l10n_lu.account_group_7333
msgid ""
"Other fixtures and fittings, tools and equipment (included motor vehicles)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2243
#: model:account.group.template,name:l10n_lu.account_group_223
#: model:account.group.template,name:l10n_lu.account_group_2243
msgid ""
"Other fixtures and fittings, tools and equipment (including rolling stock)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6738
#: model:account.group.template,name:l10n_lu.account_group_6738
msgid "Other foreign income taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421818
#: model:account.account.template,name:l10n_lu.lu_2020_account_46158
#: model:account.group.template,name:l10n_lu.account_group_421818
#: model:account.group.template,name:l10n_lu.account_group_46158
msgid "Other foreign taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106168
msgid "Other in kind withdrawals"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7548
msgid "Other income from transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421628
#: model:account.account.template,name:l10n_lu.lu_2011_account_461428
#: model:account.group.template,name:l10n_lu.account_group_421628
#: model:account.group.template,name:l10n_lu.account_group_461428
msgid "Other indirect taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6148
#: model:account.group.template,name:l10n_lu.account_group_6148
msgid "Other insurances"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_755
msgid "Other interest income from current assets and discounts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_221118
#: model:account.group.template,name:l10n_lu.account_group_221118
msgid "Other land"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_47161
#: model:account.account.template,name:l10n_lu.lu_2020_account_47261
#: model:account.group.template,name:l10n_lu.account_group_47161
#: model:account.group.template,name:l10n_lu.account_group_47261
msgid "Other loans"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4718
#: model:account.account.template,name:l10n_lu.lu_2011_account_4728
#: model:account.group.template,name:l10n_lu.account_group_4718
#: model:account.group.template,name:l10n_lu.account_group_4728
msgid "Other miscellaneous debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6188
#: model:account.group.template,name:l10n_lu.account_group_6188
msgid "Other miscellaneous external charges"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_648
msgid "Other miscellaneous operating charges"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_748
msgid "Other miscellaneous operating income"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42188
#: model:account.account.template,name:l10n_lu.lu_2011_account_42288
#: model:account.group.template,name:l10n_lu.account_group_42188
#: model:account.group.template,name:l10n_lu.account_group_42288
msgid "Other miscellaneous receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_5088
#: model:account.group.template,name:l10n_lu.account_group_5088
msgid "Other miscellaneous transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_64
msgid "Other operating charges"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_74
msgid "Other operating income"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_45118
#: model:account.account.template,name:l10n_lu.lu_2011_account_45128
#: model:account.account.template,name:l10n_lu.lu_2011_account_45218
#: model:account.account.template,name:l10n_lu.lu_2011_account_45228
#: model:account.group.template,name:l10n_lu.account_group_45118
#: model:account.group.template,name:l10n_lu.account_group_45128
#: model:account.group.template,name:l10n_lu.account_group_45218
#: model:account.group.template,name:l10n_lu.account_group_45228
msgid "Other payables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106148
msgid "Other private insurance premiums"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61348
#: model:account.group.template,name:l10n_lu.account_group_61348
msgid "Other professional fees"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_188
msgid "Other provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6088
#: model:account.group.template,name:l10n_lu.account_group_6088
msgid "Other purchases included in the production of goods and services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61518
#: model:account.group.template,name:l10n_lu.account_group_61518
msgid "Other purchases of advertising services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6082
#: model:account.group.template,name:l10n_lu.account_group_6082
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
#: model:account.group.template,name:l10n_lu.account_group_41118
#: model:account.group.template,name:l10n_lu.account_group_41128
#: model:account.group.template,name:l10n_lu.account_group_41218
#: model:account.group.template,name:l10n_lu.account_group_41228
#: model:account.group.template,name:l10n_lu.account_group_42
#: model:account.group.template,name:l10n_lu.account_group_42168
#: model:account.group.template,name:l10n_lu.account_group_6454
msgid "Other receivables"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_422
msgid "Other receivables after one year"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_421
msgid "Other receivables within one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_64658
#: model:account.group.template,name:l10n_lu.account_group_64658
msgid "Other registration fees, stamp duties and mortgage duties"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6138
#: model:account.group.template,name:l10n_lu.account_group_6138
msgid "Other remuneration of intermediaries and professional fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_1381
#: model:account.group.template,name:l10n_lu.account_group_1381
msgid "Other reserves available for distribution"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_1382
msgid "Other reserves not available for distribution"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_138
msgid "Other reserves, including fair-value reserve"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_128
#: model:account.group.template,name:l10n_lu.account_group_128
msgid "Other revaluation reserves"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2358
#: model:account.group.template,name:l10n_lu.account_group_2358
msgid "Other securities held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_23528
#: model:account.group.template,name:l10n_lu.account_group_23528
msgid "Other securities held as fixed assets (creditor's right)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_23518
#: model:account.group.template,name:l10n_lu.account_group_23518
msgid "Other securities held as fixed assets (equity right)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_47164
#: model:account.account.template,name:l10n_lu.lu_2020_account_47264
#: model:account.group.template,name:l10n_lu.account_group_47168
#: model:account.group.template,name:l10n_lu.account_group_47268
msgid "Other similar debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_208
#: model:account.group.template,name:l10n_lu.account_group_208
msgid "Other similar expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6438
#: model:account.group.template,name:l10n_lu.account_group_6438
msgid "Other similar remuneration"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_64158
#: model:account.account.template,name:l10n_lu.lu_2011_account_721258
#: model:account.account.template,name:l10n_lu.lu_2011_account_74158
#: model:account.account.template,name:l10n_lu.lu_2020_account_703158
#: model:account.group.template,name:l10n_lu.account_group_64158
#: model:account.group.template,name:l10n_lu.account_group_703158
#: model:account.group.template,name:l10n_lu.account_group_721258
#: model:account.group.template,name:l10n_lu.account_group_74158
msgid "Other similar rights and assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_212158
#: model:account.group.template,name:l10n_lu.account_group_212158
msgid "Other similar rights and assets acquired for consideration"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_212258
#: model:account.group.template,name:l10n_lu.account_group_212258
msgid "Other similar rights and assets created by the undertaking itself"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42178
#: model:account.account.template,name:l10n_lu.lu_2011_account_4628
#: model:account.group.template,name:l10n_lu.account_group_42178
#: model:account.group.template,name:l10n_lu.account_group_4628
msgid "Other social bodies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6232
#: model:account.group.template,name:l10n_lu.account_group_6232
msgid "Other social security costs (including illness, accidents, a.s.o.)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106198
msgid "Other special private withdrawals"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_624
msgid "Other staff expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6248
#: model:account.group.template,name:l10n_lu.account_group_6248
msgid "Other staff expenses not mentioned above"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_622
msgid "Other staff remuneration"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_42138
#: model:account.account.template,name:l10n_lu.lu_2011_account_42238
#: model:account.group.template,name:l10n_lu.account_group_42138
#: model:account.group.template,name:l10n_lu.account_group_42238
msgid "Other subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7458
#: model:account.group.template,name:l10n_lu.account_group_7458
msgid "Other subsidies for operating activities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_621128
#: model:account.group.template,name:l10n_lu.account_group_621128
msgid "Other supplements"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106288
msgid "Other tax refunds"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106188
#: model:account.account.template,name:l10n_lu.lu_2011_account_688
#: model:account.group.template,name:l10n_lu.account_group_688
msgid "Other taxes"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_68
msgid "Other taxes not included in the previous caption"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_75488
#: model:account.account.template,name:l10n_lu.lu_2020_account_65428
#: model:account.account.template,name:l10n_lu.lu_2020_account_75318
#: model:account.account.template,name:l10n_lu.lu_2020_account_75428
#: model:account.group.template,name:l10n_lu.account_group_508
#: model:account.group.template,name:l10n_lu.account_group_65428
#: model:account.group.template,name:l10n_lu.account_group_75428
#: model:account.group.template,name:l10n_lu.account_group_75488
msgid "Other transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6168
#: model:account.group.template,name:l10n_lu.account_group_6168
msgid "Other transportation"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_60814
#: model:account.group.template,name:l10n_lu.account_group_60814
msgid "Outsourcing included in the production of goods and services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_621123
#: model:account.group.template,name:l10n_lu.account_group_621123
msgid "Overtime"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_75482
#: model:account.account.template,name:l10n_lu.lu_2020_account_65422
#: model:account.account.template,name:l10n_lu.lu_2020_account_75312
#: model:account.account.template,name:l10n_lu.lu_2020_account_75422
#: model:account.group.template,name:l10n_lu.account_group_65422
#: model:account.group.template,name:l10n_lu.account_group_75422
#: model:account.group.template,name:l10n_lu.account_group_75482
msgid "Own shares or corporate units"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_502
#: model:account.group.template,name:l10n_lu.account_group_502
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
#: model:account.group.template,name:l10n_lu.account_group_233
#: model:account.group.template,name:l10n_lu.account_group_65213
#: model:account.group.template,name:l10n_lu.account_group_75213
#: model:account.group.template,name:l10n_lu.account_group_75223
msgid "Participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_21212
#: model:account.account.template,name:l10n_lu.lu_2011_account_21222
#: model:account.account.template,name:l10n_lu.lu_2011_account_6412
#: model:account.account.template,name:l10n_lu.lu_2011_account_72122
#: model:account.account.template,name:l10n_lu.lu_2011_account_7412
#: model:account.account.template,name:l10n_lu.lu_2020_account_70312
#: model:account.group.template,name:l10n_lu.account_group_21212
#: model:account.group.template,name:l10n_lu.account_group_21222
#: model:account.group.template,name:l10n_lu.account_group_6412
#: model:account.group.template,name:l10n_lu.account_group_70312
#: model:account.group.template,name:l10n_lu.account_group_72122
#: model:account.group.template,name:l10n_lu.account_group_7412
msgid "Patents"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10622
msgid "Personal holdings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2221
#: model:account.group.template,name:l10n_lu.account_group_2221
msgid "Plant"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2242
#: model:account.account.template,name:l10n_lu.lu_2011_account_7222
#: model:account.group.template,name:l10n_lu.account_group_222
#: model:account.group.template,name:l10n_lu.account_group_2242
#: model:account.group.template,name:l10n_lu.account_group_7222
msgid "Plant and machinery"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61531
#: model:account.group.template,name:l10n_lu.account_group_61531
msgid "Postal charges"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6153
msgid "Postal charges and telecommunication costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_62411
#: model:account.group.template,name:l10n_lu.account_group_62411
msgid "Premiums for external pensions funds"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_114
#: model:account.group.template,name:l10n_lu.account_group_114
msgid "Premiums on conversion of bonds into shares"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61511
#: model:account.group.template,name:l10n_lu.account_group_61511
msgid "Press advertising"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_67322
#: model:account.group.template,name:l10n_lu.account_group_67322
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
#: model:account.group.template,name:l10n_lu.account_group_7451
msgid "Product subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6134
msgid "Professional fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_221112
#: model:account.group.template,name:l10n_lu.account_group_221112
msgid "Property rights and similar"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_18
msgid "Provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_181
#: model:account.group.template,name:l10n_lu.account_group_181
msgid "Provisions for pensions and similar obligations"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_182
#: model:account.group.template,name:l10n_lu.account_group_182
msgid "Provisions for taxation"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_621122
#: model:account.group.template,name:l10n_lu.account_group_621122
msgid "Public holidays"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6083
msgid "Purchase of greenhous gas and similar emission quotas"
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
#: model:account.group.template,name:l10n_lu.account_group_45111
#: model:account.group.template,name:l10n_lu.account_group_45121
#: model:account.group.template,name:l10n_lu.account_group_45211
#: model:account.group.template,name:l10n_lu.account_group_45221
msgid "Purchases and services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6063
#: model:account.group.template,name:l10n_lu.account_group_6063
msgid "Purchases of buildings for resale"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_603
msgid "Purchases of consumable materials and supplies"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_608
msgid "Purchases of items included in the production of goods and services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6062
#: model:account.group.template,name:l10n_lu.account_group_6062
msgid "Purchases of land for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6061
#: model:account.group.template,name:l10n_lu.account_group_6061
msgid "Purchases of merchandise"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_606
msgid "Purchases of merchandise and other goods for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_604
#: model:account.group.template,name:l10n_lu.account_group_604
msgid "Purchases of packaging"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_601
#: model:account.group.template,name:l10n_lu.account_group_601
msgid "Purchases of raw materials"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7095
#: model:account.group.template,name:l10n_lu.account_group_7095
msgid "RDR on commissions and brokerage fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7098
#: model:account.group.template,name:l10n_lu.account_group_7098
msgid "RDR on other components of turnover"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6098
#: model:account.group.template,name:l10n_lu.account_group_6098
msgid "RDR on purchases included in the production of goods and services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6093
#: model:account.group.template,name:l10n_lu.account_group_6093
msgid "RDR on purchases of consumable materials and supplies"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6096
#: model:account.group.template,name:l10n_lu.account_group_6096
msgid "RDR on purchases of merchandise and other goods for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6094
#: model:account.group.template,name:l10n_lu.account_group_6094
msgid "RDR on purchases of packaging"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6091
#: model:account.group.template,name:l10n_lu.account_group_6091
msgid "RDR on purchases of raw materials"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7092
#: model:account.group.template,name:l10n_lu.account_group_7092
msgid "RDR on sales of goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7096
#: model:account.group.template,name:l10n_lu.account_group_7096
msgid "RDR on sales of merchandise and other goods for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7094
#: model:account.group.template,name:l10n_lu.account_group_7094
msgid "RDR on sales of packages"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7093
#: model:account.group.template,name:l10n_lu.account_group_7093
msgid "RDR on sales of services"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_735
msgid "RVA and FVA on receivables from current assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_75112
msgid "RVA on amounts owed by affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7352
#: model:account.group.template,name:l10n_lu.account_group_7352
msgid ""
"RVA on amounts owed by affiliated undertakings and undertakings with which "
"the undertaking is linked by virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_75114
msgid ""
"RVA on amounts owed by undertakings with which the company is linked by "
"virtue of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_73313
#: model:account.group.template,name:l10n_lu.account_group_73313
msgid "RVA on buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7322
#: model:account.group.template,name:l10n_lu.account_group_7322
msgid ""
"RVA on concessions, patents, licences, trademarks and similar rights and "
"assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7321
#: model:account.group.template,name:l10n_lu.account_group_7321
msgid "RVA on development costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7324
#: model:account.group.template,name:l10n_lu.account_group_7324
msgid "RVA on down payments and intangible fixed assets under development"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7334
#: model:account.group.template,name:l10n_lu.account_group_7334
msgid "RVA on down payments and tangible fixed assets under development"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7345
#: model:account.group.template,name:l10n_lu.account_group_7345
msgid "RVA on down payments on inventories"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7511
msgid "RVA on financial fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_73314
#: model:account.group.template,name:l10n_lu.account_group_73314
msgid "RVA on fixtures and fittings-out of buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_73312
#: model:account.group.template,name:l10n_lu.account_group_73312
msgid "RVA on fixtures and fittings-out of land"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_732
msgid "RVA on intangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_734
msgid "RVA on inventories"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7343
#: model:account.group.template,name:l10n_lu.account_group_7343
msgid "RVA on inventories of goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7344
#: model:account.group.template,name:l10n_lu.account_group_7344
msgid "RVA on inventories of merchandise and other goods for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7341
#: model:account.group.template,name:l10n_lu.account_group_7341
msgid "RVA on inventories of raw materials and consumables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7342
#: model:account.group.template,name:l10n_lu.account_group_7342
msgid "RVA on inventories of work and contracts in progress"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_73311
#: model:account.group.template,name:l10n_lu.account_group_73311
msgid "RVA on land"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7331
msgid ""
"RVA on land, fixtures and fittings-out and buildings and FVA on investment "
"properties"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_75116
msgid "RVA on loans, deposits and claims held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7353
#: model:account.group.template,name:l10n_lu.account_group_7353
msgid "RVA on other receivables"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_75318
msgid "RVA on other transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_75312
msgid "RVA on own shares or corporate units"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_75113
msgid "RVA on participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7332
#: model:account.group.template,name:l10n_lu.account_group_7332
msgid "RVA on plant and machinery"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_75115
msgid "RVA on securities held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_75111
#: model:account.group.template,name:l10n_lu.account_group_75311
msgid "RVA on shares in affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_75313
msgid ""
"RVA on shares in undertakings with which the undertaking is linked by virtue"
" of participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_733
msgid ""
"RVA on tangible fixed assets and fair value adjustments (FVA) on investment "
"properties"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7351
#: model:account.group.template,name:l10n_lu.account_group_7351
msgid "RVA on trade receivables"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7531
msgid "RVA on transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6461
#: model:account.group.template,name:l10n_lu.account_group_6461
msgid "Real property tax"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_709
msgid ""
"Rebates, discounts and refunds (RDR) granted and not immediately deducted "
"from sales"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_609
msgid ""
"Rebates, discounts and refunds (RDR) received and not directly deducted from"
" purchases"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_619
#: model:account.group.template,name:l10n_lu.account_group_619
msgid "Rebates, discounts and refunds received on other external charges"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10627
msgid "Received child benefit"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_4711
#: model:account.account.template,name:l10n_lu.lu_2020_account_4721
#: model:account.group.template,name:l10n_lu.account_group_4711
#: model:account.group.template,name:l10n_lu.account_group_4721
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
#: model:account.group.template,name:l10n_lu.account_group_61524
msgid "Receptions and entertainment costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106193
msgid "Refund of private debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6219
#: model:account.group.template,name:l10n_lu.account_group_6219
msgid "Refunds on wages paid"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421621
#: model:account.account.template,name:l10n_lu.lu_2011_account_461421
#: model:account.group.template,name:l10n_lu.account_group_421621
#: model:account.group.template,name:l10n_lu.account_group_461421
msgid "Registration duties"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_64651
#: model:account.group.template,name:l10n_lu.account_group_64651
msgid "Registration fees"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6465
msgid "Registration fees, stamp duties and mortgage duties"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61522
#: model:account.group.template,name:l10n_lu.account_group_61522
msgid "Relocation expenses"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_613
msgid "Remuneration of intermediaries and professional fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106162
msgid "Rent"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_7032
msgid "Rental income"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_742
msgid "Rental income from ancillary activities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_70322
#: model:account.group.template,name:l10n_lu.account_group_70322
msgid "Rental income from movable property"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_70321
#: model:account.group.template,name:l10n_lu.account_group_70321
msgid "Rental income from real property"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7422
#: model:account.group.template,name:l10n_lu.account_group_7422
msgid "Rental income on movable property"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7421
#: model:account.group.template,name:l10n_lu.account_group_7421
msgid "Rental income on real property"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6112
msgid "Rents and operational leasing on movable property"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6111
msgid "Rents and operationnal leasing for real property"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_611
msgid "Rents and service charges"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106191
msgid "Repairs to private buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_60812
#: model:account.group.template,name:l10n_lu.account_group_60812
msgid "Research and development"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_13821
#: model:account.group.template,name:l10n_lu.account_group_13821
msgid "Reserve for net wealth tax (NWT)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_13
msgid "Reserves"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_132
#: model:account.group.template,name:l10n_lu.account_group_132
msgid "Reserves for own shares or own corporate units"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_13822
#: model:account.group.template,name:l10n_lu.account_group_13822
msgid "Reserves in application of fair value"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_122
#: model:account.group.template,name:l10n_lu.account_group_122
msgid "Reserves in application of the equity method"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_13828
#: model:account.group.template,name:l10n_lu.account_group_13828
msgid "Reserves not available for distribution not mentioned above"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_133
#: model:account.group.template,name:l10n_lu.account_group_133
msgid "Reserves provided for by the articles of association"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_221311
#: model:account.group.template,name:l10n_lu.account_group_221311
msgid "Residential buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_142
#: model:account.group.template,name:l10n_lu.account_group_142
msgid "Result for the financial year"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_14
msgid "Result for the financial year and results brought forward"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_141
msgid "Results brought forward"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_1412
#: model:account.group.template,name:l10n_lu.account_group_1412
msgid "Results brought forward (assigned)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_1411
#: model:account.group.template,name:l10n_lu.account_group_1411
msgid "Results brought forward in the process of assignment"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2237
#: model:account.group.template,name:l10n_lu.account_group_2237
msgid "Returnable packaging"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_12
msgid "Revaluation reserves"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_759
msgid "Reversals of financial provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7591
#: model:account.group.template,name:l10n_lu.account_group_7591
msgid "Reversals of financial provisions - affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7592
#: model:account.group.template,name:l10n_lu.account_group_7592
msgid "Reversals of financial provisions - other"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7492
#: model:account.group.template,name:l10n_lu.account_group_7492
msgid "Reversals of operating provisions"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_749
msgid "Reversals of provisions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_779
#: model:account.group.template,name:l10n_lu.account_group_779
msgid "Reversals of provisions for deferred taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7491
#: model:account.group.template,name:l10n_lu.account_group_7491
msgid "Reversals of provisions for taxes"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_747
msgid ""
"Reversals of temporarily not taxable capital gains and of investment "
"subsidies"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_751
msgid ""
"Reversals of value adjustments (RVA) and fair-value adjustments (FVA) on "
"financial fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_753
msgid ""
"Reversals of value adjustments (RVA) and fair-value adjustments (FVA) on "
"transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_73
msgid ""
"Reversals of value adjustments (RVA) on intangible, tangible and current "
"assets (except transferable securities)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61123
#: model:account.account.template,name:l10n_lu.lu_2011_account_61153
#: model:account.account.template,name:l10n_lu.lu_2011_account_61223
#: model:account.account.template,name:l10n_lu.lu_2011_account_61412
#: model:account.group.template,name:l10n_lu.account_group_61123
#: model:account.group.template,name:l10n_lu.account_group_61153
#: model:account.group.template,name:l10n_lu.account_group_61223
#: model:account.group.template,name:l10n_lu.account_group_61412
msgid "Rolling stock"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_703001
msgid "Sale of Services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7063
#: model:account.group.template,name:l10n_lu.account_group_7063
msgid "Sales of buildings for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7021
#: model:account.group.template,name:l10n_lu.account_group_7021
msgid "Sales of finished goods"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_702
msgid "Sales of goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7062
#: model:account.group.template,name:l10n_lu.account_group_7062
msgid "Sales of land resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7061
#: model:account.group.template,name:l10n_lu.account_group_7061
msgid "Sales of merchandise"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_706
msgid "Sales of merchandise and other goods for resale"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_704
#: model:account.group.template,name:l10n_lu.account_group_704
msgid "Sales of packaging"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7023
#: model:account.group.template,name:l10n_lu.account_group_7023
msgid "Sales of residual products"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7022
#: model:account.group.template,name:l10n_lu.account_group_7022
msgid "Sales of semi-finished goods"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_703
msgid "Sales of services"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7039
#: model:account.group.template,name:l10n_lu.account_group_7039
msgid "Sales of services in the course of completion"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7033
#: model:account.group.template,name:l10n_lu.account_group_7033
msgid "Sales of services not mentioned above"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7029
#: model:account.group.template,name:l10n_lu.account_group_7029
msgid "Sales of work in progress"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61512
#: model:account.group.template,name:l10n_lu.account_group_61512
msgid "Samples"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_75115
#: model:account.account.template,name:l10n_lu.lu_2020_account_65215
#: model:account.account.template,name:l10n_lu.lu_2020_account_75215
#: model:account.group.template,name:l10n_lu.account_group_235
#: model:account.group.template,name:l10n_lu.account_group_65215
#: model:account.group.template,name:l10n_lu.account_group_75215
#: model:account.group.template,name:l10n_lu.account_group_75225
msgid "Securities held as fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_2352
msgid "Securities held as fixed assets (creditor's right)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_2351
msgid "Securities held as fixed assets (equity right)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6113
#: model:account.group.template,name:l10n_lu.account_group_6113
msgid "Service charges and co-ownership expenses"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6081
msgid "Services included in the production of goods and services"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6122
msgid "Servicing, repairs and maintenance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_201
#: model:account.group.template,name:l10n_lu.account_group_201
msgid "Set-up and start-up costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_62116
#: model:account.group.template,name:l10n_lu.account_group_62116
msgid "Severance pay"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_657
#: model:account.group.template,name:l10n_lu.account_group_657
msgid ""
"Share in the losses of undertakings accounted for under the equity method"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_757
#: model:account.group.template,name:l10n_lu.account_group_757
msgid ""
"Share of profit from undertakings accounted for under the equity method"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_111
#: model:account.group.template,name:l10n_lu.account_group_111
msgid "Share premium"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_11
msgid "Share premium and similar premiums"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_5081
#: model:account.group.template,name:l10n_lu.account_group_5081
msgid "Shares - listed securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_5082
#: model:account.group.template,name:l10n_lu.account_group_5082
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
#: model:account.group.template,name:l10n_lu.account_group_231
#: model:account.group.template,name:l10n_lu.account_group_501
#: model:account.group.template,name:l10n_lu.account_group_65211
#: model:account.group.template,name:l10n_lu.account_group_65421
#: model:account.group.template,name:l10n_lu.account_group_75211
#: model:account.group.template,name:l10n_lu.account_group_75221
#: model:account.group.template,name:l10n_lu.account_group_75421
#: model:account.group.template,name:l10n_lu.account_group_75481
msgid "Shares in affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_65423
msgid ""
"Shares in in undertakings with which the undertaking is linked by virtue of "
"participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_503
#: model:account.account.template,name:l10n_lu.lu_2011_account_75483
#: model:account.account.template,name:l10n_lu.lu_2020_account_65423
#: model:account.account.template,name:l10n_lu.lu_2020_account_75313
#: model:account.account.template,name:l10n_lu.lu_2020_account_75423
#: model:account.group.template,name:l10n_lu.account_group_503
#: model:account.group.template,name:l10n_lu.account_group_75423
#: model:account.group.template,name:l10n_lu.account_group_75483
msgid ""
"Shares in undertakings with which the undertaking is linked by virtue of "
"participating interests"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_2353
#: model:account.group.template,name:l10n_lu.account_group_2353
msgid "Shares of collective investment funds"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_23511
msgid "Shares or corporate units"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_21215
#: model:account.group.template,name:l10n_lu.account_group_21225
#: model:account.group.template,name:l10n_lu.account_group_6415
#: model:account.group.template,name:l10n_lu.account_group_70315
#: model:account.group.template,name:l10n_lu.account_group_72125
#: model:account.group.template,name:l10n_lu.account_group_7415
msgid "Similar rights and assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_61852
#: model:account.group.template,name:l10n_lu.account_group_61852
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
#: model:account.group.template,name:l10n_lu.account_group_623
msgid "Social security costs (employer's share)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_462
msgid "Social security debts and other social securities offices"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6231
#: model:account.group.template,name:l10n_lu.account_group_6231
msgid "Social security on pensions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_21213
#: model:account.account.template,name:l10n_lu.lu_2011_account_21223
#: model:account.account.template,name:l10n_lu.lu_2011_account_6413
#: model:account.account.template,name:l10n_lu.lu_2011_account_72123
#: model:account.account.template,name:l10n_lu.lu_2011_account_7413
#: model:account.account.template,name:l10n_lu.lu_2020_account_70313
#: model:account.group.template,name:l10n_lu.account_group_21213
#: model:account.group.template,name:l10n_lu.account_group_21223
#: model:account.group.template,name:l10n_lu.account_group_6413
#: model:account.group.template,name:l10n_lu.account_group_70313
#: model:account.group.template,name:l10n_lu.account_group_72123
#: model:account.group.template,name:l10n_lu.account_group_7413
msgid "Software licences"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_60311
#: model:account.account.template,name:l10n_lu.lu_2020_account_61841
#: model:account.group.template,name:l10n_lu.account_group_60311
#: model:account.group.template,name:l10n_lu.account_group_61841
msgid "Solid fuels"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61517
#: model:account.group.template,name:l10n_lu.account_group_61517
msgid "Sponsorship"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_615212
#: model:account.group.template,name:l10n_lu.account_group_615212
msgid "Staff"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4211
msgid "Staff - Advances and down payments"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_4221
#: model:account.group.template,name:l10n_lu.account_group_4221
msgid "Staff - advances and down payments"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_62
msgid "Staff expenses"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_621
msgid "Staff remuneration"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_483
msgid "State - Greenhous gas and similar emission quotas received"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_4715
#: model:account.group.template,name:l10n_lu.account_group_4715
msgid ""
"State - Greenhous gas and similar emission quotas to be returned or acquired"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_483
msgid "State - Greenhouse gas and similar emission quotas received"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4213
#: model:account.group.template,name:l10n_lu.account_group_4223
msgid "State - Subsidies to be received"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6221
#: model:account.group.template,name:l10n_lu.account_group_6221
msgid "Students"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_612
msgid "Subcontracting, servicing, repairs and maintenance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_101
#: model:account.group.template,name:l10n_lu.account_group_101
msgid "Subscribed capital"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_103
#: model:account.group.template,name:l10n_lu.account_group_103
msgid "Subscribed capital called but unpaid"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_102
#: model:account.group.template,name:l10n_lu.account_group_102
msgid "Subscribed capital not called"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_10
msgid "Subscribed capital or branches' assigned capital and owner's account"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421622
#: model:account.account.template,name:l10n_lu.lu_2011_account_461422
#: model:account.account.template,name:l10n_lu.lu_2011_account_682
#: model:account.group.template,name:l10n_lu.account_group_421622
#: model:account.group.template,name:l10n_lu.account_group_461422
#: model:account.group.template,name:l10n_lu.account_group_682
msgid "Subscription tax"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_745
msgid "Subsidies for operating activities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_7454
#: model:account.group.template,name:l10n_lu.account_group_7454
msgid "Subsidies in favour of employment development"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_161
msgid "Subsidies on intangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_1621
#: model:account.group.template,name:l10n_lu.account_group_1621
msgid "Subsidies on land, fitting-outs and buildings"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_1623
#: model:account.group.template,name:l10n_lu.account_group_1623
msgid ""
"Subsidies on other fixtures, fittings, tools and equipment (including "
"rolling stock)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_1622
#: model:account.group.template,name:l10n_lu.account_group_1622
msgid "Subsidies on plant and machinery"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_162
msgid "Subsidies on tangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_621121
#: model:account.group.template,name:l10n_lu.account_group_621121
msgid "Sunday"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_44111
#: model:account.account.template,name:l10n_lu.lu_2011_account_44121
#: model:account.group.template,name:l10n_lu.account_group_44111
#: model:account.group.template,name:l10n_lu.account_group_44121
msgid "Suppliers"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_44112
#: model:account.group.template,name:l10n_lu.account_group_44112
msgid "Suppliers - invoices not yet received"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_44113
#: model:account.account.template,name:l10n_lu.lu_2020_account_44123
#: model:account.group.template,name:l10n_lu.account_group_44113
#: model:account.group.template,name:l10n_lu.account_group_44123
msgid "Suppliers with a debit balance"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6185
msgid "Supplies and small equipment"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6186
#: model:account.group.template,name:l10n_lu.account_group_6186
msgid "Surveillance and security charges"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_62117
#: model:account.group.template,name:l10n_lu.account_group_62117
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
#: model:account.group.template,name:l10n_lu.account_group_60811
msgid "Tailoring"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_22
#: model:account.group.template,name:l10n_lu.account_group_722
msgid "Tangible fixed assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_46
msgid "Tax and social security debts"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_461
msgid "Tax debts"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6733
#: model:account.group.template,name:l10n_lu.account_group_6733
msgid "Taxes levied on non-resident undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6732
msgid "Taxes levied on permanent establishments"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_646
msgid "Taxes, duties and similar expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_61532
#: model:account.group.template,name:l10n_lu.account_group_61532
msgid "Telecommunication costs"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106165
msgid "Telephone"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_13823
msgid "Temporarily not taxable capital gains"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7471
#: model:account.group.template,name:l10n_lu.account_group_7471
msgid "Temporarily not taxable capital gains not reinvested"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_7472
#: model:account.account.template,name:l10n_lu.lu_2020_account_138232
#: model:account.group.template,name:l10n_lu.account_group_138232
#: model:account.group.template,name:l10n_lu.account_group_7472
msgid "Temporarily not taxable capital gains reinvested"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_138231
#: model:account.group.template,name:l10n_lu.account_group_138231
msgid "Temporarily not taxable capital gains to reinvest"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_123
#: model:account.group.template,name:l10n_lu.account_group_123
msgid "Temporarily not taxable currency translation adjustments"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6171
#: model:account.group.template,name:l10n_lu.account_group_6171
msgid "Temporary staff"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_106144
#: model:account.account.template,name:l10n_lu.lu_2011_account_6146
#: model:account.group.template,name:l10n_lu.account_group_6146
msgid "Third-party insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2233
#: model:account.group.template,name:l10n_lu.account_group_2233
msgid "Tools"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_441
msgid "Trade payables"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4412
msgid "Trade payables after more than one year"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_44
msgid "Trade payables and bills of exchange"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_4411
msgid "Trade payables within one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_41111
#: model:account.account.template,name:l10n_lu.lu_2011_account_41121
#: model:account.account.template,name:l10n_lu.lu_2011_account_41211
#: model:account.account.template,name:l10n_lu.lu_2011_account_41221
#: model:account.account.template,name:l10n_lu.lu_2011_account_6451
#: model:account.group.template,name:l10n_lu.account_group_41111
#: model:account.group.template,name:l10n_lu.account_group_41121
#: model:account.group.template,name:l10n_lu.account_group_41211
#: model:account.group.template,name:l10n_lu.account_group_41221
#: model:account.group.template,name:l10n_lu.account_group_6451
msgid "Trade receivables"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_40
msgid "Trade receivables (Receivables from sales and rendering of services)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_402
msgid "Trade receivables due and payable after more than one year"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_401
msgid "Trade receivables due and payable within one year"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6414
#: model:account.group.template,name:l10n_lu.account_group_6414
msgid "Trademarks and franchise"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_21214
#: model:account.account.template,name:l10n_lu.lu_2011_account_21224
#: model:account.account.template,name:l10n_lu.lu_2011_account_72124
#: model:account.account.template,name:l10n_lu.lu_2011_account_7414
#: model:account.account.template,name:l10n_lu.lu_2020_account_70314
#: model:account.group.template,name:l10n_lu.account_group_21214
#: model:account.group.template,name:l10n_lu.account_group_21224
#: model:account.group.template,name:l10n_lu.account_group_70314
#: model:account.group.template,name:l10n_lu.account_group_72124
#: model:account.group.template,name:l10n_lu.account_group_7414
msgid "Trademarks and franchises"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_50
msgid "Transferable securities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_484
#: model:account.group.template,name:l10n_lu.account_group_484
msgid "Transitory or suspense accounts - Assets"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_485
#: model:account.group.template,name:l10n_lu.account_group_485
msgid "Transitory or suspense accounts - Liabilities"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_6143
#: model:account.group.template,name:l10n_lu.account_group_6143
msgid "Transport insurance"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_2231
#: model:account.group.template,name:l10n_lu.account_group_2231
msgid "Transportation and handling equipment"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_616
msgid "Transportation of goods and collective staff transportation"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6161
#: model:account.group.template,name:l10n_lu.account_group_6161
msgid "Transportation of purchased goods"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6162
#: model:account.group.template,name:l10n_lu.account_group_6162
msgid "Transportation of sold goods"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_6152
msgid "Travel and entertainment expenses"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_61521
msgid "Travel expenses"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6099
#: model:account.group.template,name:l10n_lu.account_group_6099
msgid "Unallocated RDR"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_5085
#: model:account.group.template,name:l10n_lu.account_group_5085
msgid "Unlisted debenture loans"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_235112
#: model:account.group.template,name:l10n_lu.account_group_235112
msgid "Unlisted shares"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_60
msgid "Use of merchandise, raw and consumable materials"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461418
#: model:account.group.template,name:l10n_lu.account_group_461418
msgid "VAT - Other payables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421618
#: model:account.group.template,name:l10n_lu.account_group_421618
msgid "VAT - Other receivables"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421613
#: model:account.group.template,name:l10n_lu.account_group_421613
msgid "VAT down payments made"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461413
#: model:account.group.template,name:l10n_lu.account_group_461413
msgid "VAT down payments received"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_421611
#: model:account.group.template,name:l10n_lu.account_group_421611
msgid "VAT paid and recoverable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_461412
#: model:account.group.template,name:l10n_lu.account_group_461412
msgid "VAT payable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_421612
#: model:account.group.template,name:l10n_lu.account_group_421612
msgid "VAT receivable"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_461411
#: model:account.group.template,name:l10n_lu.account_group_461411
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
#: model:account.group.template,name:l10n_lu.account_group_4019
#: model:account.group.template,name:l10n_lu.account_group_4029
#: model:account.group.template,name:l10n_lu.account_group_41119
#: model:account.group.template,name:l10n_lu.account_group_41129
#: model:account.group.template,name:l10n_lu.account_group_41219
#: model:account.group.template,name:l10n_lu.account_group_41229
#: model:account.group.template,name:l10n_lu.account_group_42119
#: model:account.group.template,name:l10n_lu.account_group_42189
#: model:account.group.template,name:l10n_lu.account_group_42289
msgid "Value adjustments"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_42161
#: model:account.group.template,name:l10n_lu.account_group_46141
msgid "Value-added tax (VAT)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_62112
msgid "Wage supplements"
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
#: model:account.group.template,name:l10n_lu.account_group_60314
msgid "Water and sewage"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_61844
#: model:account.group.template,name:l10n_lu.account_group_61844
msgid "Water and waste water"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_10612
msgid "Withdrawals of merchandise, finished products and services (at cost)"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2020_account_62413
#: model:account.group.template,name:l10n_lu.account_group_62413
msgid "Withholding tax on complementary pensions"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_46126
#: model:account.account.template,name:l10n_lu.lu_2020_account_42146
#: model:account.group.template,name:l10n_lu.account_group_42146
#: model:account.group.template,name:l10n_lu.account_group_46126
msgid "Withholding tax on director's fees"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_46125
#: model:account.account.template,name:l10n_lu.lu_2020_account_42145
#: model:account.group.template,name:l10n_lu.account_group_42145
#: model:account.group.template,name:l10n_lu.account_group_46125
msgid "Withholding tax on financial investment income"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_46124
#: model:account.account.template,name:l10n_lu.lu_2020_account_42144
#: model:account.group.template,name:l10n_lu.account_group_42144
#: model:account.group.template,name:l10n_lu.account_group_46124
msgid "Withholding tax on wages and salaries"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6731
#: model:account.group.template,name:l10n_lu.account_group_6731
msgid "Withholding taxes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6034
#: model:account.account.template,name:l10n_lu.lu_2020_account_61853
#: model:account.group.template,name:l10n_lu.account_group_6034
#: model:account.group.template,name:l10n_lu.account_group_61853
msgid "Work clothes"
msgstr ""

#. module: l10n_lu
#: model:account.account.template,name:l10n_lu.lu_2011_account_6033
#: model:account.group.template,name:l10n_lu.account_group_6033
msgid "Workshop, factory and store supplies and small equipment"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_16121
msgid "acquired against payment (except Goodwill)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_2121
msgid "acquired for consideration (except Goodwill)"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_16122
#: model:account.group.template,name:l10n_lu.account_group_2122
msgid "created by the undertaking itself"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_1922
#: model:account.group.template,name:l10n_lu.account_group_1932
#: model:account.group.template,name:l10n_lu.account_group_1942
msgid "due and payable after more than one year"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_1921
#: model:account.group.template,name:l10n_lu.account_group_1931
#: model:account.group.template,name:l10n_lu.account_group_1941
msgid "due and payable within one year"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_755231
msgid "from affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_755232
msgid "from other"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_65413
msgid "from other receivables from current assets"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_75411
msgid "on affiliated undertakings"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_75413
msgid "on other current receivables"
msgstr ""

#. module: l10n_lu
#: model:account.group.template,name:l10n_lu.account_group_75412
msgid ""
"on undertakings with which the undertaking is linked by virtue of "
"participating interests"
msgstr ""

```

## File: migrations\2.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.addons.account.models.chart_template import update_taxes_from_templates


def migrate(cr, version):
    update_taxes_from_templates(cr, 'l10n_lu.lu_2011_chart_1')

```

## File: models\account_chart_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models, fields


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    @api.model
    def _prepare_all_journals(self, acc_template_ref, company, journals_dict=None):
        journal_data = super(AccountChartTemplate, self)._prepare_all_journals(
            acc_template_ref, company, journals_dict)
        for journal in journal_data:
            if journal['type'] in ('sale', 'purchase') and company.country_id.code == "LU":
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

