# Odoo Module: l10n_fr

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2008 JAILLET Simon - CrysaLEAD - www.crysalead.fr

from . import models
from odoo import api, SUPERUSER_ID

def _l10n_fr_post_init_hook(cr, registry):
    _preserve_tag_on_taxes(cr, registry)
    _setup_inalterability(cr, registry)

def _preserve_tag_on_taxes(cr, registry):
    from odoo.addons.account.models.chart_template import preserve_existing_tags_on_taxes
    preserve_existing_tags_on_taxes(cr, registry, 'l10n_fr')

def _setup_inalterability(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    # enable ping for this module
    env['publisher_warranty.contract'].update_notification(cron_mode=True)

    fr_companies = env['res.company'].search([('partner_id.country_id.code', 'in', env['res.company']._get_unalterable_country())])
    if fr_companies:
        fr_companies._create_secure_sequence(['l10n_fr_closing_sequence_id'])

        for fr_company in fr_companies:
            fr_journals = env['account.journal'].search([('company_id', '=', fr_company.id)])
            fr_journals.filtered(lambda x: not x.secure_sequence_id)._create_secure_sequence(['secure_sequence_id'])

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2008 JAILLET Simon - CrysaLEAD - www.crysalead.fr

{
    'name': 'France - Accounting',
    'version': '2.1',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the module to manage the accounting chart for France in Odoo.
========================================================================

This module applies to companies based in France mainland. It doesn't apply to
companies based in the DOM-TOMs (Guadeloupe, Martinique, Guyane, Réunion, Mayotte).

This localisation module creates the VAT taxes of type 'tax included' for purchases
(it is notably required when you use the module 'hr_expense'). Beware that these
'tax included' VAT taxes are not managed by the fiscal positions provided by this
module (because it is complex to manage both 'tax excluded' and 'tax included'
scenarios in fiscal positions).

This localisation module doesn't properly handle the scenario when a France-mainland
company sells services to a company based in the DOMs. We could manage it in the
fiscal positions, but it would require to differentiate between 'product' VAT taxes
and 'service' VAT taxes. We consider that it is too 'heavy' to have this by default
in l10n_fr; companies that sell services to DOM-based companies should update the
configuration of their taxes and fiscal positions manually.

**Credits:** Sistheo, Zeekom, CrysaLEAD, Akretion and Camptocamp.
""",
    'depends': [
        'account',
        'base_iban',
        'base_vat',
    ],
    'data': [
        'data/l10n_fr_chart_data.xml',
        'data/account.account.template.csv',
        'data/account.group.template.csv',
        'data/account_chart_template_data.xml',
        'views/l10n_fr_view.xml',
        'data/account_tax_group_data.xml',
        'data/tax_report_data.xml',
        'data/account_tax_data.xml',
        'data/res_country_data.xml',
        'data/account_fiscal_position_template_data.xml',
        'data/account_reconcile_model_template.xml',
        'data/account_chart_template_configure_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'post_init_hook': '_l10n_fr_post_init_hook',
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","tag_ids/id","reconcile"
"pcg_1011","Capital souscrit - non appelé","101100","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1012","Capital souscrit - appelé non versé","101200","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1013"," Capital souscrit appelé, versé","101300","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_10131","Capital non amorti","101310","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_10132","Capital amorti","101320","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1018","Capital souscrit soumis à des réglementations particulières","101800","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_102"," Fonds fiduciaires","102000","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1041","Primes d'émission","104100","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1042","Primes de fusion","104200","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1043","Primes d'apport","104300","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1044","Primes de conversion d'obligations en actions","104400","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1045","Bons de souscription d'actions","104500","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1051","Réserve spéciale de réévaluation","105100","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1052","Écart de réévaluation libre","105200","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1053","Réserve de réévaluation","105300","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1055","Écarts de réévaluation (autres opérations légales)","105500","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1057","Autres écarts de réévaluation en France","105700","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1058","Autres écarts de réévaluation à l'étranger","105800","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1061"," Réserve légale","106100","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_10611","Réserve légale proprement dite","106110","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_10612","Plus-values nettes à long terme","106120","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1062","Réserves indisponibles","106200","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1063","Réserves statutaires ou contractuelles","106300","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1064"," Réserves réglementées","106400","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_10641","Plus-values nettes à long terme","106410","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_10643","Réserves consécutives à l'octroi de subventions d'investissement","106430","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_10648","Autres réserves réglementées","106480","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1068"," Autres réserves","106800","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_10681","Réserve de propre assureur","106810","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_10688","Réserves diverses","106880","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_107","Écarts d'équivalence","107000","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_108","Compte de l'exploitant","108000","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_109","Actionnaires : capital souscrit - non appelé","109000","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_110","Report à nouveau (solde créditeur)","110000","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_119","Report à nouveau (solde débiteur)","119000","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_120","Résultat de l'exercice (bénéfice)","120000","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_129","Résultat de l'exercice (perte)","129000","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1311","Subventions d'équipement - État","131100","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1312","Subventions d'équipement - Régions","131200","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1313","Subventions d'équipement - Départements","131300","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1314","Subventions d'équipement - Communes","131400","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1315","Subventions d'équipement - Collectivités publiques","131500","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1316","Subventions d'équipement - Entreprises publiques","131600","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1317","Subventions d'équipement - Entreprises et organismes privés","131700","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1318","Subventions d'équipement - Autres","131800","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_138","Autres subventions d'investissement (même ventilation que celle du compte 131)","138000","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_13911","Subventions d'équipement inscrites au compte de résultat - État","139110","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_13912","Subventions d'équipement inscrites au compte de résultat - Régions","139120","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_13913","Subventions d'équipement inscrites au compte de résultat - Départements","139130","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_13914","Subventions d'équipement inscrites au compte de résultat - Communes","139140","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_13915","Subventions d'équipement inscrites au compte de résultat - Collectivités publiques","139150","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_13916","Subventions d'équipement inscrites au compte de résultat - Entreprises publiques","139160","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_13917","Subventions d'équipement inscrites au compte de résultat - Entreprises et organismes privés","139170","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_13918","Subventions d'équipement inscrites au compte de résultat - Autres","139180","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1398","Autres subventions d'investissement (même ventilation que celle du compte 1391)","139800","account.data_account_type_equity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1423","Provisions reconstitution des gisements miniers et pétroliers","142300","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1424","Provisions pour investissement (participation des salariés)","142400","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1431","Provisions réglementées relatives aux stocks - Hausse de prix","143100","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1432","Provisions réglementées relatives aux stocks - Fluctuation des cours","143200","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_144","Provisions réglementées relatives aux autres éléments de l'actif","144000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_145","Amortissements dérogatoires","145000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_146","Provision spéciale de réévaluation","146000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_147","Plus-values réinvesties","147000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_148","Autres provisions réglementées","148000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1511","Provisions pour litiges","151100","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1512","Provisions pour garanties données aux clients","151200","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1513","Provisions pour pertes sur marchés à terme","151300","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1514","Provisions pour amendes et pénalités","151400","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1515","Provisions pour pertes de change","151500","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1516","Provisions pour pertes sur contrats","151600","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1518","Autres provisions pour risques","151800","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_153","Provisions pour pensions et obligations similaires","153000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_154","Provisions pour restructurations","154000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_155","Provisions pour impôts","155000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_156","Provisions pour renouvellement des immobilisations (entreprises concessionnaires)","156000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_157"," Provisions pour charges à répartir sur plusieurs exercices","157000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1572","Provisions pour charges à répartir sur plusieurs exercices - Gros entretien ou grandes révisions","157200","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1581","Provisions pour remises en état","158100","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_161","Emprunts obligataires convertibles","161000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_162","Obligations représentatives de passifs nets remis en fiducie","162000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_163","Autres emprunts obligataires","163000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_164","Emprunts auprès des établissements de crédit","164000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1651","Dépôts","165100","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1655","Cautionnements","165500","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1661","Participation des salariés aux résultats - Comptes bloqués","166100","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1662","Participation des salariés aux résultats - Fonds de participation","166200","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1671","Emprunts et dettes assortis de conditions particulières - Emissions de titres participatifs","167100","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1674","Emprunts et dettes assortis de conditions particulières - Avances conditionnées de l'État","167400","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1675","Emprunts et dettes assortis de conditions particulières - Emprunts participatifs","167500","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1681","Autres emprunts et dettes assimilées - Autres emprunts","168100","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1685","Autres emprunts et dettes assimilées - Rentes viagères capitalisées","168500","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1687","Autres emprunts et dettes assimilées - Autres dettes","168700","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1688","Intérêts courus","168800","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_16881","Intérêts courus sur emprunts obligataires convertibles","168810","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_16883","Intérêts courus sur autres emprunts obligataires","168830","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_16884","Intérêts courus sur emprunts auprès des établissements de crédit","168840","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_16885","Intérêts courus sur dépôts et cautionnements reçus","168850","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_16886","Intérêts courus sur participation des salariés aux résultats","168860","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_16887","Intérêts courus sur emprunts et dettes assortis de conditions particulières","168870","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_16888","Intérêts courus sur autres emprunts et dettes assimilées","168880","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_169","Primes de remboursement des obligations","169000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_171","Dettes rattachées à des participations (groupe)","171000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_174","Dettes rattachées à des participations (hors groupe)","174000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1781","Dettes rattachées à des sociétés en participation - Principal","178100","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_1788","Dettes rattachées à des sociétés en participation - Intérêts courus","178800","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_181","Comptes de liaison des établissements","181000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_186","Biens et prestations de services échangés entre établissements (charges)","186000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_187","Biens et prestations de services échangés entre établissements (produits)","187000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_188","Comptes de liaison des sociétés en participation","188000","account.data_account_type_non_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2011","Immobilisations incorporelles - Frais d'établissement - Frais de constitution","201100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2012"," Frais de premier établissement","201200","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_20121","Immobilisations incorporelles - Frais d'établissement - Frais de prospection","201210","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_20122","Immobilisations incorporelles - Frais d'établissement - Frais de publicité","201220","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2013","Immobilisations incorporelles - Frais d'augmentation de capital et d'opérations diverses (fusions, scissions, transformations)","201300","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_203","Immobilisations incorporelles - Frais de recherche et de développement","203000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_205","Immobilisations incorporelles - Concessions et droits similaires, brevets, licences, marques, procédés, logiciels, droits et valeurs similaires","205000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_206","Immobilisations incorporelles - Droit au bail","206000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_207","Immobilisations incorporelles - Fonds commercial","207000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_208","Autres immobilisations incorporelles","208000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2111","Immobilisations corporelles - Terrains nus","211100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2112","Immobilisations corporelles - Terrains aménagés","211200","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2113","Immobilisations corporelles - Sous-sols et sur-sols","211300","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2114","Terrains de carrières (Tréfonds)","211400","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2115","Terrains bâtis","211500","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21151","Immobilisations corporelles - Terrains bâtis - Ensembles immobiliers industriels","211510","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21155","Immobilisations corporelles - Terrains bâtis - Ensembles immobiliers administratifs et commerciaux","211550","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21158"," Autres ensembles immobiliers","211580","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_211581","Immobilisations corporelles - Terrains bâtis affectés aux opérations professionnelles","211581","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_211588","Immobilisations corporelles - Terrains bâtis affectés aux opérations non professionnelles","211588","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2116","Immobilisations corporelles - Compte d'ordre sur immobilisations","211600","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_212","Immobilisations corporelles - Agencements et aménagements de terrains (même ventilation que celle du compte 211)","212000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2131"," Bâtiments","213100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21311","Immobilisations corporelles - Bâtiments - Ensembles immobiliers industriels","213110","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21315","Immobilisations corporelles - Bâtiments - Ensembles immobiliers administratifs et commerciaux","213150","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_213181","Immobilisations corporelles - Bâtiments affectés aux opérations professionnelles","213181","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_213188","Immobilisations corporelles - Bâtiments affectés aux opérations non professionnelles","213188","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2135","Immobilisations corporelles - Installations générales, agencements, aménagements des constructions (même ventilation que celle du compte 2131)","213500","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21351"," Ensembles immobiliers industriels (A, B)","213510","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21355"," Ensembles immobiliers administratifs et commerciaux (A, B)","213550","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_213581"," affectés aux opérations professionnelles (A, B)","213581","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_213588"," affectés aux opérations non professionnelles (A, B)","213588","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21381","Immobilisations corporelles - Ouvrages d'infrastructure - Voies de terre","213810","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21382","Immobilisations corporelles - Ouvrages d'infrastructure - Voies de fer","213820","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21383","Immobilisations corporelles - Ouvrages d'infrastructure - Voies d'eau","213830","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21384","Immobilisations corporelles - Ouvrages d'infrastructure - Barrages","213840","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21385","Immobilisations corporelles - Ouvrages d'infrastructure - Pistes d'aérodromes","213850","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_214","Immobilisations corporelles - Constructions sur sol d'autrui (même ventilation que celle du compte 213)","214000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21511","Immobilisations corporelles - Installations complexes spécialisées sur sol propre","215110","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21514","Immobilisations corporelles - Installations complexes spécialisées sur sol d'autrui","215140","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21531","Immobilisations corporelles - Installations à caractère spécifique sur sol propre","215310","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_21534","Immobilisations corporelles - Installations à caractère spécifique sur sol d'autrui","215340","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2154","Immobilisations corporelles - Matériels industriels","215400","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2155","Immobilisations corporelles - Outillage industriel","215500","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2157","Immobilisations corporelles - Agencements et aménagements des matériels et outillage industriels","215700","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2181","Immobilisations corporelles - Installations générales agencements aménagements divers","218100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2182","Immobilisations corporelles - Matériel de transport","218200","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2183","Immobilisations corporelles - Matériel de bureau et matériel informatique","218300","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2184","Immobilisations corporelles - Mobilier","218400","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2185","Immobilisations corporelles - Cheptel","218500","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2186","Immobilisations corporelles - Emballages récupérables","218600","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2187","Mali de fusions sur actifs corporels","218700","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_22","Immobilisations mises en concession","220000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2312","Immobilisations corporelles en cours - Terrains","231200","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2313","Immobilisations corporelles en cours - Constructions","231300","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2315","Immobilisations corporelles en cours - Installations techniques matériel et outillage industriels","231500","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2318","Autres immobilisations corporelles en cours","231800","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_232","Immobilisations incorporelles en cours","232000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_237","Avances et acomptes versés sur commandes d'immobilisations incorporelles","237000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2382","Avances et acomptes versés sur commandes d'immobilisations corporelles - Terrains","238200","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2383","Avances et acomptes versés sur commandes d'immobilisations corporelles - Constructions","238300","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2385","Avances et acomptes versés sur commandes d'immobilisations corporelles - Installations techniques matériel et outillage industriels","238500","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2388","Avances et acomptes versés sur commandes d'immobilisations corporelles - Autres immobilisations corporelles","238800","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_25","Parts dans des entreprises liées et créances sur des entreprises liées","250000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2611","Titres de participation - Actions","261100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2618","Autres titres de participation","261800","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_266","Autres formes de participation","266000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2661","Droits représentatifs d’actifs nets remis en fiducie","266100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2671","Créances rattachées à des participations (groupe)","267100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2674","Créances rattachées à des participations (hors groupe)","267400","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2675","Versements représentatifs d'apports non capitalisés (appel de fonds)","267500","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2676","Avances consolidables","267600","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2677","Autres créances rattachées à des participations","267700","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2678","Créances rattachées à des participations - Intérêts courus","267800","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2681","Créances rattachées à des sociétés en participation - Principal","268100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2688","Créances rattachées à des sociétés en participation - Intérêts courus","268800","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_269","Versements restant à effectuer sur titres de participation non libérés","269000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2711","Titres immobilisés autres que les titres immobilisés de l'activité de portefeuille - Actions","271100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2718","Titres immobilisés autres que les titres immobilisés de l'activité de portefeuille - Autres titres","271800","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2721","Titres immobilisés - Obligations","272100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2722","Titres immobilisés - Bons","272200","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_273","Titres immobilisés de l'activité de portefeuille","273000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2741","Prêts participatifs","274100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_2742","Prêts aux associés","274200","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_2743","Prêts au personnel","274300","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_2748","Autres prêts","274800","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_2751","Dépôts","275100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_2755","Cautionnements","275500","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_2761","Autres créances immobilisées - Créances diverses","276100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_27682","Autres créances immobilisées - Intérêts courus sur titres immobilisés (droits de créance)","276820","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_27684","Autres créances immobilisées - Intérêts courus sur prêts","276840","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_27685","Autres créances immobilisées - Intérêts courus sur dépôts et cautionnements","276850","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_27688","Autres créances immobilisées - Intérêts courus sur créances diverses","276880","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2771","Actions propres ou parts propres","277100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2772","Actions propres ou parts propres en voie d'annulation","277200","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_278"," Mali de fusion sur actifs financiers","278000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_279","Versements restant à effectuer sur titres immobilisés non libérés","279000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2801","Amortissements des immobilisations incorporelles - Frais d'établissement (même ventilation que celle du compte 201)","280100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2803","Amortissements des immobilisations incorporelles - Frais de recherche et de développement","280300","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2805","Amortissements des immobilisations incorporelles - Concessions et droits similaires, brevets, licences, logiciels, droits et valeurs similaires","280500","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2807","Amortissements des immobilisations incorporelles - Fonds commercial","280700","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2808","Amortissements des autres immobilisations incorporelles","280800","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_28081"," Amortissements du mali de fusion sur actifs incorporels","280810","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2812","Amortissements des immobilisations corporelles - Agencements aménagements de terrains (même ventilation que celle du compte 212)","281200","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2813","Amortissements des immobilisations corporelles - Constructions (même ventilation que celle du compte 213)","281300","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2814","Amortissements des immobilisations corporelles - Constructions sur sol d'autrui (même ventilation que celle du compte 214)","281400","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2815","Amortissements des immobilisations corporelles - Installations matériel et outillage industriels (même ventilation que celle du compte 215)","281500","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2818","Amortissements des autres immobilisations corporelles (même ventilation que celle du compte 218)","281800","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_28187"," Amortissement du mali de fusion sur actifs corporels","281870","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_282","Amortissements des immobilisations mises en concession","282000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2905","Dépréciations des immobilisations incorporelles - Marques, procédés, droits et valeurs similaires","290500","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2906","Dépréciations des immobilisations incorporelles - Droit au bail","290600","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2907","Dépréciations des immobilisations incorporelles - Fonds commercial","290700","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2908","Dépréciations des autres immobilisations incorporelles","290800","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_29081","Dépréciation du mali de fusion sur actifs incorporels","290810","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_29187"," Dépréciation du mali de fusion sur actifs corporels","291870","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_292","Dépréciations des immobilisations mises en concession","292000","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2931","Dépréciations des immobilisations corporelles en cours","293100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2932","Dépréciations des immobilisations incorporelles en cours","293200","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2961","Provisions pour dépréciation des titres de participation","296100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2966","Provisions pour dépréciation des autres formes de participation","296600","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2967","Provisions pour dépréciation des créances rattachées à des participations (même ventilation que celle du compte 267)","296700","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2968","Provisions pour dépréciation des créances rattachées à des sociétés en participation (même ventilation que celle du compte 268)","296800","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2971","Provisions pour dépréciation des titres immobilisés autres que les titres immobilisés de l'activité de portefeuille - droit de propriété (ventilation : 271)","297100","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2972","Provisions pour dépréciation des titres immobilisés - droit de créance (même ventilation que celle du compte 272)","297200","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2973","Provisions pour dépréciation des titres immobilisés de l'activité de portefeuille","297300","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2974","Provisions pour dépréciation des prêts (même ventilation que celle du compte 274)","297400","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2975","Provisions pour dépréciation des dépôts et cautionnements versés (même ventilation que celle du compte 275)","297500","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_2976","Provisions pour dépréciation des autres créances immobilisées (même ventilation que celle du compte 276)","297600","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_29787"," Dépréciation du mali de fusion sur actifs financiers","297870","account.data_account_type_fixed_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_311","Matières premières (ou groupe) A","311000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_312","Matières premières (ou groupe) B","312000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_317","Fournitures A, B, C, ..","317000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3211","Matières consommables (ou groupe) C","321100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3212","Matières consommables (ou groupe) D","321200","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3221","Combustibles","322100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3222","Produits d'entretien","322200","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3223","Fournitures d'atelier et d usine","322300","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3224","Fournitures de magasin","322400","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3225","Fournitures de bureau","322500","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3261","Emballages perdus","326100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3265","Emballages récupérables non identifiables","326500","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3267","Emballages à usage mixte","326700","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3311","Produit en cours P 1","331100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3312","Produit en cours P 2","331200","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3351","Travaux en cours T 1","335100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3352","Travaux en cours T 2","335200","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3411","Études en cours E 1","341100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3412","Études en cours E 2","341200","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3451","Prestations de services en cours S 1","345100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3452","Prestations de services en cours S 2","345200","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3511","Stocks produits intermédiaires (ou groupe) A","351100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3512","Stocks produits intermédiaires (ou groupe) B","351200","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3551","Stocks produits finis (ou groupe) A","355100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3552","Stocks produits finis (ou groupe) B","355200","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3581","Stocks produits résiduels - Déchets","358100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3585","Stocks produits résiduels - Rebuts","358500","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3586","Stocks produits résiduels - Matières de récupération","358600","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_36","Stocks provenant d'immobilisations","360000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_371","Stocks de marchandises (ou groupe) A","371000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_372","Stocks de marchandises (ou groupe) B","372000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_38","Stocks en voie d'acheminement","380000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3911","Provisions pour dépréciation des matières premières (ou groupe) A","391100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3912","Provisions pour dépréciation des matières premières (ou groupe) B","391200","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3917","Provisions pour dépréciation des fournitures A, B, C, ..","391700","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3921","Provisions pour dépréciation des matières consommables (même ventilation que celle du compte 321)","392100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3922","Provisions pour dépréciation des fournitures consommables (même ventilation que celle du compte 322)","392200","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3926","Provisions pour dépréciation des emballages (même ventilation que celle du compte 326)","392600","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3931","Provisions pour dépréciation des produits en cours (même ventilation que celle du compte 331)","393100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3935","Provisions pour dépréciation des travaux en cours (même ventilation que celle du compte 335)","393500","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3941","Provisions pour dépréciation des études en cours (même ventilation que celle du compte 341)","394100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3945","Provisions pour dépréciation des prestations de services en cours (même ventilation que celle du compte 345)","394500","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3951","Provisions pour dépréciation des produits intermédiaires (même ventilation que celle du compte 351)","395100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3955","Provisions pour dépréciation des produits finis (même ventilation que celle du compte 355)","395500","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3971","Provisions pour dépréciation des stocks de marchandises (ou groupe) A","397100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_3972","Provisions pour dépréciation des stocks de marchandises (ou groupe) B","397200","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_400","Fournisseurs et comptes rattachés","400000","account.data_account_type_payable","l10n_fr.l10n_fr_pcg_chart_template","","True"
"fr_pcg_pay","Fournisseurs - Achats de biens et prestations de services","401100","account.data_account_type_payable","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4017","Fournisseurs - Retenues de garantie","401700","account.data_account_type_payable","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_403","Fournisseurs - Effets à payer","403000","account.data_account_type_payable","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4041","Fournisseurs - Achats d'immobilisations","404100","account.data_account_type_payable","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4047","Fournisseurs d'immobilisations - Retenues de garantie","404700","account.data_account_type_payable","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_405","Fournisseurs d'immobilisations - Effets à payer","405000","account.data_account_type_payable","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4081","Factures non parvenues - Fournisseurs","408100","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4084","Factures non parvenues - Fournisseurs d'immobilisations","408400","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4088","Factures non parvenues - Fournisseurs - Intérêts courus","408800","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4091"," Fournisseurs  avance et acomptes versés sur commandes","409100","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_4096","Fournisseurs débiteurs - Créances pour emballages et matériel à rendre","409600","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_40971","Fournisseurs débiteurs - Autres avoirs des fournisseurs d'exploitation","409710","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_40974","Fournisseurs débiteurs - Autres avoirs des fournisseurs d'immobilisations","409740","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4098","Fournisseurs débiteurs - Rabais, remises, ristournes à obtenir et autres avoirs non encore reçus","409800","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_410","Clients et comptes rattachés","410000","account.data_account_type_receivable","l10n_fr.l10n_fr_pcg_chart_template","","True"
"fr_pcg_recv","Clients - Ventes de biens ou de prestations de services","411100","account.data_account_type_receivable","l10n_fr.l10n_fr_pcg_chart_template","","True"
"fr_pcg_recv_pos","Clients - Ventes de biens ou de prestations de services (PoS)","411101","account.data_account_type_receivable","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4117","Clients - Retenues de garantie","411700","account.data_account_type_receivable","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_413","Clients - Effets à recevoir","413000","account.data_account_type_receivable","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_416","Clients douteux ou litigieux","416000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4181","Clients - Factures à établir","418100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4188","Clients - Intérêts courus non encore facturés","418800","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4191","Clients créditeurs - Avances et acomptes reçus sur commandes","419100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4196","Clients créditeurs - Dettes pour emballages et matériels consignés","419600","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4197","Clients créditeurs - Autres avoirs","419700","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4198","Clients créditeurs - Rabais, remises, ristournes à accorder et autres avoirs à établir","419800","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_421","Personnel - Rémunérations dues","421000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_422","Comités d'entreprise, d'établissement","422000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4246","Participation des salariés aux résultats - Réserve spéciale","424600","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4248","Participation des salariés aux résultats - Comptes courants","424800","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_425","Personnel - Avances et acomptes","425000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_426","Personnel - Dépôts","426000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_427","Personnel - Oppositions","427000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4282","Personnel - Dettes provisionnées pour congés à payer","428200","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4284","Personnel - Dettes provisionnées pour participation des salariés aux résultats","428400","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4286","Personnel - Autres charges à payer","428600","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4287","Personnel - Produits à recevoir","428700","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_431","Sécurité Sociale","431000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_437","Autres organismes sociaux","437000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4382","Charges sociales sur congés à payer","438200","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4386","Organismes sociaux - Autres charges à payer","438600","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4387","Organismes sociaux - Produits à recevoir","438700","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4411","État - Subventions à recevoir - Subventions d'investissement","441100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4417","État - Subventions à recevoir - Subventions d'exploitation","441700","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4418","État - Subventions à recevoir - Subventions d'équilibre","441800","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4419","État - Subventions à recevoir - Avances sur subventions","441900","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4424","État - Impôts et taxes recouvrables sur des tiers - Obligataires","442400","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4425","État - Impôts et taxes recouvrables sur des tiers - Associés","442500","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4431","Créances sur l'État résultant de la suppression de la règle du décalage d'un mois en matière de TVA","443100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4438","État - Intérêts courus sur créances figurant au compte 4431","443800","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_444","État - Impôts sur les bénéfices","444000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4452","TVA due sur acquisitions intracommunautaires","445200","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_44521","TVA due sur prestations intracommunautaires","445210","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_4453","TVA due sur importations (autoliquidation)","445300","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_44531","TVA due sur prestations hors UE","445310","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_44551","TVA à décaisser","445510","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_44558","Taxes assimilées à la TVA","445580","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_44562","TVA déductible sur immobilisations","445620","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_44563","TVA déductible transférée par d'autres entreprises","445630","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_44564","TVA déductible sur opérations non réglées","445640","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_44566","TVA déductible sur autres biens et services","445660","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_445662","TVA déductible intracommunautaire","445662","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_445663","TVA déductible hors UE (autoliquidation)","445663","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_44567","Crédit de TVA à reporter","445670","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_44568","Taxes déductibles assimilées à la TVA","445680","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_44571","TVA collectée","445710","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_44574","TVA collectée sur opérations non réglées","445740","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_44578","Taxes collectées assimilées à la TVA","445780","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_445800","Taxes sur le chiffre d'affaires à régulariser ou en attente","445800","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_44581","Acomptes - Régime simplifié d'imposition","445810","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_44582","Acomptes - Régime du forfait","445820","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_44583","Remboursement de taxes sur le chiffre d'affaires demandé","445830","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_44584","TVA récupérée d'avance","445840","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_44586","Taxes sur le chiffre d'affaires sur factures non parvenues","445860","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_44587","Taxes sur le chiffre d'affaires sur factures à établir","445870","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_446","Obligations cautionnées","446000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_447","Autres impôts, taxes et versements assimilés","447000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4482","État - Charges fiscales sur congés à payer","448200","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4486","État - Charges à payer","448600","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4487","État - Produits à recevoir","448700","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_449","Quotas d'émission à restituer à l'État","449000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_451","Groupe","451000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4551","Associés - Comptes courants - Principal","455100","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4558","Associés - Comptes courants - Intérêts courus","455800","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_45611","Associés - Comptes d'apport en société - Apports en nature","456110","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_45615","Associés - Comptes d'apport en société - Apports en numéraire","456150","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_45621","Actionnaires - Capital souscrit et appelé, non versé","456210","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_45625","Associés - Capital appelé, non versé","456250","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4563","Associés - Versements reçus sur augmentation de capital","456300","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4564","Associés - Versements anticipés","456400","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4566","Actionnaires défaillants","456600","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4567","Associés - Capital à rembourser","456700","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_457","Associés - Dividendes à payer","457000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4581","Associés - Opérations faites en commun et en GIE - Opérations courantes","458100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4588","Associés - Opérations faites en commun et en GIE - Intérêts courus","458800","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_462","Créances sur cessions d'immobilisations","462000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_464","Dettes sur acquisitions de valeurs mobilières de placement","464000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_465","Créances sur cessions de valeurs mobilières de placement","465000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_467","Autres comptes débiteurs ou créditeurs","467000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_4686","Charges à payer","468600","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4687","Produits à recevoir","468700","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_471","Compte d'attente","471000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_472"," Comptes d'attente","472000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_473"," Comptes d'attente","473000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_474"," Comptes d'attente","474000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_475"," Comptes d'attente","475000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4761","Différence de conversion - Actif - Diminution des créances","476100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4762","Différence de conversion - Actif - Augmentation des dettes","476200","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4768","Différence de conversion - Actif - Différences compensées par couverture de change","476800","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4771","Différences de conversion - Passif - Augmentation des créances","477100","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4772","Différences de conversion - Passif - Diminution des dettes","477200","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4778","Différences de conversion - Passif - Différences compensées par couverture de change","477800","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_478","Autres comptes transitoires","478000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_4816","Charges à répartir sur plusieurs exercices - Frais d'émission des emprunts","481600","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_486","Charges constatées d'avance","486000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_487","Produits constatés d'avance","487000","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4886","Comptes de répartition périodique des charges","488600","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4887","Comptes de répartition périodique des produits","488700","account.data_account_type_current_liabilities","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_491","Provisions pour dépréciation des comptes de clients","491000","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4951","Provisions pour dépréciation des comptes du groupe","495100","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4955","Provisions pour dépréciation des comptes courants des associés","495500","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4958","Provisions pour dépréciation des opérations faites en commun et en GIE","495800","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4962","Provisions pour dépréciation des créances sur cessions d'immobilisations","496200","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4965","Provisions pour dépréciation des créances sur cessions de valeurs mobilières de placement","496500","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_4967","Provisions pour dépréciation - Autres comptes débiteurs","496700","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_501","Valeurs mobilières de placement - Parts dans entreprises liées","501000","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_502","Valeurs mobilières de placement - Actions propres","502000","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5021","Actons destinées à être attribuées aux employés et affectées à des plans déterminés","502100","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5022","Actons disponibles pour être attribuées aux employés ou pour la régularisation des cours de la bourse","502200","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5031","Valeurs mobilières de placement - Titres cotés","503100","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5035","Valeurs mobilières de placement - Titres non cotés","503500","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_504","Valeurs mobilières de placement - Autres titres conférant un droit de propriété","504000","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_505","Obligations et bons émis par la société et rachetés par elle","505000","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5061","Obligations cotés","506100","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5065","Obligations non cotés","506500","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_507","Bons du Trésor et bons de caisse à court terme","507000","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5081","Autres valeurs mobilières de placement","508100","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5082","Bons de souscription","508200","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5088","Intérêts courus sur obligations, bons et valeurs assimilées","508800","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_509","Versements restant à effectuer sur valeurs mobilières de placement non libérées","509000","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5111","Coupons échus à l'encaissement","511100","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_5112","Chèques à encaisser","511200","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_5113","Effets à l'encaissement","511300","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_5114","Effets à l'escompte","511400","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","True"
"pcg_5121"," Comptes en monnaie nationale","512100","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5124","Banques - Comptes en devises","512400","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_514","Chèques postaux","514000","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_515","Caisses du Trésor et des établissements publics","515000","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_516","Sociétés de bourse","516000","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_517","Autres organismes financiers","517000","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5181","Intérêts courus à payer","518100","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5188","Intérêts courus à recevoir","518800","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5191","Concours bancaires courants - Crédit de mobilisation de créances commerciales (CMCC)","519100","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5193","Concours bancaires courants - Mobilisation de créances nées à l'étranger","519300","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5198","Concours bancaires courants - Intérêts courus sur concours bancaires courants","519800","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_52","Instruments de trésorerie","520000","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5311","Caisse en monnaie nationale","531100","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5314","Caisse en devises","531400","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_532","Caisse succursale (ou usine) A","532000","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_533","Caisse succursale (ou usine) B","533000","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_54","Régies d'avances et accréditifs","540000","account.data_account_type_liquidity","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5903","Provisions pour dépréciation des actions","590300","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5904","Provisions pour dépréciation des autres titres conférant un droit de propriété","590400","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5906","Provisions pour dépréciation des obligations","590600","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_5908","Provisions pour dépréciation des autres valeurs mobilières de placement et créances assimilées (provisions)","590800","account.data_account_type_current_assets","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_6011","Achats stockés - Matières premières (ou groupe) A","601100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6012","Achats stockés - matières premières (ou groupe) B","601200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6017","Achats stockés - Fournitures A, B, C, ..","601700","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_60211","Achats stockés - Matières consommables (ou groupe) C","602110","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_60212","Achats stockés - Matières consommables (ou groupe) D","602120","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_60221","Achats stockés - Combustibles","602210","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_60222","Achats stockés - Produits d'entretien","602220","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_60223","Achats stockés - Fournitures d'atelier et d'usine","602230","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_60224","Achats stockés - Fournitures de magasin","602240","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_60225","Achats stockés - Fournitures de bureau","602250","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_60261","Achats stockés - Emballages perdus","602610","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_60265","Achats stockés - Emballages récupérables non identifiables","602650","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_60267","Achats stockés - Emballages à usage mixte","602670","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6031","Variation des stocks de matières premières (et fournitures)","603100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6032","Variation des stocks des autres approvisionnements","603200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6037","Variation des stocks de marchandises","603700","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_604","Achats d'études et prestations de services","604000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_605","Achats de matériel équipements et travaux","605000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6061","Fournitures non stockables (eau, énergie...)","606100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6063","Fournitures d'entretien et de petit équipement","606300","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6064","Fournitures administratives","606400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6068","Achats autres matières et fournitures","606800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6071","Achats de marchandises (ou groupe) A","607100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6072","Achats de marchandises (ou groupe) B","607200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_608","Frais accessoires incorporés aux achats","608000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_6091","Rabais, remises et ristournes obtenus sur achats de matières premières (et fournitures)","609100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6092","Rabais, remises et ristournes obtenus sur achats d'autres approvisionnements stockés","609200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6094","Rabais, remises et ristournes obtenus sur achats d'études et prestations de services","609400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6095","Rabais, remises et ristournes obtenus sur achats de matériel, équipements et travaux","609500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6096","Rabais, remises et ristournes obtenus sur achats d'approvisionnements non stockés","609600","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6097","Rabais, remises et ristournes obtenus sur achats de marchandises","609700","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6098","Rabais, remises et ristournes non affectés","609800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_611","Sous-traitance générale","611000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6122","Redevances de crédit-bail mobilier","612200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6125","Redevances de crédit-bail immobilier","612500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6132","Locations immobilières","613200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6135","Locations mobilières","613500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6136","Locations malis sur emballages","613600","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_614","Charges locatives et de copropriété","614000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6152","Entretien et réparations sur biens immobiliers","615200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6155","Entretien et réparations sur biens mobiliers","615500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6156","Maintenance","615600","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6161","Assurance multirisques","616100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6162","Assurance obligatoire dommage construction","616200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_61636","Assurance transport sur achats","616360","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_61637","Assurance transport sur ventes","616370","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_61638","Assurance transport sur autres biens","616380","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6164","Assurance risques d'exploitation","616400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6165","Assurance insolvabilité clients","616500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_617","Études et recherches","617000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6181","Documentation générale","618100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6183","Documentation technique","618300","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6185","Frais de colloques, séminaires, conférences","618500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_619","Rabais, remises et ristournes obtenus sur services extérieurs","619000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6211","Personnel intérimaire","621100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6214","Personnel détaché ou prêté à l'entreprise","621400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6221","Commissions et courtages sur achats","622100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6222","Commissions et courtages sur ventes","622200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6224","Rémunérations des transitaires","622400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6225","Rémunérations d'affacturage","622500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6226","Honoraires","622600","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6227","Frais d'actes et de contentieux","622700","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6228","Rémunérations d'intermédiaires et honoraires - Divers","622800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6231","Annonces et insertions","623100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6232","Échantillons","623200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6233","Foires et expositions","623300","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6234","Cadeaux à la clientèle","623400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6235","Primes","623500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6236","Catalogues et imprimés","623600","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6237","Publications","623700","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6238","Publicité, publications, relations publiques - Divers (pourboires, dons courants...)","623800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6241","Transports sur achats","624100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6242","Transports sur ventes","624200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6243","Transports entre établissements ou chantiers","624300","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6244","Transports administratifs","624400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6247","Transports collectifs du personnel","624700","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6248","Transports divers","624800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6251","Voyages et déplacements","625100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6255","Frais de déménagement","625500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6256","Missions","625600","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6257","Réceptions","625700","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_626","Frais postaux et frais de télécommunications","626000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6271","Frais sur titres (achat, vente, garde)","627100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6272","Commissions et frais sur émission d'emprunts","627200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6275","Frais sur effets","627500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6276","Location de coffres","627600","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6278","Autres frais et commissions sur prestations de services","627800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6281","Concours divers (cotisations...)","628100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6284","Frais de recrutement de personnel","628400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_629","Rabais, remises et ristournes obtenus sur autres services extérieurs","629000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6311","Taxe sur les salaires","631100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6312","Taxe d'apprentissage","631200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6313","Participation des employeurs à la formation professionnelle continue","631300","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6314","Cotisation pour défaut d'investissement obligatoire dans la construction","631400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6318","Autres impôts, taxes et versements assimilés sur rémunérations (administrations des impôts)","631800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6331","Versement de transport","633100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6332","Allocation logement","633200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6333","Participation des employeurs à la formation professionnelle continue","633300","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6334","Participation des employeurs à l'effort de construction","633400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6335","Versements libératoires ouvrant droit à l'exonération de la taxe d'apprentissage","633500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6338","Autres impôts, taxes et versements assimilés sur rémunérations (autres organismes)","633800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_63511"," Contribution économique territoriale","635110","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_63512","Taxes foncières","635120","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_63513","Autres impôts locaux","635130","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_63514","Taxe sur les véhicules des sociétés","635140","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6352","Taxes sur le chiffre d'affaires non récupérables","635200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6353","Impôts indirects","635300","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_63541","Droits de mutation","635410","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6358","Autres droits","635800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6371","Contribution sociale de solidarité à la charge des sociétés","637100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6372","Taxes perçues par les organismes publics internationaux","637200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6374","Impôts et taxes exigibles à l'étranger","637400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6378","Taxes diverses (autres organismes)","637800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6411","Salaires et appointements","641100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6412","Congés payés","641200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6413","Primes et gratifications","641300","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6414","Indemnités et avantages divers","641400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6415","Supplément familial","641500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_644","Rémunération du travail de l'exploitant","644000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6451","Cotisations à l'URSSAF","645100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6452","Cotisations aux mutuelles","645200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6453","Cotisations aux caisses de retraites","645300","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6454","Cotisations aux ASSEDIC","645400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6458","Cotisations aux autres organismes sociaux","645800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_646","Cotisations sociales personnelles de l'exploitant","646000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6471","Prestations directes","647100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6472","Versements aux comités d'entreprise et d'établissement","647200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6473","Versements aux comités d'hygiène et de sécurité","647300","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6474","Versements aux autres oeuvres sociales","647400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6475","Médecine du travail, pharmacie","647500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_648","Autres charges de personnel","648000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6511","Redevances pour concessions brevets, licences, marques, procédés, logiciels","651100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6516","Droits d'auteur et de reproduction","651600","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6518","Autres droits et valeurs similaires","651800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_653","Jetons de présence","653000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6541","Créances de l'exercice","654100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6544","Créances des exercices antérieurs","654400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6551","Quote-part de bénéfice transférée (comptabilité du gérant)","655100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_6555","Quote-part de perte supportée (comptabilité des associés non gérants)","655500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_658","Charges diverses de gestion courante","658000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_66116","Intérêts des emprunts et dettes assimilées","661160","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_66117","Intérêts des dettes rattachées à des participations","661170","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_6612","Charges de la fiducie, résultat de la période","661200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_6615","Intérêts des comptes courants et des dépôts créditeurs","661500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_6616","Intérêts bancaires et sur opérations de financement (escompte, ...)","661600","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_6617","Intérêts des obligations cautionnées","661700","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_66181","Intérêts des dettes commerciales","661810","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_66188","Intérêts des dettes diverses","661880","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_664","Pertes sur créances liées à des participations","664000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_665","Escomptes accordés","665000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_666","Pertes de change","666000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_667","Charges nettes sur cessions de valeurs mobilières de placement","667000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_668","Autres charges financières","668000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_6711","Charges exceptionnelles - Pénalités sur marchés (et dédits payés sur achats et ventes)","671100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_6712","Charges exceptionnelles - Pénalités, amendes fiscales et pénales","671200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_6713","Charges exceptionnelles - Dons, libéralités","671300","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_6714","Charges exceptionnelles - Créances devenues irrécouvrables dans l'exercice","671400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_6715","Charges exceptionnelles - Subventions accordées","671500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_6717","Charges exceptionnelles - Rappels d'impôts (autres qu'impôts sur les bénéfices)","671700","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_6718","Autres charges exceptionnelles sur opération de gestion","671800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_672","Charges exceptionnelles sur exercices antérieurs (en cours d'exercice seulement)","672000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_6741","Opérations liées à la constitution de fiducie ","674100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_6742","Opérations liées à la liquidation de la fiducie","674200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_6751","Valeurs comptables des éléments d'actif cédés - Immobilisations incorporelles","675100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_6752","Valeurs comptables des éléments d'actif cédés - Immobilisations corporelles","675200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_6756","Valeurs comptables des éléments d'actif cédés - Immobilisations financières","675600","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_6758","Valeurs comptables des éléments d'actif cédés - Autres éléments d'actif","675800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_6781","Charges exceptionnelles - Malis provenant de clauses d'indexation","678100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_6782","Charges exceptionnelles - Lots","678200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_6783","Charges exceptionnelles - Malis provenant du rachat par l'entreprise d'actions et obligations émises par elle-même","678300","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_6788","Charges exceptionnelles diverses","678800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_6811"," Dotations aux amortissements sur immobilisations incorporelles et corporelles","681100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_68111","Dotations aux amortissements sur immobilisations incorporelles","681110","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_68112","Dotations aux amortissements sur immobilisations corporelles","681120","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6812","Dotations aux amortissements des charges d'exploitation à répartir","681200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6815","Dotations aux provisions pour risques et charges d'exploitation","681500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_68161","Dotations aux dépréciations des immobilisations incorporelles","681610","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_68162","Dotations aux dépréciations des immobilisations corporelles","681620","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_68173","Dotations aux dépréciations des stocks et en-cours","681730","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_68174","Dotations aux dépréciations des créances","681740","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_6861","Dotations aux amortissements des primes de remboursement des obligations","686100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_6865","Dotations aux provisions pour risques et charges financiers","686500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_68662","Dotations aux dépréciations des immobilisations financières","686620","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_68665","Dotations aux dépréciations des valeurs mobilières de placement","686650","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_6868","Autres dotations aux amortissements, dépréciations et provisions - Charges financières","686800","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_6871","Dotations aux amortissements exceptionnels des immobilisations","687100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_68725","Dotations aux provisions réglementées exceptionnelles (immobilisations) - Amortissements dérogatoires","687250","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_6873","Dotations aux provisions réglementées exceptionnelles (stocks)","687300","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_6874","Dotations aux autres provisions réglementées exceptionnelles","687400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_6875","Dotations aux provisions exceptionnelles","687500","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_6876","Dotations aux dépréciations exceptionnelles","687600","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_691","Participation des salariés aux résultats","691000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_6951","Impôts sur les bénéfices dus en France","695100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_6952","Contribution additionnelle à l'impôt sur les bénéfices","695200","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_6954","Impôts sur les bénéfices dus à l'étranger","695400","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_696","Supplément d'impôt sur les sociétés lié aux distributions","696000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_6981","Intégration fiscale - Charges","698100","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_6989","Intégration fiscale - Produits","698900","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_699","Produits, Reports en arrière des déficits","699000","account.data_account_type_expenses","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_7011","Ventes de produits finis (ou groupe) A","701100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7012","Ventes de produits finis (ou groupe) B","701200","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_702","Ventes de produits intermédiaires","702000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_703","Ventes de produits résiduels","703000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7041","Ventes de travaux de catégorie (ou activité) A","704100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7042","Ventes de travaux de catégorie (ou activité) B","704200","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_705","Ventes d'études","705000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_706","Ventes de prestations de services","706000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7071","Ventes de marchandises (ou groupe) A","707100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7072","Ventes de marchandises (ou groupe) B","707200","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7081","Produits des services exploités dans l'intérêt du personnel","708100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7082","Commissions et courtages","708200","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7083","Locations diverses","708300","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7084","Mise à disposition de personnel facturée","708400","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7085","Ports et frais accessoires facturés","708500","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7086","Bonis sur reprises d'emballages consignés","708600","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7087","Bonifications obtenues des clients et primes sur ventes","708700","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7088","Autres produits d'activités annexes (cessions d'approvisionnements...)","708800","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7091","Rabais, remises et ristournes sur ventes de produits finis","709100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7092","Rabais, remises et ristournes sur ventes de produits intermédiaires","709200","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7094","Rabais, remises et ristournes sur travaux","709400","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7095","Rabais, remises et ristournes sur études","709500","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7096","Rabais, remises et ristournes sur prestations de services","709600","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7097","Rabais, remises et ristournes sur ventes de marchandises","709700","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7098","Rabais, remises et ristournes sur produits des activités annexes","709800","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_71331","Variation des en-cours de production de biens - Produits en cours","713310","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_71335","Variation des en-cours de production de biens - Travaux en cours","713350","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_71341","Variation des en-cours de production de services - Études en cours","713410","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_71345","Variation des en-cours de production de services - Prestations de services en cours","713450","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_71351","Variation des stocks de produits intermédiaires","713510","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_71355","Variation des stocks de produits finis","713550","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_71358","Variation des stocks de produits résiduels","713580","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_721","Production immobilisée - Immobilisations incorporelles","721000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_722","Production immobilisée - Immobilisations corporelles","722000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_74","Subventions d'exploitation","740000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7511","Redevances pour concessions, brevets, licences, marques, procédés, logiciels","751100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7516","Droits d'auteur et de reproduction","751600","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7518","Redevances pour autres droits et valeurs similaires","751800","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_752","Revenus des immeubles non affectés aux activités professionnelles","752000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_753","Jetons de présence et rémunérations d'administrateurs, gérants..","753000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_754","Ristournes perçues des coopératives (provenant des excédents)","754000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7551","Quote-part de perte transférée (comptabilité du gérant)","755100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_7555","Quote-part de bénéfice attribuée (comptabilité des associés non-gérants)","755500","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_758","Produits divers de gestion courante","758000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7611","Revenus des titres de participation","761100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_7612","Produits de la fiducie, résultat de la période","761200","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_7616","Revenus sur autres formes de participation","761600","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_7617","Revenus des créances rattachées à des participations","761700","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_7621","Revenus des titres immobilisés","762100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_7626","Revenus des prêts","762600","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_7627","Revenus des créances immobilisées","762700","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_7631","Revenus des créances commerciales","763100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_7638","Revenus des créances diverses","763800","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_764","Revenus des valeurs mobilières de placement","764000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_765","Escomptes obtenus","765000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_766","Gains de change","766000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_767","Produits nets sur cessions de valeurs mobilières de placement","767000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_768","Autres produits financiers","768000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_7711","Produits exceptionnels - Dédits et pénalités perçus sur achats et sur ventes","771100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7713","Produits exceptionnels - Libéralités reçues","771300","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7714","Produits exceptionnels - Rentrées sur créances amorties","771400","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7715","Produits exceptionnels - Subventions d'équilibre","771500","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7717","Produits exceptionnels - Dégrèvements d'impôts autres qu'impôts sur les bénéfices","771700","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7718","Autres produits exceptionnels sur opérations de gestion","771800","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_772","Produits exceptionnels sur exercices antérieurs (en cours d'exercice seulement)","772000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_7741","Opérations liées à la constitution de fiducie - transfert des éléments","774100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_7742","Opérations liées à la liquidation de la fiducie","774200","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_7751","Produits exceptionnels des cessions d'éléments d'actif - Immobilisations incorporelles","775100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7752","Produits exceptionnels des cessions d'éléments d'actif - Immobilisations corporelles","775200","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7756","Produits exceptionnels des cessions d'éléments d'actif - Immobilisations financières","775600","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7758","Produits exceptionnels des cessions d'éléments d'actif - Autres éléments d'actif","775800","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_777","Produits exceptionnels - Quote-part des subventions d'investissement virée au résultat de l'exercice","777000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7781","Produits exceptionnels - Bonis provenant de clauses d'indexation","778100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7782","Produits exceptionnels - Lots","778200","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7783","Produits exceptionnels - Bonis provenant du rachat par l'entreprise d'actions et d'obligations émises par elle-même","778300","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7788","Produits exceptionnels divers","778800","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7811"," Reprises sur amortissements des immobilisations incorporelles et corporelles","781100","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_78111","Reprises sur amortissements des immobilisations incorporelles","781110","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_78112","Reprises sur amortissements des immobilisations corporelles","781120","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7815","Reprises sur provisions d'exploitation","781500","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7816"," Reprises sur dépréciations des immobilisations incorporelles et corporelles","781600","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_78161","Reprises sur dépréciations des immobilisations incorporelles","781610","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_78162","Reprises sur dépréciations des immobilisations corporelles","781620","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7817"," Reprises sur dépréciations des actifs circulants","781700","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","","False"
"pcg_78173","Reprises sur dépréciations des actifs circulants - Stocks et en-cours","781730","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_78174","Reprises sur dépréciations des actifs circulants - Créances","781740","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_7865","Reprises sur provisions financières","786500","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_78662","Reprises sur dépréciations des immobilisations financières","786620","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_78665","Reprises sur dépréciations des valeurs mobilières de placement","786650","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_78725","Reprises sur provisions réglementées (immobilisations) - Amortissements dérogatoires","787250","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_78726","Reprises sur provisions réglementées (immobilisations) - Provision spéciale de réévaluation","787260","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_78727","Reprises sur provisions réglementées (immobilisations) - Plus-values réinvesties","787270","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7873","Reprises sur provisions réglementées (stocks)","787300","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7874","Reprises sur autres provisions réglementées","787400","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7875","Reprises sur provisions exceptionnelles","787500","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_7876","Reprises sur dépréciations exceptionnelles","787600","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"
"pcg_791","Transferts de charges d'exploitation","791000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_operating","False"
"pcg_796","Transferts de charges financières","796000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_financing","False"
"pcg_797","Transferts de charges exceptionnelles","797000","account.data_account_type_revenue","l10n_fr.l10n_fr_pcg_chart_template","account.account_tag_investing","False"

```

## File: data\account.group.template.csv

```csv
"id","code_prefix_start","name","chart_template_id/id"
"pcg_10","10"," Capital et réserves","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_101","101"," Capital","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_104","104"," Primes liées au capital social","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_105","105"," Ecarts de réévaluation","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_106","106"," Réserves","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_13","13"," Subventions d'investissement","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_131","131","Subventions d'équipement","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_139","139"," Subventions d'investissement inscrites au compte de résultat","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_1391","1391"," Subventions d'équipement","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_14","14"," Provisions réglementées","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_142","142"," Provisions réglementées relatives aux immobilisations","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_143","143"," Provisions réglementées relatives aux stocks","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_15","15"," Provisions","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_151","151"," Provisions pour risques","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_158","158","Autres provisions pour charges","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_16","16"," Emprunts et dettes assimilées","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_165","165"," Dépôts et cautionnements reçus","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_166","166"," Participation des salariés aux résultats","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_167","167"," Emprunts et dettes assortis de conditions particulières","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_168","168"," Autres emprunts et dettes assimilées","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_17","17"," Dettes rattachées à des participations","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_178","178"," Dettes rattachées à des sociétés en participation","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_18","18"," Comptes de liaison des établissements et sociétés en participation ","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_20","20"," Immobilisations incorporelles","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_201","201"," Frais d'établissement","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_21","21"," Immobilisations corporelles","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_211","211"," Terrains","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_213","213"," Constructions","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_21318","21318"," Autres ensembles immobiliers","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_21358","21358"," Autres ensembles immobiliers","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_2138","2138"," Ouvrages d'infrastructure","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_215","215"," Installations techniques, matériels et outillage industriels","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_2151","2151"," Installations complexes spécialisées","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_2153","2153"," Installations à caractère spécifique","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_218","218"," Autres immobilisations corporelles","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_23","23"," Immobilisations en cours","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_231","231"," Immobilisations corporelles en cours","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_238","238"," Avances et acomptes versés sur commandes d'immobilisations corporelles","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_26","26"," Participations et créances rattachées à des participations","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_261","261"," Titres de participation","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_267","267"," Créances rattachées à des participations","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_268","268"," Créances rattachées à des sociétés en participation","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_27","27"," Autres immobilisations financières","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_271","271"," Titres immobilisés autres que les titres immobilisés de l'activité de portefeuille (droit de","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_272","272"," Titres immobilisés (droit de créance)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_274","274","Prêts","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_275","275"," Dépôts et cautionnements versés","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_276","276"," Autres créances immobilisées","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_2768","2768","Intérêts courus","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_277","277"," (Actions propres ou parts propres)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_28","28"," Amortissements des immobilisations incorporelles","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_281","281"," Amortissements des immobilisations corporelles","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_29","29"," Dépréciations des immobilisations incorporelles","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_291","291"," Dépréciations des immobilisations corporelles (même ventilation que celle","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_293","293"," Dépréciations des immobilisations en cours","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_296","296"," Dépréciations des participations et créances rattachées à des participations","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_297","297"," Dépréciations des autres immobilisations financières","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_31","31"," Matières premières (et fournitures)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_32","32"," Autres approvisionnements","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_321","321"," Matières consommables","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_322","322"," Fournitures consommables","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_326","326","Emballages","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_33","33"," En cours de production de biens","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_331","331"," Produits en cours","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_335","335"," Travaux en cours","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_34","34"," En cours de production de services","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_341","341"," Etudes en cours","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_345","345"," Prestations de services en cours","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_35","35"," Stocks de produits","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_351","351"," Produits intermédiaires","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_355","355"," Produits finis","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_358","358"," Produits résiduels (ou matières de récupération)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_37","37"," Stocks de marchandises","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_39","39"," Dépréciations des stocks et en cours","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_391","391"," Dépréciations des matières premières (et fournitures)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_392","392"," Dépréciations des autres approvisionnements","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_393","393"," Dépréciations des en cours de production de biens","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_394","394"," Dépréciations des en cours de production de services","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_395","395"," Dépréciations des stocks de produits","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_397","397"," Dépréciations des stocks de marchandises","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_40","40"," Fournisseurs et comptes rattachés ","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_401","401","Fournisseurs ","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_404","404","Fournisseurs d'immobilisations","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_408","408","Fournisseurs factures non parvenues","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_409","409"," Fournisseurs débiteurs","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_4097","4097"," Fournisseurs autres avoirs","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_411","411","Clients","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_418","418"," Clients  produits non encore facturés","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_419","419"," Clients créditeurs","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_42","42"," Personnel et comptes rattachés","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_424","424"," Participation des salariés aux résultats","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_428","428"," Personnel  charges à payer et produits à recevoir","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_43","43"," Sécurité sociale et autres organismes sociaux","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_438","438"," Organismes sociaux - charges à payer et produits à recevoir","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_44","44"," État et autres collectivités publiques","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_441","441"," État subvention à recevoir ","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_442","442"," Etat impots et taxes recouvrables sur des tiers","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_443","443"," Opérations particulières avec l'Etat les collectivités publiques, les organismes","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_445","445"," Etat taxes sur le chiffres d'affaires","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_4455","4455"," Taxes sur le chiffre d'affaires à décaisser","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_4456","4456"," Taxes sur le chiffre d'affaires déductibles","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_4457","4457"," Taxes sur le chiffre d'affaires collectées par l'entreprise","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_448","448"," Etat charges à payer et produits à recevoir","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_45","45"," Groupe et associés","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_455","455"," Associés comptes courants","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_456","456"," Associés opérations sur le capital","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_4561","4561","Associés comptes d'appport en société","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_4562","4562"," Apporteurs capital appelé, non versé ","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_458","458"," Associés opérations faites en commun et en GIE ","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_46","46"," Débiteurs divers et créditeurs divers","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_468","468"," Divers charges à payer et produit à recevoir","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_47","47"," Comptes transitoires ou d'attente","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_476","476"," Différence de conversion  actif","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_477","477"," Différences de conversion  passif","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_48","48"," Comptes de régularisation","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_481","481"," Charges à répartir sur plusieurs exercices","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_488","488"," Comptes de répartition périodique des charges et des produits","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_49","49"," Dépréciations des comptes de tiers","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_495","495"," Dépréciations des comptes du groupe et des associés","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_496","496"," Dépréciations des comptes de débiteurs divers","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_50","50"," Valeurs mobilières de placement","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_503","503"," Actions","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_506","506"," Obligations","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_508","508"," Autres valeurs mobilières de placement et autres créances assimilées","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_51","51"," Banques, établissements financiers et assimilés","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_511","511"," Valeurs à l'encaissement","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_512","512"," Banques","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_518","518"," Intérêts courus","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_519","519"," Concours bancaires courants","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_53","53"," Caisse","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_531","531"," Caisse siège social","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_58","58"," Virements internes","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_59","59"," Dépréciations des valeurs mobilières de placement","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_60","60"," Achats (sauf 603)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_601","601"," Achats stockés matières premières (et fournitures)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_602","602"," Achats stockés autres approvisionnements","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_6021","6021"," Matières consommables","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_6022","6022"," Fournitures consommables","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_6026","6026","Emballages","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_603","603"," Variations des stocks (approvisionnements et marchandises)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_606","606"," Achats non stockés de matière et fournitures","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_607","607"," Achats de marchandises","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_609","609"," Rabais, remises et ristournes obtenus sur achats","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_61","61"," Services extérieurs","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_612","612"," Redevances de crédit bail","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_615","615"," Entretien et réparations","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_616","616"," Primes d'assurances","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_6163","6163"," Assurance transport","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_618","618"," Divers","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_62","62"," Autres services extérieurs","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_621","621"," Personnel extérieur à l'entreprise","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_622","622"," Rémunérations d'intermédiaires et honoraires","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_623","623"," Publicité, publications, relations publiques","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_624","624"," Transports de biens et transports collectifs du personnel","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_625","625"," Déplacements, missions et réceptions","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_627","627"," Services bancaires et assimilés","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_628","628"," Divers","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_63","63","Impôts, taxes et versements assimilés","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_631","631"," Impôts, taxes et versements assimilés sur rémunérations (administrations des impôts)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_633","633"," Impôts, taxes et versements assimilés sur rémunérations (autres organismes)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_635","635"," Autres impôts, taxes et versements assimilés (administrations des impôts)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_6351","6351"," Impôts directs (sauf impôts sur les bénéfices)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_6354","6354"," Droits d'enregistrement et de timbre","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_637","637"," Autres impôts, taxes et versements assimilés (autres organismes)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_64","64"," Charges de personnel","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_641","641"," Rémunérations du personnel","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_645","645"," Charges de sécurité sociale et de prévoyance","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_647","647"," Autres charges sociales","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_65","65"," Autres charges de gestion courante","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_651","651"," Redevances pour concessions, brevets, licences, marques, procédés, logiciels, droits et valeurs simulaires","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_654","654"," Pertes sur créances irrécouvrables ","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_655","655"," Quote part de résultat sur opérations faites en commun","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_66","66"," Charges financières","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_661","661"," Charges d'intérêts","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_6611","6611"," Intérêts des emprunts et dettes","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_6618","6618"," Intérêts des autres dettes","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_67","67"," Charges exceptionnelles","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_671","671"," Charges exceptionnelles sur opérations de gestion","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_674","674","Opérations de constitution ou liquidation des fiducies","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_675","675"," Valeurs comptables des éléments d'actif cédés","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_678","678"," Autres charges exceptionnelles","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_68","68"," Dotations aux amortissements, aux dépréciations et aux provisions","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_681","681"," Dotations aux amortissements, aux dépréciations et aux provisions ","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_6816","6816"," Dotations pour dépréciations des immobilisations incorporelles et corporelles","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_6817","6817"," Dotations pour dépréciations des actifs circulants","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_686","686"," Dotations aux amortissements, aux dépréciations et aux provisions ","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_6866","6866"," Dotations pour dépréciations des éléments financiers","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_687","687"," Dotations aux amortissements, aux dépréciations et aux provisions ","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_6872","6872"," Dotations aux provisions réglementées (immobilisations)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_69","69"," Participation des salariés  impots sur les bénéfices et assimilés","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_695","695"," Impôts sur les bénéfices","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_698","698"," Intégration fiscale","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_70","70"," Ventes de produits fabriqués, prestations de services, marchandises","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_701","701"," Ventes de produits finis","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_704","704"," Travaux","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_707","707"," Ventes de marchandises","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_708","708"," Produits des activités annexes","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_709","709"," Rabais, remises et ristournes accordés par l'entreprise","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_71","71"," Production stockée (ou déstockage)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_713","713"," Variation des stocks (en cours de production, produits)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_7133","7133"," Variation des en cours de production de biens ","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_7134","7134"," Variation des en cours de production de services","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_7135","7135"," Variation des stocks de produits","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_72","72"," Production immobilisée","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_75","75"," Autres produits de gestion courante","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_751","751"," Redevances pour concessions, brevets, licences, marques, procédés, logiciels, droits et","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_755","755"," Quote part de résultat sur opérations faites en commun","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_76","76"," Produits financiers","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_761","761"," Produits de participations","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_762","762"," Produits des autres immobilisations financières","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_763","763"," Revenus des autres créances","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_77","77"," Produits exceptionnels","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_771","771"," Produits exceptionnels sur opérations de gestion","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_774","774","Opérations de constitution ou liquidation des fiducies","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_775","775"," Produits des cessions d'éléments d'actif","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_778","778"," Autres produits exceptionnels","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_78","78"," Reprises sur amortissements, dépréciations et provisions","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_781","781"," Reprises sur amortissements, dépréciations et provisions (à inscrire dans les produits d'exploitation)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_786","786"," Reprises sur provisions pour risques et dépréciations (à inscrire dans les produits financiers)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_7866","7866"," Reprises sur dépréciations des éléments financiers","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_787","787"," Reprises sur provisions et dépréciations (à inscrire dans les produits exceptionnels)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_7872","7872"," Reprises sur provisions réglementées (immobilisations)","l10n_fr.l10n_fr_pcg_chart_template"
"pcg_79","79"," Transferts de charges","l10n_fr.l10n_fr_pcg_chart_template"

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_fr.l10n_fr_pcg_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_fr_tag_salaires" model="account.account.tag">
        <field name="name">Salaires</field>
        <field name="applicability">accounts</field>
    </record>

      <record id="account_fr_tag_charges_sociales" model="account.account.tag">
        <field name="name">Charges Sociales</field>
        <field name="applicability">accounts</field>
    </record>

    <menuitem id="account_reports_fr_statements_menu" name="France" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>

    <record id="l10n_fr_pcg_chart_template" model="account.chart.template">
      <field name="property_account_receivable_id" ref="fr_pcg_recv"/>
      <field name="property_account_payable_id" ref="fr_pcg_pay"/>
      <field name="property_account_expense_categ_id" ref="pcg_6071"/>
      <field name="property_account_income_categ_id" ref="pcg_7071"/>
      <field name="income_currency_exchange_account_id" ref="pcg_766"/>
      <field name="expense_currency_exchange_account_id" ref="pcg_666"/>
      <field name="property_tax_payable_account_id" ref="pcg_44551"/>
      <field name="property_tax_receivable_account_id" ref="pcg_44567"/>
      <field name="default_pos_receivable_account_id" ref="fr_pcg_recv_pos" />
      <field name="currency_id" ref="base.EUR"/>
    </record>
</odoo>

```

## File: data\account_fiscal_position_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- To solve bug 1240265, we have to delete all fiscal position templates before each update.
         The valid ones will be re-created later during the update.
         /!\ This must be executed *before* loading the fiscal position templates!! -->
    <delete model="account.fiscal.position.template" search="[('chart_template_id','=',ref('l10n_fr_pcg_chart_template'))]"/>


    <!-- = = = = = = = = = = = = = = = -->
    <!-- Fiscal Position Templates     -->
    <!-- = = = = = = = = = = = = = = = -->

<!-- Position Géographique du partenaire -->
        <record id="fiscal_position_template_domestic" model="account.fiscal.position.template">
            <field name="sequence">1</field>
            <field name="name">Domestique - France</field>
            <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
            <field name="auto_apply" eval="True" />
            <field name="vat_required" eval="True" />
            <field name="country_group_id" ref="l10n_fr.fr_and_mc"></field>
        </record>
        <record id="fiscal_position_template_intraeub2c" model="account.fiscal.position.template">
            <field name="sequence">2</field>
            <field name="name">EU privé</field>
            <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
            <field name="auto_apply" eval="True" />
            <field name="country_group_id" ref="base.europe"></field>
        </record>
        <record id="fiscal_position_template_intraeub2b" model="account.fiscal.position.template">
            <field name="sequence">3</field>
            <field name="name">Intra-EU B2B</field>
            <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
            <field name="note">French VAT exemption according to articles 262 ter I (for products) and/or 283-2 (for services) of "CGI"</field>
            <field name="auto_apply" eval="True" />
            <field name="country_group_id" ref="base.europe"></field>
            <field name="vat_required" eval="True" />
        </record>
        <record id="fiscal_position_template_import_export" model="account.fiscal.position.template">
            <field name="sequence">50</field>
            <field name="name">Import/Export Hors Europe + DOM-TOM</field>
            <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
            <field name="note">French VAT exemption according to articles 291, 294 and 262 I of "CGI"</field>
            <field name="auto_apply" eval="True" />
        </record>

    <!-- = = = = = = = = = = = = = = = -->
    <!-- Fiscal Position Tax Templates -->
    <!-- = = = = = = = = = = = = = = = -->

<!-- Par défaut, les produits doivent être paramétrés pour utiliser les taxes, paramétrées pour des numéro de comptes (nationaux) -->

<!-- Zone Intracommunautaire B2B -->
<!-- ventes -->
<!-- Taux Normal -->
        <record id="fp_tax_template_intraeub2b_vt_normale" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_normale" />
            <field name="tax_dest_id" ref="tva_sale_good_intra_0" />
        </record>
        <record id="fp_tax_template_intraeub2b_vt_normale_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_normale_ttc" />
            <field name="tax_dest_id" ref="tva_sale_good_intra_0" />
        </record>
        <record id="fp_tax_template_intraeub2b_vt_intermediaire_encaissement" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_intermediaire_encaissement" />
            <field name="tax_dest_id" ref="tva_sale_service_intra_0" />
        </record>
        <record id="fp_tax_template_intraeub2b_vt_normale_encaissement" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_normale_encaissement" />
            <field name="tax_dest_id" ref="tva_sale_service_intra_0" />
        </record>
        <record id="fp_tax_template_intraeub2b_vt_normale_encaissement_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_normale_encaissement_ttc" />
            <field name="tax_dest_id" ref="tva_sale_service_intra_0" />
        </record>
        <record id="fp_tax_template_intraeub2b_vt_intermediaire_encaissement_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_intermediaire_encaissement_ttc" />
            <field name="tax_dest_id" ref="tva_sale_service_intra_0" />
        </record>
<!-- Taux DOM-TOM -->
        <record id="fp_tax_template_intraeub2b_vt_specifique" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_specifique" />
            <field name="tax_dest_id" ref="tva_sale_good_intra_0" />
        </record>
        <record id="fp_tax_template_intraeub2b_vt_specifique_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_specifique_ttc" />
            <field name="tax_dest_id" ref="tva_sale_good_intra_0" />
        </record>
<!-- Taux Intermédiaire -->
        <record id="fp_tax_template_intraeub2b_vt_intermediaire" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_intermediaire" />
            <field name="tax_dest_id" ref="tva_sale_good_intra_0" />
        </record>
        <record id="fp_tax_template_intraeub2b_vt_intermediaire_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_intermediaire_ttc" />
            <field name="tax_dest_id" ref="tva_sale_good_intra_0" />
        </record>
<!-- Taux réduit -->
        <record id="fp_tax_template_intraeub2b_vt_reduite" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_reduite" />
            <field name="tax_dest_id" ref="tva_sale_good_intra_0" />
        </record>
        <record id="fp_tax_template_intraeub2b_vt_reduite_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_reduite_ttc" />
            <field name="tax_dest_id" ref="tva_sale_good_intra_0" />
        </record>
        <record id="fp_tax_template_intraeub2b_vt_reduite_encaissement_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_reduite_encaissement_ttc" />
            <field name="tax_dest_id" ref="tva_sale_service_intra_0" />
        </record>
        <record id="fp_tax_template_intraeub2b_vt_reduite_encaissement" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_reduite_encaissement" />
            <field name="tax_dest_id" ref="tva_sale_service_intra_0" />
        </record>
<!-- Taux super réduit -->
        <record id="fp_tax_template_intraeub2b_vt_super_reduite" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_super_reduite" />
            <field name="tax_dest_id" ref="tva_sale_good_intra_0" />
        </record>
        <record id="fp_tax_template_intraeub2b_vt_super_reduite_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_super_reduite_ttc" />
            <field name="tax_dest_id" ref="tva_sale_good_intra_0" />
        </record>
        <record id="fp_tax_template_intraeub2b_vt_super_reduite_encaissement_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_super_reduite_encaissement_ttc" />
            <field name="tax_dest_id" ref="tva_sale_service_intra_0" />
        </record>
        <record id="fp_tax_template_intraeub2b_vt_super_reduite_encaissement" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_super_reduite_encaissement" />
            <field name="tax_dest_id" ref="tva_sale_service_intra_0" />
        </record>
<!-- achats -->
<!-- Taux Normal -->
        <record id="fp_tax_template_intraeub2b_ha_normale_deduc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_acq_normale" />
            <field name="tax_dest_id" ref="tva_intra_normale_biens" />
        </record>
<!-- Taux DOM-TOM -->
        <record id="fp_tax_template_intraeub2b_ha_specifique_deduc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_acq_specifique" />
            <field name="tax_dest_id" ref="tva_intra_specifique_biens" />
        </record>
        <record id="fp_tax_template_intraeub2b_ha_encaissement_deduc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_acq_encaissement" />
            <field name="tax_dest_id" ref="tva_intra_normale_services" />
        </record>
        <record id="fp_tax_template_intraeub2b_ha_encaissement_deduc_intermediaire" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_acq_intermediaire_encaissement" />
            <field name="tax_dest_id" ref="tva_intra_intermediaire_services" />
        </record>
<!-- Taux Intermédiaire -->
        <record id="fp_tax_template_intraeub2b_ha_intermediaire_deduc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_acq_intermediaire" />
            <field name="tax_dest_id" ref="tva_intra_intermediaire_biens" />
        </record>
<!-- Taux réduit -->
        <record id="fp_tax_template_intraeub2b_ha_reduite_deduc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_acq_reduite" />
            <field name="tax_dest_id" ref="tva_intra_reduite_biens" />
        </record>
        <record id="fp_tax_template_intraeub2b_ha_encaissement_reduite_deduc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_acq_encaissement_reduite" />
            <field name="tax_dest_id" ref="tva_intra_reduite_services" />
        </record>
<!-- Taux super réduit -->
        <record id="fp_tax_template_intraeub2b_ha_super_reduite_deduc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_acq_super_reduite" />
            <field name="tax_dest_id" ref="tva_intra_super_reduite_biens" />
        </record>
        <record id="fp_tax_template_intraeub2b_ha_encaissement_super_reduite_deduc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_intraeub2b" />
            <field name="tax_src_id" ref="tva_acq_encaissement_super_reduite" />
            <field name="tax_dest_id" ref="tva_intra_super_reduite_services" />
        </record>

<!-- Import/Export + DOM/TOM -->
<!-- ventes -->
<!-- Taux Normal -->
        <record id="fp_tax_template_impexp_vt_normale" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_normale" />
            <field name="tax_dest_id" ref="tva_sale_good_export_0" />
        </record>
        <record id="fp_tax_template_impexp_vt_normale_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_normale_ttc" />
            <field name="tax_dest_id" ref="tva_sale_good_export_0" />
        </record>
        <record id="fp_tax_template_impexp_vt_normale_encaissement" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_normale_encaissement" />
            <field name="tax_dest_id" ref="tva_sale_service_export_0" />
        </record>
        <record id="fp_tax_template_impexp_vt_intermediaire_encaissement" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_intermediaire_encaissement" />
            <field name="tax_dest_id" ref="tva_sale_service_export_0" />
        </record>
        <record id="fp_tax_template_impexp_vt_normale_encaissement_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_normale_encaissement_ttc" />
            <field name="tax_dest_id" ref="tva_sale_service_export_0" />
        </record>
        <record id="fp_tax_template_impexp_vt_intermediaire_encaissement_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_intermediaire_encaissement_ttc" />
            <field name="tax_dest_id" ref="tva_sale_service_export_0" />
        </record>
<!-- Taux DOM-TOM -->
        <record id="fp_tax_template_impexp_vt_specifique" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_specifique" />
            <field name="tax_dest_id" ref="tva_sale_good_export_0" />
        </record>
        <record id="fp_tax_template_impexp_vt_specifique_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_specifique_ttc" />
            <field name="tax_dest_id" ref="tva_sale_good_export_0" />
        </record>
<!-- Taux Intermédiare -->
        <record id="fp_tax_template_impexp_vt_intermediaire" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_intermediaire" />
            <field name="tax_dest_id" ref="tva_sale_good_export_0" />
        </record>
        <record id="fp_tax_template_impexp_vt_intermediaire_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_intermediaire_ttc" />
            <field name="tax_dest_id" ref="tva_sale_good_export_0" />
        </record>
<!-- Taux Réduit -->
        <record id="fp_tax_template_impexp_vt_reduite" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_reduite" />
            <field name="tax_dest_id" ref="tva_sale_good_export_0" />
        </record>
        <record id="fp_tax_template_impexp_vt_reduite_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_reduite_ttc" />
            <field name="tax_dest_id" ref="tva_sale_good_export_0" />
        </record>
        <record id="fp_tax_template_impexp_vt_reduite_encaissement_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_reduite_encaissement_ttc" />
            <field name="tax_dest_id" ref="tva_sale_service_export_0" />
        </record>
        <record id="fp_tax_template_impexp_vt_reduite_encaissement" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_reduite_encaissement" />
            <field name="tax_dest_id" ref="tva_sale_service_export_0" />
        </record>
<!-- Taux super réduit -->
        <record id="fp_tax_template_impexp_vt_super_reduite" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_super_reduite" />
            <field name="tax_dest_id" ref="tva_sale_good_export_0" />
        </record>
        <record id="fp_tax_template_impexp_vt_super_reduite_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_super_reduite_ttc" />
            <field name="tax_dest_id" ref="tva_sale_good_export_0" />
        </record>
        <record id="fp_tax_template_impexp_vt_super_reduite_encaissement_ttc" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_super_reduite_encaissement_ttc" />
            <field name="tax_dest_id" ref="tva_sale_service_export_0" />
        </record>
        <record id="fp_tax_template_impexp_vt_super_reduite_encaissement" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_super_reduite_encaissement" />
            <field name="tax_dest_id" ref="tva_sale_service_export_0" />
        </record>
<!-- achats -->
<!-- Taux Normal -->
        <record id="fp_tax_template_impexp_ha_normale" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_acq_normale" />
            <field name="tax_dest_id" ref="tva_import_outside_eu_20" />
        </record>
<!-- Taux DOM-TOM -->
        <record id="fp_tax_template_impexp_ha_specifique" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_acq_specifique" />
            <field name="tax_dest_id" ref="tva_import_outside_eu_8_5" />
        </record>
<!-- Taux Intermédiare -->
        <record id="fp_tax_template_impexp_ha_intermediaire" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_acq_intermediaire" />
            <field name="tax_dest_id" ref="tva_import_outside_eu_10" />
        </record>
<!-- Taux Réduit -->
        <record id="fp_tax_template_impexp_ha_reduite" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_acq_reduite" />
            <field name="tax_dest_id" ref="tva_import_outside_eu_5_5" />
        </record>
<!-- Taux super réduit -->
        <record id="fp_tax_template_impexp_ha_super_reduite" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_import_export" />
            <field name="tax_src_id" ref="tva_acq_super_reduite" />
            <field name="tax_dest_id" ref="tva_import_outside_eu_2_1" />
        </record>
</odoo>

```

## File: data\account_reconcile_model_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
  @author: Alexis de Lattre <alexis.delattre@akretion.com>
-->
<odoo>
    <record id="bank_charges_reconcile_model" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">Frais bancaires</field>
    </record>
    <record id="bank_charges_reconcile_model_line" model="account.reconcile.model.line.template">
        <field name="model_id" ref="l10n_fr.bank_charges_reconcile_model"/>
        <field name="account_id" ref="pcg_6278"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
    </record>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- The taxes are sorted in the file: Type of tax > Scope of tax > Tax Due

         * PURCHASE
            - Goods (Based on Invoice)
            - Services (Based on Invoice)
            - Services (Based on Payment)
         * SALE
            - Goods (Based on Invoice)
            - Services (Based Payment)
    -->



    <!-- ########### PURCHASE, Goods (Based on Invoice) ########### -->
    <record model="account.tax.template" id="tva_acq_normale">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20%</field>
        <field name="description">TVA 20%</field>
        <field name="amount" eval="20.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="9"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acq_specifique">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 8,5%</field>
        <field name="description">TVA 8,5%</field>
        <field name="amount" eval="8.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_85"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acq_intermediaire">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 10%</field>
        <field name="description">TVA 10%</field>
        <field name="amount" eval="10.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acq_reduite">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 5,5%</field>
        <field name="description">TVA 5,5%</field>
        <field name="amount" eval="5.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_55"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acq_super_reduite">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 2,1%</field>
        <field name="description">TVA 2,1%</field>
        <field name="amount" eval="2.1"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <!-- CARBURANT -->
    <record model="account.tax.template" id="tva_purchase_good_fuel">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20% CARBURANT</field>
        <field name="description">TVA 20%</field>
        <field name="amount" eval="20.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="9"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 80,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),

            (0,0, {
                'factor_percent': 20,
                'repartition_type': 'tax',
                'plus_report_line_ids': [],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 80,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),

            (0,0, {
                'factor_percent': 20,
                'repartition_type': 'tax',
                'minus_report_line_ids': [],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_purchase_good_fuel_TTC">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20% CARBURANT TTC</field>
        <field name="description">TVA 20%</field>
        <field name="amount" eval="20.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="9"/>
        <field name="price_include" eval="1"/>
        <field name="include_base_amount" eval="1"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 80,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),

            (0,0, {
                'factor_percent': 20,
                'repartition_type': 'tax',
                'plus_report_line_ids': [],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 80,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),

            (0,0, {
                'factor_percent': 20,
                'repartition_type': 'tax',
                'minus_report_line_ids': [],
            }),
        ]"/>
    </record>

    <!-- TTC -->
    <record model="account.tax.template" id="tva_acq_normale_TTC">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20% TTC</field>
        <field name="description">TVA 20%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="20.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acq_specifique_TTC">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 8,5% TTC</field>
        <field name="description">TVA 8,5%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="8.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="tax_group_id" ref="tax_group_tva_85"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acq_intermediaire_TTC">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 10% TTC</field>
        <field name="description">TVA 10%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="10.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="tax_group_id" ref="tax_group_tva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acq_reduite_TTC">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 5,5% TTC</field>
        <field name="description">TVA 5,5%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="5.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="tax_group_id" ref="tax_group_tva_55"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acq_super_reduite_TTC">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 2,1% TTC</field>
        <field name="description">TVA 2,1%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="2.1"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="tax_group_id" ref="tax_group_tva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <!-- IMMO -->
    <record model="account.tax.template" id="tva_imm_normale">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20% IMMO</field>
        <field name="description">TVA 20%</field>
        <field name="amount" eval="20.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_19')],
                'account_id': ref('pcg_44562'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_19')],
                'account_id': ref('pcg_44562'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_imm_specifique">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 8,5% IMMO</field>
        <field name="description">TVA 8,5%</field>
        <field name="amount" eval="8.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_85"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_19')],
                'account_id': ref('pcg_44562'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_19')],
                'account_id': ref('pcg_44562'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_imm_intermediaire">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 10% IMMO</field>
        <field name="description">TVA 10%</field>
        <field name="amount" eval="10.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_19')],
                'account_id': ref('pcg_44562'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_19')],
                'account_id': ref('pcg_44562'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_imm_reduite">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 5,5% IMMO</field>
        <field name="description">TVA 5,5%</field>
        <field name="amount" eval="5.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_55"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_19')],
                'account_id': ref('pcg_44562'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_19')],
                'account_id': ref('pcg_44562'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_imm_super_reduite">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 2,1% IMMO</field>
        <field name="description">TVA 2,1%</field>
        <field name="amount" eval="2.1"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_19')],
                'account_id': ref('pcg_44562'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_19')],
                'account_id': ref('pcg_44562'),
            }),
        ]"/>
    </record>

    <!-- IMPORT -->
    <record model="account.tax.template" id="tva_import_outside_eu_20">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20% IMPORT</field>
        <field name="description">TVA 20%</field>
        <field name="amount" eval="20.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="11"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A4'), ref('l10n_fr.tax_report_I1_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20'), ref('l10n_fr.tax_report_24')],
                'account_id': ref('pcg_445663'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_I1_taxe')],
                'account_id': ref('pcg_4453'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A4'), ref('l10n_fr.tax_report_I1_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20'), ref('l10n_fr.tax_report_24')],
                'account_id': ref('pcg_445663'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_I1_taxe')],
                'account_id': ref('pcg_4453'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_import_outside_eu_10">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 10% IMPORT</field>
        <field name="description">TVA 10%</field>
        <field name="amount" eval="10.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="11"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A4'), ref('l10n_fr.tax_report_I2_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20'), ref('l10n_fr.tax_report_24')],
                'account_id': ref('pcg_445663'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_I2_taxe')],
                'account_id': ref('pcg_4453'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A4'), ref('l10n_fr.tax_report_I2_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20'), ref('l10n_fr.tax_report_24')],
                'account_id': ref('pcg_445663'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_I2_taxe')],
                'account_id': ref('pcg_4453'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_import_outside_eu_8_5">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 8,5% IMPORT</field>
        <field name="description">TVA 8,5%</field>
        <field name="amount" eval="8.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="11"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_85"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A4'), ref('l10n_fr.tax_report_I3_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20'), ref('l10n_fr.tax_report_24')],
                'account_id': ref('pcg_445663'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_I3_taxe')],
                'account_id': ref('pcg_4453'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A4'), ref('l10n_fr.tax_report_I3_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20'), ref('l10n_fr.tax_report_24')],
                'account_id': ref('pcg_445663'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_I3_base')],
                'account_id': ref('pcg_4453'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_import_outside_eu_5_5">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 5,5% IMPORT</field>
        <field name="description">TVA 5,5%</field>
        <field name="amount" eval="5.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="11"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_55"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A4'), ref('l10n_fr.tax_report_I4_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20'), ref('l10n_fr.tax_report_24')],
                'account_id': ref('pcg_445663'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_I4_taxe')],
                'account_id': ref('pcg_4453'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A4'), ref('l10n_fr.tax_report_I4_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20'), ref('l10n_fr.tax_report_24')],
                'account_id': ref('pcg_445663'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_I4_taxe')],
                'account_id': ref('pcg_4453'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_import_outside_eu_2_1">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 2,1% IMPORT</field>
        <field name="description">TVA 2,1%</field>
        <field name="amount" eval="2.1"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="11"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A4'), ref('l10n_fr.tax_report_I5_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20'), ref('l10n_fr.tax_report_24')],
                'account_id': ref('pcg_445663'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_I5_taxe')],
                'account_id': ref('pcg_4453'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A4'), ref('l10n_fr.tax_report_I5_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20'), ref('l10n_fr.tax_report_24')],
                'account_id': ref('pcg_445663'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_I5_taxe')],
                'account_id': ref('pcg_4453'),
            }),
        ]"/>
    </record>

    <!-- INTRACOM -->
    <record model="account.tax.template" id="tva_intra_normale_biens">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20% EU M</field>
        <field name="description">TVA 20%</field>
        <field name="amount" eval="20.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_B2'), ref('l10n_fr.tax_report_08_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_08_taxe'), ref('l10n_fr.tax_report_17')],
                'account_id': ref('pcg_4452'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_B2'), ref('l10n_fr.tax_report_08_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_08_taxe'), ref('l10n_fr.tax_report_17')],
                'account_id': ref('pcg_4452'),
            }),
        ]"/>
    </record>



    <!-- ########### PURCHASE, Services (Based on Invoice) ########### -->
    <record model="account.tax.template" id="tva_intra_normale_services">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20% EU S</field>
        <field name="description">TVA 20%</field>
        <field name="amount" eval="20.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A3'), ref('l10n_fr.tax_report_08_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_08_taxe')],
                'account_id': ref('pcg_44521'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A3'), ref('l10n_fr.tax_report_08_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_08_taxe')],
                'account_id': ref('pcg_44521'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_purchase_service_20_import">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20% IMPORT</field>
        <field name="description">TVA 20%</field>
        <field name="amount" eval="20.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_B4'), ref('l10n_fr.tax_report_08_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445663'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_08_taxe')],
                'account_id': ref('pcg_44531'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_B4'), ref('l10n_fr.tax_report_08_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445663'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_08_taxe')],
                'account_id': ref('pcg_44531'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_purchase_service_0">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 0% EXO</field>
        <field name="description">TVA 0%</field>
        <field name="amount" eval="0.00"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="tax_exigibility">on_invoice</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44574"/>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
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
                'minus_report_line_ids': [],
            }),
           (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>


    <!-- ########### PURCHASE, Services (Based on Payment) ########### -->
    <record model="account.tax.template" id="tva_acq_encaissement">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20%</field>
        <field name="description">TVA 20%</field>
        <field name="amount" eval="20.0"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44564" />
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="include_base_amount" eval="1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acq_intermediaire_encaissement">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 10%</field>
        <field name="description">TVA 10%</field>
        <field name="amount" eval="10.0"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44564" />
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="tax_group_id" ref="tax_group_tva_10"/>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acq_encaissement_reduite">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 5,5%</field>
        <field name="description">TVA 5,5%</field>
        <field name="amount" eval="5.5"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44564" />
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="tax_group_id" ref="tax_group_tva_55"/>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acq_encaissement_super_reduite">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 2,1% </field>
        <field name="description">TVA 2,1%</field>
        <field name="amount" eval="2.1"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44564" />
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="tax_group_id" ref="tax_group_tva_21"/>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <!-- TTC -->
    <record model="account.tax.template" id="tva_acq_encaissement_TTC">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20% TTC</field>
        <field name="description">TVA 20%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="20.0"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44564" />
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acq_intermediaire_encaissement_TTC">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 10% TTC</field>
        <field name="description">TVA 10%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="10.0"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44564" />
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acq_encaissement_reduite_TTC">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 5,5% TTC</field>
        <field name="description">TVA 5,5%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="5.5"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44564"/>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_55"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acq_encaissement_super_reduite_TTC">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 2,1% TTC</field>
        <field name="description">TVA 2,1%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="2.1"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="active" eval="0"/>
        <field name="cash_basis_transition_account_id" ref="pcg_44564"/>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="tax_group_id" ref="tax_group_tva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44566'),
            }),
        ]"/>
    </record>

    <!-- Others -->
    <record model="account.tax.template" id="tva_purchase_imm_normale">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20% IMMO</field>
        <field name="description">TVA 20%</field>
        <field name="amount" eval="20.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44564" />
        <field name="tax_scope">service</field>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="include_base_amount" eval="1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_19')],
                'account_id': ref('pcg_44562'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_19')],
                'account_id': ref('pcg_44562'),
            }),
        ]"/>
    </record>


    <!-- ########### SALE, Goods (Based on Invoice) ########### -->
    <record model="account.tax.template" id="tva_normale">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20%</field>
        <field name="description">TVA 20%</field>
        <field name="amount" eval="20.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_08_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_08_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_08_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_08_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_intermediaire">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 10%</field>
        <field name="description">TVA 10%</field>
        <field name="amount" eval="10.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="0"/>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_9B_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_9B_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_9B_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_9B_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_reduite">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 5,5%</field>
        <field name="description">TVA 5,5%</field>
        <field name="amount" eval="5.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="0"/>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_55"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_09_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_09_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_09_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_09_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_specifique">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 8,5%</field>
        <field name="description">TVA 8,5%</field>
        <field name="amount" eval="8.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_85"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_10_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_10_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_10_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_10_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_super_reduite">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 2,1%</field>
        <field name="description">TVA 2,1%</field>
        <field name="amount" eval="2.1"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_11_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_11_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_11_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_11_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <!-- TTC -->
    <record model="account.tax.template" id="tva_normale_ttc">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20% TTC</field>
        <field name="description">TVA 20%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="20"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="active" eval="0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_08_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_08_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_08_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_08_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_intermediaire_ttc">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 10% TTC</field>
        <field name="description">TVA 10%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="10"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_9B_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_9B_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_9B_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_9B_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_specifique_ttc">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 8,5% TTC</field>
        <field name="description">TVA 8,5%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="8.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_85"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_10_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_10_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_10_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_10_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_reduite_ttc">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 5,5% TTC</field>
        <field name="description">TVA 5,5%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="5.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_55"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_09_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_09_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_09_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_09_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_super_reduite_ttc">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 2,1% TTC</field>
        <field name="description">TVA 2,1%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="2.1"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_11_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_11_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_11_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_11_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <!-- Others (EXO, EXPORT, INTRACOM) -->
    <record model="account.tax.template" id="tva_sale_good_0">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 0% EXO</field>
        <field name="description">TVA 0%</field>
        <field name="amount" eval="0.00"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_E2')],
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
                'minus_report_line_ids': [ref('l10n_fr.tax_report_E2')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>  <!-- previous id: tva_0 -->

    <record model="account.tax.template" id="tva_sale_good_export_0">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 0% EXPORT</field>
        <field name="description">TVA 0%</field>
        <field name="amount" eval="0.00"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_E1')],
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
                'minus_report_line_ids': [ref('l10n_fr.tax_report_E1')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>  <!-- previous id: tva_export_0 -->

    <record model="account.tax.template" id="tva_sale_good_intra_0">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 0% EU M</field>
        <field name="description">TVA 0%</field>
        <field name="amount" eval="0.00"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_F2')],
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
                'minus_report_line_ids': [ref('l10n_fr.tax_report_F2')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>  <!-- previous id: tva_intra_0 -->


    <!-- ########### SALE, Services (Based Payment) ########### -->
    <record model="account.tax.template" id="tva_normale_encaissement">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20%</field>
        <field name="description">TVA 20%</field>
        <field name="amount" eval="20.0"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44574"/>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">service</field>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_08_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_08_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_08_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_08_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_intermediaire_encaissement">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 10%</field>
        <field name="description">TVA 10%</field>
        <field name="amount" eval="10.0"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44574"/>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">service</field>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_10"/>
        <field name="include_base_amount" eval="1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_9B_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_9B_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_9B_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_9B_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_reduite_encaissement">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 5,5%</field>
        <field name="description">TVA 5,5%</field>
        <field name="amount" eval="5.5"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44574" />
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">service</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_55"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_09_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_09_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_09_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_09_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_super_reduite_encaissement">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 2,1%</field>
        <field name="description">TVA 2,1%</field>
        <field name="amount" eval="2.1"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44574" />
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">service</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_11_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_11_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_11_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_11_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <!-- TTC -->
    <record model="account.tax.template" id="tva_normale_encaissement_ttc">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 20% TTC</field>
        <field name="description">TVA 20%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="20"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_445800" />
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">service</field>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_08_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_08_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_08_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_08_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_intermediaire_encaissement_ttc">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 10% TTC</field>
        <field name="description">TVA 10%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="10"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_445800" />
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">service</field>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_9B_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_9B_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_9B_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_9B_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_reduite_encaissement_ttc">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 5,5% TTC</field>
        <field name="description">TVA 5,5%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="5.5"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_445800" />
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">service</field>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_55"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_09_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_09_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_09_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_09_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_super_reduite_encaissement_ttc">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 2,1% TTC</field>
        <field name="description">TVA 2,1%</field>
        <field name="price_include" eval="1"/>
        <field name="amount" eval="2.1"/>
        <field name="amount_type">percent</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_445800" />
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">service</field>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_11_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_11_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A1'), ref('l10n_fr.tax_report_11_base')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_11_taxe')],
                'account_id': ref('pcg_44571'),
            }),
        ]"/>
    </record>

    <!-- Others (EXO, EXPORT, INTRACOM) -->
    <record model="account.tax.template" id="tva_sale_service_0">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 0% EXO</field>
        <field name="description">TVA 0%</field>
        <field name="amount" eval="0.00"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">service</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44574"/>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_E2')],
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
                'minus_report_line_ids': [ref('l10n_fr.tax_report_E2')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>  <!-- previous id: tva_0 -->

    <record model="account.tax.template" id="tva_sale_service_export_0">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 0% EXPORT</field>
        <field name="description">TVA 0%</field>
        <field name="amount" eval="0.00"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">service</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44574"/>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_E2')],
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
                'minus_report_line_ids': [ref('l10n_fr.tax_report_E2')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>  <!-- previous id: tva_export_0 -->

    <record model="account.tax.template" id="tva_sale_service_intra_0">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 0% EU S</field>
        <field name="description">TVA 0%</field>
        <field name="amount" eval="0.00"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_scope">service</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44574"/>
        <field name="include_base_amount" eval="1"/>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_E2')],
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
                'minus_report_line_ids': [ref('l10n_fr.tax_report_E2')],
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>  <!-- previous id: tva_intra_0 -->


    <!-- ACHATS INTRACOMMUNAUTAIRE -->
    <record model="account.tax.template" id="tva_intra_specifique_biens">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 8,5% EU M</field>
        <field name="description">TVA 8,5%</field>
        <field name="amount" eval="8.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_85"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_B2'), ref('l10n_fr.tax_report_10_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_10_taxe'), ref('l10n_fr.tax_report_17')],
                'account_id': ref('pcg_4452'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_B2'), ref('l10n_fr.tax_report_10_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_10_taxe'), ref('l10n_fr.tax_report_17')],
                'account_id': ref('pcg_4452'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_intra_specifique_services">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 8,5% EU S</field>
        <field name="description">TVA 8,5%</field>
        <field name="amount" eval="8.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_85"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A3'),
                                         ref('l10n_fr.tax_report_10_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_17'),
                                         ref('l10n_fr.tax_report_10_taxe')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44521'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A3'),
                                          ref('l10n_fr.tax_report_10_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_17'),
                                          ref('l10n_fr.tax_report_10_taxe')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_44521'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_intra_intermediaire_biens">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 10% EU M</field>
        <field name="description">TVA 10%</field>
        <field name="amount" eval="10.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_B2'), ref('l10n_fr.tax_report_9B_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_9B_taxe'), ref('l10n_fr.tax_report_17')],
                'account_id': ref('pcg_4452'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_B2'), ref('l10n_fr.tax_report_9B_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_9B_taxe'), ref('l10n_fr.tax_report_17')],
                'account_id': ref('pcg_4452'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_intra_intermediaire_services">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 10% EU S</field>
        <field name="description">TVA 10%</field>
        <field name="amount" eval="10.0"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A3'), ref('l10n_fr.tax_report_9B_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_9B_taxe')],
                'account_id': ref('pcg_44521'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A3'), ref('l10n_fr.tax_report_9B_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_9B_taxe')],
                'account_id': ref('pcg_44521'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_intra_reduite_biens">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 5,5% EU M</field>
        <field name="description">TVA 5,5%</field>
        <field name="amount" eval="5.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_55"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_B2'), ref('l10n_fr.tax_report_09_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_09_taxe'), ref('l10n_fr.tax_report_17')],
                'account_id': ref('pcg_4452'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_B2'), ref('l10n_fr.tax_report_09_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_09_taxe'), ref('l10n_fr.tax_report_17')],
                'account_id': ref('pcg_4452'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_intra_reduite_services">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 5,5% EU S</field>
        <field name="description">TVA 5,5%</field>
        <field name="amount" eval="5.5"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_55"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A3'), ref('l10n_fr.tax_report_09_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_09_taxe')],
                'account_id': ref('pcg_44521'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A3'), ref('l10n_fr.tax_report_09_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_09_taxe')],
                'account_id': ref('pcg_44521'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_intra_super_reduite_biens">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 2,1% EU M</field>
        <field name="description">TVA 2,1%</field>
        <field name="amount" eval="2.1"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_B2'), ref('l10n_fr.tax_report_11_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_11_taxe'), ref('l10n_fr.tax_report_17')],
                'account_id': ref('pcg_4452'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_B2'), ref('l10n_fr.tax_report_11_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_11_taxe'), ref('l10n_fr.tax_report_17')],
                'account_id': ref('pcg_4452'),
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_intra_super_reduite_services">
        <field name="chart_template_id" ref="l10n_fr_pcg_chart_template"/>
        <field name="name">TVA 2,1% EU S</field>
        <field name="description">TVA 2,1%</field>
        <field name="amount" eval="2.1"/>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="10"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="include_base_amount" eval="1"/>
        <field name="active" eval="0"/>
        <field name="tax_group_id" ref="tax_group_tva_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_A3'), ref('l10n_fr.tax_report_11_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_11_taxe')],
                'account_id': ref('pcg_44521'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_A3'), ref('l10n_fr.tax_report_11_base')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('l10n_fr.tax_report_20')],
                'account_id': ref('pcg_445662'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('l10n_fr.tax_report_11_taxe')],
                'account_id': ref('pcg_44521'),
            }),
        ]"/>
    </record>

</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_tva_0" model="account.tax.group">
            <field name="name">TVA 0%</field>
            <field name="country_id" ref="base.fr"/>
        </record>
        <record id="tax_group_tva_20" model="account.tax.group">
            <field name="name">TVA 20%</field>
            <field name="country_id" ref="base.fr"/>
        </record>
        <record id="tax_group_tva_85" model="account.tax.group">
            <field name="name">TVA 8.5%</field>
            <field name="country_id" ref="base.fr"/>
        </record>
        <record id="tax_group_tva_55" model="account.tax.group">
            <field name="name">TVA 5.5%</field>
            <field name="country_id" ref="base.fr"/>
        </record>
        <record id="tax_group_tva_10" model="account.tax.group">
            <field name="name">TVA 10%</field>
            <field name="country_id" ref="base.fr"/>
        </record>
        <record id="tax_group_tva_21" model="account.tax.group">
            <field name="name">TVA 2.1%</field>
            <field name="country_id" ref="base.fr"/>
        </record>
    </data>

    <data noupdate="0">
        <record id="display_name_in_footer_param" model="ir.config_parameter">
            <field name="key">account.display_name_in_footer</field>
            <field name="value" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\l10n_fr_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
  	<!-- Chart template -->
    <record id="l10n_fr_pcg_chart_template" model="account.chart.template">
        <field name="name">Plan Comptable Général (France)</field>
        <field name="currency_id" ref="base.EUR"/>
        <field name="code_digits" eval="6"/>
        <field name="bank_account_code_prefix">512</field>
        <field name="cash_account_code_prefix">53</field>
        <field name="transfer_account_code_prefix">58</field>
        <field name="complete_tax_set" eval="True" />
        <field name="country_id" ref="base.fr"/>
    </record>
</odoo>

```

## File: data\res_country_data.xml

```xml
<odoo>
    <record id="dom-tom" model="res.country.group">
             <field name="name">DOM-TOM</field>
             <field name="country_ids" eval="[(6,0,[
                                                ref('base.yt'),
                                                ref('base.gp'),
                                                ref('base.mq'),
                                                ref('base.gf'),
                                                ref('base.re'),
                                                ref('base.pf'),
                                                ref('base.pm'),
                                                ref('base.mf'),
                                                ref('base.bl'),
                                                ref('base.nc')])]"/>
       </record>

    <record id="fr_and_mc" model="res.country.group">
        <field name="name">France and Monaco</field>
        <field name="country_ids" eval="[Command.set([ref('base.fr'), ref('base.mc')])]"/>
    </record>
</odoo>

```

## File: data\tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.fr"/>
    </record>

    <record id="tax_report_montant_op_realisees" model="account.tax.report.line">
        <field name="name">A. Montant des opérations réalisées</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
        <field name="formula">None</field>
    </record>

    <!-- OPÉRATIONS TAXÉES (HT.) -->
    <record id="tax_report_op_imposables_ht" model="account.tax.report.line">
        <field name="name">Opérations imposables (H.T.)</field>
        <field name="parent_id" ref="tax_report_montant_op_realisees"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_A1" model="account.tax.report.line">
        <field name="name">A1 - Ventes, prestations de services</field>
        <field name="tag_name">A1</field>
        <field name="code">box_A1</field>
        <field name="parent_id" ref="tax_report_op_imposables_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_A2" model="account.tax.report.line">
        <field name="name">A2 - Autres opérations imposables</field>
        <field name="tag_name">A2</field>
        <field name="code">box_A2</field>
        <field name="parent_id" ref="tax_report_op_imposables_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_A3" model="account.tax.report.line">
        <field name="name">A3 - Achats de prestations de services intracommunautaires</field>
        <field name="tag_name">A3</field>
        <field name="code">box_A3</field>
        <field name="parent_id" ref="tax_report_op_imposables_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_A4" model="account.tax.report.line">
        <field name="name">A4 - Importations (autres que les produits pétroliers)</field>
        <field name="tag_name">A4</field>
        <field name="code">box_A4</field>
        <field name="parent_id" ref="tax_report_op_imposables_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
    </record>

    <record id="tax_report_A5" model="account.tax.report.line">
        <field name="name">A5 - Sorties de régime fiscal suspensif (autres que les produits pétroliers)</field>
        <field name="tag_name">A5</field>
        <field name="code">box_A5</field>
        <field name="parent_id" ref="tax_report_op_imposables_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">5</field>
    </record>

    <record id="tax_report_B1" model="account.tax.report.line">
        <field name="name">B1 - Mises à la consommation de produits pétroliers</field>
        <field name="tag_name">B1</field>
        <field name="code">box_B1</field>
        <field name="parent_id" ref="tax_report_op_imposables_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">6</field>
    </record>

    <record id="tax_report_B2" model="account.tax.report.line">
        <field name="name">B2 - Acquisitions intracommunautaires</field>
        <field name="tag_name">B2</field>
        <field name="code">box_B2</field>
        <field name="parent_id" ref="tax_report_op_imposables_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">7</field>
    </record>

    <record id="tax_report_B3" model="account.tax.report.line">
        <field name="name">B3 - Livraisons d'électricité, de gaz naturel, de chaleur ou de froid imposables en France
        </field>
        <field name="tag_name">B3</field>
        <field name="code">box_B3</field>
        <field name="parent_id" ref="tax_report_op_imposables_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">8</field>
    </record>

    <record id="tax_report_B4" model="account.tax.report.line">
        <field name="name">B4 - Achats de bien ou de prestations de services réalisés auprès d'un assujetti
            non établi en France
        </field>
        <field name="tag_name">B4</field>
        <field name="code">box_B4</field>
        <field name="parent_id" ref="tax_report_op_imposables_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">9</field>
    </record>

    <record id="tax_report_B5" model="account.tax.report.line">
        <field name="name">B5 - Régularisations</field>
        <field name="tag_name">B5</field>
        <field name="code">box_B5</field>
        <field name="parent_id" ref="tax_report_op_imposables_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
    </record>

    <!-- OPÉRATIONS NON TAXÉES  -->
    <record id="tax_report_op_non_imposables" model="account.tax.report.line">
        <field name="name">Opérations Non Taxées</field>
        <field name="parent_id" ref="tax_report_montant_op_realisees"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_E1" model="account.tax.report.line">
        <field name="name">E1 - Exportations hors UE</field>
        <field name="tag_name">E1</field>
        <field name="code">box_E1</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_E2" model="account.tax.report.line">
        <field name="name">E2 - Autres opérations non imposables</field>
        <field name="tag_name">E2</field>
        <field name="code">box_E2</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_E3" model="account.tax.report.line">
        <field name="name">E3 - Ventes à distance taxables dans un autre État membre au profit des personnes
            non assujetties
        </field>
        <field name="tag_name">E3</field>
        <field name="code">box_E3</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_E4" model="account.tax.report.line">
        <field name="name">E4 - Importations (autres que les produits pétroliers)</field>
        <field name="tag_name">E4</field>
        <field name="code">box_E4</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
    </record>

    <record id="tax_report_E5" model="account.tax.report.line">
        <field name="name">E5 - Sorties de régime fiscal suspensif (autres que les produits pétroliers)</field>
        <field name="tag_name">E5</field>
        <field name="code">box_E5</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">5</field>
    </record>

    <record id="tax_report_E6" model="account.tax.report.line">
        <field name="name">E6 - Importations placées sous régime fiscal suspensif (autres que les produits pétroliers)</field>
        <field name="tag_name">E6</field>
        <field name="code">box_E6</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">6</field>
    </record>

    <record id="tax_report_F1" model="account.tax.report.line">
        <field name="name">F1 - Acquisitions intracommunautaires</field>
        <field name="tag_name">F1</field>
        <field name="code">box_F1</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">8</field>
    </record>

    <record id="tax_report_F2" model="account.tax.report.line">
        <field name="name">F2 - Livraisons intracommunautaires à destination d'une personne assujettie</field>
        <field name="tag_name">F2</field>
        <field name="code">box_F2</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">9</field>
    </record>

    <record id="tax_report_F3" model="account.tax.report.line">
        <field name="name">F3 - Livraisons d’électricité, de gaz naturel, de chaleur ou de
             froid non imposables en France
        </field>
        <field name="tag_name">F3</field>
        <field name="code">box_F3</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
    </record>

    <record id="tax_report_F4" model="account.tax.report.line">
        <field name="name">F4 - Mises à la consommation de produits pétroliers</field>
        <field name="tag_name">F4</field>
        <field name="code">box_F4</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">11</field>
    </record>

    <record id="tax_report_F5" model="account.tax.report.line">
        <field name="name">F5 - Importations de produits pétroliers placées sous régime fiscal suspensif</field>
        <field name="tag_name">F5</field>
        <field name="code">box_F5</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">12</field>
    </record>

    <record id="tax_report_F6" model="account.tax.report.line">
        <field name="name">F6 - Achats en franchise</field>
        <field name="tag_name">F6</field>
        <field name="code">box_F6</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">13</field>
    </record>

    <record id="tax_report_F7" model="account.tax.report.line">
        <field name="name">F7 - Ventes de biens ou prestations de services réalisées par un assujetti non établi en
            France
        </field>
        <field name="tag_name">F7</field>
        <field name="code">box_F7</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">14</field>
    </record>

    <record id="tax_report_F8" model="account.tax.report.line">
        <field name="name">F8 - Régularisations</field>
        <field name="tag_name">7B</field>
        <field name="code">box_7B</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">15</field>
    </record>

    <record id="tax_report_F9" model="account.tax.report.line">
        <field name="name">F9 - Opérations internes réalisées entre membres d'un assujetti unique</field>
        <field name="tag_name">F9</field>
        <field name="code">box_F9</field>
        <field name="parent_id" ref="tax_report_op_non_imposables"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">16</field>
    </record>

    <!-- DÉCOMPTE DE LA TVA À PAYER -->
    <record id="tax_report_decompte_tva" model="account.tax.report.line">
        <field name="name">B. Décompte de la TVA à payer</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
        <field name="formula">None</field>
    </record>

    <!-- TVA BRUTE -->
    <record id="tax_report_tva_brute" model="account.tax.report.line">
        <field name="name">TVA Brute</field>
        <field name="parent_id" ref="tax_report_decompte_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
        <field name="formula">None</field>
    </record>

    <!-- Opérations réalisées en France métropolitaine -->
    <record id="tax_report_tva_brute_metropo" model="account.tax.report.line">
        <field name="name">Opérations réalisées en France métropolitaine</field>
        <field name="parent_id" ref="tax_report_decompte_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_08_base" model="account.tax.report.line">
        <field name="name">08 - Taux normal 20 % (base)</field>
        <field name="tag_name">08_base</field>
        <field name="code">box_08_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_metropo"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>
    <record id="tax_report_08_taxe" model="account.tax.report.line">
        <field name="name">08 - Taux normal 20 % (taxe)</field>
        <field name="tag_name">08_taxe</field>
        <field name="code">box_08_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_metropo"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_09_base" model="account.tax.report.line">
        <field name="name">09 - Taux réduit 5,5 % (base)</field>
        <field name="tag_name">09_base</field>
        <field name="code">box_09_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_metropo"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>
    <record id="tax_report_09_taxe" model="account.tax.report.line">
        <field name="name">09 - Taux réduit 5,5 % (taxe)</field>
        <field name="tag_name">09_taxe</field>
        <field name="code">box_09_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_metropo"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
    </record>

    <record id="tax_report_9B_base" model="account.tax.report.line">
        <field name="name">9B - Taux réduit 10 % (base)</field>
        <field name="tag_name">9B_base</field>
        <field name="code">box_9B_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_metropo"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">5</field>
    </record>
    <record id="tax_report_9B_taxe" model="account.tax.report.line">
        <field name="name">9B - Taux réduit 10 % (taxe)</field>
        <field name="tag_name">9B_taxe</field>
        <field name="code">box_9B_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_metropo"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">6</field>
    </record>

    <!-- Opérations réalisées dans les DOM -->
    <record id="tax_report_tva_brute_dom" model="account.tax.report.line">
        <field name="name">Opérations réalisées dans les DOM</field>
        <field name="parent_id" ref="tax_report_decompte_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_10_base" model="account.tax.report.line">
        <field name="name">10 - Taux normal 8,5 % (base)</field>
        <field name="tag_name">10_base</field>
        <field name="code">box_10_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_dom"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">7</field>
    </record>
    <record id="tax_report_10_taxe" model="account.tax.report.line">
        <field name="name">10 - Taux normal 8,5 % (taxe)</field>
        <field name="tag_name">10_taxe</field>
        <field name="code">box_10_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_dom"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">8</field>
    </record>

    <record id="tax_report_11_base" model="account.tax.report.line">
        <field name="name">11 - Taux réduit 2,1 % (base)</field>
        <field name="tag_name">11_base</field>
        <field name="code">box_11_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_dom"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">9</field>
    </record>
    <record id="tax_report_11_taxe" model="account.tax.report.line">
        <field name="name">11 - Taux réduit 2,1 % (taxe)</field>
        <field name="tag_name">11_taxe</field>
        <field name="code">box_11_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_dom"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
    </record>

    <!-- Opérations imposables à un autre taux (France métropolitaine ou DOM) -->
    <record id="tax_report_tva_brute_autre" model="account.tax.report.line">
        <field name="name">Opérations imposables à un autre taux (France métropolitaine ou DOM)</field>
        <field name="parent_id" ref="tax_report_decompte_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_T1_base" model="account.tax.report.line">
        <field name="name">T1 - Opérations réalisées dans les DOM et imposables au taux de 1,75% (base)</field>
        <field name="tag_name">T1_base</field>
        <field name="code">box_T1_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>
    <record id="tax_report_T1_taxe" model="account.tax.report.line">
        <field name="name">T1 - Opérations réalisées dans les DOM et imposables au taux de 1,75% (taxe)</field>
        <field name="tag_name">T1_taxe</field>
        <field name="code">box_T1_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>
    <record id="tax_report_T2_base" model="account.tax.report.line">
        <field name="name">T2 - Opérations réalisées dans les DOM et imposables au taux de 1,05% (base)</field>
        <field name="tag_name">T2_base</field>
        <field name="code">box_T2_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>
    <record id="tax_report_T2_taxe" model="account.tax.report.line">
        <field name="name">T2 - Opérations réalisées dans les DOM et imposables au taux de 1,05% (taxe)</field>
        <field name="tag_name">T2_taxe</field>
        <field name="code">box_T2_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
    </record>
    <record id="tax_report_T3_base" model="account.tax.report.line">
        <field name="name">T3 - Opérations réalisées en Corse et imposables au taux de 10% (base)</field>
        <field name="tag_name">T3_base</field>
        <field name="code">box_T3_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">5</field>
    </record>
    <record id="tax_report_T3_taxe" model="account.tax.report.line">
        <field name="name">T3 - Opérations réalisées en Corse et imposables au taux de 10% (taxe)</field>
        <field name="tag_name">T3_taxe</field>
        <field name="code">box_T3_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">6</field>
    </record>
    <record id="tax_report_T4_base" model="account.tax.report.line">
        <field name="name">T4 - Opérations réalisées en Corse et imposables au taux de 2,1% (base)</field>
        <field name="tag_name">T4_base</field>
        <field name="code">box_T4_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">7</field>
    </record>
    <record id="tax_report_T4_taxe" model="account.tax.report.line">
        <field name="name">T4 - Opérations réalisées en Corse et imposables au taux de 2,1% (taxe)</field>
        <field name="tag_name">T4_taxe</field>
        <field name="code">box_T4_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">8</field>
    </record>
    <record id="tax_report_T5_base" model="account.tax.report.line">
        <field name="name">T5 - Opérations réalisées en Corse et imposables au taux de 0,9% (base)</field>
        <field name="tag_name">T5_base</field>
        <field name="code">box_T5_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">9</field>
    </record>
    <record id="tax_report_T5_taxe" model="account.tax.report.line">
        <field name="name">T5 - Opérations réalisées en Corse et imposables au taux de 0,9% (taxe)</field>
        <field name="tag_name">T5_taxe</field>
        <field name="code">box_T5_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
    </record>
    <record id="tax_report_T6_base" model="account.tax.report.line">
        <field name="name">T6 - Opérations réalisées en France continentale au taux de 2,1% (base)</field>
        <field name="tag_name">T6_base</field>
        <field name="code">box_T6_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">11</field>
    </record>
    <record id="tax_report_T6_taxe" model="account.tax.report.line">
        <field name="name">T6 - Opérations réalisées en France continentale au taux de 2,1% (taxe)</field>
        <field name="tag_name">T6_taxe</field>
        <field name="code">box_T6_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">12</field>
    </record>
    <record id="tax_report_T7_base" model="account.tax.report.line">
        <field name="name">T7 - Retenue de TVA sur droits d'auteur (base)</field>
        <field name="tag_name">T7_base</field>
        <field name="code">box_T7_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">13</field>
    </record>
    <record id="tax_report_T7_taxe" model="account.tax.report.line">
        <field name="name">T7 - Retenue de TVA sur droits d'auteur (taxe)</field>
        <field name="tag_name">T7_taxe</field>
        <field name="code">box_T7_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">14</field>
    </record>

    <record id="tax_report_13_base" model="account.tax.report.line">
        <field name="name">13 - Anciens taux (base)</field>
        <field name="tag_name">13_base</field>
        <field name="code">box_13_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">15</field>
    </record>
    <record id="tax_report_13_taxe" model="account.tax.report.line">
        <field name="name">13 - Anciens taux (taxe)</field>
        <field name="tag_name">13_taxe</field>
        <field name="code">box_13_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">16</field>
    </record>

    <record id="tax_report_14_base" model="account.tax.report.line">
        <field name="name">14 - Opérations imposables à un taux particulier (base)</field>
        <field name="tag_name">14_base</field>
        <field name="code">box_14_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">17</field>
    </record>
    <record id="tax_report_14_taxe" model="account.tax.report.line">
        <field name="name">14 - Opérations imposables à un taux particulier (taxe)</field>
        <field name="tag_name">14_taxe</field>
        <field name="code">box_14_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_autre"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">18</field>
    </record>

    <!-- Produits pétroliers -->
    <record id="tax_report_tva_brute_petrolier" model="account.tax.report.line">
        <field name="name">Produits pétroliers</field>
        <field name="parent_id" ref="tax_report_decompte_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_P1_base" model="account.tax.report.line">
        <field name="name">P1 - Taux normal 20% (base)</field>
        <field name="tag_name">P1_base</field>
        <field name="code">box_P1_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_petrolier"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">15</field>
    </record>
    <record id="tax_report_P1_taxe" model="account.tax.report.line">
        <field name="name">P1 - Taux normal 20% (taxe)</field>
        <field name="tag_name">P1_taxe</field>
        <field name="code">box_P1_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_petrolier"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">16</field>
    </record>

    <record id="tax_report_P2_base" model="account.tax.report.line">
        <field name="name">P2 - Taux réduit 13% (base)</field>
        <field name="tag_name">P2_base</field>
        <field name="code">box_P2_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_petrolier"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">17</field>
    </record>
    <record id="tax_report_P2_taxe" model="account.tax.report.line">
        <field name="name">P2 - Taux réduit 13% (taxe)</field>
        <field name="tag_name">P2_taxe</field>
        <field name="code">box_P2_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_petrolier"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">18</field>
    </record>

    <!-- Importations -->
    <record id="tax_report_tva_brute_import" model="account.tax.report.line">
        <field name="name">Importations</field>
        <field name="parent_id" ref="tax_report_decompte_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">5</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_I1_base" model="account.tax.report.line">
        <field name="name">I1 - Taux normal 20% (base)</field>
        <field name="tag_name">I1_base</field>
        <field name="code">box_I1_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">19</field>
    </record>
    <record id="tax_report_I1_taxe" model="account.tax.report.line">
        <field name="name">I1 - Taux normal 20% (taxe)</field>
        <field name="tag_name">I1_taxe</field>
        <field name="code">box_I1_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
    </record>

    <record id="tax_report_I2_base" model="account.tax.report.line">
        <field name="name">I2 - Taux réduit 10% (base)</field>
        <field name="tag_name">I2_base</field>
        <field name="code">box_I2_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">21</field>
    </record>
    <record id="tax_report_I2_taxe" model="account.tax.report.line">
        <field name="name">I2 - Taux réduit 10% (taxe)</field>
        <field name="tag_name">I2_taxe</field>
        <field name="code">box_I2_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">22</field>
    </record>

    <record id="tax_report_I3_base" model="account.tax.report.line">
        <field name="name">I3 - Taux réduit 8.5% (base)</field>
        <field name="tag_name">I3_base</field>
        <field name="code">box_I3_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">23</field>
    </record>
    <record id="tax_report_I3_taxe" model="account.tax.report.line">
        <field name="name">I3 - Taux réduit 8.5% (taxe)</field>
        <field name="tag_name">I3_taxe</field>
        <field name="code">box_I3_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">24</field>
    </record>

    <record id="tax_report_I4_base" model="account.tax.report.line">
        <field name="name">I4 - Taux réduit 5.5% (base)</field>
        <field name="tag_name">I4_base</field>
        <field name="code">box_I4_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">25</field>
    </record>
    <record id="tax_report_I4_taxe" model="account.tax.report.line">
        <field name="name">I4 - Taux réduit 5.5% (taxe)</field>
        <field name="tag_name">I4_taxe</field>
        <field name="code">box_I4_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">26</field>
    </record>

    <record id="tax_report_I5_base" model="account.tax.report.line">
        <field name="name">I5 - Taux réduit 2.1% (base)</field>
        <field name="tag_name">I5_base</field>
        <field name="code">box_I5_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">27</field>
    </record>
    <record id="tax_report_I5_taxe" model="account.tax.report.line">
        <field name="name">I5 - Taux réduit 2.1% (taxe)</field>
        <field name="tag_name">I5_taxe</field>
        <field name="code">box_I5_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">28</field>
    </record>

    <record id="tax_report_I6_base" model="account.tax.report.line">
        <field name="name">I6 - Taux réduit 1.05% (base)</field>
        <field name="tag_name">I6_base</field>
        <field name="code">box_I6_base</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">29</field>
    </record>
    <record id="tax_report_I6_taxe" model="account.tax.report.line">
        <field name="name">I6 - Taux réduit 1.05% (taxe)</field>
        <field name="tag_name">I6_taxe</field>
        <field name="code">box_I6_taxe</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">30</field>
    </record>

    <record id="tax_report_15" model="account.tax.report.line">
        <field name="name">15 - TVA antérieurement déduite à reverser</field>
        <field name="tag_name">15</field>
        <field name="code">box_15</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">31</field>
    </record>
    <record id="tax_report_15_1" model="account.tax.report.line">
        <field name="name">dont TVA sur les produits pétroliers</field>
        <field name="tag_name">15_1</field>
        <field name="code">box_15_1</field>
        <field name="parent_id" ref="tax_report_15"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">32</field>
    </record>
    <record id="tax_report_15_2" model="account.tax.report.line">
        <field name="name">dont TVA sur les produits importés hors produits pétroliers</field>
        <field name="tag_name">15_2</field>
        <field name="code">box_15_2</field>
        <field name="parent_id" ref="tax_report_15"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">33</field>
    </record>

    <record id="tax_report_5B" model="account.tax.report.line">
        <field name="name">5B - Sommes à ajouter, y compris acompte congés</field>
        <field name="tag_name">5B</field>
        <field name="code">box_5B</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">32</field>
    </record>

    <record id="tax_report_16" model="account.tax.report.line">
        <field name="name">16 - Total de la TVA brute due</field>
        <field name="code">box_16</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">33</field>
        <field name="formula">
            box_08_taxe + box_09_taxe + box_9B_taxe + box_10_taxe + box_11_taxe + box_13_taxe + box_14_taxe + box_T1_taxe + box_T2_taxe + box_T3_taxe + box_T4_taxe + box_T5_taxe + box_T6_taxe + box_T7_taxe + box_P1_taxe + box_P2_taxe + box_I1_taxe + box_I2_taxe + box_I3_taxe + box_I4_taxe + box_I5_taxe + box_I6_taxe + box_15 + box_5B
        </field>
    </record>

    <record id="tax_report_17" model="account.tax.report.line">
        <field name="name">17 - Dont TVA sur acquisitions intracommunautaires</field>
        <field name="tag_name">17</field>
        <field name="code">box_17</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">34</field>
    </record>

    <record id="tax_report_18" model="account.tax.report.line">
        <field name="name">18 - Dont TVA sur opérations à destination de Monaco</field>
        <field name="tag_name">18</field>
        <field name="code">box_18</field>
        <field name="parent_id" ref="tax_report_tva_brute_import"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">35</field>
    </record>

    <!-- TVA DÉDUCTIBLE -->
    <record id="tax_report_tva_deductible" model="account.tax.report.line">
        <field name="name">TVA Déductible</field>
        <field name="parent_id" ref="tax_report_decompte_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">6</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_19" model="account.tax.report.line">
        <field name="name">19 - Biens constituant des immobilisations</field>
        <field name="tag_name">19</field>
        <field name="code">box_19</field>
        <field name="parent_id" ref="tax_report_tva_deductible"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_20" model="account.tax.report.line">
        <field name="name">20 - Autres bien et services</field>
        <field name="tag_name">20</field>
        <field name="code">box_20</field>
        <field name="parent_id" ref="tax_report_tva_deductible"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_21" model="account.tax.report.line">
        <field name="name">21 - Autre TVA à déduire</field>
        <field name="tag_name">21</field>
        <field name="code">box_21</field>
        <field name="parent_id" ref="tax_report_tva_deductible"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_22" model="account.tax.report.line">
        <field name="name">22 - Report du crédit apparaissant ligne 27 de la précédente déclaration</field>
        <field name="tag_name">22</field>
        <field name="code">box_22</field>
        <field name="parent_id" ref="tax_report_tva_deductible"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
        <field name="is_carryover_used_in_balance">True</field>
    </record>

    <record id="tax_report_2C" model="account.tax.report.line">
        <field name="name">2C - Sommes à imputer, y compris acompte congés</field>
        <field name="tag_name">2C</field>
        <field name="code">box_2C</field>
        <field name="parent_id" ref="tax_report_tva_deductible"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">5</field>
    </record>

    <record id="tax_report_23" model="account.tax.report.line">
        <field name="name">23 - Total TVA déductible</field>
        <field name="code">box_23</field>
        <field name="parent_id" ref="tax_report_tva_deductible"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">7</field>
        <field name="formula">box_19 + box_20 + box_21 + box_22 + box_2C</field>
    </record>

    <record id="tax_report_24" model="account.tax.report.line">
        <field name="name">24 - Dont TVA déductible sur importations</field>
        <field name="tag_name">24</field>
        <field name="code">box_24</field>
        <field name="parent_id" ref="tax_report_tva_deductible"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">8</field>
    </record>

    <record id="tax_report_2E" model="account.tax.report.line">
        <field name="name">2E - Dont TVA déductible sur les produits pétroliers</field>
        <field name="tag_name">2E</field>
        <field name="code">box_2E</field>
        <field name="parent_id" ref="tax_report_tva_deductible"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">9</field>
    </record>

    <!-- TVA due ou crédit de TVA -->
    <record id="tax_report_credit" model="account.tax.report.line">
        <field name="name">TVA due ou crédit de TVA</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_25" model="account.tax.report.line">
        <field name="name">25 - Crédit de TVA</field>
        <field name="code">box_25</field>
        <field name="parent_id" ref="tax_report_credit"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
        <field name="formula">box_23 - box_16 if box_23 - box_16 > 0 else 0</field>
    </record>

    <record id="tax_report_TD" model="account.tax.report.line">
        <field name="name">TD - TVA Due</field>
        <field name="code">box_TD</field>
        <field name="parent_id" ref="tax_report_credit"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
        <field name="formula">box_16 - box_23 if box_16 - box_23 > 0 else 0</field>
    </record>

    <!-- Régularisation des taxes intérieures de consommation (TIC) -->
    <record id="tax_report_regularisation" model="account.tax.report.line">
        <field name="name">Régularisation des taxes intérieures de consommation (TIC)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
    </record>

    <!-- Crédit constaté -->
    <record id="tax_report_credit_constate" model="account.tax.report.line">
        <field name="name">Crédit constaté</field>
        <field name="parent_id" ref="tax_report_regularisation"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>

    <record id="tax_report_TICFE" model="account.tax.report.line">
        <field name="name">TICFE</field>
        <field name="code">box_TICFE</field>
        <field name="tag_name">TICFE_constate</field>
        <field name="parent_id" ref="tax_report_credit_constate"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>
    <record id="tax_report_TICGN" model="account.tax.report.line">
        <field name="name">TICGN</field>
        <field name="code">box_TICGN</field>
        <field name="tag_name">TICGN_constate</field>
        <field name="parent_id" ref="tax_report_credit_constate"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>
    <record id="tax_report_TICC" model="account.tax.report.line">
        <field name="name">TICC</field>
        <field name="code">box_TICC</field>
        <field name="tag_name">TICC_constate</field>
        <field name="parent_id" ref="tax_report_credit_constate"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>
    <record id="tax_report_TIC_total" model="account.tax.report.line">
        <field name="name">Total</field>
        <field name="code">box_TIC_total</field>
        <field name="parent_id" ref="tax_report_credit_constate"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
        <field name="formula">box_TICFE + box_TICGN + box_TICC</field>
    </record>

    <!-- Crédit imputé -->
    <record id="tax_report_credit_impute" model="account.tax.report.line">
        <field name="name">Crédit imputé</field>
        <field name="parent_id" ref="tax_report_regularisation"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_X1" model="account.tax.report.line">
        <field name="name">X1</field>
        <field name="tag_name">X1</field>
        <field name="code">box_X1</field>
        <field name="parent_id" ref="tax_report_credit_impute"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>
    <record id="tax_report_X2" model="account.tax.report.line">
        <field name="name">X2</field>
        <field name="tag_name">X2</field>
        <field name="code">box_X2</field>
        <field name="parent_id" ref="tax_report_credit_impute"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>
    <record id="tax_report_X3" model="account.tax.report.line">
        <field name="name">X3</field>
        <field name="tag_name">X3</field>
        <field name="code">box_X3</field>
        <field name="parent_id" ref="tax_report_credit_impute"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>
    <record id="tax_report_X4" model="account.tax.report.line">
        <field name="name">X4</field>
        <field name="code">box_X4</field>
        <field name="parent_id" ref="tax_report_credit_impute"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
        <field name="formula">box_X1 + box_X2 + box_X3</field>
    </record>

    <!-- Reliquat de crédit à rembourser -->
    <record id="tax_report_reliquat" model="account.tax.report.line">
        <field name="name">Reliquat de crédit à rembourser</field>
        <field name="parent_id" ref="tax_report_regularisation"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_Y1" model="account.tax.report.line">
        <field name="name">Y1</field>
        <field name="code">box_Y1</field>
        <field name="parent_id" ref="tax_report_reliquat"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
        <field name="formula">box_TICFE - box_X1</field>
    </record>
    <record id="tax_report_Y2" model="account.tax.report.line">
        <field name="name">Y2</field>
        <field name="code">box_Y2</field>
        <field name="parent_id" ref="tax_report_reliquat"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
        <field name="formula">box_TICGN - box_X2</field>
    </record>
    <record id="tax_report_Y3" model="account.tax.report.line">
        <field name="name">Y3</field>
        <field name="code">box_Y3</field>
        <field name="parent_id" ref="tax_report_reliquat"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
        <field name="formula">box_TICC - box_X3</field>
    </record>
    <record id="tax_report_Y4" model="account.tax.report.line">
        <field name="name">Y4</field>
        <field name="code">box_Y4</field>
        <field name="parent_id" ref="tax_report_reliquat"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
        <field name="formula">box_Y1 + box_Y2 + box_Y3</field>
    </record>

    <!-- Taxe due -->
    <record id="tax_report_tic_tax" model="account.tax.report.line">
        <field name="name">Taxe due</field>
        <field name="parent_id" ref="tax_report_regularisation"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
    </record>

    <record id="tax_report_Z1" model="account.tax.report.line">
        <field name="name">Z1</field>
        <field name="tag_name">Z1</field>
        <field name="code">box_Z1</field>
        <field name="parent_id" ref="tax_report_tic_tax"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">1</field>
    </record>
    <record id="tax_report_Z2" model="account.tax.report.line">
        <field name="name">Z2</field>
        <field name="tag_name">Z2</field>
        <field name="code">box_Z2</field>
        <field name="parent_id" ref="tax_report_tic_tax"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>
    <record id="tax_report_Z3" model="account.tax.report.line">
        <field name="name">Z3</field>
        <field name="tag_name">Z3</field>
        <field name="code">box_Z3</field>
        <field name="parent_id" ref="tax_report_tic_tax"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>
    <record id="tax_report_Z4" model="account.tax.report.line">
        <field name="name">Z4</field>
        <field name="code">box_Z4</field>
        <field name="parent_id" ref="tax_report_tic_tax"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
        <field name="formula">box_Z1 + box_Z2 + box_Z3</field>
    </record>

    <!-- Détermination du montant à payer et/ou des crédits de TVA et/ou de TIC-->
    <record id="tax_report_determination" model="account.tax.report.line">
        <field name="name">Détermination du montant à payer et/ou des crédits de TVA et/ou de TIC</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">5</field>
    </record>

    <record id="tax_report_26" model="account.tax.report.line">
        <field name="name">26 - Remboursement de crédit demandé sur formulaire n°3519 joint</field>
        <field name="tag_name">26</field>
        <field name="code">box_26</field>
        <field name="parent_id" ref="tax_report_determination"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">2</field>
    </record>

    <record id="tax_report_AA" model="account.tax.report.line">
        <field name="name">AA - Crédit de TVA transféré à la société tête de groupe sur la déclaration récapitulative
            3310-CA3G
        </field>
        <field name="tag_name">AA</field>
        <field name="code">box_AA</field>
        <field name="parent_id" ref="tax_report_determination"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">3</field>
    </record>

    <record id="tax_report_27" model="account.tax.report.line">
        <field name="name">27 - Crédit à reporter</field>
        <field name="code">box_27</field>
        <field name="parent_id" ref="tax_report_determination"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">4</field>
        <field name="formula">box_25 - box_26 - box_AA if box_25 - box_26 - box_AA > 0 else 0</field>
        <field name="carry_over_condition_method">always_carry_over_and_set_to_0</field>
        <field name="carry_over_destination_line_id" ref="tax_report_22"/>
        <field name="is_carryover_persistent">False</field>
    </record>

    <record id="tax_report_Y5" model="account.tax.report.line">
        <field name="name">Y5 - Remboursement de reliquat de TIC demandé (report de la ligne Y4)</field>
        <field name="code">box_Y5</field>
        <field name="parent_id" ref="tax_report_determination"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">5</field>
        <field name="formula">box_Y4</field>
    </record>

    <record id="tax_report_Y6" model="account.tax.report.line">
        <field name="name">Y6 - Crédit de TIC transféré à la société tête de groupe sur la déclaration récapitulative
            3310-CA3G (report de la ligne Y4)</field>
        <field name="code">box_Y6</field>
        <field name="parent_id" ref="tax_report_determination"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">5</field>
    </record>

    <record id="tax_report_X5" model="account.tax.report.line">
        <field name="name">X5 - Crédit de TIC imputé sur la TVA (report de la ligne X4)</field>
        <field name="code">box_X5</field>
        <field name="parent_id" ref="tax_report_determination"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">6</field>
        <field name="formula">box_X4</field>
    </record>

    <record id="tax_report_28" model="account.tax.report.line">
        <field name="name">28 - TVA nette due</field>
        <field name="code">box_28</field>
        <field name="parent_id" ref="tax_report_determination"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">7</field>
        <field name="formula">box_TD - box_X5 if box_TD - box_X5 > 0 else 0</field>
    </record>

    <record id="tax_report_29" model="account.tax.report.line">
        <field name="name">29 - Taxes assimilées calculées sur annexe n°3310-A-SD</field>
        <field name="tag_name">29</field>
        <field name="code">box_29</field>
        <field name="parent_id" ref="tax_report_determination"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">8</field>
    </record>

    <record id="tax_report_Z5" model="account.tax.report.line">
        <field name="name">Z5 - Total des taxes intérieures de consommation dues (report de la ligne Z4)</field>
        <field name="code">box_Z5</field>
        <field name="parent_id" ref="tax_report_determination"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">9</field>
        <field name="formula">box_Z4</field>
    </record>

    <record id="tax_report_AB" model="account.tax.report.line">
        <field name="name">AB - Total à payer acquitté par la société tête de groupe sur la déclaration récapitulative
            3310-CA3G
        </field>
        <field name="code">box_AB</field>
        <field name="parent_id" ref="tax_report_determination"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <!--According to the trustee, it's very rare that a company fill this box-->
        <field name="formula">None</field>
    </record>

    <record id="tax_report_32" model="account.tax.report.line">
        <field name="name">32 - Total à payer</field>
        <field name="code">box_32</field>
        <field name="parent_id" ref="tax_report_determination"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">11</field>
        <!--Omit "... - box_AB" in the formula because box_AB not zero is a rare edge case -->
        <field name="formula">box_28 + box_29 + box_Z5</field>
    </record>

</odoo>

```

## File: migrations\2.1\post-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.addons.account.models.chart_template import update_taxes_from_templates


def migrate(cr, version):
    update_taxes_from_templates(cr, 'l10n_fr.l10n_fr_pcg_chart_template')

```

## File: migrations\9.0.1.1\pre-set_tags_and_taxes_updatable.py

```python
from openerp.modules.registry import RegistryManager

def migrate(cr, version):
    registry = RegistryManager.get(cr.dbname)
    from openerp.addons.account.models.chart_template import migrate_set_tags_and_taxes_updatable
    migrate_set_tags_and_taxes_updatable(cr, registry, 'l10n_fr')


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
        if company.account_fiscal_country_id.code != 'FR':
            return journal_data

        for journal in journal_data:
            if journal['type'] in ('sale', 'purchase'):
                journal.update({'refund_sequence': True})
        return journal_data

```

## File: models\l10n_fr.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import fields, models, api, _


class ResPartner(models.Model):
    _inherit = 'res.partner'

    siret = fields.Char(string='SIRET', size=14)


```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_fr_closing_sequence_id = fields.Many2one('ir.sequence', 'Sequence to use to build sale closings', readonly=True)
    siret = fields.Char(related='partner_id.siret', string='SIRET', size=14, readonly=False)
    ape = fields.Char(string='APE')

    @api.model
    def _get_france_country_codes(self):
        """Returns every country code that can be used to represent France
        """
        return ['FR', 'MF', 'MQ', 'NC', 'PF', 'RE', 'GF', 'GP', 'TF'] # These codes correspond to France and DOM-TOM.

    @api.model
    def _get_unalterable_country(self):
        return self._get_france_country_codes()

    def _is_accounting_unalterable(self):
        if not self.vat and not self.country_id:
            return False
        return self.country_id and self.country_id.code in self._get_unalterable_country()

    @api.model_create_multi
    def create(self, vals_list):
        companies = super().create(vals_list)
        for company in companies:
            #when creating a new french company, create the securisation sequence as well
            if company._is_accounting_unalterable():
                sequence_fields = ['l10n_fr_closing_sequence_id']
                company._create_secure_sequence(sequence_fields)
        return companies

    def write(self, vals):
        res = super(ResCompany, self).write(vals)
        #if country changed to fr, create the securisation sequence
        for company in self:
            if company._is_accounting_unalterable():
                sequence_fields = ['l10n_fr_closing_sequence_id']
                company._create_secure_sequence(sequence_fields)
        return res

    def _create_secure_sequence(self, sequence_fields):
        """This function creates a no_gap sequence on each company in self that will ensure
        a unique number is given to all posted account.move in such a way that we can always
        find the previous move of a journal entry on a specific journal.
        """
        for company in self:
            vals_write = {}
            for seq_field in sequence_fields:
                if not company[seq_field]:
                    vals = {
                        'name': _('Securisation of %s - %s') % (seq_field, company.name),
                        'code': 'FRSECURE%s-%s' % (company.id, seq_field),
                        'implementation': 'no_gap',
                        'prefix': '',
                        'suffix': '',
                        'padding': 0,
                        'company_id': company.id}
                    seq = self.env['ir.sequence'].create(vals)
                    vals_write[seq_field] = seq.id
            if vals_write:
                company.write(vals_write)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import l10n_fr
from . import account_chart_template
from . import res_company

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="5.81" width="50.4" height="36.36" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.38" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="413" height="275" transform="translate(4.8 5.81) scale(0.12 0.13)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAZ0AAAEqCAYAAADOEI1vAAAACXBIWXMAAFq8AABavAEpkzmBAAAEnUlEQVR4Xu3VoVFkARBF0Td/qNocEASzHkMQZEEWOKpIAL3ZDQMWQ+GuWM6xXd2y72m3rx+D73ycdv9wt7fnv/tz3XZc9n692XFsp123HT9d4L923d6P7XzZHp+2l39fZpdt5+8W+aV8DAAyogNARnQAyIgOABnRASAjOgBkRAeAjOgAkBEdADKiA0BGdADIiA4AGdEBICM6AGREB4CM6ACQER0AMqIDQEZ0AMiIDgAZ0QEgIzoAZEQHgIzoAJARHQAyogNARnQAyIgOABnRASAjOgBkRAeAjOgAkBEdADKiA0BGdADIiA4AGdEBICM6AGREB4CM6ACQER0AMqIDQEZ0AMiIDgAZ0QEgIzoAZEQHgIzoAJARHQAyogNARnQAyIgOABnRASAjOgBkRAeAjOgAkBEdADKiA0BGdADIiA4AGdEBICM6AGREB4CM6ACQER0AMqIDQEZ0AMiIDgAZ0QEgIzoAZEQHgIzoAJARHQAyogNARnQAyIgOABnRASAjOgBkRAeAjOgAkBEdADKiA0BGdADIiA4AGdEBICM6AGREB4CM6ACQER0AMqIDQEZ0AMiIDgAZ0QEgIzoAZEQHgIzoAJARHQAyogNARnQAyIgOABnRASAjOgBkRAeAjOgAkBEdADKiA0BGdADIiA4AGdEBICM6AGREB4CM6ACQER0AMqIDQEZ0AMiIDgAZ0QEgIzoAZEQHgIzoAJARHQAyogNARnQAyIgOABnRASAjOgBkRAeAjOgAkBEdADKiA0BGdADIiA4AGdEBICM6AGREB4CM6ACQER0AMqIDQEZ0AMiIDgAZ0QEgIzoAZEQHgIzoAJARHQAyogNARnQAyIgOABnRASAjOgBkRAeAjOgAkBEdADKiA0BGdADIiA4AGdEBICM6AGREB4CM6ACQER0AMqIDQEZ0AMiIDgAZ0QEgIzoAZEQHgIzoAJARHQAyogNARnQAyIgOABnRASAjOgBkRAeAjOgAkBEdADKiA0BGdADIiA4AGdEBICM6AGREB4CM6ACQER0AMqIDQEZ0AMiIDgAZ0QEgIzoAZEQHgIzoAJARHQAyogNARnQAyIgOABnRASAjOgBkRAeAjOgAkBEdADKiA0BGdADIiA4AGdEBICM6AGREB4CM6ACQER0AMqIDQEZ0AMiIDgAZ0QEgIzoAZEQHgIzoAJARHQAyogNARnQAyIgOABnRASAjOgBkRAeAjOgAkBEdADKiA0BGdADIiA4AGdEBICM6AGREB4CM6ACQER0AMqIDQEZ0AMiIDgAZ0QEgIzoAZEQHgIzoAJARHQAyogNARnQAyIgOABnRASAjOgBkRAeAjOgAkBEdADKiA0BGdADIiA4AGdEBICM6AGREB4CM6ACQER0AMqIDQEZ0AMiIDgAZ0QEgIzoAZEQHgIzoAJARHQAyogNARnQAyIgOABnRASAjOgBkRAeAjOgAkBEdADKiA0BGdADIiA4AGdEBICM6AGREB4CM6ACQER0AMqIDQEZ0AMiIDgAZ0QEgIzoAZEQHgIzoAJARHQAyogNARnQAyIgOABnRASAjOgBkRAeAjOgAkBEdADKfIdoSfs2KQjEAAAAASUVORK5CYII="/>
    </g>
  </g>
</svg>

```

## File: views\l10n_fr_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="res_company_form_l10n_fr" model="ir.ui.view">
            <field name="name">res.company.form.l10n.fr</field>
            <field name="model">res.company</field>
            <field name="priority">20</field>
            <field name="inherit_id" ref="account.view_company_form"/>
            <field name="arch" type="xml">
            <data>
                 <xpath expr="//field[@name='company_registry']" position="after">
                    <field name="siret" attrs="{'invisible': [('country_code', '!=', 'FR')]}"/>
                    <field name="ape" attrs="{'invisible': [('country_code', '!=', 'FR')]}"/>
                 </xpath>
            </data>
            </field>
        </record>

        <record id="res_partner_form_l10n_fr" model="ir.ui.view">
            <field name="name">res.partner.form.l10n.fr</field>
            <field name="model">res.partner</field>
            <field name="priority">20</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
            <data>
                 <xpath expr="//field[@name='ref']" position="after">
                    <field name="siret" attrs="{'invisible': [('is_company', '=', False)]}"/>
                 </xpath>
            </data>
            </field>
        </record>
</odoo>

```

## File: views\report_l10nfrbilan.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
<template id="report_l10nfrbilan">
    <t t-call="web.html_container">
        <t t-call="web.internal_layout">
            <div class="page">
                <h2>Bilan</h2>
                <div class="row mt32 mb32">
                    <div class="col-3">
                        <span t-esc="res_company.name"/>
                        <br/>au
                        <span t-esc="time.strftime('%d-%m-%Y', time.strptime(date_stop,'%Y-%m-%d'))"/>
                    </div>
                    <div class="col-3">
                        <p>Imprimé le
                            <span t-esc="time.strftime('%d-%m-%Y')"/>
                            <br/>Tenue de compte:
                            <span t-esc="res_company.currency_id.name"/>
                        </p>
                    </div>
                </div>
                <h3>Actif</h3>
                <table class="table table-sm">
                    <thead>
                        <tr>
                            <th></th>
                            <th>Brut</th>
                            <th>Amortissements et d&#xE9;pr&#xE9;ciations</th>
                            <th>Net</th>
                        </tr>
                    </thead>
                    <tr>
                        <td>Capital souscrit - non appel&#xE9;</td>
                        <td class="text-right">
                            <span t-esc="bavar1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-right">
                            <span t-esc="bavar1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>
                            <strong>ACTIF IMMOBILIS&#xC9;</strong>
                        </td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>
                            <strong>IMMOBILISATIONS INCORPORELLES</strong>
                        </td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>Frais d'&#xE9;tablissement</td>
                        <td class="text-right">
                            <span t-esc="bavar2" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar2b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar2+bavar2b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Frais de recherche et de d&#xE9;veloppement</td>
                        <td class="text-right">
                            <span t-esc="bavar3" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar3b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar3+bavar3b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Concessions, brevets, licences,..., droits et valeurs similaires</td>
                        <td class="text-right">
                            <span t-esc="bavar4" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar4b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar4+bavar4b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Fonds commercial</td>
                        <td class="text-right">
                            <span t-esc="bavar5" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar5b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar5+bavar5b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Autres</td>
                        <td class="text-right">
                            <span t-esc="bavar6" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar6b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar6+bavar6b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Immobilisations incorporelles en cours</td>
                        <td class="text-right">
                            <span t-esc="bavar7" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar7b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar7+bavar7b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Avances et acomptes</td>
                        <td class="text-right">
                            <span t-esc="bavar8" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-right">
                            <span t-esc="bavar8" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>
                            <strong>IMMOBILISATIONS CORPORELLES</strong>
                        </td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>Terrains</td>
                        <td class="text-right">
                            <span t-esc="bavar9" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar9b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar9+bavar9b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Constructions</td>
                        <td class="text-right">
                            <span t-esc="bavar10" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar10b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar10+bavar10b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Installations techniques,mat&#xE9;riel et outillage</td>
                        <td class="text-right">
                            <span t-esc="bavar11" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar11b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar11+bavar11b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Autres</td>
                        <td class="text-right">
                            <span t-esc="bavar12" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar12b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar12+bavar12b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Immobilisations corporelles en cours</td>
                        <td class="text-right">
                            <span t-esc="bavar13" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar13b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar13+bavar13b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Avances et acomptes</td>
                        <td class="text-right">
                            <span t-esc="bavar14" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-right">
                            <span t-esc="bavar14" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>
                            <strong>IMMOBILISATIONS FINANCI&#xC9;RES</strong>
                        </td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>Participations</td>
                        <td class="text-right">
                            <span t-esc="bavar15" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar15b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar15+bavar15b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Cr&#xE9;ances rattach&#xE9;es &#xE0; des participations</td>
                        <td class="text-right">
                            <span t-esc="bavar16" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar16b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar16+bavar16b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Titres immobilis&#xE9;s de l'activit&#xE9; de portefeuille</td>
                        <td class="text-right">
                            <span t-esc="bavar17" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar17b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar17+bavar17b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Autres titres immobilis&#xE9;s</td>
                        <td class="text-right">
                            <span t-esc="bavar18" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar18b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar18+bavar18b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Pr&#xEA;ts</td>
                        <td class="text-right">
                            <span t-esc="bavar19" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar19b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar19+bavar19b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Autres</td>
                        <td class="text-right">
                            <span t-esc="bavar20" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar20b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar20+bavar20b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td class="text-right">
                            <strong>TOTAL I</strong>
                        </td>
                        <td class="text-right">
                            <span t-esc="at1a" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-at1b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="at1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>ACTIF CIRCULANT</td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>STOCK EN COURS</td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>Mati&#xE8;res premi&#xE8;res et autres approvisionnements</td>
                        <td class="text-right">
                            <span t-esc="bavar21" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar21b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar21+bavar21b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>En-cours de production [biens et services]</td>
                        <td class="text-right">
                            <span t-esc="bavar22" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar22b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar22+bavar22b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Produits interm&#xE9;diaires et finis</td>
                        <td class="text-right">
                            <span t-esc="bavar23" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar23b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar23+bavar23b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Marchandises</td>
                        <td class="text-right">
                            <span t-esc="bavar24" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar24b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar24+bavar24b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Avances et acomptes vers&#xE9;s sur commandes</td>
                        <td class="text-right">
                            <span t-esc="bavar25" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-right">
                            <span t-esc="bavar25" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>CR&#xC9;ANCES</td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>Cr&#xE9;ances clients et comptes rattach&#xE9;s</td>
                        <td class="text-right">
                            <span t-esc="bavar26" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar26b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar26+bavar26b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Autres</td>
                        <td class="text-right">
                            <span t-esc="bavar27" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar27b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar27+bavar27b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Capital souscrit - appel&#xE9; , non vers&#xE9;</td>
                        <td class="text-right">
                            <span t-esc="bavar28" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-right">
                            <span t-esc="bavar28" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>VALEURS MOBILI&#xC8;RES DE PLACEMENT</td>
                        <td></td>
                        <td></td>
                        <td></td>
                    </tr>
                    <tr>
                        <td>Actions propres</td>
                        <td class="text-right">
                            <span t-esc="bavar29" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar29b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar29+bavar29b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Autres titres</td>
                        <td class="text-right">
                            <span t-esc="bavar30" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-bavar30b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="bavar30+bavar30b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Instruments de tr&#xE9;sorerie</td>
                        <td class="text-right">
                            <span t-esc="bavar31" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-right">
                            <span t-esc="bavar31" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Disponibilit&#xE9;s</td>
                        <td class="text-right">
                            <span t-esc="bavar32" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-right">
                            <span t-esc="bavar32" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Charges constat&#xE9;s d'avance</td>
                        <td class="text-right">
                            <span t-esc="bavar33" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-right">
                            <span t-esc="bavar33" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td class="text-right">
                            <strong>TOTAL II</strong>
                        </td>
                        <td class="text-right">
                            <span t-esc="at2a" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-at2b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="at2" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Charges &#xE0; r&#xE9;partir sur plusieurs exercices ( III )</td>
                        <td class="text-right">
                            <span t-esc="bavar34" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-right">
                            <span t-esc="bavar34" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>Primes de remboursement des emprunts ( IV )</td>
                        <td class="text-right">
                            <span t-esc="bavar35" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-right">
                            <span t-esc="bavar35" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td>&#xC9;carts de conversion actif ( V )</td>
                        <td class="text-right">
                            <span t-esc="bavar36" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td></td>
                        <td class="text-right">
                            <span t-esc="bavar36" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                    <tr>
                        <td class="text-right">
                            <strong>TOTAL ACTIF ( I + II + III + IV + V )</strong>
                        </td>
                        <td class="text-right">
                            <span t-esc="at1a+at2a" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="-at1b-at2b" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                        <td class="text-right">
                            <span t-esc="actif" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                        </td>
                    </tr>
                </table>

                <h3>Passif</h3>
                <table class="table table-sm">
                    <tbody>
                        <tr>
                            <td><strong>CAPITAUX PROPRES</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Capital [dont vers&#xE9;...]</td>
                            <td><span t-esc="bpvar1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Primes d'&#xE9;mission, de fusion, d'apport</td>
                            <td><span t-esc="bpvar2" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>&#xC9;carts de r&#xE9;&#xE9;valuation</td>
                            <td><span t-esc="bpvar3" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>&#xC9;cart d'&#xE9;quivalence</td>
                            <td><span t-esc="bpvar4" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td><strong>R&#xC9;SERVES</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>R&#xE9;serve l&#xE9;gale</td>
                            <td><span t-esc="bpvar5" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>R&#xE9;serves statutaires ou contractuelles</td>
                            <td><span t-esc="bpvar6" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>R&#xE9;serves r&#xE9;glement&#xE9;es</td><td><span t-esc="bpvar7" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td></tr>
                        <tr>
                            <td>Autres r&#xE9;serves</td>
                            <td><span t-esc="bpvar8" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Report &#xE0; nouveau</td>
                            <td><span t-esc="bpvar9" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td><strong>R&#xC9;SULTAT DE L'EXERCICE [b&#xE9;n&#xE9;fice ou perte]</strong></td>
                            <td><span t-esc="bpvar10" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Subventions d'investissement</td>
                            <td><span t-esc="bpvar11" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Provisions r&#xE9;glement&#xE9;es</td>
                            <td><span t-esc="bpvar12" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td class="text-right"><strong>TOTAL I</strong></td>
                            <td><span t-esc="pt1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td><strong>PROVISIONS</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Provisions pour risques</td>
                            <td><span t-esc="bpvar13" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Provisions pour charges</td>
                            <td><span t-esc="bpvar14" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td class="text-right"><strong>TOTAL II</strong></td>
                            <td><span t-esc="pt2" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td><strong>DETTES</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Emprunts obligataires convertibles</td>
                            <td><span t-esc="bpvar15" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Autres emprunts obligataires</td>
                            <td><span t-esc="bpvar16" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Emprunts et dettes aupr&#xE8;s des &#xE9;tablissements de cr&#xE9;dit</td>
                            <td><span t-esc="bpvar17" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Emprunts et dettes financi&#xE8;res diverses</td>
                            <td><span t-esc="bpvar18" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Avances et acomptes re&#xE7;us sur commandes en cours</td>
                            <td><span t-esc="bpvar19" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Dettes fournisseurs et comptes rattach&#xE9;s </td>
                            <td><span t-esc="bpvar20" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Dettes fiscales et sociales</td>
                            <td><span t-esc="bpvar21" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Dettes sur immobilisations et comptes rattach&#xE9;s</td>
                            <td><span t-esc="bpvar22" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Autres dettes</td>
                            <td><span t-esc="bpvar23" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Instruments de tr&#xE9;sorerie</td>
                            <td><span t-esc="bpvar24" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>Produits constat&#xE9;s d'avance</td>
                            <td><span t-esc="bpvar25" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td class="text-right"><strong>TOTAL III</strong></td>
                            <td><span t-esc="pt3" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>&#xC9;carts de conversion passif <font face="Times-Roman">( IV )</font></td>
                            <td><span t-esc="bpvar26" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td><strong>TOTAL G&#xC9;N&#xC9;RAL (I + II + III + IV)</strong></td>
                            <td><span t-esc="passif" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                        <tr>
                            <td>&amp;nbsp;</td>
                            <td>&amp;nbsp;</td>
                        </tr>
                        <tr>
                            <td><strong>ACTIF - PASSIF</strong></td>
                            <td><span t-esc="round(actif-passif,2)" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/></td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </t>
    </t>
</template>
</data>
</odoo>

```

## File: views\report_l10nfrresultat.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
<template id="report_l10nfrresultat">
    <t t-call="web.html_container">
        <t t-call="web.internal_layout">
            <div class="page">
                <h2>Compte de résultat</h2>
                <div class="row mt32 mb32">
                    <div class="col-3">
                        <span t-esc="res_company.name"/>
                        <br/>au
                        <span t-esc="time.strftime('%d-%m-%Y', time.strptime(date_stop,'%Y-%m-%d'))"/>
                    </div>
                    <div class="col-3">
                        <p>Imprimé le
                            <span t-esc="time.strftime('%d-%m-%Y')"/>
                            <br/>Tenue de compte:
                            <span t-esc="res_company.currency_id.name"/>
                        </p>
                    </div>
                </div>
                <table class="table table-sm">
                    <thead>
                        <tr>
                            <th>Charges (hors taxes)</th>
                            <th></th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td><strong>CHARGES D'EXPLOITATION</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Achat de marchandises</td>
                            <td>
                                <span class="text-right" t-esc="cdrc1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Variation des stocks</td>
                            <td>
                                <span class="text-right" t-esc="cdrc2" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Achats de mati&#xE8;res premi&#xE8;res et autres approvisionnements</td>
                            <td>
                                <span class="text-right" t-esc="cdrc3" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Variation des stocks</td>
                            <td>
                                <span class="text-right" t-esc="cdrc4" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Autres achats et charges externes</td>
                            <td>
                                <span class="text-right" t-esc="cdrc5" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Redevances de cr&#xE9;dit-bail mobilier</td>
                            <td>
                                <span class="text-right" t-esc="cdrc6" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Redevances de cr&#xE9;dit-bail immobilier</td>
                            <td>
                                <span class="text-right" t-esc="cdrc7" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Imp&#xF4;ts, taxes et versements assimil&#xE9;s</td>
                            <td>
                                <span class="text-right" t-esc="cdrc8" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Salaires et traitements</td>
                            <td>
                                <span class="text-right" t-esc="cdrc9" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Charges sociales</td>
                            <td>
                                <span class="text-right" t-esc="cdrc10" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Dotation aux amortissements et aux d&#xE9;pr&#xE9;ciations</td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Sur immobilisations : dotations aux amortissements</td>
                            <td>
                                <span class="text-right" t-esc="cdrc11" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Sur immobilisations : dotations aux d&#xE9;pr&#xE9;ciations</td>
                            <td>
                                <span class="text-right" t-esc="cdrc12" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Sur actif circulant : dotations aux d&#xE9;pr&#xE9;ciations</td>
                            <td>
                                <span class="text-right" t-esc="cdrc13" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Dotations aux provisions</td>
                            <td>
                                <span class="text-right" t-esc="cdrc14" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Autres charges</td>
                            <td>
                                <span class="text-right" t-esc="cdrc15" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-right"><strong>TOTAL I</strong></td>
                            <td>
                                <span class="text-right" t-esc="ct1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>Quotes-parts de r&#xE9;sultat sur op&#xE9;rations faites en commun ( II )</strong></td>
                            <td>
                                <span class="text-right" t-esc="cdrc16" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>                            
                            <td><strong>CHARGES FINANCI&#xC8;RES</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Dotations aux amortissements, aux d&#xE9;pr&#xE9;ciations et aux provisions</td>
                            <td>
                                <span class="text-right" t-esc="cdrc17" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Int&#xE9;r&#xEA;ts et charges assimil&#xE9;es</td>
                            <td>
                                <span class="text-right" t-esc="cdrc18" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Diff&#xE9;rences n&#xE9;gatives de change</td>
                            <td>
                                <span class="text-right" t-esc="cdrc19" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Charges nettes sur cessions de valeurs mobili&#xE8;res de placement</td>
                            <td>
                                <span class="text-right" t-esc="cdrc20" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-right"><strong>TOTAL III</strong></td>
                            <td>
                                <span class="text-right" t-esc="ct3" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>CHARGES EXCEPTIONNELLES</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Sur op&#xE9;rations de gestion</td>
                            <td>
                                <span class="text-right" t-esc="cdrc21" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Sur op&#xE9;rations en capital</td>
                            <td>
                                <span class="text-right" t-esc="cdrc22" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Dotations aux amortissements, aux d&#xE9;pr&#xE9;ciations et aux provisions</td>
                            <td>
                                <span class="text-right" t-esc="cdrc23" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-right"><strong>TOTAL IV</strong></td>
                            <td>
                                <span class="text-right" t-esc="ct4" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>Participation des salariés aux résultats ( V )</strong></td>
                            <td>
                                <span class="text-right" t-esc="cdrc24" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>Impôts sur les bénéfices ( VI )</strong></td>
                            <td>
                                <span class="text-right" t-esc="cdrc25" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-right"><strong>TOTAL CHARGES ( I + II + III + IV+ V+ VI )</strong></td>
                            <td>
                                <span class="text-right" t-esc="charges" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>

                    </tbody>
                </table>
                <table class="table table-sm">
                    <thead>
                        <tr>
                            <th>PRODUITS (hors taxes)</th>
                            <th></th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td><strong>PRODUITS D'EXPLOITATION</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Vente de marchandises</td>
                            <td>
                                <span class="text-right" t-esc="cdrp1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Production vendue [biens et services]</td>
                            <td>
                                <span class="text-right" t-esc="cdrp2" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>Sous-total A - Montant net du chiffre d'affaires</strong></td>
                            <td>
                                <span class="text-right" t-esc="pta" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Production stock&#xE9;e</td>
                            <td>
                                <span class="text-right" t-esc="cdrp3" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Production immobilis&#xE9;e</td>
                            <td>
                                <span class="text-right" t-esc="cdrp4" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Subventions d'exploitation</td>
                            <td>
                                <span class="text-right" t-esc="cdrp5" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Reprises sur provisions, d&#xE9;pr&#xE9;ciations (et amortissements) et transferts de charges</td>
                            <td>
                                <span class="text-right" t-esc="cdrp6" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Autres produits</td>
                            <td>
                                <span class="text-right" t-esc="cdrp7" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>Sous-total B</strong></td>
                            <td>
                                <span class="text-right" t-esc="ptb" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-right"><strong>TOTAL I ( A + B )</strong></td>
                            <td>
                                <span class="text-right" t-esc="pt1" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>Quotes-parts de r&#xE9;sultat sur op&#xE9;rations faites en commun (II)</strong></td>
                            <td>
                                <span class="text-right" t-esc="cdrp8" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>PRODUITS FINANCIERS</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>De participation</td>
                            <td>
                                <span class="text-right" t-esc="cdrp9" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>D'autres valeurs mobili&#xE8;res et cr&#xE9;ances de l'actif immobilis&#xE9;</td>
                            <td>
                                <span class="text-right" t-esc="cdrp10" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Autres int&#xE9;r&#xEA;ts et produits assimil&#xE9;s</td>
                            <td>
                                <span class="text-right" t-esc="cdrp11" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Reprises sur provisions, d&#xE9;pr&#xE9;ciations et transferts de charges</td>
                            <td>
                                <span class="text-right" t-esc="cdrp12" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Diff&#xE9;rences positives de change</td>
                            <td>
                                <span class="text-right" t-esc="cdrp13" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Produits nets sur cessions de valeurs mobili&#xE8;res de placement</td>
                            <td>
                                <span class="text-right" t-esc="cdrp14" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-right"><strong>TOTAL III</strong></td>
                            <td>
                                <span class="text-right" t-esc="pt3" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td><strong>PRODUITS EXCEPTIONNELS</strong></td>
                            <td></td>
                        </tr>
                        <tr>
                            <td>Sur op&#xE9;rations de gestion</td>
                            <td>
                                <span class="text-right" t-esc="cdrp15" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Sur op&#xE9;rations en capital</td>
                            <td>
                                <span class="text-right" t-esc="cdrp16" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td>Reprises sur provisions, d&#xE9;pr&#xE9;ciations et transferts de charges</td>
                            <td>
                                <span class="text-right" t-esc="cdrp17" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-right"><strong>TOTAL IV</strong></td>
                            <td>
                                <span class="text-right" t-esc="pt4" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-right"><strong>TOTAL DES PRODUITS ( I + II + III + IV )</strong></td>
                            <td>
                                <span class="text-right" t-esc="produits" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                        <tr>
                            <td class="text-right"><strong>PRODUITS - CHARGES</strong></td>
                            <td>
                                <span class="text-right" t-esc="produits-charges" t-options="{'widget': 'monetary', 'display_currency': res_company.currency_id}"/>
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </t>
    </t>
</template>
</data>
</odoo>

```

