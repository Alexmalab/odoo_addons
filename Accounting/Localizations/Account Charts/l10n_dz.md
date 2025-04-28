# Odoo Module: l10n_dz

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
    'name': 'Algeria - Accounting',
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the module to manage the accounting chart for Algeria in Odoo.
======================================================================
This module applies to companies based in Algeria.
""",
    'author': 'Osis',
    'depends': ['account', 'l10n_multilang'],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/account_chart_template_post_data.xml',
        'data/account_tax_data.xml',
        'data/account_fiscal_position_template_data.xml',
        'data/account_chart_template_configuration_data.xml',
        'report/account_move_report.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,name,code,user_type_id/id,chart_template_id/id,reconcile
pcg_10111,Capital souscrit non appelé,10111,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_10112,Capital souscrit - appelé non versé,10112,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_10113,"Capital souscrit appelé, versé",10113,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1012,Fonds de dotation,1012,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1013,Fonds d'exploitation,1013,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1018,Autres fonds propres,1018,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1031,Prime d’émission,1031,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1032,Prime de fusion,1032,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1033,Prime d’apports,1033,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1034,Prime de conversion d’obligations en actions,1034,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1035,Bons de souscription d’actions (BSA),1035,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_104,Ecart d’évaluation,104,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1051,Ecart de réévaluation sur immobilisations corporelles,1051,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1059,Ecart de réévaluation rapporté au résultat de l’exercice,1059,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1061,Réserve légale,1061,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_10621,Réserves réglementées : Bénéfice taxé au taux réduit,10621,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_10622,Réserves réglementées : Plus value de cession à réinvestir,10622,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1063,Réserves statutaires ou contractuelles,1063,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1064,Réserves ordinaires,1064,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1065,Réserves Spéciales consécutive à l’octroi d’avantages,1065,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1066,Réserves facultatives,1066,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_1068,Autres réserves,1068,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_107,Écarts d'équivalence,107,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_108,Compte de l'exploitant,108,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_109,Actionnaires: capital souscrit - non appelé,109,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_110,Report à nouveau (bénéfice),110,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1151,Impact d'ajustement positif (plus value),1151,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1159,Impact d'ajustement négatif (moins value),1159,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1171,Impact d'assainissement comptable et financier : positif,1171,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1175,Ajustement résultant de correction d'estimation,1175,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1176,Ajustement résultant de correction d'erreurs comptables,1176,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1179,Impact d'assainissement comptable et financier : négatif,1179,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_118,Acompte sur dividendes,118,account.data_account_type_equity,l10n_dz_pcg_chart_template,False
pcg_119,Report à nouveau (Déficit),119,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_120,Résultat net de l’exercice – (BENEFICE),120,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_121,Marge commerciale,121,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_122,Production de l’exercice,122,account.data_account_type_direct_costs,l10n_dz_pcg_chart_template,False
pcg_123,Valeur Ajoutée d’exploitation,123,account.data_account_type_direct_costs,l10n_dz_pcg_chart_template,False
pcg_124,Excédent (ou insuffisance) brut d’exploitation (EBE) – EBITDA,124,account.data_account_type_direct_costs,l10n_dz_pcg_chart_template,False
pcg_125,Résultat opérationnel,125,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_126,Résultat ordinaire avant impôts,126,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_127,Résultat extraordinaire,127,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_128,Résultat brut de l’exercice,128,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_129,Résultat net de l’exercice –(DEFICIT),129,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_1311,Subventions d’équipements: Financementde l'Etat,1311,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1312,Subventions d’équipements: Financement d'autres organismes,1312,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1313,Transfert gratuit d’immobilisations,1313,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1319,Subventions inscrites au compte de résultat,1319,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1321,Subventions d’investissement,1321,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1322,Subventions pour financement d'activités à long terme,1322,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1329,Autres subventions inscrites au compte de résultat,1329,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_133,Impôts différés actif,133,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_134,Impôts différés passif,134,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1386,Charges différées – Passif non courant,1386,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1387,Produits différés – Passif non courant,1387,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1531,Provisions pour avantages au personnel (IDR–IFC),1531,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1551,Provision pour rappel d'impôts et amendes fiscales,1551,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_156,Provisions pour renouvellement des immobilisations (concession),156,account.data_account_type_fixed_assets,l10n_dz_pcg_chart_template,False
pcg_1581,Provisions pour remise en l’état ou de démantèlement,1581,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1582,Provision pour perte à terminaison (contrats déficitaires),1582,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1583,Provision pour procès et litiges (au-delà de 12 mois),1583,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1584,Provision pour garantie données aux clients (au-delà de 12 mois),1584,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1588,Autres provisions résultant d’obligation légale ou implicite,1588,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1611,Emprunt Principal (Titres),1611,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1618,Intérêts courus et non échus (Titres),1618,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1621,Emprunt Principal (obligataires convertibles),1621,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1628,Intérêts courus et non échus (obligataires convertibles),1628,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1631,Emprunt Principal (Autres emprunts obligataires),1631,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1638,Intérêts courus et non échus (Autres emprunts obligataires),1638,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1641,Emprunts  à Court  Terme,1641,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1642,Emprunts à Moyen Terme,1642,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1643,Emprunts à Long Terme,1643,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1646,Intérêts intercalaires,1646,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1648,Intérêts courus et non échus,1648,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1651,Dépôts reçus,1651,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1655,Cautionnements reçus,1655,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1658,Intérêts courus et non échus,1658,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1671,Financement Principal,1671,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1678,Intérêts courus et non échus,1678,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1681,Emprunt Principal,1681,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1688,Intérêts courus et non échus,1688,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_169,Primes de remboursement des obligations,169,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_171,Dettes rattachées à des participations groupe,171,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_172,Dettes rattachées à des participations hors groupe,172,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_173,Dettes rattachés à des sociétés en participation,173,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_178,Autres dettes rattachés à des participations,178,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1814,Créances inter-Unités (ou établissement),1814,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1815,Dettes inter-Unités (ou établissement),1815,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1816,Charges inter-Unités (ou établissement),1816,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_1817,Produis inter-Unités (ou établissement),1817,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_18841,Créances commerciales,18841,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_18842,Créances financières,18842,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_18851,Dettes commerciales,18851,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_18852,Dettes financières,18852,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_203,Frais de développement immobilisables,203,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2041,Logiciels de traitements informatiques,2041,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2042,Hébergement site Web,2042,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2051,Droit à la propriété industrielle et commerciale,2051,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2058,Autres droits similaires (concession et  franchise),2058,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2071,Ecart d'acquisition positif (goodwill),2071,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2072,Autres écarts d’acquisitions ou de fusions,2072,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2079,Ecart d'acquisition négatif (badwill),2079,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2081,Fonds de commerce,2081,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2082,Droit au bail,2082,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2087,Actif environnemental,2087,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2111,Terrains nus,2111,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21121,Terrains administratifs et commerciaux,21121,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21122,Terrains industriels,21122,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21131,Terrains aménagés en aires de stockage,21131,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21132,Terrains aménagés en aires de stationnement et parking,21132,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2114,Terrains de gisement (carrières),2114,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2121,Aménagement d’espaces verts,2121,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2122,Autres agencements et aménagements de terrains,2122,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21311,Bâtiment Structure,21311,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21312,Bâtiment composant A,21312,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21313,Bâtiment composant B,21313,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21314,Bâtiment composant C,21314,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2132,Bâtiments industriels (à décomposer en cas de besoin),2132,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2133,Bâtiments sociaux (CMS – CANTINE – REFECTOIRE),2133,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21341,Logements du personnel,21341,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21348,Autres immeubles de placement,21348,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2135,Autres constructions,2135,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2136,"Agencements, aménagements et installations des bâtiments",2136,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2137,Ouvrages d’infrastructures : Voies de transport ou d'accès,2137,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2138,Autres travaux d'aménagement sur ouvrages d’infrastructures,2138,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2151,Installations complexes spécialisées,2151,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2153,Installations à caractère spécifique,2153,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2154,Matériel industriel,2154,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2155,Outillage industriel,2155,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2157,Agencements et aménagements du matériel et outillage industriel,2157,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2158,Autres matériels et outillage industriel,2158,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2181,Installations générales agencements aménagements divers,2181,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21821,Matériel de transport : Véhicules Lourds,21821,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21822,"Matériel de transport : Véhicules Utilitaires inférieurs à 3,5 T",21822,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21823,Matériel de transport : Véhicules Légers de tourisme,21823,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21824,Matériels de transport en commun,21824,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21828,Autres matériels roulants,21828,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21831,Matériel de bureau,21831,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21832,Matériel informatique,21832,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21834,"Matériel de communication, de projection et d’insonorisation",21834,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21835,Matériel didactique de formation,21835,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21838,Autres équipements de bureau,21838,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21841,Mobilier de bureau,21841,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21848,Autres mobiliers d'ameublement ou d'accueil,21848,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21851,Mobiliers et équipements ménagers des logements de fonction,21851,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21852,Mobilier et matériel médical de secours,21852,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21853,Mobilier et matériel de cantine et réfectoire,21853,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_21858,Autres mobiliers et matériels divers,21858,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2186,Actifs biologiques,2186,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2187,Emballages récupérables,2187,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_221,Terrains en concession,221,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_222,Agencements et aménagements de terrain en concession,222,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_223,Constructions en concession,223,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_225,Installations techniques en concession,225,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_228,Autres immobilisations corporelles en concession,228,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_229,Droits du concédant,229,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2322,Terrains en cours ,2322,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2323,Constructions en cours,2323,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2325,"Installations techniques, matériels et outillage industriels en cours",2325,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2328,Autres immobilisations corporelles en cours,2328,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_237,Immobilisations incorporelles en cours,237,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_23821,Avances sur acquisitions de logiciels,23821,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_23828,Avances sur acquisitions d'autres immobilisations incorporelles,23828,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_23841,Avances & acomptes sur acquisitions de terrains,23841,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_23842,Avances & acomptes sur constructions,23842,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_23843,Avances sur acquisitions d'installations techniques,23843,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_23844,Avances sur acquisitions de matériels et outillages industriels,23844,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_23845,Avances sur acquisitions autres mobiliers et matériels,23845,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_23848,Avances sur commandes d’autres immobilisations corporelles,23848,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_261,Titres de filiales,261,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_262,Autres titres de participation,262,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_265,Titres de participation évalués par équivalence (entreprises associés),265,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_266,Créances rattachées à des participations groupe,266,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_267,Créances rattachés à des participations hors groupe,267,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_268,Créances rattachés à des sociétés en participation,268,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_269,Versements restant à effectuer sur titres de participation non libérés,269,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_271,Titres immobilisés autres que les titres immobilisés de l'activité de portefeuille,271,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2721,Obligations à terme,2721,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2722,Bons du Trésor,2722,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2723,Dépôts à terme (DAT > 12 mois),2723,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_273,Titres immobilisés de l'activité de portefeuille (TIAP),273,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2741,Prêts participatifs,2741,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2742,Prêts au personnel,2742,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2743,Autres prêts accordés,2743,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2744,Créances sur contrat de location-financement,2744,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2748,Intérêts courus sur prêts et créances de crédit bail,2748,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2751,Dépôts versés,2751,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2752,Cautionnements versés aux fonds de garantie,2752,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2753,Cautions de soumissions,2753,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2754,Cautions de bonne exécution ou de bonne fin,2754,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2755,Cautionnements versés sur loyers (Loyers d'avance),2755,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2756,"Cautionnements versés sur Téléphone, Eau, Gaz et électricité",2756,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2758,Autres cautionnements versés,2758,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_27611,Billet de fonds à recevoir,27611,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2768,Intérêts courus et non échus sur actifs non courants,2768,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_279,Versements restant à effectuer sur titres immobilisés non libérés,279,account.data_account_type_fixed_assets,l10n_dz_pcg_chart_template,False
pcg_2803,Amortissements des frais de développement immobilisables,2803,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28041,Amortissement : Logiciels de traitements informatiques,28041,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28042,Amortissement : Hébergement site Web,28042,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28051,Amortissement du droit à la propriété industrielle et commerciale,28051,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28058,Amortissement des autres droits similaires (concession et franchise),28058,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28081,Amortissements du fonds de commerce (acquis à durée limitée),28081,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28087,Amortissements des actifs environnementaux,28087,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2811,Amortissements des terrains de gisement (carrières...),2811,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28121,Amortissement des aménagements d'espaces verts,28121,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28122,Amortissement des autres agencements et aménagements de terrains,28122,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281311,Amortissement : Bâtiment Structure,281311,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281312,Amortissement : Bâtiments composant A,281312,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281313,Amortissement : Bâtiments composant B,281313,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281314,Amortissement : Bâtiments composant C,281314,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28132,Amortissement des bâtiments industriels (à décomposer),28132,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28133,Amortissements : Bâtiments sociaux ( CMS-CANTINE-REFECTOIRE),28133,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28134,Amortissements des immeubles de placement (au coût amorti),28134,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28135,Amortissements des agencements et aménagements de bâtiments,28135,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28136,Amortissements des ouvrages d’infrastructure : Voies de transport,28136,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28138,Amortissements des travaux d’aménagement : Autres ouvrages,28138,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28151,Amortissement des installations complexes spécialisées,28151,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28153,Amortissement des installations à caractère spécifique,28153,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28154,Amortissement du Matériel industriel,28154,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28155,Amortissement de l'outillage industriel,28155,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28157,Amortis. des agencements et aménagements du matériel & outillage,28157,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28158,Amortissement des Autres matériels et outillage industriel,28158,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28181,"Amort. Installations générales, agencements et aménagements divers",28181,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281821,Amortissement du Matériel de transport : Véhicules Lourds,281821,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281822,Amortissement du Matériel de transport : Véhicules Utilitaires,281822,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281823,Amortissement du Matériel de transport : Véhicules Légers,281823,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281824,Amortissement du Matériel de transport en commun,281824,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281828,Amortissement : Autres matériels roulants,281828,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281831,Amortissement du Matériel de bureau,281831,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281832,Amortissement du Matériel informatique,281832,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281834,"Amort. du matériel de communication, projection et d’insonorisation",281834,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281835,Amortissement du matériel didactique de formation,281835,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281838,Amortissement des Autres équipements de bureau,281838,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281841,Amortissement du Mobilier de bureau,281841,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281848,Amortissement des Autres mobiliers d'ameublement ou d'accueil,281848,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281851,Amortis.: Mobiliers et équipements ménagers des logements de fonction,281851,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281852,Amortissement : Mobilier et matériel médical de secours,281852,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281853,Amortissement : Mobilier et matériel de cantine et réfectoire,281853,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_281858,Amortissement : Autres mobiliers et matériels divers,281858,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28186,Amortissement des Actifs biologiques (évalués au coût amorti),28186,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_28187,Amortissement des Emballages récupérables,28187,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2821,Amortissement : Terrains en concession,2821,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2822,Amortissement : Agencements et aménagements de terrains,2822,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2823,Amortissement : Constructions en concession,2823,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2825,Amortissement : Installations techniques en concession,2825,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2828,Amortissement : Autres immobilisations corporelles en concession,2828,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2903,Pertes de valeur sur frais de développement immobilisables,2903,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2904,Pertes de valeur sur logiciels informatiques et assimilés,2904,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2905,"Pertes  de valeur sur concessions et droits similaires, brevets, licences, marques",2905,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2907,Pertes de valeur sur écart d’acquisition (Goodwill),2907,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_29081,Perte de valeur sur fonds de commerce,29081,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_29082,Perte de valeur sur droit au bail,29082,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_29087,Perte de valeur sur actifs environnementaux,29087,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2911,Perte de valeur des terrains (autres que les terrains de gisement),2911,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2912,Pertes de valeur sur agencements et aménagements de terrain,2912,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2913,Pertes de valeur sur constructions,2913,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2914,Pertes de valeur sur immeubles de placement (Juste valeur),2914,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2915,"Pertes de valeur sur Installations techniques, matériel et outillage industriel",2915,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2916,Pertes de valeur sur actifs biologiques (Juste valeur),2916,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2918,Pertes de valeur sur autres immobilisations corporelles,2918,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_292,Pertes de valeur sur immobilisations mises en concession,292,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2931,Pertes de valeurs sur immobilisations incorporelles en cours,2931,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2932,Pertes de valeurs sur immobilisations corporelles en cours,2932,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2941,Pertes de valeur sur U.G.T (1),2941,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2942,Pertes de valeur sur U.G.T (2),2942,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2961,Pertes de valeur sur titres de filiales,2961,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2962,Pertes de valeur sur autres titres de participation,2962,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2966,Pertes de valeur sur créances rattachées à des participations – groupe,2966,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2967,Pertes de valeur sur créances rattachées à des participations – HG,2967,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2968,Pertes de valeur sur créances rattachées à des SEP ,2968,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2971,Pertes de valeur sur titres de placement,2971,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2972,Pertes de valeurs sur les TIAP,2972,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2981,Pertes de valeur sur les titres participatifs,2981,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2982,Pertes de valeur sur les prêts ordinaires,2982,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2983,Pertes de valeur sur les dépôts et cautionnements versés,2983,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_2988,Pertes de valeur sur autres actifs financiers immobilisés,2988,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_301,Marchandises (ou groupe) A,301,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_302,Marchandises (ou groupe) B,302,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3111,Matière (ou groupe) A,3111,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3112,Matière (ou groupe) B,3112,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3118,Autres matières premières,3118,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3171,Fournitures A,3171,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3172,Fournitures B,3172,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3178,Autres fournitures accessoires,3178,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3211,Matières (ou groupe) C,3211,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3212,Matières (ou groupe) D,3212,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3218,Autres matières consommables,3218,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3221,Carburant et lubrifiant –Véhicules,3221,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3222,Petit outillage et fournitures d'ateliers,3222,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3223,Droguerie et Produits d'entretien,3223,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3224,Pièces de rechange pour équipements et matériels techniques   ,3224,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3225,Pièces de rechange et pneumatique pour matériel de transport,3225,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3226,Quincaillerie générale et fournitures de magasin,3226,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3227,Fournitures de bureau et imprimés,3227,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3228,Autres fournitures consommables,3228,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3261,Emballages perdus,3261,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3262,Emballages récupérables non identifiables,3262,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3263,Emballages à usage mixte,3263,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3311,Produit en cours - P1,3311,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3312,Produit en cours - P2,3312,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3313,Produit en cours - P3,3313,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3351,Travaux en cours – T1,3351,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3352,Travaux en cours – T2,3352,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3353,Travaux en cours – T3,3353,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3411,Etudes en cours – E1,3411,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3412,Etudes en cours – E2,3412,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3413,Etudes en cours – E3,3413,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3451,Prestations de services en cours – S1,3451,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3452,Prestations de services en cours – S2,3452,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3453,Prestations de services en cours – S3,3453,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3511,Produits intermédiaires (ou groupe) – PI 1,3511,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3512,Produits intermédiaires (ou groupe) – PI 2,3512,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3551,Produits finis (ou groupe) – PF 1,3551,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3552,Produits finis (ou groupe) – PF 2,3552,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3553,Produits finis (ou groupe) – PF 3,3553,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3581,Produits résiduels,3581,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3582,Matières de récupération (à recycler),3582,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3583,Déchets et rebuts divers destinés à être cédés ou détruits,3583,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_361,Lot de bord et pièces d’accompagnement,361,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_362,Organes et accessoires démantelés,362,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_365,Pièces de rechanges récupérées sur matériels,365,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_368,Actifs non courants destinés à être cédés (réformes),368,account.data_account_type_non_current_assets,l10n_dz_pcg_chart_template,False
pcg_371,Matières premières en cours de réception,371,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_372,Autres approvisionnements à réceptionner,372,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_373,Stocks en dépôt ou en consignation,373,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_374,Stocks sous douanes,374,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3801,Achats de Marchandises (ou groupe) A,3801,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3802,Achats de Marchandises (ou groupe) B,3802,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38111,Matières (ou groupe) A,38111,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38112,Matières (ou groupe) B,38112,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38118,Autres matières premières,38118,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38171,Achats de Fournitures A,38171,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38172,Achats de Fournitures B,38172,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38178,Achats d'autres fournitures accessoires,38178,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38211,Achat : Matières (ou groupe) C,38211,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38212,Achat : Matières (ou groupe) D,38212,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38218,Achat : Autres matières consommables,38218,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38221,Achat : Carburant et lubrifiant – Véhicules,38221,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38222,Achat : Petit outillage et fournitures d'ateliers,38222,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38223,Achat : Droguerie et Produits d'entretien,38223,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38224,Achat : Pièces de rechange pour équipements & matériels techniques,38224,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38225,Achat : Pièces de rechange et pneumatique pour matériel de transport,38225,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38226,Achat : Quincaillerie générale et fournitures de magasin,38226,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38227,Achat : Fournitures de bureau et imprimés,38227,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38228,Achat : Autres fournitures consommables,38228,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38261,Achat d'Emballages perdus,38261,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38262,Achat d'Emballages récupérables non identifiables,38262,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38263,Achat d'Emballages à usage mixte,38263,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38271,Achat non stocké de carburant,38271,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38272,Achat non stocké d'énergie et de force motrice (ateliers),38272,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38273,"Fournitures non stockées : Eau, Gaz & Electricité (bureaux)",38273,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38274,Fournitures administratives et de bureautique non stockées,38274,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38275,Achats non stockés de fournitures d'atelier et de magasin,38275,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38276,Achat non stocké de Tenues de travail et de sécurité,38276,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_38278,Achat : Autres fournitures consommables non stockées,38278,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3831,Frais Accessoires sur Achats de matières premières et fournitures,3831,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3832,Frais Accessoires sur Achats d'autres approvisionnements,3832,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3841,Achats locaux,3841,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_3842,Achats à l'importation,3842,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_390,Pertes de valeur sur Stocks de marchandises,390,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_391,Pertes de valeur sur Matières premières et fournitures,391,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_392,Pertes de valeur sur Autres approvisionnements,392,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_393,Pertes de valeur sur En cours de production de biens,393,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_394,Pertes de valeur sur En cours de production de services,394,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_395,Pertes de valeur sur stocks de produits,395,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_397,Pertes de valeur sur Stocks à l'extérieur,397,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
dz_pcg_pay,Fournisseurs de stocks : Nationaux – Publics,40131,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_40132,Fournisseurs de stocks : Nationaux – Privés,40132,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_40133,Fournisseurs de stocks : Etrangers,40133,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_40161,Fournisseurs de services : Nationaux – Publics,40161,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_40162,Fournisseurs de services : Nationaux – Privés,40162,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_40163,Fournisseurs de services : Etrangers,40163,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_40171,Fournisseurs : Retenues de garantie – Publics,40171,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_40172,Fournisseurs : Retenues de garantie – Privés,40172,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_40173,Fournisseurs : Retenues de garantie – Etrangers,40173,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_4031,Effets à payer aux fournisseurs de stocks,4031,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_4032,Effets à payer aux fournisseurs de services,4032,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_4041,Fournisseurs d'immobilisations : Nationaux – Publics,4041,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_4042,Fournisseurs d'immobilisations : Nationaux – Privés,4042,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_4043,Fournisseurs d'immobilisations : Etrangers,4043,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_4047,Fournisseurs d'immobilisations - Retenues de garantie,4047,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_405,Fournisseurs d'immobilisations - Effets à payer,405,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_4081,Fournisseurs de stocks nationaux : Factures non parvenues,4081,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_4082,Fournisseurs de services – factures non parvenues,4082,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_4083,Factures à recevoir des Fournisseurs d’immobilisations,4083,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_4086,"Factures à recevoir des Consultants, CAC et Experts",4086,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_4088,Fournisseurs : Intérêts courus à payer,4088,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_40913,Avances et acomptes sur acquisition de Stocks,40913,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_40916,Avances et acomptes sur achats de services,40916,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4094,"Autres avances et acomptes (Consultants, CAC et Experts)",4094,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4096,Fournisseurs : Créances pour emballages et matériels à rendre,4096,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4097,Fournisseurs à soldes débiteurs (à détailler par code Tiers),4097,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4098,"Rabais, remises, ristournes à obtenir et autres avoirs non encore reçus",4098,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4111,Clients : Ventes de biens et services,4111,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4114,Clients : Autres créances diverses,4114,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4117,Clients : Retenues de garantie,4117,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
dz_pcg_recv_pos,Clients : effets à recevoir (PoS),412,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
dz_pcg_recv,Clients : effets à recevoir,413,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4161,Clients douteux : secteur public,4161,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4162,Clients douteux : secteur privé,4162,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4171,Créances sur travaux en cours (à l'avancement),4171,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4172,Créances sur prestations en cours (à l'avancement),4172,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4181,Clients - Factures à établir – Secteur Public,4181,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4182,Clients : Factures à établir – Secteur Privé,4182,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4188,Clients : intérêts moratoires à facturer,4188,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4191,Clients - Avances et acomptes reçus sur commandes,4191,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4196,Clients - Dettes pour emballages et matériels consignés ,4196,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4197,Clients à solde créditeur (à détailler par code Tiers),4197,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4198,"Rabais, remises, ristournes à accorder et autres avoirs à établir",4198,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4211,Salaires et appointements à payer,4211,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4212,Présalaires à payer,4212,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4213,Soldes de tout compte restant dus,4213,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4221,Contributions annuelles à payer (3%),4221,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4229,Autres avances pour le compte des œuvres sociales,4229,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4231,Quote-part de bénéfice attribuée au personnel,4231,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4251,Avances sur salaires,4251,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4256,Avances sur frais de mission,4256,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4258,Autres avances exceptionnelles,4258,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4261,Frais médicaux à rembourser,4261,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4268,Autres dépôts et prestations reçus au profit du personnel,4268,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4271,Saisie-arrêt et cession de rémunération (sur décision de Justice),4271,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4272,Retenue sur prêts sociaux,4272,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4273,Retenue pour fonds de solidarité nationale et internationale,4273,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4274,Retenue sur loyers logements du personnel,4274,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4275,Salaires bloqués,4275,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4278,Autres retenues sur salaires,4278,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4281,Dettes provisionnées pour congés à payer,4281,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4286,Frais de mission à payer,4286,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4288,Personnel : Autres charges à payer,4288,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4289,Personnel : Produits à recevoir,4289,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4311,Retenue sécurité sociale (part des employés),4311,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4312,Cotisations FNPOS,4312,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4313,CNAS à payer (part employeur),4313,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4314,Cotisations CASNOS à payer,4314,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4321,Cotisations Mutuelle à payer,4321,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4322,Cotisations OPREBAT à payer,4322,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4323,Cotisations CACOBATPH à payer,4323,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4326,Assurance groupe (Quote-part employeur) à payer,4326,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4327,Assurance groupe (Quote-part des travailleurs) à payer,4327,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4328,Autres organismes sociaux,4328,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4386,Organismes sociaux : Charges sociales sur congés à payer,4386,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4388,Organismes sociaux : Autres charges à payer,4388,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4389,Organismes sociaux : Produits à recevoir,4389,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4411,Subventions d'investissement à recevoir,4411,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4412,Subventions d’exploitation à recevoir,4412,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4413,Subventions d’équilibre à recevoir,4413,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4414,Subventions certification ISO à recevoir,4414,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4418,Autres subventions et aides publiques à recevoir,4418,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4419,Avances sur subventions,4419,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4421,IRG retenu sur salaires (barème),4421,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4422,IRG – Libératoire sur rappels de salaires et participation,4422,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4423,IRG – Libératoire sur bénéfices distribués,4423,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4424,IRG – Libératoire sur revenus mobiliers,4424,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4425,IRG – Libératoire sur Jetons de présence et tantièmes,4425,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4426,IRG – Libératoire retenu à la source aux consultants,4426,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4428,Autres impôts retenus à la source,4428,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4431,Créances sur l'Etat – Indemnisations,4431,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4432,Avance pour compte ANEM – Emplois aidés (Quote part de l'Etat),4432,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4438,Créances ou dettes résultant du changement de la réglementation fiscale,4438,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4441,Acomptes provisionnels sur I.B.S,4441,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4449,Impôts sur les bénéfices – I.B.S à payer,4449,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_44551,TVA à décaisser (à payer sur G.50),44551,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_44552,Obligations cautionnées (au profit des Douanes),44552,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_44562,TVA récupérable sur immobilisations,44562,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_44563,TVA transférée par d'autres entreprises,44563,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_44566,TVA récupérable sur achats de stocks et de services,44566,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,False
pcg_44568,Crédit de TVA à reporter (Précompte),44568,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_44571,TVA collectée sur Ventes – (livraisons juridiques ou matérielles),44571,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_44572,TVA collectée sur les encaissements,44572,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_44575,Droits de timbre perçus au profit du Trésor,44575,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_44583,Remboursement de TVA demandé,44583,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_44584,TVA récupérée d’avance ou en attente de régularisation,44584,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_44586,TVA sur factures d’achats non parvenues,44586,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_44587,TVA sur factures de ventes ou de prestations à établir,44587,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4461,Fonds de solidarité internationale,4461,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4462,Subventions d’organismes étrangers,4462,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4471,Taxe sur l’activité professionnelle – TAP à payer (sur G.50),4471,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4472,Taxe sur l’apprentissage et la formation professionnelle,4472,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4473,Taxe sur l’environnement – (Écotaxe),4473,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4474,Droits d’enregistrement à payer,4474,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4475,Taxe foncière à payer,4475,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4476,Taxes spéciales (ou spécifiques) à payer,4476,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4478,"Autres droits, impôts et taxes à payer",4478,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4482,Etat – charges à payer : Charges fiscales sur congés à payer,4482,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4486,Rappels d'impôts à payer (hors IBS),4486,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4487,Etat – produits à recevoir : Trop versés sur impôts,4487,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4488,Etat – produits à recevoir : Dégrèvement d’impôts,4488,account.data_account_type_current_assets,l10n_dz_pcg_chart_template,True
pcg_4511,Avances de fonds,4511,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4518,Autres opérations,4518,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4551,Associés – Comptes courants : Principal,4551,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_4556,Associés – Comptes courants : Intérêts courus,4556,account.data_account_type_payable,l10n_dz_pcg_chart_template,True
pcg_45611,Associés : Apports en nature,45611,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_45615,Associés : Apports en numéraires,45615,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4562,"Associés : Capital souscrit, appelé et non versé (apports à libérer)",4562,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4563,Associés : Versements reçus sur augmentation de Capital,4563,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4564,Associés : Versements anticipés,4564,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4566,Actionnaires défaillants,4566,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4567,Associés - Capital à rembourser,4567,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4571,Coupons d’action,4571,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4572,Dividendes mis en paiement,4572,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4577,Tantièmes à payer aux administrateurs et gérants,4577,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4586,Opérations faites en commun : Charges à payer,4586,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4587,Opérations faites en commun : Produits à recevoir,4587,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4588,Intérêts courus,4588,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4621,Fonds à recevoir sur cessions d’Actifs réformés,4621,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4622,Débiteurs sur cessions d’immobilisations,4622,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_464,Dettes sur acquisitions de valeurs mobilières de placement et instruments financiers dérivés,464,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_465,Créances sur cessions de valeurs mobilières de placement et instruments financiers dérivés,465,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4671,"Fonds en dépôt chez les officiers ministériels (notaire, commis. priseurs)",4671,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_4672,Avances sur frais divers,4672,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_4673,Achats pour compte de tiers,4673,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_4674,Autres avances pour compte,4674,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_4675,Jetons de présence à payer aux Administrateurs,4675,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_4676,Débiteurs divers (à détailler),4676,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_4679,Créditeurs de frais divers (à détailler),4679,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_46861,Amendes et pénalités à payer,46861,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_46862,Autres charges opérationnelles à payer,46862,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_46871,Remboursement indemnisations d’assurances à percevoir,46871,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_46872,Autres produits opérationnels à recevoir,46872,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_476,Dépenses en attente d’imputation,476,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_477,Recettes en attente d’imputations,477,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_478,Autres opérations à régulariser,478,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,False
pcg_4811,Provisions pour procès et litiges (moins de 12 mois),4811,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4812,Provisions pour garanties données aux clients (moins de 12 mois),4812,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4815,Provisions pour amendes et pénalités,4815,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4816,Provisions pour pertes de change,4816,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4818,Autres provisions : Passifs courants,4818,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4861,Loyers payés d’avance (autre que cautionnement),4861,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4862,Primes d’assurances payées d’avance,4862,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4863,Abonnements payés d’avance,4863,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4866,Charges financières différées,4866,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4868,Autres charges constatées d’avance,4868,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4871,Loyers perçus d’avance (immeubles de placement),4871,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4877,Subventions à étaler,4877,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4878,Autres produits constatés d’avance,4878,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4911,Pertes de valeurs sur comptes de clients,4911,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4912,Pertes de valeurs sur retenues de garanties,4912,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4951,Pertes de valeur sur compte du groupe,4951,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4955,Pertes de valeur sur comptes courants des associés,4955,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4958,Pertes de valeur sur opérations faites en commun (SEP – GIE),4958,account.data_account_type_current_liabilities,l10n_dz_pcg_chart_template,True
pcg_4962,Pertes de valeur sur créances de cessions d'immobilisations,4962,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4965,Pertes de valeur sur créances de cessions de VMP,4965,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_4967,Pertes de valeur sur autres comptes débiteurs,4967,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_498,Pertes de valeur sur autres comptes de tiers,498,account.data_account_type_receivable,l10n_dz_pcg_chart_template,True
pcg_501,Parts dans entreprises liées,501,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_502,Actions propres,502,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5031,Actions détenues en vue de la revente,5031,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5061,Obligations à moins de 12 mois,5061,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5062,Bons du Trésor à court terme,5062,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5063,Bons de caisse à court terme,5063,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5081,Autres valeurs mobilières,5081,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5082,Bons de souscription et instruments financiers composés,5082,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5083,Certificat d'investissement (Art. 715-61 bis du C.Com),5083,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5088,Intérêts courus,5088,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_509,Versements restant à effectuer sur valeurs mobilières de placement non libérées,509,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5111,Chèques à encaisser,5111,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5112,Traites remises à l’escompte,5112,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5113,Effets à l'encaissement,5113,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5118,Autres valeurs à l’encaissement,5118,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5121,Comptes bancaires : Dinars,5121,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5122,Comptes bancaires : Devises,5122,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5151,Caisses du Trésor Public,5151,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5152,Caisses des établissements publics,5152,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5171,Banque postale (C.C.P),5171,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5172,Sociétés de leasing (crédit-bail),5172,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5173,Sociétés de factoring (affacturage),5173,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5176,Société de bourse,5176,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5178,Autres organismes et établissements financiers,5178,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5186,Intérêts courus à payer (agios à payer),5186,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5187,Intérêts courus à recevoir (intérêts des dépôts créditeurs à recevoir),5187,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5191,Facilité de caisse,5191,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5192,Découvert bancaire,5192,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5193,Autres avances bancairesr,5193,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5198,Intérêts courus sur concours bancaires courants,5198,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_521,Instruments financiers dérivés : Actifs,521,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_529,Instruments financiers dérivés : Passifs,529,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5311,Caisse principale en Dinars,5311,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5314,Caisse principale en Devises,5314,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5321,Caisse auxiliaire : Succursale (ou Unité) A,5321,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5322,Caisse auxiliaire : Succursale (ou Unité) B,5322,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_541,Régies d'avances,541,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_542,Accréditifs,542,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5811,Virements interbancaires,5811,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5812,Versements d'espèces en banques,5812,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5813,Alimentation caisses,5813,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5886,Achats au comptant,5886,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5887,Ventes au comptant,5887,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_5888,Autres transferts en espèces,5888,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_591,Pertes de valeur sur valeurs en banque et Etablissements financiers,591,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_594,Pertes de valeurs sur régies d'avances et accréditifs,594,account.data_account_type_liquidity,l10n_dz_pcg_chart_template,False
pcg_6001,Marchandises (ou groupe) A,6001,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6002,Marchandises (ou groupe) B,6002,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60111,Matières (ou groupe) A,60111,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60112,Matières (ou groupe) B,60112,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60118,Autres matières premières consommées,60118,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60171,Fournitures A,60171,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60172,Fournitures B,60172,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60178,Autres fournitures accessoires consommées,60178,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60211,Matières (ou groupe) C,60211,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60212,Matières (ou groupe) D,60212,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60218,Autres matières consommées,60218,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60221,Carburant et lubrifiant – Véhicules,60221,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60222,Petit outillage et fournitures d'ateliers,60222,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60223,Droguerie et Produits d'entretien,60223,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60224,Pièces de rechange pour équipements et matériels techniques,60224,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60225,Pièces de rechange et pneumatique pour matériel de transport,60225,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60226,Quincaillerie générale et fournitures de magasin,60226,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60227,Fournitures de bureau et imprimés,60227,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60228,Autres fournitures consommées,60228,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60261,Emballages perdus consommés,60261,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60262,Emballages récupérables non identifiables consommés,60262,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_60263,Emballages à usage mixte consommés,60263,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6031,Variations de stock de matières premières et Fournitures,6031,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6032,Variations de stock des autres approvisionnements,6032,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6041,Achats d'études,6041,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6042,Achats de prestations de services,6042,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6051,Achats d’immobilisations de faible valeur : Petit matériel et outillage,6051,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6052,Achats d’immobilisations de faible valeur : Mobilier,6052,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6053,Achats d’immobilisations de faible valeur : Matériel informatique,6053,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6054,Achats d’immobilisations de faible valeur : Matériel de communicat,6054,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6058,"Autres achats de matériels, d'équipements et de travaux",6058,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6071,Achat non stocké de carburant,6071,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6072,Achat non stocké d'énergie et de force motrice (ateliers),6072,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6073,"Fournitures non stockées : Eau, Gaz et Electricité (bureaux)",6073,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6074,Fournitures administratives et de bureautique non stockées,6074,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6075,Achats non stockés de fournitures d'atelier et de magasin,6075,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6076,Achat non stocké de Tenues de travail et de sécurité,6076,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6078,Achat : Autres fournitures consommables non stockées,6078,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_608,Frais accessoires d’achat (inventaire intermittent),608,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6091,R.R.R Obtenus sur achats de matières premières,6091,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6092,R.R.R Obtenus sur achats d’autres approvisionnements,6092,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6094,R.R.R Obtenus sur achats sur études et prestations,6094,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6095,R.R.R Obtenus sur achats de petits matériels et équipements,6095,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6111,Sous traitance de spécialité,6111,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6112,Sous traitance de capacité,6112,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6113,Sous traitance de marché,6113,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_612,(Disponible),612,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_61311,Locations immobilières : Redevance de crédit bail immobilier,61311,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_61312,Locations mobilières : Redevance de crédit bail mobilier,61312,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6137,Malis sur emballages rendus,6137,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6141,Charges locatives d’immeubles,6141,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6142,Charges de copropriété logements de fonction,6142,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6143,Loyers garages et parking de stationnement de véhicules,6143,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6148,Autres charges locatives,6148,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6151,Entretien et réparation sur biens immobiliers,6151,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6152,Entretien et réparation sur biens mobiliers,6152,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6156,Maintenance,6156,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6161,Assurances multirisques professionnels,6161,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6162,Assurances transport,6162,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6163,Assurance responsabilité civile,6163,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6164,Assurance catastrophe naturelle,6164,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6165,Assurance couverture risque de Vie,6165,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6168,Autres assurances,6168,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6171,Assistance technique,6171,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6178,Autres frais d’études et de recherches,6178,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6181,Documentation générale,6181,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6182,"Abonnements aux Journaux, revues et JO/BOAL/BOMOP ...etc.",6182,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6183,"Documentation technique (plans, schémas, maquettes, etc.)",6183,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6184,Frais de reprographie,6184,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6185,Frais de soumissions et de marchés,6185,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6186,Frais de conseils et assemblées,6186,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6187,"Frais de colloques, séminaires et conférences",6187,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_619,"Rabais, remises et ristournes obtenus sur services extérieurs",619,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6211,Personnel intérimaire,6211,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6212,Charges de sécurité et de gardiennage,6212,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6213,Charges de nettoyage et travaux d’insalubrité,6213,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6216,Personnel détaché ou prêté à l'entreprise,6216,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6222,Commissions et courtages sur ventes,6222,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6223,Rémunération de consultants,6223,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6225,Rémunérations d'affacturage (factoring),6225,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6226,Honoraires (autres que ceux incorporés dans les coûts),6226,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6227,Frais d'actes et de contentieux,6227,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6228,Autres rémunérations divers (non incorporables aux coûts),6228,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6231,Annonces et insertions,6231,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6232,Échantillons,6232,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6233,"Foires, expositions et festivités",6233,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6234,"Parrainage, Sponsoring et mécénat",6234,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6235,Cadeau à la clientèle,6235,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6236,Catalogues et imprimés,6236,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6237,Publications Autres publicités,6237,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6238,Autres frais de relations publiques,6238,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6241,Transport sur ventes,6241,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6242,Transport collectif du personnel,6242,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6243,Transport entre établissements ou chantiers,6243,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6244,Transports administratifs,6244,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6248,Autres transports divers,6248,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6251,Voyages et déplacements à l’étranger,6251,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6252,Frais de déménagement,6252,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_62551,Missions : intérieur du pays - Frais de voyage,62551,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_62552,Missions : intérieur du pays - Frais de séjour,62552,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_62571,Réceptions internes,62571,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_62572,Réceptions externes,62572,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_62573,Hébergement,62573,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6261,Affranchissement courrier,6261,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6262,Abonnement boite postale,6262,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6263,Abonnements et communications téléphonie mobile,6263,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6264,Abonnement internet,6264,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6265,Frais de télécommunication (fixe et fax),6265,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6271,Frais sur effets et sur titres,6271,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_62721,Commissions et frais sur émission d’emprunts,62721,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_62722,Commissions sur ouverture de crédit,62722,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_62723,Commissions sur aval et cautions,62723,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6276,Location de coffres,6276,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6278,Autres frais et commissions sur prestations de services,6278,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6281,Frais de recrutement de personnel,6281,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6282,Frais de formation externe du personnel,6282,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6283,Cotisations professionnelles,6283,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6284,Droits de stationnement parking privé,6284,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6288,Autres cotisations et dons courants divers,6288,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_629,"Rabais, remises et ristournes obtenus sur autres services extérieurs",629,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63111,"Traitements, salaires et appointements",63111,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63112,Présalaires et indemnités de stage et d’apprentissage,63112,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63113,Heures supplémentaires et IFSP,63113,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63114,Congés payés,63114,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63121,Prime de responsabilité,63121,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63122,Prime de panier,63122,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63123,Primes d’encouragement et de rendement individuel (PRI-PRC),63123,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63124,Primes sur résultats au profit des cadres dirigeants,63124,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63125,Primes de véhicule,63125,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63126,Prime de caisse et de bilan,63126,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63127,Prime de sujétion,63127,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63128,Primes spéciales,63128,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63131,Indemnité d'Expérience professionnelle (I.E.P),63131,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63132,Indemnité de nuisance,63132,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63133,Indemnité de travail posté,63133,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63134,Indemnités de fin de carrière et de départ à la retraite (IDR – IFC),63134,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63135,Indemnités kilométriques et de transport,63135,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63138,Autres Indemnités liées au salaire,63138,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63141,Bonifications,63141,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63142,"Supplément familial : Prime de scolarité, IPSU, ICAF",63142,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_63148,Autres indemnités et avantages divers,63148,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6315,Avantages et prestations en nature,6315,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_634,Rémunération de l'exploitant,634,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6351,Cotisations de sécurité sociale – CNAS,6351,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6352,Cotisations au FNPOS,6352,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6353,Cotisations à la retraite – CNR,6353,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6354,Cotisations aux mutuelles,6354,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6355,Cotisations à la CACOBATPH,6355,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6361,Cotisation annuelle à la CASNOS,6361,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6368,Autres cotisations,6368,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6374,Versements aux œuvres sociales,6374,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6375,Autres contributions sociales,6375,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6381,Médecine du travail et pharmacie,6381,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6382,Frais de formation et de perfectionnement du personnel (en interne),6382,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6383,Frais d’activités socioculturelles et sportives,6383,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6384,Avantages en nature accordés au personnel,6384,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6385,Médailles du mérite et Jubilé,6385,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6387,"Intéressement, primes et avantages liés aux résultats",6387,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6388,Autres charges de personnel,6388,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6412,Taxe d’apprentissage,6412,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6413,Taxe sur la formation professionnelle,6413,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6418,Autres taxes sur les salaires,6418,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6421,Taxe sur l’activité professionnelle – T.A.P. (sur les débits),6421,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6422,Taxe sur l’activité professionnelle – T.A.P. (sur encaissements),6422,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6451,Taxe foncière sur les propriétés bâties et non bâties,6451,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6452,Taxes spéciales et droits d'accises,6452,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6453,Taxe sur l’environnement (écotaxe),6453,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6454,Taxes sur les véhicules (Vignettes),6454,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6455,Droits de stationnement parking communal,6455,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6456,Droits d’enregistrement sur actes et marchés,6456,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6457,Droits de timbre,6457,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6458,"Autres droits, impôts et taxes divers",6458,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_65111,Redevances de concession de services publics,65111,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_65112,Redevances d'exploitation de carrières et gisements,65112,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_65118,Autres redevances de gestion,65118,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6512,Redevances pour brevets et licences de fabrication,6512,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6516,Redevances pour exploitation de logiciels et valeurs similaires,6516,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6521,Moins value sur cession matériel et outillage,6521,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6522,Moins value sur cession matériel de transport,6522,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6523,Moins value sur cession mobilier et matériel de bureau,6523,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6528,Moins value sur cession autres équipements et matériels,6528,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6531,Jetons de présence aux administrateurs,6531,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6541,Pertes sur créances de l'exercice,6541,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6542,Pertes sur créances des exercices antérieurs,6542,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6551,Quote-part de bénéfice transféré dans le cadre d'un GIE ou d'une SEP,6551,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6552,Quote-part de perte supportée dans le cadre d'un GIE ou d'une SEP,6552,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6562,"Amendes et pénalités (pénales, fiscales et parafiscales)",6562,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6563,Subventions accordées,6563,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_65661,Dons au profit des associations caritatives agréées,65661,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_65662,Frais de solidarité nationale et internationale,65662,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_65663,Autres dons et libéralités,65663,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6571,Ecart de stocks (négatif) de marchandises,6571,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6572,"Ecart de stocks (négatif) de matières, fournitures et autres approvisionnements",6572,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6575,Rappel d’impôts (autre qu’IBS),6575,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6578,Autres charges exceptionnelles,6578,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6581,Charges sur exercices antérieurs (en cours d'exercice),6581,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6588,Autres charges diverses de gestion courante,6588,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6611,Intérêts des emprunts,6611,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6612,Intérêts des dettes,6612,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6616,"Intérêts bancaires et sur opérations de financement (escompte, ...)",6616,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6618,Intérêts des autres dettes,6618,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_664,Pertes sur créances liées à des participations,664,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6651,Mali provenant du rachat des actions propres,6651,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_666,Pertes de change,666,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_667,Pertes nettes sur cessions d’actifs financiers,667,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_668,Autres charges financières,668,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6811,DAP-PDV : Immobilisations incorporelles,6811,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6812,DAP-PDV : Immobilisations corporelles,6812,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_682,"Dotations aux amortissements, provisions et PDV des biens mis en concession",682,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6853,Dotations pour pertes de valeurs sur stocks,6853,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6854,Dotations pour pertes de valeurs sur créances,6854,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6855,Dotation annuelle pour Indemnité de départ à la retraite et IFC,6855,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6856,Dotations aux provisions pour pertes et charges – PNC,6856,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6858,Dotations aux provisions – passifs courants,6858,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6861,Dotations aux amortissements des primes de remboursement des obligations,6861,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_6862,Pertes de valeur des immobilisations financières,6862,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_691,Participation des travailleurs aux bénéfices,691,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_692,Imposition différée actif – Produits,692,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_693,Imposition différée passif – Charges,693,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_695,Impôts sur les bénéfices basés sur le résultat des activités ordinaires,695,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_698,Autres impôts sur les résultats,698,account.data_account_type_expenses,l10n_dz_pcg_chart_template,False
pcg_7001,Marchandises (ou groupe) A,7001,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7002,Marchandises (ou groupe) B,7002,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7011,Ventes de Produit fini (ou groupe) – PF1,7011,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7012,Ventes de Produit fini (ou groupe) – PF2,7012,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7013,Ventes de Produit fini (ou groupe) – PF3,7013,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7021,Ventes de Produits intermédiaires (ou groupe) – PI-1,7021,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7022,Ventes de Produits intermédiaires (ou groupe) – PI-2,7022,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7028,Ventes de sous produits (PR et accessoires fabriqués),7028,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7031,Ventes de déchets,7031,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7032,Ventes de rebuts,7032,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7033,Ventes de matières de récupération recyclables,7033,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7041,Travaux hydrauliques,7041,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_70421,Routes et voies d'accès,70421,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_70422,Ouvrages d'art,70422,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_70431,Travaux de réalisation en T.C.E,70431,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_70432,Travaux de réalisation en Génie civil et Gros Œuvres,70432,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_70433,Travaux de réalisation en C.E.S,70433,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7044,Travaux de restauration de sites et monuments,7044,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7045,Travaux de VRD et d'assainissement,7045,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7046,Travaux d'entretien et de nettoyage,7046,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7047,Travaux forestiers,7047,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7048,Autres travaux de réalisation,7048,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7051,Etudes : E1,7051,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7058,Autres études techniques,7058,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7061,Prestations de services fournis : S1,7061,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7068,Autres Prestations de services,7068,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_707,(disponible),707,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7081,Cession de matières premières,7081,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7082,Cession d’autres approvisionnements,7082,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7086,Bonis sur reprise d’emballages consignés,7086,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7088,Autres produits des activités annexes,7088,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7092,Réduction sur ventes – Hors factures,7092,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7093,Réduction sur prestations – Hors factures,7093,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_72331,Produits en cours,72331,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_72335,Travaux en cours,72335,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_72341,Etudes en cours,72341,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_72342,Prestations de services en cours,72342,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7241,Produits intermédiaires,7241,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7245,Produits finis,7245,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7248,Produits résiduels,7248,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_729,Coût de la sous activité (différence d'imputation coût du chômage),729,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_731,Production immobilisée d'actifs incorporels (logiciels),731,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_732,Production immobilisée d'actifs corporels,732,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_741,Subvention d'équilibre,741,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_748,Autres subventions d'exploitation,748,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_751,"Redevances pour concessions, brevets, licences, logiciels et valeurs similaires",751,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7521,Plus value sur cession d’éléments d’actifs mis à la réforme,7521,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7528,Plus value sur autres éléments d’actifs corporels cédés,7528,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7531,Jetons de présence perçus,7531,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_754,Quotes-parts de subventions d’investissement virées au résultat de l’exercice,754,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7551,Quote-part de perte transférée (dans les SEP et GIE),7551,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7552,Quote-part de bénéfice attribué (dans les SEP et GIE),7552,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_756,Rentrées sur créances amorties,756,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7571,Ecart de stocks (positif) de marchandises,7571,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7572,"Ecart de stocks (positif) de matières, fournitures et autres approvisionnements",7572,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7575,Dégrèvements d’impôts (autre qu’IBS),7575,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7578,Autres produits exceptionnels,7578,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7581,Lots de bord et pièces de rechange gratuites,7581,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7582,Remboursements sinistres et dégâts assurés,7582,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7583,Revenus des immeubles de placement,7583,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7586,Produits des exercices antérieurs (en cours d'exercice),7586,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7588,Autres produits divers de gestion courante,7588,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_761,Produits de participations,761,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_762,Revenus des actifs financiers,762,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_763,Revenus de créances,763,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_765,Ecart d’évaluation sur actifs financiers – Plus-values,765,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_766,Gains de change,766,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_767,Profits nets sur cessions d’actifs financiers,767,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_768,Autres produits financiers,768,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7811,Reprises sur amortissements : Immobilisations incorporelles,7811,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7812,Reprises sur perte de valeur : Immobilisations incorporelles,7812,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7813,Reprises sur amortissements : Immobilisations corporelles,7813,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7814,Reprises sur perte de valeur : Immobilisations corporelles,7814,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7815,Reprises sur écarts de réévaluation : Immobilisations corporelles,7815,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7817,Reprises sur plus values à réinvestir,7817,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7853,Reprise sur pertes de valeurs – Stocks et encours,7853,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7854,Reprise sur pertes de valeurs – Créances,7854,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_7858,Reprise sur provisions pour pertes et charges – PNC,7858,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False
pcg_786,Reprises financières sur pertes de valeur et provisions,786,account.data_account_type_revenue,l10n_dz_pcg_chart_template,False

```

## File: data\account_chart_template_configuration_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_dz.l10n_dz_pcg_chart_template')]"/>
    </function>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="l10n_dz_pcg_chart_template" model="account.chart.template">
        <field name="name">Plan Comptable Général (Algerie)</field>
        <field name="currency_id" ref="base.DZD"/>
        <field name="code_digits" eval="6"/>
        <field name="bank_account_code_prefix">512</field>
        <field name="cash_account_code_prefix">53</field>
        <field name="transfer_account_code_prefix">58</field>
        <field name="country_id" ref="base.dz"/>
    </record>

</odoo>

```

## File: data\account_chart_template_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_dz_pcg_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="dz_pcg_recv"/>
        <field name="default_pos_receivable_account_id" ref="dz_pcg_recv_pos" />
        <field name="property_account_payable_id" ref="dz_pcg_pay"/>
        <field name="property_account_expense_categ_id" ref="pcg_6001"/>
        <field name="property_account_income_categ_id" ref="pcg_7001"/>
        <field name="income_currency_exchange_account_id" ref="pcg_766"/>
        <field name="expense_currency_exchange_account_id" ref="pcg_666"/>
    </record>
</odoo>

```

## File: data\account_fiscal_position_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="fiscal_position_template_national" model="account.fiscal.position.template">
        <field name="sequence">1</field>
        <field name="name">Régime National</field>
        <field name="chart_template_id" ref="l10n_dz_pcg_chart_template"/>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="country_id" ref="base.dz"/>
    </record>

    <record id="fiscal_position_template_exo" model="account.fiscal.position.template">
        <field name="name">EXO</field>
        <field name="chart_template_id" ref="l10n_dz_pcg_chart_template"/>
        <field name="note">Exo de TVA</field>
    </record>

    <record id="fiscal_position_template_import_export" model="account.fiscal.position.template">
        <field name="name">Import/Export</field>
        <field name="chart_template_id" ref="l10n_dz_pcg_chart_template"/>
        <field name="note">Import Export</field>
    </record>

    <!-- Fiscal Position Tax Templates -->
    <!-- ventes -->
    <!-- 19% -->
    <record id="fp_tax_ve_template_exo_normale19" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_exo" />
        <field name="tax_src_id" ref="tva_normale19" />
        <field name="tax_dest_id" ref="tva_0" />
    </record>
    <!-- 09% -->
    <record id="fp_tax_ve_template_exo_specifique9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_exo" />
        <field name="tax_src_id" ref="tva_specifique9" />
        <field name="tax_dest_id" ref="tva_0" />
    </record>

    <!-- Achat -->
    <!-- 19% -->
    <record id="fp_tax_ac_template_exo_normale19" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_exo" />
        <field name="tax_src_id" ref="tva_acq_normale19" />
        <field name="tax_dest_id" ref="tva_exo_0" />
    </record>
    <!-- 09% -->
    <record id="fp_tax_ac_template_exo_specifique9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_exo" />
        <field name="tax_src_id" ref="tva_acq_specifique9" />
        <field name="tax_dest_id" ref="tva_exo_0" />
    </record>

    <!-- Import/Export -->
    <!-- ventes -->
    <!-- 19% -->
    <record id="fp_tax_ve_template_imp_exp_normale19" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_import_export" />
        <field name="tax_src_id" ref="tva_normale19" />
        <field name="tax_dest_id" ref="tva_export_0" />
    </record>
    <!-- 09% -->
    <record id="fp_tax_ve_template_imp_exp_specifique9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_import_export" />
        <field name="tax_src_id" ref="tva_specifique9" />
        <field name="tax_dest_id" ref="tva_export_0" />
    </record>

    <!-- Achat -->
    <!-- 19% -->
    <record id="fp_tax_ac_template_imp_exp_normale19" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_import_export" />
        <field name="tax_src_id" ref="tva_acq_normale19" />
        <field name="tax_dest_id" ref="tva_import_0" />
    </record>
    <!-- 09% -->
    <record id="fp_tax_ac_template_imp_exp_specifique9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_import_export" />
        <field name="tax_src_id" ref="tva_acq_specifique9" />
        <field name="tax_dest_id" ref="tva_import_0" />
    </record>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tva_acq_normale19" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_dz_pcg_chart_template"/>
        <field name="name">TVA (achat) 19,0%</field>
        <field name="description">ACH-19.0</field>
        <field name="amount" eval="19.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="9"/>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_44566'),
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
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record id="tva_normale19" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_dz_pcg_chart_template"/>
        <field name="name">TVA (vente) 19,0%</field>
        <field name="description">19.0</field>
        <field name="amount" eval="19.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="9"/>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_44571'),
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
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record id="tva_acq_specifique9" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_dz_pcg_chart_template"/>
        <field name="name">TVA (achat) 9,0%</field>
        <field name="description">ACH-9.0</field>
        <field name="amount" eval="9.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_44566'),
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
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record id="tva_specifique9" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_dz_pcg_chart_template"/>
        <field name="name">TVA (vente) 9,0%</field>
        <field name="description">9.0</field>
        <field name="amount" eval="9.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_44571'),
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
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record id="tva_imm_normale19" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_dz_pcg_chart_template"/>
        <field name="name">TVA immobilisation (achat) 19,0%</field>
        <field name="description">IMMO-19.0</field>
        <field name="amount" eval="19.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_44562'),
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
                'account_id': ref('pcg_44562'),
            }),
        ]"/>
    </record>

    <record id="tva_imm_specifique9" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_dz_pcg_chart_template"/>
        <field name="name">TVA immobilisation (achat) 9,0%</field>
        <field name="description">IMMO-9.0</field>
        <field name="amount" eval="9.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0, 0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_44562'),
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
                'account_id': ref('pcg_44562'),
            }),
        ]"/>
    </record>

    <record id="tva_exo_0" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_dz_pcg_chart_template"/>
        <field name="name">TVA 0% EXO (achat)</field>
        <field name="description">ACHAT-0</field>
        <field name="amount" eval="0.00"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
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

    <record id="tva_purchase_0" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_dz_pcg_chart_template"/>
        <field name="name">TVA 0% (achat)</field>
        <field name="description">ACHAT-0</field>
        <field name="amount" eval="0.00"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
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

    <record id="tva_0" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_dz_pcg_chart_template"/>
        <field name="name">TVA 0% (vente)</field>
        <field name="description">EXO-0</field>
        <field name="amount" eval="0.00"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
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

    <record id="tva_export_0" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_dz_pcg_chart_template"/>
        <field name="name">TVA 0% export (vente)</field>
        <field name="description">EXPORT-0</field>
        <field name="amount" eval="0.00"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
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

    <record id="tva_import_0" model="account.tax.template">
        <field name="chart_template_id" ref="l10n_dz_pcg_chart_template"/>
        <field name="name">TVA 0% import (achat)</field>
        <field name="description">IMPORT-0</field>
        <field name="amount" eval="0.00"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
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

## File: models\account_move.py

```python
from odoo import api, models, fields


class AccountMove(models.Model):
    _inherit = 'account.move'

    amount_total_words = fields.Char("Amount total in words", compute="_compute_amount_total_words")

    @api.depends('amount_total', 'currency_id')
    def _compute_amount_total_words(self):
        for record in self:
            record.amount_total_words = record.currency_id.amount_to_text(record.amount_total)

```

## File: models\__init__.py

```python
from . import account_move

```

## File: report\account_move_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="invoice_report_amount_in_words_inherit" inherit_id="account.report_invoice_document">
        <xpath expr="//div[hasclass('clearfix')]" position="after">
            <div style="margin-top:30px; margin-bottom:20px;">
                <p class="font-weight-bold">
                    Arranged the present invoice in the amount of : <span t-field="o.amount_total_words"/>
                </p>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="6.84" width="50.4" height="33.84" maskUnits="userSpaceOnUse">
      <rect x="5.85" y="9" width="48" height="31" rx="1" style="fill: #fff"/>
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
    <rect x="5.83" y="10.57" width="48.45" height="31.57" rx="1" style="fill: #393939;opacity: 0.44;isolation: isolate"/>
    <g style="mask: url(#b)">
      <image width="640" height="427" transform="translate(4.8 6.84) scale(0.08 0.08)" xlink:href="data:image/png;base64,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"/>
    </g>
  </g>
</svg>

```

