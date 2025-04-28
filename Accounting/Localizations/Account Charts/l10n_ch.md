# Odoo Module: l10n_ch

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models
from . import report

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
This module defines a chart of account for Switzerland (Swiss PME/KMU 2015), taxes and enables the generation of ISR when you print an invoice or send it by mail.

An ISR will be generated if you specify the information it needs :
    - The bank account you expect to be paid on must be set, and have a valid postal reference.
    - Your invoice must have been set assigned a bank account to receive its payment
      (this can be done manually, but a default value is automatically set if you have defined a bank account).
    - You must have set the postal references of your bank.
    - Your invoice must be in EUR or CHF (as ISRs do not accept other currencies)

The generation of the ISR is automatic if you meet the previous criteria.

Here is how it works:
    - Printing the invoice will trigger the download of two files: the invoice, and its ISR
    - Clicking the 'Send by mail' button will attach two files to your draft mail : the invoice, and the corresponding ISR.
    """,
    'version': '11.1',
    'category': 'Accounting/Localizations/Account Charts',

    'depends': ['account', 'l10n_multilang', 'base_iban'],

    'data': [
        'data/l10n_ch_chart_data.xml',
        'data/account.account.template.csv',
        'data/l10n_ch_chart_post_data.xml',
        'data/account_data.xml',
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

    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","reconcile"
"ch_coa_1060","Titres","1060","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1069","Ajustement de la valeur des titres","1069","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1091","Compte d'attente pour salaires","1091","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","True"
"ch_coa_1099","Compte d'attente autre","1099","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","True"
"ch_coa_1100","Débiteurs","1100","account.data_account_type_receivable","l10n_ch.l10nch_chart_template","True"
"ch_coa_1101","Débiteurs (PoS)","1101","account.data_account_type_receivable","l10n_ch.l10nch_chart_template","True"
"ch_coa_1109","Ducroire","1109","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1140","Avances et prêts","1140","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1149","Ajustement de la valeur des avances et des prêts","1149","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1170","Impôt préalable: TVA s/matériel, marchandises, prestations et énergie","1170","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1171","Impôt préalable: TVA s/investissements et autres charges d’exploitation","1171","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1176","Impôt anticipé","1176","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1180","Créances envers les assurances sociales et institutions de prévoyance","1180","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1189","Impôt à la source","1189","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1190","Autres créances à court terme","1190","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1199","Ajustement de la valeur des créances à court terme","1199","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1200","Marchandises commerciales","1200","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1207","Variation des stocks de marchandises","1207","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1208","Acomptes sur les marchandises commerciales","1208","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1209","Corrections de la valeur des stocks de marchandises","1209","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1210","Matières premières","1210","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1217","Variation des stocks des matières premières","1217","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1218","Acomptes sur matières premières","1218","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1219","Corrections de la valeur sur matières premières","1219","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1220","Matières auxiliaires","1220","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1230","Matières consommables","1230","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1250","Marchandises en consignation","1250","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1260","Stocks de produits finis","1260","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1267","Variation de stocks de produits finis","1267","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1269","Correction de la valeur de stocks de produits finis","1269","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1270","Stocks de produits semi-ouvrés","1270","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1277","Variation de stock produits semi-ouvrés","1277","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1279","Corrections de la valeur des stock produits semi-ouvrés","1279","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1280","Travaux en cours","1280","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1287","Variation de la valeur des travaux en cours","1287","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1289","Corrections de la valeur des travaux en cours","1289","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1300","Charges payées d‘avance","1300","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1301","Produits à recevoir","1301","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1400","Titres à long terme","1400","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1409","Ajustement de la valeur des titres","1409","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1440","Prêts","1440","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1441","Hypothèques","1441","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1449","Ajustement de la valeur des créances à long terme","1449","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1480","Participations","1480","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1489","Ajustement de la valeur des participations","1489","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1500","Machines et appareils","1500","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1509","Amortissements sur les machines et appareils","1509","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1510","Mobilier et installations","1510","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1519","Amortissements sur le mobilier et les installations","1519","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1520","Machines de bureau, informatique, systèmes de communication","1520","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1529","Amortissements sur les machines de bureau, inf. et syst. comm.","1529","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1530","Véhicules","1530","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1539","Amortissements sur les véhicules","1539","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1540","Outillages et appareils","1540","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1549","Amortissements sur les outillages et appareils","1549","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1550","Installations de stockage","1550","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1559","Amortissements sur les installations de stockage","1559","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1570","Equipements et Installations","1570","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1579","Amortissements sur les équipements et installations","1579","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1590","Autres immobilisations corporelles meubles","1590","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1599","Amortissements sur les autres immobilisations corporelles meubles","1599","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1600","Immeubles d’exploitation","1600","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1609","Amortissements sur les immeubles d’exploitation","1609","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1700","Brevets, know-how, licences, droits, développement","1700","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1709","Amortissements sur les brevets, know-how, licences, droits, dév.","1709","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1770","Goodwill","1770","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1779","Ajustement de la valeur des goodwill","1779","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_1850","Capital actions, capital social, droits de participations ou capital de fondation non versés","1850","account.data_account_type_current_assets","l10n_ch.l10nch_chart_template","False"
"ch_coa_2000","Créanciers","2000","account.data_account_type_payable","l10n_ch.l10nch_chart_template","True"
"ch_coa_2030","Acomptes de clients","2030","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2100","Dettes bancaires","2100","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2120","Engagements de financement par leasing","2120","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2140","Autres dettes à court terme rémunérées","2140","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2160","Dettes envers l'actionnaire","2160","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2200","TVA due","2200","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2201","Décompte TVA","2201","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2206","Impôt anticipé dû","2206","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2208","Impôts directs","2208","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2210","Autres dettes à court terme","2210","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2261","Dividendes","2261","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2270","Assurances sociales et institutions de prévoyance","2270","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2279","Impôt à la source","2279","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2300","Charges à payer","2300","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2301","Produits encaissés d’avance","2301","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2330","Provisions à court terme","2330","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2400","Dettes bancaires","2400","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2420","Engagements de financement par leasing","2420","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2430","Emprunts obligataires","2430","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2450","Emprunts","2450","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2451","Hypothèques","2451","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2500","Autres dettes à long terme","2500","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2600","Provisions","2600","account.data_account_type_current_liabilities","l10n_ch.l10nch_chart_template","False"
"ch_coa_2800","Capital-actions, capital social, capital de fondation","2800","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_2900","Réserves légales issues du capital","2900","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_2940","Réserves d‘évaluation","2940","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_2950","Réserves légales issues du bénéfice","2950","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_2960","Réserves libres","2960","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_2970","Bénéfice / perte reporté","2970","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_2979","Bénéfice / perte de l’exercice","2979","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_2980","Propres actions, parts sociales, droits de participations (poste négatif)","2980","account.data_account_type_equity","l10n_ch.l10nch_chart_template","False"
"ch_coa_3000","Ventes de produits fabriqués","3000","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3009","Déductions sur ventes","3009","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3200","Ventes de marchandises","3200","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3400","Ventes de prestations","3400","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3600","Autres ventes et prestations de services","3600","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3700","Prestations propres","3700","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3710","Consommations propres","3710","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3800","Escomptes","3800","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3801","Rabais et réduction de prix","3801","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3802","Ristournes","3802","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3803","Commissions de tiers","3803","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3804","Frais d'encaissement","3804","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3805","Pertes sur créances clients, variation ducroire","3805","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3806","Différences de change","3806","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3807","Frais d'expédition","3807","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3900","Variation des stocks de produits semi-finis","3900","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3901","Variation des stocks de produits finis","3901","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_3940","Variation de la valeur des prestations non facturées","3940","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_4000","Charges de matériel de l‘atelier","4000","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4008","Variations de stocks","4008","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4009","Déductions obtenues sur achats","4009","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4070","Frêts à l'achat","4070","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4071","Droits de douanes à l'importation","4071","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4072","Frais de transport à l'achat","4072","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4080","Variations de stocks","4080","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4086","Pertes de matières","4086","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4200","Achats de marchandises destinées à la revente","4200","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4400","Prestations / travaux de tiers","4400","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4500","Electricité","4500","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4510","Gaz","4510","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4520","Mazout","4520","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4521","Charbon, briquettes, bois","4521","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4530","Essence","4530","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4540","Eau","4540","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4800","Variation des stocks de marchandises","4800","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4801","Variation des stocks de matières premières","4801","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4900","Escomptes","4900","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4901","Rabais et réductions de prix","4901","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4092","Ristournes","4902","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4903","Commissions obtenues sur achats","4903","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_4906","Différences de change","4906","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_5000","Salaires","5000","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_5700","Charges sociales","5700","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_5800","Autres charges du personnel","5800","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_5900","Charges de personnels temporaires","5900","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6000","Charges de locaux","6000","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6100","Entretien, réparations et remplacement des inst. servant à l’exploitation","6100","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6105","Leasing immobilisations corporelles meubles","6105","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6200","Charges de véhicules et de transport","6200","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6260","Leasing et location de véhicules","6260","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6300","Assurances-choses, droits, taxes, autorisations","6300","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6400","Charges d’énergie et évacuation des déchets","6400","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6500","Charges d‘administration","6500","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6570","Charges et leasing d’informatique","6570","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6600","Publicité","6600","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6700","Autres charges d‘exploitation","6700","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6800","Amortissements et ajustements de valeur des postes sur immobilisations corporelles","6800","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6900","Charges financières","6900","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_6950","Produits financiers","6950","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_7000","Produits accessoires","7000","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_7010","Charges accessoires","7010","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_7500","Produits des immeubles d‘exploitation","7500","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_7510","Charges des immeubles d‘exploitation","7510","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_8000","Charges hors exploitation","8000","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_8100","Produits hors exploitation","8100","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_8500","Charges extraordinaires, exceptionnelles ou hors période","8500","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"
"ch_coa_8510","Produits extraordinaires, exceptionnels ou hors période","8510","account.data_account_type_revenue","l10n_ch.l10nch_chart_template","False"
"ch_coa_8900","Impôts directs","8900","account.data_account_type_expenses","l10n_ch.l10nch_chart_template","False"

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

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Account Tax Group (pre-2024 rates change) -->
        <record id="tax_group_tva_0" model="account.tax.group">
            <field name="name">TVA 0%</field>
        </record>
        <record id="tax_group_tva_25" model="account.tax.group">
            <field name="name">TVA 2.5%</field>
        </record>
        <record id="tax_group_tva_37" model="account.tax.group">
            <field name="name">TVA 3.7%</field>
        </record>
        <record id="tax_group_tva_77" model="account.tax.group">
            <field name="name">TVA 7.7%</field>
        </record>
        <record id="tax_group_tva_100" model="account.tax.group">
            <field name="name">TVA 100%</field>
        </record>

        <!-- Account Tax Group (post-2024 rates change) -->
        <record id="tax_group_vat_26" model="account.tax.group">
            <field name="name">VAT 2.6%</field>
        </record>
        <record id="tax_group_vat_38" model="account.tax.group">
            <field name="name">VAT 3.8%</field>
        </record>
        <record id="tax_group_vat_81" model="account.tax.group">
            <field name="name">VAT 8.1%</field>
        </record>

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

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.ch"/>
    </record>

    <record id="account_tax_report_line_chiffre_af" model="account.tax.report.line">
        <field name="name">I. CHIFFRE D'AFFAIRES</field>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
        <field name="sequence" eval="1"/>
    </record>

    <record id="account_tax_report_line_chtax_200" model="account.tax.report.line">
        <field name="name">200 - Total des contre-prestations convenues ou reçues, y c. de prestations imposées par option, de transferts par procédure de déclaration, de prestations à l’étranger</field>
        <field name="code">tax_ch_200</field>
        <field name="formula">tax_ch_302a + tax_ch_303a + tax_ch_312a + tax_ch_313a + tax_ch_342a + tax_ch_343a + tax_ch_205 + tax_ch_289</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_205" model="account.tax.report.line">
        <field name="name">205 - Contre-prestations déclarées sous ch. 200 qui proviennent de prestations exclues du champ de l’impôt (art. 21) pour lesquelles il a été opté en vertu de l’art. 22</field>
        <field name="code">tax_ch_205</field>
        <field name="tag_name">205</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_220_289" model="account.tax.report.line">
        <field name="name">220 - Prestations exonérées (p. ex. exportations, art. 23), prestations exonérées fournies à des institutions et à des personnes bénéficiaires (art. 107, al. 1, let. a)</field>
        <field name="code">tax_ch_220</field>
        <field name="tag_name">220</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_221" model="account.tax.report.line">
        <field name="name">221 - Prestations fournies à l’étranger</field>
        <field name="code">tax_ch_221</field>
        <field name="tag_name">221</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_225" model="account.tax.report.line">
        <field name="name">225 - Transferts avec la procédure de déclaration (art. 38)</field>
        <field name="code">tax_ch_225</field>
        <field name="tag_name">225</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_230" model="account.tax.report.line">
        <field name="name">230 - Prestations exclues du champ de l’impôt (art. 21) fournies sur le territoire suisse pour lesquelles il n’a pas été opté selon l’art. 22</field>
        <field name="code">tax_ch_230</field>
        <field name="tag_name">230</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_235" model="account.tax.report.line">
        <field name="name">235 - Diminutions de la contre-prestation telles que rabais, escomptes, etc.</field>
        <field name="code">tax_ch_235</field>
        <field name="tag_name">235</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_280" model="account.tax.report.line">
        <field name="name">280 - Divers (p.ex. valeur du terrain)</field>
        <field name="code">tax_ch_280</field>
        <field name="tag_name">280</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_289" model="account.tax.report.line">
        <field name="name">289 - Déductions (Total ch. 220 à 280)</field>
        <field name="code">tax_ch_289</field>
        <field name="formula">tax_ch_220 + tax_ch_221 + tax_ch_225 + tax_ch_230 + tax_ch_235 + tax_ch_280</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="9"/>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_chtax_299" model="account.tax.report.line">
        <field name="name">299 - Total du chiffre d’affaires imposable (ch. 200 moins ch. 289)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="10"/>
        <field name="formula">tax_ch_200 - tax_ch_289</field>
        <field name="parent_id" ref="account_tax_report_line_chiffre_af"/>
    </record>

    <record id="account_tax_report_line_calc_impot" model="account.tax.report.line">
        <field name="name">II. CALCUL DE L'IMPÔT</field>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
        <field name="sequence" eval="2"/>
    </record>

    <record id="account_tax_report_line_supplies_1" model="account.tax.report.line">
        <field name="name">Prestations CHF dès le 01.01.2024</field>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_303a" model="account.tax.report.line">
        <field name="name">303a - Taux normal (8,1%): Prestations CHF dès le 01.01.2024</field>
        <field name="code">tax_ch_303a</field>
        <field name="tag_name">303a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_1"/>
    </record>

    <record id="account_tax_report_line_chtax_313a" model="account.tax.report.line">
        <field name="name">313a - Taux réduit (2,6%): Prestations CHF dès le 01.01.2024</field>
        <field name="code">tax_ch_313a</field>
        <field name="tag_name">313a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_1"/>
    </record>

    <record id="account_tax_report_line_chtax_343a" model="account.tax.report.line">
        <field name="name">343a - Taux spécial pour l'hébergement (3,8%): Prestations CHF dès le 01.01.2024</field>
        <field name="code">tax_ch_343a</field>
        <field name="tag_name">343a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_1"/>
    </record>

    <record id="account_tax_report_line_chtax_383a" model="account.tax.report.line">
        <field name="name">383a - Impôt sur les acquisitions: Prestations CHF dès le 01.01.2024</field>
        <field name="code">tax_ch_383a</field>
        <field name="tag_name">383a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_1"/>
    </record>

    <record id="account_tax_report_line_supplies_2" model="account.tax.report.line">
        <field name="name">Prestations CHF jusqu'au 31.12.2023</field>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_302a" model="account.tax.report.line">
        <field name="name">302a - Taux normal (7,7%): Prestations CHF jusqu'au 31.12.2023</field>
        <field name="code">tax_ch_302a</field>
        <field name="tag_name">302a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_2"/>
    </record>

    <record id="account_tax_report_line_chtax_312a" model="account.tax.report.line">
        <field name="name">312a - Taux réduit (2,5%): Prestations CHF jusqu'au 31.12.2023</field>
        <field name="code">tax_ch_312a</field>
        <field name="tag_name">312a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_2"/>
    </record>

    <record id="account_tax_report_line_chtax_342a" model="account.tax.report.line">
        <field name="name">342a - Taux spécial pour l'hébergement (3,7%): Prestations CHF jusqu'au 31.12.2023</field>
        <field name="code">tax_ch_342a</field>
        <field name="tag_name">342a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_2"/>
    </record>

    <record id="account_tax_report_line_chtax_382a" model="account.tax.report.line">
        <field name="name">382a - Impôt sur les acquisitions: Prestations CHF jusqu'au 31.12.2023</field>
        <field name="code">tax_ch_382a</field>
        <field name="tag_name">382a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_supplies_2"/>
    </record>

    <record id="account_tax_report_line_tax_amount_1" model="account.tax.report.line">
        <field name="name">Impôt CHF / ct. dès le 01.01.2024</field>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_303b" model="account.tax.report.line">
        <field name="name">303b - Taux normal (8,1%): Impôt CHF / ct. dès le 01.01.2024</field>
        <field name="code">tax_ch_303b</field>
        <field name="tag_name">303b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_1"/>
    </record>

    <record id="account_tax_report_line_chtax_313b" model="account.tax.report.line">
        <field name="name">313b - Taux réduit (2,6%): Impôt CHF / ct. dès le 01.01.2024</field>
        <field name="code">tax_ch_313b</field>
        <field name="tag_name">313b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_1"/>
    </record>

    <record id="account_tax_report_line_chtax_343b" model="account.tax.report.line">
        <field name="name">343b - Taux spécial pour l'hébergement (3,8%): Impôt CHF / ct. dès le 01.01.2024</field>
        <field name="code">tax_ch_343b</field>
        <field name="tag_name">343b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_1"/>
    </record>

    <record id="account_tax_report_line_chtax_383b" model="account.tax.report.line">
        <field name="name">383b - Impôt sur les acquisitions: Impôt CHF / ct. dès le 01.01.2024</field>
        <field name="code">tax_ch_383b</field>
        <field name="tag_name">383b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_1"/>
    </record>

    <record id="account_tax_report_line_tax_amount_2" model="account.tax.report.line">
        <field name="name">Impôt CHF / ct. jusqu'au 31.12.2023</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_302b" model="account.tax.report.line">
        <field name="name">302b - Taux normal (7,7%): Impôt CHF / ct. jusqu'au 31.12.2023</field>
        <field name="code">tax_ch_302b</field>
        <field name="tag_name">302b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_2"/>
    </record>

    <record id="account_tax_report_line_chtax_312b" model="account.tax.report.line">
        <field name="name">312b - Taux réduit (2,5%): Impôt CHF / ct. jusqu'au 31.12.2023</field>
        <field name="code">tax_ch_312b</field>
        <field name="tag_name">312b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_2"/>
    </record>

    <record id="account_tax_report_line_chtax_342b" model="account.tax.report.line">
        <field name="name">342b - Taux spécial pour l'hébergement (3,7%): Impôt CHF / ct. jusqu'au 31.12.2023</field>
        <field name="code">tax_ch_342b</field>
        <field name="tag_name">342b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_2"/>
    </record>

    <record id="account_tax_report_line_chtax_382b" model="account.tax.report.line">
        <field name="name">382b - Impôt sur les acquisitions: Impôt CHF / ct. jusqu'au 31.12.2023</field>
        <field name="code">tax_ch_382b</field>
        <field name="tag_name">382b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_tax_amount_2"/>
    </record>

    <record id="account_tax_report_line_chtax_399" model="account.tax.report.line">
        <field name="name">399 - Total de l’impôt dû</field>
        <field name="code">tax_ch_399</field>
        <field name="formula">tax_ch_302b + tax_ch_303b + tax_ch_312b + tax_ch_313b + tax_ch_342b + tax_ch_343b + tax_ch_382b + tax_ch_383b</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_400" model="account.tax.report.line">
        <field name="name">400 - Impôt préalable grevant les coûts en matériel et en prestations de services</field>
        <field name="tag_name">400</field>
        <field name="code">tax_ch_400</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_405" model="account.tax.report.line">
        <field name="name">405 - Impôt préalable grevant les investissements et autres charges d’exploitation</field>
        <field name="code">tax_ch_405</field>
        <field name="tag_name">405</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_410" model="account.tax.report.line">
        <field name="name">410 - Dégrèvement ultérieur de l’impôt préalable (art. 32)</field>
        <field name="code">tax_ch_410</field>
        <field name="tag_name">410</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_415" model="account.tax.report.line">
        <field name="name">415 - Corrections de l’impôt préalable: double affectation (art. 30), prestations à soi-même (art. 31)</field>
        <field name="code">tax_ch_415</field>
        <field name="tag_name">415</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="9"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_420" model="account.tax.report.line">
        <field name="name">420 - Réductions de la déduction de l’impôt préalable: prestations n’étant pas considérées comme des contre-prestations, telles subventions, taxes touristiques (art. 33, al. 2)</field>
        <field name="code">tax_ch_420</field>
        <field name="tag_name">420</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="10"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_479" model="account.tax.report.line">
        <field name="name">479 - Total ch. 400 à 420</field>
        <field name="code">tax_ch_479</field>
        <field name="formula">tax_ch_400 + tax_ch_405 + tax_ch_410 - tax_ch_415 - tax_ch_420</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="11"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_500" model="account.tax.report.line">
        <field name="name">500 - Montant à payer</field>
        <field name="formula">tax_ch_399 - tax_ch_479 &gt; 0 and tax_ch_399 - tax_ch_479 or 0.0</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="12"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_510" model="account.tax.report.line">
        <field name="name">510 - Solde en faveur de l’assujetti</field>
        <field name="formula">tax_ch_479 - tax_ch_399 &gt; 0 and tax_ch_479 - tax_ch_399 or 0.0</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="13"/>
        <field name="parent_id" ref="account_tax_report_line_calc_impot"/>
    </record>

    <record id="account_tax_report_line_chtax_autres_mouv" model="account.tax.report.line">
        <field name="name">III. AUTRES MOUVEMENTS DE FONDS</field>
        <field name="sequence" eval="3"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_chtax_900" model="account.tax.report.line">
        <field name="name">900 - Subventions, taxes touristiques perçues par les offices du tourisme, contributions aux établissements d'élimination des déchets et d'approvisionnement en eau</field>
        <field name="tag_name">900</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_chtax_autres_mouv"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="account_tax_report_line_chtax_910" model="account.tax.report.line">
        <field name="name">910 - Les dons, les dividendes, les dédommagements, etc.</field>
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
        <field name="name">TVA due a 2.6% (TR)</field>
        <field name="description">2.6%</field>
        <field name="amount" eval="2.6"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_26"/>
        <field name="active" eval="False"/>
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
        <field name="name">TVA due à 2.6% (Incl. TR)</field>
        <field name="description">2.6% Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="2.6"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_26"/>
        <field name="active" eval="False"/>
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
        <field name="name">TVA 2.6% sur achat B&amp;S (TR)</field>
        <field name="description">2.6% achat</field>
        <field name="amount" eval="2.6"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_26"/>
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

    <record model="account.tax.template" id="vat_purchase_26_incl">
        <field name="name">TVA 2.6% sur achat B&amp;S (Incl. TR)</field>
        <field name="description">2.6% achat Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="2.6"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_26"/>
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

    <record model="account.tax.template" id="vat_purchase_26_invest">
        <field name="name">TVA 2.6% sur invest. et autres ch. (TR)</field>
        <field name="description">2.6% invest.</field>
        <field name="amount" eval="2.6"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_26"/>
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

    <record model="account.tax.template" id="vat_purchase_26_invest_incl">
        <field name="name">TVA 2.6% sur invest. et autres ch. (Incl. TR)</field>
        <field name="description">2.6% invest. Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="2.6"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_26"/>
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

    <record model="account.tax.template" id="vat_sale_38">
        <field name="name">TVA due a 3.8% (TS)</field>
        <field name="description">3.8%</field>
        <field name="amount" eval="3.8"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_38"/>
        <field name="active" eval="False"/>
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
        <field name="name">TVA due à 3.8% (Incl. TS)</field>
        <field name="description">3.8% Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="3.8"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_38"/>
        <field name="active" eval="False"/>
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
        <field name="name">TVA 3.8% sur achat B&amp;S (TS)</field>
        <field name="description">3.8% achat</field>
        <field name="amount" eval="3.8"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_38"/>
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

    <record model="account.tax.template" id="vat_purchase_38_incl">
        <field name="name">TVA 3.8% sur achat B&amp;S (Incl. TS)</field>
        <field name="description">3.8% achat Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="3.8"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_38"/>
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

    <record model="account.tax.template" id="vat_purchase_38_invest">
        <field name="name">TVA 3.8% sur invest. et autres ch. (TS)</field>
        <field name="description">3.8% invest</field>
        <field name="amount" eval="3.8"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_38"/>
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

    <record model="account.tax.template" id="vat_purchase_38_invest_incl">
        <field name="name">TVA 3.8% sur invest. et autres ch. (Incl. TS)</field>
        <field name="description">3.8% invest Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="3.8"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_38"/>
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

    <record model="account.tax.template" id="vat_sale_81">
        <field name="name">TVA due a 8.1% (TN)</field>
        <field name="description">8.1%</field>
        <field name="amount" eval="8.1"/>
        <field name="sequence" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_81"/>
        <field name="active" eval="False"/>
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
        <field name="name">TVA due à 8.1% (Incl. TN)</field>
        <field name="description">8.1% Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="8.1"/>
        <field name="sequence" eval="0"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_81"/>
        <field name="active" eval="False"/>
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
        <field name="name">TVA 8.1% sur achat B&amp;S (TN)</field>
        <field name="description">8.1% achat</field>
        <field name="amount" eval="8.1"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="0"/>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_81"/>
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

    <record model="account.tax.template" id="vat_purchase_81_incl">
        <field name="name">TVA 8.1% sur achat B&amp;S (Incl. TN)</field>
        <field name="description">8.1% achat Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="8.1"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="0"/>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_81"/>
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

    <record model="account.tax.template" id="vat_purchase_81_invest">
        <field name="name">TVA 8.1% sur invest. et autres ch. (TN)</field>
        <field name="description">8.1% invest.</field>
        <field name="amount" eval="8.1"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_81"/>
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

    <record model="account.tax.template" id="vat_purchase_81_invest_incl">
        <field name="name">TVA 8.1% sur invest. et autres ch. (Incl. TN)</field>
        <field name="description">8.1% invest. Incl.</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="8.1"/>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_81"/>
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

    <record model="account.tax.template" id="vat_purchase_81_return">
        <field name="name">TVA due a 8.1% (TN) (return)</field>
        <field name="description">8.1% achat (return)</field>
        <field name="amount" eval="-8.1"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="0"/>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="type_tax_use">none</field>
        <field name="active" eval="False"/>
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
        <field name="name">TVA 8.1% sur achat service a l'etranger (reverse charge)</field>
        <field name="description">8.1% achat</field>
        <field name="amount_type">group</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_81"/>
        <field name="chart_template_id" ref="l10nch_chart_template"/>
        <field name="active" eval="False"/>
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
            <field name="name">TVA due a 2.5% (TR)</field>
            <field name="description">2.5%</field>
            <field name="amount" eval="2.5"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_25"/>
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
            <field name="name">TVA due à 2.5% (Incl. TR)</field>
            <field name="description">2.5% Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="2.5"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_25"/>
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
            <field name="name">TVA 2.5% sur achat B&amp;S (TR)</field>
            <field name="description">2.5% achat</field>
            <field name="amount" eval="2.5"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_25"/>
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
            <field name="name">TVA 2.5% sur achat B&amp;S (Incl. TR)</field>
            <field name="description">2.5% achat Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="2.5"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_25"/>
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
            <field name="name">TVA 2.5% sur invest. et autres ch. (TR)</field>
            <field name="description">2.5% invest.</field>
            <field name="amount" eval="2.5"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_25"/>
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
            <field name="name">TVA 2.5% sur invest. et autres ch. (Incl. TR)</field>
            <field name="description">2.5% invest. Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="2.5"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_25"/>
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
            <field name="name">TVA due a 3.7% (TS)</field>
            <field name="description">3.7%</field>
            <field name="amount" eval="3.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_37"/>
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
            <field name="name">TVA due à 3.7% (Incl. TS)</field>
            <field name="description">3.7% Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="3.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_37"/>
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
            <field name="name">TVA 3.7% sur achat B&amp;S (TS)</field>
            <field name="description">3.7% achat</field>
            <field name="amount" eval="3.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_37"/>
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
            <field name="name">TVA 3.7% sur achat B&amp;S (Incl. TS)</field>
            <field name="description">3.7% achat Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="3.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_37"/>
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
            <field name="name">TVA 3.7% sur invest. et autres ch. (TS)</field>
            <field name="description">3.7% invest</field>
            <field name="amount" eval="3.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_37"/>
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
            <field name="name">TVA 3.7% sur invest. et autres ch. (Incl. TS)</field>
            <field name="description">3.7% invest Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="3.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_37"/>
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
            <field name="name">TVA due a 7.7% (TN)</field>
            <field name="description">7.7%</field>
            <field name="amount" eval="7.7"/>
            <field name="sequence" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_77"/>
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
            <field name="name">TVA due à 7.7% (Incl. TN)</field>
            <field name="description">7.7% Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="7.7"/>
            <field name="sequence" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_tva_77"/>
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
            <field name="name">TVA 7.7% sur achat B&amp;S (Incl. TN)</field>
            <field name="description">7.7% achat Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="7.7"/>
            <field name="amount_type">percent</field>
            <field name="sequence" eval="0"/>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_77"/>
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
            <field name="name">TVA 7.7% sur invest. et autres ch. (TN)</field>
            <field name="description">7.7% invest.</field>
            <field name="amount" eval="7.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_77"/>
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
            <field name="name">TVA 7.7% sur invest. et autres ch. (Incl. TN)</field>
            <field name="description">7.7% invest. Incl.</field>
            <field name="price_include" eval="1"/>
            <field name="amount" eval="7.7"/>
            <field name="amount_type">percent</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_tva_77"/>
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
            <field name="name">TVA due a 0% (Exportations)</field>
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
            <field name="name">TVA 0% exclue</field>
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
            <field name="name">TVA 0% Importations de biens et services</field>
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
            <field name="name">Dédouanement TVA (biens et services)</field>
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
            <field name="name">Dédouanement TVA (invest. et autres ch.)</field>
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
            <field name="name">TVA due a 7.7% (TN) (return)</field>
            <field name="description">7.7% achat (return)</field>
            <field name="amount" eval="-7.7"/>
            <field name="amount_type">percent</field>
            <field name="sequence" eval="0"/>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">none</field>
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
            <field name="name">TVA 7.7% sur achat B&amp;S (TN)</field>
            <field name="description">7.7% achat</field>
            <field name="amount" eval="7.7"/>
            <field name="amount_type">percent</field>
            <field name="sequence" eval="0"/>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="type_tax_use">purchase</field>
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
            <field name="description">7.7% achat</field>
            <field name="name">TVA 7.7% sur achat service a l'etranger (reverse charge)</field>
            <field name="amount_type">group</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nch_chart_template"/>
            <field name="children_tax_ids" eval="[(6, 0, [ref('vat_77_purchase_return'), ref('vat_77_purchase')])]"/>
        </record>


        <!-- Taxes for other movements -->
        <record model="account.tax.template" id="vat_other_movements_900">
            <field name="name">Subventions, taxes touristiques à 0%</field>
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
            <field name="name">Dons, dividendes, dédommagements à 0%</field>
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
            <field name="spoken_languages" eval="'it_IT;de_DE;de_CH;en_US;en_GB'"/>
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
    </record>
</odoo>

```

## File: i18n_extra\l10n_ch.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_ch
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 14.0+e\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2023-07-19 13:58+0000\n"
"PO-Revision-Date: 2023-07-19 13:58+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_ch
#: model:ir.actions.report,print_report_name:l10n_ch.l10n_ch_isr_report
msgid "'ISR-%s' % object.name"
msgstr ""

#. module: l10n_ch
#: model:ir.actions.report,print_report_name:l10n_ch.l10n_ch_qr_report
msgid "'QR-bill-%s' % object.name"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_other_movements_910
#: model:account.tax.template,description:l10n_ch.vat_other_movements_910
msgid "0% dons"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_O_exclude
#: model:account.tax.template,description:l10n_ch.vat_O_exclude
msgid "0% excl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_O_import
#: model:account.tax.template,description:l10n_ch.vat_O_import
msgid "0% import."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_other_movements_900
#: model:account.tax.template,description:l10n_ch.vat_other_movements_900
msgid "0% subventions"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_100_import
#: model:account.tax.template,description:l10n_ch.vat_100_import
msgid "100% imp."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_100_import_invest
#: model:account.tax.template,description:l10n_ch.vat_100_import_invest
msgid "100% imp.invest."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_25
#: model:account.tax.template,description:l10n_ch.vat_25
msgid "2.5%"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_25_incl
#: model:account.tax.template,description:l10n_ch.vat_25_incl
msgid "2.5% Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_25_purchase
#: model:account.tax.template,description:l10n_ch.vat_25_purchase
msgid "2.5% achat"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_25_purchase_incl
#: model:account.tax.template,description:l10n_ch.vat_25_purchase_incl
msgid "2.5% achat Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_25_invest
#: model:account.tax.template,description:l10n_ch.vat_25_invest
msgid "2.5% invest."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_25_invest_incl
#: model:account.tax.template,description:l10n_ch.vat_25_invest_incl
msgid "2.5% invest. Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_sale_26
#: model:account.tax.template,description:l10n_ch.vat_sale_26
msgid "2.6%"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_sale_26_incl
#: model:account.tax.template,description:l10n_ch.vat_sale_26_incl
msgid "2.6% Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_purchase_26
#: model:account.tax.template,description:l10n_ch.vat_purchase_26
msgid "2.6% achat"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_purchase_26_incl
#: model:account.tax.template,description:l10n_ch.vat_purchase_26_incl
msgid "2.6% achat Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_purchase_26_invest
#: model:account.tax.template,description:l10n_ch.vat_purchase_26_invest
msgid "2.6% invest."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_purchase_26_invest_incl
#: model:account.tax.template,description:l10n_ch.vat_purchase_26_invest_incl
msgid "2.6% invest. Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_200
msgid ""
"200 - Total des contre-prestations convenues ou reçues, y c. de prestations "
"imposées par option, de transferts par procédure de déclaration, de "
"prestations à l’étranger"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_205
msgid "205"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_205
msgid ""
"205 - Contre-prestations déclarées sous ch. 200 qui proviennent de "
"prestations exclues du champ de l’impôt (art. 21) pour lesquelles il a été "
"opté en vertu de l’art. 22"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_220_289
msgid "220"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_220_289
msgid ""
"220 - Prestations exonérées (p. ex. exportations, art. 23), prestations "
"exonérées fournies à des institutions et à des personnes bénéficiaires (art."
" 107, al. 1, let. a)"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_221
msgid "221"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_221
msgid "221 - Prestations fournies à l’étranger"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_225
msgid "225"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_225
msgid "225 - Transferts avec la procédure de déclaration (art. 38)"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_230
msgid "230"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_230
msgid ""
"230 - Prestations exclues du champ de l’impôt (art. 21) fournies sur le "
"territoire suisse pour lesquelles il n’a pas été opté selon l’art. 22"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_235
msgid "235"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_235
msgid ""
"235 - Diminutions de la contre-prestation telles que rabais, escomptes, etc."
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_280
msgid "280"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_280
msgid "280 - Divers (p.ex. valeur du terrain)"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_289
msgid "289 - Déductions (Total ch. 220 à 280)"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_299
msgid "299 - Total du chiffre d’affaires imposable (ch. 200 moins ch. 289)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_37
#: model:account.tax.template,description:l10n_ch.vat_37
msgid "3.7%"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_37_incl
#: model:account.tax.template,description:l10n_ch.vat_37_incl
msgid "3.7% Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_37_purchase
#: model:account.tax.template,description:l10n_ch.vat_37_purchase
msgid "3.7% achat"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_37_purchase_incl
#: model:account.tax.template,description:l10n_ch.vat_37_purchase_incl
msgid "3.7% achat Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_37_invest
#: model:account.tax.template,description:l10n_ch.vat_37_invest
msgid "3.7% invest"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_37_invest_incl
#: model:account.tax.template,description:l10n_ch.vat_37_invest_incl
msgid "3.7% invest Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_sale_38
#: model:account.tax.template,description:l10n_ch.vat_sale_38
msgid "3.8%"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_sale_38_incl
#: model:account.tax.template,description:l10n_ch.vat_sale_38_incl
msgid "3.8% Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_purchase_38
#: model:account.tax.template,description:l10n_ch.vat_purchase_38
msgid "3.8% achat"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_purchase_38_incl
#: model:account.tax.template,description:l10n_ch.vat_purchase_38_incl
msgid "3.8% achat Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_purchase_38_invest
#: model:account.tax.template,description:l10n_ch.vat_purchase_38_invest
msgid "3.8% invest"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_purchase_38_invest_incl
#: model:account.tax.template,description:l10n_ch.vat_purchase_38_invest_incl
msgid "3.8% invest Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_302a
msgid "302a"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_302a
msgid "302a - Taux normal (7,7%): Prestations CHF jusqu'au 31.12.2023"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_302b
msgid "302b"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_302b
msgid "302b - Taux normal (7,7%): Impôt CHF / ct. jusqu'au 31.12.2023"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_303a
msgid "303a"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_303a
msgid "303a - Taux normal (8,1%): Prestations CHF dès le 01.01.2024"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_303b
msgid "303b"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_303b
msgid "303b - Taux normal (8,1%): Impôt CHF / ct. dès le 01.01.2024"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_312a
msgid "312a"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_312a
msgid "312a - Taux réduit (2,5%): Prestations CHF jusqu'au 31.12.2023"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_312b
msgid "312b"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_312b
msgid "312b - Taux réduit (2,5%): Impôt CHF / ct. jusqu'au 31.12.2023"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_313a
msgid "313a"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_313a
msgid "313a - Taux réduit (2,6%): Prestations CHF dès le 01.01.2024"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_313b
msgid "313b"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_313b
msgid "313b - Taux réduit (2,6%): Impôt CHF / ct. dès le 01.01.2024"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_342a
msgid "342a"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_342a
msgid ""
"342a - Taux spécial pour l'hébergement (3,7%): Prestations CHF jusqu'au "
"31.12.2023"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_342b
msgid "342b"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_342b
msgid ""
"342b - Taux spécial pour l'hébergement (3,7%): Impôt CHF / ct. jusqu'au "
"31.12.2023"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_343a
msgid "343a"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_343a
msgid ""
"343a - Taux spécial pour l'hébergement (3,8%): Prestations CHF dès le "
"01.01.2024"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_343b
msgid "343b"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_343b
msgid ""
"343b - Taux spécial pour l'hébergement (3,8%): Impôt CHF / ct. dès le "
"01.01.2024"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_382a
msgid "382a"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_382a
msgid "382a - Impôt sur les acquisitions: Prestations CHF jusqu'au 31.12.2023"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_382b
msgid "382b"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_382b
msgid "382b - Impôt sur les acquisitions: Impôt CHF / ct. jusqu'au 31.12.2023"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_383a
msgid "383a"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_383a
msgid "383a - Impôt sur les acquisitions: Prestations CHF dès le 01.01.2024"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_383b
msgid "383b"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_383b
msgid "383b - Impôt sur les acquisitions: Impôt CHF / ct. dès le 01.01.2024"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_399
msgid "399 - Total de l’impôt dû"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_400
msgid "400"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_400
msgid ""
"400 - Impôt préalable grevant les coûts en matériel et en prestations de "
"services"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_405
msgid "405"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_405
msgid ""
"405 - Impôt préalable grevant les investissements et autres charges "
"d’exploitation"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_410
msgid "410"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_410
msgid "410 - Dégrèvement ultérieur de l’impôt préalable (art. 32)"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_415
msgid "415"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_415
msgid ""
"415 - Corrections de l’impôt préalable: double affectation (art. 30), "
"prestations à soi-même (art. 31)"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_420
msgid "420"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_420
msgid ""
"420 - Réductions de la déduction de l’impôt préalable: prestations n’étant "
"pas considérées comme des contre-prestations, telles subventions, taxes "
"touristiques (art. 33, al. 2)"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_479
msgid "479 - Total ch. 400 à 420"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_500
msgid "500 - Montant à payer"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_510
msgid "510 - Solde en faveur de l’assujetti"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_77
#: model:account.tax.template,description:l10n_ch.vat_77
msgid "7.7%"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_77_incl
#: model:account.tax.template,description:l10n_ch.vat_77_incl
msgid "7.7% Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_77_purchase
#: model:account.tax,description:l10n_ch.1_vat_77_purchase_reverse
#: model:account.tax.template,description:l10n_ch.vat_77_purchase
#: model:account.tax.template,description:l10n_ch.vat_77_purchase_reverse
msgid "7.7% achat"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_77_purchase_return
#: model:account.tax.template,description:l10n_ch.vat_77_purchase_return
msgid "7.7% achat (return)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_77_purchase_incl
#: model:account.tax.template,description:l10n_ch.vat_77_purchase_incl
msgid "7.7% achat Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_77_invest
#: model:account.tax.template,description:l10n_ch.vat_77_invest
msgid "7.7% invest."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_77_invest_incl
#: model:account.tax.template,description:l10n_ch.vat_77_invest_incl
msgid "7.7% invest. Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_sale_81
#: model:account.tax.template,description:l10n_ch.vat_sale_81
msgid "8.1%"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_sale_81_incl
#: model:account.tax.template,description:l10n_ch.vat_sale_81_incl
msgid "8.1% Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_purchase_81
#: model:account.tax,description:l10n_ch.1_vat_purchase_81_reverse
#: model:account.tax.template,description:l10n_ch.vat_purchase_81
#: model:account.tax.template,description:l10n_ch.vat_purchase_81_reverse
msgid "8.1% achat"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_purchase_81_return
#: model:account.tax.template,description:l10n_ch.vat_purchase_81_return
msgid "8.1% achat (return)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_purchase_81_incl
#: model:account.tax.template,description:l10n_ch.vat_purchase_81_incl
msgid "8.1% achat Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_purchase_81_invest
#: model:account.tax.template,description:l10n_ch.vat_purchase_81_invest
msgid "8.1% invest."
msgstr ""

#. module: l10n_ch
#: model:account.tax,description:l10n_ch.1_vat_purchase_81_invest_incl
#: model:account.tax.template,description:l10n_ch.vat_purchase_81_invest_incl
msgid "8.1% invest. Incl."
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_900
msgid "900"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_900
msgid ""
"900 - Subventions, taxes touristiques perçues par les offices du tourisme, "
"contributions aux établissements d'élimination des déchets et "
"d'approvisionnement en eau"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,tag_name:l10n_ch.account_tax_report_line_chtax_910
msgid "910"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_910
msgid "910 - Les dons, les dividendes, les dédommagements, etc."
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.res_config_settings_view_form
msgid "<span class=\"o_form_label\">ISR scan line offset</span>"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_swissqr_template
msgid "<span class=\"title\">Acceptance point</span>"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_swissqr_template
msgid "<span class=\"title\">Reference</span>"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_swissqr_template
msgid "<span>Account / Payable to</span>"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_swissqr_template
msgid "<span>Account / Payable to</span><br/>"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_swissqr_template
msgid "<span>Additional information</span>"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_swissqr_template
msgid "<span>Amount</span>"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_swissqr_template
msgid "<span>Amount</span><br/>"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_swissqr_template
msgid "<span>Currency</span>"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_swissqr_template
msgid "<span>Payable by</span>"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_swissqr_template
msgid "<span>Payment part</span>"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_swissqr_template
msgid "<span>Receipt</span>"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_swissqr_template
msgid "<span>Reference</span>"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4200
#: model:account.account.template,name:l10n_ch.ch_coa_4200
msgid "Achats de marchandises destinées à la revente"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2030
#: model:account.account.template,name:l10n_ch.ch_coa_2030
msgid "Acomptes de clients"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1208
#: model:account.account.template,name:l10n_ch.ch_coa_1208
msgid "Acomptes sur les marchandises commerciales"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1218
#: model:account.account.template,name:l10n_ch.ch_coa_1218
msgid "Acomptes sur matières premières"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1149
#: model:account.account.template,name:l10n_ch.ch_coa_1149
msgid "Ajustement de la valeur des avances et des prêts"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1199
#: model:account.account.template,name:l10n_ch.ch_coa_1199
msgid "Ajustement de la valeur des créances à court terme"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1449
#: model:account.account.template,name:l10n_ch.ch_coa_1449
msgid "Ajustement de la valeur des créances à long terme"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1779
#: model:account.account.template,name:l10n_ch.ch_coa_1779
msgid "Ajustement de la valeur des goodwill"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1489
#: model:account.account.template,name:l10n_ch.ch_coa_1489
msgid "Ajustement de la valeur des participations"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1069
#: model:account.account,name:l10n_ch.1_ch_coa_1409
#: model:account.account.template,name:l10n_ch.ch_coa_1069
#: model:account.account.template,name:l10n_ch.ch_coa_1409
msgid "Ajustement de la valeur des titres"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_6800
#: model:account.account.template,name:l10n_ch.ch_coa_6800
msgid ""
"Amortissements et ajustements de valeur des postes sur immobilisations "
"corporelles"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1519
#: model:account.account.template,name:l10n_ch.ch_coa_1519
msgid "Amortissements sur le mobilier et les installations"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1599
#: model:account.account.template,name:l10n_ch.ch_coa_1599
msgid "Amortissements sur les autres immobilisations corporelles meubles"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1709
#: model:account.account.template,name:l10n_ch.ch_coa_1709
msgid "Amortissements sur les brevets, know-how, licences, droits, dév."
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1609
#: model:account.account.template,name:l10n_ch.ch_coa_1609
msgid "Amortissements sur les immeubles d’exploitation"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1559
#: model:account.account.template,name:l10n_ch.ch_coa_1559
msgid "Amortissements sur les installations de stockage"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1529
#: model:account.account.template,name:l10n_ch.ch_coa_1529
msgid "Amortissements sur les machines de bureau, inf. et syst. comm."
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1509
#: model:account.account.template,name:l10n_ch.ch_coa_1509
msgid "Amortissements sur les machines et appareils"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1549
#: model:account.account.template,name:l10n_ch.ch_coa_1549
msgid "Amortissements sur les outillages et appareils"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1539
#: model:account.account.template,name:l10n_ch.ch_coa_1539
msgid "Amortissements sur les véhicules"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1579
#: model:account.account.template,name:l10n_ch.ch_coa_1579
msgid "Amortissements sur les équipements et installations"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2270
#: model:account.account.template,name:l10n_ch.ch_coa_2270
msgid "Assurances sociales et institutions de prévoyance"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_6300
#: model:account.account.template,name:l10n_ch.ch_coa_6300
msgid "Assurances-choses, droits, taxes, autorisations"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_5800
#: model:account.account.template,name:l10n_ch.ch_coa_5800
msgid "Autres charges du personnel"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_6700
#: model:account.account.template,name:l10n_ch.ch_coa_6700
msgid "Autres charges d‘exploitation"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1190
#: model:account.account.template,name:l10n_ch.ch_coa_1190
msgid "Autres créances à court terme"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2210
#: model:account.account.template,name:l10n_ch.ch_coa_2210
msgid "Autres dettes à court terme"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2140
#: model:account.account.template,name:l10n_ch.ch_coa_2140
msgid "Autres dettes à court terme rémunérées"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2500
#: model:account.account.template,name:l10n_ch.ch_coa_2500
msgid "Autres dettes à long terme"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1590
#: model:account.account.template,name:l10n_ch.ch_coa_1590
msgid "Autres immobilisations corporelles meubles"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3600
#: model:account.account.template,name:l10n_ch.ch_coa_3600
msgid "Autres ventes et prestations de services"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1140
#: model:account.account.template,name:l10n_ch.ch_coa_1140
msgid "Avances et prêts"
msgstr ""

#. module: l10n_ch
#: model:ir.model,name:l10n_ch.model_res_partner_bank
msgid "Bank Accounts"
msgstr ""

#. module: l10n_ch
#: model:ir.model,name:l10n_ch.model_account_bank_statement_line
msgid "Bank Statement Line"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,help:l10n_ch.field_res_company__l10n_ch_isr_print_bank_location
#: model:ir.model.fields,help:l10n_ch.field_res_config_settings__l10n_ch_isr_print_bank_location
msgid ""
"Boolean option field indicating whether or not the alternate layout (the one"
" printing bank name and address) must be used when generating an ISR."
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,help:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_sent
#: model:ir.model.fields,help:l10n_ch.field_account_move__l10n_ch_isr_sent
#: model:ir.model.fields,help:l10n_ch.field_account_payment__l10n_ch_isr_sent
msgid ""
"Boolean value telling whether or not the ISR corresponding to this invoice "
"has already been printed or sent by mail."
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,help:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_valid
#: model:ir.model.fields,help:l10n_ch.field_account_move__l10n_ch_isr_valid
#: model:ir.model.fields,help:l10n_ch.field_account_payment__l10n_ch_isr_valid
msgid ""
"Boolean value. True iff all the data required to generate the ISR are "
"present"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1700
#: model:account.account.template,name:l10n_ch.ch_coa_1700
msgid "Brevets, know-how, licences, droits, développement"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2979
#: model:account.account.template,name:l10n_ch.ch_coa_2979
msgid "Bénéfice / perte de l’exercice"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2970
#: model:account.account.template,name:l10n_ch.ch_coa_2970
msgid "Bénéfice / perte reporté"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_setup_bank_manual_config__l10n_ch_isr_subscription_chf
#: model:ir.model.fields,field_description:l10n_ch.field_res_partner_bank__l10n_ch_isr_subscription_chf
msgid "CHF ISR Subscription Number"
msgstr ""

#. module: l10n_ch
#: code:addons/l10n_ch/models/account_invoice.py:0
#, python-format
msgid ""
"Cannot generate the QR-bill. Please check you have configured the address of"
" your company and debtor. If you are using a QR-IBAN, also check the "
"invoice's payment reference is a QR reference."
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1850
#: model:account.account.template,name:l10n_ch.ch_coa_1850
msgid ""
"Capital actions, capital social, droits de participations ou capital de "
"fondation non versés"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2800
#: model:account.account.template,name:l10n_ch.ch_coa_2800
msgid "Capital-actions, capital social, capital de fondation"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4521
#: model:account.account.template,name:l10n_ch.ch_coa_4521
msgid "Charbon, briquettes, bois"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_7010
#: model:account.account.template,name:l10n_ch.ch_coa_7010
msgid "Charges accessoires"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_6000
#: model:account.account.template,name:l10n_ch.ch_coa_6000
msgid "Charges de locaux"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4000
#: model:account.account.template,name:l10n_ch.ch_coa_4000
msgid "Charges de matériel de l‘atelier"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_5900
#: model:account.account.template,name:l10n_ch.ch_coa_5900
msgid "Charges de personnels temporaires"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_6200
#: model:account.account.template,name:l10n_ch.ch_coa_6200
msgid "Charges de véhicules et de transport"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_7510
#: model:account.account.template,name:l10n_ch.ch_coa_7510
msgid "Charges des immeubles d‘exploitation"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_6500
#: model:account.account.template,name:l10n_ch.ch_coa_6500
msgid "Charges d‘administration"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_6400
#: model:account.account.template,name:l10n_ch.ch_coa_6400
msgid "Charges d’énergie et évacuation des déchets"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_6570
#: model:account.account.template,name:l10n_ch.ch_coa_6570
msgid "Charges et leasing d’informatique"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_8500
#: model:account.account.template,name:l10n_ch.ch_coa_8500
msgid "Charges extraordinaires, exceptionnelles ou hors période"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_6900
#: model:account.account.template,name:l10n_ch.ch_coa_6900
msgid "Charges financières"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_8000
#: model:account.account.template,name:l10n_ch.ch_coa_8000
msgid "Charges hors exploitation"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1300
#: model:account.account.template,name:l10n_ch.ch_coa_1300
msgid "Charges payées d‘avance"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_5700
#: model:account.account.template,name:l10n_ch.ch_coa_5700
msgid "Charges sociales"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2300
#: model:account.account.template,name:l10n_ch.ch_coa_2300
msgid "Charges à payer"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_bank_statement_import_journal_creation__l10n_ch_postal
#: model:ir.model.fields,field_description:l10n_ch.field_account_journal__l10n_ch_postal
msgid "Client Number"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3803
#: model:account.account.template,name:l10n_ch.ch_coa_3803
msgid "Commissions de tiers"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4903
#: model:account.account.template,name:l10n_ch.ch_coa_4903
msgid "Commissions obtenues sur achats"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_bank_statement_import_journal_creation__invoice_reference_model
#: model:ir.model.fields,field_description:l10n_ch.field_account_journal__invoice_reference_model
msgid "Communication Standard"
msgstr ""

#. module: l10n_ch
#: model:ir.model,name:l10n_ch.model_res_company
msgid "Companies"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1099
#: model:account.account.template,name:l10n_ch.ch_coa_1099
msgid "Compte d'attente autre"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1091
#: model:account.account.template,name:l10n_ch.ch_coa_1091
msgid "Compte d'attente pour salaires"
msgstr ""

#. module: l10n_ch
#: model:ir.model,name:l10n_ch.model_res_config_settings
msgid "Config Settings"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3710
#: model:account.account.template,name:l10n_ch.ch_coa_3710
msgid "Consommations propres"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1269
#: model:account.account.template,name:l10n_ch.ch_coa_1269
msgid "Correction de la valeur de stocks de produits finis"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1279
#: model:account.account.template,name:l10n_ch.ch_coa_1279
msgid "Corrections de la valeur des stock produits semi-ouvrés"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1209
#: model:account.account.template,name:l10n_ch.ch_coa_1209
msgid "Corrections de la valeur des stocks de marchandises"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1289
#: model:account.account.template,name:l10n_ch.ch_coa_1289
msgid "Corrections de la valeur des travaux en cours"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1219
#: model:account.account.template,name:l10n_ch.ch_coa_1219
msgid "Corrections de la valeur sur matières premières"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1180
#: model:account.account.template,name:l10n_ch.ch_coa_1180
msgid "Créances envers les assurances sociales et institutions de prévoyance"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2000
#: model:account.account.template,name:l10n_ch.ch_coa_2000
msgid "Créanciers"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_bank_statement_line__l10n_ch_currency_name
#: model:ir.model.fields,field_description:l10n_ch.field_account_move__l10n_ch_currency_name
#: model:ir.model.fields,field_description:l10n_ch.field_account_payment__l10n_ch_currency_name
msgid "Currency Name"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2100
#: model:account.account,name:l10n_ch.1_ch_coa_2400
#: model:account.account.template,name:l10n_ch.ch_coa_2100
#: model:account.account.template,name:l10n_ch.ch_coa_2400
msgid "Dettes bancaires"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2160
#: model:account.account.template,name:l10n_ch.ch_coa_2160
msgid "Dettes envers l'actionnaire"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3806
#: model:account.account,name:l10n_ch.1_ch_coa_4906
#: model:account.account.template,name:l10n_ch.ch_coa_3806
#: model:account.account.template,name:l10n_ch.ch_coa_4906
msgid "Différences de change"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_bank_statement_line__display_name
#: model:ir.model.fields,field_description:l10n_ch.field_account_journal__display_name
#: model:ir.model.fields,field_description:l10n_ch.field_account_move__display_name
#: model:ir.model.fields,field_description:l10n_ch.field_ir_actions_report__display_name
#: model:ir.model.fields,field_description:l10n_ch.field_mail_template__display_name
#: model:ir.model.fields,field_description:l10n_ch.field_report_l10n_ch_qr_report_main__display_name
#: model:ir.model.fields,field_description:l10n_ch.field_res_company__display_name
#: model:ir.model.fields,field_description:l10n_ch.field_res_config_settings__display_name
#: model:ir.model.fields,field_description:l10n_ch.field_res_partner_bank__display_name
msgid "Display Name"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2261
#: model:account.account.template,name:l10n_ch.ch_coa_2261
msgid "Dividendes"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_other_movements_910
#: model:account.tax.template,name:l10n_ch.vat_other_movements_910
msgid "Dons, dividendes, dédommagements à 0%"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4071
#: model:account.account.template,name:l10n_ch.ch_coa_4071
msgid "Droits de douanes à l'importation"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1109
#: model:account.account.template,name:l10n_ch.ch_coa_1109
msgid "Ducroire"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1100
#: model:account.account.template,name:l10n_ch.ch_coa_1100
msgid "Débiteurs"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1101
#: model:account.account.template,name:l10n_ch.ch_coa_1101
msgid "Débiteurs (PoS)"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2201
#: model:account.account.template,name:l10n_ch.ch_coa_2201
msgid "Décompte TVA"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_100_import
#: model:account.tax.template,name:l10n_ch.vat_100_import
msgid "Dédouanement TVA (biens et services)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_100_import_invest
#: model:account.tax.template,name:l10n_ch.vat_100_import_invest
msgid "Dédouanement TVA (invest. et autres ch.)"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4009
#: model:account.account.template,name:l10n_ch.ch_coa_4009
msgid "Déductions obtenues sur achats"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3009
#: model:account.account.template,name:l10n_ch.ch_coa_3009
msgid "Déductions sur ventes"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_setup_bank_manual_config__l10n_ch_isr_subscription_eur
#: model:ir.model.fields,field_description:l10n_ch.field_res_partner_bank__l10n_ch_isr_subscription_eur
msgid "EUR ISR Subscription Number"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4540
#: model:account.account.template,name:l10n_ch.ch_coa_4540
msgid "Eau"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4500
#: model:account.account.template,name:l10n_ch.ch_coa_4500
msgid "Electricité"
msgstr ""

#. module: l10n_ch
#: model:ir.model,name:l10n_ch.model_mail_template
msgid "Email Templates"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2450
#: model:account.account.template,name:l10n_ch.ch_coa_2450
msgid "Emprunts"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2430
#: model:account.account.template,name:l10n_ch.ch_coa_2430
msgid "Emprunts obligataires"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2120
#: model:account.account,name:l10n_ch.1_ch_coa_2420
#: model:account.account.template,name:l10n_ch.ch_coa_2120
#: model:account.account.template,name:l10n_ch.ch_coa_2420
msgid "Engagements de financement par leasing"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_6100
#: model:account.account.template,name:l10n_ch.ch_coa_6100
msgid ""
"Entretien, réparations et remplacement des inst. servant à l’exploitation"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1570
#: model:account.account.template,name:l10n_ch.ch_coa_1570
msgid "Equipements et Installations"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3800
#: model:account.account,name:l10n_ch.1_ch_coa_4900
#: model:account.account.template,name:l10n_ch.ch_coa_3800
#: model:account.account.template,name:l10n_ch.ch_coa_4900
msgid "Escomptes"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4530
#: model:account.account.template,name:l10n_ch.ch_coa_4530
msgid "Essence"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3804
#: model:account.account.template,name:l10n_ch.ch_coa_3804
msgid "Frais d'encaissement"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3807
#: model:account.account.template,name:l10n_ch.ch_coa_3807
msgid "Frais d'expédition"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4072
#: model:account.account.template,name:l10n_ch.ch_coa_4072
msgid "Frais de transport à l'achat"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4070
#: model:account.account.template,name:l10n_ch.ch_coa_4070
msgid "Frêts à l'achat"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4510
#: model:account.account.template,name:l10n_ch.ch_coa_4510
msgid "Gaz"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1770
#: model:account.account.template,name:l10n_ch.ch_coa_1770
msgid "Goodwill"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_res_config_settings__l10n_ch_isr_scan_line_left
msgid "Horizontal offset"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1441
#: model:account.account,name:l10n_ch.1_ch_coa_2451
#: model:account.account.template,name:l10n_ch.ch_coa_1441
#: model:account.account.template,name:l10n_ch.ch_coa_2451
msgid "Hypothèques"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chiffre_af
msgid "I. CHIFFRE D'AFFAIRES"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_bank_statement_line__id
#: model:ir.model.fields,field_description:l10n_ch.field_account_journal__id
#: model:ir.model.fields,field_description:l10n_ch.field_account_move__id
#: model:ir.model.fields,field_description:l10n_ch.field_ir_actions_report__id
#: model:ir.model.fields,field_description:l10n_ch.field_mail_template__id
#: model:ir.model.fields,field_description:l10n_ch.field_report_l10n_ch_qr_report_main__id
#: model:ir.model.fields,field_description:l10n_ch.field_res_company__id
#: model:ir.model.fields,field_description:l10n_ch.field_res_config_settings__id
#: model:ir.model.fields,field_description:l10n_ch.field_res_partner_bank__id
msgid "ID"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_calc_impot
msgid "II. CALCUL DE L'IMPÔT"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_chtax_autres_mouv
msgid "III. AUTRES MOUVEMENTS DE FONDS"
msgstr ""

#. module: l10n_ch
#: model:ir.actions.report,name:l10n_ch.l10n_ch_isr_report
msgid "ISR"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.isr_partner_bank_form
#: model_terms:ir.ui.view,arch_db:l10n_ch.setup_bank_account_wizard_inherit
msgid "ISR Client Identification Number"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_isr_report_template
msgid "ISR for invoice"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,help:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_number_spaced
#: model:ir.model.fields,help:l10n_ch.field_account_move__l10n_ch_isr_number_spaced
#: model:ir.model.fields,help:l10n_ch.field_account_payment__l10n_ch_isr_number_spaced
msgid ""
"ISR number split in blocks of 5 characters (right-justified), to generate "
"ISR report."
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.isr_invoice_search_view
msgid "ISR reference number"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,help:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_subscription
#: model:ir.model.fields,help:l10n_ch.field_account_move__l10n_ch_isr_subscription
#: model:ir.model.fields,help:l10n_ch.field_account_payment__l10n_ch_isr_subscription
msgid ""
"ISR subscription number identifying your company or your bank to generate "
"ISR."
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,help:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_subscription_formatted
#: model:ir.model.fields,help:l10n_ch.field_account_move__l10n_ch_isr_subscription_formatted
#: model:ir.model.fields,help:l10n_ch.field_account_payment__l10n_ch_isr_subscription_formatted
msgid ""
"ISR subscription number your company or your bank, formated with '-' and "
"without the padding zeros, to generate ISR report."
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1600
#: model:account.account.template,name:l10n_ch.ch_coa_1600
msgid "Immeubles d’exploitation"
msgstr ""

#. module: l10n_ch
#: model:account.fiscal.position,name:l10n_ch.1_fiscal_position_template_import
#: model:account.fiscal.position.template,name:l10n_ch.fiscal_position_template_import
msgid "Import/Export"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_tax_amount_1
msgid "Impôt CHF / ct. dès le 01.01.2024"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_tax_amount_2
msgid "Impôt CHF / ct. jusqu'au 31.12.2023"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1176
#: model:account.account.template,name:l10n_ch.ch_coa_1176
msgid "Impôt anticipé"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2206
#: model:account.account.template,name:l10n_ch.ch_coa_2206
msgid "Impôt anticipé dû"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1171
#: model:account.account.template,name:l10n_ch.ch_coa_1171
msgid ""
"Impôt préalable: TVA s/investissements et autres charges d’exploitation"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1170
#: model:account.account.template,name:l10n_ch.ch_coa_1170
msgid "Impôt préalable: TVA s/matériel, marchandises, prestations et énergie"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1189
#: model:account.account,name:l10n_ch.1_ch_coa_2279
#: model:account.account.template,name:l10n_ch.ch_coa_1189
#: model:account.account.template,name:l10n_ch.ch_coa_2279
msgid "Impôt à la source"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2208
#: model:account.account,name:l10n_ch.1_ch_coa_8900
#: model:account.account.template,name:l10n_ch.ch_coa_2208
#: model:account.account.template,name:l10n_ch.ch_coa_8900
msgid "Impôts directs"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1550
#: model:account.account.template,name:l10n_ch.ch_coa_1550
msgid "Installations de stockage"
msgstr ""

#. module: l10n_ch
#: model:ir.model,name:l10n_ch.model_account_journal
msgid "Journal"
msgstr ""

#. module: l10n_ch
#: model:ir.model,name:l10n_ch.model_account_move
msgid "Journal Entry"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_needs_fixing
#: model:ir.model.fields,field_description:l10n_ch.field_account_move__l10n_ch_isr_needs_fixing
#: model:ir.model.fields,field_description:l10n_ch.field_account_payment__l10n_ch_isr_needs_fixing
msgid "L10N Ch Isr Needs Fixing"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_number
#: model:ir.model.fields,field_description:l10n_ch.field_account_move__l10n_ch_isr_number
#: model:ir.model.fields,field_description:l10n_ch.field_account_payment__l10n_ch_isr_number
msgid "L10N Ch Isr Number"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_number_spaced
#: model:ir.model.fields,field_description:l10n_ch.field_account_move__l10n_ch_isr_number_spaced
#: model:ir.model.fields,field_description:l10n_ch.field_account_payment__l10n_ch_isr_number_spaced
msgid "L10N Ch Isr Number Spaced"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_optical_line
#: model:ir.model.fields,field_description:l10n_ch.field_account_move__l10n_ch_isr_optical_line
#: model:ir.model.fields,field_description:l10n_ch.field_account_payment__l10n_ch_isr_optical_line
msgid "L10N Ch Isr Optical Line"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_sent
#: model:ir.model.fields,field_description:l10n_ch.field_account_move__l10n_ch_isr_sent
#: model:ir.model.fields,field_description:l10n_ch.field_account_payment__l10n_ch_isr_sent
msgid "L10N Ch Isr Sent"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_subscription
#: model:ir.model.fields,field_description:l10n_ch.field_account_move__l10n_ch_isr_subscription
#: model:ir.model.fields,field_description:l10n_ch.field_account_payment__l10n_ch_isr_subscription
msgid "L10N Ch Isr Subscription"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_subscription_formatted
#: model:ir.model.fields,field_description:l10n_ch.field_account_move__l10n_ch_isr_subscription_formatted
#: model:ir.model.fields,field_description:l10n_ch.field_account_payment__l10n_ch_isr_subscription_formatted
msgid "L10N Ch Isr Subscription Formatted"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_valid
#: model:ir.model.fields,field_description:l10n_ch.field_account_move__l10n_ch_isr_valid
#: model:ir.model.fields,field_description:l10n_ch.field_account_payment__l10n_ch_isr_valid
msgid "L10N Ch Isr Valid"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_setup_bank_manual_config__l10n_ch_show_subscription
#: model:ir.model.fields,field_description:l10n_ch.field_res_partner_bank__l10n_ch_show_subscription
msgid "L10N Ch Show Subscription"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_bank_statement_line____last_update
#: model:ir.model.fields,field_description:l10n_ch.field_account_journal____last_update
#: model:ir.model.fields,field_description:l10n_ch.field_account_move____last_update
#: model:ir.model.fields,field_description:l10n_ch.field_ir_actions_report____last_update
#: model:ir.model.fields,field_description:l10n_ch.field_mail_template____last_update
#: model:ir.model.fields,field_description:l10n_ch.field_report_l10n_ch_qr_report_main____last_update
#: model:ir.model.fields,field_description:l10n_ch.field_res_company____last_update
#: model:ir.model.fields,field_description:l10n_ch.field_res_config_settings____last_update
#: model:ir.model.fields,field_description:l10n_ch.field_res_partner_bank____last_update
msgid "Last Modified on"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_6260
#: model:account.account.template,name:l10n_ch.ch_coa_6260
msgid "Leasing et location de véhicules"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_6105
#: model:account.account.template,name:l10n_ch.ch_coa_6105
msgid "Leasing immobilisations corporelles meubles"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_l10nch_chart_template_liquidity_transfer
#: model:account.account.template,name:l10n_ch.l10nch_chart_template_liquidity_transfer
msgid "Liquidity Transfer"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1520
#: model:account.account.template,name:l10n_ch.ch_coa_1520
msgid "Machines de bureau, informatique, systèmes de communication"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1500
#: model:account.account.template,name:l10n_ch.ch_coa_1500
msgid "Machines et appareils"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1200
#: model:account.account.template,name:l10n_ch.ch_coa_1200
msgid "Marchandises commerciales"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1250
#: model:account.account.template,name:l10n_ch.ch_coa_1250
msgid "Marchandises en consignation"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1220
#: model:account.account.template,name:l10n_ch.ch_coa_1220
msgid "Matières auxiliaires"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1230
#: model:account.account.template,name:l10n_ch.ch_coa_1230
msgid "Matières consommables"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1210
#: model:account.account.template,name:l10n_ch.ch_coa_1210
msgid "Matières premières"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4520
#: model:account.account.template,name:l10n_ch.ch_coa_4520
msgid "Mazout"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1510
#: model:account.account.template,name:l10n_ch.ch_coa_1510
msgid "Mobilier et installations"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.res_config_settings_view_form
msgid "Offset to move the scan line in mm"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,help:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_optical_line
#: model:ir.model.fields,help:l10n_ch.field_account_move__l10n_ch_isr_optical_line
#: model:ir.model.fields,help:l10n_ch.field_account_payment__l10n_ch_isr_optical_line
msgid "Optical reading line, as it will be printed on ISR"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1540
#: model:account.account.template,name:l10n_ch.ch_coa_1540
msgid "Outillages et appareils"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1480
#: model:account.account.template,name:l10n_ch.ch_coa_1480
msgid "Participations"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4086
#: model:account.account.template,name:l10n_ch.ch_coa_4086
msgid "Pertes de matières"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3805
#: model:account.account.template,name:l10n_ch.ch_coa_3805
msgid "Pertes sur créances clients, variation ducroire"
msgstr ""

#. module: l10n_ch
#: model:account.chart.template,name:l10n_ch.l10nch_chart_template
msgid "Plan comptable 2015 (Suisse)"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.isr_invoice_form
msgid ""
"Please fill in a correct ISR reference in the payment reference.  The banks "
"will refuse your payment file otherwise."
msgstr ""

#. module: l10n_ch
#: code:addons/l10n_ch/models/res_bank.py:0
#, python-format
msgid "Postal"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_res_company__l10n_ch_isr_preprinted_account
#: model:ir.model.fields,field_description:l10n_ch.field_res_config_settings__l10n_ch_isr_preprinted_account
msgid "Preprinted account"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_res_company__l10n_ch_isr_preprinted_bank
#: model:ir.model.fields,field_description:l10n_ch.field_res_config_settings__l10n_ch_isr_preprinted_bank
msgid "Preprinted bank"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4400
#: model:account.account.template,name:l10n_ch.ch_coa_4400
msgid "Prestations / travaux de tiers"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_supplies_1
msgid "Prestations CHF dès le 01.01.2024"
msgstr ""

#. module: l10n_ch
#: model:account.tax.report.line,name:l10n_ch.account_tax_report_line_supplies_2
msgid "Prestations CHF jusqu'au 31.12.2023"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3700
#: model:account.account.template,name:l10n_ch.ch_coa_3700
msgid "Prestations propres"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.isr_invoice_form
msgid "Print QR-bill"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_res_company__l10n_ch_isr_print_bank_location
msgid "Print bank location"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_res_config_settings__l10n_ch_isr_print_bank_location
msgid "Print bank on ISR"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.res_config_settings_view_form
msgid ""
"Print the coordinates of your bank under the 'Payment for' title of the ISR.\n"
"                                Your address will be moved to the 'in favour of' section."
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_7000
#: model:account.account.template,name:l10n_ch.ch_coa_7000
msgid "Produits accessoires"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_7500
#: model:account.account.template,name:l10n_ch.ch_coa_7500
msgid "Produits des immeubles d‘exploitation"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2301
#: model:account.account.template,name:l10n_ch.ch_coa_2301
msgid "Produits encaissés d’avance"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_8510
#: model:account.account.template,name:l10n_ch.ch_coa_8510
msgid "Produits extraordinaires, exceptionnels ou hors période"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_6950
#: model:account.account.template,name:l10n_ch.ch_coa_6950
msgid "Produits financiers"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_8100
#: model:account.account.template,name:l10n_ch.ch_coa_8100
msgid "Produits hors exploitation"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1301
#: model:account.account.template,name:l10n_ch.ch_coa_1301
msgid "Produits à recevoir"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2980
#: model:account.account.template,name:l10n_ch.ch_coa_2980
msgid ""
"Propres actions, parts sociales, droits de participations (poste négatif)"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2600
#: model:account.account.template,name:l10n_ch.ch_coa_2600
msgid "Provisions"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2330
#: model:account.account.template,name:l10n_ch.ch_coa_2330
msgid "Provisions à court terme"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1440
#: model:account.account.template,name:l10n_ch.ch_coa_1440
msgid "Prêts"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_6600
#: model:account.account.template,name:l10n_ch.ch_coa_6600
msgid "Publicité"
msgstr ""

#. module: l10n_ch
#: model:ir.actions.report,name:l10n_ch.l10n_ch_qr_report
msgid "QR-bill"
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_swissqr_template
msgid "QR-bill for invoice"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3801
#: model:account.account.template,name:l10n_ch.ch_coa_3801
msgid "Rabais et réduction de prix"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4901
#: model:account.account.template,name:l10n_ch.ch_coa_4901
msgid "Rabais et réductions de prix"
msgstr ""

#. module: l10n_ch
#: model:ir.model,name:l10n_ch.model_ir_actions_report
msgid "Report Action"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3802
#: model:account.account,name:l10n_ch.1_ch_coa_4092
#: model:account.account.template,name:l10n_ch.ch_coa_3802
#: model:account.account.template,name:l10n_ch.ch_coa_4092
msgid "Ristournes"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2940
#: model:account.account.template,name:l10n_ch.ch_coa_2940
msgid "Réserves d‘évaluation"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2960
#: model:account.account.template,name:l10n_ch.ch_coa_2960
msgid "Réserves libres"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2950
#: model:account.account.template,name:l10n_ch.ch_coa_2950
msgid "Réserves légales issues du bénéfice"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2900
#: model:account.account.template,name:l10n_ch.ch_coa_2900
msgid "Réserves légales issues du capital"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_5000
#: model:account.account.template,name:l10n_ch.ch_coa_5000
msgid "Salaires"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_res_company__l10n_ch_isr_scan_line_left
msgid "Scan line horizontal offset (mm)"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_res_company__l10n_ch_isr_scan_line_top
msgid "Scan line vertical offset (mm)"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1260
#: model:account.account.template,name:l10n_ch.ch_coa_1260
msgid "Stocks de produits finis"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1270
#: model:account.account.template,name:l10n_ch.ch_coa_1270
msgid "Stocks de produits semi-ouvrés"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_other_movements_900
#: model:account.tax.template,name:l10n_ch.vat_other_movements_900
msgid "Subventions, taxes touristiques à 0%"
msgstr ""

#. module: l10n_ch
#: model:account.fiscal.position,name:l10n_ch.1_fiscal_position_template_1
#: model:account.fiscal.position.template,name:l10n_ch.fiscal_position_template_1
msgid "Suisse national"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_account_setup_bank_manual_config__l10n_ch_postal
#: model:ir.model.fields,field_description:l10n_ch.field_res_partner_bank__l10n_ch_postal
msgid "Swiss Postal Account"
msgstr ""

#. module: l10n_ch
#: code:addons/l10n_ch/models/res_bank.py:0
#, python-format
msgid "Swiss QR bill"
msgstr ""

#. module: l10n_ch
#: model:ir.model,name:l10n_ch.model_report_l10n_ch_qr_report_main
msgid "Swiss QR-bill report"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields.selection,name:l10n_ch.selection__account_journal__invoice_reference_model__ch
#: model:ir.ui.menu,name:l10n_ch.account_reports_ch_statements_menu
msgid "Switzerland"
msgstr ""

#. module: l10n_ch
#: model:account.tax.group,name:l10n_ch.tax_group_tva_0
msgid "TVA 0%"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_O_import
#: model:account.tax.template,name:l10n_ch.vat_O_import
msgid "TVA 0% Importations de biens et services"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_O_exclude
#: model:account.tax.template,name:l10n_ch.vat_O_exclude
msgid "TVA 0% exclue"
msgstr ""

#. module: l10n_ch
#: model:account.tax.group,name:l10n_ch.tax_group_tva_100
msgid "TVA 100%"
msgstr ""

#. module: l10n_ch
#: model:account.tax.group,name:l10n_ch.tax_group_tva_25
msgid "TVA 2.5%"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_25_purchase_incl
#: model:account.tax.template,name:l10n_ch.vat_25_purchase_incl
msgid "TVA 2.5% sur achat B&S (Incl. TR)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_25_purchase
#: model:account.tax.template,name:l10n_ch.vat_25_purchase
msgid "TVA 2.5% sur achat B&S (TR)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_25_invest_incl
#: model:account.tax.template,name:l10n_ch.vat_25_invest_incl
msgid "TVA 2.5% sur invest. et autres ch. (Incl. TR)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_25_invest
#: model:account.tax.template,name:l10n_ch.vat_25_invest
msgid "TVA 2.5% sur invest. et autres ch. (TR)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_purchase_26_incl
#: model:account.tax.template,name:l10n_ch.vat_purchase_26_incl
msgid "TVA 2.6% sur achat B&S (Incl. TR)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_purchase_26
#: model:account.tax.template,name:l10n_ch.vat_purchase_26
msgid "TVA 2.6% sur achat B&S (TR)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_purchase_26_invest_incl
#: model:account.tax.template,name:l10n_ch.vat_purchase_26_invest_incl
msgid "TVA 2.6% sur invest. et autres ch. (Incl. TR)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_purchase_26_invest
#: model:account.tax.template,name:l10n_ch.vat_purchase_26_invest
msgid "TVA 2.6% sur invest. et autres ch. (TR)"
msgstr ""

#. module: l10n_ch
#: model:account.tax.group,name:l10n_ch.tax_group_tva_37
msgid "TVA 3.7%"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_37_purchase_incl
#: model:account.tax.template,name:l10n_ch.vat_37_purchase_incl
msgid "TVA 3.7% sur achat B&S (Incl. TS)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_37_purchase
#: model:account.tax.template,name:l10n_ch.vat_37_purchase
msgid "TVA 3.7% sur achat B&S (TS)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_37_invest_incl
#: model:account.tax.template,name:l10n_ch.vat_37_invest_incl
msgid "TVA 3.7% sur invest. et autres ch. (Incl. TS)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_37_invest
#: model:account.tax.template,name:l10n_ch.vat_37_invest
msgid "TVA 3.7% sur invest. et autres ch. (TS)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_purchase_38_incl
#: model:account.tax.template,name:l10n_ch.vat_purchase_38_incl
msgid "TVA 3.8% sur achat B&S (Incl. TS)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_purchase_38
#: model:account.tax.template,name:l10n_ch.vat_purchase_38
msgid "TVA 3.8% sur achat B&S (TS)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_purchase_38_invest_incl
#: model:account.tax.template,name:l10n_ch.vat_purchase_38_invest_incl
msgid "TVA 3.8% sur invest. et autres ch. (Incl. TS)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_purchase_38_invest
#: model:account.tax.template,name:l10n_ch.vat_purchase_38_invest
msgid "TVA 3.8% sur invest. et autres ch. (TS)"
msgstr ""

#. module: l10n_ch
#: model:account.tax.group,name:l10n_ch.tax_group_tva_77
msgid "TVA 7.7%"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_77_purchase_incl
#: model:account.tax.template,name:l10n_ch.vat_77_purchase_incl
msgid "TVA 7.7% sur achat B&S (Incl. TN)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_77_purchase
#: model:account.tax.template,name:l10n_ch.vat_77_purchase
msgid "TVA 7.7% sur achat B&S (TN)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_77_purchase_reverse
#: model:account.tax.template,name:l10n_ch.vat_77_purchase_reverse
msgid "TVA 7.7% sur achat service a l'etranger (reverse charge)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_77_invest_incl
#: model:account.tax.template,name:l10n_ch.vat_77_invest_incl
msgid "TVA 7.7% sur invest. et autres ch. (Incl. TN)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_77_invest
#: model:account.tax.template,name:l10n_ch.vat_77_invest
msgid "TVA 7.7% sur invest. et autres ch. (TN)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_purchase_81_incl
#: model:account.tax.template,name:l10n_ch.vat_purchase_81_incl
msgid "TVA 8.1% sur achat B&S (Incl. TN)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_purchase_81
#: model:account.tax.template,name:l10n_ch.vat_purchase_81
msgid "TVA 8.1% sur achat B&S (TN)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_purchase_81_reverse
#: model:account.tax.template,name:l10n_ch.vat_purchase_81_reverse
msgid "TVA 8.1% sur achat service a l'etranger (reverse charge)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_purchase_81_invest_incl
#: model:account.tax.template,name:l10n_ch.vat_purchase_81_invest_incl
msgid "TVA 8.1% sur invest. et autres ch. (Incl. TN)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_purchase_81_invest
#: model:account.tax.template,name:l10n_ch.vat_purchase_81_invest
msgid "TVA 8.1% sur invest. et autres ch. (TN)"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_2200
#: model:account.account.template,name:l10n_ch.ch_coa_2200
msgid "TVA due"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_XO
#: model:account.tax.template,name:l10n_ch.vat_XO
msgid "TVA due a 0% (Exportations)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_25
#: model:account.tax.template,name:l10n_ch.vat_25
msgid "TVA due a 2.5% (TR)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_sale_26
#: model:account.tax.template,name:l10n_ch.vat_sale_26
msgid "TVA due a 2.6% (TR)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_37
#: model:account.tax.template,name:l10n_ch.vat_37
msgid "TVA due a 3.7% (TS)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_sale_38
#: model:account.tax.template,name:l10n_ch.vat_sale_38
msgid "TVA due a 3.8% (TS)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_77
#: model:account.tax.template,name:l10n_ch.vat_77
msgid "TVA due a 7.7% (TN)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_77_purchase_return
#: model:account.tax.template,name:l10n_ch.vat_77_purchase_return
msgid "TVA due a 7.7% (TN) (return)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_sale_81
#: model:account.tax.template,name:l10n_ch.vat_sale_81
msgid "TVA due a 8.1% (TN)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_purchase_81_return
#: model:account.tax.template,name:l10n_ch.vat_purchase_81_return
msgid "TVA due a 8.1% (TN) (return)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_25_incl
#: model:account.tax.template,name:l10n_ch.vat_25_incl
msgid "TVA due à 2.5% (Incl. TR)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_sale_26_incl
#: model:account.tax.template,name:l10n_ch.vat_sale_26_incl
msgid "TVA due à 2.6% (Incl. TR)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_37_incl
#: model:account.tax.template,name:l10n_ch.vat_37_incl
msgid "TVA due à 3.7% (Incl. TS)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_sale_38_incl
#: model:account.tax.template,name:l10n_ch.vat_sale_38_incl
msgid "TVA due à 3.8% (Incl. TS)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_77_incl
#: model:account.tax.template,name:l10n_ch.vat_77_incl
msgid "TVA due à 7.7% (Incl. TN)"
msgstr ""

#. module: l10n_ch
#: model:account.tax,name:l10n_ch.1_vat_sale_81_incl
#: model:account.tax.template,name:l10n_ch.vat_sale_81_incl
msgid "TVA due à 8.1% (Incl. TN)"
msgstr ""

#. module: l10n_ch
#: code:addons/l10n_ch/models/res_bank.py:0
#, python-format
msgid ""
"The ISR subcription {} for {} number is not valid.\n"
"It must starts with {} and we a valid postal number format. eg. {}"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,help:l10n_ch.field_account_bank_statement_line__l10n_ch_currency_name
#: model:ir.model.fields,help:l10n_ch.field_account_move__l10n_ch_currency_name
#: model:ir.model.fields,help:l10n_ch.field_account_payment__l10n_ch_currency_name
msgid "The name of this invoice's currency"
msgstr ""

#. module: l10n_ch
#: code:addons/l10n_ch/models/res_bank.py:0
#, python-format
msgid ""
"The partner must have a complete postal address (street, zip, city and "
"country)."
msgstr ""

#. module: l10n_ch
#: code:addons/l10n_ch/models/res_bank.py:0
#, python-format
msgid ""
"The partner set on the bank account meant to receive the payment (%s) must "
"have a complete postal address (street, zip, city and country)."
msgstr ""

#. module: l10n_ch
#: code:addons/l10n_ch/models/res_bank.py:0
#, python-format
msgid ""
"The postal number {} is not valid.\n"
"It must be a valid postal number format. eg. 10-8060-7"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,help:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_number
#: model:ir.model.fields,help:l10n_ch.field_account_move__l10n_ch_isr_number
#: model:ir.model.fields,help:l10n_ch.field_account_payment__l10n_ch_isr_number
msgid "The reference number associated with this invoice"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,help:l10n_ch.field_account_setup_bank_manual_config__l10n_ch_isr_subscription_chf
#: model:ir.model.fields,help:l10n_ch.field_res_partner_bank__l10n_ch_isr_subscription_chf
msgid ""
"The subscription number provided by the bank or Postfinance to identify the "
"bank, used to generate ISR in CHF. eg. 01-162-8"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,help:l10n_ch.field_account_setup_bank_manual_config__l10n_ch_isr_subscription_eur
#: model:ir.model.fields,help:l10n_ch.field_res_partner_bank__l10n_ch_isr_subscription_eur
msgid ""
"The subscription number provided by the bank or Postfinance to identify the "
"bank, used to generate ISR in EUR. eg. 03-162-5"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,help:l10n_ch.field_account_bank_statement_import_journal_creation__l10n_ch_postal
#: model:ir.model.fields,help:l10n_ch.field_account_journal__l10n_ch_postal
#: model:ir.model.fields,help:l10n_ch.field_account_setup_bank_manual_config__l10n_ch_postal
#: model:ir.model.fields,help:l10n_ch.field_res_partner_bank__l10n_ch_postal
msgid ""
"This field is used for the Swiss postal account number on a vendor account "
"and for the client number on your own account. The client number is mostly 6"
" numbers without -, while the postal account number can be e.g. 01-162-8"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1060
#: model:account.account.template,name:l10n_ch.ch_coa_1060
msgid "Titres"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1400
#: model:account.account.template,name:l10n_ch.ch_coa_1400
msgid "Titres à long terme"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1280
#: model:account.account.template,name:l10n_ch.ch_coa_1280
msgid "Travaux en cours"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,help:l10n_ch.field_account_bank_statement_line__l10n_ch_isr_needs_fixing
#: model:ir.model.fields,help:l10n_ch.field_account_move__l10n_ch_isr_needs_fixing
#: model:ir.model.fields,help:l10n_ch.field_account_payment__l10n_ch_isr_needs_fixing
msgid ""
"Used to show a warning banner when the vendor bill needs a correct ISR "
"payment reference. "
msgstr ""

#. module: l10n_ch
#: model:account.tax.group,name:l10n_ch.tax_group_vat_26
msgid "VAT 2.6%"
msgstr ""

#. module: l10n_ch
#: model:account.tax.group,name:l10n_ch.tax_group_vat_38
msgid "VAT 3.8%"
msgstr ""

#. module: l10n_ch
#: model:account.tax.group,name:l10n_ch.tax_group_vat_81
msgid "VAT 8.1%"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3940
#: model:account.account.template,name:l10n_ch.ch_coa_3940
msgid "Variation de la valeur des prestations non facturées"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1287
#: model:account.account.template,name:l10n_ch.ch_coa_1287
msgid "Variation de la valeur des travaux en cours"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1277
#: model:account.account.template,name:l10n_ch.ch_coa_1277
msgid "Variation de stock produits semi-ouvrés"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1267
#: model:account.account.template,name:l10n_ch.ch_coa_1267
msgid "Variation de stocks de produits finis"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1207
#: model:account.account,name:l10n_ch.1_ch_coa_4800
#: model:account.account.template,name:l10n_ch.ch_coa_1207
#: model:account.account.template,name:l10n_ch.ch_coa_4800
msgid "Variation des stocks de marchandises"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4801
#: model:account.account.template,name:l10n_ch.ch_coa_4801
msgid "Variation des stocks de matières premières"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3901
#: model:account.account.template,name:l10n_ch.ch_coa_3901
msgid "Variation des stocks de produits finis"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3900
#: model:account.account.template,name:l10n_ch.ch_coa_3900
msgid "Variation des stocks de produits semi-finis"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1217
#: model:account.account.template,name:l10n_ch.ch_coa_1217
msgid "Variation des stocks des matières premières"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_4008
#: model:account.account,name:l10n_ch.1_ch_coa_4080
#: model:account.account.template,name:l10n_ch.ch_coa_4008
#: model:account.account.template,name:l10n_ch.ch_coa_4080
msgid "Variations de stocks"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3200
#: model:account.account.template,name:l10n_ch.ch_coa_3200
msgid "Ventes de marchandises"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3400
#: model:account.account.template,name:l10n_ch.ch_coa_3400
msgid "Ventes de prestations"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_3000
#: model:account.account.template,name:l10n_ch.ch_coa_3000
msgid "Ventes de produits fabriqués"
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,field_description:l10n_ch.field_res_config_settings__l10n_ch_isr_scan_line_top
msgid "Vertical offset"
msgstr ""

#. module: l10n_ch
#: model:account.account,name:l10n_ch.1_ch_coa_1530
#: model:account.account.template,name:l10n_ch.ch_coa_1530
msgid "Véhicules"
msgstr ""

#. module: l10n_ch
#: code:addons/l10n_ch/models/res_bank.py:0
#, python-format
msgid ""
"When using a QR-IBAN as the destination account of a QR-code, the payment "
"reference must be a QR-reference."
msgstr ""

#. module: l10n_ch
#: model:ir.model.fields,help:l10n_ch.field_account_bank_statement_import_journal_creation__invoice_reference_model
#: model:ir.model.fields,help:l10n_ch.field_account_journal__invoice_reference_model
msgid ""
"You can choose different models for each type of reference. The default one "
"is the Odoo reference."
msgstr ""

#. module: l10n_ch
#: code:addons/l10n_ch/models/account_invoice.py:0
#, python-format
msgid ""
"You cannot generate an ISR yet.\n"
"\n"
"                                   For this, you need to :\n"
"\n"
"                                   - set a valid postal account number (or an IBAN referencing one) for your company\n"
"\n"
"                                   - define its bank\n"
"\n"
"                                   - associate this bank with a postal reference for the currency used in this invoice\n"
"\n"
"                                   - fill the 'bank account' field of the invoice with the postal to be used to receive the related payment. A default account will be automatically set for all invoices created after you defined a postal account for your company."
msgstr ""

#. module: l10n_ch
#: model_terms:ir.ui.view,arch_db:l10n_ch.l10n_ch_swissqr_template
msgid "padding-top:6.2mm; padding-left:8.2mm; padding-right:8.2mm;"
msgstr ""

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
    new_template_to_tax = update_taxes_from_templates(cr, 'l10n_ch.l10nch_chart_template')
    if new_template_to_tax:
        _, new_tax_ids = zip(*new_template_to_tax)
        env = api.Environment(cr, SUPERUSER_ID, {})
        env['account.tax'].browse(new_tax_ids).active = True

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
        if self.company_id.country_id.code in ('CH', 'LI') and _is_l10n_ch_postal(self.account_number):
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
            super(AccountBankStatementLine, self)._find_or_create_bank_account()
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
        self.ensure_one()
        return self.partner_bank_id.l10n_ch_postal or ''

    @api.depends('name', 'partner_bank_id.l10n_ch_postal')
    def _compute_l10n_ch_isr_number(self):
        """Generates the ISR or QRR reference

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
        for record in self:
            has_qriban = record.partner_bank_id and record.partner_bank_id._is_qr_iban() or False
            isr_subscription = record.l10n_ch_isr_subscription
            if (has_qriban or isr_subscription) and record.name:
                id_number = record._get_isrb_id_number()
                if id_number:
                    id_number = id_number.zfill(l10n_ch_ISR_ID_NUM_LENGTH)
                invoice_ref = re.sub('[^\d]', '', record.name)
                # keep only the last digits if it exceed boundaries
                full_len = len(id_number) + len(invoice_ref)
                ref_payload_len = l10n_ch_ISR_NUMBER_LENGTH - 1
                extra = full_len - ref_payload_len
                if extra > 0:
                    invoice_ref = invoice_ref[extra:]
                internal_ref = invoice_ref.zfill(ref_payload_len - len(id_number))
                record.l10n_ch_isr_number = mod10r(id_number + internal_ref)
            else:
                record.l10n_ch_isr_number = False

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
        """ Compute the optical line to print on the bottom of the ISR.

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
            if inv.move_type == 'in_invoice' and inv.company_id.country_id.code in ('CH', 'LI'):
                partner_bank = inv.partner_bank_id
                if partner_bank:
                    needs_isr_ref = partner_bank._is_qr_iban() or partner_bank._is_isr_issuer()
                else:
                    needs_isr_ref = False
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
        return super(AccountMove, self.with_context(mail_post_autofollow=True)).message_post(**kwargs)

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

    # creation of bank journals by giving the account number, allow craetion of the
    l10n_ch_postal = fields.Char('Client Number', related='bank_account_id.l10n_ch_postal', readonly=False)
    invoice_reference_model = fields.Selection(selection_add=[
        ('ch', 'Switzerland')
    ], ondelete={'ch': lambda recs: recs.write({'invoice_reference_model': 'odoo'})})

    @api.model
    def create(self, vals):
        rslt = super(AccountJournal, self).create(vals)

        # The call to super() creates the related bank_account_id field
        if 'l10n_ch_postal' in vals:
            rslt.l10n_ch_postal = vals['l10n_ch_postal']
        return rslt

    def write(self, vals):
        rslt = super(AccountJournal, self).write(vals)

        # The call to super() creates the related bank_account_id field if necessary
        if 'l10n_ch_postal' in vals:
            for record in self.filtered('bank_account_id'):
                record.bank_account_id.l10n_ch_postal = vals['l10n_ch_postal']
        return rslt

    @api.onchange('bank_acc_number')
    def _onchange_set_l10n_ch_postal(self):
        try:
            validate_iban(self.bank_acc_number)
            is_iban = True
        except ValidationError:
            is_iban = False

        if is_iban:
            self.l10n_ch_postal = self.env['res.partner.bank']._retrieve_l10n_ch_postal(sanitize_account_number(self.bank_acc_number))
        else:
            self.l10n_ch_postal = self.bank_acc_number

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

    if re.match('\d+$', account_ref or ''):
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


class ResPartnerBank(models.Model):
    _inherit = 'res.partner.bank'

    l10n_ch_postal = fields.Char(
        string="Swiss Postal Account",
        readonly=False, store=True,
        compute='_compute_l10n_ch_postal',
        help="This field is used for the Swiss postal account number on a vendor account and for the client number on "
             "your own account. The client number is mostly 6 numbers without -, while the postal account number can "
             "be e.g. 01-162-8")

    # fields to configure ISR payment slip generation
    l10n_ch_isr_subscription_chf = fields.Char(string='CHF ISR Subscription Number', help='The subscription number provided by the bank or Postfinance to identify the bank, used to generate ISR in CHF. eg. 01-162-8')
    l10n_ch_isr_subscription_eur = fields.Char(string='EUR ISR Subscription Number', help='The subscription number provided by the bank or Postfinance to identify the bank, used to generate ISR in EUR. eg. 03-162-5')
    l10n_ch_show_subscription = fields.Boolean(compute='_compute_l10n_ch_show_subscription', default=lambda self: self.env.company.country_id.code == 'CH')

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
                bank.l10n_ch_show_subscription = bank.company_id.country_id.code in ('CH', 'LI')
            else:
                bank.l10n_ch_show_subscription = self.env.company.country_id.code in ('CH', 'LI')

    @api.depends('acc_number', 'acc_type')
    def _compute_sanitized_acc_number(self):
        #Only remove spaces in case it is not postal
        postal_banks = self.filtered(lambda b: b.acc_type == "postal")
        for bank in postal_banks:
            bank.sanitized_acc_number = bank.acc_number
        super(ResPartnerBank, self - postal_banks)._compute_sanitized_acc_number()

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
        if self._is_qr_iban():
            # _check_for_qr_code_errors ensures we can't have a QR-IBAN without a QR-reference here
            reference_type = 'QRR'
            reference = structured_communication
        elif self._is_iso11649_reference(structured_communication):
            reference_type = 'SCOR'
            reference = structured_communication.replace(' ', '')

        currency = currency or self.currency_id or self.company_id.currency_id

        return [
            'SPC',                                                # QR Type
            '0200',                                               # Version
            '1',                                                  # Coding Type
            self.sanitized_acc_number,                            # IBAN
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

    def _check_qr_iban_range(self, iban):
        if not iban or len(iban) < 9:
            return False
        iid_start_index = 4
        iid_end_index = 8
        iid = iban[iid_start_index : iid_end_index+1]
        return re.match('\d+', iid) \
               and 30000 <= int(iid) <= 31999 # Those values for iid are reserved for QR-IBANs only

    def _is_qr_iban(self):
        """ Tells whether or not this bank account has a QR-IBAN account number.
        QR-IBANs are specific identifiers used in Switzerland as references in
        QR-codes. They are formed like regular IBANs, but are actually something
        different.
        """
        self.ensure_one()

        return self.sanitized_acc_number.startswith(('CH', 'LI'))\
               and self.acc_type == 'iban'\
               and self._check_qr_iban_range(self.sanitized_acc_number)

    @api.model
    def _is_qr_reference(self, reference):
        """ Checks whether the given reference is a QR-reference, i.e. it is
        made of 27 digits, the 27th being a mod10r check on the 26 previous ones.
        """
        return reference \
               and len(reference) == 27 \
               and re.match('\d+$', reference) \
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

            if self._is_qr_iban() and not self._is_qr_reference(structured_communication):
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

        <template id="assets_common" name="l10n_ch_isr_report" inherit_id="web.report_assets_common">
            <xpath expr="." position="inside">
                <link rel="stylesheet" type="text/scss" href="/l10n_ch/static/src/scss/report_isr.scss"/>
            </xpath>
        </template>

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

        <template id="assets_common_qr" inherit_id="web.report_assets_common">
            <xpath expr="." position="inside">
                <link type="text/scss" rel="stylesheet" href="/l10n_ch/static/src/scss/report_swissqr.scss"/>
            </xpath>
        </template>

        <template id="l10n_ch_swissqr_template">
            <t t-set="o" t-value="o.with_context(lang=lang)"/>
            <t t-call="web.external_layout">
                <!-- TODO master: remove this -->
                <script t-if="0">document.body.className += " l10n_ch_qr";</script>
                <!-- add default margin for header (matching A4 European margin) -->
                <t t-set="report_header_style">padding-top:6.2mm; padding-left:8.2mm; padding-right:8.2mm;</t>

                <t t-set="formated_amount" t-value="'{:,.2f}'.format(o.amount_residual).replace(',','\xa0')"/>

                <t t-set="is_qrr" t-value="o.partner_bank_id._is_qr_iban()"/>
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
                                <span t-field="o.partner_bank_id.acc_number"/><br/>
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
                                <span t-field="o.partner_bank_id.acc_number"/><br/>
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

        <record id="isr_bank_journal_form" model="ir.ui.view">
            <field name="name">l10n_ch.bank.journal.form</field>
            <field name="model">account.journal</field>
            <field name="inherit_id" ref="account.view_account_journal_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='bank_account_id']" position="after">
                    <field name="l10n_ch_postal" attrs="{'invisible': [('bank_account_id', '=', False)]}"/>
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
                </field>
            </field>
        </record>
    </data>
</odoo>

```

