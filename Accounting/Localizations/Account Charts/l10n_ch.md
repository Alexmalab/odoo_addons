# Odoo Module: l10n_ch

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report
from . import wizard

from odoo import api, SUPERUSER_ID


def load_translations(env):
    env.ref('l10n_ch.l10nch_chart_template').process_coa_translations()


def init_settings(env):
    '''If the company is localized in Switzerland, activate the cash rounding by default.
    '''
    # The cash rounding is activated by default only if the company is localized in Switzerland or Liechtenstein.
    for company in env['res.company'].search([('partner_id.country_id.code', 'in', ["CH", "LI"])]):
        res_config_id = env['res.config.settings'].create({
            'company_id': company.id,
            'group_cash_rounding': True
        })
        # We need to call execute, otherwise the "implied_group" in fields are not processed.
        res_config_id.execute()


def post_init(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    load_translations(env)
    init_settings(env)

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
# Main contributor: Nicolas Bessi. Camptocamp SA
# Financial contributors: Hasa SA, Open Net SA,
#                         Prisme Solutions Informatique SA, Quod SA
# Translation contributors: brain-tec AG, Agile Business Group
{
    'name': "Switzerland - Accounting",
    'description': """
Swiss localization
==================
This module defines a chart of account for Switzerland (Swiss PME/KMU 2015), taxes and enables the generation of ISR and QR-bill when you print an invoice or send it by mail.

An ISR will be generated if you specify the information it needs :
    - The bank account you expect to be paid on must be set, and have a valid postal reference.
    - Your invoice must have been set assigned a bank account to receive its payment
      (this can be done manually, but a default value is automatically set if you have defined a bank account).
    - You must have set the postal references of your bank.
    - Your invoice must be in EUR or CHF (as ISRs do not accept other currencies)

A QR-bill will be generated if:
    - The partner set on your invoice has a complete address (street, city, postal code and country) in Switzerland
    - The option to generate the Swiss QR-code is selected on the invoice (done by default)
    - A correct account number/QR IBAN is set on your bank journal
    - (when using a QR-IBAN): the payment reference of the invoice is a QR-reference

The generation of the ISR and QR-bill is automatic if you meet the previous criteria.

Here is how it works:
    - Printing the invoice will trigger the download of three files: the invoice, its ISR and its QR-bill
    - Clicking the 'Send by mail' button will attach three files to your draft mail : the invoice, the ISR and the QR-bill.
    """,
    'version': '11.1',
    'category': 'Accounting/Localizations/Account Charts',

    'depends': ['account', 'l10n_multilang', 'base_iban'],

    'data': [
        'data/l10n_ch_chart_data.xml',
        'data/account.account.template.csv',
        'data/l10n_ch_chart_post_data.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_vat2011_data.xml',
        'data/account_tax_template_data_2024.xml',
        'data/account_fiscal_position_data.xml',
        'data/account_fiscal_position_data_2024.xml',
        'data/account_chart_template_data.xml',
        'report/isr_report.xml',
        'report/swissqr_report.xml',
        'views/res_bank_view.xml',
        'views/account_invoice_view.xml',
        'views/account_invoice.xml',
        'views/res_config_settings_views.xml',
        'views/setup_wizard_views.xml',
    ],

    'demo': [
        'demo/account_cash_rounding.xml',
        'demo/demo_company.xml',
    ],
    'post_init_hook': 'post_init',
    'assets': {
        'web.report_assets_common': [
            'l10n_ch/static/src/scss/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","reconcile"
"ch_coa_1060","Securities (with stock exchange price)","1060","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1069","Accumulated depreciation on securities","1069","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1091","Transfer account: Salaries","1091","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","True"
"ch_coa_1099","Transfer account: miscellaneous","1099","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","True"
"ch_coa_1100","Accounts receivable from goods and services (Debtors)","1100","account.data_account_type_receivable","l10n_ch.l10nch_chart_template","True"
"ch_coa_1101","Receivable (PoS)","1101","account.data_account_type_receivable","l10n_ch.l10nch_chart_template","True"
"ch_coa_1109","Del credere (Acc. depr. on debtors)","1109","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1140","Advances and loans","1140","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1149","Advances and loans adjustments","1149","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1170","Input Tax (VAT) receivable on material, goods, services, energy","1170","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1171","Input Tax (VAT) receivable on investments, other operating expenses","1171","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1176","Withholding Tax (WT) receivable","1176","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1180","Receivables from social insurances and social security institutions","1180","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1189","Withholding tax","1189","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1190","Other short-term receivables","1190","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1199","Accumulated depreciation on short-terms receivables","1199","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1200","Goods / Merchandise (Trade)","1200","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1207","Accumulated depreciation on Goods / Merchandise (Trade)","1207","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1208","Downpayment on Goods / Merchandise (Trade)","1208","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1209","Correction on Goods / Merchandise (Trade)","1209","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1210","Raw materials","1210","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1217","Accumulated depreciation on raw material","1217","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1218","Downpayment on raw material","1218","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1219","Correction on raw material","1219","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1220","Auxiliary material","1220","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1230","Consumables","1230","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1250","Consignments Goods ","1250","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1260","Finished products","1260","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1267","Accumulated depreciation on Finished products","1267","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1269","Correction on Finished products","1269","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1270","Products in process / Unfinished products","1270","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1277","Accumulated depreciation on Products in process / Unfinished products","1277","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1279","Correction on Products in process / Unfinished products","1279","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1280","Work in progress","1280","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1287","Accumulated depreciation on work in progress","1287","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1289","Correction on work in progress","1289","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1300","Accrued revenue and deferred expense (Accounts paid in advance)","1300","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1301","Deferred expense (Accounts paid in advance)","1301","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1400","Long-term securities","1400","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1409","Accumulated depreciation on long-term securities","1409","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1440","Loan (Asset)","1440","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1441","Mortgages","1441","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1449","Accumulated depreciation on long term receivables","1449","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1480","Participations","1480","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1489","Accumulated depreciation on participations","1489","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1500","Machinery","1500","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1509","Accumulated depreciation on machinery","1509","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1510","Equipment","1510","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1519","Accumulated depreciation on equipment","1519","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1520","Office Equipment (including Information & Communication Technology)","1520","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1529","Accumulated depreciation on office equipment (incl. ICT)","1529","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1530","Vehicles","1530","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1539","Accumulated depreciation on vehicles","1539","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1540","Tools","1540","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1549","Accumulated depreciation on tools","1549","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1550","Warehouse","1550","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1559","Accumulated depreciation on warehouse","1559","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1570","Equipments and Facilities","1570","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1579","Accumulated depreciation on Equipments and Facilities","1579","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1590","Other movable tangible assets","1590","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1599","Accumulated depreciation on Other movable tangible assets","1599","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1600","Real Estate","1600","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1609","Accumulated depreciation on real estate","1609","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1700","Patents, Licences","1700","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1709","Accumulated depreciation on Patents, Licences","1709","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1770","Goodwill","1770","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1779","Accumulated depreciation on goodwill","1779","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1850","Non-paid-in share capital","1850","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_2000","Accounts payable from goods and services (Creditors)","2000","account.data_account_type_payable","l10n_ch.l10nch_chart_template","True"
"ch_coa_2030","Prepayments received","2030","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2100","Bank Overdraft (Bank)","2100","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2120","Leasing bondings","2120","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2140","Other interest-bearing short terms liabilities","2140","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2160","Dettes envers l'actionnaire","2160","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2200","Sales Tax (VAT) owed","2200","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2201","VAT payable","2201","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2206","Withholding Tax (WT) owed","2206","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2208","Direct Taxes","2208","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2210","Others short term liabilities","2210","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2261","Dividend payouts resolved (Dividends)","2261","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2270","Social insurances owed","2270","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2279","Withholding taxes","2279","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2300","Deferred revenue and accrued expenses (Accounts received in advance)","2300","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2301","Deferred revenue (Accounts Received in Advance)","2301","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2330","Short-term provisions","2330","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2400","Bank debts","2400","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2420","Finance lease commitments","2420","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2430","Debentures","2430","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2450","Loans","2450","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2451","Mortgages","2451","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2500","Other long term liabilities","2500","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2600","Long-term provisions","2600","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2800","Share capital","2800","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_2900","Legal capital reserves","2900","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_2940","Valuation Reserves","2940","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_2950","Legal retained earnings (Reserves)","2950","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_2960","Voluntary retained earnings","2960","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_2970","Profits brought forward / Losses brought forward","2970","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_2979","Annual profit or annual loss","2979","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_2980","Treasury stock, shares, participation rights (negative item) ","2980","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_3000","Sales of products (Manufacturing)","3000","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3009","Deductions on sales","3009","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3200","Sales of goods (Trade)","3200","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3400","Revenues from services","3400","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3600","Other revenues","3600","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3700","Own services","3700","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3710","Own consumption","3710","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3800","Financial discount","3800","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3801","Discounts and price reduction","3801","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3802","Rebates","3802","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3803","Third-party commissions","3803","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3804","Collection fees","3804","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3805","Losses from bad debts","3805","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3806","Exchange rate differences","3806","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3807","Shipping & Returns","3807","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3900","Changes in inventories of unfinished and finished products","3900","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3901","Change in inventories of finished goods","3901","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3940","Change in the value of unbilled services","3940","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_4000","Cost of raw materials (Manufacturing)","4000","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4008","Inventory changes","4008","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4009","Deductions obtained on purchases","4009","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4070","Purchase Loans","4070","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4071","Customs duties on importation","4071","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4072","Transport costs at purchase","4072","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4080","Inventory changes","4080","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4086","Loss of material","4086","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4200","Cost of materials (Trade)","4200","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4400","Cost of purchased services","4400","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4500","Electricity","4500","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4510","Gas","4510","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4520","Fuel oil","4520","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4521","Coal, briquettes, wood","4521","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4530","Petrol","4530","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4540","Water","4540","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4800","Change in inventories of goods","4800","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4801","Change in raw material inventories","4801","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4900","Financial Discounts","4900","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4901","Discounts and price reductions","4901","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4092","Rebates","4902","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4903","Commissions on purchases","4903","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4906","Exchange rate differences","4906","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4991","Cash Difference Loss","4991","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4992","Cash Difference Gain","4992","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_5000","Wages and salaries","5000","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_5700","Social benefits","5700","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_5800","Other staff cost","5800","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_5900","Temporary staff expenditures","5900","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6000","Rent","6000","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6100","Maintenance & repair expenses","6100","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6105","Leasing movable tangible fixed assets","6105","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6200","Vehicle expenses","6200","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6260","Vehicules leasing and renting","6260","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6300","Insurance premiums","6300","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6400","Energy expenses & disposal expenses","6400","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6500","Administration expenses","6500","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6570","IT leasing","6570","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6600","Promotion and advertising expenses","6600","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6700","Other operating expenses","6700","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6800","Depreciations","6800","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6900","Financial expenses (Interest expenses, Securities expenses, Participations expenses)","6900","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6950","Financial revenues (Interest revenues, Securities revenues, Participations revenues)","6950","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_7000","Non-core business revenues","7000","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_7010","Non-core business expenses","7010","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_7500","Revenues from operational real estate","7500","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_7510","Expenses from operational real estate","7510","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_8000","Non-operational expenses","8000","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_8100","Non-operational revenues","8100","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_8500","Extraordinary expenses","8500","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_8510","Extraordinary revenues","8510","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_8900","Direct Taxes","8900","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_ch.l10nch_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_fiscal_position_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Fiscal Position Templates -->
        <record id="fiscal_position_template_1" model="account.fiscal.position.template">
            <field name="name">Suisse national</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="country_id" ref="base.ch"/>
        </record>

        <record id="fiscal_position_template_import" model="account.fiscal.position.template">
            <field name="sequence">1</field>
            <field name="name">Import/Export</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="auto_apply" eval="True"/>
        </record>

        <!-- Fiscal Position Tax Templates (pre-2024 rates change) -->
        <record id="fiscal_position_tax_template_3" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"  />
            <field name="tax_src_id" ref="vat_25_purchase" />
            <field name="tax_dest_id" ref="vat_O_import" />
        </record>
        <record id="fiscal_position_tax_template_4" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"  />
            <field name="tax_src_id" ref="vat_25_invest" />
            <field name="tax_dest_id" ref="vat_O_import" />
        </record>

        <record id="fiscal_position_tax_template_5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"  />
            <field name="tax_src_id" ref="vat_37_purchase" />
            <field name="tax_dest_id" ref="vat_O_import" />
        </record>
        <record id="fiscal_position_tax_template_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"  />
            <field name="tax_src_id" ref="vat_37_invest" />
            <field name="tax_dest_id" ref="vat_O_import" />
        </record>


        <record id="fiscal_position_tax_template_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"  />
            <field name="tax_src_id" ref="vat_77_purchase_reverse" />
            <field name="tax_dest_id" ref="vat_O_import" />
        </record>
        <record id="fiscal_position_tax_template_10" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"  />
            <field name="tax_src_id" ref="vat_77_invest" />
            <field name="tax_dest_id" ref="vat_O_import" />
        </record>

        <record id="fiscal_position_tax_template_14" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"  />
            <field name="tax_src_id" ref="vat_25" />
            <field name="tax_dest_id" ref="vat_XO" />
        </record>

        <record id="fiscal_position_tax_template_15" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"  />
            <field name="tax_src_id" ref="vat_37" />
            <field name="tax_dest_id" ref="vat_XO" />
        </record>

        <record id="fiscal_position_tax_template_17" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"  />
            <field name="tax_src_id" ref="vat_77" />
            <field name="tax_dest_id" ref="vat_XO" />
        </record>
    </data>
</odoo>

```

## File: data\account_fiscal_position_data_2024.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Fiscal Position Tax Templates (post-2024 rates change) -->
        <record id="fiscal_position_tax_template_1_2024" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"/>
            <field name="tax_src_id" ref="vat_purchase_26"/>
            <field name="tax_dest_id" ref="vat_O_import"/>
        </record>

        <record id="fiscal_position_tax_template_2_2024" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"/>
            <field name="tax_src_id" ref="vat_purchase_26_invest"/>
            <field name="tax_dest_id" ref="vat_O_import"/>
        </record>

        <record id="fiscal_position_tax_template_3_2024" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"/>
            <field name="tax_src_id" ref="vat_purchase_38"/>
            <field name="tax_dest_id" ref="vat_O_import"/>
        </record>

        <record id="fiscal_position_tax_template_4_2024" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"/>
            <field name="tax_src_id" ref="vat_purchase_38_invest"/>
            <field name="tax_dest_id" ref="vat_O_import"/>
        </record>

        <record id="fiscal_position_tax_template_5_2024" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"/>
            <field name="tax_src_id" ref="vat_purchase_81_reverse"/>
            <field name="tax_dest_id" ref="vat_O_import"/>
        </record>

        <record id="fiscal_position_tax_template_6_2024" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"/>
            <field name="tax_src_id" ref="vat_purchase_81_invest"/>
            <field name="tax_dest_id" ref="vat_O_import"/>
        </record>

        <record id="fiscal_position_tax_template_7_2024" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"/>
            <field name="tax_src_id" ref="vat_sale_26"/>
            <field name="tax_dest_id" ref="vat_XO"/>
        </record>

        <record id="fiscal_position_tax_template_8_2024" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"/>
            <field name="tax_src_id" ref="vat_sale_38"/>
            <field name="tax_dest_id" ref="vat_XO"/>
        </record>

        <record id="fiscal_position_tax_template_9_2024" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import"/>
            <field name="tax_src_id" ref="vat_sale_81"/>
            <field name="tax_dest_id" ref="vat_XO"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Account Tax Group (pre-2024 rates change) -->
        <record id="tax_group_tva_0" model="account.tax.group">
            <field name="name">VAT 0%</field>
            <field name="country_id" ref="base.ch"/>
        </record>
        <record id="tax_group_tva_25" model="account.tax.group">
            <field name="name">VAT 2.5%</field>
            <field name="country_id" ref="base.ch"/>
        </record>
        <record id="tax_group_tva_37" model="account.tax.group">
            <field name="name">VAT 3.7%</field>
            <field name="country_id" ref="base.ch"/>
        </record>
        <record id="tax_group_tva_77" model="account.tax.group">
            <field name="name">VAT 7.7%</field>
            <field name="country_id" ref="base.ch"/>
        </record>
        <record id="tax_group_tva_100" model="account.tax.group">
            <field name="name">VAT 100%</field>
            <field name="country_id" ref="base.ch"/>
        </record>
        <!-- Account Tax Group (post-2024 rates change) -->
        <record id="tax_group_vat_26" model="account.tax.group">
            <field name="name">VAT 2.6%</field>
            <field name="country_id" ref="base.ch"/>
        </record>
        <record id="tax_group_vat_38" model="account.tax.group">
            <field name="name">VAT 3.8%</field>
            <field name="country_id" ref="base.ch"/>
        </record>
        <record id="tax_group_vat_81" model="account.tax.group">
            <field name="name">VAT 8.1%</field>
            <field name="country_id" ref="base.ch"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.ch"/>
    </record>

    <record id="account_tax_report_line_chiffre_af" model="account.tax.report.line">
        <field name="name">I. TURNOVER</field>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
        <field name="sequence" eval="1"/>
    </record>

    <record id="account_tax_report_line_chtax_200" model="account.tax.report.line">
        <field name="name">200 - Total amount of agreed or collected consideration incl. from supplies opted for taxation, transfer of supplies acc. to the notification procedure and supplies provided abroad (worldwide turnover)</field>
        <field name="code">tax_ch_200</field>
        <field name="formula">tax_ch_302a + tax_ch_303a + tax_ch_312a + tax_ch_313a + tax_ch_342a + tax_ch_343a + tax_ch_205 + tax_ch_289</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_205" model="account.tax.report.line">
        <field name="name">205 - Consideration reported in Ref. 200 from supplies exempt from the tax without credit (art. 21) where the option for their taxation according to art. 22 has been exercised</field>
        <field name="code">tax_ch_205</field>
        <field name="tag_name">205</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_220_289" model="account.tax.report.line"> <!-- FIXME in master: the xml is as it is for historical reasons but it does represent box 220 only -->
        <field name="name">220 - Supplies exempt from the tax (e.g. export, art. 23) and supplies provided to institutional and individual beneficiaries that are exempt from liability for tax (art. 107 para. 1 lit. a)</field>
        <field name="code">tax_ch_220</field>
        <field name="tag_name">220</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_221" model="account.tax.report.line">
        <field name="name">221 - Supplies provided abroad</field>
        <field name="code">tax_ch_221</field>
        <field name="tag_name">221</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_225" model="account.tax.report.line">
        <field name="name">225 - Transfer of supplies according to the notification procedure (art. 38, please submit Form 764)</field>
        <field name="code">tax_ch_225</field>
        <field name="tag_name">225</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_230" model="account.tax.report.line">
        <field name="name">230 - Supplies provided on Swiss territory exempt from the tax without credit (art. 21) and where the option for their taxation according to art. 22 has not been exercised</field>
        <field name="code">tax_ch_230</field>
        <field name="tag_name">230</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_235" model="account.tax.report.line">
        <field name="name">235 - Reduction of consideration (discounts, rebates etc.)</field>
        <field name="code">tax_ch_235</field>
        <field name="tag_name">235</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_280" model="account.tax.report.line">
        <field name="name">280 - Miscellaneous (e.g. land value, purchase prices in case of margin taxation)</field>
        <field name="code">tax_ch_280</field>
        <field name="tag_name">280</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_289" model="account.tax.report.line">
        <field name="name">289 - Deductions (Total Ref. 220 to 280)</field>
        <field name="code">tax_ch_289</field>
        <field name="formula">tax_ch_220 + tax_ch_221 + tax_ch_225 + tax_ch_230 + tax_ch_235 + tax_ch_280</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="9"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_299" model="account.tax.report.line">
        <field name="name">299 - Taxable turnover (Ref. 200 minus Ref. 289)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="10"/>
        <field name="formula">tax_ch_200 - tax_ch_289</field>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_calc_impot" model="account.tax.report.line">
        <field name="name">II. TAX CALCULATION</field>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
        <field name="sequence" eval="2"/>
    </record>

    <record id="account_tax_report_line_supplies_1" model="account.tax.report.line">
        <field name="name">Supplies CHF from 01.01.2024</field>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_303a" model="account.tax.report.line">
        <field name="name">303a - Standard rate (8,1%): Supplies CHF from 01.01.2024</field>
        <field name="code">tax_ch_303a</field>
        <field name="tag_name">303a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_1"/>
    </record>

    <record id="account_tax_report_line_chtax_313a" model="account.tax.report.line">
        <field name="name">313a - Reduced rate (2,6%): Supplies CHF from 01.01.2024</field>
        <field name="code">tax_ch_313a</field>
        <field name="tag_name">313a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_1"/>
    </record>

    <record id="account_tax_report_line_chtax_343a" model="account.tax.report.line">
        <field name="name">343a - Accommodation rate (3,8%): Supplies CHF from 01.01.2024</field>
        <field name="code">tax_ch_343a</field>
        <field name="tag_name">343a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_1"/>
    </record>

    <record id="account_tax_report_line_chtax_383a" model="account.tax.report.line">
        <field name="name">383a - Acquisition tax: Supplies CHF from 01.01.2024</field>
        <field name="code">tax_ch_383a</field>
        <field name="tag_name">383a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_1"/>
    </record>

    <record id="account_tax_report_line_supplies_2" model="account.tax.report.line">
        <field name="name">Supplies CHF to 31.12.2023</field>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_302a" model="account.tax.report.line">
        <field name="name">302a - Standard rate (7,7%): Supplies CHF to 31.12.2023</field>
        <field name="code">tax_ch_302a</field>
        <field name="tag_name">302a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_2"/>
    </record>

    <record id="account_tax_report_line_chtax_312a" model="account.tax.report.line">
        <field name="name">312a - Reduced rate (2,5%): Supplies CHF to 31.12.2023</field>
        <field name="code">tax_ch_312a</field>
        <field name="tag_name">312a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_2"/>
    </record>

    <record id="account_tax_report_line_chtax_342a" model="account.tax.report.line">
        <field name="name">342a - Accommodation rate (3,7%): Supplies CHF to 31.12.2023</field>
        <field name="code">tax_ch_342a</field>
        <field name="tag_name">342a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_2"/>
    </record>

    <record id="account_tax_report_line_chtax_382a" model="account.tax.report.line">
        <field name="name">382a - Acquisition tax: Supplies CHF to 31.12.2023</field>
        <field name="code">tax_ch_382a</field>
        <field name="tag_name">382a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_2"/>
    </record>

    <record id="account_tax_report_line_tax_amount_1" model="account.tax.report.line">
        <field name="name">Tax amount CHF / cent. from 01.01.2024</field>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_303b" model="account.tax.report.line">
        <field name="name">303b - Standard rate (8,1%): Tax amount CHF / cent. from 01.01.2024</field>
        <field name="code">tax_ch_303b</field>
        <field name="tag_name">303b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_1"/>
    </record>

    <record id="account_tax_report_line_chtax_313b" model="account.tax.report.line">
        <field name="name">313b - Reduced rate (2,6%): Tax amount CHF / cent. from 01.01.2024</field>
        <field name="code">tax_ch_313b</field>
        <field name="tag_name">313b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_1"/>
    </record>

    <record id="account_tax_report_line_chtax_343b" model="account.tax.report.line">
        <field name="name">343b - Accommodation rate (3,8%): Tax amount CHF / cent. from 01.01.2024</field>
        <field name="code">tax_ch_343b</field>
        <field name="tag_name">343b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_1"/>
    </record>

    <record id="account_tax_report_line_chtax_383b" model="account.tax.report.line">
        <field name="name">383b - Acquisition tax: Tax amount CHF / cent. from 01.01.2024</field>
        <field name="code">tax_ch_383b</field>
        <field name="tag_name">383b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_1"/>
    </record>

    <record id="account_tax_report_line_tax_amount_2" model="account.tax.report.line">
        <field name="name">Tax amount CHF / cent. to 31.12.2023</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_302b" model="account.tax.report.line">
        <field name="name">302b - Standard rate (7,7%): Tax amount CHF / cent. to 31.12.2023</field>
        <field name="code">tax_ch_302b</field>
        <field name="tag_name">302b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_2"/>
    </record>

    <record id="account_tax_report_line_chtax_312b" model="account.tax.report.line">
        <field name="name">312b - Reduced rate (2,5%): Tax amount CHF / cent. to 31.12.2023</field>
        <field name="code">tax_ch_312b</field>
        <field name="tag_name">312b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_2"/>
    </record>

    <record id="account_tax_report_line_chtax_342b" model="account.tax.report.line">
        <field name="name">342b - Accommodation rate (3,7%): Tax amount CHF / cent. to 31.12.2023</field>
        <field name="code">tax_ch_342b</field>
        <field name="tag_name">342b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_2"/>
    </record>

    <record id="account_tax_report_line_chtax_382b" model="account.tax.report.line">
        <field name="name">382b - Acquisition tax: Tax amount CHF / cent. to 31.12.2023</field>
        <field name="code">tax_ch_382b</field>
        <field name="tag_name">382b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_2"/>
    </record>

    <record id="account_tax_report_line_chtax_399" model="account.tax.report.line">
        <field name="name">399 - Total amount of tax due</field>
        <field name="code">tax_ch_399</field>
        <field name="formula">tax_ch_302b + tax_ch_303b + tax_ch_312b + tax_ch_313b + tax_ch_342b + tax_ch_343b + tax_ch_382b + tax_ch_383b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_400" model="account.tax.report.line">
        <field name="name">400 - Input tax on cost of materials and supplies of services</field>
        <field name="code">tax_ch_400</field>
        <field name="tag_name">400</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_405" model="account.tax.report.line">
        <field name="name">405 - Input tax on investments and other operating costs</field>
        <field name="code">tax_ch_405</field>
        <field name="tag_name">405</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_410" model="account.tax.report.line">
        <field name="name">410 - De-taxation (art. 32)</field>
        <field name="code">tax_ch_410</field>
        <field name="tag_name">410</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_415" model="account.tax.report.line">
        <field name="name">415 - Correction of the input tax deduction: mixed use (art. 30), own use (art. 31)</field>
        <field name="code">tax_ch_415</field>
        <field name="tag_name">415</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="9"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_420" model="account.tax.report.line">
        <field name="name">420 - Reduction of the input tax deduction: Flow of funds, which are not deemed to be consideration, such as subsidies, tourist charges (art. 33 para. 2)</field>
        <field name="code">tax_ch_420</field>
        <field name="tag_name">420</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="10"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_479" model="account.tax.report.line">
        <field name="name">479 - Total Ref. 400 to 420</field>
        <field name="code">tax_ch_479</field>
        <field name="formula">tax_ch_400 + tax_ch_405 + tax_ch_410 - tax_ch_415 - tax_ch_420</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="11"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_500" model="account.tax.report.line">
        <field name="name">500 - Amount payable</field>
        <field name="formula">tax_ch_399 - tax_ch_479 &gt; 0 and tax_ch_399 - tax_ch_479 or 0.0</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="12"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_510" model="account.tax.report.line">
        <field name="name">510 - Credit in favour of the taxable person</field>
        <field name="formula">tax_ch_479 - tax_ch_399 &gt; 0 and tax_ch_479 - tax_ch_399 or 0.0</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="13"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_autres_mouv" model="account.tax.report.line">
        <field name="name">III. OTHER CASH FLOWS</field>
        <field name="sequence" eval="3"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_chtax_900" model="account.tax.report.line">
        <field name="name">900 - Subsidies, tourist funds collected by tourist offices, contributions from cantonal water, sewage or waste funds (art. 18 para. 2 lit. a to c)</field>
        <field name="tag_name">900</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_chtax_autres_mouv"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_chtax_910" model="account.tax.report.line">
        <field name="name">910 - Donations, dividends, payments of damages etc. (art. 18 para. 2 lit. d to l)</field>
        <field name="tag_name">910</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_chtax_autres_mouv"/>
        <field name="report_id" ref="tax_report"/>
    </record>
</odoo>

```

## File: data\account_tax_template_data_2024.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Account Tax Templates (post-2024 rates change) -->
    <record model="account.tax.template" id="vat_sale_26">
        <field name="name">2.6% Sales</field>
        <field name="description">2.6%</field>
        <field name="amount" eval="2.6"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_26"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_313a')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_2200'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_313b')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_313a')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_2200'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_313b')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_sale_26_incl">
        <field name="name">2.6% Sales (incl.)</field>
        <field name="description">2.6% incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="2.6"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_26"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_313a')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_2200'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_313b')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_313a')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_2200'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_313b')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_purchase_26">
        <field name="name">2.6% on goods and services</field>
        <field name="description">2.6% purch.</field>
        <field name="amount" eval="2.6"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_26"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_1170'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
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
                'account_id': ref('ch_coa_1170'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_purchase_26_incl">
        <field name="name">2.6% on goods and services (incl.)</field>
        <field name="description">2.6% purch. Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="2.6"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_26"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_1170'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
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
                'account_id': ref('ch_coa_1170'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_purchase_26_invest">
        <field name="name">2.6% on invest. and others expenses</field>
        <field name="description">2.6% invest.</field>
        <field name="amount" eval="2.6"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_26"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_1171'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
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
                'account_id': ref('ch_coa_1171'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_purchase_26_invest_incl">
        <field name="name">2.6% on invest. and others expenses (incl.)</field>
        <field name="description">2.6% invest. Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="2.6"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_26"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_1171'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
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
                'account_id': ref('ch_coa_1171'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_sale_38">
        <field name="name">3.8% Sales</field>
        <field name="description">3.8%</field>
        <field name="amount" eval="3.8"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_38"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_343a')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_2200'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_343b')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_343a')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_2200'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_343b')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_sale_38_incl">
        <field name="name">3.8% Sales (incl.)</field>
        <field name="description">3.8% Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="3.8"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_38"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_343a')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_2200'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_343b')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_343a')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_2200'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_343b')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_purchase_38">
        <field name="name">3.8% on goods and services</field>
        <field name="description">3.8% purch.</field>
        <field name="amount" eval="3.8"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_38"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_1170'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
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
                'account_id': ref('ch_coa_1170'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_purchase_38_incl">
        <field name="name">3.8% on goods and services (incl.)</field>
        <field name="description">3.8% purch. Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="3.8"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_38"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_1170'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
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
                'account_id': ref('ch_coa_1170'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_purchase_38_invest">
        <field name="name">3.8% on invest. and others expenses</field>
        <field name="description">3.8% invest</field>
        <field name="amount" eval="3.8"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_38"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_1171'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
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
                'account_id': ref('ch_coa_1171'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_purchase_38_invest_incl">
        <field name="name">3.8% on invest. and others expenses (incl.)</field>
        <field name="description">3.8% invest Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="3.8"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_38"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_1171'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
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
                'account_id': ref('ch_coa_1171'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_sale_81">
        <field name="name">8.1% Sales</field>
        <field name="description">8.1%</field>
        <field name="amount" eval="8.1"/>
        <field name="sequence" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_81"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_303a')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_2200'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_303b')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_303a')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_2200'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_303b')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_sale_81_incl">
        <field name="name">8.1% Sales (incl.)</field>
        <field name="description">8.1% Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="8.1"/>
        <field name="sequence" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_81"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_303a')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_2200'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_303b')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_303a')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_2200'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_303b')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_purchase_81">
        <field name="name">8.1% on goods and services</field>
        <field name="description">8.1% purch.</field>
        <field name="amount" eval="8.1"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="0"/>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_81"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_1170'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
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
                'account_id': ref('ch_coa_1170'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_purchase_81_incl">
        <field name="name">8.1% on goods and services (incl.)</field>
        <field name="description">8.1% purch. Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="8.1"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="0"/>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_81"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_1170'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
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
                'account_id': ref('ch_coa_1170'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_purchase_81_invest">
        <field name="name">8.1% on invest. and others expenses</field>
        <field name="description">8.1% invest.</field>
        <field name="amount" eval="8.1"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_81"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_1171'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
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
                'account_id': ref('ch_coa_1171'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_purchase_81_invest_incl">
        <field name="name">8.1% on invest. and others expenses (incl.)</field>
        <field name="description">8.1% invest. Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="8.1"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_81"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_1171'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
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
                'account_id': ref('ch_coa_1171'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="vat_purchase_81_return">
        <field name="name">8.1% Purchase (reverse)</field>
        <field name="description">8.1% purch. (reverse)</field>
        <field name="amount" eval="-8.1"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="0"/>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">none</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_383a')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_1170'),
                'minus_report_line_ids': [ref('account_tax_report_line_chtax_383b')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_383a')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('ch_coa_1170'),
                'plus_report_line_ids': [ref('account_tax_report_line_chtax_383b')],
            }),
        ]"/>
    </record>

    <!--# for reverse charge or VAT on Acquisition (group of taxes)-->
    <record model="account.tax.template" id="vat_purchase_81_reverse">
        <field name="name">8.1% on purchase of service abroad (reverse charge)</field>
        <field name="description">8.1% rev</field>
        <field name="amount_type">group</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_81"/>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('vat_purchase_81'), ref('vat_purchase_81_return')])]"/>
    </record>
</odoo>

```

## File: data\account_vat2011_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!--
        #  TVA - Taxe sur la Valeur Ajoutée (pre-2024 rates change)
        -->
        <record model="account.tax.template" id="vat_25">
            <field name="name">2.5% Sales</field>
            <field name="description">2.50%</field>
            <field name="amount" eval="2.5"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_25"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_312a')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_2200'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_312b')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_312a')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_2200'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_312b')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_25_incl">
            <field name="name">2.5% Sales (incl.)</field>
            <field name="description">2.5% Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="2.5"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_25"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_312a')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_2200'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_312b')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_312a')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_2200'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_312b')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_25_purchase">
            <field name="name">2.5% on goods and services</field>
            <field name="description">2.5% purch.</field>
            <field name="amount" eval="2.5"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_25"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1170'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
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
                    'account_id': ref('ch_coa_1170'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_25_purchase_incl">
            <field name="name">2.5% on goods and services (incl.)</field>
            <field name="description">2.5% purch. Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="2.5"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_25"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1170'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
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
                    'account_id': ref('ch_coa_1170'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_25_invest">
            <field name="name">2.5% on invest. and others expenses</field>
            <field name="description">2.5% invest.</field>
            <field name="amount" eval="2.5"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_25"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1171'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
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
                    'account_id': ref('ch_coa_1171'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_25_invest_incl">
            <field name="name">2.5% on invest. and others expenses (incl.)</field>
            <field name="description">2.5% invest. Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="2.5"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_25"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1171'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
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
                    'account_id': ref('ch_coa_1171'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_37">
            <field name="name">3.7% Sales</field>
            <field name="description">3.70%%</field>
            <field name="amount" eval="3.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_37"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_342a')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_2200'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_342b')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_342a')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_2200'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_342b')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_37_incl">
            <field name="name">3.7% Sales (incl.)</field>
            <field name="description">3.7% Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="3.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_37"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_342a')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_2200'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_342b')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_342a')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_2200'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_342b')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_37_purchase">
            <field name="name">3.7% on goods and services</field>
            <field name="description">3.7% purch.</field>
            <field name="amount" eval="3.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_37"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1170'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
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
                    'account_id': ref('ch_coa_1170'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_37_purchase_incl">
            <field name="name">3.7% on goods and services (incl.)</field>
            <field name="description">3.7% purch. Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="3.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_37"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1170'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
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
                    'account_id': ref('ch_coa_1170'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_37_invest">
            <field name="name">3.7% on invest. and others expenses</field>
            <field name="description">3.7% invest</field>
            <field name="amount" eval="3.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_37"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1171'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
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
                    'account_id': ref('ch_coa_1171'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_37_invest_incl">
            <field name="name">3.7% on invest. and others expenses (incl.)</field>
            <field name="description">3.7% invest Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="3.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_37"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1171'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
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
                    'account_id': ref('ch_coa_1171'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_77">
            <field name="name">7.7% Sales</field>
            <field name="description">7.70%</field>
            <field name="amount" eval="7.7"/>
            <field name="sequence" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_77"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_302a')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_2200'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_302b')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_302a')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_2200'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_302b')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_77_incl">
            <field name="name">7.7% Sales (incl.)</field>
            <field name="description">7.7% Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="7.7"/>
            <field name="sequence" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_77"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_302a')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_2200'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_302b')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_302a')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_2200'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_302b')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_77_purchase_incl">
            <field name="name">7.7% on goods and services (incl.)</field>
            <field name="description">7.7% purch. Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="7.7"/>
            <field name="amount_type">percent</field>
            <field name="sequence" eval="0"/>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_77"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1170'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
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
                    'account_id': ref('ch_coa_1170'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_77_invest">
            <field name="name">7.7% on invest. and others expenses</field>
            <field name="description">7.7% invest.</field>
            <field name="amount" eval="7.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_77"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1171'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
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
                    'account_id': ref('ch_coa_1171'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_77_invest_incl">
            <field name="name">7.7% on invest. and others expenses (incl.)</field>
            <field name="description">7.7% invest. Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="7.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_77"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1171'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
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
                    'account_id': ref('ch_coa_1171'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_XO">
            <field name="name">0% Export</field>
            <field name="amount" eval="0.00"/>
            <field name="description">0%</field>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_220_289')],
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
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_220_289')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_O_exclude">
            <field name="name">0% Excluded</field>
            <field name="description">0% excl.</field>
            <field name="amount" eval="0.00"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_230')],
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
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_230')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_O_import">
            <field name="name">0% Import</field>
            <field name="description">0% import.</field>
            <field name="amount" eval="0.00"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
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

        <record model="account.tax.template" id="vat_100_import">
            <field name="name">Customs VAT on goods and services</field>
            <field name="description">100% imp.</field>
            <field name="amount" eval="100"/>
            <field name="amount_type">division</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_100"/>
            <field name="price_include" eval="1"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1170'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
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
                    'account_id': ref('ch_coa_1170'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_100_import_invest">
            <field name="name">Customs VAT on invest. and others expenses</field>
            <field name="description">100% imp.invest.</field>
            <field name="amount" eval="100"/>
            <field name="amount_type">division</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_100"/>
            <field name="price_include" eval="1"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1171'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
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
                    'account_id': ref('ch_coa_1171'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_405')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_77_purchase_return">
            <field name="name">7.7% Purchase (reverse)</field>
            <field name="description">7.7% purch. (reverse)</field>
            <field name="amount" eval="-7.7"/>
            <field name="amount_type">percent</field>
            <field name="sequence" eval="0"/>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_77"/>
            <field name="type_tax_use">none</field>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_382a')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1170'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_382b')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_382a')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1170'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_382b')],
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_77_purchase">
            <field name="name">7.7% on goods and services</field>
            <field name="description">7.7% purch.</field>
            <field name="amount" eval="7.7"/>
            <field name="amount_type">percent</field>
            <field name="sequence" eval="0"/>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_77"/>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('ch_coa_1170'),
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
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
                    'account_id': ref('ch_coa_1170'),
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_400')],
                }),
            ]"/>
        </record>

         <!--# for reverse charge or VAT on Acquisition (group of taxes)-->
        <record model="account.tax.template" id="vat_77_purchase_reverse">
            <field name="description">7.7% rev.</field>
            <field name="name">7.7% on purchase of service abroad (reverse charge)</field>
            <field name="amount_type">group</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_77"/>
            <field name="active" eval="False"/>
            <field name="children_tax_ids" eval="[(6, 0, [ref('vat_77_purchase_return'), ref('vat_77_purchase')])]"/>
        </record>


        <!-- Taxes for other movements -->
        <record model="account.tax.template" id="vat_other_movements_900">
            <field name="name">0% - Subsidies, tourist taxes</field>
            <field name="amount" eval="0.00"/>
            <field name="description">0% subventions</field>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_900')],
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
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_900')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record model="account.tax.template" id="vat_other_movements_910">
            <field name="name">0% - Donations, dividends, compensation</field>
            <field name="amount" eval="0.00"/>
            <field name="description">0% dons</field>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_chtax_910')],
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
                    'minus_report_line_ids': [ref('account_tax_report_line_chtax_910')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>
</odoo>

```

## File: data\l10n_ch_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_ch_statements_menu" name="Switzerland" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>

    <data>
         <!-- Account Tags -->
        <record id="l10nch_chart_template" model="account.chart.template">
            <field name="name">Plan comptable 2015 (Suisse)</field>
            <field name="code_digits">4</field>
            <field name="bank_account_code_prefix">102</field>
            <field name="cash_account_code_prefix">100</field>
            <field name="transfer_account_code_prefix">1090</field>
            <field name="currency_id" ref="base.CHF"/>
            <field name="country_id" ref="base.ch"/>
            <field name="spoken_languages" eval="'it_IT;de_DE;de_CH;fr_FR;fr_CH'"/>
        </record>
    </data>
</odoo>

```

## File: data\l10n_ch_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10nch_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="ch_coa_1100"/>
        <field name="property_account_payable_id" ref="ch_coa_2000"/>
        <field name="property_account_expense_categ_id" ref="ch_coa_4200"/>
        <field name="property_account_income_categ_id" ref="ch_coa_3200"/>
        <field name="income_currency_exchange_account_id" ref="ch_coa_3806"/>
        <field name="expense_currency_exchange_account_id" ref="ch_coa_4906"/>
        <field name="default_pos_receivable_account_id" ref="ch_coa_1101" />
        <field name="default_cash_difference_expense_account_id" ref="ch_coa_4991"/>
        <field name="default_cash_difference_income_account_id" ref="ch_coa_4992"/>
    </record>
</odoo>

```

## File: migrations\0.0.0\pre-migrate-qr-template.py

```python
# -*- coding: utf-8 -*-


def migrate(cr, version):
    """ From 12.0, to saas-13.3, l10n_ch_swissqr_template
    used to inherit from another template. This isn't the case
    anymore since https://github.com/odoo/odoo/commit/719f087b1b5be5f1f276a0f87670830d073f6ef4
    (made in 12.0, and forward-ported). The module will not be updatable if we
    don't manually clean inherit_id.
    """
    cr.execute("""
        update ir_ui_view v
        set inherit_id = NULL, mode='primary'
        from ir_model_data mdata
        where
        v.id = mdata.res_id
        and mdata.model= 'ir.ui.view'
        and mdata.name = 'l10n_ch_swissqr_template'
        and mdata.module='l10n_ch';
    """)
```

## File: migrations\11.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID
from odoo.addons.account.models.chart_template import update_taxes_from_templates


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    # We had corrupted data, handle the correction so the tax update can proceed.
    # See https://github.com/odoo/odoo/commit/7b07df873535446f97abc1de9176b9332de5cb07
    for company in env.companies:
        taxes_to_check = (f'{company.id}_vat_purchase_81_reverse', f'{company.id}_vat_77_purchase_reverse')
        tax_ids = env['ir.model.data'].search([
            ('name', 'in', taxes_to_check),
            ('model', '=', 'account.tax'),
        ]).mapped('res_id')
        for tax in env['account.tax'].browse(tax_ids).with_context(active_test=False):
            for child in tax.children_tax_ids:
                if child.type_tax_use not in ('none', tax.type_tax_use):
                    # set the child to it's parent's value
                    child.type_tax_use = tax.type_tax_use

    # Update taxes
    update_taxes_from_templates(cr, 'l10n_ch.l10nch_chart_template')

```

## File: migrations\9.0.9.0\pre-set_tags_and_taxes_updatable.py

```python
# -*- coding: utf-8 -*-

import odoo

def migrate(cr, version):
    registry = odoo.registry(cr.dbname)
    from odoo.addons.account.models.chart_template import migrate_set_tags_and_taxes_updatable
    migrate_set_tags_and_taxes_updatable(cr, registry, 'l10n_ch')


```

## File: models\account_bank_statement.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _
from odoo.addons.l10n_ch.models.res_bank import _is_l10n_ch_postal

class AccountBankStatementLine(models.Model):

    _inherit = "account.bank.statement.line"

    def _find_or_create_bank_account(self):
        if self.company_id.account_fiscal_country_id.code in ('CH', 'LI') and _is_l10n_ch_postal(self.account_number):
            bank_account = self.env['res.partner.bank'].search(
                [('company_id', '=', self.company_id.id),
                 ('sanitized_acc_number', 'like', self.account_number + '%'),
                 ('partner_id', '=', self.partner_id.id)])
            if not bank_account:
                bank_account = self.env['res.partner.bank'].create({
                    'company_id': self.company_id.id,
                    'acc_number': self.account_number + " " + self.partner_id.name,
                    'partner_id': self.partner_id.id
                })
            return bank_account
        else:
            return super()._find_or_create_bank_account()

```

## File: models\account_invoice.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re

from odoo import models, fields, api, _
from odoo.exceptions import ValidationError, UserError
from odoo.tools.float_utils import float_split_str
from odoo.tools.misc import mod10r


l10n_ch_ISR_NUMBER_LENGTH = 27
l10n_ch_ISR_ID_NUM_LENGTH = 6

class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_ch_isr_subscription = fields.Char(compute='_compute_l10n_ch_isr_subscription', help='ISR subscription number identifying your company or your bank to generate ISR.')
    l10n_ch_isr_subscription_formatted = fields.Char(compute='_compute_l10n_ch_isr_subscription', help="ISR subscription number your company or your bank, formated with '-' and without the padding zeros, to generate ISR report.")

    l10n_ch_isr_number = fields.Char(compute='_compute_l10n_ch_isr_number', store=True, help='The reference number associated with this invoice')
    l10n_ch_isr_number_spaced = fields.Char(compute='_compute_l10n_ch_isr_number_spaced', help="ISR number split in blocks of 5 characters (right-justified), to generate ISR report.")

    l10n_ch_isr_optical_line = fields.Char(compute="_compute_l10n_ch_isr_optical_line", help='Optical reading line, as it will be printed on ISR')

    l10n_ch_isr_valid = fields.Boolean(compute='_compute_l10n_ch_isr_valid', help='Boolean value. True iff all the data required to generate the ISR are present')

    l10n_ch_isr_sent = fields.Boolean(default=False, help="Boolean value telling whether or not the ISR corresponding to this invoice has already been printed or sent by mail.")
    l10n_ch_currency_name = fields.Char(related='currency_id.name', readonly=True, string="Currency Name", help="The name of this invoice's currency") #This field is used in the "invisible" condition field of the 'Print ISR' button.
    l10n_ch_isr_needs_fixing = fields.Boolean(compute="_compute_l10n_ch_isr_needs_fixing", help="Used to show a warning banner when the vendor bill needs a correct ISR payment reference. ")

    @api.depends('partner_bank_id.l10n_ch_isr_subscription_eur', 'partner_bank_id.l10n_ch_isr_subscription_chf')
    def _compute_l10n_ch_isr_subscription(self):
        """ Computes the ISR subscription identifying your company or the bank that allows to generate ISR. And formats it accordingly"""
        def _format_isr_subscription(isr_subscription):
            #format the isr as per specifications
            currency_code = isr_subscription[:2]
            middle_part = isr_subscription[2:-1]
            trailing_cipher = isr_subscription[-1]
            middle_part = re.sub('^0*', '', middle_part)
            return currency_code + '-' + middle_part + '-' + trailing_cipher

        def _format_isr_subscription_scanline(isr_subscription):
            # format the isr for scanline
            return isr_subscription[:2] + isr_subscription[2:-1].rjust(6, '0') + isr_subscription[-1:]

        for record in self:
            record.l10n_ch_isr_subscription = False
            record.l10n_ch_isr_subscription_formatted = False
            if record.partner_bank_id:
                if record.currency_id.name == 'EUR':
                    isr_subscription = record.partner_bank_id.l10n_ch_isr_subscription_eur
                elif record.currency_id.name == 'CHF':
                    isr_subscription = record.partner_bank_id.l10n_ch_isr_subscription_chf
                else:
                    #we don't format if in another currency as EUR or CHF
                    continue

                if isr_subscription:
                    isr_subscription = isr_subscription.replace("-", "")  # In case the user put the -
                    record.l10n_ch_isr_subscription = _format_isr_subscription_scanline(isr_subscription)
                    record.l10n_ch_isr_subscription_formatted = _format_isr_subscription(isr_subscription)

    def _get_isrb_id_number(self):
        """Hook to fix the lack of proper field for ISR-B Customer ID"""
        # FIXME
        # replace l10n_ch_postal by an other field to not mix ISR-B
        # customer ID as it forbid the following validations on l10n_ch_postal
        # number for Vendor bank accounts:
        # - validation of format xx-yyyyy-c
        # - validation of checksum
        return self.partner_bank_id.l10n_ch_postal or ''

    @api.depends('name', 'partner_bank_id.l10n_ch_postal')
    def _compute_l10n_ch_isr_number(self):
        for record in self:
            if (record.partner_bank_id.l10n_ch_qr_iban or record.l10n_ch_isr_subscription) and record.name:
                invoice_ref = re.sub(r'\D', '', record.name)
                record.l10n_ch_isr_number = record._compute_isr_number(invoice_ref)
            else:
                record.l10n_ch_isr_number = False

    @api.model
    def _compute_isr_number(self, invoice_ref):
        r"""Generates the ISR or QRR reference

        An ISR references are 27 characters long.
        QRR is a recycling of ISR for QR-bills. Thus works the same.

        The invoice sequence number is used, removing each of its non-digit characters,
        and pad the unused spaces on the left of this number with zeros.
        The last digit is a checksum (mod10r).

        There are 2 types of references:

        * ISR (Postfinance)

            The reference is free but for the last
            digit which is a checksum.
            If shorter than 27 digits, it is filled with zeros on the left.

            e.g.

                120000000000234478943216899
                \________________________/|
                         1                2
                (1) 12000000000023447894321689 | reference
                (2) 9: control digit for identification number and reference

        * ISR-B (Indirect through a bank, requires a customer ID)

            In case of ISR-B The firsts digits (usually 6), contain the customer ID
            at the Bank of this ISR's issuer.
            The rest (usually 20 digits) is reserved for the reference plus the
            control digit.
            If the [customer ID] + [the reference] + [the control digit] is shorter
            than 27 digits, it is filled with zeros between the customer ID till
            the start of the reference.

            e.g.

                150001123456789012345678901
                \____/\__________________/|
                   1           2          3
                (1) 150001 | id number of the customer (size may vary)
                (2) 12345678901234567890 | reference
                (3) 1: control digit for identification number and reference
        """
        id_number = self._get_isrb_id_number()
        if id_number:
            id_number = id_number.zfill(l10n_ch_ISR_ID_NUM_LENGTH)
        # keep only the last digits if it exceed boundaries
        full_len = len(id_number) + len(invoice_ref)
        ref_payload_len = l10n_ch_ISR_NUMBER_LENGTH - 1
        extra = full_len - ref_payload_len
        if extra > 0:
            invoice_ref = invoice_ref[extra:]
        internal_ref = invoice_ref.zfill(ref_payload_len - len(id_number))

        return mod10r(id_number + internal_ref)

    @api.depends('l10n_ch_isr_number')
    def _compute_l10n_ch_isr_number_spaced(self):
        def _space_isr_number(isr_number):
            to_treat = isr_number
            res = ''
            while to_treat:
                res = to_treat[-5:] + res
                to_treat = to_treat[:-5]
                if to_treat:
                    res = ' ' + res
            return res

        for record in self:
            if record.l10n_ch_isr_number:
                record.l10n_ch_isr_number_spaced = _space_isr_number(record.l10n_ch_isr_number)
            else:
                record.l10n_ch_isr_number_spaced = False

    def _get_l10n_ch_isr_optical_amount(self):
        """Prepare amount string for ISR optical line"""
        self.ensure_one()
        currency_code = None
        if self.currency_id.name == 'CHF':
            currency_code = '01'
        elif self.currency_id.name == 'EUR':
            currency_code = '03'
        units, cents = float_split_str(self.amount_residual, 2)
        amount_to_display = units + cents
        amount_ref = amount_to_display.zfill(10)
        optical_amount = currency_code + amount_ref
        optical_amount = mod10r(optical_amount)
        return optical_amount

    @api.depends(
        'currency_id.name', 'amount_residual', 'name',
        'partner_bank_id.l10n_ch_isr_subscription_eur',
        'partner_bank_id.l10n_ch_isr_subscription_chf')
    def _compute_l10n_ch_isr_optical_line(self):
        r""" Compute the optical line to print on the bottom of the ISR.

        This line is read by an OCR.
        It's format is:

            amount>reference+ creditor>

        Where:

           - amount: currency and invoice amount
           - reference: ISR structured reference number
                - in case of ISR-B contains the Customer ID number
                - it can also contains a partner reference (of the debitor)
           - creditor: Subscription number of the creditor

        An optical line can have the 2 following formats:

        * ISR (Postfinance)

            0100003949753>120000000000234478943216899+ 010001628>
            |/\________/| \________________________/|  \_______/
            1     2     3          4                5      6

            (1) 01 | currency
            (2) 0000394975 | amount 3949.75
            (3) 4 | control digit for amount
            (5) 12000000000023447894321689 | reference
            (6) 9: control digit for identification number and reference
            (7) 010001628: subscription number (01-162-8)

        * ISR-B (Indirect through a bank, requires a customer ID)

            0100000494004>150001123456789012345678901+ 010234567>
            |/\________/| \____/\__________________/|  \_______/
            1     2     3    4           5          6      7

            (1) 01 | currency
            (2) 0000049400 | amount 494.00
            (3) 4 | control digit for amount
            (4) 150001 | id number of the customer (size may vary, usually 6 chars)
            (5) 12345678901234567890 | reference
            (6) 1: control digit for identification number and reference
            (7) 010234567: subscription number (01-23456-7)
        """
        for record in self:
            record.l10n_ch_isr_optical_line = ''
            if record.l10n_ch_isr_number and record.l10n_ch_isr_subscription and record.currency_id.name:
                # Final assembly (the space after the '+' is no typo, it stands in the specs.)
                record.l10n_ch_isr_optical_line = '{amount}>{reference}+ {creditor}>'.format(
                    amount=record._get_l10n_ch_isr_optical_amount(),
                    reference=record.l10n_ch_isr_number,
                    creditor=record.l10n_ch_isr_subscription,
                )

    @api.depends(
        'move_type', 'name', 'currency_id.name',
        'partner_bank_id.l10n_ch_isr_subscription_eur',
        'partner_bank_id.l10n_ch_isr_subscription_chf')
    def _compute_l10n_ch_isr_valid(self):
        """Returns True if all the data required to generate the ISR are present"""
        for record in self:
            record.l10n_ch_isr_valid = record.move_type == 'out_invoice' and\
                record.name and \
                record.l10n_ch_isr_subscription and \
                record.l10n_ch_currency_name in ['EUR', 'CHF']

    @api.depends('move_type', 'partner_bank_id', 'payment_reference')
    def _compute_l10n_ch_isr_needs_fixing(self):
        for inv in self:
            if inv.move_type == 'in_invoice' and inv.company_id.account_fiscal_country_id.code in ('CH', 'LI'):
                partner_bank = inv.partner_bank_id
                needs_isr_ref = partner_bank.l10n_ch_qr_iban or partner_bank._is_isr_issuer()
                if needs_isr_ref and not inv._has_isr_ref():
                    inv.l10n_ch_isr_needs_fixing = True
                    continue
            inv.l10n_ch_isr_needs_fixing = False

    def _has_isr_ref(self):
        """Check if this invoice has a valid ISR reference (for Switzerland)
        e.g.
        12371
        000000000000000000000012371
        210000000003139471430009017
        21 00000 00003 13947 14300 09017
        """
        self.ensure_one()
        ref = self.payment_reference or self.ref
        if not ref:
            return False
        ref = ref.replace(' ', '')
        if re.match(r'^(\d{2,27})$', ref):
            return ref == mod10r(ref[:-1])
        return False

    def split_total_amount(self):
        """ Splits the total amount of this invoice in two parts, using the dot as
        a separator, and taking two precision digits (always displayed).
        These two parts are returned as the two elements of a tuple, as strings
        to print in the report.

        This function is needed on the model, as it must be called in the report
        template, which cannot reference static functions
        """
        return float_split_str(self.amount_residual, 2)

    def isr_print(self):
        """ Triggered by the 'Print ISR' button.
        This button isn't available anymore and will be removed in 16.2.
        This function is kept for stable policy.
        """
        self.ensure_one()
        if self.l10n_ch_isr_valid:
            self.l10n_ch_isr_sent = True
            return self.env.ref('l10n_ch.l10n_ch_isr_report').report_action(self)
        else:
            raise ValidationError(_("""You cannot generate an ISR yet.\n
                                   For this, you need to :\n
                                   - set a valid postal account number (or an IBAN referencing one) for your company\n
                                   - define its bank\n
                                   - associate this bank with a postal reference for the currency used in this invoice\n
                                   - fill the 'bank account' field of the invoice with the postal to be used to receive the related payment. A default account will be automatically set for all invoices created after you defined a postal account for your company."""))

    def print_ch_qr_bill(self):
        """ Triggered by the 'Print QR-bill' button.
        """
        self.ensure_one()

        if not self.partner_bank_id:
            raise UserError(_("QR-Bill can not be generated on paid invoices. If the invoice is not fully paid, please make sure Recipient Bank field is not empty and try again."))

        if not self.partner_bank_id._eligible_for_qr_code('ch_qr', self.partner_id, self.currency_id):
            raise UserError(_("Cannot generate the QR-bill. Please check you have configured the address of your company and debtor. If you are using a QR-IBAN, also check the invoice's payment reference is a QR reference."))

        self.l10n_ch_isr_sent = True
        return self.env.ref('l10n_ch.l10n_ch_qr_report').report_action(self)

    def action_invoice_sent(self):
        # OVERRIDE
        rslt = super(AccountMove, self).action_invoice_sent()

        if self.l10n_ch_isr_valid:
            rslt['context']['l10n_ch_mark_isr_as_sent'] = True

        return rslt

    @api.returns('mail.message', lambda value: value.id)
    def message_post(self, **kwargs):
        if self.env.context.get('l10n_ch_mark_isr_as_sent'):
            self.filtered(lambda inv: not inv.l10n_ch_isr_sent).write({'l10n_ch_isr_sent': True})
        return super(AccountMove, self.with_context(mail_post_autofollow=self.env.context.get('mail_post_autofollow', True))).message_post(**kwargs)

    def _get_invoice_reference_ch_invoice(self):
        """ This sets ISR reference number which is generated based on customer's `Bank Account` and set it as
        `Payment Reference` of the invoice when invoice's journal is using Switzerland's communication standard
        """
        self.ensure_one()
        return self.l10n_ch_isr_number

    def _get_invoice_reference_ch_partner(self):
        """ This sets ISR reference number which is generated based on customer's `Bank Account` and set it as
        `Payment Reference` of the invoice when invoice's journal is using Switzerland's communication standard
        """
        self.ensure_one()
        return self.l10n_ch_isr_number

    @api.model
    def space_qrr_reference(self, qrr_ref):
        """ Makes the provided QRR reference human-friendly, spacing its elements
        by blocks of 5 from right to left.
        """
        spaced_qrr_ref = ''
        i = len(qrr_ref) # i is the index after the last index to consider in substrings
        while i > 0:
            spaced_qrr_ref = qrr_ref[max(i-5, 0) : i] + ' ' + spaced_qrr_ref
            i -= 5

        return spaced_qrr_ref

    @api.model
    def space_scor_reference(self, iso11649_ref):
        """ Makes the provided SCOR reference human-friendly, spacing its elements
        by blocks of 5 from right to left.
        """

        return ' '.join(iso11649_ref[i:i + 4] for i in range(0, len(iso11649_ref), 4))

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api

from odoo.exceptions import ValidationError

from odoo.addons.base_iban.models.res_partner_bank import validate_iban
from odoo.addons.base.models.res_bank import sanitize_account_number


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    invoice_reference_model = fields.Selection(selection_add=[
        ('ch', 'Switzerland')
    ], ondelete={'ch': lambda recs: recs.write({'invoice_reference_model': 'odoo'})})

    def _process_reference_for_sale_order(self, order_reference):
        '''
        returns the order reference to be used for the payment respecting the ISR
        '''
        self.ensure_one()
        if self.invoice_reference_model == 'ch':
            # converting the sale order name into a unique number. Letters are converted to their base10 value
            invoice_ref = "".join([a if a.isdigit() else str(ord(a)) for a in order_reference])
            # id_number = self.company_id.bank_ids.l10n_ch_postal or ''
            order_reference = self.env['account.move']._compute_isr_number(invoice_ref)
            return order_reference
        return super()._process_reference_for_sale_order(order_reference)

```

## File: models\ir_actions_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, models

from pathlib import Path
from reportlab.graphics.shapes  import Image as ReportLabImage
from reportlab.lib.units import mm

CH_QR_CROSS_SIZE_RATIO = 0.1522 # Ratio between the side length of the Swiss QR-code cross image and the QR-code's
CH_QR_CROSS_FILE = Path('../static/src/img/CH-Cross_7mm.png') # Image file containing the Swiss QR-code cross to add on top of the QR-code

class IrActionsReport(models.Model):
    _inherit = 'ir.actions.report'

    @api.model
    def get_available_barcode_masks(self):
        rslt = super(IrActionsReport, self).get_available_barcode_masks()
        rslt['ch_cross'] = self.apply_qr_code_ch_cross_mask
        return rslt

    @api.model
    def apply_qr_code_ch_cross_mask(self, width, height, barcode_drawing):
        cross_width = CH_QR_CROSS_SIZE_RATIO * width
        cross_height = CH_QR_CROSS_SIZE_RATIO * height
        cross_path = Path(__file__).absolute().parent / CH_QR_CROSS_FILE
        qr_cross = ReportLabImage((width/2 - cross_width/2) / mm, (height/2 - cross_height/2) / mm, cross_width / mm, cross_height / mm, cross_path.as_posix())
        barcode_drawing.add(qr_cross)

```

## File: models\mail_template.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import base64

from odoo import api, models


class MailTemplate(models.Model):
    _inherit = 'mail.template'

    def generate_email(self, res_ids, fields):
        """ Method overridden in order to add an attachment containing the ISR
        to the draft message when opening the 'send by mail' wizard on an invoice.
        This attachment generation will only occur if all the required data are
        present on the invoice. Otherwise, no ISR attachment will be created, and
        the mail will only contain the invoice (as defined in the mother method).
        """
        result = super(MailTemplate, self).generate_email(res_ids, fields)
        if self.model != 'account.move':
            return result

        multi_mode = True
        if isinstance(res_ids, int):
            res_ids = [res_ids]
            multi_mode = False

        if self.model == 'account.move':
            for record in self.env[self.model].browse(res_ids):
                inv_print_name = self._render_field('report_name', record.ids, compute_lang=True)[record.id]
                new_attachments = []

                include_qr_report = 'l10n_ch.l10n_ch_qr_report' not in self.env.context.get('l10n_ch_mail_skip_report', [])
                if include_qr_report and record.move_type == 'out_invoice' and record.partner_bank_id._eligible_for_qr_code('ch_qr', record.partner_id, record.currency_id):
                    # We add an attachment containing the QR-bill
                    qr_report_name = 'QR-bill-' + inv_print_name + '.pdf'
                    qr_pdf = self.env.ref('l10n_ch.l10n_ch_qr_report')._render_qweb_pdf(record.ids)[0]
                    qr_pdf = base64.b64encode(qr_pdf)
                    new_attachments.append((qr_report_name, qr_pdf))

                record_dict = multi_mode and result[record.id] or result
                attachments_list = record_dict.get('attachments', False)
                if attachments_list:
                    attachments_list.extend(new_attachments)
                else:
                    record_dict['attachments'] = new_attachments
        return result

```

## File: models\res_bank.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

import re
from stdnum.util import clean

from odoo import api, fields, models, _
from odoo.addons.base.models.res_bank import sanitize_account_number
from odoo.addons.base_iban.models.res_partner_bank import normalize_iban, pretty_iban, validate_iban
from odoo.exceptions import ValidationError
from odoo.tools.misc import mod10r


ISR_SUBSCRIPTION_CODE = {'CHF': '01', 'EUR': '03'}
CLEARING = "09000"
_re_postal = re.compile('^[0-9]{2}-[0-9]{1,6}-[0-9]$')


def _is_l10n_ch_postal(account_ref):
    """ Returns True if the string account_ref is a valid postal account number,
    i.e. it only contains ciphers and is last cipher is the result of a recursive
    modulo 10 operation ran over the rest of it. Shorten form with - is also accepted.
    """
    if _re_postal.match(account_ref or ''):
        ref_subparts = account_ref.split('-')
        account_ref = ref_subparts[0] + ref_subparts[1].rjust(6, '0') + ref_subparts[2]

    if re.match(r'\d+$', account_ref or ''):
        account_ref_without_check = account_ref[:-1]
        return mod10r(account_ref_without_check) == account_ref
    return False

def _is_l10n_ch_isr_issuer(account_ref, currency_code):
    """ Returns True if the string account_ref is a valid a valid ISR issuer
    An ISR issuer is postal account number that starts by 01 (CHF) or 03 (EUR),
    """
    if (account_ref or '').startswith(ISR_SUBSCRIPTION_CODE[currency_code]):
        return _is_l10n_ch_postal(account_ref)
    return False

def validate_qr_iban(qr_iban):
    # Check first if it's a valid IBAN.
    validate_iban(qr_iban)

    # We sanitize first so that _check_qr_iban_range() can extract correct IID from IBAN to validate it.
    sanitized_qr_iban = sanitize_account_number(qr_iban)

    if sanitized_qr_iban[:2] not in ['CH', 'LI']:
        raise ValidationError(_("QR-IBAN numbers are only available in Switzerland."))

    # Now, check if it's valid QR-IBAN (based on its IID).
    if not check_qr_iban_range(sanitized_qr_iban):
        raise ValidationError(_("QR-IBAN '%s' is invalid.") % qr_iban)

    return True

def check_qr_iban_range(iban):
    if not iban or len(iban) < 9:
        return False
    iid_start_index = 4
    iid_end_index = 8
    iid = iban[iid_start_index : iid_end_index+1]
    return re.match(r'\d+', iid) and 30000 <= int(iid) <= 31999 # Those values for iid are reserved for QR-IBANs only


class ResPartnerBank(models.Model):
    _inherit = 'res.partner.bank'

    l10n_ch_postal = fields.Char(
        string="Swiss Postal Account",
        readonly=False, store=True,
        compute='_compute_l10n_ch_postal',
        help="This field is used for the Swiss postal account number on a vendor account and for the client number on "
             "your own account. The client number is mostly 6 numbers without -, while the postal account number can "
             "be e.g. 01-162-8")

    l10n_ch_qr_iban = fields.Char(string='QR-IBAN',
                                  compute='_compute_l10n_ch_qr_iban',
                                  store=True,
                                  readonly=False,
                                  help="Put the QR-IBAN here for your own bank accounts.  That way, you can "
                                       "still use the main IBAN in the Account Number while you will see the "
                                       "QR-IBAN for the barcode.  ")

    # fields to configure ISR payment slip generation
    l10n_ch_isr_subscription_chf = fields.Char(string='CHF ISR Subscription Number', help='The subscription number provided by the bank or Postfinance to identify the bank, used to generate ISR in CHF. eg. 01-162-8')
    l10n_ch_isr_subscription_eur = fields.Char(string='EUR ISR Subscription Number', help='The subscription number provided by the bank or Postfinance to identify the bank, used to generate ISR in EUR. eg. 03-162-5')
    l10n_ch_show_subscription = fields.Boolean(compute='_compute_l10n_ch_show_subscription', default=lambda self: self.env.company.account_fiscal_country_id.code == 'CH')

    def _is_isr_issuer(self):
        return (_is_l10n_ch_isr_issuer(self.l10n_ch_postal, 'CHF')
                or _is_l10n_ch_isr_issuer(self.l10n_ch_postal, 'EUR'))

    @api.constrains("l10n_ch_postal", "partner_id")
    def _check_postal_num(self):
        """Validate postal number format"""
        for rec in self:
            if rec.l10n_ch_postal and not _is_l10n_ch_postal(rec.l10n_ch_postal):
                # l10n_ch_postal is used for the purpose of Client Number on your own accounts, so don't do the check there
                if rec.partner_id and not rec.partner_id.ref_company_ids:
                    raise ValidationError(
                        _("The postal number {} is not valid.\n"
                          "It must be a valid postal number format. eg. 10-8060-7").format(rec.l10n_ch_postal))
        return True

    @api.constrains("l10n_ch_isr_subscription_chf", "l10n_ch_isr_subscription_eur")
    def _check_subscription_num(self):
        """Validate ISR subscription number format
        Subscription number can only starts with 01 or 03
        """
        for rec in self:
            for currency in ["CHF", "EUR"]:
                subscrip = rec.l10n_ch_isr_subscription_chf if currency == "CHF" else rec.l10n_ch_isr_subscription_eur
                if subscrip and not _is_l10n_ch_isr_issuer(subscrip, currency):
                    example = "01-162-8" if currency == "CHF" else "03-162-5"
                    raise ValidationError(
                        _("The ISR subcription {} for {} number is not valid.\n"
                          "It must starts with {} and we a valid postal number format. eg. {}"
                          ).format(subscrip, currency, ISR_SUBSCRIPTION_CODE[currency], example))
        return True

    @api.depends('partner_id', 'company_id')
    def _compute_l10n_ch_show_subscription(self):
        for bank in self:
            if bank.partner_id:
                bank.l10n_ch_show_subscription = bank.partner_id.ref_company_ids.country_id.code in ('CH', 'LI')
            elif bank.company_id:
                bank.l10n_ch_show_subscription = bank.company_id.account_fiscal_country_id.code in ('CH', 'LI')
            else:
                bank.l10n_ch_show_subscription = self.env.company.account_fiscal_country_id.code in ('CH', 'LI')

    @api.depends('acc_number', 'acc_type')
    def _compute_sanitized_acc_number(self):
        #Only remove spaces in case it is not postal
        postal_banks = self.filtered(lambda b: b.acc_type == "postal")
        for bank in postal_banks:
            bank.sanitized_acc_number = bank.acc_number
        super(ResPartnerBank, self - postal_banks)._compute_sanitized_acc_number()

    @api.depends('acc_number')
    def _compute_l10n_ch_qr_iban(self):
        for record in self:
            try:
                validate_qr_iban(record.acc_number)
                valid_qr_iban = True
            except ValidationError:
                valid_qr_iban = False

            if valid_qr_iban:
                record.l10n_ch_qr_iban = record.sanitized_acc_number
            else:
                record.l10n_ch_qr_iban = None

    @api.model
    def create(self, vals):
        if vals.get('l10n_ch_qr_iban'):
            validate_qr_iban(vals['l10n_ch_qr_iban'])
            vals['l10n_ch_qr_iban'] = pretty_iban(normalize_iban(vals['l10n_ch_qr_iban']))
        return super().create(vals)

    def write(self, vals):
        if vals.get('l10n_ch_qr_iban'):
            validate_qr_iban(vals['l10n_ch_qr_iban'])
            vals['l10n_ch_qr_iban'] = pretty_iban(normalize_iban(vals['l10n_ch_qr_iban']))
        return super().write(vals)

    @api.model
    def _get_supported_account_types(self):
        rslt = super(ResPartnerBank, self)._get_supported_account_types()
        rslt.append(('postal', _('Postal')))
        return rslt

    @api.model
    def retrieve_acc_type(self, acc_number):
        """ Overridden method enabling the recognition of swiss postal bank
        account numbers.
        """
        acc_number_split = ""
        # acc_number_split is needed to continue to recognize the account
        # as a postal account even if the difference
        if acc_number and " " in acc_number:
            acc_number_split = acc_number.split(" ")[0]
        if _is_l10n_ch_postal(acc_number) or (acc_number_split and _is_l10n_ch_postal(acc_number_split)):
            return 'postal'
        else:
            return super(ResPartnerBank, self).retrieve_acc_type(acc_number)

    @api.depends('acc_number', 'partner_id', 'acc_type')
    def _compute_l10n_ch_postal(self):
        for record in self:
            if record.acc_type == 'iban':
                record.l10n_ch_postal = self._retrieve_l10n_ch_postal(record.sanitized_acc_number)
            elif record.acc_type == 'postal':
                if record.acc_number and " " in record.acc_number:
                    record.l10n_ch_postal = record.acc_number.split(" ")[0]
                else:
                    record.l10n_ch_postal = record.acc_number
                    # In case of ISR issuer, this number is not
                    # unique and we fill acc_number with partner
                    # name to give proper information to the user
                    if record.partner_id and record.acc_number[:2] in ["01", "03"]:
                        record.acc_number = ("{} {}").format(record.acc_number, record.partner_id.name)

    @api.model
    def _is_postfinance_iban(self, iban):
        """Postfinance IBAN have format
        CHXX 0900 0XXX XXXX XXXX K
        Where 09000 is the clearing number
        """
        return iban.startswith(('CH', 'LI')) and iban[4:9] == CLEARING

    @api.model
    def _pretty_postal_num(self, number):
        """format a postal account number or an ISR subscription number
        as per specifications with '-' separators.
        eg. 010001628 -> 01-162-8
        """
        if re.match('^[0-9]{2}-[0-9]{1,6}-[0-9]$', number or ''):
            return number
        currency_code = number[:2]
        middle_part = number[2:-1]
        trailing_cipher = number[-1]
        middle_part = middle_part.lstrip("0")
        return currency_code + '-' + middle_part + '-' + trailing_cipher

    @api.model
    def _retrieve_l10n_ch_postal(self, iban):
        """Reads a swiss postal account number from a an IBAN and returns it as
        a string. Returns None if no valid postal account number was found, or
        the given iban was not from Swiss Postfinance.

        CH09 0900 0000 1000 8060 7 -> 10-8060-7
        """
        if self._is_postfinance_iban(iban):
            # the IBAN corresponds to a swiss account
            return self._pretty_postal_num(iban[-9:])
        return None

    def _l10n_ch_get_qr_vals(self, amount, currency, debtor_partner, free_communication, structured_communication):
        comment = ""
        if free_communication:
            comment = (free_communication[:137] + '...') if len(free_communication) > 140 else free_communication

        creditor_addr_1, creditor_addr_2 = self._get_partner_address_lines(self.partner_id)
        debtor_addr_1, debtor_addr_2 = self._get_partner_address_lines(debtor_partner)

        # Compute reference type (empty by default, only mandatory for QR-IBAN,
        # and must then be 27 characters-long, with mod10r check digit as the 27th one,
        # just like ISR number for invoices)
        reference_type = 'NON'
        reference = ''
        acc_number = self.sanitized_acc_number

        if self.l10n_ch_qr_iban:
            # _check_for_qr_code_errors ensures we can't have a QR-IBAN without a QR-reference here
            reference_type = 'QRR'
            reference = structured_communication
            acc_number = sanitize_account_number(self.l10n_ch_qr_iban)
        elif self._is_iso11649_reference(structured_communication):
            reference_type = 'SCOR'
            reference = structured_communication.replace(' ', '')

        currency = currency or self.currency_id or self.company_id.currency_id

        return [
            'SPC',                                                # QR Type
            '0200',                                               # Version
            '1',                                                  # Coding Type
            acc_number,                                           # IBAN / QR-IBAN
            'K',                                                  # Creditor Address Type
            (self.acc_holder_name or self.partner_id.name)[:70],  # Creditor Name
            creditor_addr_1,                                      # Creditor Address Line 1
            creditor_addr_2,                                      # Creditor Address Line 2
            '',                                                   # Creditor Postal Code (empty, since we're using combined addres elements)
            '',                                                   # Creditor Town (empty, since we're using combined addres elements)
            self.partner_id.country_id.code,                      # Creditor Country
            '',                                                   # Ultimate Creditor Address Type
            '',                                                   # Name
            '',                                                   # Ultimate Creditor Address Line 1
            '',                                                   # Ultimate Creditor Address Line 2
            '',                                                   # Ultimate Creditor Postal Code
            '',                                                   # Ultimate Creditor Town
            '',                                                   # Ultimate Creditor Country
            '{:.2f}'.format(amount),                              # Amount
            currency.name,                                        # Currency
            'K',                                                  # Ultimate Debtor Address Type
            debtor_partner.commercial_partner_id.name[:70],       # Ultimate Debtor Name
            debtor_addr_1,                                        # Ultimate Debtor Address Line 1
            debtor_addr_2,                                        # Ultimate Debtor Address Line 2
            '',                                                   # Ultimate Debtor Postal Code (not to be provided for address type K)
            '',                                                   # Ultimate Debtor Postal City (not to be provided for address type K)
            debtor_partner.country_id.code,                       # Ultimate Debtor Postal Country
            reference_type,                                       # Reference Type
            reference,                                            # Reference
            comment,                                              # Unstructured Message
            'EPD',                                                # Mandatory trailer part
        ]

    def _get_qr_vals(self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication):
        if qr_method == 'ch_qr':
            return self._l10n_ch_get_qr_vals(amount, currency, debtor_partner, free_communication, structured_communication)
        return super()._get_qr_vals(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)

    def _get_qr_code_generation_params(self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication):
        if qr_method == 'ch_qr':
            return {
                'barcode_type': 'QR',
                'width': 256,
                'height': 256,
                'quiet': 1,
                'mask': 'ch_cross',
                'value': '\n'.join(self._get_qr_vals(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)),
                # Swiss QR code requires Error Correction Level = 'M' by specification
                'barLevel': 'M',
            }
        return super()._get_qr_code_generation_params(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)

    def _get_partner_address_lines(self, partner):
        """ Returns a tuple of two elements containing the address lines to use
        for this partner. Line 1 contains the street and number, line 2 contains
        zip and city. Those two lines are limited to 70 characters
        """
        streets = [partner.street, partner.street2]
        line_1 = ' '.join(filter(None, streets))
        line_2 = partner.zip + ' ' + partner.city
        return line_1[:70], line_2[:70]

    @api.model
    def _is_qr_reference(self, reference):
        """ Checks whether the given reference is a QR-reference, i.e. it is
        made of 27 digits, the 27th being a mod10r check on the 26 previous ones.
        """
        return reference \
               and len(reference) == 27 \
               and re.match(r'\d+$', reference) \
               and reference == mod10r(reference[:-1])

    @api.model
    def _is_iso11649_reference(self, reference):
        """ Checks whether the given reference is a ISO11649 (SCOR) reference.
        """
        return reference \
               and len(reference) >= 5 \
               and len(reference) <= 25 \
               and reference.startswith('RF') \
               and int(''.join(str(int(x, 36)) for x in clean(reference[4:] + reference[:4], ' -.,/:').upper().strip())) % 97 == 1
               # see https://github.com/arthurdejong/python-stdnum/blob/master/stdnum/iso11649.py

    def _eligible_for_qr_code(self, qr_method, debtor_partner, currency):
        if qr_method == 'ch_qr':
            return self.acc_type == 'iban' and \
                   (not debtor_partner or debtor_partner.country_id.code in ('CH', 'LI')) \
                   and currency.name in ('EUR', 'CHF')

        return super()._eligible_for_qr_code(qr_method, debtor_partner, currency)

    def _check_for_qr_code_errors(self, qr_method, amount, currency, debtor_partner, free_communication, structured_communication):
        def _partner_fields_set(partner):
            return partner.zip and \
                   partner.city and \
                   partner.country_id.code and \
                   (partner.street or partner.street2)

        if qr_method == 'ch_qr':
            if not _partner_fields_set(self.partner_id):
                return _("The partner set on the bank account meant to receive the payment (%s) must have a complete postal address (street, zip, city and country).", self.acc_number)

            if debtor_partner and not _partner_fields_set(debtor_partner):
                return _("The partner must have a complete postal address (street, zip, city and country).")

            if self.l10n_ch_qr_iban and not self._is_qr_reference(structured_communication):
                return _("When using a QR-IBAN as the destination account of a QR-code, the payment reference must be a QR-reference.")

        return super()._check_for_qr_code_errors(qr_method, amount, currency, debtor_partner, free_communication, structured_communication)

    @api.model
    def _get_available_qr_methods(self):
        rslt = super()._get_available_qr_methods()
        rslt.append(('ch_qr', _("Swiss QR bill"), 10))
        return rslt

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models

class Company(models.Model):
    _inherit = "res.company"

    l10n_ch_isr_preprinted_account = fields.Boolean(string='Preprinted account', compute='_compute_l10n_ch_isr', inverse='_set_l10n_ch_isr')
    l10n_ch_isr_preprinted_bank = fields.Boolean(string='Preprinted bank', compute='_compute_l10n_ch_isr', inverse='_set_l10n_ch_isr')
    l10n_ch_isr_print_bank_location = fields.Boolean(string='Print bank location', default=False, help='Boolean option field indicating whether or not the alternate layout (the one printing bank name and address) must be used when generating an ISR.')
    l10n_ch_isr_scan_line_left = fields.Float(string='Scan line horizontal offset (mm)', compute='_compute_l10n_ch_isr', inverse='_set_l10n_ch_isr')
    l10n_ch_isr_scan_line_top = fields.Float(string='Scan line vertical offset (mm)', compute='_compute_l10n_ch_isr', inverse='_set_l10n_ch_isr')

    def _compute_l10n_ch_isr(self):
        get_param = self.env['ir.config_parameter'].sudo().get_param
        for company in self:
            company.l10n_ch_isr_preprinted_account = bool(get_param('l10n_ch.isr_preprinted_account', default=False))
            company.l10n_ch_isr_preprinted_bank = bool(get_param('l10n_ch.isr_preprinted_bank', default=False))
            company.l10n_ch_isr_scan_line_top = float(get_param('l10n_ch.isr_scan_line_top', default=0))
            company.l10n_ch_isr_scan_line_left = float(get_param('l10n_ch.isr_scan_line_left', default=0))

    def _set_l10n_ch_isr(self):
        set_param = self.env['ir.config_parameter'].sudo().set_param
        for company in self:
            set_param("l10n_ch.isr_preprinted_account", company.l10n_ch_isr_preprinted_account)
            set_param("l10n_ch.isr_preprinted_bank", company.l10n_ch_isr_preprinted_bank)
            set_param("l10n_ch.isr_scan_line_top", company.l10n_ch_isr_scan_line_top)
            set_param("l10n_ch.isr_scan_line_left", company.l10n_ch_isr_scan_line_left)

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_ch_isr_preprinted_account = fields.Boolean(string='Preprinted account',
        related="company_id.l10n_ch_isr_preprinted_account", readonly=False)
    l10n_ch_isr_preprinted_bank = fields.Boolean(string='Preprinted bank',
        related="company_id.l10n_ch_isr_preprinted_bank", readonly=False)
    l10n_ch_isr_print_bank_location = fields.Boolean(string="Print bank on ISR",
        related="company_id.l10n_ch_isr_print_bank_location", readonly=False,
        required=True)
    l10n_ch_isr_scan_line_left = fields.Float(string='Horizontal offset',
        related="company_id.l10n_ch_isr_scan_line_left", readonly=False)
    l10n_ch_isr_scan_line_top = fields.Float(string='Vertical offset',
        related="company_id.l10n_ch_isr_scan_line_top", readonly=False)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_config_settings
from . import account_invoice
from . import account_journal
from . import mail_template
from . import res_bank
from . import res_company
from . import account_bank_statement
from . import ir_actions_report

```

## File: report\isr_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="paperformat_euro_no_margin" model="report.paperformat">
            <field name="name">European A4 without borders</field>
            <field name="default" eval="False" />
            <field name="format">A4</field>
            <field name="orientation">Portrait</field>
            <field name="margin_top">0</field>
            <field name="margin_bottom">0</field>
            <field name="margin_left">0</field>
            <field name="margin_right">0</field>
            <field name="header_line" eval="False" />
            <field name="header_spacing">0</field>
        </record>

        <!--Report containing an ISR corrresponding to an invoice.-->
        <record id="l10n_ch_isr_report" model="ir.actions.report">
            <field name="name">ISR</field>
            <field name="model">account.move</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">l10n_ch.isr_report_main</field>
            <field name="report_file">l10n_ch.isr_report_main</field>
            <field name="print_report_name">'ISR-%s' % object.name</field>
            <field name="paperformat_id" ref="paperformat_euro_no_margin"/>
            <field name="attachment">'ISR-' + object.name + '.pdf'</field>
            <field name="attachment_use">True</field>
        </record>
        <!--No additional condition in report name on invoice state or type as this
            report is only available to be printed for out invoices after
            'draft' state, if the fields required by the ISR have been set.-->

        <template id="l10n_ch_isr_report_template">
            <t t-call="web.external_layout">
                <t t-set="split_total_amount" t-value="invoice.split_total_amount()"/>
                <t t-set="print_bank" t-value="invoice.company_id.l10n_ch_isr_print_bank_location"/>

                <!-- add class to body tag -->
                <script>document.body.className += " l10n_ch_isr";</script>

                <!-- since the body content take the whole page we need a way to add margin
                     back on content outside the ISR so it does not overlap with the header -->
                <div id="content_outside_isr">
                    <h1>ISR for invoice <t t-esc="invoice.name"/></h1>
                </div>

                <div id="isr" t-att-class="'isr-print-bank' if print_bank else None">

                    <!--Voucher, left part of the ISR.-->
                    <div id="voucher">
                        <!--Einzahlung für/Versement pour/Versamento per-->

                        <!--If we use the alternate ISR layout, displaying name
                        and location of the bank.-->
                        <t t-if="print_bank">
                            <div id="voucher-for-bank">
                                <p t-if="not invoice.company_id.l10n_ch_isr_preprinted_bank">
                                    <t t-esc="invoice.partner_bank_id.bank_id.name"/><br />
                                    <t t-esc="invoice.partner_bank_id.bank_id.zip"/>
                                    <t t-esc="invoice.partner_bank_id.bank_id.city"/>
                                </p>
                            </div>
                            <!--Zugunsten von/En faveur de/A favore di-->
                        </t>
                        <div id="voucher-for-contact">
                            <p id="voucher-for_name" t-field="invoice.company_id.display_name"/>
                            <p id="voucher-for_address1" t-field="invoice.company_id.street"/>
                            <p id="voucher-for_address2" t-field="invoice.company_id.street2"/>
                            <p id="voucher-for_address3">
                                <t t-esc="invoice.company_id.zip"/>
                                <t t-esc="invoice.company_id.city"/>
                            </p>
                        </div>

                        <div id="voucher-bank" t-if="not print_bank or not invoice.company_id.l10n_ch_isr_preprinted_account">
                            <!--Konto/Compte/Conto-->
                            <p id="voucher-bank_ref" t-field="invoice.l10n_ch_isr_subscription_formatted"/>
                        </div>

                        <p id="voucher-amount_units" t-esc="split_total_amount[0]"/>
                        <p id="voucher-amount_cents" t-esc="split_total_amount[1]"/>

                        <div id="voucher-by">
                            <!--Einbezahlt von/Versé par/Versato da-->
                            <p id="voucher-by_reference_number" t-field="invoice.l10n_ch_isr_number"/>
                            <address id="voucher-by_customer_address" t-field="invoice.partner_id" t-options='{"widget": "contact", "fields": ["address","name"], "no_marker": True}' />
                        </div>
                    </div>

                    <!--Slip, right part of the ISR.-->
                    <div id="slip">
                        <!--Einzahlung für/Versement pour/Versamento per-->

                        <!--If we use the alternate ISR layout, displaying name
                        and location of the bank.-->
                        <t t-if="print_bank">
                            <div id="slip-for-bank">
                                <p t-if="not invoice.company_id.l10n_ch_isr_preprinted_bank">
                                    <t t-esc="invoice.partner_bank_id.bank_id.name"/><br />
                                    <t t-esc="invoice.partner_bank_id.bank_id.zip"/>
                                    <t t-esc="invoice.partner_bank_id.bank_id.city"/>
                                </p>
                            </div>
                            <!--Zugunsten von/En faveur de/A favore di-->
                        </t>

                        <div id="slip-for-contact">
                            <p id="slip-for_name" t-field="invoice.company_id.display_name"/>
                            <p id="slip-for_address1" t-field="invoice.company_id.street"/>
                            <p id="slip-for_address2" t-field="invoice.company_id.street2"/>
                            <p id="slip-for_address3">
                                <t t-esc="invoice.company_id.zip"/>
                                <t t-esc="invoice.company_id.city"/>
                            </p>
                        </div>

                        <div id="slip-bank" t-if="not print_bank or not invoice.company_id.l10n_ch_isr_preprinted_account">
                            <!--Konto/Compte/Conto-->
                            <!--aka ISR Subscriber number provided by the financial institution-->
                            <p id="slip-bank_ref" t-field="invoice.l10n_ch_isr_subscription_formatted"/>
                        </div>

                        <p id="slip-amount_units" t-esc="split_total_amount[0]"/>
                        <p id="slip-amount_cents" t-esc="split_total_amount[1]"/>

                        <div id="slip-reference">
                            <!--Referenz-Nr./N°de référence/N°di riferimento-->
                            <p id="slip-reference_number" t-field="invoice.l10n_ch_isr_number_spaced"/>
                        </div>

                        <div id="slip-by">
                            <!--Einbezahlt von/Versé par/Versato da-->
                            <address id="slip-by_customer_address" t-field="invoice.partner_id" t-options='{"widget": "contact", "fields": ["address","name"], "no_marker": True}' />
                        </div>

                        <div id="slip-optical-line">
                            <!--Optical reference-->
                            <div t-attf-style="top: {{ invoice.company_id.l10n_ch_isr_scan_line_top }}mm; left: {{ invoice.company_id.l10n_ch_isr_scan_line_left }}mm;">
                                <div t-foreach="invoice.l10n_ch_isr_optical_line" t-as="char" t-esc="char" t-attf-style="right: {{ round((char_size - char_index - 1) * 0.1, 1) }}in"/>
                            </div>
                        </div>
                    </div>
                </div>
            </t>
        </template>

        <template id="l10n_ch.isr_report_main">
            <t t-call="web.html_container">
                <t t-foreach="docs" t-as="invoice">
                    <t t-set="o" t-value="invoice"/>
                    <t t-call="l10n_ch.l10n_ch_isr_report_template"/>
                </t>
            </t>
        </template>
    </data>
</odoo>

```

## File: report\swissqr_report.py

```python
# -*- coding:utf-8 -*-

from odoo import api, models

class ReportSwissQR(models.AbstractModel):
    _name = 'report.l10n_ch.qr_report_main'
    _description = 'Swiss QR-bill report'

    @api.model
    def _get_report_values(self, docids, data=None):
        docs = self.env['account.move'].browse(docids)

        qr_code_urls = {}
        for invoice in docs:
            qr_code_urls[invoice.id] = invoice.partner_bank_id.build_qr_code_base64(invoice.amount_residual, invoice.ref or invoice.name, invoice.payment_reference, invoice.currency_id, invoice.partner_id, qr_method='ch_qr', silent_errors=False)

        return {
            'doc_ids': docids,
            'doc_model': 'account.move',
            'docs': docs,
            'qr_code_urls': qr_code_urls,
        }
```

## File: report\swissqr_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="l10n_ch_qr_report" model="ir.actions.report">
            <field name="name">QR-bill</field>
            <field name="model">account.move</field>
            <field name="report_type">qweb-pdf</field>
            <field name="report_name">l10n_ch.qr_report_main</field>
            <field name="report_file">l10n_ch.qr_report_main</field>
            <field name="print_report_name">'QR-bill-%s' % object.name</field>
            <field name="paperformat_id" ref="l10n_ch.paperformat_euro_no_margin"/>
            <field name="attachment">'QR-bill-' + object.name + '.pdf'</field>
        </record>

        <template id="l10n_ch_swissqr_template">
            <t t-set="o" t-value="o.with_context(lang=lang)"/>
            <t t-call="web.external_layout">
                <!-- TODO master: remove this -->
                <script t-if="0">document.body.className += " l10n_ch_qr";</script>
                <!-- add default margin for header (matching A4 European margin) -->
                <t t-set="report_header_style">padding-top:6.2mm; padding-left:8.2mm; padding-right:8.2mm;</t>

                <t t-set="formated_amount" t-value="'{:,.2f}'.format(o.amount_residual).replace(',','\xa0')"/>

                <t t-set="is_qrr" t-value="o.partner_bank_id.l10n_ch_qr_iban"/>
                <t t-set="is_scor" t-value="o.partner_bank_id._is_iso11649_reference(o.payment_reference)"/>

                <div class="swissqr_page_title">
                    <h1>QR-bill for invoice <t t-esc="o.name"/></h1>
                </div>

                <div class="swissqr_content_v2">

                    <div class="swissqr_receipt">

                        <img src="/l10n_ch/static/src/img/scissors_h.png" class="scissors horizontal_scissors"/>

                        <div id="receipt_title_zone" class="swissqr_section_title">
                            <span>Receipt</span>
                        </div>

                        <div id="receipt_indication_zone" class="receipt_indication_zone">

                            <div class="swissqr_text title">
                                <span>Account / Payable to</span>
                            </div>
                            <div class="swissqr_text content">
                                <span t-field="o.partner_bank_id.acc_number" t-if="not o.partner_bank_id.l10n_ch_qr_iban"/>
                                <span t-field="o.partner_bank_id.l10n_ch_qr_iban" t-if="o.partner_bank_id.l10n_ch_qr_iban"/>
                                <br/>
                                <span t-esc="o.partner_bank_id.acc_holder_name or o.company_id.name"/><br/>
                                <span t-field="o.company_id.street"/><br/>
                                <t t-if="o.company_id.country_id.code != 'CH'">
                                    <span t-field="o.company_id.country_id.code"/>
                                </t>
                                <span t-field="o.company_id.zip"/>
                                <span t-field="o.company_id.city"/><br/>
                                <br/>
                            </div>

                            <t t-if="is_qrr or is_scor">
                                <div class="swissqr_text title">
                                    <span>Reference</span>
                                </div>
                            </t>
                            <t t-if="is_qrr">
                                <div class="swissqr_text content">
                                    <span t-esc="o.space_qrr_reference(o.payment_reference)"/><br/>
                                    <br/>
                                </div>
                            </t>
                            <t t-if="is_scor">
                                <div class="swissqr_text content">
                                    <span t-esc="o.space_scor_reference(o.payment_reference)"/><br/>
                                    <br/>
                                </div>
                            </t>

                            <div class="swissqr_text title">
                                <span>Payable by</span>
                            </div>
                            <div class="swissqr_text content">
                                <span t-field="o.partner_id.commercial_partner_id.name"/><br/>
                                <span t-field="o.partner_id.street"/>
                                <span t-field="o.partner_id.street2"/><br/>
                                <t t-if="o.partner_id.country_id.code != 'CH'">
                                    <span t-field="o.partner_id.country_id.code"/>
                                </t>
                                <span t-field="o.partner_id.zip"/>
                                <span t-field="o.partner_id.city"/>
                            </div>

                        </div>

                        <div id="receipt_amount_zone" class="swissqr_column_left receipt_amount_zone">
                            <div class="swissqr_text">
                                <div class="column">
                                    <div class="title">
                                        <span>Currency</span>
                                    </div>
                                    <div class="content">
                                        <span t-field="o.currency_id.name"/>
                                    </div>
                                </div>
                                <div class="column">
                                    <div class="title">
                                        <span>Amount</span>
                                    </div>
                                    <div class="content">
                                        <span t-esc="formated_amount"/>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <div id="receipt_acceptance_point_zone" class="receipt_acceptance_point_zone">
                            <div class="swissqr_text content">
                                <span class="title">Acceptance point</span>
                            </div>
                        </div>

                    </div>

                    <div class="swissqr_body">

                        <img src="/l10n_ch/static/src/img/scissors_v.png" class="scissors vertical_scissors"/>

                        <div class="swissqr_column_left">

                            <div class="swissqr_section_title">
                                <span>Payment part</span>
                            </div>

                            <img class="swissqr" t-att-src="qr_code_urls[o.id]"/>

                            <div id="amount_zone" class="amount_zone">
                                <div class="swissqr_text">
                                    <div class="column">
                                        <div class="title">
                                            <span>Currency</span>
                                        </div>
                                        <div class="content">
                                            <span t-field="o.currency_id.name"/>
                                        </div>
                                    </div>
                                    <div class="column">
                                        <div class="title">
                                            <span>Amount</span><br/>
                                        </div>
                                        <div class="content">
                                            <span t-esc="formated_amount"/>
                                        </div>
                                    </div>
                                </div>
                            </div>

                        </div>

                        <div id="indications_zone" class="swissqr_column_right">
                            <div class="swissqr_text title">
                                <span>Account / Payable to</span><br/>
                            </div>
                            <div class="swissqr_text content">
                                <span t-field="o.partner_bank_id.acc_number" t-if="not o.partner_bank_id.l10n_ch_qr_iban"/>
                                <span t-field="o.partner_bank_id.l10n_ch_qr_iban" t-if="o.partner_bank_id.l10n_ch_qr_iban"/>
                                <br/>
                                <span t-esc="o.partner_bank_id.acc_holder_name or o.company_id.name"/><br/>
                                <span t-field="o.company_id.street"/><br/>
                                <t t-if="o.company_id.country_id.code != 'CH'">
                                    <span t-field="o.company_id.country_id.code"/>
                                </t>
                                <span t-field="o.company_id.zip"/>
                                <span t-field="o.company_id.city"/><br/>
                                <br/>
                            </div>

                            <t t-if="is_qrr or is_scor">
                                <div class="swissqr_text title">
                                    <span class="title">Reference</span>
                                </div>
                            </t>
                            <t t-if="is_qrr">
                                <div class="swissqr_text content">
                                    <span t-esc="o.space_qrr_reference(o.payment_reference)"/><br/>
                                    <br/>
                                </div>
                            </t>
                            <t t-if="is_scor">
                                <div class="swissqr_text content">
                                    <span t-esc="o.space_scor_reference(o.payment_reference)"/><br/>
                                    <br/>
                                </div>
                            </t>

                            <t t-set="additional_info" t-value="(o.ref or o.name if is_qrr or is_scor else o.payment_reference or o.ref or o.name)"/>
                            <t t-if="additional_info">
                                <div class="swissqr_text title">
                                    <span>Additional information</span>
                                </div>
                                <div class="swissqr_text content">
                                    <span t-esc="additional_info"/><br/>
                                    <br/>
                                </div>
                            </t>

                            <div class="swissqr_text title">
                                <span>Payable by</span>
                            </div>
                            <div class="swissqr_text content">
                                <span t-field="o.partner_id.commercial_partner_id.name"/><br/>
                                <span t-field="o.partner_id.street"> </span>
                                <span t-field="o.partner_id.street2"/><br/>
                                <t t-if="o.partner_id.country_id.code != 'CH'">
                                    <span t-field="o.partner_id.country_id.code"/>
                                </t>
                                <span t-field="o.partner_id.zip"/>
                                <span t-field="o.partner_id.city"/><br/>
                                <br/>
                            </div>

                        </div>

                    </div>
                </div>
            </t>
        </template>

        <template id="l10n_ch.qr_report_main">
            <t t-call="web.html_container">
                <t t-foreach="docs" t-as="o">
                    <t t-set="lang" t-value="o.partner_id.lang"/>
                    <t t-call="l10n_ch.l10n_ch_swissqr_template" t-lang="lang"/>
                </t>
            </t>
        </template>
        <template id="minimal_layout_with_report_attribute" inherit_id="web.minimal_layout">
            <body position="attributes">
                <attribute name="t-att-data-report-id">report_xml_id</attribute>
            </body>
        </template> 
    </data>
</odoo>

```

## File: report\__init__.py

```python
# -*- coding:utf-8 -*-

from . import swissqr_report
```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="5.6" y="5.02" width="60" height="41" maskUnits="userSpaceOnUse">
      <rect x="16.59" y="7.94" width="37.14" height="35.14" rx="1" style="fill: #fff"/>
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
    <rect x="16.42" y="8.69" width="37.38" height="37.88" rx="1" style="fill: #393939;opacity: 0.44;isolation: isolate"/>
    <g style="mask: url(#b)">
      <image width="1500" height="1000" transform="translate(5.6 5.02) scale(0.04 0.04)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABdwAAAQBCAYAAAAw4kghAAAACXBIWXMAARTSAAEU0gH6NdiXAAAgAElEQVR4XuzcwUmcYRSG0WtwP0VI9lqBYxMuLCCQigIpwIU1JDhWoHuxCCuY7Aay+gLzEPjhnPW7v/As7sVx5jgAAAAAAMBZvqwGAAAAAADAmuAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACBwuRoAAMBf7h9mvl6tVrB97x8zT4+rFQAAnFwcZ46rEQAAnDwfZva3qxVs3+Fl5m6/WgEAwImXMgAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACF8eZ42oEAJtw/zDz/dtqBZzr5npmt1utYPs+P2de31Yr4Fw/fs48Pa5WALAJl6sBAGzG16uZ/e1qBQD/ZrdzV+B/+PV7tQCAzfBSBgAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABA4HI1AIDNeP+YObysVsC5bq5ndrvVCrbv83Pm9W21As71/rFaAMBmXBxnjqsRAACcPB9m9rerFWzf4WXmbr9aAQDAiZcyAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAADAn3bu2AZhIAqi4CG5/xSJWnAlFrUcsRN+8hKkmXgreMECAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAIHBMAwAAuHm+1nqf0wr+3/WZFgAAcPPYa+1pBAAAAAAA/OZSBgAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAOLc5F8AAAGNSURBVICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAICA4A4AAAAAAAHBHQAAAAAAAoI7AAAAAAAEBHcAAAAAAAgI7gAAAAAAEBDcAQAAAAAgILgDAAAAAEBAcAcAAAAAgIDgDgAAAAAAAcEdAAAAAAACgjsAAAAAAAQEdwAAAAAACAjuAAAAAAAQENwBAAAAACAguAMAAAAAQEBwBwAAAACAgOAOAAAAAAABwR0AAAAAAAKCOwAAAAAABAR3AAAAAAAICO4AAAAAABAQ3AEAAAAAICC4AwAAAABAQHAHAAAAAIDAF+7QNizsqopbAAAAAElFTkSuQmCC"/>
    </g>
  </g>
</svg>

```

## File: static\src\font\liberation_sans_license.txt

```text
Digitized data copyright (c) 2010 Google Corporation
	with Reserved Font Arimo, Tinos and Cousine.
Copyright (c) 2012 Red Hat, Inc.
	with Reserved Font Name Liberation.

This Font Software is licensed under the SIL Open Font License,
Version 1.1.

This license is copied below, and is also available with a FAQ at:
http://scripts.sil.org/OFL

SIL OPEN FONT LICENSE Version 1.1 - 26 February 2007

PREAMBLE The goals of the Open Font License (OFL) are to stimulate
worldwide development of collaborative font projects, to support the font
creation efforts of academic and linguistic communities, and to provide
a free and open framework in which fonts may be shared and improved in
partnership with others.

The OFL allows the licensed fonts to be used, studied, modified and
redistributed freely as long as they are not sold by themselves.
The fonts, including any derivative works, can be bundled, embedded,
redistributed and/or sold with any software provided that any reserved
names are not used by derivative works.  The fonts and derivatives,
however, cannot be released under any other type of license.  The
requirement for fonts to remain under this license does not apply to
any document created using the fonts or their derivatives.

 

DEFINITIONS
"Font Software" refers to the set of files released by the Copyright
Holder(s) under this license and clearly marked as such.
This may include source files, build scripts and documentation.

"Reserved Font Name" refers to any names specified as such after the
copyright statement(s).

"Original Version" refers to the collection of Font Software components
as distributed by the Copyright Holder(s).

"Modified Version" refers to any derivative made by adding to, deleting,
or substituting ? in part or in whole ?
any of the components of the Original Version, by changing formats or
by porting the Font Software to a new environment.

"Author" refers to any designer, engineer, programmer, technical writer
or other person who contributed to the Font Software.


PERMISSION & CONDITIONS

Permission is hereby granted, free of charge, to any person obtaining a
copy of the Font Software, to use, study, copy, merge, embed, modify,
redistribute, and sell modified and unmodified copies of the Font
Software, subject to the following conditions:

1) Neither the Font Software nor any of its individual components,in
   Original or Modified Versions, may be sold by itself.

2) Original or Modified Versions of the Font Software may be bundled,
   redistributed and/or sold with any software, provided that each copy
   contains the above copyright notice and this license. These can be
   included either as stand-alone text files, human-readable headers or
   in the appropriate machine-readable metadata fields within text or
   binary files as long as those fields can be easily viewed by the user.

3) No Modified Version of the Font Software may use the Reserved Font
   Name(s) unless explicit written permission is granted by the
   corresponding Copyright Holder. This restriction only applies to the
   primary font name as presented to the users.

4) The name(s) of the Copyright Holder(s) or the Author(s) of the Font
   Software shall not be used to promote, endorse or advertise any
   Modified Version, except to acknowledge the contribution(s) of the
   Copyright Holder(s) and the Author(s) or with their explicit written
   permission.

5) The Font Software, modified or unmodified, in part or in whole, must
   be distributed entirely under this license, and must not be distributed
   under any other license. The requirement for fonts to remain under
   this license does not apply to any document created using the Font
   Software.


 
TERMINATION
This license becomes null and void if any of the above conditions are not met.

 

DISCLAIMER
THE FONT SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO ANY WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT
OF COPYRIGHT, PATENT, TRADEMARK, OR OTHER RIGHT.  IN NO EVENT SHALL THE
COPYRIGHT HOLDER BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY,
INCLUDING ANY GENERAL, SPECIAL, INDIRECT, INCIDENTAL, OR CONSEQUENTIAL
DAMAGES, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
FROM, OUT OF THE USE OR INABILITY TO USE THE FONT SOFTWARE OR FROM OTHER
DEALINGS IN THE FONT SOFTWARE.


```

## File: static\src\font\ocrb-license.txt

```text
Files: ocrb.otf
Copyright: 2012 Matthew Skala
License: public-domain
  This file is released to the public domain by its author, Matthew Skala.

```

## File: views\account_invoice.xml

```xml
<?xml version="1.0"?>
<odoo>
    <template id="l10n_ch_report_invoice_document" inherit_id="account.report_invoice_document">
        <xpath expr="//div[@id='qrcode']" position="attributes">
            <attribute name="t-if" add="and o.qr_code_method != 'ch_qr'" separator=" "/>
        </xpath>
    </template>
</odoo>

```

## File: views\account_invoice_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="isr_invoice_form" model="ir.ui.view">
            <field name="name">l10n_ch.account.invoice.form</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='is_move_sent']" position="after">
                    <field name="l10n_ch_isr_sent" invisible="1"/>
                    <field name="l10n_ch_currency_name" invisible="1" readonly="1"/>
                </xpath>

                <xpath expr="//button[@id='account_invoice_payment_btn']" position="before">
                    <button
                        name="print_ch_qr_bill"
                        string="Print QR-bill"
                        type="object"
                        attrs="{'invisible':['|', ('state', '!=', 'posted'),
                                             '|', ('l10n_ch_isr_sent', '=', True),
                                             '|', ('move_type', '!=', 'out_invoice'),
                                             ('l10n_ch_currency_name', 'not in', ['EUR', 'CHF'])]}"
                        groups="base.group_user"
                        class="oe_highlight"
                        />
                    <button
                        name="print_ch_qr_bill"
                        string="Print QR-bill"
                        type="object"
                        attrs="{'invisible':['|', ('state', '!=', 'posted'),
                                             '|', ('l10n_ch_isr_sent', '=', False),
                                             '|', ('move_type', '!=', 'out_invoice'),
                                             ('l10n_ch_currency_name', 'not in', ['EUR', 'CHF'])]}"
                        groups="base.group_user"
                        />
                </xpath>
                <header position="after">
                    <field name="l10n_ch_isr_needs_fixing" invisible="1"/>
                    <div groups="account.group_account_invoice" class="alert alert-warning" role="alert" style="margin-bottom:0px;" attrs="{'invisible': [('l10n_ch_isr_needs_fixing', '=', False)]}">
                        Please fill in a correct ISR reference in the payment reference.  The banks will refuse your payment file otherwise.
                    </div>
                </header>
            </field>
        </record>

        <record id="isr_invoice_search_view" model="ir.ui.view">
            <field name="name">l10n_ch.invoice.select</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_account_invoice_filter"/>
            <field name="mode">primary</field>
            <field name="arch" type="xml">
                <xpath expr="//search" position="inside">
                    <field name="l10n_ch_isr_number" string="ISR reference number"/>
                </xpath>
            </field>
        </record>

        <!--Overridden action (and primary child view), so the filter are only
        available for customer invoices-->
        <record id="account.action_move_out_invoice_type" model="ir.actions.act_window">
            <field name="name">Customer Invoices</field>
            <field name="res_model">account.move</field>
            <field name="search_view_id" ref="isr_invoice_search_view"/>
        </record>
    </data>
</odoo>

```

## File: views\res_bank_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="isr_partner_bank_form" model="ir.ui.view">
            <field name="name">l10n_ch.res.partner.bank.form</field>
            <field name="model">res.partner.bank</field>
            <field name="inherit_id" ref="base.view_partner_bank_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='acc_number']" position="after">
                    <field name="l10n_ch_qr_iban" attrs="{'invisible': [('l10n_ch_show_subscription', '=', False)]}"/>
                    <label for="l10n_ch_postal" string="ISR Client Identification Number" attrs="{'invisible': [('l10n_ch_show_subscription', '=', False)]}"/>
                    <field name="l10n_ch_postal" nolabel="1" attrs="{'invisible': [('l10n_ch_show_subscription', '=', False)]}"/>
                    <field name="l10n_ch_postal" attrs="{'invisible': [('l10n_ch_show_subscription', '=', True)]}"/>
                    <field name="l10n_ch_show_subscription" invisible="1"/>
                    <field name="l10n_ch_isr_subscription_chf" attrs="{'invisible': [('l10n_ch_show_subscription', '=', False)]}"/>
                    <field name="l10n_ch_isr_subscription_eur" attrs="{'invisible': [('l10n_ch_show_subscription', '=', False)]}"/>
                </xpath>
            </field>
        </record>

        <record id="isr_partner_bank_tree" model="ir.ui.view">
            <field name="name">l10n_ch.res.partner.bank.tree</field>
            <field name="model">res.partner.bank</field>
            <field name="inherit_id" ref="base.view_partner_bank_tree"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='acc_number']" position="after">
                    <field name="l10n_ch_postal" invisible="1"/>
                </xpath>
            </field>
        </record>

        <record id="isr_partner_property_bank_tree" model="ir.ui.view">
            <field name="name">l10n_ch.res.partner.property.form</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="account.view_partner_property_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='acc_number']" position="after">
                    <field name="l10n_ch_postal" invisible="1"/>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_config_settings_view_form" model="ir.ui.view">
            <field name="name">res.config.settings.view.form.inherit.l10n.ch</field>
            <field name="model">res.config.settings</field>
            <field name="inherit_id" ref="account.res_config_settings_view_form"/>
            <field name="arch" type="xml">
                <xpath expr="//div[@id='invoicing_settings']" position="inside">
                    <div class="col-12 col-lg-6 o_setting_box" id="l10n_ch-isr_print_bank" attrs="{'invisible': [('country_code', '!=', 'CH')]}">
                        <div class="o_setting_left_pane">
                            <field name="l10n_ch_isr_print_bank_location"/>
                        </div>
                        <div class="o_setting_right_pane">
                            <label for="l10n_ch_isr_print_bank_location"/>
                            <div class="text-muted">
                                Print the coordinates of your bank under the &apos;Payment for&apos; title of the ISR.
                                Your address will be moved to the &apos;in favour of&apos; section.
                            </div>
                            <div class="content-group" attrs="{'invisible': [('l10n_ch_isr_print_bank_location', '=', False)]}">
                                <div class="row mt16">
                                    <label for="l10n_ch_isr_preprinted_bank" class="col-lg-4 o_light_label"/>
                                    <field name="l10n_ch_isr_preprinted_bank"/>
                                </div>
                                <div class="row">
                                    <label for="l10n_ch_isr_preprinted_account" class="col-lg-4 o_light_label"/>
                                    <field name="l10n_ch_isr_preprinted_account"/>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="col-12 col-lg-6 o_setting_box" id="l10n_ch-isr_print_scanline_offset" attrs="{'invisible': [('country_code', '!=', 'CH')]}">
                        <div class="o_setting_left_pane"/>
                        <div class="o_setting_right_pane">
                            <span class="o_form_label">ISR scan line offset</span>
                            <div class="text-muted">
                                Offset to move the scan line in mm
                            </div>
                            <div class="content-group">
                                <div class="row mt16">
                                    <label for="l10n_ch_isr_scan_line_top" class="col-lg-4 o_light_label"/>
                                    <field name="l10n_ch_isr_scan_line_top"/>
                                </div>
                                <div class="row">
                                    <label for="l10n_ch_isr_scan_line_left" class="col-lg-4 o_light_label"/>
                                    <field name="l10n_ch_isr_scan_line_left"/>
                                </div>
                            </div>
                        </div>
                    </div>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\setup_wizard_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="setup_bank_account_wizard_inherit" model="ir.ui.view">
            <field name="name">account.setup.bank.manual.config.form.ch.inherit</field>
            <field name="model">account.setup.bank.manual.config</field>
            <field name="inherit_id" ref="account.setup_bank_account_wizard"/>
            <field name="arch" type="xml">
                <field name="bank_bic" position="after">
                    <field name="l10n_ch_show_subscription" invisible="1"/>
                    <field name="l10n_ch_isr_subscription_chf" attrs="{'invisible': [('l10n_ch_show_subscription', '=', False)]}"/>
                    <label for="l10n_ch_postal" string="ISR Client Identification Number" attrs="{'invisible': [('l10n_ch_show_subscription', '=', False)]}"/>
                    <field name="l10n_ch_postal" nolabel="1" attrs="{'invisible': [('l10n_ch_show_subscription', '=', False)]}"/>
                    <field name="l10n_ch_qr_iban" attrs="{'invisible': [('l10n_ch_show_subscription', '=', False)]}"/>
                </field>
            </field>
        </record>
    </data>
</odoo>

```

## File: wizard\setup_wizards.py

```python
# -*- coding: utf-8 -*-

from odoo import api, models


class SwissSetupBarBankConfigWizard(models.TransientModel):
    _inherit = 'account.setup.bank.manual.config'

    @api.onchange('acc_number')
    def _onchange_recompute_qr_iban(self):
        # Needed because ORM doesn't properly call the compute in 'new' mode, due to inherits, and
        # we want this field to be displayed in the wizard. We need to manually set acc_number
        # on the inherits m2o before calling the compute function manually.
        self.res_partner_bank_id.acc_number = self.acc_number
        self.res_partner_bank_id._compute_l10n_ch_qr_iban()
        self.l10n_ch_qr_iban = self.res_partner_bank_id.l10n_ch_qr_iban

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-

from . import setup_wizards

```

