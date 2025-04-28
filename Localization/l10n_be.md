# Odoo Module: l10n_be

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, SUPERUSER_ID

from . import models

def load_translations(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref('l10n_be.l10nbe_chart_template').process_coa_translations()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Belgium - Accounting',
    'version': '2.0',
    'category': 'Localization',
    'description': """
This is the base module to manage the accounting chart for Belgium in Odoo.
==============================================================================

After installing this module, the Configuration wizard for accounting is launched.
    * We have the account templates which can be helpful to generate Charts of Accounts.
    * On that particular wizard, you will be asked to pass the name of the company,
      the chart template to follow, the no. of digits to generate, the code for your
      account and bank account, currency to create journals.

Thus, the pure copy of Chart Template is generated.

Wizards provided by this module:
--------------------------------
    * Partner VAT Intra: Enlist the partners with their related VAT and invoiced
      amounts. Prepares an XML file format.

        **Path to access :** Invoicing/Reporting/Legal Reports/Belgium Statements/Partner VAT Intra
    * Periodical VAT Declaration: Prepares an XML file for Vat Declaration of
      the Main company of the User currently Logged in.

        **Path to access :** Invoicing/Reporting/Legal Reports/Belgium Statements/Periodical VAT Declaration
    * Annual Listing Of VAT-Subjected Customers: Prepares an XML file for Vat
      Declaration of the Main company of the User currently Logged in Based on
      Fiscal year.

        **Path to access :** Invoicing/Reporting/Legal Reports/Belgium Statements/Annual Listing Of VAT-Subjected Customers

    """,
    'author': 'Noviat, Odoo SA',
    'depends': [
        'account',
        'base_iban',
        'base_vat',
        'l10n_multilang',
    ],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/account_pcmn_belgium_data.xml',
        'data/account_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/l10n_be_sequence_data.xml',
        'data/fiscal_templates_data.xml',
        'data/account_fiscal_position_tax_template_data.xml',
        'data/account_reconcile_model_template.xml',
        'data/account_chart_template_configure_data.xml',
        'data/menuitem_data.xml',
    ],
    'demo': [
        'demo/l10n_be_demo.xml',
    ],
    'post_init_hook': 'load_translations',
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","tag_ids/id","reconcile"
"a1000","Capital","1000","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a11","Primes d'émission","1100","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a130","Réserve légale","130","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a132","Réserves immunisées","132","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a140","Bénéfice reporté","140","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a141","Perte reportée","141","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a150","Subsides obtenus","150","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a151","Montants transférés aux résultats","151","account.data_account_type_equity","l10n_be.l10nbe_chart_template","","False"
"a1600","Provisions pour risques et charges","1600","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1720","Dettes de location-financement de biens immobiliers","1720","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1721","Dettes de location-financement de biens mobiliers","1721","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a1722","Dettes sur droits réels sur immeubles","1722","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a17300","Dettes en compte \ Banque A","17300","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a174","Autres emprunts","174","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a175110","Effets à payer \ Fournisseurs belges","175110","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a175111","Effets à payer \ Fournisseurs C.E.E.","175111","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a175112","Effets à payer \ Fournisseurs importation","175112","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a176","Acomptes reçus sur commandes","176","account.data_account_type_non_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a2000","Frais de constitution et d'augmentation de capital","2000","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2009","Amortissements sur frais de constitution et d'augmentation de capital","2009","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2100","Frais de recherche et de mise au point","2100","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2108","Plus-values actées sur frais de recherche et de mise au point","2108","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2109","Amortissements sur frais de recherche et de mise au point","2109","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2110","Concessions, brevets, licences, savoir-faire, marques, etc...","2110","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2118","Plus-values actées sur concessions, brevets, etc...","2118","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2119","Amortissements sur concessions, brevets, etc...","2119","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a24000","Mobilier des bâtiments industriels","24000","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a24001","Mobilier des bâtiments administratifs et commerciaux","24001","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a24010","Matériel de bureau de bâtiments industriels","24010","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a24011","Matériel de bureau de bâtiments administratifs et commerciaux","24011","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a24090","Amortissements sur mobilier","24090","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a24100","Voitures","24100","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a24105","Camions","24105","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a24190","Amortissements sur matériel automobile","24190","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2500","Terrains","2500","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2501","Constructions","2501","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2520","Mobilier","2520","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2521","Matériel roulant","2521","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2528","Plus-values actées sur mobilier et matériel roulant en leasing","2528","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2529","Amortissements sur mobilier et matériel roulant en leasing","2529","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a260","Frais d'aménagements de locaux pris en location","260","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2690","Amortissements sur frais d'aménagement des locaux pris en location","2690","account.data_account_type_non_current_assets","l10n_be.l10nbe_chart_template","","False"
"a2800","Immobilisations Financières","2800","account.data_account_type_fixed_assets","l10n_be.l10nbe_chart_template","","False"
"a29000","Créances commerciales","29000","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a300","Matières premières","300","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a3100","Approvisionements et fournitures","310","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a3200","En cours de fabrication","320","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a329","Réductions de valeur actées","329","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a3300","Produits finis","330","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a3400","Marchandises","340","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a3500","Immeubles destinés à la vente","350","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a360","Acomptes versés sur achats pour stocks","360","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a370","Commandes en cours d'exécution","370","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4000","Clients","4000","account.data_account_type_receivable","l10n_be.l10nbe_chart_template","","True"
"a4001","Clients (PoS)","4001","account.data_account_type_receivable","l10n_be.l10nbe_chart_template","","True"
"a404","Produits à recevoir","404","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","True"
"a406","Acomptes versés","406","account.data_account_type_receivable","l10n_be.l10nbe_chart_template","","True"
"a407","Créances douteuses","407","account.data_account_type_receivable","l10n_be.l10nbe_chart_template","","True"
"a411059","T.V.A Déductible","4110","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4112","Compte courant administration T.V.A.","4112","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4118","Taxe d'égalisation due","4118","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4120","Impôts belges sur le résultat","4120","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4125","Autres impôts belges","4125","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4128","Impôts étrangers","4128","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a414","Produits à recevoir","414","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4160","Associés","4160","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4161","Avances et prêts au personnel","4161","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4162","Compte courant des associés en S.P.R.L.","4162","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4163","Compte courant des administrateurs et gérants","4163","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4164","Créances sur sociétés apparentées","4164","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4166","Emballages et matériel à rendre","4166","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a41670","Subsides à recevoir","41670","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a41671","Autres créances","41671","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4168","Rabais, ristournes, remises à obtenir et autres avoirs non encore reçus","4168","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a417","Créances douteuses","417","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a418","Cautionnements versés en numéraires","418","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a419","Réductions de valeur actées","419","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a4220","Financement de biens immobiliers","4220","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4221","Financement de biens mobiliers","4221","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4230","Etablissements de crédit","4230","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a424","Autres emprunts","424","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a430","Etablissements de crédit. Emprunts en compte à terme fixe","430","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a440","Fournisseurs","440","account.data_account_type_payable","l10n_be.l10nbe_chart_template","","True"
"a444","Factures à recevoir","444","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","True"
"a448","Compensations fournisseurs","448","account.data_account_type_payable","l10n_be.l10nbe_chart_template","","True"
"a4500","Dettes fiscales estimées","4500","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a451054","T.V.A. à payer","451000","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a451055","T.V.A. à payer - Intra-communautaire","451055","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a451056","T.V.A. à payer - Cocontractant","451056","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a451057","T.V.A. à payer - Import","451057","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4512","Compte courant administration T.V.A.","4512","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4530","Précompte professionnel retenu sur rémunérations","4530","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4533","Précompte mobilier retenu sur intérêts payés","4533","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4538","Autres précomptes retenus","4538","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4540","ONSS. Arriérés","4540","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4541","ONSS. 1er trimestre","4541","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4542","ONSS. 2ème trimestre","4542","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4543","ONSS. 3ème trimestre","4543","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4544","ONSS. 4ème trimestre","4544","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4550","Administrateurs, gérants et commissaires","4550","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4551","Direction","4551","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4552","Employés","4552","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4553","Ouvriers","4553","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4560","Direction","4560","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4561","Employés","4561","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4562","Ouvriers","4562","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4590","Provision pour gratifications de fin d'année","4590","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a45930","Assurance loi","45930","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a45931","Assurance salaire garanti ","45931","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a45932","Assurance groupe ","45932","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a45933","Assurances individuelles","45933","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4594","Caisse d'assurances sociales pour travailleurs indépendants","4594","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4597","Dettes et provisions sociales diverses","4597","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a460","Acomptes à recevoir","460","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a461","Acomptes reçus","461","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a490","Charges à reporter","490","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a49100","Ristournes, rabais à obtenir","49100","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a49101","Commissions à obtenir","49101","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a49102","Autres produits d'exploitation","49102","account.data_account_type_current_assets","l10n_be.l10nbe_chart_template","","False"
"a492","Charges à imputer","492","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4930","Produits d'exploitation à reporter","4930","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a4990","Compte d'attente","4990","account.data_account_type_current_liabilities","l10n_be.l10nbe_chart_template","","False"
"a540","Chèques à encaisser","540","account.data_account_type_liquidity","l10n_be.l10nbe_chart_template","","False"
"a600","Achats de matières premières","600","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a601","Achats de fournitures","601","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a602","Achats de services, travaux et études","602","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a603","Sous-traitances générales","603","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a604","Achats de marchandises","604","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6100","Loyers divers","6100","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6101","Charges locatives","6101","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a611","Entretien et réparation","611","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a612","Fournitures","612","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a613","Rétributions de tiers","613","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a614","Annonces, publicité, propagande et documentation","614","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6200","Rémunérations: Administrateurs ou gérants","6200","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6201","Rémunérations: Personnel de direction","6201","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6202","Rémunérations: Employés","6202","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a62021","Précompte professionnel","62021","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a62022","ATN - Employés","62022","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6203","Rémunérations: Ouvriers","6203","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6204","Rémunérations: Autres membres du personnel","6204","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a621","Cotisations patronales d'assurances sociales","621","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6231","Frais de déplacement","6231","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6301","Dotations aux amortissements sur immobilisations incorporelles","6301","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6302","Dotations aux amortissements sur immobilisations corporelles","6302","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6310","Réductions de valeur sur stocks","631","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a6320","Réductions de valeur sur commandes en cours d'exécution","632","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a64000","Taxes sur autos et camions","64000","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a64012","T.V.A. non déductible","64012","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a653","Escomptes clients","653","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a654","Différences de change","654","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a656","Frais de banques, de chèques postaux","656","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a6600","Charges exceptionelles","6600","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_investing","False"
"a6700","Impôts et précomptes dus ou versés","6700","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a690","Perte reportée de l'exercice précédent","690","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a693","Bénéfice à reporter","693","account.data_account_type_expenses","l10n_be.l10nbe_chart_template","","False"
"a7000","Ventes en Belgique (marchandises)","7000","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7001","Ventes dans les pays membres de la C.E.E. (marchandises)","7001","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7002","Ventes à l'exportation (marchandises)","7002","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7010","Ventes en Belgique (produits finis)","7010","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7011","Ventes dans les pays membres de la C.E.E. (produits finis)","7011","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7012","Ventes à l'exportation (produits finis)","7012","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7050","Prestations de services en Belgique","7050","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7051","Prestations de services dans les pays membres de la C.E.E.","7051","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7052","Prestations de services en vue de l'exportation","7052","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a706","Pénalités et dédits obtenus par l'entreprise","706","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a740","Autres produits d'exploitation","740","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7492","Autres produits d'exploitation","7492","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7500","Revenus des actions","7500","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a7501","Revenus des obligations","7501","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a7502","Revenus des créances à plus d'un an","7502","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a751","Produits des actifs circulants","751","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a752","Plus-values sur réalisations d'actifs circulants","752","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a753","Subsides en capital et en intérêts","753","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a754","Différences de change","754","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a755","Ecarts de conversion des devises","755","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a756","Produits des autres créances","756","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a757","Escomptes obtenus","757","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_financing","False"
"a7600","Produits exceptionnels","7600","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_investing","False"
"a7710","Régularisations d'impôts dus ou versés","7710","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a7711","Régularisations d'impôts estimés","7711","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a780","Prélèvements sur les impôts différés","780","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a789","Prélèvements sur les réserves immunisées","789","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","account.account_tag_operating","False"
"a790","Bénéfice reporté de l'exercice précédent","790","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a791","Prélèvement sur le capital et les primes d'émission","791","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"
"a793","Perte à reporter","793","account.data_account_type_revenue","l10n_be.l10nbe_chart_template","","False"

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_be.l10nbe_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- Chart template -->
        <record id="l10nbe_chart_template" model="account.chart.template">
            <field name="name">Belgian PCMN</field>
            <field name="bank_account_code_prefix">550</field>
            <field name="cash_account_code_prefix">570</field>
            <field name="transfer_account_code_prefix">580</field>
            <field name="currency_id" ref="base.EUR"/>
            <field name="spoken_languages" eval="'nl_BE;nl_NL'"/>
        </record>
</odoo>

```

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>

<odoo>
    <data noupdate="1">

        <!-- Account Tax Group -->
        <record id="tax_group_tva_21" model="account.tax.group">
            <field name="name">TVA 21%</field>
        </record>

        <record id="tax_group_tva_12" model="account.tax.group">
            <field name="name">TVA 12%</field>
        </record>

        <record id="tax_group_tva_6" model="account.tax.group">
            <field name="name">TVA 6%</field>
        </record>

        <record id="tax_group_tva_0" model="account.tax.group">
            <field name="name">TVA 0%</field>
        </record>

    </data>
</odoo>

```

## File: data\account_fiscal_position_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- account.fiscal.position.tax.template -->
        <record id="afpttn_intracom_1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-00-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-S"/>
        </record>
        <record id="afpttn_intracom_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-00-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-L"/>
        </record>
        <record id="afpttn_intracom_3" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-06-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-S"/>
        </record>
        <record id="afpttn_intracom_4" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-06-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-L"/>
        </record>
        <record id="afpttn_intracom_5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-12-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-S"/>
        </record>
        <record id="afpttn_intracom_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-12-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-L"/>
        </record>
        <record id="afpttn_intracom_7" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-21-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-S"/>
        </record>
        <record id="afpttn_intracom_8" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-21-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-EU-L"/>
        </record>
        <record id="afpttn_intracom_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-00"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-00-EU"/>
        </record>
        <record id="afpttn_intracom_10" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-06"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-06-EU"/>
        </record>
        <record id="afpttn_intracom_11" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-12"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-12-EU"/>
        </record>
        <record id="afpttn_intracom_12" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-21"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-21-EU"/>
        </record>
        <record id="afpttn_intracom_13" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-00-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-00-EU-S"/>
        </record>
        <record id="afpttn_intracom_14" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-00-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-00-EU-G"/>
        </record>
        <record id="afpttn_intracom_15" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-06-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-06-EU-S"/>
        </record>
        <record id="afpttn_intracom_16" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-06-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-06-EU-G"/>
        </record>
        <record id="afpttn_intracom_17" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-12-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-12-EU-S"/>
        </record>
        <record id="afpttn_intracom_18" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-12-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-12-EU-G"/>
        </record>
        <record id="afpttn_intracom_19" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-21-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-21-EU-S"/>
        </record>
        <record id="afpttn_intracom_20" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-21-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-21-EU-G"/>
        </record>
        <record id="afpttn_intracom_21" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-00"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-00-EU"/>
        </record>
        <record id="afpttn_intracom_22" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-06"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-06-EU"/>
        </record>
        <record id="afpttn_intracom_23" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-12"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-12-EU"/>
        </record>
        <record id="afpttn_intracom_24" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-21"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-21-EU"/>
        </record>
        <record id="afpttn_extracom_1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-00-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-00-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_3" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-06-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_4" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-06-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-12-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-12-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_7" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-21-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_8" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-21-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-ROW"/>
        </record>
        <record id="afpttn_extracom_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-06"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-06-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_10" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-12"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-12-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_11" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-21"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-21-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_12" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-06-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-06-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_13" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-06-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-06-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_14" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-12-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-12-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_15" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-12-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-12-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_16" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-21-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-21-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_17" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-21-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-21-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_18" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-06"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-06-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_19" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-12"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-12-ROW-CC"/>
        </record>
        <record id="afpttn_extracom_20" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-21"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-21-ROW-CC"/>
        </record>
        <record id="afpttn_cocontractant_1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-00-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-00-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_3" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-06-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_4" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-06-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-12-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-12-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_7" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-21-S"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_8" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-OUT-21-L"/>
            <field name="tax_dest_id" ref="attn_VAT-OUT-00-CC"/>
        </record>
        <record id="afpttn_cocontractant_9" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-06"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-06-CC"/>
        </record>
        <record id="afpttn_cocontractant_10" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-12"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-12-CC"/>
        </record>
        <record id="afpttn_cocontractant_11" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V81-21"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V81-21-CC"/>
        </record>
        <record id="afpttn_cocontractant_12" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-06-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-06-CC"/>
        </record>
        <record id="afpttn_cocontractant_13" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-06-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-06-CC"/>
        </record>
        <record id="afpttn_cocontractant_14" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-12-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-12-CC"/>
        </record>
        <record id="afpttn_cocontractant_15" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-12-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-12-CC"/>
        </record>
        <record id="afpttn_cocontractant_16" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-21-S"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-21-CC"/>
        </record>
        <record id="afpttn_cocontractant_17" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V82-21-G"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V82-21-CC"/>
        </record>
        <record id="afpttn_cocontractant_18" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-06"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-06-CC"/>
        </record>
        <record id="afpttn_cocontractant_19" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-12"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-12-CC"/>
        </record>
        <record id="afpttn_cocontractant_20" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_4"/>
            <field name="tax_src_id" ref="attn_VAT-IN-V83-21"/>
            <field name="tax_dest_id" ref="attn_VAT-IN-V83-21-CC"/>
        </record>
</odoo>

```

## File: data\account_pcmn_belgium_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="l10nbe_chart_template" model="account.chart.template">
            <field name="name">Belgian PCMN</field>
            <field name="code_digits">6</field>
            <field name="property_account_receivable_id" ref="a4000"/>
            <field name="property_account_payable_id" ref="a440"/>
            <field name="property_account_expense_categ_id" ref="a600"/>
            <field name="property_account_income_categ_id" ref="a7010"/>
            <field name="expense_currency_exchange_account_id" ref="a654"/>
            <field name="income_currency_exchange_account_id" ref="a754"/>
            <field name="property_tax_payable_account_id" ref="a4512"/>
            <field name="property_tax_receivable_account_id" ref="a4112"/>
            <field name="default_pos_receivable_account_id" ref="a4001" />
        </record>
</odoo>

```

## File: data\account_reconcile_model_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="escompte_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="l10nbe_chart_template"/>
        <field name="name">Escompte</field>
        <field name="account_id" ref="a653"/>
        <field name="amount_type">percentage</field>
        <field name="amount">100</field>
        <field name="label">Escompte accordé</field>
    </record>
    <record id="frais_bancaires_htva_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="l10nbe_chart_template"/>
        <field name="name">Frais bancaires HTVA</field>
        <field name="account_id" ref="a656" />
        <field name="amount_type">percentage</field>
        <field name="amount">100</field>
        <field name="label">Frais bancaires HTVA</field>
    </record>
    <record id="frais_bancaires_tva21_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="l10nbe_chart_template"/>
        <field name="name">Frais bancaires TVA21</field>
        <field name="account_id" ref="a656"/>
        <field name="amount_type">percentage</field>
        <field name="tax_ids" eval="[(6, 0, [ref('l10n_be.attn_TVA-21-inclus-dans-prix')])]"/>
        <field name="amount">100</field>
        <field name="label">Frais bancaires TVA21</field>
    </record>
    <record id="virements_internes_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="l10nbe_chart_template"/>
        <field name="account_id" search="[('code', '=like', obj().env.ref('l10n_be.l10nbe_chart_template').transfer_account_code_prefix + '%'), ('chart_template_id', '=', obj().env.ref('l10n_be.l10nbe_chart_template').id)]"/>
        <field name="name">Virements internes</field>
        <field name="amount_type">percentage</field>
        <field name="amount">100</field>
        <field name="label">Virements internes</field>
        <field name="to_check" eval="False"/>
    </record>
    <record id="compte_attente_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="l10nbe_chart_template"/>
        <field name="account_id" ref="a4990"/>
        <field name="name">Compte Attente</field>
        <field name="amount_type">percentage</field>
        <field name="amount">100</field>
        <field name="label"></field>
        <field name="to_check" eval="True"/>
    </record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record id="tax_report_title_operations" model="account.tax.report.line">
            <field name="name">Opérations</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_title_operations_sortie" model="account.tax.report.line">
            <field name="name">II A la sortie</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_operations"/>
        </record>

        <record id="tax_report_line_00" model="account.tax.report.line">
            <field name="name">00 - Opérations soumises à un régime particulier</field>
            <field name="code">c00</field>
            <field name="tag_name">00</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        </record>

        <record id="tax_report_line_01" model="account.tax.report.line">
            <field name="name">01 - Opérations avec TVA à 6%</field>
            <field name="code">c01</field>
            <field name="tag_name">01</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        </record>

        <record id="tax_report_line_02" model="account.tax.report.line">
            <field name="name">02 - Opérations avec TVA à 12%</field>
            <field name="code">c02</field>
            <field name="tag_name">02</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">3</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        </record>

        <record id="tax_report_line_03" model="account.tax.report.line">
            <field name="name">03 - Opérations avec TVA à 21%</field>
            <field name="code">c03</field>
            <field name="tag_name">03</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">4</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        </record>

        <record id="tax_report_line_44" model="account.tax.report.line">
            <field name="name">44 - Services intra-communautaires</field>
            <field name="code">c44</field>
            <field name="tag_name">44</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">5</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        </record>

        <record id="tax_report_line_45" model="account.tax.report.line">
            <field name="name">45 - Opérations avec TVA due par le cocontractant</field>
            <field name="code">c45</field>
            <field name="tag_name">45</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">6</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        </record>

        <record id="tax_report_title_operations_sortie_46" model="account.tax.report.line">
            <field name="name">46 - Livraisons intra-communautaires exemptées</field>
            <field name="code">c46</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">7</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        </record>

        <record id="tax_report_line_46L" model="account.tax.report.line">
            <field name="name">46L - Livraisons biens intra-communautaires exemptées</field>
            <field name="tag_name">46L</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie_46"/>
        </record>

        <record id="tax_report_line_46T" model="account.tax.report.line">
            <field name="name">46T - Livraisons biens intra-communautaire exemptées</field>
            <field name="tag_name">46T</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie_46"/>
        </record>

        <record id="tax_report_line_47" model="account.tax.report.line">
            <field name="name">47 - Autres opérations exemptées</field>
            <field name="code">c47</field>
            <field name="tag_name">47</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">8</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        </record>

        <record id="tax_report_title_operations_sortie_48" model="account.tax.report.line">
            <field name="name">48 - Notes de crédit aux opérations grilles [44] et [46]</field>
            <field name="code">c48</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">9</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        </record>

        <record id="tax_report_line_48s44" model="account.tax.report.line">
            <field name="name">48s44 - Notes de crédit aux opérations grilles [44]</field>
            <field name="tag_name">48s44</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie_48"/>
        </record>

        <record id="tax_report_line_48s46L" model="account.tax.report.line">
            <field name="name">48s46L - Notes de crédit aux opérations grilles [46L]</field>
            <field name="tag_name">48s46L</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie_48"/>
        </record>

        <record id="tax_report_line_48s46T" model="account.tax.report.line">
            <field name="name">48s46T - Notes de crédit aux opérations grilles [46T]</field>
            <field name="tag_name">48s46T</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">3</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie_48"/>
        </record>

        <record id="tax_report_line_49" model="account.tax.report.line">
            <field name="name">49 - Notes de crédit aux opérations du point II</field>
            <field name="code">c49</field>
            <field name="tag_name">49</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">10</field>
            <field name="parent_id" ref="tax_report_title_operations_sortie"/>
        </record>

        <record id="tax_report_title_operations_entree" model="account.tax.report.line">
            <field name="name">III A l'entrée</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_operations"/>
        </record>

        <record id="tax_report_line_81" model="account.tax.report.line">
            <field name="name">81 - Marchandises, matières premières et auxiliaires</field>
            <field name="code">c81</field>
            <field name="tag_name">81</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_operations_entree"/>
        </record>

        <record id="tax_report_line_82" model="account.tax.report.line">
            <field name="name">82 - Services et biens divers</field>
            <field name="code">c82</field>
            <field name="tag_name">82</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_operations_entree"/>
        </record>

        <record id="tax_report_line_83" model="account.tax.report.line">
            <field name="name">83 - Biens d'investissement</field>
            <field name="code">c83</field>
            <field name="tag_name">83</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">3</field>
            <field name="parent_id" ref="tax_report_title_operations_entree"/>
        </record>

        <record id="tax_report_line_84" model="account.tax.report.line">
            <field name="name">84 - Notes de crédits sur opérations case [86] et [88]</field>
            <field name="code">c84</field>
            <field name="tag_name">84</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">4</field>
            <field name="parent_id" ref="tax_report_title_operations_entree"/>
        </record>

        <record id="tax_report_line_85" model="account.tax.report.line">
            <field name="name">85 - Notes de crédits autres opérations</field>
            <field name="code">c85</field>
            <field name="tag_name">85</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">5</field>
            <field name="parent_id" ref="tax_report_title_operations_entree"/>
        </record>

        <record id="tax_report_line_86" model="account.tax.report.line">
            <field name="name">86 - Acquisition intra-communautaires et ventes ABC</field>
            <field name="code">c86</field>
            <field name="tag_name">86</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">6</field>
            <field name="parent_id" ref="tax_report_title_operations_entree"/>
        </record>

        <record id="tax_report_line_87" model="account.tax.report.line">
            <field name="name">87 - Autres opérations</field>
            <field name="code">c87</field>
            <field name="tag_name">87</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">7</field>
            <field name="parent_id" ref="tax_report_title_operations_entree"/>
        </record>

        <record id="tax_report_line_88" model="account.tax.report.line">
            <field name="name">88 - Acquisition services intra-communautaires</field>
            <field name="code">c88</field>
            <field name="tag_name">88</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">8</field>
            <field name="parent_id" ref="tax_report_title_operations_entree"/>
        </record>

        <record id="tax_report_title_taxes" model="account.tax.report.line">
            <field name="name">Taxes</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">2</field>
        </record>

        <record id="tax_report_title_taxes_dues" model="account.tax.report.line">
            <field name="name">IV Dues</field>
            <field name="code">tax_be_iv</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_taxes"/>
        </record>

        <record id="tax_report_line_54" model="account.tax.report.line">
            <field name="name">54 - TVA sur opérations des grilles [01], [02], [03]</field>
            <field name="code">c54</field>
            <field name="tag_name">54</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_taxes_dues"/>
        </record>

        <record id="tax_report_line_55" model="account.tax.report.line">
            <field name="name">55 - TVA sur opérations des grilles [86] et [88]</field>
            <field name="code">c55</field>
            <field name="tag_name">55</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_taxes_dues"/>
        </record>

        <record id="tax_report_line_56" model="account.tax.report.line">
            <field name="name">56 - TVA sur opérations de la grille [87]</field>
            <field name="code">c56</field>
            <field name="tag_name">56</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">3</field>
            <field name="parent_id" ref="tax_report_title_taxes_dues"/>
        </record>

        <record id="tax_report_line_57" model="account.tax.report.line">
            <field name="name">57 - TVA relatives aux importations</field>
            <field name="code">c57</field>
            <field name="tag_name">57</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">4</field>
            <field name="parent_id" ref="tax_report_title_taxes_dues"/>
        </record>

        <record id="tax_report_line_61" model="account.tax.report.line">
            <field name="name">61 - Diverses régularisations en faveur de l'Etat</field>
            <field name="code">c61</field>
            <field name="tag_name">61</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">5</field>
            <field name="parent_id" ref="tax_report_title_taxes_dues"/>
        </record>

        <record id="tax_report_line_63" model="account.tax.report.line">
            <field name="name">63 - TVA à reverser sur notes de crédit recues</field>
            <field name="code">c63</field>
            <field name="tag_name">63</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">6</field>
            <field name="parent_id" ref="tax_report_title_taxes_dues"/>
        </record>

        <record id="tax_report_title_taxes_deductibles" model="account.tax.report.line">
            <field name="name">V Déductibles</field>
            <field name="code">tax_be_v</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_taxes"/>
        </record>

        <record id="tax_report_line_59" model="account.tax.report.line">
            <field name="name">59 - TVA déductible</field>
            <field name="code">c59</field>
            <field name="tag_name">59</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_taxes_deductibles"/>
        </record>

        <record id="tax_report_line_62" model="account.tax.report.line">
            <field name="name">62 - Diverses régularisations en faveur du déclarant</field>
            <field name="code">c62</field>
            <field name="tag_name">62</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_taxes_deductibles"/>
        </record>

        <record id="tax_report_line_64" model="account.tax.report.line">
            <field name="name">64 - TVA à récupérer sur notes de crédit delivrées</field>
            <field name="code">c64</field>
            <field name="tag_name">64</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">3</field>
            <field name="parent_id" ref="tax_report_title_taxes_deductibles"/>
        </record>

        <record id="tax_report_title_taxes_soldes" model="account.tax.report.line">
            <field name="name">VI Soldes</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">3</field>
            <field name="parent_id" ref="tax_report_title_taxes"/>
        </record>

        <record id="tax_report_line_71" model="account.tax.report.line">
            <field name="name">71 - Taxes dues à l'état</field>
            <field name="formula">tax_be_iv&gt;tax_be_v and tax_be_iv-tax_be_v or 0</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_taxes_soldes"/>
        </record>

        <record id="tax_report_line_72" model="account.tax.report.line">
            <field name="name">72 - Somme due par l'état</field>
            <field name="formula">tax_be_iv&lt;tax_be_v and tax_be_v-tax_be_iv or 0</field>
            <field name="country_id" ref="base.be"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_taxes_soldes"/>
        </record>

    </data>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

        <record id="attn_VAT-OUT-21-L" model="account.tax.template">
            <field name="sequence">10</field>
            <field name="description">TVA 21%</field>
            <field name="name">21%</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_03')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451054'),
                    'plus_report_line_ids': [ref('tax_report_line_54')],
                }),

            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451054'),
                    'plus_report_line_ids': [ref('tax_report_line_64')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-21-S" model="account.tax.template">
            <field name="sequence">11</field>
            <field name="description">TVA 21%</field>
            <field name="name">21% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_03')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451054'),
                    'plus_report_line_ids': [ref('tax_report_line_54')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451054'),
                    'plus_report_line_ids': [ref('tax_report_line_64')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-12-S" model="account.tax.template">
            <field name="sequence">20</field>
            <field name="description">TVA 12%</field>
            <field name="name">12% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_02')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451054'),
                    'plus_report_line_ids': [ref('tax_report_line_54')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451054'),
                    'plus_report_line_ids': [ref('tax_report_line_64')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-12-L" model="account.tax.template">
            <field name="sequence">21</field>
            <field name="description">TVA 12%</field>
            <field name="name">12%</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_02')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451054'),
                    'plus_report_line_ids': [ref('tax_report_line_54')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451054'),
                    'plus_report_line_ids': [ref('tax_report_line_64')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-06-S" model="account.tax.template">
            <field name="sequence">30</field>
            <field name="description">TVA 6%</field>
            <field name="name">6% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_01')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451054'),
                    'plus_report_line_ids': [ref('tax_report_line_54')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451054'),
                    'plus_report_line_ids': [ref('tax_report_line_64')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-06-L" model="account.tax.template">
            <field name="sequence">31</field>
            <field name="description">TVA 6%</field>
            <field name="name">6%</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_01')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451054'),
                    'plus_report_line_ids': [ref('tax_report_line_54')]
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451054'),
                    'plus_report_line_ids': [ref('tax_report_line_64')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-00-S" model="account.tax.template">
            <field name="sequence">40</field>
            <field name="description">TVA 0%</field>
            <field name="name">0% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_00')],
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
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-00-L" model="account.tax.template">
            <field name="sequence">41</field>
            <field name="description">TVA 0%</field>
            <field name="name">0%</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_00')],
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
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-00-CC" model="account.tax.template">
            <field name="sequence">50</field>
            <field name="description">TVA 0% Cocont.</field>
            <field name="name">0% Cocont.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_45')],
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
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-00-EU-S" model="account.tax.template">
            <field name="sequence">60</field>
            <field name="description">TVA 0% EU</field>
            <field name="name">0% EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_44')],
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
                    'plus_report_line_ids': [ref('tax_report_line_48s44')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-00-EU-L" model="account.tax.template">
            <field name="sequence">61</field>
            <field name="description">TVA 0% EU</field>
            <field name="name">0% EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_46L')],
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
                    'plus_report_line_ids': [ref('tax_report_line_48s46L')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-00-EU-T" model="account.tax.template">
            <field name="sequence">62</field>
            <field name="description">TVA 0% EU</field>
            <field name="name">0% EU T.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_46T')],
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
                    'plus_report_line_ids': [ref('tax_report_line_48s46T')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-OUT-00-ROW" model="account.tax.template">
            <field name="sequence">70</field>
            <field name="description">TVA 0% Non EU</field>
            <field name="name">0% Non EU</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_47')],
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
                    'plus_report_line_ids': [ref('tax_report_line_49')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-21" model="account.tax.template">
            <field name="sequence">110</field>
            <field name="description">TVA 21%</field>
            <field name="name">21% M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-12" model="account.tax.template">
            <field name="sequence">120</field>
            <field name="description">TVA 12%</field>
            <field name="name">12% M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-06" model="account.tax.template">
            <field name="sequence">130</field>
            <field name="description">TVA 6%</field>
            <field name="name">6% M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-00" model="account.tax.template">
            <field name="sequence">140</field>
            <field name="description">TVA 0%</field>
            <field name="name">0% M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81')],
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
                    'minus_report_line_ids': [ref('tax_report_line_81')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

         <record id="attn_TVA-21-inclus-dans-prix" model="account.tax.template">
            <field name="sequence">150</field>
            <field name="description">TVA 21% TTC</field>
            <field name="name">21% S. TTC</field>
            <field name="price_include" eval="1"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-21-S" model="account.tax.template">
            <field name="sequence">210</field>
            <field name="description">TVA 21%</field>
            <field name="name">21% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-21-G" model="account.tax.template">
            <field name="sequence">220</field>
            <field name="description">TVA 21%</field>
            <field name="name">21% Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-12-S" model="account.tax.template">
            <field name="sequence">230</field>
            <field name="description">TVA 12%</field>
            <field name="name">12% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-12-G" model="account.tax.template">
            <field name="sequence">240</field>
            <field name="description">TVA 12%</field>
            <field name="name">12% Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-06-S" model="account.tax.template">
            <field name="sequence">250</field>
            <field name="description">TVA 6%</field>
            <field name="name">6% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-06-G" model="account.tax.template">
            <field name="sequence">260</field>
            <field name="description">TVA 6%</field>
            <field name="name">6% Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-00-S" model="account.tax.template">
            <field name="sequence">270</field>
            <field name="description">TVA 0%</field>
            <field name="name">0% S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
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
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-00-G" model="account.tax.template">
            <field name="sequence">280</field>
            <field name="description">TVA 0%</field>
            <field name="name">0% Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
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
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-21" model="account.tax.template">
            <field name="sequence">310</field>
            <field name="description">TVA 21%</field>
            <field name="name">21% Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-12" model="account.tax.template">
            <field name="sequence">320</field>
            <field name="description">TVA 12%</field>
            <field name="name">12% Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-06" model="account.tax.template">
            <field name="sequence">330</field>
            <field name="description">TVA 6%</field>
            <field name="name">6% Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-00" model="account.tax.template">
            <field name="sequence">340</field>
            <field name="description">TVA 0%</field>
            <field name="name">0% Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83')],
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
                    'minus_report_line_ids': [ref('tax_report_line_83')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-21-CC" model="account.tax.template">
            <field name="sequence">410</field>
            <field name="description">TVA 21% Cocont.</field>
            <field name="name">21% Cocont. M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-12-CC" model="account.tax.template">
            <field name="sequence">420</field>
            <field name="description">TVA 12% Cocont.</field>
            <field name="name">12% Cocont. M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-06-CC" model="account.tax.template">
            <field name="sequence">430</field>
            <field name="description">TVA 6% Cocont.</field>
            <field name="name">6% Cocont. M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="amount">6</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-00-CC" model="account.tax.template">
            <field name="sequence">440</field>
            <field name="description">TVA 0% Cocont.</field>
            <field name="name">0% Cocont. M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
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
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-21-CC" model="account.tax.template">
            <field name="sequence">510</field>
            <field name="description">TVA 21% Cocont.</field>
            <field name="name">21% Cocont .S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-12-CC" model="account.tax.template">
            <field name="sequence">520</field>
            <field name="description">TVA 12% Cocont.</field>
            <field name="name">12% Cocont. S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-06-CC" model="account.tax.template">
            <field name="sequence">530</field>
            <field name="description">TVA 6% Cocont.</field>
            <field name="name">6% Cocont. S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-00-CC" model="account.tax.template">
            <field name="sequence">540</field>
            <field name="description">TVA 0% Cocont.</field>
            <field name="name">0% Cocont. S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
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
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-21-CC" model="account.tax.template">
            <field name="sequence">610</field>
            <field name="description">TVA 21% Cocont.</field>
            <field name="name">21% Cocont. - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-12-CC" model="account.tax.template">
            <field name="sequence">620</field>
            <field name="description">TVA 12% Cocont.</field>
            <field name="name">12% Cocont. - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-06-CC" model="account.tax.template">
            <field name="sequence">630</field>
            <field name="description">TVA 6% Cocont.</field>
            <field name="name">6% Cocont. - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                    'minus_report_line_ids': [ref('tax_report_line_56')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451056'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-00-CC" model="account.tax.template">
            <field name="sequence">640</field>
            <field name="description">TVA 0% Cocont.</field>
            <field name="name">0% Cocont. - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
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
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-CAR-EXC" model="account.tax.template">
            <field name="sequence">720</field>
            <field name="description">TVA 50% Non Déductible - Frais de voiture (Prix Excl.)</field>
            <field name="name">50% Non Déductible - Frais de voiture (Prix Excl.)</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 50,
                    'repartition_type': 'tax',
                    'account_id': ref('a64012'),
                    'plus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 50,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                }),

                (0,0, {
                    'factor_percent': 50,
                    'repartition_type': 'tax',
                    'account_id': ref('a64012'),
                    'minus_report_line_ids': [ref('tax_report_line_82')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 50,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_63')],
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-21-EU" model="account.tax.template">
            <field name="sequence">1110</field>
            <field name="description">TVA 21% EU</field>
            <field name="name">21% EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-12-EU" model="account.tax.template">
            <field name="sequence">1120</field>
            <field name="description">TVA 12% EU</field>
            <field name="name">12% EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-06-EU" model="account.tax.template">
            <field name="sequence">1130</field>
            <field name="description">TVA 6% EU</field>
            <field name="name">6% EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-00-EU" model="account.tax.template">
            <field name="sequence">1140</field>
            <field name="description">TVA 0% EU</field>
            <field name="name">0% EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
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
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-21-EU-S" model="account.tax.template">
            <field name="sequence">1210</field>
            <field name="description">TVA 21% EU</field>
            <field name="name">21% EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-21-EU-G" model="account.tax.template">
            <field name="sequence">1220</field>
            <field name="description">TVA 21% EU</field>
            <field name="name">21% EU - Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-12-EU-S" model="account.tax.template">
            <field name="sequence">1230</field>
            <field name="description">TVA 12% EU</field>
            <field name="name">12% EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-12-EU-G" model="account.tax.template">
            <field name="sequence">1240</field>
            <field name="description">TVA 12% EU</field>
            <field name="name">12% EU - Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-06-EU-S" model="account.tax.template">
            <field name="sequence">1250</field>
            <field name="description">TVA 6% EU</field>
            <field name="name">6% EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="amount">6</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-06-EU-G" model="account.tax.template">
            <field name="sequence">1260</field>
            <field name="description">TVA 6% EU</field>
            <field name="name">6% EU - Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-00-EU-S" model="account.tax.template">
            <field name="sequence">1270</field>
            <field name="description">TVA 0% EU</field>
            <field name="name">0% EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
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
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_88')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-21-EU" model="account.tax.template">
            <field name="sequence">1310</field>
            <field name="description">TVA 21% EU</field>
            <field name="name">21% EU - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-00-EU-G" model="account.tax.template">
            <field name="sequence">1280</field>
            <field name="description">TVA 0% EU</field>
            <field name="name">0% EU - Biens divers</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
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
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-12-EU" model="account.tax.template">
            <field name="sequence">1320</field>
            <field name="description">TVA 12% EU</field>
            <field name="name">12% EU - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-06-EU" model="account.tax.template">
            <field name="sequence">1330</field>
            <field name="description">TVA 6% EU</field>
            <field name="name">6% EU - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="amount">6</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                    'minus_report_line_ids': [ref('tax_report_line_55')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451055'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-00-EU" model="account.tax.template">
            <field name="sequence">1340</field>
            <field name="description">TVA 0% EU</field>
            <field name="name">0% EU - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
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
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_86')],
                    'plus_report_line_ids': [ref('tax_report_line_84')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-21-ROW-CC" model="account.tax.template">
            <field name="sequence">2110</field>
            <field name="description">TVA 21% Non EU</field>
            <field name="name">21% Non EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">21</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-12-ROW-CC" model="account.tax.template">
            <field name="sequence">2120</field>
            <field name="description">TVA 12% Non EU</field>
            <field name="name">12% Non EU M.</field>
            <field name="amount">12</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-06-ROW-CC" model="account.tax.template">
            <field name="sequence">2130</field>
            <field name="description">TVA 6% Non EU</field>
            <field name="name">6% Non EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V81-00-ROW-CC" model="account.tax.template">
            <field name="sequence">2140</field>
            <field name="description">TVA 0% Non EU</field>
            <field name="name">0% Non EU M.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
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
                    'minus_report_line_ids': [ref('tax_report_line_81'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-21-ROW-CC" model="account.tax.template">
            <field name="sequence">2210</field>
            <field name="description">TVA 21% Non EU</field>
            <field name="name">21% Non EU S.</field>
            <field name="amount">21</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-12-ROW-CC" model="account.tax.template">
            <field name="sequence">2220</field>
            <field name="description">TVA 12% Non EU</field>
            <field name="name">12% Non EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-06-ROW-CC" model="account.tax.template">
            <field name="sequence">2230</field>
            <field name="description">TVA 6% Non EU</field>
            <field name="name">6% Non EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">6</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V82-00-ROW-CC" model="account.tax.template">
            <field name="sequence">2240</field>
            <field name="description">TVA 0% Non EU</field>
            <field name="name">0% Non EU S.</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
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
                    'minus_report_line_ids': [ref('tax_report_line_82'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-21-ROW-CC" model="account.tax.template">
            <field name="sequence">2310</field>
            <field name="description">TVA 21% Non EU</field>
            <field name="name">21% Non EU - Biens d'investissement</field>
            <field name="amount">21</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_21"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-12-ROW-CC" model="account.tax.template">
            <field name="sequence">2320</field>
            <field name="description">TVA 12% Non EU</field>
            <field name="name">12% Non EU - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-06-ROW-CC" model="account.tax.template"> <!--merged group-->
            <field name="sequence">2330</field>
            <field name="description">TVA 6% Non EU</field>
            <field name="name">6% Non EU - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount">6</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                    'plus_report_line_ids': [ref('tax_report_line_59')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                    'minus_report_line_ids': [ref('tax_report_line_57')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a411059'),
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a451057'),
                }),
            ]"/>
        </record>

        <record id="attn_VAT-IN-V83-00-ROW-CC" model="account.tax.template">
            <field name="sequence">2340</field>
            <field name="description">TVA 0% Non EU</field>
            <field name="name">0% Non EU - Biens d'investissement</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="active" eval="False"/>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="tax_group_id" ref="tax_group_tva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
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
                    'minus_report_line_ids': [ref('tax_report_line_83'), ref('tax_report_line_87')],
                    'plus_report_line_ids': [ref('tax_report_line_85')],
                }),

                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

</odoo>

```

## File: data\fiscal_templates_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Fiscal Position Templates -->
    
    <record id="fiscal_position_template_1" model="account.fiscal.position.template">
            <field name="sequence">1</field>
            <field name="name">Régime National</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_id" ref="base.be"/>
    </record>

    <record id="fiscal_position_template_5" model="account.fiscal.position.template">
            <field name="sequence">2</field>
            <field name="name">EU privé</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
    </record>

    <record id="fiscal_position_template_2" model="account.fiscal.position.template">
            <field name="sequence">4</field>
            <field name="name">Régime Extra-Communautaire</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="auto_apply" eval="True"/>
    </record>

    <record id="fiscal_position_template_3" model="account.fiscal.position.template">
            <field name="sequence">3</field>
            <field name="name">Régime Intra-Communautaire</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
    </record>

    <record id="fiscal_position_template_4" model="account.fiscal.position.template">
            <field name="name">Régime Cocontractant</field>
            <field name="sequence">5</field>
            <field name="chart_template_id" ref="l10nbe_chart_template"/>
    </record>    

    <!-- Fiscal Position Account Templates -->
    
    <record id="fiscal_position_account_template_3" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="account_src_id" ref="l10n_be.a7000" />
            <field name="account_dest_id" ref="l10n_be.a7001" />
    </record>
    
    <record id="fiscal_position_account_template_4" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="account_src_id" ref="l10n_be.a7010" />
            <field name="account_dest_id" ref="l10n_be.a7011" />
    </record>

    <record id="fiscal_position_account_template_6" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="account_src_id" ref="l10n_be.a7050" />
            <field name="account_dest_id" ref="l10n_be.a7051" />
    </record>

    <record id="fiscal_position_account_template_7" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="account_src_id" ref="l10n_be.a7000" />
            <field name="account_dest_id" ref="l10n_be.a7002" />
    </record>
    <record id="fiscal_position_account_template_8" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="account_src_id" ref="l10n_be.a7010" />
            <field name="account_dest_id" ref="l10n_be.a7012" />
    </record>
    <record id="fiscal_position_account_template_10" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="account_src_id" ref="l10n_be.a7050" />
            <field name="account_dest_id" ref="l10n_be.a7052" />
    </record>
</odoo>

```

## File: data\l10n_be_sequence_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!--
    Sequences for declarantnum will be used in wizard for "Listing of VAT Customers"..in creating xml file
        -->
    <record model="ir.sequence" id="seq_declarantnum">
        <field name="name">Declarantnum</field>
        <field name="code">declarantnum</field>
        <field name="padding">5</field>
        <field name="company_id" eval="False"/>
    </record>
</odoo>

```

## File: data\menuitem_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_be_statements_menu" name="Belgium" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_user"/>
</odoo>

```

## File: i18n_extra\l10n_be.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_be
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 13.0\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2020-01-27 14:01+0000\n"
"PO-Revision-Date: 2020-01-27 14:01+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-00
msgid "0% Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-00-G
msgid "0% Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-00-CC
msgid "0% Cocont."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-00-CC
msgid "0% Cocont. - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-00-CC
msgid "0% Cocont. M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-00-CC
msgid "0% Cocont. S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-00-EU
msgid "0% EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-00-EU-G
msgid "0% EU - Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-00-EU
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-00-EU-L
msgid "0% EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-00-EU-S
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-00-EU-S
msgid "0% EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-00-EU-T
msgid "0% EU T."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-00
msgid "0% M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-00-ROW
msgid "0% Non EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-00-ROW-CC
msgid "0% Non EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-00-ROW-CC
msgid "0% Non EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-00-ROW-CC
msgid "0% Non EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-00-S
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-00-S
msgid "0% S."
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_00
msgid "00"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_00
msgid "00 - Opérations soumises à un régime particulier"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_01
msgid "01"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_01
msgid "01 - Opérations avec TVA à 6%"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_02
msgid "02"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_02
msgid "02 - Opérations avec TVA à 12%"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_03
msgid "03"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_03
msgid "03 - Opérations avec TVA à 21%"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-12-L
msgid "12%"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-12
msgid "12% Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-12-G
msgid "12% Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-12-CC
msgid "12% Cocont. - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-12-CC
msgid "12% Cocont. M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-12-CC
msgid "12% Cocont. S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-12-EU
msgid "12% EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-12-EU-G
msgid "12% EU - Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-12-EU
msgid "12% EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-12-EU-S
msgid "12% EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-12
msgid "12% M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-12-ROW-CC
msgid "12% Non EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-12-ROW-CC
msgid "12% Non EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-12-ROW-CC
msgid "12% Non EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-12-S
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-12-S
msgid "12% S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-21-L
msgid "21%"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-21
msgid "21% Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-21-G
msgid "21% Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-21-CC
msgid "21% Cocont .S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-21-CC
msgid "21% Cocont. - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-21-CC
msgid "21% Cocont. M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-21-EU
msgid "21% EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-21-EU-G
msgid "21% EU - Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-21-EU
msgid "21% EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-21-EU-S
msgid "21% EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-21
msgid "21% M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-21-ROW-CC
msgid "21% Non EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-21-ROW-CC
msgid "21% Non EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-21-ROW-CC
msgid "21% Non EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-21-S
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-21-S
msgid "21% S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_TVA-21-inclus-dans-prix
msgid "21% S. TTC"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_44
msgid "44"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_44
msgid "44 - Services intra-communautaires"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_45
msgid "45"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_45
msgid "45 - Opérations avec TVA due par le cocontractant"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_operations_sortie_46
msgid "46 - Livraisons intra-communautaires exemptées"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_46L
msgid "46L"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_46L
msgid "46L - Livraisons biens intra-communautaires exemptées"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_46T
msgid "46T"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_46T
msgid "46T - Livraisons biens intra-communautaire exemptées"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_47
msgid "47"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_47
msgid "47 - Autres opérations exemptées"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_operations_sortie_48
msgid "48 - Notes de crédit aux opérations grilles [44] et [46]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_48s44
msgid "48s44"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_48s44
msgid "48s44 - Notes de crédit aux opérations grilles [44]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_48s46L
msgid "48s46L"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_48s46L
msgid "48s46L - Notes de crédit aux opérations grilles [46L]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_48s46T
msgid "48s46T"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_48s46T
msgid "48s46T - Notes de crédit aux opérations grilles [46T]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_49
msgid "49"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_49
msgid "49 - Notes de crédit aux opérations du point II"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-CAR-EXC
msgid "50% Non Déductible - Frais de voiture (Prix Excl.)"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_54
msgid "54"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_54
msgid "54 - TVA sur opérations des grilles [01], [02], [03]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_55
msgid "55"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_55
msgid "55 - TVA sur opérations des grilles [86] et [88]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_56
msgid "56"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_56
msgid "56 - TVA sur opérations de la grille [87]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_57
msgid "57"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_57
msgid "57 - TVA relatives aux importations"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_59
msgid "59"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_59
msgid "59 - TVA déductible"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-06
msgid "6% Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-06-G
msgid "6% Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-06-CC
msgid "6% Cocont. - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-06-CC
msgid "6% Cocont. M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-06-CC
msgid "6% Cocont. S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-06-EU
msgid "6% EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-06-EU-G
msgid "6% EU - Biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-06-EU
msgid "6% EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-06-EU-S
msgid "6% EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-06
msgid "6% M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V83-06-ROW-CC
msgid "6% Non EU - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V81-06-ROW-CC
msgid "6% Non EU M."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-06-ROW-CC
msgid "6% Non EU S."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,name:l10n_be.attn_VAT-IN-V82-06-S
#: model:account.tax.template,name:l10n_be.attn_VAT-OUT-06-S
msgid "6% S."
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_61
msgid "61"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_61
msgid "61 - Diverses régularisations en faveur de l'Etat"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_62
msgid "62"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_62
msgid "62 - Diverses régularisations en faveur du déclarant"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_63
msgid "63"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_63
msgid "63 - TVA à reverser sur notes de crédit recues"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_64
msgid "64"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_64
msgid "64 - TVA à récupérer sur notes de crédit delivrées"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_71
msgid "71 - Taxes dues à l'état"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_72
msgid "72 - Somme due par l'état"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_81
msgid "81"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_81
msgid "81 - Marchandises, matières premières et auxiliaires"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_82
msgid "82"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_82
msgid "82 - Services et biens divers"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_83
msgid "83"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_83
msgid "83 - Biens d'investissement"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_84
msgid "84"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_84
msgid "84 - Notes de crédits sur opérations case [86] et [88]"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_85
msgid "85"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_85
msgid "85 - Notes de crédits autres opérations"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_86
msgid "86"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_86
msgid "86 - Acquisition intra-communautaires et ventes ABC"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_87
msgid "87"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_87
msgid "87 - Autres opérations"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,tag_name:l10n_be.tax_report_line_88
msgid "88"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_line_88
msgid "88 - Acquisition services intra-communautaires"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a62022
msgid "ATN - Employés"
msgstr ""

#. module: l10n_be
#: model:ir.model,name:l10n_be.model_account_chart_template
msgid "Account Chart Template"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a601
msgid "Achats de fournitures"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a604
msgid "Achats de marchandises"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a600
msgid "Achats de matières premières"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a602
msgid "Achats de services, travaux et études"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a446
msgid "Acomptes reçus"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a176
msgid "Acomptes reçus sur commandes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a406
msgid "Acomptes versés"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a360
msgid "Acomptes versés sur achats pour stocks"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4550
msgid "Administrateurs, gérants et commissaires"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2119
msgid "Amortissements sur concessions, brevets, etc..."
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2690
msgid "Amortissements sur frais d'aménagement des locaux pris en location"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2009
msgid "Amortissements sur frais de constitution et d'augmentation de capital"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2109
msgid "Amortissements sur frais de recherche et de mise au point"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a24190
msgid "Amortissements sur matériel automobile"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a24090
msgid "Amortissements sur mobilier"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2529
msgid "Amortissements sur mobilier et matériel roulant en leasing"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a614
msgid "Annonces, publicité, propagande et documentation"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a3100
msgid "Approvisionements et fournitures"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4160
msgid "Associés"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a45932
msgid "Assurance groupe "
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a45930
msgid "Assurance loi"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a45931
msgid "Assurance salaire garanti "
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a45933
msgid "Assurances individuelles"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a41671
msgid "Autres créances"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a174
#: model:account.account.template,name:l10n_be.a424
msgid "Autres emprunts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4125
msgid "Autres impôts belges"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a49102
#: model:account.account.template,name:l10n_be.a740
#: model:account.account.template,name:l10n_be.a7492
msgid "Autres produits d'exploitation"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4538
msgid "Autres précomptes retenus"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4161
msgid "Avances et prêts au personnel"
msgstr ""

#. module: l10n_be
#: model:account.chart.template,name:l10n_be.l10nbe_chart_template
msgid "Belgian PCMN"
msgstr ""

#. module: l10n_be
#: model:ir.ui.menu,name:l10n_be.account_reports_be_statements_menu
msgid "Belgium"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a140
msgid "Bénéfice reporté"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a790
msgid "Bénéfice reporté de l'exercice précédent"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a693
msgid "Bénéfice à reporter"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4594
msgid "Caisse d'assurances sociales pour travailleurs indépendants"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a24105
msgid "Camions"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1000
msgid "Capital"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a418
msgid "Cautionnements versés en numéraires"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6600
msgid "Charges exceptionelles"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6101
msgid "Charges locatives"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a492
msgid "Charges à imputer"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a490
msgid "Charges à reporter"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a540
msgid "Chèques à encaisser"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4000
msgid "Clients"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4001
msgid "Clients (PoS)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a370
msgid "Commandes en cours d'exécution"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a49101
msgid "Commissions à obtenir"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a448
msgid "Compensations fournisseurs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4112
#: model:account.account.template,name:l10n_be.a4512
msgid "Compte courant administration T.V.A."
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4163
msgid "Compte courant des administrateurs et gérants"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4162
msgid "Compte courant des associés en S.P.R.L."
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4990
msgid "Compte d'attente"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2110
msgid "Concessions, brevets, licences, savoir-faire, marques, etc..."
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2501
msgid "Constructions"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a621
msgid "Cotisations patronales d'assurances sociales"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a29000
msgid "Créances commerciales"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a407
#: model:account.account.template,name:l10n_be.a417
msgid "Créances douteuses"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4164
msgid "Créances sur sociétés apparentées"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1720
msgid "Dettes de location-financement de biens immobiliers"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1721
msgid "Dettes de location-financement de biens mobiliers"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a17300
msgid "Dettes en compte \\ Banque A"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4597
msgid "Dettes et provisions sociales diverses"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4500
msgid "Dettes fiscales estimées"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1722
msgid "Dettes sur droits réels sur immeubles"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a654
#: model:account.account.template,name:l10n_be.a754
msgid "Différences de change"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4551
#: model:account.account.template,name:l10n_be.a4560
msgid "Direction"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6302
msgid "Dotations aux amortissements sur immobilisations corporelles"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6301
msgid "Dotations aux amortissements sur immobilisations incorporelles"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a755
msgid "Ecarts de conversion des devises"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a175111
msgid "Effets à payer \\ Fournisseurs C.E.E."
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a175110
msgid "Effets à payer \\ Fournisseurs belges"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a175112
msgid "Effets à payer \\ Fournisseurs importation"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4166
msgid "Emballages et matériel à rendre"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4552
#: model:account.account.template,name:l10n_be.a4561
msgid "Employés"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a3200
msgid "En cours de fabrication"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a611
msgid "Entretien et réparation"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a653
msgid "Escomptes clients"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a757
msgid "Escomptes obtenus"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4230
msgid "Etablissements de crédit"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a430
msgid "Etablissements de crédit. Emprunts en compte à terme fixe"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a444
msgid "Factures à recevoir"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4220
msgid "Financement de biens immobiliers"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4221
msgid "Financement de biens mobiliers"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a440
msgid "Fournisseurs"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a612
msgid "Fournitures"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a260
msgid "Frais d'aménagements de locaux pris en location"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a656
msgid "Frais de banques, de chèques postaux"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2000
msgid "Frais de constitution et d'augmentation de capital"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6231
msgid "Frais de déplacement"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2100
msgid "Frais de recherche et de mise au point"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_operations_sortie
msgid "II A la sortie"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_operations_entree
msgid "III A l'entrée"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_taxes_dues
msgid "IV Dues"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a3500
msgid "Immeubles destinés à la vente"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2800
msgid "Immobilisations Financières"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4120
msgid "Impôts belges sur le résultat"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6700
msgid "Impôts et précomptes dus ou versés"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4128
msgid "Impôts étrangers"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.l10nbe_chart_template_liquidity_transfer
msgid "Liquidity Transfer"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6100
msgid "Loyers divers"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a3400
msgid "Marchandises"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a300
msgid "Matières premières"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a24011
msgid "Matériel de bureau de bâtiments administratifs et commerciaux"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a24010
msgid "Matériel de bureau de bâtiments industriels"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2521
msgid "Matériel roulant"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2520
msgid "Mobilier"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a24001
msgid "Mobilier des bâtiments administratifs et commerciaux"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a24000
msgid "Mobilier des bâtiments industriels"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a151
msgid "Montants transférés aux résultats"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4541
msgid "ONSS. 1er trimestre"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4542
msgid "ONSS. 2ème trimestre"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4543
msgid "ONSS. 3ème trimestre"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4544
msgid "ONSS. 4ème trimestre"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4540
msgid "ONSS. Arriérés"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_operations
msgid "Opérations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4553
#: model:account.account.template,name:l10n_be.a4562
msgid "Ouvriers"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a141
msgid "Perte reportée"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a690
msgid "Perte reportée de l'exercice précédent"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a793
msgid "Perte à reporter"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2118
msgid "Plus-values actées sur concessions, brevets, etc..."
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2108
msgid "Plus-values actées sur frais de recherche et de mise au point"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2528
msgid "Plus-values actées sur mobilier et matériel roulant en leasing"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a752
msgid "Plus-values sur réalisations d'actifs circulants"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7051
msgid "Prestations de services dans les pays membres de la C.E.E."
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7050
msgid "Prestations de services en Belgique"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7052
msgid "Prestations de services en vue de l'exportation"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a11
msgid "Primes d'émission"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4930
msgid "Produits d'exploitation à reporter"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a751
msgid "Produits des actifs circulants"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a756
msgid "Produits des autres créances"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7600
msgid "Produits exceptionnels"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a3300
msgid "Produits finis"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a404
#: model:account.account.template,name:l10n_be.a414
msgid "Produits à recevoir"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4590
msgid "Provision pour gratifications de fin d'année"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a1600
msgid "Provisions pour risques et charges"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4533
msgid "Précompte mobilier retenu sur intérêts payés"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a62021
msgid "Précompte professionnel"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4530
msgid "Précompte professionnel retenu sur rémunérations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a791
msgid "Prélèvement sur le capital et les primes d'émission"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a780
msgid "Prélèvements sur les impôts différés"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a789
msgid "Prélèvements sur les réserves immunisées"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a706
msgid "Pénalités et dédits obtenus par l'entreprise"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4168
msgid ""
"Rabais, ristournes, remises à obtenir et autres avoirs non encore reçus"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7500
msgid "Revenus des actions"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7502
msgid "Revenus des créances à plus d'un an"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7501
msgid "Revenus des obligations"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a49100
msgid "Ristournes, rabais à obtenir"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a329
#: model:account.account.template,name:l10n_be.a419
msgid "Réductions de valeur actées"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6320
msgid "Réductions de valeur sur commandes en cours d'exécution"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6310
msgid "Réductions de valeur sur stocks"
msgstr ""

#. module: l10n_be
#: model:account.fiscal.position.template,name:l10n_be.fiscal_position_template_5
msgid "EU privé"
msgstr ""

#. module: l10n_be
#: model:account.fiscal.position.template,name:l10n_be.fiscal_position_template_4
msgid "Régime Cocontractant"
msgstr ""

#. module: l10n_be
#: model:account.fiscal.position.template,name:l10n_be.fiscal_position_template_2
msgid "Régime Extra-Communautaire"
msgstr ""

#. module: l10n_be
#: model:account.fiscal.position.template,name:l10n_be.fiscal_position_template_3
msgid "Régime Intra-Communautaire"
msgstr ""

#. module: l10n_be
#: model:account.fiscal.position.template,name:l10n_be.fiscal_position_template_1
msgid "Régime National"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7710
msgid "Régularisations d'impôts dus ou versés"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7711
msgid "Régularisations d'impôts estimés"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6200
msgid "Rémunérations: Administrateurs ou gérants"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6204
msgid "Rémunérations: Autres membres du personnel"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6202
msgid "Rémunérations: Employés"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6203
msgid "Rémunérations: Ouvriers"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a6201
msgid "Rémunérations: Personnel de direction"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a130
msgid "Réserve légale"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a132
msgid "Réserves immunisées"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a613
msgid "Rétributions de tiers"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a603
msgid "Sous-traitances générales"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a753
msgid "Subsides en capital et en intérêts"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a150
msgid "Subsides obtenus"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a41670
msgid "Subsides à recevoir"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a411059
msgid "T.V.A Déductible"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a64012
msgid "T.V.A. non déductible"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a451054
msgid "T.V.A. à payer"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a451056
msgid "T.V.A. à payer - Cocontractant"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a451057
msgid "T.V.A. à payer - Import"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a451055
msgid "T.V.A. à payer - Intra-communautaire"
msgstr ""

#. module: l10n_be
#: model:account.tax.group,name:l10n_be.tax_group_tva_0
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-00
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-00-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-00-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-00
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-00-L
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-00-S
msgid "TVA 0%"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-00-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-00-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-00-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-00-CC
msgid "TVA 0% Cocont."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-00-EU
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-00-EU-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-00-EU-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-00-EU
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-00-EU-L
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-00-EU-S
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-00-EU-T
msgid "TVA 0% EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-00-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-00-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-00-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-00-ROW
msgid "TVA 0% Non EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.group,name:l10n_be.tax_group_tva_12
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-12
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-12-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-12-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-12
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-12-L
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-12-S
msgid "TVA 12%"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-12-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-12-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-12-CC
msgid "TVA 12% Cocont."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-12-EU
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-12-EU-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-12-EU-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-12-EU
msgid "TVA 12% EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-12-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-12-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-12-ROW-CC
msgid "TVA 12% Non EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.group,name:l10n_be.tax_group_tva_21
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-21
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-21-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-21-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-21
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-21-L
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-21-S
msgid "TVA 21%"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-21-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-21-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-21-CC
msgid "TVA 21% Cocont."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-21-EU
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-21-EU-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-21-EU-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-21-EU
msgid "TVA 21% EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-21-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-21-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-21-ROW-CC
msgid "TVA 21% Non EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_TVA-21-inclus-dans-prix
msgid "TVA 21% TTC"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-CAR-EXC
msgid "TVA 50% Non Déductible - Frais de voiture (Prix Excl.)"
msgstr ""

#. module: l10n_be
#: model:account.tax.group,name:l10n_be.tax_group_tva_6
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-06
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-06-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-06-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-06
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-06-L
#: model:account.tax.template,description:l10n_be.attn_VAT-OUT-06-S
msgid "TVA 6%"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-06-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-06-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-06-CC
msgid "TVA 6% Cocont."
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-06-EU
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-06-EU-G
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-06-EU-S
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-06-EU
msgid "TVA 6% EU"
msgstr ""

#. module: l10n_be
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V81-06-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V82-06-ROW-CC
#: model:account.tax.template,description:l10n_be.attn_VAT-IN-V83-06-ROW-CC
msgid "TVA 6% Non EU"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a4118
msgid "Taxe d'égalisation due"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_taxes
msgid "Taxes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a64000
msgid "Taxes sur autos et camions"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a2500
msgid "Terrains"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_taxes_deductibles
msgid "V Déductibles"
msgstr ""

#. module: l10n_be
#: model:account.tax.report.line,name:l10n_be.tax_report_title_taxes_soldes
msgid "VI Soldes"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7001
msgid "Ventes dans les pays membres de la C.E.E. (marchandises)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7011
msgid "Ventes dans les pays membres de la C.E.E. (produits finis)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7000
msgid "Ventes en Belgique (marchandises)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7010
msgid "Ventes en Belgique (produits finis)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7002
msgid "Ventes à l'exportation (marchandises)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a7012
msgid "Ventes à l'exportation (produits finis)"
msgstr ""

#. module: l10n_be
#: model:account.account.template,name:l10n_be.a24100
msgid "Voitures"
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
        rslt.append('BE')
        return rslt

    @api.model
    def _prepare_all_journals(self, acc_template_ref, company, journals_dict=None):
        journal_data = super(AccountChartTemplate, self)._prepare_all_journals(
            acc_template_ref, company, journals_dict)
        for journal in journal_data:
            if journal['type'] in ('sale', 'purchase') and company.country_id == self.env.ref('base.be'):
                journal.update({'refund_sequence': True})
        return journal_data

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_chart_template

```

