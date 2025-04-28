# Odoo Module: l10n_fr_account

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2008 JAILLET Simon - CrysaLEAD - www.crysalead.fr
from . import models
from . import wizard

from odoo.addons.account.models.chart_template import preserve_existing_tags_on_taxes


def _l10n_fr_post_init_hook(env):
    _preserve_tag_on_taxes(env)
    _setup_inalterability(env)

def _preserve_tag_on_taxes(env):
    preserve_existing_tags_on_taxes(env, 'l10n_fr_account')

def _setup_inalterability(env):
    # enable ping for this module
    env['publisher_warranty.contract'].update_notification(cron_mode=True)

    fr_companies = env['res.company'].search([('partner_id.country_id.code', 'in', env['res.company']._get_france_country_codes())])
    if fr_companies:
        fr_companies._create_secure_sequence(['l10n_fr_closing_sequence_id'])

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'France - Accounting',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations/france.html',
    'icon': '/account/static/description/l10n.png',
    'version': '2.2',
    'countries': ['fr'],
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
in l10n_fr_account; companies that sell services to DOM-based companies should update the
configuration of their taxes and fiscal positions manually.

**Credits:** Sistheo, Zeekom, CrysaLEAD, Akretion and Camptocamp.
""",
    'depends': [
        'base_iban',
        'base_vat',
        'account',
        'l10n_fr',
    ],
    'auto_install': ['account'],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account_data.xml',
        'data/tax_report_data.xml',
        'views/report_invoice.xml',
        'views/res_partner_views.xml',
        'wizard/account_fr_fec_export_wizard_view.xml',
        'security/ir.model.access.csv',
        'data/res.bank.csv',
    ],
    'demo': [
        'data/l10n_fr_account_demo.xml',
    ],
    'post_init_hook': '_l10n_fr_post_init_hook',
    'license': 'LGPL-3',
}

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_fr_tag_salaires" model="account.account.tag">
        <field name="name">Salaries</field>
        <field name="applicability">accounts</field>
    </record>

      <record id="account_fr_tag_charges_sociales" model="account.account.tag">
        <field name="name">Social charges</field>
        <field name="applicability">accounts</field>
    </record>

    <menuitem id="account_reports_fr_statements_menu" name="France" parent="account.menu_finance_reports" sequence="5" groups="account.group_account_readonly"/>

    </odoo>

```

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <record forcecreate="True" id="display_name_in_footer_param" model="ir.config_parameter">
            <field name="key">account.display_name_in_footer</field>
            <field name="value">True</field>
        </record>
    </data>
</odoo>

```

## File: data\l10n_fr_account_demo.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <function model="account.chart.template" name="try_loading">
        <value eval="[]"/>
        <value>fr</value>
        <value model="res.company" eval="obj().env.ref('base.demo_company_fr')"/>
        <value name="install_demo" eval="True"/>
    </function>
</odoo>

```

## File: data\res.bank.csv

```csv
"id","name","bic","country:id"
"bank_fr_ccbpfrpp","BANQUES POPULAIRES-GROUPE BPCE","CCBPFRPP","base.fr"
"bank_fr_cepafrpp","CAISSES D'EPARGNE-GROUPE BPCE","CEPAFRPP","base.fr"
"bank_fr_sogefrpp","SOCIETE GENERALE","SOGEFRPP","base.fr"
"bank_fr_cmcifrpa","CREDIT MUTUEL - CIC BANQUES","CMCIFRPA","base.fr"
"bank_fr_agrifrpp","CREDIT AGRICOLE SA","AGRIFRPP","base.fr"
"bank_fr_crlyfrpp","CREDIT LYONNAIS","CRLYFRPP","base.fr"
"bank_fr_bnpafrpp","BNP PARIBAS SA","BNPAFRPP","base.fr"
"bank_fr_bousfrpp","BOURSORAMA","BOUSFRPP","base.fr"
"bank_fr_bredfrpp","BRED BANQUE POPULAIRE","BREDFRPP","base.fr"
"bank_fr_aecffr21","AMERICAN EXPRESS CARTE - FRANCE SA","AECFFR21","base.fr"
"bank_fr_psstfrpp","LA BANQUE POSTALE","PSSTFRPP","base.fr"
"bank_fr_cmbrfr2b","CREDIT MUTUEL ARKEA","CMBRFR2B","base.fr"
"bank_fr_ftnofrp1","FORTUNEO","FTNOFRP1","base.fr"
"bank_fr_bsavfr2c","BANQUE DE SAVOIE S.A.","BSAVFR2C","base.fr"
"bank_fr_axabfrpp","AXA BANQUE SA","AXABFRPP","base.fr"
"bank_fr_bbvafrpp","BANCO BILBAO VIZCAYA ARGENTARIA","BBVAFRPP","base.fr"
"bank_fr_bpsmfrpp","BANQUE BCP","BPSMFRPP","base.fr"
"bank_fr_agfbfrcc","ALLIANZ BANQUE S.A.","AGFBFRCC","base.fr"
"bank_fr_bnpafrph","BNP PARIBAS","BNPAFRPH","base.fr"
"bank_fr_trzofr21","TREEZOR","TRZOFR21","base.fr"
"bank_fr_nordfrpp","CREDIT DU NORD","NORDFRPP","base.fr"
"bank_fr_cmcifrpp","CM - CIC BANQUES","CMCIFRPP","base.fr"
"bank_fr_smctfr2a","SOCIETE MARSEILLAISE DE CREDIT","SMCTFR2A","base.fr"
"bank_fr_ccmvfr21","CAISSE DE CREDIT MUTUEL DE LAVAL BRETAGNE","CCMVFR21","base.fr"
"bank_fr_ccopfrcp","CREDIT COOPERATIF","CCOPFRCP","base.fr"
"bank_fr_courfr2t","BANQUE COURTOIS S.A.","COURFR2T","base.fr"
"bank_fr_ccopfrpp","CREDIT COOPERATIF","CCOPFRPP","base.fr"
"bank_fr_kolbfr21","BANQUE KOLB SA","KOLBFR21","base.fr"
"bank_fr_ralpfr2g","BANQUE RHONE ALPES","RALPFR2G","base.fr"
"bank_fr_tarnfr2l","BANQUE TARNEAUD","TARNFR2L","base.fr"
"bank_fr_cobafrpx","COMMERZBANK AG","COBAFRPX","base.fr"
"bank_fr_laydfr2w","BANQUE LAYDERNIER","LAYDFR2W","base.fr"
"bank_fr_kredfrpp","KBC BANK NV PARIS","KREDFRPP","base.fr"
"bank_fr_bnugfr21","BANQUE NUGER","BNUGFR21","base.fr"
"bank_fr_bspffrpp","BANQUE PALATINE","BSPFFRPP","base.fr"
"bank_fr_inrefr21","'INTERFIN' SOCIETE A RESPONSABILITE LIMITEE","INREFR21","base.fr"
"bank_fr_venrfrp1","123VENTURE SA","VENRFRP1","base.fr"
"bank_fr_gstifrp1","1818 GESTION","GSTIFRP1","base.fr"
"bank_fr_patffrp1","2020 PATRIMOINE FINANCE","PATFFRP1","base.fr"
"bank_fr_soclfrp1","21 SOCIETE CENTRALE POUR L'INDUSTRIE","SOCLFRP1","base.fr"
"bank_fr_amatfrp1","2I2C ASSET MANAGEMENT","AMATFRP1","base.fr"
"bank_fr_tsigfr22","3 SUISSES INTERNATIONAL","TSIGFR22","base.fr"
"bank_fr_amdefrp1","360 AM","AMDEFRP1","base.fr"
"bank_fr_asmrfrp1","360 ASSET MANAGERS","ASMRFRP1","base.fr"
"bank_fr_ivprfr21","38X INVESTMENT PARTNERS","IVPRFR21","base.fr"
"bank_fr_plfifrp1","A PLUS FINANCE SA","PLFIFRP1","base.fr"
"bank_fr_gettfr21","A.2. GESTION SAS","GETTFR21","base.fr"
"bank_fr_aavpfr21","A.V.I.P. - ASSURANCE VIE ET PREVOYANCE SA","AAVPFR21","base.fr"
"bank_fr_mamefrp1","A3M (ASSOCIATION DE MOYENS MALAKOFF MEDERIC)","MAMEFRP1","base.fr"
"bank_fr_aarbfrp1","AAREAL BANK AG, PARIS BRANCH","AARBFRP1","base.fr"
"bank_fr_aafafrp1","AAZ FINANCES SA","AAFAFRP1","base.fr"
"bank_fr_abaxfrp1","ABAXBOURSE","ABAXFRP1","base.fr"
"bank_fr_annffrl1","ABBEY NATIONAL FRANCE","ANNFFRL1","base.fr"
"bank_fr_antsfrp1","ABBEY NATIONAL TREASURY SERVICES PLC, PARIS BRANCH","ANTSFRP1","base.fr"
"bank_fr_aaeafrp1","ABC ARBITRAGE ASSET MANAGEMENT","AAEAFRP1","base.fr"
"bank_fr_abcofrpp","ABC INTERNATIONAL BANK PARIS BRANCH","ABCOFRPP","base.fr"
"bank_fr_ebapfrpp","ABE CLEARING SAS/EBA CLEARING","EBAPFRPP","base.fr"
"bank_fr_ebapfrps","ABE CLEARING SAS/EBA CLEARING (SYSTEM OPERATOR)","EBAPFRPS","base.fr"
"bank_fr_abtmfrp1","ABERDEEN ASSET MANAGEMENT","ABTMFRP1","base.fr"
"bank_fr_abegfrp1","ABF ASSET MANAGEMENT","ABEGFRP1","base.fr"
"bank_fr_abcgfrp1","ABF CAPITAL MANAGEMENT","ABCGFRP1","base.fr"
"bank_fr_aacifrp1","ABN AMRO COMMUNICATIONS INTERNATIONALES","AACIFRP1","base.fr"
"bank_fr_aafnfrp1","ABN AMRO FIXED INCOME FRANCE","AAFNFRP1","base.fr"
"bank_fr_aafffrp1","ABN AMRO FUTURES FRANCE","AAFFFRP1","base.fr"
"bank_fr_acfnfrp1","ACCEA FINANCE SOCIETE ANONYME","ACFNFRP1","base.fr"
"bank_fr_acpnfrp1","ACCESS CAPITAL PARTNERS","ACPNFRP1","base.fr"
"bank_fr_gracfrp2","ACCOR","GRACFRP2","base.fr"
"bank_fr_acemfrp1","ACE MANAGEMENT","ACEMFRP1","base.fr"
"bank_fr_aceffrp1","ACER FINANCE SOCIETE ANONYME","ACEFFRP1","base.fr"
"bank_fr_acgefrp1","ACOFI GESTION","ACGEFRP1","base.fr"
"bank_fr_acrofrp1","ACROPOLE AM","ACROFRP1","base.fr"
"bank_fr_acaafr21","ACTE IARD","ACAAFR21","base.fr"
"bank_fr_actafrp1","ACTI-BAIL","ACTAFRP1","base.fr"
"bank_fr_actgfrp1","ACTIGEP","ACTGFRP1","base.fr"
"bank_fr_actpfrp1","ACTION CONTREPARTIE","ACTPFRP1","base.fr"
"bank_fr_acsifrp1","ACTIS","ACSIFRP1","base.fr"
"bank_fr_acptfrp1","ACTIS PATRIMOINE SARL","ACPTFRP1","base.fr"
"bank_fr_adaafrp1","ADDAX ASSET MANAGEMENT","ADAAFRP1","base.fr"
"bank_fr_adfifrp1","ADEQUATION FINANCE SA","ADFIFRP1","base.fr"
"bank_fr_adanfrp1","ADI ALTERNATIVE INVESTMENTS","ADANFRP1","base.fr"
"bank_fr_adssfr22","ADISSEO FRANCE","ADSSFR22","base.fr"
"bank_fr_adrefrp1","ADREA","ADREFRP1","base.fr"
"bank_fr_amcufr21","ADREA MUTUELLE CENTRE AUVERGNE","AMCUFR21","base.fr"
"bank_fr_aedpfrp1","AEROPORTS DE PARIS","AEDPFRP1","base.fr"
"bank_fr_aegpfr21","AESOPE GESTION DE PORTEFEUILLES SAS","AEGPFR21","base.fr"
"bank_fr_afeafr21","AFI EUROPE IARD SA","AFEAFR21","base.fr"
"bank_fr_aferfr21","AFI EUROPE SAS","AFERFR21","base.fr"
"bank_fr_afopfr21","AFONE PAIEMENT","AFOPFR21","base.fr"
"bank_fr_afgefrp1","AFORGE GESTION SAS","AFGEFRP1","base.fr"
"bank_fr_agfcfr21","AGCO FINANCE SNC","AGFCFR21","base.fr"
"bank_fr_agebfrp1","AGEBANQUE","AGEBFRP1","base.fr"
"bank_fr_agfdfrp1","AGENCE FRANCAISE DE DEVELOPPEMENT","AGFDFRP1","base.fr"
"bank_fr_agfdfrpa","AGENCE FRANCAISE DE DEVELOPPEMENT","AGFDFRPA","base.fr"
"bank_fr_agfofr21","AGENCE FRANCE LOCALE","AGFOFR21","base.fr"
"bank_fr_aftrfrpp","AGENCE FRANCE TRESOR (AFT)","AFTRFRPP","base.fr"
"bank_fr_agfmfrp1","AGF ASSET MANAGEMENT","AGFMFRP1","base.fr"
"bank_fr_agcsfrp1","AGF CASH SNC","AGCSFRP1","base.fr"
"bank_fr_agfnfrp1","AGF FINANCEMENT 2","AGFNFRP1","base.fr"
"bank_fr_agiafrp1","AGICAM SA","AGIAFRP1","base.fr"
"bank_fr_aggsfrp1","AGILIS GESTION SA","AGGSFRP1","base.fr"
"bank_fr_agenfrp1","AGIPI ENERGIES","AGENFRP1","base.fr"
"bank_fr_agrdfr21","AGIRA RETRAITE DES CADRES","AGRDFR21","base.fr"
"bank_fr_agrsfr21","AGIRA RETRAITE DES SALARIES","AGRSFR21","base.fr"
"bank_fr_agprfr21","AGPM ASSURANCES","AGPRFR21","base.fr"
"bank_fr_aggtfrp1","AGREGATOR GESTION","AGGTFRP1","base.fr"
"bank_fr_agfsfr21","AGRI FINANCE SNC","AGFSFR21","base.fr"
"bank_fr_agnefr21","AGRI NEGOCE","AGNEFR21","base.fr"
"bank_fr_agepfrp1","AGRICA EPARGNE","AGEPFRP1","base.fr"
"bank_fr_aasffrp1","AGRIFIGEST-ALMA-SOCIETE FINANCIERE ET AGRICOLE DE GESTION","AASFFRP1","base.fr"
"bank_fr_aigvfr21","AIG VIE","AIGVFR21","base.fr"
"bank_fr_airffrpp","AIR FRANCE - KLM SA","AIRFFRPP","base.fr"
"bank_fr_rliqfrpp","AIR LIQUIDE SA","RLIQFRPP","base.fr"
"bank_fr_airbfr22","AIRBUS SAS","AIRBFR22","base.fr"
"bank_fr_akkafrpp","AKKA TECHNOLOGIES","AKKAFRPP","base.fr"
"bank_fr_licofrpp","AL KHALIJI FRANCE S.A.","LICOFRPP","base.fr"
"bank_fr_alggfrp1","ALBINGIA","ALGGFRP1","base.fr"
"bank_fr_atelfrpp","ALCATEL-LUCENT","ATELFRPP","base.fr"
"bank_fr_alsnfrpp","ALCATEL-LUCENT SUBMARINE NETWORKS","ALSNFRPP","base.fr"
"bank_fr_algefrp1","ALCIS GESTION","ALGEFRP1","base.fr"
"bank_fr_alsufrp1","ALCIS SECURITIES","ALSUFRP1","base.fr"
"bank_fr_alnifrp1","ALCYONE FINANCE","ALNIFRP1","base.fr"
"bank_fr_aldefr2v","ALDES AERAULIQUE","ALDEFR2V","base.fr"
"bank_fr_aalefr22","ALE INTERNATIONAL","AALEFR22","base.fr"
"bank_fr_alncfrp1","ALEXANDRE FINANCE","ALNCFRP1","base.fr"
"bank_fr_alpbfrp1","ALFABANQUE SA","ALPBFRP1","base.fr"
"bank_fr_alfifrp1","ALFI GESTION","ALFIFRP1","base.fr"
"bank_fr_aliofr21","ALICO","ALIOFR21","base.fr"
"bank_fr_altpfr21","ALIENOR CAPITAL","ALTPFR21","base.fr"
"bank_fr_alpgfrp1","ALIS CAPITAL MANAGEMENT","ALPGFRP1","base.fr"
"bank_fr_alztfrp1","ALIZE TRADING","ALZTFRP1","base.fr"
"bank_fr_alnpfrp1","ALLIANCE ENTREPRENDRE","ALNPFRP1","base.fr"
"bank_fr_aaamfrp1","ALLIANZ ALTERNATIVE ASSET MANAGEMENT SA","AAAMFRP1","base.fr"
"bank_fr_agfbfrpp","ALLIANZ BANQUE S.A.","AGFBFRPP","base.fr"
"bank_fr_allzfrpp","ALLIANZ IARD","ALLZFRPP","base.fr"
"bank_fr_alrbfrp1","ALLIED IRISH BANKS PLC (AIB)","ALRBFRP1","base.fr"
"bank_fr_amaifrp1","ALMA CAPITAL","AMAIFRP1","base.fr"
"bank_fr_almafrp1","ALMA FINANCE SARL","ALMAFRP1","base.fr"
"bank_fr_apesfrp1","ALOE PRIVATE EQUITY SAS","APESFRP1","base.fr"
"bank_fr_apgsfr21","ALPHA GEN SAS","APGSFR21","base.fr"
"bank_fr_aacbfr21","ALSACIENNE DE CREDIT-BAIL IMMOBILIER","AACBFR21","base.fr"
"bank_fr_alsofr21","ALSOLIA","ALSOFR21","base.fr"
"bank_fr_alhofrpp","ALSTOM HOLDINGS","ALHOFRPP","base.fr"
"bank_fr_alsyfrpp","ALSYON TECHNOLOGIES","ALSYFRPP","base.fr"
"bank_fr_algmfrp1","ALTADIS CIGARS INVESTMENTS","ALGMFRP1","base.fr"
"bank_fr_atsmfrp1","ALTAROCCA ASSET MANAGEMENT","ATSMFRP1","base.fr"
"bank_fr_faltfrp1","ALTAVIA","FALTFRP1","base.fr"
"bank_fr_almufr21","ALTEIS MUTUELLES GROUPEMENT D'INTERET ECONOMIQUE","ALMUFR21","base.fr"
"bank_fr_altnfrpp","ALTEN SA","ALTNFRPP","base.fr"
"bank_fr_aliifrp1","ALTER FINANCE CAPITAL SAS","ALIIFRP1","base.fr"
"bank_fr_altzfr21","ALTERGAZ","ALTZFR21","base.fr"
"bank_fr_aldvfrp1","ALTERNATIVE AND DERIVATIVE INVESTMENTS","ALDVFRP1","base.fr"
"bank_fr_agiffrp1","ALTERNATIVE LEADERS FRANCE","AGIFFRP1","base.fr"
"bank_fr_alaufr21","ALTIMA ASSURANCE","ALAUFR21","base.fr"
"bank_fr_altmfrp1","ALTIMEO AM","ALTMFRP1","base.fr"
"bank_fr_aamrfrp1","ALTIVIE ASSET MANAGEMENT DU GROUPE ALTIVIE SA","AAMRFRP1","base.fr"
"bank_fr_alnnfrp1","ALTO INVEST SA","ALNNFRP1","base.fr"
"bank_fr_alrafr22","ALTRAD INVESTMENT AUTHORITY","ALRAFR22","base.fr"
"bank_fr_gmtsfrp1","ALTRAN TECHNOLOGIES","GMTSFRP1","base.fr"
"bank_fr_altgfrp1","ALTUS FINANCE GESTION","ALTGFRP1","base.fr"
"bank_fr_allrfrp1","ALVEN CAPITAL PARTNERS","ALLRFRP1","base.fr"
"bank_fr_alvtfrpp","ALVEST","ALVTFRPP","base.fr"
"bank_fr_frnnfrp1","AM FRANCE","FRNNFRP1","base.fr"
"bank_fr_amfcfrpp","AM FRANCE","AMFCFRPP","base.fr"
"bank_fr_amaefrp1","AMADEIS","AMAEFRP1","base.fr"
"bank_fr_amsmfrp1","AMAIKA ASSET MANAGEMENT","AMSMFRP1","base.fr"
"bank_fr_amsafrp1","AMILTON ASSET MANAGEMENT","AMSAFRP1","base.fr"
"bank_fr_amgefrp1","AMIRAL GESTION","AMGEFRP1","base.fr"
"bank_fr_amplfrp1","AMPLEGEST","AMPLFRP1","base.fr"
"bank_fr_amgsfrp1","AMPLEO GESTION","AMGSFRP1","base.fr"
"bank_fr_amaufr21","AMT ASSURANCES","AMAUFR21","base.fr"
"bank_fr_agrifrpi","AMUNDI","AGRIFRPI","base.fr"
"bank_fr_aanafrp1","AMUNDI ALTERNATIVE INVESTMENTS SAS","AANAFRP1","base.fr"
"bank_fr_apeffrp1","AMUNDI PRIVATE EQUITY FUNDS","APEFFRP1","base.fr"
"bank_fr_crllfrp1","AMUNDI TENUE DE COMPTES","CRLLFRP1","base.fr"
"bank_fr_anagfrp1","ANAXIS ASSET MANAGEMENT","ANAGFRP1","base.fr"
"bank_fr_ammufrp1","ANCIENNE MAISON MARCEL BAUCHE SA","AMMUFRP1","base.fr"
"bank_fr_antufrp1","ANTARIUS SA","ANTUFRP1","base.fr"
"bank_fr_antrfrp1","ANTEIS EPARGNE SA","ANTRFRP1","base.fr"
"bank_fr_anenfr21","ANTENOR","ANENFR21","base.fr"
"bank_fr_anthfrp1","ANTHIUM FINANCE","ANTHFRP1","base.fr"
"bank_fr_antnfrp1","ANTIN BAIL SA","ANTNFRP1","base.fr"
"bank_fr_abtpfrp1","APAS BATIMENT TRAVAUX PUBLICS","ABTPFRP1","base.fr"
"bank_fr_apeifr21","APEI","APEIFR21","base.fr"
"bank_fr_apigfr22","APICIL GESTION","APIGFR22","base.fr"
"bank_fr_apigfrp1","APICIL GESTION","APIGFRP1","base.fr"
"bank_fr_apogfrp1","APOGE","APOGFRP1","base.fr"
"bank_fr_apdifrp1","APREP DIFFUSION SAS","APDIFRP1","base.fr"
"bank_fr_aprefr21","APREVA","APREFR21","base.fr"
"bank_fr_apasfr21","APRIL ASSURANCES SA","APASFR21","base.fr"
"bank_fr_apdvfr21","APRIL DEVELOPPEMENT","APDVFR21","base.fr"
"bank_fr_aglafr21","APRIL MARKETING SOLUTIONS SA","AGLAFR21","base.fr"
"bank_fr_apsnfrp1","APRIL SOLUTIONS ENTREPRISES SA","APSNFRP1","base.fr"
"bank_fr_apaefr21","APRIONIS ASSURANCES DE PERSONNES","APAEFR21","base.fr"
"bank_fr_aptgfr22","APTARGROUP HOLDING SAS","APTGFR22","base.fr"
"bank_fr_aqobfrp1","AQOBA EP","AQOBFRP1","base.fr"
"bank_fr_ariifrp1","ARABELLE INVESTISSEMENTS","ARIIFRP1","base.fr"
"bank_fr_arctfr21","ARC INTERNATIONAL","ARCTFR21","base.fr"
"bank_fr_arpafr21","ARCA - BANQUE DU PAYS BASQUE SA","ARPAFR21","base.fr"
"bank_fr_arcefrp1","ARCALIS ACTIF GENERAL LA POSTE","ARCEFRP1","base.fr"
"bank_fr_usinfrpp","ARCELORMITTAL TREASURY SNC","USINFRPP","base.fr"
"bank_fr_ahsafr21","ARCHIMED SAS","AHSAFR21","base.fr"
"bank_fr_ardofrp1","AREAS DOMMAGES","ARDOFRP1","base.fr"
"bank_fr_arevfrpp","AREVA SA","AREVFRPP","base.fr"
"bank_fr_arfnfrp1","ARFINCO SAS","ARFNFRP1","base.fr"
"bank_fr_aroffrp1","ARGOS SODITIC FRANCE","AROFFRP1","base.fr"
"bank_fr_argofrp1","ARGOVIE SOCIETE ANONYME","ARGOFRP1","base.fr"
"bank_fr_aracfr21","ARIAL ASSURANCE","ARACFR21","base.fr"
"bank_fr_arnnfr21","ARIANESPACE S.A","ARNNFR21","base.fr"
"bank_fr_arnnfr22","ARIANESPACE S.A","ARNNFR22","base.fr"
"bank_fr_aiisfrp1","ARIS","AIISFRP1","base.fr"
"bank_fr_arjifrp1","ARJIL SOCIETE PAR ACTIONS SIMPLIFIEE","ARJIFRP1","base.fr"
"bank_fr_arpgfr21","ARKEA CAPITAL GESTION","ARPGFR21","base.fr"
"bank_fr_arkefr22","ARKEMA","ARKEFR22","base.fr"
"bank_fr_arfcfrp1","ARKEON FINANCE SOCIETE ANONYME","ARFCFRP1","base.fr"
"bank_fr_arfifrp1","ARPEGE FINANCES SOCIETE ANONYME","ARFIFRP1","base.fr"
"bank_fr_arccfrp1","ARRCO","ARCCFRP1","base.fr"
"bank_fr_arrofrp1","ARROCHE","ARROFRP1","base.fr"
"bank_fr_grsrfr22","ART ET TECHNIQUES DU PROGRES","GRSRFR22","base.fr"
"bank_fr_arfrfrp1","ARTE FRANCE","ARFRFRP1","base.fr"
"bank_fr_aralfr21","ARTESIA BAIL SA","ARALFR21","base.fr"
"bank_fr_arslfrp1","ARVALIS INSTITUT DU VEGETAL","ARSLFRP1","base.fr"
"bank_fr_asfcfr21","ASCOT FINANCE","ASFCFR21","base.fr"
"bank_fr_acpgfrp1","ASF COMPTE PIVOT GROUPE ASF","ACPGFRP1","base.fr"
"bank_fr_aaadfrp1","ASSET ALLOCATION ADVISORS SA","AAADFRP1","base.fr"
"bank_fr_asmsfr21","ASSETFI MANAGEMENT SERVICES","ASMSFR21","base.fr"
"bank_fr_aeddfrp1","ASSOC EURO D'AUTOROUTES ET D'OUVRAGES A PEAGES","AEDDFRP1","base.fr"
"bank_fr_acclfrp1","ASSOCIATES COMMERCIAL CORPO LOCAVIA SAS","ACCLFRP1","base.fr"
"bank_fr_aaagfrp1","ASSOCIATION ADMINISTRATIVE AGRR","AAAGFRP1","base.fr"
"bank_fr_gmagfrp1","ASSOCIATION DE GESTION DU GROUPE MEDERIC","GMAGFRP1","base.fr"
"bank_fr_amacfrpp","ASSOCIATION DE MOYENS ASSURANCES","AMACFRPP","base.fr"
"bank_fr_amrtfrpp","ASSOCIATION DE MOYENS RETRAITE","AMRTFRPP","base.fr"
"bank_fr_asddfr21","ASSOCIATION DIOCESAINE D'ORLEANS","ASDDFR21","base.fr"
"bank_fr_adbofr21","ASSOCIATION DIOCESAINE DE BOURGES","ADBOFR21","base.fr"
"bank_fr_asdofr21","ASSOCIATION DIOCESAINE DE SOISSONS","ASDOFR21","base.fr"
"bank_fr_adpvfr21","ASSOCIATION DIOCESAINE DU PUY EN VELAY","ADPVFR21","base.fr"
"bank_fr_andpfrp1","ASSOCIATION NATIONALE D'ENTRAIDE ET DE PREVOYANCE","ANDPFRP1","base.fr"
"bank_fr_asrcfr21","ASSOCIATION RENOUVEAU CARMF","ASRCFR21","base.fr"
"bank_fr_aspvfrp1","ASSURANCE BANQUE POPULAIRE VIE","ASPVFRP1","base.fr"
"bank_fr_asctfrp1","ASSURANCE DU CREDIT MUTUEL","ASCTFRP1","base.fr"
"bank_fr_mumofr21","ASSURANCE MUTUELLE DES MOTARDS","MUMOFR21","base.fr"
"bank_fr_asprfr21","ASSURANCES BANQUE POPULAIRE IARD","ASPRFR21","base.fr"
"bank_fr_asarfr21","ASSURANCES CARREFOUR","ASARFR21","base.fr"
"bank_fr_acunfr21","ASSURANCES CREDIT MUTUEL NORD IARD","ACUNFR21","base.fr"
"bank_fr_acmdfrp1","ASSURANCES CREDIT MUTUEL NORD VIE","ACMDFRP1","base.fr"
"bank_fr_gsarfr21","ASSURATOME","GSARFR21","base.fr"
"bank_fr_asslfrp1","ASSURBAIL SA","ASSLFRP1","base.fr"
"bank_fr_asurfr21","ASSURVAL SOCIETE A RESPONSABILITE LIMITEE","ASURFR21","base.fr"
"bank_fr_assyfrpp","ASSYSTEM","ASSYFRPP","base.fr"
"bank_fr_asgefrp1","ASTRIA GESTION","ASGEFRP1","base.fr"
"bank_fr_ansafrp1","ATALANTE SAS","ANSAFRP1","base.fr"
"bank_fr_atlnfrpp","ATALIAN","ATLNFRPP","base.fr"
"bank_fr_atgefrp1","ATHYMIS GESTION","ATGEFRP1","base.fr"
"bank_fr_awfmfrp1","ATOS WORLDLINE FINANCIAL MARKETS","AWFMFRP1","base.fr"
"bank_fr_bcmafrpp","ATTIJARIWAFA BANK EUROPE-PARIS","BCMAFRPP","base.fr"
"bank_fr_aulofrp1","AUBOYNEAU, LABOURET, OLLIVIER S.A.","AULOFRP1","base.fr"
"bank_fr_audefr21","AUDIENS","AUDEFR21","base.fr"
"bank_fr_aueffr21","AUDIOVISUEL EXTERIEUR DE LA FRANCE","AUEFFR21","base.fr"
"bank_fr_aulsfrp1","AUREL BGC","AULSFRP1","base.fr"
"bank_fr_aulsfrp2","AUREL BGC","AULSFRP2","base.fr"
"bank_fr_aummfrp1","AUREL MONEY MARKET SNC","AUMMFRP1","base.fr"
"bank_fr_aupafrp1","AURIGA PARTNERS","AUPAFRP1","base.fr"
"bank_fr_augpfrp1","AURIS GESTION","AUGPFRP1","base.fr"
"bank_fr_anzbfrpp","AUSTRALIA AND NEW ZEALAND BANKING GROUP LIMITED","ANZBFRPP","base.fr"
"bank_fr_ausffrp1","AUTOROUTES DU SUD DE LA FRANCE SA","AUSFFRP1","base.fr"
"bank_fr_augefr21","AUXENSE GESTION","AUGEFR21","base.fr"
"bank_fr_auxbfrp1","AUXIA IMMOBILIER","AUXBFRP1","base.fr"
"bank_fr_auxffrp1","AUXIFIJP","AUXFFRP1","base.fr"
"bank_fr_auxpfr21","AUXIFIP","AUXPFR21","base.fr"
"bank_fr_avagfr21","AVENIR AGRO","AVAGFR21","base.fr"
"bank_fr_avfgfr21","AVENIR FINANCE GESTION SA","AVFGFR21","base.fr"
"bank_fr_afiafrp1","AVENIR FINANCE INVESTMENT MANAGERS SA","AFIAFRP1","base.fr"
"bank_fr_avlifr21","AVIP LILLE","AVLIFR21","base.fr"
"bank_fr_avcofr21","AVIVA COURTAGE","AVCOFR21","base.fr"
"bank_fr_avdifr21","AVIVA DIRECT","AVDIFR21","base.fr"
"bank_fr_avgafrp1","AVIVA GESTION D'ACTIFS SA","AVGAFRP1","base.fr"
"bank_fr_avvifr21","AVIVA VIE-SOCIETE ANONYME D'ASSURANCES VIE ET DE CAPITALISATION","AVVIFR21","base.fr"
"bank_fr_awfafrp1","AWF","AWFAFRP1","base.fr"
"bank_fr_axabfr21","AXA BANK EUROPE SCF","AXABFR21","base.fr"
"bank_fr_axfifrpp","AXA BANQUE FINANCEMENT","AXFIFRPP","base.fr"
"bank_fr_axfifr21","AXA BANQUE FINANCEMENT","AXFIFR21","base.fr"
"bank_fr_axcofrp1","AXA COURTAGE GROUPEMENT D'INTERET ECONOMIQUE","AXCOFRP1","base.fr"
"bank_fr_axcrfrp1","AXA CREDIT","AXCRFRP1","base.fr"
"bank_fr_axeefrp1","AXA EPARGNE ENTREPRISE SA","AXEEFRP1","base.fr"
"bank_fr_axfffrp1","AXA FRANCE FINANCE","AXFFFRP1","base.fr"
"bank_fr_axaffrpa","AXA FRANCE VIE","AXAFFRPA","base.fr"
"bank_fr_axfvfrp1","AXA FRANCE VIE SA","AXFVFRP1","base.fr"
"bank_fr_axiefr21","AXA INVESTMENT MANAGERS IF","AXIEFR21","base.fr"
"bank_fr_axigfrp1","AXA INVESTMENT MANAGERS PARIS SA","AXIGFRP1","base.fr"
"bank_fr_aimefrp1","AXA INVESTMENT MANAGERS PRIVATE EQUITY EUROPE","AIMEFRP1","base.fr"
"bank_fr_axpmfrp1","AXA PRIVATE MANAGEMENT","AXPMFRP1","base.fr"
"bank_fr_axarfr21","AXA REIM SGP","AXARFR21","base.fr"
"bank_fr_axltfrpp","AXELTIS SA","AXLTFRPP","base.fr"
"bank_fr_axrlfr22","AXEREAL FINANCES","AXRLFR22","base.fr"
"bank_fr_axirfr21","AXERIA IARD","AXIRFR21","base.fr"
"bank_fr_axprfr21","AXERIA PREVOYANCE","AXPRFR21","base.fr"
"bank_fr_axvifrp1","AXERIA VIE","AXVIFRP1","base.fr"
"bank_fr_scixfr21","AXIALIM","SCIXFR21","base.fr"
"bank_fr_axiofrp1","AXIOM ALTERNATIVE INVESTMENTS","AXIOFRP1","base.fr"
"bank_fr_azurfr21","AZUR - FINANCES","AZURFR21","base.fr"
"bank_fr_agmafrp1","AZUR-GMF MUTUELLES D'ASSURANCES ASSOCIEES SA","AGMAFRP1","base.fr"
"bank_fr_cpptfrp1","B SA","CPPTFRP1","base.fr"
"bank_fr_bfctfrp1","B.F.T. COURT TERME S.I.C.A.V","BFCTFRP1","base.fr"
"bank_fr_bmabfrp1","B.M.A. - BANQUE DE MARCHES ET D'ARBITRAGE SA","BMABFRP1","base.fr"
"bank_fr_btvpfrp1","B2V","BTVPFRP1","base.fr"
"bank_fr_bcfifrp1","BACHE COMMODITIES LIMITED","BCFIFRP1","base.fr"
"bank_fr_baaufr21","BADENIA BAUSPARKASSE AG","BAAUFR21","base.fr"
"bank_fr_baiifr22","BAI SA","BAIIFR22","base.fr"
"bank_fr_batlfr21","BAIL ACTEA SA","BATLFR21","base.fr"
"bank_fr_bapofr21","BAIL BANQUE POPULAIRE","BAPOFR21","base.fr"
"bank_fr_baenfrp1","BAIL ECONOMIE","BAENFRP1","base.fr"
"bank_fr_baeifrp1","BAIL ECUREUIL","BAEIFRP1","base.fr"
"bank_fr_baepfr21","BAIL ENTREPRISES SOCIETE PAR ACTIONS SIMPLIFIEE","BAEPFR21","base.fr"
"bank_fr_biiofr21","BAIL IMMO NORD SA","BIIOFR21","base.fr"
"bank_fr_bavnfrp1","BAIL INVESTISSEMENT","BAVNFRP1","base.fr"
"bank_fr_bahtfrp1","BALZAC SHORT TERM EURO SICAV","BAHTFRP1","base.fr"
"bank_fr_crgefr2x","BANCA CARIGE SPA","CRGEFR2X","base.fr"
"bank_fr_bepofr21","BANCA POPOLARE DI BERGAMO - CREDITO VARESINO","BEPOFR21","base.fr"
"bank_fr_breufr22","BANCA REGIONALE EUROPEA S.P.A.","BREUFR22","base.fr"
"bank_fr_bbpifrpp","BANCO BPI, PARIS","BBPIFRPP","base.fr"
"bank_fr_bsabfrpp","BANCO DE SABADELL","BSABFRPP","base.fr"
"bank_fr_brasfrpp","BANCO DO BRASIL AG SUCCURSALE FRANCE","BRASFRPP","base.fr"
"bank_fr_espcfrp1","BANCO ESPANOL DE CREDITO","ESPCFRP1","base.fr"
"bank_fr_guhefr21","BANCO GUIPUZCUANO HENDAYE","GUHEFR21","base.fr"
"bank_fr_bschfrpp","BANCO SANTANDER S.A.","BSCHFRPP","base.fr"
"bank_fr_audifrpp","BANK AUDI FRANCE","AUDIFRPP","base.fr"
"bank_fr_melifrpp","BANK MELLI IRAN","MELIFRPP","base.fr"
"bank_fr_bofafrpp","BANK OF AMERICA MERRILL LYNCH INTERNATIONAL LIMITED","BOFAFRPP","base.fr"
"bank_fr_bkchfrpp","BANK OF CHINA, PARIS BRANCH","BKCHFRPP","base.fr"
"bank_fr_commfrpp","BANK OF COMMUNICATIONS (LUXEMBOURG) S.A. PARIS BRANCH","COMMFRPP","base.fr"
"bank_fr_bkidfrpp","BANK OF INDIA","BKIDFRPP","base.fr"
"bank_fr_bofsfrp1","BANK OF SCOTLAND","BOFSFRP1","base.fr"
"bank_fr_botkfrcc","BANK OF TOKYO-MITSUBISHI UFJ, LTD., THE","BOTKFRCC","base.fr"
"bank_fr_botkfrpx","BANK OF TOKYO-MITSUBISHI UFJ, LTD., THE","BOTKFRPX","base.fr"
"bank_fr_sdinfrp1","BANK SADERAT IRAN","SDINFRP1","base.fr"
"bank_fr_sepbfrp1","BANK SEPAH","SEPBFRP1","base.fr"
"bank_fr_sepbfrpp","BANK SEPAH","SEPBFRPP","base.fr"
"bank_fr_btejfrpp","BANK TEJARAT","BTEJFRPP","base.fr"
"bank_fr_bktffrp1","BANKERS TRUST (FRANCE) SA","BKTFFRP1","base.fr"
"bank_fr_bkoafr21","BANKOA SA","BKOAFR21","base.fr"
"bank_fr_alcyfr21","BANQUE ALCYON","ALCYFR21","base.fr"
"bank_fr_arjbfrp1","BANQUE ARJIL ET COMPAGNIE","ARJBFRP1","base.fr"
"bank_fr_arngfr21","BANQUE ARNAUD-GAIDAN","ARNGFR21","base.fr"
"bank_fr_biarfrpp","BANQUE BIA","BIARFRPP","base.fr"
"bank_fr_bcgefr21","BANQUE CANTONALE DE GENEVE FRANCE SA","BCGEFR21","base.fr"
"bank_fr_bacpfrpp","BANQUE CENTRALE DE COMPENSATION - LCH.CLEARNET SA","BACPFRPP","base.fr"
"bank_fr_bcpofrp1","BANQUE CENTRALE POPULAIRE","BCPOFRP1","base.fr"
"bank_fr_bcdmfrpp","BANQUE CHAABI DU MAROC","BCDMFRPP","base.fr"
"bank_fr_bcibfrp1","BANQUE CHABRIERES SA","BCIBFRP1","base.fr"
"bank_fr_bchafr21","BANQUE CHALUS SA","BCHAFR21","base.fr"
"bank_fr_bdfefr2t","BANQUE DE FRANCE","BDFEFR2T","base.fr"
"bank_fr_bdfefr31","BANQUE DE FRANCE","BDFEFR31","base.fr"
"bank_fr_bdfefrpp","BANQUE DE FRANCE","BDFEFRPP","base.fr"
"bank_fr_rgfifrpp","BANQUE DE REALISATIONS DE GESTION ET DE FINANCEMENT","RGFIFRPP","base.fr"
"bank_fr_altrfrp1","BANQUE DEGROOF PETERCAM FRANCE","ALTRFRP1","base.fr"
"bank_fr_delufr22","BANQUE DELUBAC ET CIE","DELUFR22","base.fr"
"bank_fr_bapyfr21","BANQUE DES PYRENEES","BAPYFR21","base.fr"
"bank_fr_tuilfrp1","BANQUE DES TUILERIES","TUILFRP1","base.fr"
"bank_fr_batifrp1","BANQUE DU BATIMENT ET DES TRAVAUX PUBLICS SA","BATIFRP1","base.fr"
"bank_fr_grcsfrp1","BANQUE DU GROUPE CASINO SOCIETE ANONYME","GRCSFRP1","base.fr"
"bank_fr_bdupfr2s","BANQUE DUPUY, DE PARSEVAL","BDUPFR2S","base.fr"
"bank_fr_edelfrp1","BANQUE EDEL SNC","EDELFRP1","base.fr"
"bank_fr_besvfrpp","BANQUE ESPIRITO SANTO ET DE LA VENETIE","BESVFRPP","base.fr"
"bank_fr_fidcfr21","BANQUE FIDUCIAL","FIDCFR21","base.fr"
"bank_fr_filnfr21","BANQUE FINANCIAL","FILNFR21","base.fr"
"bank_fr_bfcafrp1","BANQUE FINANCIERE CARDIF","BFCAFRP1","base.fr"
"bank_fr_bfcofrpp","BANQUE FRANCAISE COMMERCIALE DE L'OCEAN INDIEN","BFCOFRPP","base.fr"
"bank_fr_bfdifrp1","BANQUE FRANCAISE D'INVESTISSEMENT","BFDIFRP1","base.fr"
"bank_fr_fembfrp1","BANQUE FRANCAISE MUTUALISTE","FEMBFRP1","base.fr"
"bank_fr_femufr21","BANQUE FRANCAISE MUTUALISTE","FEMUFR21","base.fr"
"bank_fr_frabfrp1","BANQUE FRANCAISE SA","FRABFRP1","base.fr"
"bank_fr_gravfrp1","BANQUE GRAVEREAU","GRAVFRP1","base.fr"
"bank_fr_bgrdfr21","BANQUE GUIRAUD ET CIE","BGRDFR21","base.fr"
"bank_fr_hrfrfrp1","BANQUE HOTTINGUER","HRFRFRP1","base.fr"
"bank_fr_bincfrp1","BANQUE INCHAUSPE ET CIE","BINCFRP1","base.fr"
"bank_fr_bicffrpp","BANQUE INTERNATIONALE DE COMMERCE-BRED","BICFFRPP","base.fr"
"bank_fr_bifefrp1","BANQUE INTERNATIONALE DE FINANCEMENT ET DE NEGOCIATION - BIFEN","BIFEFRP1","base.fr"
"bank_fr_fipifrp1","BANQUE LEONARDO","FIPIFRP1","base.fr"
"bank_fr_majofr21","BANQUE MAJOREL SCA","MAJOFR21","base.fr"
"bank_fr_magifr21","BANQUE MARIN ET GIANOLA","MAGIFR21","base.fr"
"bank_fr_bmmmfrcp","BANQUE MARTIN-MAUREL","BMMMFRCP","base.fr"
"bank_fr_bmmmfr2a","BANQUE MARTIN-MAUREL","BMMMFR2A","base.fr"
"bank_fr_bmrzfr21","BANQUE MARZE SA","BMRZFR21","base.fr"
"bank_fr_bamyfr22","BANQUE MICHEL INCHAUSPE - BAMI","BAMYFR22","base.fr"
"bank_fr_bmisfrpp","BANQUE MISR - SUCCURSALE DE PARIS","BMISFRPP","base.fr"
"bank_fr_mofifr21","BANQUE MONETAIRE ET FINANCIERE SA","MOFIFR21","base.fr"
"bank_fr_nsmbfrpp","BANQUE NEUFLIZE OBC","NSMBFRPP","base.fr"
"bank_fr_nomafrp1","BANQUE NOMURA FRANCE SA","NOMAFRP1","base.fr"
"bank_fr_bpgdfrp1","BANQUE PARISIENNE DE GESTION ET DE DEPOTS","BPGDFRP1","base.fr"
"bank_fr_paimfrp1","BANQUE PATRIMOINE ET IMMOBILIER","PAIMFRP1","base.fr"
"bank_fr_gepifrp1","BANQUE POSTALE GESTION PRIVEE","GEPIFRP1","base.fr"
"bank_fr_inplfrp1","BANQUE POUR INVEST. PROF. LIBERALES","INPLFRP1","base.fr"
"bank_fr_pouyfr21","BANQUE POUYANNE SA","POUYFR21","base.fr"
"bank_fr_preufrp1","BANQUE PRIVEE EUROPEENNE SA","PREUFRP1","base.fr"
"bank_fr_palmfrp1","BANQUE PRIVEE SAINT DOMINIQUE SA","PALMFRP1","base.fr"
"bank_fr_psabfrpp","BANQUE PSA FINANCE SA","PSABFRPP","base.fr"
"bank_fr_revifrpp","BANQUE REVILLON","REVIFRPP","base.fr"
"bank_fr_robofrp1","BANQUE ROBECO SA","ROBOFRP1","base.fr"
"bank_fr_bsaffrp1","BANQUE SAFRA","BSAFFRP1","base.fr"
"bank_fr_bsaofr21","BANQUE SAINT OLIVE SA","BSAOFR21","base.fr"
"bank_fr_sbaafrpp","BANQUE SBA","SBAAFRPP","base.fr"
"bank_fr_basofrp1","BANQUE SOCREDIT","BASOFRP1","base.fr"
"bank_fr_bpetfrp1","BANQUE SOLFEA SOCIETE ANONYME","BPETFRP1","base.fr"
"bank_fr_bdeifrpp","BANQUE THEMIS","BDEIFRPP","base.fr"
"bank_fr_travfrpp","BANQUE TRAVELEX SA","TRAVFRPP","base.fr"
"bank_fr_travfrp1","BANQUE TRAVELEX SA","TRAVFRP1","base.fr"
"bank_fr_escbfrpp","BANQUE WORMSER FRERES / BANQUE D'ESCOMPTE","ESCBFRPP","base.fr"
"bank_fr_bpcbfrp1","BANQUES POPULAIRES COVERED BONDS","BPCBFRP1","base.fr"
"bank_fr_bablfrp1","BARCLAYS BAIL SA","BABLFRP1","base.fr"
"bank_fr_barcfrpc","BARCLAYS BANK PLC FRANCE","BARCFRPC","base.fr"
"bank_fr_barcfrpp","BARCLAYS BANK PLC FRANCE","BARCFRPP","base.fr"
"bank_fr_barcfr21","BARCLAYS CAPITAL FRANCE SA","BARCFR21","base.fr"
"bank_fr_privfrpp","BARCLAYS FRANCE S.A.","PRIVFRPP","base.fr"
"bank_fr_blvifrp1","BARCLAYS VIE","BLVIFRP1","base.fr"
"bank_fr_bwmrfrp1","BARCLAYS WEALTH MANAGERS FRANCE","BWMRFRP1","base.fr"
"bank_fr_batgfr21","BAREP ASSET MANAGEMENT SA","BATGFR21","base.fr"
"bank_fr_bseffrp1","BARING SECURITIES BOURSE S.A.","BSEFFRP1","base.fr"
"bank_fr_btilfr21","BATI-LEASE","BTILFR21","base.fr"
"bank_fr_bttcfr21","BATICAL","BTTCFR21","base.fr"
"bank_fr_btcafr21","BATICENTRE BATIROC-CENTRE","BTCAFR21","base.fr"
"bank_fr_bacnfrp1","BATICENTREST","BACNFRP1","base.fr"
"bank_fr_batffr21","BATIFRANC SA","BATFFR21","base.fr"
"bank_fr_batmfrp1","BATIMAP SA","BATMFRP1","base.fr"
"bank_fr_bauifrp1","BATIMUR SAS","BAUIFRP1","base.fr"
"bank_fr_baoifr21","BATINOREST SOCIETE ANONYME","BAOIFR21","base.fr"
"bank_fr_baoefr21","BATIROC NORMANDIE SOCIETE ANONYME","BAOEFR21","base.fr"
"bank_fr_baylfr21","BATIROC PAYS DE LA LOIRE SA","BAYLFR21","base.fr"
"bank_fr_bawhfr21","BAUSPARKASSE SCHWABISCH HALL AG","BAWHFR21","base.fr"
"bank_fr_bylafrp1","BAYERISCHE LANDESBANK, SUCCURSALE DE PARIS","BYLAFRP1","base.fr"
"bank_fr_bbrrfrp1","BBR ROGIER SOCIETE ANONYME","BBRRFRP1","base.fr"
"bank_fr_bdftfrp1","BDF-GESTION","BDFTFRP1","base.fr"
"bank_fr_bdlbfrp1","BDL","BDLBFRP1","base.fr"
"bank_fr_bstnfrp1","BEAR STEARNS BANK PUBLIC LIMITED COMPANY","BSTNFRP1","base.fr"
"bank_fr_betifrp1","BEAR STEARNS INTERNATIONAL LTD","BETIFRP1","base.fr"
"bank_fr_beipfrp1","BELACO CAPITAL","BEIPFRP1","base.fr"
"bank_fr_bepvfrp1","BELLINI PREVOYANCE","BEPVFRP1","base.fr"
"bank_fr_bemofrpp","BEMO EUROPE BANQUE PRIVEE","BEMOFRPP","base.fr"
"bank_fr_bntofr2s","BENETEAU","BNTOFR2S","base.fr"
"bank_fr_bedrfrp1","BERNHEIM DREYFUS AND CO SA","BEDRFRP1","base.fr"
"bank_fr_bemifrp1","BERTRAND MICHEL","BEMIFRP1","base.fr"
"bank_fr_betwfrp1","BETWEEN","BETWFRP1","base.fr"
"bank_fr_bfbkfrp1","BFORBANK","BFBKFRP1","base.fr"
"bank_fr_bfgefrp1","BFT GESTION","BFGEFRP1","base.fr"
"bank_fr_bgfifrpp","BGFIBANK EUROPE","BGFIFRPP","base.fr"
"bank_fr_bifafr21","BIBBY FACTOR FRANCE SA","BIFAFR21","base.fr"
"bank_fr_sobcfr21","BIC","SOBCFR21","base.fr"
"bank_fr_bifnfrp1","BIL FINANCE","BIFNFRP1","base.fr"
"bank_fr_bikcfrp1","BINCKBANK","BIKCFRP1","base.fr"
"bank_fr_biomfr22","BIOMERIEUX","BIOMFR22","base.fr"
"bank_fr_bliafr21","BLACKROCK INVESTMENT MANAGEMENT (UK) LIMITED","BLIAFR21","base.fr"
"bank_fr_blcgfrp1","BLC GESTION SA","BLCGFRP1","base.fr"
"bank_fr_blomfrpp","BLOM BANK FRANCE","BLOMFRPP","base.fr"
"bank_fr_blpsfrp1","BLUEHIVE CAPITAL SAS","BLPSFRP1","base.fr"
"bank_fr_bmsofr23","BM SOFTWARE","BMSOFR23","base.fr"
"bank_fr_bmcefrpp","BMCE BANK INTERNATIONAL PLC SUCCURSALE EN FRANCE","BMCEFRPP","base.fr"
"bank_fr_bmeufrp1","BMCE EUROSERVICES","BMEUFRP1","base.fr"
"bank_fr_bmamfr21","BMG ASSET MANAGEMENT SA","BMAMFR21","base.fr"
"bank_fr_bmfcfr21","BMW FINANCE SNC","BMFCFR21","base.fr"
"bank_fr_bmlefr21","BMW LEASE SNC","BMLEFR21","base.fr"
"bank_fr_bnemfrp1","BNP EMERGIS","BNEMFRP1","base.fr"
"bank_fr_bnabfrpp","BNP PARIBAS ARBITRAGE","BNABFRPP","base.fr"
"bank_fr_parbfrpm","BNP PARIBAS ASSET MANAGEMENT","PARBFRPM","base.fr"
"bank_fr_bppafr21","BNP PARIBAS ASSURANCE SA","BPPAFR21","base.fr"
"bank_fr_bpppfrp1","BNP PARIBAS CAPITAL PARTNERS","BPPPFRP1","base.fr"
"bank_fr_bnpdfrp1","BNP PARIBAS CARDIF","BNPDFRP1","base.fr"
"bank_fr_bpcdfrp1","BNP PARIBAS COVERED BONDS","BPCDFRP1","base.fr"
"bank_fr_bnpcfr21","BNP PARIBAS FACTOR SA","BNPCFR21","base.fr"
"bank_fr_famsfrpp","BNP PARIBAS FINAMS","FAMSFRPP","base.fr"
"bank_fr_bnpafrpl","BNP PARIBAS HOME LOAN SFH","BNPAFRPL","base.fr"
"bank_fr_bpnifrp1","BNP PARIBAS INVEST IMMO","BPNIFRP1","base.fr"
"bank_fr_bplgfr21","BNP PARIBAS LEASE GROUP SA","BPLGFR21","base.fr"
"bank_fr_cetefrp1","BNP PARIBAS PERSONAL FINANCE","CETEFRP1","base.fr"
"bank_fr_bppbfrp1","BNP PARIBAS PRIVATE BANK SA","BPPBFRP1","base.fr"
"bank_fr_bnpafrps","BNP PARIBAS PUBLIC SECTOR SCF","BNPAFRPS","base.fr"
"bank_fr_bpasfrpp","BNP PARIBAS SECURITIES SERVICES","BPASFRPP","base.fr"
"bank_fr_parbfrph","BNP PARIBAS SECURITIES SERVICES","PARBFRPH","base.fr"
"bank_fr_parbfrpp","BNP PARIBAS SECURITIES SERVICES, FRANCE","PARBFRPP","base.fr"
"bank_fr_bnpafrt1","BNP PARIBAS-THEAM","BNPAFRT1","base.fr"
"bank_fr_afrifrpp","BOA-FRANCE","AFRIFRPP","base.fr"
"bank_fr_schnfr22","BOISSIERE FINANCE","SCHNFR22","base.fr"
"bank_fr_sneifr22","BOISSIERE FINANCE","SNEIFR22","base.fr"
"bank_fr_bosffr21","BOISSY FINANCES SA","BOSFFR21","base.fr"
"bank_fr_bollfr22","BOLLORE SA","BOLLFR22","base.fr"
"bank_fr_bocnfr21","BOMBARDIER CAPITAL INTERNATIONAL","BOCNFR21","base.fr"
"bank_fr_bndufr22","BONDUELLE S.A.","BNDUFR22","base.fr"
"bank_fr_bongfr21","BONGRAIN SA","BONGFR21","base.fr"
"bank_fr_bonafrpp","BONNA SABLA SA","BONAFRPP","base.fr"
"bank_fr_bogpfrp1","BORDIER GESTION PRIVEE SA","BOGPFRP1","base.fr"
"bank_fr_boscfrp1","BOSCHER","BOSCFRP1","base.fr"
"bank_fr_bfcsfr21","BOURGOGNE FRANCHE COMTE CONSTRUCTIONS SARL","BFCSFR21","base.fr"
"bank_fr_bogafr21","BOURGOGNE-GARANTIE SA","BOGAFR21","base.fr"
"bank_fr_bdirfrpp","BOURSE DIRECT SA","BDIRFRPP","base.fr"
"bank_fr_boggfrp1","BOUSSARD AND GAVAUDAN GESTION SAS","BOGGFRP1","base.fr"
"bank_fr_boutfrp1","BOUVIER GESTION SA","BOUTFRP1","base.fr"
"bank_fr_bycnfr22","BOUYGUES CONSTRUCTION","BYCNFR22","base.fr"
"bank_fr_byimfr22","BOUYGUES IMMOBILIER","BYIMFR22","base.fr"
"bank_fr_bouyfrp1","BOUYGUES SA","BOUYFRP1","base.fr"
"bank_fr_fncefr21","BP FINANCE SARL","FNCEFR21","base.fr"
"bank_fr_fracfr21","BP FRANCE SA","FRACFR21","base.fr"
"bank_fr_bpcefrpp","BPCE","BPCEFRPP","base.fr"
"bank_fr_sdbffrpp","BPCE DEFI","SDBFFRPP","base.fr"
"bank_fr_bpaxfr22","BPIFRANCE ASSURANCE EXPORT","BPAXFR22","base.fr"
"bank_fr_cpmefrpp","BPIFRANCE FINANCEMENT","CPMEFRPP","base.fr"
"bank_fr_bpivfr21","BPIFRANCE INVESTISSEMENT","BPIVFR21","base.fr"
"bank_fr_bpnpfrp1","BPN-BANCO PORTUGUES DE NEGOCIOS SA","BPNPFRP1","base.fr"
"bank_fr_brncfrp1","BRANICS","BRNCFRP1","base.fr"
"bank_fr_brolfrp1","BRED COFILEASE","BROLFRP1","base.fr"
"bank_fr_brgefrpp","BRED GESTION","BRGEFRPP","base.fr"
"bank_fr_brgifrp1","BRETEUIL GESTION PRIVEE SA","BRGIFRP1","base.fr"
"bank_fr_bfrffrpp","BRINK'S FRANCE FINANCE","BFRFFRPP","base.fr"
"bank_fr_brrafrp1","BRYAN GARNIER AM","BRRAFRP1","base.fr"
"bank_fr_brgafrp1","BRYAN GARNIER AND CO LIMITED","BRGAFRP1","base.fr"
"bank_fr_btprfr21","BTP PRO SARL","BTPRFR21","base.fr"
"bank_fr_btrtfrp1","BTP RETRAITE RTA","BTRTFRP1","base.fr"
"bank_fr_buiofrp1","BUISSON ET CIE","BUIOFRP1","base.fr"
"bank_fr_busafrpp","BULL","BUSAFRPP","base.fr"
"bank_fr_bufifr21","BULL FINANCE SA","BUFIFR21","base.fr"
"bank_fr_bufrfr21","BULL FRANCE","BUFRFR21","base.fr"
"bank_fr_kiaifr22","BUNSHA INTERNATIONAL","KIAIFR22","base.fr"
"bank_fr_buvefrpp","BUREAU VERITAS - REGISTRE INTERNATIONAL DE CLASSIFICATION DE NAVIRES ET D'AERONEFS","BUVEFRPP","base.fr"
"bank_fr_bflcfr21","BUSINESS FLOW CONSULTING SAS","BFLCFR21","base.fr"
"bank_fr_bucefrp1","BUTLER CAPITAL PARTNERS","BUCEFRP1","base.fr"
"bank_fr_buysfrp1","BUYSTER","BUYSFRP1","base.fr"
"bank_fr_bybbfrpp","BYBLOS BANK EUROPE S.A. (PARIS BRANCH)","BYBBFRPP","base.fr"
"bank_fr_bygffrp1","BYFIELD GLOBAL FUND LIMITED","BYGFFRP1","base.fr"
"bank_fr_bzinfrp1","BZX INVESTMENTS LTD","BZINFRP1","base.fr"
"bank_fr_sefefr21","C-FER-J SA","SEFEFR21","base.fr"
"bank_fr_ccetfr21","C.C.P.B EST","CCETFR21","base.fr"
"bank_fr_ceulfr21","C.C.P.B EURE ET LOIRE","CEULFR21","base.fr"
"bank_fr_cchrfr21","C.C.P.B HAUT RHIN","CCHRFR21","base.fr"
"bank_fr_cctofr21","C.C.P.B TOULOUSE","CCTOFR21","base.fr"
"bank_fr_ccvbfr22","C.C.V. BEAUMANOIR","CCVBFR22","base.fr"
"bank_fr_cghtfrp1","C.G.M. INTERMEDIATION S.A.","CGHTFRP1","base.fr"
"bank_fr_ccdvfr21","C.R.C.A.M. CHARENTE-MARITIME DEUX SEVRES","CCDVFR21","base.fr"
"bank_fr_crnufr21","C.R.C.A.M. DE LA TOURAINE ET DU POITOU","CRNUFR21","base.fr"
"bank_fr_cpcdfr21","C.R.C.A.M. PROVENCE COTE D'AZUR","CPCDFR21","base.fr"
"bank_fr_colqfr21","C2A COMPAGNIE DE L'ARC ATLANTIQUE","COLQFR21","base.fr"
"bank_fr_agrlfr22","CA CONSUMER FINANCE","AGRLFR22","base.fr"
"bank_fr_siblfrpp","CA INDOSUEZ WEALTH (FRANCE)","SIBLFRPP","base.fr"
"bank_fr_cabefr21","CABINET BESSE SOCIETE D'EXERCICE LIBERAL A RESPONSABILITE LIMITEE","CABEFR21","base.fr"
"bank_fr_isaefrpp","CACEIS BANK","ISAEFRPP","base.fr"
"bank_fr_cctefrp1","CACEIS CORPORATE TRUST (CACEIS CT)","CCTEFRP1","base.fr"
"bank_fr_fnetfrpp","CACEIS FUND ADMINISTRATION","FNETFRPP","base.fr"
"bank_fr_agrjfrpp","CACI - CREDIT AGRICOLE CREDITOR INSURANCE SA","AGRJFRPP","base.fr"
"bank_fr_cadefrp1","CADES","CADEFRP1","base.fr"
"bank_fr_cautfr21","CAISSE ASSURANCE MUTUELLE DU BTP","CAUTFR21","base.fr"
"bank_fr_cavhfrp1","CAISSE ASSURANCE VIEILLESSE PHARMACIE","CAVHFRP1","base.fr"
"bank_fr_caarfrp1","CAISSE AUTONOME DE REFINANCEMENT","CAARFRP1","base.fr"
"bank_fr_caeyfrp1","CAISSE AUTONOME DE RETRAITES ET DE PREVOYANCE DES VETERINAIRES","CAEYFRP1","base.fr"
"bank_fr_curifrp1","CAISSE AUTONOME RETRAITE CHIRURGIENS DENTISTE ET SAGES-FEMMES","CURIFRP1","base.fr"
"bank_fr_ccdofr21","CAISSE CENTRALE D'ACTIVITES SOCIALES","CCDOFR21","base.fr"
"bank_fr_caaufrp1","CAISSE CENTRALE DE REASSURANCE S.A.","CAAUFRP1","base.fr"
"bank_fr_cciffrc1","CAISSE CENTRALE DU CREDIT IMMOBILIER DE FRANCE","CCIFFRC1","base.fr"
"bank_fr_cciffrpp","CAISSE CENTRALE DU CREDIT IMMOBILIER DE FRANCE","CCIFFRPP","base.fr"
"bank_fr_ccabfr21","CAISSE CONGES PAYES BAT COTE AZUR CORSE","CCABFR21","base.fr"
"bank_fr_cbcrfr21","CAISSE DE BRETAGNE DE CREDIT AGRICOLE MUTUEL SOCIETE COOPERATIVE A CAPITAL VARIABLE","CBCRFR21","base.fr"
"bank_fr_ccybfr21","CAISSE DE CONGES PAYES DU BATIMENT","CCYBFR21","base.fr"
"bank_fr_ccpefr21","CAISSE DE CONGES PAYES DU BATIMENT (CGE)","CCPEFR21","base.fr"
"bank_fr_cscafr21","CAISSE DE CREDIT MUNICIPAL D'AVIGNON","CSCAFR21","base.fr"
"bank_fr_ccmofr21","CAISSE DE CREDIT MUNICIPAL DE BORDEAUX","CCMOFR21","base.fr"
"bank_fr_cmubfr21","CAISSE DE CREDIT MUNICIPAL DE BOULOGNE-SUR-MER","CMUBFR21","base.fr"
"bank_fr_cmdifr21","CAISSE DE CREDIT MUNICIPAL DE DIJON","CMDIFR21","base.fr"
"bank_fr_ccmefr21","CAISSE DE CREDIT MUNICIPAL DE LILLE","CCMEFR21","base.fr"
"bank_fr_ccmmfr21","CAISSE DE CREDIT MUNICIPAL DE LIMOGES","CCMMFR21","base.fr"
"bank_fr_ccmlfr21","CAISSE DE CREDIT MUNICIPAL DE LYON","CCMLFR21","base.fr"
"bank_fr_ccmrfr21","CAISSE DE CREDIT MUNICIPAL DE MARSEILLE","CCMRFR21","base.fr"
"bank_fr_ccmyfr21","CAISSE DE CREDIT MUNICIPAL DE NANCY","CCMYFR21","base.fr"
"bank_fr_ccmtfr21","CAISSE DE CREDIT MUNICIPAL DE NANTES","CCMTFR21","base.fr"
"bank_fr_ccuifr21","CAISSE DE CREDIT MUNICIPAL DE NICE","CCUIFR21","base.fr"
"bank_fr_ccmnfr21","CAISSE DE CREDIT MUNICIPAL DE NIMES","CCMNFR21","base.fr"
"bank_fr_ccmsfr21","CAISSE DE CREDIT MUNICIPAL DE REIMS","CCMSFR21","base.fr"
"bank_fr_cmrofr21","CAISSE DE CREDIT MUNICIPAL DE ROUBAIX","CMROFR21","base.fr"
"bank_fr_ccmufr21","CAISSE DE CREDIT MUNICIPAL DE ROUEN","CCMUFR21","base.fr"
"bank_fr_ccusfr21","CAISSE DE CREDIT MUNICIPAL DE STRASBOURG","CCUSFR21","base.fr"
"bank_fr_ccutfr21","CAISSE DE CREDIT MUNICIPAL DE TOULON","CCUTFR21","base.fr"
"bank_fr_ccuofr21","CAISSE DE CREDIT MUNICIPAL DE TOULOUSE","CCUOFR21","base.fr"
"bank_fr_ccmhfr21","CAISSE DE CREDIT MUNICIPAL DU HAVRE","CCMHFR21","base.fr"
"bank_fr_cadcfr21","CAISSE DE DEVELOPPEMENT DE LA CORSE SAS","CADCFR21","base.fr"
"bank_fr_cginfrp1","CAISSE DE GARANTIE DE L'IMMOBILIER F.N.A.I.M. SCM","CGINFRP1","base.fr"
"bank_fr_cagofrp1","CAISSE DE GARANTIE DU LOGEMENT","CAGOFRP1","base.fr"
"bank_fr_cgiffrp1","CAISSE DE GARANTIE IMMOB. FED. FSE BAT","CGIFFRP1","base.fr"
"bank_fr_cgmcfr21","CAISSE DE GARANTIE MUT. CIT ET CIT BAIL","CGMCFR21","base.fr"
"bank_fr_cmfcfrp1","CAISSE DE MUTUALISATION DES FINANCEMENTS","CMFCFRP1","base.fr"
"bank_fr_cirafrp1","CAISSE DE RETRAITE DES NOTAIRES","CIRAFRP1","base.fr"
"bank_fr_crpvfr21","CAISSE DE RETRAITE DU PERSONNEL NAVIGANT PROFESSIONNEL DE L'AERONAUTIQUE CIVILE","CRPVFR21","base.fr"
"bank_fr_cdcgfrpp","CAISSE DES DEPOTS ET CONSIGNATIONS","CDCGFRPP","base.fr"
"bank_fr_cfcufr21","CAISSE FEDERALE DE CREDIT MUTUEL DE NORMANDIE SOCIETE ANONYME","CFCUFR21","base.fr"
"bank_fr_cfcrfrp1","CAISSE FONCIERE DE CREDIT","CFCRFRP1","base.fr"
"bank_fr_cfdifrp1","CAISSE FRANCAISE DE DEVELOPPEMENT INDUSTRIEL","CFDIFRP1","base.fr"
"bank_fr_dxmafrpp","CAISSE FRANCAISE DE FINANCEMENT LOCAL","DXMAFRPP","base.fr"
"bank_fr_cgpcfrp1","CAISSE GENERALE DE PREVOYANCE DES CAISSES D'EPARGNES","CGPCFRP1","base.fr"
"bank_fr_cgrcfrp1","CAISSE GENERALE RETRAITE CAISSE D EPARGNE","CGRCFRP1","base.fr"
"bank_fr_cmgifrp1","CAISSE MUTUELLE DE GARANTIE DES INDUSTRIES MECANIQUES ET TRANSFORMATRICES DES METAUX","CMGIFRP1","base.fr"
"bank_fr_cntpfrp1","CAISSE NAT ENTREPRENEURS TRAVAUX PUBLICS","CNTPFRP1","base.fr"
"bank_fr_cnaffr21","CAISSE NATIONALE DES ALLOCATIONS FAMILIALES","CNAFFR21","base.fr"
"bank_fr_canufrp1","CAISSE NATIONALE DES AUTOROUTES","CANUFRP1","base.fr"
"bank_fr_canlfrp1","CAISSE NATIONALE DES TELECOMMUNICATIONS","CANLFRP1","base.fr"
"bank_fr_cnagfr21","CAISSE NATIONALE DU GENDARME","CNAGFR21","base.fr"
"bank_fr_cpgsfrp1","CAISSE PREVOYANCE AGENT SEC SOCIAL ASSIMILLES","CPGSFRP1","base.fr"
"bank_fr_cpcnfrp1","CAISSE PREVOYANCE CADRE ENTR AGRIC-ARRCO","CPCNFRP1","base.fr"
"bank_fr_ceeefr21","CAISSE REGIONALE CREDIT AGRICOLE MUTUEL CHAMPAGNE BOURGOGNE","CEEEFR21","base.fr"
"bank_fr_ceddfr21","CAISSE REGIONALE DE CREDIT AGRICOLE DE L'ANJOU ET DU MAINE","CEDDFR21","base.fr"
"bank_fr_cgclfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL ALPES PROVENCE","CGCLFR21","base.fr"
"bank_fr_cgcrfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL ATLANTIQUE VENDEE","CGCRFR21","base.fr"
"bank_fr_cgrifr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL BRIE PICARDIE","CGRIFR21","base.fr"
"bank_fr_cgecfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL CENTRE LOIRE","CGECFR21","base.fr"
"bank_fr_cgeifr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL CENTRE-EST","CGEIFR21","base.fr"
"bank_fr_ceegfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL CHARENTE PERIGORD","CEEGFR21","base.fr"
"bank_fr_ceigfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL COTE D'ARMOR","CEIGFR21","base.fr"
"bank_fr_cedgfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL D'AQUITAINE","CEDGFR21","base.fr"
"bank_fr_ceiefr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL D'ILLE ET VILAINE","CEIEFR21","base.fr"
"bank_fr_cgrufr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL DE CENTRE FRANCE","CGRUFR21","base.fr"
"bank_fr_cgcgfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL DE LA CORSE","CGCGFR21","base.fr"
"bank_fr_cgeefr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL DE NORMANDIE","CGEEFR21","base.fr"
"bank_fr_cgrrfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL DE NORMANDIE SEINE","CGRRFR21","base.fr"
"bank_fr_cgrgfrp1","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL DE PARIS ILE DE FR.","CGRGFRP1","base.fr"
"bank_fr_cgcufr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL DES SAVOIE","CGCUFR21","base.fr"
"bank_fr_cgrofr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL DU CENTRE OUEST","CGROFR21","base.fr"
"bank_fr_cedmfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL DU LANGUEDOC","CEDMFR21","base.fr"
"bank_fr_cedlfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL DU NORD EST","CEDLFR21","base.fr"
"bank_fr_cetgfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL FINISTERE","CETGFR21","base.fr"
"bank_fr_cgcmfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL FRANCHE COMTE","CGCMFR21","base.fr"
"bank_fr_ceetfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL LOIRE-HAUTE LOIRE","CEETFR21","base.fr"
"bank_fr_cgerfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL LORRAINE","CGERFR21","base.fr"
"bank_fr_cgelfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL MORBIHAN","CGELFR21","base.fr"
"bank_fr_cgrmfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL NORD DE FRANCE","CGRMFR21","base.fr"
"bank_fr_cgrtfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL PYRENEES GASCOGNE","CGRTFR21","base.fr"
"bank_fr_cgegfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL SUD MEDITERRANEE","CGEGFR21","base.fr"
"bank_fr_cgccfr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL SUD RHONE ALPES","CGCCFR21","base.fr"
"bank_fr_cedifr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL TOULOUSE 31","CEDIFR21","base.fr"
"bank_fr_cgcefr21","CAISSE REGIONALE DE CREDIT AGRICOLE MUTUEL VAL DE FRANCE","CGCEFR21","base.fr"
"bank_fr_cednfr21","CAISSE REGIONALE DE CREDIT AGRICOLE NORD MIDI PYRENEES","CEDNFR21","base.fr"
"bank_fr_cmmmfr21","CAISSE REGIONALE DE CREDIT MARITIME MUTUEL 'LA MEDITERRANEE'","CMMMFR21","base.fr"
"bank_fr_cmmafr21","CAISSE REGIONALE DE CREDIT MARITIME MUTUEL D'AQUITAINE","CMMAFR21","base.fr"
"bank_fr_cmarfr21","CAISSE REGIONALE DE CREDIT MARITIME MUTUEL DE LA REGION NORD","CMARFR21","base.fr"
"bank_fr_cfitfr21","CAISSE REGIONALE DU CREDIT MUTUEL MEDITERRANEEN SOCIETE COOPERATIVE DE CREDIT","CFITFR21","base.fr"
"bank_fr_crvsfrp1","CAISSE RETRAITE PREVOYANCE CLERCS ET NOTAIRE","CRVSFRP1","base.fr"
"bank_fr_caoofr21","CAISSE SOLIDAIRE SOCIETE ANONYME","CAOOFR21","base.fr"
"bank_fr_cgdifrpp","CAIXA GERAL DE DEPOSITOS","CGDIFRPP","base.fr"
"bank_fr_caxgfr21","CAIXA GESTION","CAXGFR21","base.fr"
"bank_fr_clfefrp1","CALAO FINANCE","CLFEFRP1","base.fr"
"bank_fr_cmcefrp1","CAM CEREALES","CMCEFRP1","base.fr"
"bank_fr_cmaifr22","CAMAIEU SA","CMAIFR22","base.fr"
"bank_fr_cmccfrp1","CAMCA","CMCCFRP1","base.fr"
"bank_fr_dxamfrpp","CANDRIAM FRANCE SA","DXAMFRPP","base.fr"
"bank_fr_cadyfr21","CANDY FRANCE SA","CADYFR21","base.fr"
"bank_fr_cafrfr21","CANON FINANCE FRANCE","CAFRFR21","base.fr"
"bank_fr_cfilfrp1","CANTOR FITZGERALD INTERNATIONAL","CFILFRP1","base.fr"
"bank_fr_capgfrpp","CAP GEMINI SERVICE SAS","CAPGFRPP","base.fr"
"bank_fr_cppmfr21","CAP MUTUELLE","CPPMFR21","base.fr"
"bank_fr_canwfr21","CAPITAL BANK PLC NEWS BANQUE","CANWFR21","base.fr"
"bank_fr_caxpfr21","CAPITAL EXPORT","CAXPFR21","base.fr"
"bank_fr_caftfrp1","CAPITAL FUND MANAGEMENT SA","CAFTFRP1","base.fr"
"bank_fr_caonfr21","CAPITAL ONE","CAONFR21","base.fr"
"bank_fr_csyifrp1","CAPITAL SYSTEME INVESTISSEMENT - C.S.I.","CSYIFRP1","base.fr"
"bank_fr_caiofrp1","CAPITOL","CAIOFRP1","base.fr"
"bank_fr_caaifr21","CAPITOLE FINANCE - TOFINSO","CAAIFR21","base.fr"
"bank_fr_cpcpfrp1","CAPMA-CAPMI","CPCPFRP1","base.fr"
"bank_fr_cacufr21","CAPRICORN CONSULTING","CACUFR21","base.fr"
"bank_fr_ccrcfr21","CARAC SRL","CCRCFR21","base.fr"
"bank_fr_carxfrp1","CARAX","CARXFRP1","base.fr"
"bank_fr_capyfrp1","CARCEPT PREVOYANCE","CAPYFRP1","base.fr"
"bank_fr_aarcfrp1","CARCO","AARCFRP1","base.fr"
"bank_fr_caiffrp1","CARDIF ASSET MANAGEMENT","CAIFFRP1","base.fr"
"bank_fr_cauvfr21","CARDIF ASSURANCE VIE SA","CAUVFR21","base.fr"
"bank_fr_cagdfrp1","CARDIF GESTION D'ACTIFS","CAGDFRP1","base.fr"
"bank_fr_clnffrp1","CARDIF LUX INTERNATIONAL FRANCE","CLNFFRP1","base.fr"
"bank_fr_cravfrp1","CARDIF RETRAITE ASSURANCE VIE","CRAVFRP1","base.fr"
"bank_fr_crrvfrp1","CARDIF SERVICES","CRRVFRP1","base.fr"
"bank_fr_cgfsfrp1","CARGILL FRANCE SAS","CGFSFRP1","base.fr"
"bank_fr_cissfr21","CARGILL INVESTOR SERVICES","CISSFR21","base.fr"
"bank_fr_crsofr21","CARLTON SELECTION","CRSOFR21","base.fr"
"bank_fr_cagvfrp1","CARMF REGIME A.S.V.","CAGVFRP1","base.fr"
"bank_fr_caeifrp1","CARMIGNAC GESTION SA","CAEIFRP1","base.fr"
"bank_fr_cppgfrp1","CARPILIG","CPPGFRP1","base.fr"
"bank_fr_caikfrp1","CARPIMKO","CAIKFRP1","base.fr"
"bank_fr_cauffr21","CARREFOUR","CAUFFR21","base.fr"
"bank_fr_soapfr22","CARREFOUR BANQUE","SOAPFR22","base.fr"
"bank_fr_ctttfrp1","CARTESIA","CTTTFRP1","base.fr"
"bank_fr_cssvfr21","CASINO SERVICES","CSSVFR21","base.fr"
"bank_fr_tcatfrpp","CAT COMPAGNIE D'AFFRETEMENT ET DE TRANSPORT","TCATFRPP","base.fr"
"bank_fr_cafffr21","CATERPILLAR FINANCE FRANCE SA","CAFFFR21","base.fr"
"bank_fr_ctopfrp1","CAUMUPRO TEOL.CAUT.MUT.NEGOC. OLEA PROTEA","CTOPFRP1","base.fr"
"bank_fr_caoafrp1","CAUTION GRAINOL","CAOAFRP1","base.fr"
"bank_fr_cmcmfrp1","CAUTION MUTUELLE DU CREDIT IMMOBILIER DE FRANCE","CMCMFRP1","base.fr"
"bank_fr_cavffr21","CAVA FINANCE","CAVFFR21","base.fr"
"bank_fr_cvaafr21","CAVAC","CVAAFR21","base.fr"
"bank_fr_cavgfr21","CAVAGESTION","CAVGFR21","base.fr"
"bank_fr_cveefrp1","CAVEC","CVEEFRP1","base.fr"
"bank_fr_cclvfrp1","CAVOM CAISSE ALLOCATION VIEILLESSE DES OFFICIERS MINISTERIELS","CCLVFRP1","base.fr"
"bank_fr_czasfrp1","CAZENOVE AND ASSOCIES","CZASFRP1","base.fr"
"bank_fr_cbsefr22","CBASE","CBSEFR22","base.fr"
"bank_fr_cbgsfrp1","CBT GESTION","CBGSFRP1","base.fr"
"bank_fr_ccclfrp1","CCF CHARTERHOUSE LEASING","CCCLFRP1","base.fr"
"bank_fr_ccfefrp1","CCM FDS DU REGIME CC PF","CCFEFRP1","base.fr"
"bank_fr_cchmfr21","CCMO CAISSE CHIRURGICALE MUTUELLE DE L OISE","CCHMFR21","base.fr"
"bank_fr_cccnfr21","CCMO CONSEIL SARL","CCCNFR21","base.fr"
"bank_fr_ugaafrp1","CCR ASSET MANAGEMENT","UGAAFRP1","base.fr"
"bank_fr_cccpfrp1","CCR CHEVRILLON PHILIPPE SA","CCCPFRP1","base.fr"
"bank_fr_cdprfrp1","CDC PROJETS","CDPRFRP1","base.fr"
"bank_fr_fbumfrp1","CDR FINANCE","FBUMFRP1","base.fr"
"bank_fr_eeacfrp1","CEA","EEACFRP1","base.fr"
"bank_fr_cerrfrp1","CEDRUS AM","CERRFRP1","base.fr"
"bank_fr_cegifrp1","CEGI","CEGIFRP1","base.fr"
"bank_fr_cgdgfr22","CEGID GROUP","CGDGFR22","base.fr"
"bank_fr_ceexfrp1","CENTRAL EXPANSION SA","CEEXFRP1","base.fr"
"bank_fr_cefafrp1","CENTRE FRANCAIS DU PATRIMOINE","CEFAFRP1","base.fr"
"bank_fr_cndsfrp1","CENTRE NATIONAL D'ETUDES SPACIALES","CNDSFRP1","base.fr"
"bank_fr_cetnfrp1","CERCLE ANTEIS","CETNFRP1","base.fr"
"bank_fr_eesafrp1","CEREALIS SAS","EESAFRP1","base.fr"
"bank_fr_cfaffr22","CFAO","CFAFFR22","base.fr"
"bank_fr_cfcgfrp1","CFD CAPITAL MANAGEMENT SA","CFCGFRP1","base.fr"
"bank_fr_cfpbfrp1","CFE PARIS BRANCH","CFPBFRP1","base.fr"
"bank_fr_cfgafr21","CFGI GALILEE","CFGAFR21","base.fr"
"bank_fr_cfrrfrp1","CFHL2014 - TO BE CREATED","CFRRFRP1","base.fr"
"bank_fr_cgkefrp1","CGM KEMPF","CGKEFRP1","base.fr"
"bank_fr_ccedfrp1","CHAMBRE DE COMMERCE ET D'INDUSTRIE DE PARIS","CCEDFRP1","base.fr"
"bank_fr_ccgafrp1","CHAMBRE DE COMPENSATION ET DE GARANTIE","CCGAFRP1","base.fr"
"bank_fr_chjlfr21","CHAMPEIL (JEAN-LOUIS)","CHJLFR21","base.fr"
"bank_fr_chnlfr22","CHANEL","CHNLFR22","base.fr"
"bank_fr_ctllfrp1","CHANTELLE SA","CTLLFRP1","base.fr"
"bank_fr_chfafr21","CHARBONNAGES DE FRANCE CDF ETABLISSEMENT PUBLIC NATIONAL","CHFAFR21","base.fr"
"bank_fr_chgefrp1","CHAUSSIER GESTION S.A.","CHGEFRP1","base.fr"
"bank_fr_pcbcfrpp","CHINA CONSTRUCTION BANK (EUROPE) S.A., PARIS BRANCH","PCBCFRPP","base.fr"
"bank_fr_cdanfrp1","CHOLET DUPONT ASSET MANAGEMENT","CDANFRP1","base.fr"
"bank_fr_chdufrpp","CHOLET DUPONT SA","CHDUFRPP","base.fr"
"bank_fr_dcoufrpp","CHRISTIAN DIOR COUTURE","DCOUFRPP","base.fr"
"bank_fr_diorfrp1","CHRISTIAN DIOR SA","DIORFRP1","base.fr"
"bank_fr_cloufrpp","CHRISTIAN LOUBOUTIN","CLOUFRPP","base.fr"
"bank_fr_bcnofr21","CI BTP CAISSE DU NORD OUEST","BCNOFR21","base.fr"
"bank_fr_cibrfrp1","CIB FINANCE","CIBRFRP1","base.fr"
"bank_fr_cicafrp1","CICOBAIL SOCIETE ANONYME","CICAFRP1","base.fr"
"bank_fr_euglfrp1","CIE EUROPEENNE DE GESTION ET PLACEMENT","EUGLFRP1","base.fr"
"bank_fr_inoafrp1","CIE INTERNAT COURTAGE MONETAIRE C I C M","INOAFRP1","base.fr"
"bank_fr_cierfrp1","CIF EUROMORTGAGE SA","CIERFRP1","base.fr"
"bank_fr_cilgfrp1","CILOGER","CILGFRP1","base.fr"
"bank_fr_cinefrp1","CINERGIE","CINEFRP1","base.fr"
"bank_fr_cipnfr21","CIP CONSEIL","CIPNFR21","base.fr"
"bank_fr_cipafr21","CIPA-CIV","CIPAFR21","base.fr"
"bank_fr_cppsfrp1","CIPS SARL","CPPSFRP1","base.fr"
"bank_fr_cioefrp1","CIRCIA FONDS DU REGIME CC PF","CIOEFRP1","base.fr"
"bank_fr_ciogfrp1","CIRNASE FONDS DU REGIME CC PF","CIOGFRP1","base.fr"
"bank_fr_cidrfrp1","CIRSIC FONDS DU REGIME CC PF","CIDRFRP1","base.fr"
"bank_fr_cgfffr21","CIT GROUP FINANCE (FRANCE) SNC","CGFFFR21","base.fr"
"bank_fr_ciudfrp1","CITCO FUND ADVISORS","CIUDFRP1","base.fr"
"bank_fr_citifrpp","CITIBANK EUROPE PLC FRANCE BRANCH","CITIFRPP","base.fr"
"bank_fr_sbilfrp1","CITIGROUP GLOBAL MARKETS LIMITED","SBILFRP1","base.fr"
"bank_fr_cigpfr21","CITY GESTION PRIVEE","CIGPFR21","base.fr"
"bank_fr_claafrp1","CLAAS FINANCIAL SERVICES SAS","CLAAFRP1","base.fr"
"bank_fr_clacfrp1","CLARESCO","CLACFRP1","base.fr"
"bank_fr_cltmfrp1","CLAYS ASSET MANAGEMENT","CLTMFRP1","base.fr"
"bank_fr_cllofrp1","CLF LOCABAIL","CLLOFRP1","base.fr"
"bank_fr_cltrfrp1","CLIC TRADE","CLTRFRP1","base.fr"
"bank_fr_clikfrp1","CLICKOPTIONS SOCIETE ANONYME","CLIKFRP1","base.fr"
"bank_fr_cbemfrp1","CLOSE BROTHERS EQUITY MARKETS S.A.","CBEMFRP1","base.fr"
"bank_fr_cmedfrpp","CLUB MEDITERRANEE","CMEDFRPP","base.fr"
"bank_fr_cisnfrp1","CM CIC ASSET MANAGEMENT","CISNFRP1","base.fr"
"bank_fr_fcoefrp1","CM FINANCE CONSEIL SARL","FCOEFRP1","base.fr"
"bank_fr_fnnsfrp1","CM FINANCES","FNNSFRP1","base.fr"
"bank_fr_cmgefrp1","CM-CIC GESTION","CMGEFRP1","base.fr"
"bank_fr_cmagfr22","CMA CGM","CMAGFR22","base.fr"
"bank_fr_cmfrfrp1","CMI FRANCE SA","CMFRFRP1","base.fr"
"bank_fr_cmpcfrp1","CMP BANQUE SA","CMPCFRP1","base.fr"
"bank_fr_cmvmfrp1","CMV MEDIFORCE SA","CMVMFRP1","base.fr"
"bank_fr_cncufrp1","CNH CAPITAL EUROPE SAS","CNCUFRP1","base.fr"
"bank_fr_cnfsfr21","CNH INDUSTRIAL FINANCIAL SERVICES SA","CNFSFR21","base.fr"
"bank_fr_cnpafrpp","CNP ASSURANCES","CNPAFRPP","base.fr"
"bank_fr_cofafr22","COFACE","COFAFR22","base.fr"
"bank_fr_cofefr2p","COFACREDIT","COFEFR2P","base.fr"
"bank_fr_coaifrp1","COFICA BAIL SA","COAIFRP1","base.fr"
"bank_fr_cooifrp1","COFICO GESTION SA","COOIFRP1","base.fr"
"bank_fr_cffifr21","COFIDIS","CFFIFR21","base.fr"
"bank_fr_cffgfr21","COFINGEST","CFFGFR21","base.fr"
"bank_fr_conofrp1","COFINOGA","CONOFRP1","base.fr"
"bank_fr_coipfrp1","COFIPLAN SA","COIPFRP1","base.fr"
"bank_fr_couofr21","COFIROUTE","COUOFR21","base.fr"
"bank_fr_cooffrp1","COFITEM-COFIMUR SA","COOFFRP1","base.fr"
"bank_fr_cggefrp1","COGEFI GESTION","CGGEFRP1","base.fr"
"bank_fr_cogrfr21","COGERA SA","COGRFR21","base.fr"
"bank_fr_coepfrp1","COGERAP SAS","COEPFRP1","base.fr"
"bank_fr_ccfmfrp1","COHEN AND COMPANY FINANCIAL LIMITED (FRANCE)","CCFMFRP1","base.fr"
"bank_fr_coeffrp1","COINSTAR MONEY TRANSFER","COEFFRP1","base.fr"
"bank_fr_cllafr21","COLAS SA","CLLAFR21","base.fr"
"bank_fr_collfr22","COLLECTE LOCALISATION SATELLITES","COLLFR22","base.fr"
"bank_fr_cotofrp1","COLLINS STEWART EUROPE LIMITED","COTOFRP1","base.fr"
"bank_fr_cmmgfrp1","COMGEST","CMMGFRP1","base.fr"
"bank_fr_coclfrp1","COMPAGNIE COMMERCIALE DE LOCATION - CCL SA","COCLFRP1","base.fr"
"bank_fr_gieofrpp","COMPAGNIE D'INVESTISSEMENTS INDUSTRIELS ET COMMERCIAUX - CIIC","GIEOFRPP","base.fr"
"bank_fr_cffofrpp","COMPAGNIE DE FINANCEMENT FONCIER","CFFOFRPP","base.fr"
"bank_fr_cogpfr21","COMPAGNIE DE GESTION ET DE PRETS","COGPFR21","base.fr"
"bank_fr_sgobfrpp","COMPAGNIE DE SAINT-GOBAIN","SGOBFRPP","base.fr"
"bank_fr_cdalfrpp","COMPAGNIE DES ALPES","CDALFRPP","base.fr"
"bank_fr_coeafrp1","COMPAGNIE EUROPEENNE DE BAIL","COEAFRP1","base.fr"
"bank_fr_aurafrp1","COMPAGNIE FINANCIERE AURA","AURAFRP1","base.fr"
"bank_fr_cfbpfrp1","COMPAGNIE FINANCIERE BNP PARIBAS","CFBPFRP1","base.fr"
"bank_fr_conufr21","COMPAGNIE FINANCIERE DE BOURBON SA","CONUFR21","base.fr"
"bank_fr_coaffrp1","COMPAGNIE FINANCIERE DE LA MACIF","COAFFRP1","base.fr"
"bank_fr_roulfr22","COMPAGNIE FINANCIERE ET DE PARTICIPATIONS ROULLIER","ROULFR22","base.fr"
"bank_fr_ecfifrp1","COMPAGNIE FINANCIERE EUROPEENNE SOCIETE A RESPONSABILITE LIMITEE","ECFIFRP1","base.fr"
"bank_fr_cmflfr21","COMPAGNIE FINANCIERE LAMARTINE","CMFLFR21","base.fr"
"bank_fr_cofpfrp1","COMPAGNIE FIRE DE PARIS","COFPFRP1","base.fr"
"bank_fr_coftfr21","COMPAGNIE FIRE DU LITTORAL","COFTFR21","base.fr"
"bank_fr_cofrfr21","COMPAGNIE FIRE POUR LA DISTRIBUTION","COFRFR21","base.fr"
"bank_fr_cfocfrp1","COMPAGNIE FONCIERE DE CREDIT SA","CFOCFRP1","base.fr"
"bank_fr_cfbbfrp1","COMPAGNIE FRANCAISE BAIL ET CIT-BAIL IMMO","CFBBFRP1","base.fr"
"bank_fr_cfetfr21","COMPAGNIE FRANCAISE D'ASSURANCE POUR LE COMMERCE EXTERIEUR SA","CFETFR21","base.fr"
"bank_fr_cficfrp1","COMPAGNIE FRANCAISE IMMOBILIERE POUR COMMERCE ET INDUSTRIE","CFICFRP1","base.fr"
"bank_fr_cogdfrp1","COMPAGNIE GENERALE D'AFFACTURAGE SA","COGDFRP1","base.fr"
"bank_fr_cgcpfrpl","COMPAGNIE GENERALE DE CREDIT AUX PARTICULIERS - CREDIPAR","CGCPFRPL","base.fr"
"bank_fr_cgcpfrp1","COMPAGNIE GENERALE DE CREDIT AUX PARTICULIERS-CREDIPAR SA","CGCPFRP1","base.fr"
"bank_fr_coggfrp1","COMPAGNIE GENERALE DE GARANTIE","COGGFRP1","base.fr"
"bank_fr_cglefr2m","COMPAGNIE GENERALE DE LOCATION D'EQUIPEMENTS","CGLEFR2M","base.fr"
"bank_fr_cglefr21","COMPAGNIE GENERALE DE LOCATION D'EQUIPEMENTS SA","CGLEFR21","base.fr"
"bank_fr_sciifr21","COMPAGNIE IMMOBILIERE RIVES DE LOIRE","SCIIFR21","base.fr"
"bank_fr_colcfrp1","COMPAGNIE LA LUCETTE","COLCFRP1","base.fr"
"bank_fr_cmfvfrp1","COMPAGNIE MEDICALE DE FINT VOIT ET MAT","CMFVFRP1","base.fr"
"bank_fr_pomnfr22","COMPAGNIE PLASTIC OMNIUM","POMNFR22","base.fr"
"bank_fr_colvfrp1","COMPAGNIE POUR LA LOCATION DE VEHICULES SOCIETE ANONYME","COLVFRP1","base.fr"
"bank_fr_cflcfrp1","COMPAGNIE POUR LE FINANCEMENT DES LOISIRS SA","CFLCFRP1","base.fr"
"bank_fr_cosffrp1","COMPAGNIE SUISSE ET FRANCAISE","COSFFRP1","base.fr"
"bank_fr_cpgffr21","COMPANGIE PARISIENNE DE GESTION FINANCIERE","CPGFFR21","base.fr"
"bank_fr_cfssfr21","COMPAQ FINANCIAL SERVICES SAS","CFSSFR21","base.fr"
"bank_fr_congfr21","COMPTOIR FINANCIER DE GARANTIE SA","CONGFR21","base.fr"
"bank_fr_cfcpfrp1","CONCES FRANC CONST EXPL TUN MONT BLANC","CFCPFRP1","base.fr"
"bank_fr_cfdtfrp1","CONFEDRATION FRANCAISE DEMOCRATIQUE DE TRAVAIL (CFDT)","CFDTFRP1","base.fr"
"bank_fr_cipefr21","CONGES INTEMPERIES BTP CAISSE BAS RHIN","CIPEFR21","base.fr"
"bank_fr_cipufr21","CONGES INTEMPERIES BTP DE L'OUEST","CIPUFR21","base.fr"
"bank_fr_cipffr21","CONGES INTEMPERIES BTP MASSIF CENTRAL","CIPFFR21","base.fr"
"bank_fr_cgfcfrp1","CONSEIL DE GESTION FINANCIERE COGEFI","CGFCFRP1","base.fr"
"bank_fr_cogffrp1","CONSEIL DE GESTION FINANCIERE SA","COGFFRP1","base.fr"
"bank_fr_cneufr21","CONSEIL DE L EUROPE","CNEUFR21","base.fr"
"bank_fr_cnoafrp1","CONSEIL NATIONAL DE L'ORDRE DES ARCHITECTES","CNOAFRP1","base.fr"
"bank_fr_copufr21","CONSEIL PLUS","COPUFR21","base.fr"
"bank_fr_cgmtfr21","CONSEILS EN GESTION ET MANAGEMENT POUR LE TRANSPORT ROUTIER SA","CGMTFR21","base.fr"
"bank_fr_coanfrp1","CONSERVATEUR FINANCE SA","COANFRP1","base.fr"
"bank_fr_cnsafrp1","CONSTANCE ASSOCIES S.A.S.","CNSAFRP1","base.fr"
"bank_fr_cgalfrpp","CONTOURGLOBAL MANAGEMENT FRANCE SAS","CGALFRPP","base.fr"
"bank_fr_covifrp1","CONVICTIONS AM","COVIFRP1","base.fr"
"bank_fr_scirfr21","COOPERATIVE IMMOBILIERE REGIONALE DE HAUTE NORMANDIE","SCIRFR21","base.fr"
"bank_fr_clmsfrp1","COOPERNEFF ALTERNATIVE MANAGERS SAS","CLMSFRP1","base.fr"
"bank_fr_cptrfr21","COPARTIS","CPTRFR21","base.fr"
"bank_fr_coeofr21","CORA-SERVICE TRESORERIE","COEOFR21","base.fr"
"bank_fr_ceivfrp1","CORREL INVEST","CEIVFRP1","base.fr"
"bank_fr_cogafr21","CORSE GARANTIE","COGAFR21","base.fr"
"bank_fr_bcrtfra1","CORTAL CONSORS FRANCE","BCRTFRA1","base.fr"
"bank_fr_cgepfrp1","COSMOS GESTION PRIVEE","CGEPFRP1","base.fr"
"bank_fr_cefpfrpp","COUNCIL OF EUROPE DEVELOPMENT BANK","CEFPFRPP","base.fr"
"bank_fr_cboufrp1","COURCOUX BOUVET","CBOUFRP1","base.fr"
"bank_fr_covafrp1","COVEA FINANCE SOCIETE PAR ACTIONS SIMPLIFIEE","COVAFRP1","base.fr"
"bank_fr_cvflfr21","COVEA FLEET SA","CVFLFR21","base.fr"
"bank_fr_cpbifrpp","CP OR DEVISES","CPBIFRPP","base.fr"
"bank_fr_cterfrp1","CPR - INTERMEDIATON","CTERFRP1","base.fr"
"bank_fr_cprmfrpp","CPR ASSET MANAGEMENT","CPRMFRPP","base.fr"
"bank_fr_schpfrp1","CPR COMPENSATION PARIS","SCHPFRP1","base.fr"
"bank_fr_cprgfrp1","CPR GESTION","CPRGFRP1","base.fr"
"bank_fr_cponfrp1","CPR ONLINE SA","CPONFRP1","base.fr"
"bank_fr_cprpfr22","CPRPSNCF","CPRPFR22","base.fr"
"bank_fr_crfnfrp1","CRAIGSTON FINANCE SAS","CRFNFRP1","base.fr"
"bank_fr_crlnfr21","CRAMA ALPES MEDITERRANEE","CRLNFR21","base.fr"
"bank_fr_crcvfr21","CRCAM DU CALVADOS","CRCVFR21","base.fr"
"bank_fr_craffrp1","CREALFI SAS","CRAFFRP1","base.fr"
"bank_fr_nactfr21","CREATION FRSE BEAUTE SISLEY","NACTFR21","base.fr"
"bank_fr_crtafr21","CREATIS SA","CRTAFR21","base.fr"
"bank_fr_crddfrp1","CREDIAL","CRDDFRP1","base.fr"
"bank_fr_cgavfr21","CREDIT AGRICOLE ALSACE VOSGES","CGAVFR21","base.fr"
"bank_fr_bsuifrpp","CREDIT AGRICOLE CIB","BSUIFRPP","base.fr"
"bank_fr_caodfrp1","CREDIT AGRICOLE COVERED BONDS","CAODFRP1","base.fr"
"bank_fr_calffrp1","CREDIT AGRICOLE LAZARD FINANCIAL PRODUCTS - PARIS BRANCH","CALFFRP1","base.fr"
"bank_fr_cdarfrp1","CREDIT DE L'ARCHE SA","CDARFRP1","base.fr"
"bank_fr_crsffrp1","CREDIT ET SERVICES FINANCIERS SA","CRSFFRP1","base.fr"
"bank_fr_crflfr21","CREDIT FINANCIER LILLOIS SA","CRFLFR21","base.fr"
"bank_fr_cfaofrp1","CREDIT FONCIER ASSURANCE COURTAGE","CFAOFRP1","base.fr"
"bank_fr_cffrfrpp","CREDIT FONCIER DE FRANCE","CFFRFRPP","base.fr"
"bank_fr_cfcsfr21","CREDIT FONCIER ET COMMUNAL D'ALSACE ET DE LORRAINE","CFCSFR21","base.fr"
"bank_fr_cfodfr21","CREDIT FONCIER ET COMMUNAL D'ALSACE ET DE LORRAINE BANQUE S.A.","CFODFR21","base.fr"
"bank_fr_cfclfr21","CREDIT FONCIER ET COMMUNAL D'ALSACE ET DE LORRAINE-BANQUE","CFCLFR21","base.fr"
"bank_fr_cifbfr21","CREDIT IMMOB FRANCE BOURGOGNE SUD ALLIER","CIFBFR21","base.fr"
"bank_fr_cirbfr21","CREDIT IMMOB FRANCE BRETAGNE ATLANTIQUE","CIRBFR21","base.fr"
"bank_fr_ciaofr21","CREDIT IMMOBILIER ALSACE LORRAINE FILIALE FINANCIERE","CIAOFR21","base.fr"
"bank_fr_cricfr21","CREDIT IMMOBILIER DE CHAMPAGNE","CRICFR21","base.fr"
"bank_fr_cifffr21","CREDIT IMMOBILIER DE FCIT FRANCE BOURGOGNE","CIFFFR21","base.fr"
"bank_fr_frhpfr21","CREDIT IMMOBILIER DE FRANCE - PAYS DE LA LOIRE SA","FRHPFR21","base.fr"
"bank_fr_cinpfrp1","CREDIT IMMOBILIER DE FRANCE DEVELOPPEMENT","CINPFRP1","base.fr"
"bank_fr_cirsfr21","CREDIT IMMOBILIER DE FRANCE EST","CIRSFR21","base.fr"
"bank_fr_ciflfr21","CREDIT IMMOBILIER DE FRANCE FINALOG","CIFLFR21","base.fr"
"bank_fr_ciiffrp1","CREDIT IMMOBILIER DE FRANCE ILE DE FRANCE SOCIETE ANONYME","CIIFFRP1","base.fr"
"bank_fr_cirdfr21","CREDIT IMMOBILIER DE FRANCE MEDITERRANEE","CIRDFR21","base.fr"
"bank_fr_cifdfr21","CREDIT IMMOBILIER DE FRANCE MIDI-PYRENEES FINANCIERE REGIONALE SA","CIFDFR21","base.fr"
"bank_fr_cirnfr21","CREDIT IMMOBILIER DE FRANCE NORD-PAS-DE-CALAIS","CIRNFR21","base.fr"
"bank_fr_cifrfr21","CREDIT IMMOBILIER DE FRANCE NORMANDIE SA","CIFRFR21","base.fr"
"bank_fr_cifefr21","CREDIT IMMOBILIER DE FRANCE-CENTRE OUEST SA","CIFEFR21","base.fr"
"bank_fr_cifvfr21","CREDIT IMMOBILIER DE FRANCE-VIVARAIS-SACI","CIFVFR21","base.fr"
"bank_fr_scmlfr21","CREDIT IMMOBILIER DES ALPES","SCMLFR21","base.fr"
"bank_fr_scitfr21","CREDIT IMMOBILIER DES ALPES SA","SCITFR21","base.fr"
"bank_fr_cifufr21","CREDIT IMMOBILIER FRANCE EURE ET DIEPPE","CIFUFR21","base.fr"
"bank_fr_cifyfr21","CREDIT IMMOBILIER FRANCE LYON SACI","CIFYFR21","base.fr"
"bank_fr_crigfrp1","CREDIT IMMOBILIER GENERAL","CRIGFRP1","base.fr"
"bank_fr_cihffr21","CREDIT IMMOBILIER HAUTS DE FRANCE","CIHFFR21","base.fr"
"bank_fr_sdcifr21","CREDIT IMMOBILIER MORBIHAN","SDCIFR21","base.fr"
"bank_fr_ciplfr21","CREDIT IMMOBILIER PAYS DE L'AIN","CIPLFR21","base.fr"
"bank_fr_crfffrp1","CREDIT LIFT SAS","CRFFFRP1","base.fr"
"bank_fr_crlgfrp1","CREDIT LOGEMENT SA","CRLGFRP1","base.fr"
"bank_fr_crlffrp1","CREDIT LYONNAIS FORFAITING","CRLFFRP1","base.fr"
"bank_fr_crlifrp1","CREDIT LYONNAIS IMMOBILIER","CRLIFRP1","base.fr"
"bank_fr_rousfrp1","CREDIT LYONNAIS ROUSE (FRANCE) SNC","ROUSFRP1","base.fr"
"bank_fr_clsrfr21","CREDIT LYONNAIS SECUR EUROPE SMALL CAPS","CLSRFR21","base.fr"
"bank_fr_cmmufrp1","CREDIT MARITIME MUTUEL-CMM","CMMUFRP1","base.fr"
"bank_fr_cmoifr21","CREDIT MODERNE OCEAN INDIEN","CMOIFR21","base.fr"
"bank_fr_crmpfrp1","CREDIT MUNICIPAL DE PARIS ETABLISSEMENT PUBLIC ADMINISTRATIF","CRMPFRP1","base.fr"
"bank_fr_cmutfr21","CREDIT MUTUEL","CMUTFR21","base.fr"
"bank_fr_cmcifr2a","CREDIT MUTUEL","CMCIFR2A","base.fr"
"bank_fr_cmcifrpb","CREDIT MUTUEL-CIC BANQUES-CM-CIC SECURITIES","CMCIFRPB","base.fr"
"bank_fr_cnavfrp1","CREDIT NAVAL","CNAVFRP1","base.fr"
"bank_fr_cresfrpp","CREDIT SUISSE (FRANCE), PARIS","CRESFRPP","base.fr"
"bank_fr_cresfrpz","CREDIT SUISSE AG, PARIS BRANCH","CRESFRPZ","base.fr"
"bank_fr_csfbfrp1","CREDIT SUISSE ASSET MANAGEMENT FRANCE SA","CSFBFRP1","base.fr"
"bank_fr_csatfrp1","CREDIT SUISSE ASSET MANAGEMENT GESTION","CSATFRP1","base.fr"
"bank_fr_csrsfrp1","CREDIT SUISSE FIRST BOSTON LTD","CSRSFRP1","base.fr"
"bank_fr_cseefrp1","CREDIT SUISSE SECURITIES (EUROPE) LIMITED","CSEEFRP1","base.fr"
"bank_fr_cunvfr21","CREDIT UNIVERSEL","CUNVFR21","base.fr"
"bank_fr_crfsfr21","CREFIDIS SA","CRFSFR21","base.fr"
"bank_fr_crrhfrp1","CRH - CAISSE DE REFINANCEMENT DE L'HABITAT SA","CRRHFRP1","base.fr"
"bank_fr_crirfrp1","CRICA REGIME","CRIRFRP1","base.fr"
"bank_fr_ctiefrpp","CRITEO SA","CTIEFRPP","base.fr"
"bank_fr_cgllfrp1","CSSE GARANTIE DU LOGEMENT LOCATIF SOCIAL","CGLLFRP1","base.fr"
"bank_fr_cndvfrp1","CSSE NAT D'ASSU VIEILLESSE DES PROF LIB (CNAVPL)","CNDVFRP1","base.fr"
"bank_fr_cypafrp1","CYPANGA","CYPAFRP1","base.fr"
"bank_fr_cfagfrp1","CYRIL FINANCE ASSET MANAGEMENT","CFAGFRP1","base.fr"
"bank_fr_cyrifrp1","CYRIL FINANCE-GESTION (C.F.G)","CYRIFRP1","base.fr"
"bank_fr_cyvbfr22","CYRILLUS VERTBAUDET GROUP","CYVBFR22","base.fr"
"bank_fr_cycofrp1","CYRUS CONSEIL SAS","CYCOFRP1","base.fr"
"bank_fr_dcndfrp1","D C N S SOCIETE ANONYME","DCNDFRP1","base.fr"
"bank_fr_dimofr21","D.I.M.O. GESTION","DIMOFR21","base.fr"
"bank_fr_dafefr21","DAF FINANCE ET SERVICES SA","DAFEFR21","base.fr"
"bank_fr_dafrfr21","DAIMLERCHRYSLER FINANCIAL SERVICES","DAFRFR21","base.fr"
"bank_fr_dscrfrp1","DAIWA SECURITIES SB CAPITAL MARKETS","DSCRFRP1","base.fr"
"bank_fr_dalpfr21","DALENYS PAYMENT","DALPFR21","base.fr"
"bank_fr_dafifrpp","DANONE S.A.","DAFIFRPP","base.fr"
"bank_fr_dacrfrp1","DARIUS CAPITAL PARTNERS","DACRFRP1","base.fr"
"bank_fr_daavfr21","DASSAULT AVIATION","DAAVFR21","base.fr"
"bank_fr_dssafr22","DASSAULT SYSTEMES","DSSAFR22","base.fr"
"bank_fr_dtamfrp1","DAY TRADE ASSET MANAGEMENT","DTAMFRP1","base.fr"
"bank_fr_dbavfrp1","DBENCH ALTERNATIVE INVESTMENTS","DBAVFRP1","base.fr"
"bank_fr_dbvtfr2m","DBV TECHNOLOGIES S.A.","DBVTFR2M","base.fr"
"bank_fr_dcnsfrpp","DCNS","DCNSFRPP","base.fr"
"bank_fr_lllsfrp1","DE LAGE LANDEN LEASING SAS","LLLSFRP1","base.fr"
"bank_fr_oxylfr22","DECATHLON SA","OXYLFR22","base.fr"
"bank_fr_decyfrp1","DECYBEN","DECYFRP1","base.fr"
"bank_fr_dectfrp1","DEFENSE CONSEIL INTERNATIONAL","DECTFRP1","base.fr"
"bank_fr_degofrp1","DEGROOF GESTION","DEGOFRP1","base.fr"
"bank_fr_defifrp1","DELAHAYE FINANCE SA","DEFIFRP1","base.fr"
"bank_fr_delefr21","DELORE SARL","DELEFR21","base.fr"
"bank_fr_deaefrp1","DELTA ALTERNATIVE MANAGEMENT","DEAEFRP1","base.fr"
"bank_fr_decafr21","DELTA CAPS","DECAFR21","base.fr"
"bank_fr_deatfrp1","DELUBAC ASSET MANAGEMENT SA","DEATFRP1","base.fr"
"bank_fr_dpfafrp1","DEPFA BANK EUROPE PLC","DPFAFRP1","base.fr"
"bank_fr_depafrp1","DEPFA BANK PUBLIC LIMITED COMPANY","DEPAFRP1","base.fr"
"bank_fr_derhfrpp","DERICHEBOURG SA","DERHFRPP","base.fr"
"bank_fr_detdfrpp","DESCARTES TRADING","DETDFRPP","base.fr"
"bank_fr_deutfra1","DEUTSCHE ASSET MANAGEMENT FRANCE","DEUTFRA1","base.fr"
"bank_fr_deutfrpp","DEUTSCHE BANK AG","DEUTFRPP","base.fr"
"bank_fr_deutfr21","DEUTSCHE BANK FRANCE S.N.C.","DEUTFR21","base.fr"
"bank_fr_dlfsfr21","DEUTSCHE LEASING FRANCE SAS","DLFSFR21","base.fr"
"bank_fr_dmgifrp1","DEUTSCHE MORGAN GRENFELL EQUITIES S.A.","DMGIFRP1","base.fr"
"bank_fr_dpbbfrp1","DEUTSCHE PFANDBRIEFBANK AG BRANCH PARIS","DPBBFRP1","base.fr"
"bank_fr_darvfr21","DEVELOPPEMENT D'APPLICATIONS SUR RESEAUX A VALEUR AJOUTEE SA","DARVFR21","base.fr"
"bank_fr_deasfrp1","DEXIA ASSURECO SA","DEASFRP1","base.fr"
"bank_fr_decmfrp1","DEXIA CLF IMMO SA","DECMFRP1","base.fr"
"bank_fr_derafrp1","DEXIA CLF REGIONS BAIL SA","DERAFRP1","base.fr"
"bank_fr_clfrfrcc","DEXIA CREDIT LOCAL","CLFRFRCC","base.fr"
"bank_fr_clfrfrpp","DEXIA CREDIT LOCAL","CLFRFRPP","base.fr"
"bank_fr_depefrp1","DEXIA EPARGNE PENSION SA","DEPEFRP1","base.fr"
"bank_fr_defofrp1","DEXIA FLOBAIL SA","DEFOFRP1","base.fr"
"bank_fr_desrfrp1","DEXIA SECURITIES FRANCE SA","DESRFRP1","base.fr"
"bank_fr_deyffrp1","DEXTER ELYSEE FINANCE","DEYFFRP1","base.fr"
"bank_fr_dfinfrp1","DG FINANCE SA","DFINFRP1","base.fr"
"bank_fr_diacfr21","DIAC SA","DIACFR21","base.fr"
"bank_fr_diclfr21","DIEBOLD COMPUTER LOCATION SA","DICLFR21","base.fr"
"bank_fr_difffrp1","DIFFUCO SA","DIFFFRP1","base.fr"
"bank_fr_dicffr21","DINERS CLUB FRANCE SA","DICFFR21","base.fr"
"bank_fr_dififrp1","DIRECT FINANCE","DIFIFRP1","base.fr"
"bank_fr_dirvfrp1","DIRECT VIE SRL","DIRVFRP1","base.fr"
"bank_fr_dissfr21","DISPONIS SOCIETE PAR ACTIONS SIMPLIFIEE","DISSFR21","base.fr"
"bank_fr_dncffrp1","DNCA FINANCE SA","DNCFFRP1","base.fr"
"bank_fr_dovifrp1","DOLCEA VIE","DOVIFRP1","base.fr"
"bank_fr_dolffr21","DOLFI FINANCE SAS","DOLFFR21","base.fr"
"bank_fr_dmmcfrp1","DOLFI MISSIKA MICHELLA","DMMCFRP1","base.fr"
"bank_fr_dofafrp1","DOM FINANCE","DOFAFRP1","base.fr"
"bank_fr_docrfrp1","DOME CLOSE BROTHERS","DOCRFRP1","base.fr"
"bank_fr_dommfrp1","DOMIMUR SAS","DOMMFRP1","base.fr"
"bank_fr_domffrp1","DOMOFINANCE SA","DOMFFRP1","base.fr"
"bank_fr_dljifrp1","DONALDSON LUFKIN AND JENRETTE INTERNATIONAL","DLJIFRP1","base.fr"
"bank_fr_dorffrp1","DORVAL FINANCE SA","DORFFRP1","base.fr"
"bank_fr_dpntfrp1","DPA INVEST","DPNTFRP1","base.fr"
"bank_fr_drgffrp1","DRESDNER BANK GESTIONS FRANCE SAS","DRGFFRP1","base.fr"
"bank_fr_drgpfrp1","DRESDNER GESTION PRIVEE","DRGPFRP1","base.fr"
"bank_fr_kbenfrp1","DRESDNER KLEINWORT SECURITIES FRANCE SA","KBENFRP1","base.fr"
"bank_fr_pafcfrp1","DU PASQUIER AND CIE (FRANCE) SA","PAFCFRP1","base.fr"
"bank_fr_dudefr21","DUBLY DENOYELLE ET CIE","DUDEFR21","base.fr"
"bank_fr_dublfr21","DUBLY-DOUILHET","DUBLFR21","base.fr"
"bank_fr_ducdfrp1","DUCATEL, DUVAL S.A.","DUCDFRP1","base.fr"
"bank_fr_dukefrp1","DUFOUR KERVERN","DUKEFRP1","base.fr"
"bank_fr_dumefrp1","DUMENIL ET ASSOCIES","DUMEFRP1","base.fr"
"bank_fr_dupofrp1","DUPONT DENANT","DUPOFRP1","base.fr"
"bank_fr_ddctfrp1","DUPONT-DENANT CONTREPARTIE","DDCTFRP1","base.fr"
"bank_fr_dfsgfrp1","DWS FINANZ-SERVICE GMBH","DFSGFRP1","base.fr"
"bank_fr_dwinfrp1","DWS INVESTMENTS FRANCE","DWINFRP1","base.fr"
"bank_fr_eciifrp1","E-CIE VIE","ECIIFRP1","base.fr"
"bank_fr_eccffr21","E-FINANCES","ECCFFR21","base.fr"
"bank_fr_easofrp1","EASYBOURSE","EASOFRP1","base.fr"
"bank_fr_ecocfrpp","EBI SA","ECOCFRPP","base.fr"
"bank_fr_eburfrp1","EBURY PARTNERS UK LIMITED","EBURFRP1","base.fr"
"bank_fr_ecinfrp1","ECOFI INVESTISSEMENTS SA","ECINFRP1","base.fr"
"bank_fr_ecoffrp1","ECOFI-FINANCE S.A.","ECOFFRP1","base.fr"
"bank_fr_ecugfrp1","ECUREUIL GESTION","ECUGFRP1","base.fr"
"bank_fr_ecvifrp1","ECUREUIL VIE","ECVIFRP1","base.fr"
"bank_fr_macdfrp1","ED AND F MAN COMMODITY ADVISERS LIMITED","MACDFRP1","base.fr"
"bank_fr_maiafrp1","ED ET F MAN INTERNATIONAL SA","MAIAFRP1","base.fr"
"bank_fr_edgefrp1","EDELWEISS GESTION SA","EDGEFRP1","base.fr"
"bank_fr_ederfr22","EDENRED SA","EDERFR22","base.fr"
"bank_fr_edfnfrpp","EDF ENERGIES NOUVELLES","EDFNFRPP","base.fr"
"bank_fr_cofifrcp","EDMOND DE ROTHSCHILD (FRANCE)","COFIFRCP","base.fr"
"bank_fr_cofifrpp","EDMOND DE ROTHSCHILD (FRANCE)","COFIFRPP","base.fr"
"bank_fr_lramfrp1","EDMOND DE ROTHSCHILD ASSET MANAGEMENT","LRAMFRP1","base.fr"
"bank_fr_rfsifrp1","EDMOND DE ROTHSCHILD FINANCIAL SERVICES INVESTMENT MANAGEMENT","RFSIFRP1","base.fr"
"bank_fr_lripfrp1","EDMOND DE ROTHSCHILD INVESTMENT PARTNER","LRIPFRP1","base.fr"
"bank_fr_lrmmfrp1","EDMOND DE ROTHSCHILD MULTI MANAGEMENT","LRMMFRP1","base.fr"
"bank_fr_edgifrp1","EDOUARD 7 GESTION PRIVEE","EDGIFRP1","base.fr"
"bank_fr_edslfrp1","EDRIM SOLUTIONS","EDSLFRP1","base.fr"
"bank_fr_eamffrp1","EFG ASSET MANAGEMENT FRANCE","EAMFFRP1","base.fr"
"bank_fr_efigfrp1","EFIGEST","EFIGFRP1","base.fr"
"bank_fr_efmmfrp1","EFIM","EFMMFRP1","base.fr"
"bank_fr_egaofrp1","EGAMO","EGAOFRP1","base.fr"
"bank_fr_eggefrp1","EGG SRL","EGGEFRP1","base.fr"
"bank_fr_egisfr22","EGIS S.A.","EGISFR22","base.fr"
"bank_fr_egfefr21","EGP FONDS ET GESTION","EGFEFR21","base.fr"
"bank_fr_eifsfrp1","EIM FRANCE SAS","EIFSFRP1","base.fr"
"bank_fr_elpnfrp1","ELAIA PARTNERS","ELPNFRP1","base.fr"
"bank_fr_edfgfrpp","ELECTRICITE DE FRANCE","EDFGFRPP","base.fr"
"bank_fr_elbafrpp","ELECTRO BANQUE","ELBAFRPP","base.fr"
"bank_fr_elfnfr21","ELECTROLUX FINANCEMENT S N C","ELFNFR21","base.fr"
"bank_fr_elpofr22","ELECTROPOLI SA","ELPOFR22","base.fr"
"bank_fr_elnvfrp1","ELIAS INVEST","ELNVFRP1","base.fr"
"bank_fr_eligfrp1","ELIGEST SA","ELIGFRP1","base.fr"
"bank_fr_eltrfrpp","ELIOR TRESORERIE","ELTRFRPP","base.fr"
"bank_fr_elisfr22","ELIS PANTIN","ELISFR22","base.fr"
"bank_fr_elagfrp1","ELLIPSIS ASSET MANAGEMENT","ELAGFRP1","base.fr"
"bank_fr_engifrpp","ENGIE","ENGIFRPP","base.fr"
"bank_fr_gszgfrpp","ENGIE","GSZGFRPP","base.fr"
"bank_fr_gdstfr21","ENGIE GLOBAL MARKETS","GDSTFR21","base.fr"
"bank_fr_ensbfrp1","ENSKILDA, SOCIETE DE BOURSE S.A.","ENSBFRP1","base.fr"
"bank_fr_encsfrp1","ENTENIAL CONSEIL","ENCSFRP1","base.fr"
"bank_fr_enfnfrp1","ENTHECA FINANCE","ENFNFRP1","base.fr"
"bank_fr_enctfr22","ENTREPOSE CONTRACTING","ENCTFR22","base.fr"
"bank_fr_vengfrp1","ENTREPRENEUR VENTURE GESTION SA","VENGFRP1","base.fr"
"bank_fr_eglgfr21","ENTREPRISE GLE LEON GROSSE","EGLGFR21","base.fr"
"bank_fr_emihfrp1","ENTREPRISE MINIERE ET CHIMIQUE","EMIHFRP1","base.fr"
"bank_fr_eofifr21","EOLE FINANCE SA","EOFIFR21","base.fr"
"bank_fr_epcmfrp1","EPARGNE CREDIT DES MILITAIRES (E.C.M)","EPCMFRP1","base.fr"
"bank_fr_eqcrfrp1","EQUALIS CAPITAL FRANCE","EQCRFRP1","base.fr"
"bank_fr_equgfrp1","EQUIGEST SA","EQUGFRP1","base.fr"
"bank_fr_epesfrp1","EQUISTONE PARTNERS EUROPE SAS","EPESFRP1","base.fr"
"bank_fr_eqiifrp1","EQUITIS","EQIIFRP1","base.fr"
"bank_fr_equufrp1","EQUUS","EQUUFRP1","base.fr"
"bank_fr_eramfrp1","ERAMET","ERAMFRP1","base.fr"
"bank_fr_eramfrpa","ERAMET","ERAMFRPA","base.fr"
"bank_fr_ergtfrp1","ERASMUS GESTION","ERGTFRP1","base.fr"
"bank_fr_eresfrp1","ERES","ERESFRP1","base.fr"
"bank_fr_erisfrp1","ERISA","ERISFRP1","base.fr"
"bank_fr_esaafr21","ESCA","ESAAFR21","base.fr"
"bank_fr_esfgfr21","ESFIN GESTION GROUPEMENT D'INTERET ECONOMIQUE","ESFGFR21","base.fr"
"bank_fr_espafr21","ESFIN PARTICIPATIONS SAS","ESPAFR21","base.fr"
"bank_fr_espifr21","ESPACE PRODUCTION INTERNATIONAL","ESPIFR21","base.fr"
"bank_fr_esitfr2p","ESSILOR INTERNATIONAL","ESITFR2P","base.fr"
"bank_fr_esftfr21","ESTER FINANCE TITRISATION SA","ESFTFR21","base.fr"
"bank_fr_efcgfr21","ETABLISSEMENT FINANCIER CLAUDE GUILLOT S.A.","EFCGFR21","base.fr"
"bank_fr_etfvfr21","ETABLISSEMENTS FAUVET-GIREL","ETFVFR21","base.fr"
"bank_fr_etmpfrp1","ETABLISSEMENTS MAUREL ET PROM","ETMPFRP1","base.fr"
"bank_fr_etpffr21","ETABLISSEMENTS PEUGEOT FRERES","ETPFFR21","base.fr"
"bank_fr_etgefrp1","ETHIEA GESTION","ETGEFRP1","base.fr"
"bank_fr_etgsfrp1","ETOILE GESTION SNC","ETGSFRP1","base.fr"
"bank_fr_souffr22","ETS J SOUFFLET","SOUFFR22","base.fr"
"bank_fr_eukrfrp1","EUKRATOS","EUKRFRP1","base.fr"
"bank_fr_ehscfr2p","EULER HERMES SFAC CREDIT","EHSCFR2P","base.fr"
"bank_fr_ehscfrp1","EULER HERMES SFAC CREDIT SAS","EHSCFRP1","base.fr"
"bank_fr_euscfrp1","EULER SFAC CREDIT","EUSCFRP1","base.fr"
"bank_fr_eulcfrp1","EULIA CAUTION","EULCFRP1","base.fr"
"bank_fr_eufefrp1","EURASIA FINANCE SA","EUFEFRP1","base.fr"
"bank_fr_eurzfrp1","EURAZEO","EURZFRP1","base.fr"
"bank_fr_euprfrp1","EURAZEO PARTNERS SAS","EUPRFRP1","base.fr"
"bank_fr_eueffrp1","EURO EMETTEURS FINANCE","EUEFFRP1","base.fr"
"bank_fr_emcsfr21","EURO MID CAPS SECURITIES","EMCSFR21","base.fr"
"bank_fr_eusffrp1","EURO SALES FINANCE SA","EUSFFRP1","base.fr"
"bank_fr_eutefrp1","EURO TRESORERIE S.A.","EUTEFRP1","base.fr"
"bank_fr_eurefrp1","EUROCENTREST","EUREFRP1","base.fr"
"bank_fr_sicvfrpp","EUROCLEAR FRANCE","SICVFRPP","base.fr"
"bank_fr_esesfr21","EUROCLEAR FRANCE ACCOUNT PAYABLES","ESESFR21","base.fr"
"bank_fr_esesfrpp","EUROCLEAR FRANCE SA","ESESFRPP","base.fr"
"bank_fr_euocfrp1","EUROCORPORATE SA","EUOCFRP1","base.fr"
"bank_fr_eurdfrp1","EURODIF","EURDFRP1","base.fr"
"bank_fr_euiifrp1","EUROLAND FINANCE SA","EUIIFRP1","base.fr"
"bank_fr_eairfrp1","EUROMAF ASSURANCE DES INGENIEURS ET ARCHITECTES EUROPEENS SA","EAIRFRP1","base.fr"
"bank_fr_xmonfrp1","EURONEXT PARIS - MONEP","XMONFRP1","base.fr"
"bank_fr_xparfrpp","EURONEXT PARIS S.A.","XPARFRPP","base.fr"
"bank_fr_euarfr21","EUROP ASSISTANCE FRANCE SA","EUARFR21","base.fr"
"bank_fr_euahfrp1","EUROP ASSISTANCE HOLDING","EUAHFRP1","base.fr"
"bank_fr_eraafrp1","EUROPANEL RESEARCH AND ALTERNATIVE ASSET MANAGEMENT-ERAAM SA","ERAAFRP1","base.fr"
"bank_fr_eufafrp1","EUROPAY FRANCE SAS","EUFAFRP1","base.fr"
"bank_fr_ecarfr22","EUROPCAR HOLDING","ECARFR22","base.fr"
"bank_fr_arabfrpp","EUROPE ARAB BANK PLC","ARABFRPP","base.fr"
"bank_fr_eucpfrp1","EUROPE COMPANY LTD, THE","EUCPFRP1","base.fr"
"bank_fr_eegffrp1","EUROPE EGIDE FINANCE","EEGFFRP1","base.fr"
"bank_fr_eufdfrp1","EUROPE FINANCE ET INDUSTRIE SA","EUFDFRP1","base.fr"
"bank_fr_ecfsfrp1","EUROPEAN CAPITAL FINANCIAL SERVICES LIMITED","ECFSFRP1","base.fr"
"bank_fr_eueqfrp1","EUROPEAN EQUITIES","EUEQFRP1","base.fr"
"bank_fr_efalfrpp","EUROPEAN FUND ADMINISTRATION FRANCE S.A.S","EFALFRPP","base.fr"
"bank_fr_esazfrpp","EUROPEAN SPACE AGENCY","ESAZFRPP","base.fr"
"bank_fr_eucufrp1","EUROPEENNE DE CAUTIONNEMENT SA","EUCUFRP1","base.fr"
"bank_fr_euosfrp1","EUROSIC SA","EUOSFRP1","base.fr"
"bank_fr_eucsfr21","EUROSUD CONSEIL","EUCSFR21","base.fr"
"bank_fr_euitfrp1","EUROTRADIA INTERNATIONAL","EUITFRP1","base.fr"
"bank_fr_evfifrp1","EVEN FINANCE","EVFIFRP1","base.fr"
"bank_fr_exagfrp1","EXANE ASSET MANAGEMENT","EXAGFRP1","base.fr"
"bank_fr_exadfrpp","EXANE DERIVATIVES SNC","EXADFRPP","base.fr"
"bank_fr_exfnfrp1","EXANE FINANCE SA","EXFNFRP1","base.fr"
"bank_fr_exaofrp1","EXANE OPTIONS","EXAOFRP1","base.fr"
"bank_fr_exanfrpp","EXANE S.A.","EXANFRPP","base.fr"
"bank_fr_essmfrp1","EXANE STRUCTURED ASSET MANAGEMENT","ESSMFRP1","base.fr"
"bank_fr_exwofrpp","EXCLUSIVE NETWORKS SAS","EXWOFRPP","base.fr"
"bank_fr_expafrp1","EXCLUSIVE PARTNERS","EXPAFRP1","base.fr"
"bank_fr_exoefrp1","EXOE SAS","EXOEFRP1","base.fr"
"bank_fr_esdrfr21","EXPANSO-LA SOCIETE POUR LE DEVELOPPEMENT REGIONAL SA","ESDRFR21","base.fr"
"bank_fr_expgfrpp","EXPERIMENTAL GROUP SAS","EXPGFRPP","base.fr"
"bank_fr_exfcfr21","EXPERT ET FINANCE","EXFCFR21","base.fr"
"bank_fr_gailfrp1","EZYNESS","GAILFRP1","base.fr"
"bank_fr_fscmfr21","F.D.I. SOCIETE AN CREDIT IMMOBILIER","FSCMFR21","base.fr"
"bank_fr_cifsfr21","F.D.I. SOCIETE ANONYME COOPERATIVE D'INTERET COLLECTIF POUR L'ACCESSION A LA PROPRIETE","CIFSFR21","base.fr"
"bank_fr_facefr21","FACET SA","FACEFR21","base.fr"
"bank_fr_facffrpp","FACTOFRANCE","FACFFRPP","base.fr"
"bank_fr_fcthfrp1","FACTOFRANCE HELLER","FCTHFRP1","base.fr"
"bank_fr_faaafrp1","FAIRVIEW ASSET MANAGEMENT","FAAAFRP1","base.fr"
"bank_fr_leytfr22","FAIVELEY TRANSPORT SA","LEYTFR22","base.fr"
"bank_fr_faapfr21","FASTEA CAPITAL","FAAPFR21","base.fr"
"bank_fr_famafrp1","FAUCHIER - MAGNAN, DURANT DES AULNOIS SA","FAMAFRP1","base.fr"
"bank_fr_faurfrpp","FAURECIA","FAURFRPP","base.fr"
"bank_fr_fbnifrpp","FBN BANK UK LTD","FBNIFRPP","base.fr"
"bank_fr_fraefr21","FC FRANCE SA","FRAEFR21","base.fr"
"bank_fr_ucfdfrp1","FC UC-FDUC","UCFDFRP1","base.fr"
"bank_fr_ucgufrp1","FC UC-GUARDIAN","UCGUFRP1","base.fr"
"bank_fr_fcccfrp1","FCC 130","FCCCFRP1","base.fr"
"bank_fr_fceffr21","FCE BANK PLC","FCEFFR21","base.fr"
"bank_fr_fplrfr21","FCE BANK PLC","FPLRFR21","base.fr"
"bank_fr_faulfrp1","FCT ALLIANCE AUTO LOANS FRANCE V1","FAULFRP1","base.fr"
"bank_fr_faaofrp1","FCT ALLIANCE AUTO LOANS GERMANY 2013","FAAOFRP1","base.fr"
"bank_fr_fcadfrp1","FCT ALLIANCE DFP 2013","FCADFRP1","base.fr"
"bank_fr_fbhofrp1","FCT BPCE HOME LOANS","FBHOFRP1","base.fr"
"bank_fr_fcbffrp1","FCT BUMPER FRANCE","FCBFFRP1","base.fr"
"bank_fr_fccffrp1","FCT CFFHL2014","FCCFFRP1","base.fr"
"bank_fr_fccafrp1","FCT CIF ASSETS 2001","FCCAFRP1","base.fr"
"bank_fr_febefrp1","FCT EVERGREEN HL1  (BORROWER HEDGING TRANSACTION)","FEBEFRP1","base.fr"
"bank_fr_feihfrp1","FCT EVERGREEN HL1 (ISSUER HEDGING TRANSACTION)","FEIHFRP1","base.fr"
"bank_fr_fcfrfrp1","FCT F-CARAT 2010-1","FCFRFRP1","base.fr"
"bank_fr_fcfifrp1","FCT FI FINANCING","FCFIFRP1","base.fr"
"bank_fr_fcgpfrp1","FCT GINGKO PRIVATE 2012","FCGPFRP1","base.fr"
"bank_fr_fgclfrp1","FCT GINKGO CONSUMER LOANS 2013-1","FGCLFRP1","base.fr"
"bank_fr_fgplfrp1","FCT GINKGO PERSONAL LOANS 2013-1","FGPLFRP1","base.fr"
"bank_fr_fgsifrp1","FCT GINKGO SALES FINANCE 2011-1","FGSIFRP1","base.fr"
"bank_fr_fgsffrp1","FCT GINKGO SALES FINANCE 2012-1","FGSFFRP1","base.fr"
"bank_fr_fgsnfrp1","FCT GINKGO SALES FINANCE 2013-1","FGSNFRP1","base.fr"
"bank_fr_fcilfrp1","FCT INFINITY 2006-1 CLASSICO","FCILFRP1","base.fr"
"bank_fr_fcisfrp1","FCT INFINITY 2007-1 SOPRANO","FCISFRP1","base.fr"
"bank_fr_fmcafrp1","FCT MASTER CREDIT CARD PASS","FMCAFRP1","base.fr"
"bank_fr_fcnafrp1","FCT NACREA","FCNAFRP1","base.fr"
"bank_fr_fcoyfrp1","FCT ONEYCORD","FCOYFRP1","base.fr"
"bank_fr_fcptfrp1","FCT PARTIMMO 05 2003","FCPTFRP1","base.fr"
"bank_fr_fcprfrp1","FCT PARTIMMO 11 2003","FCPRFRP1","base.fr"
"bank_fr_fcppfrp1","FCT PROUDEED PROPERTIES 2005","FCPPFRP1","base.fr"
"bank_fr_frbtfrp1","FCT RED  AND BLACK AUTO LOANS","FRBTFRP1","base.fr"
"bank_fr_fcsufrp1","FCT SURF","FCSUFRP1","base.fr"
"bank_fr_fceifrp1","FCT TE EURO (TITAN 2006-3)","FCEIFRP1","base.fr"
"bank_fr_fcetfrp1","FCT TE EURO (TITAN 2007-1)","FCETFRP1","base.fr"
"bank_fr_fcvefrp1","FCT VULCAN (ELOC 28)","FCVEFRP1","base.fr"
"bank_fr_fczefrp1","FCT ZEBRE 2006-1","FCZEFRP1","base.fr"
"bank_fr_fczofrp1","FCT ZEBRE ONE","FCZOFRP1","base.fr"
"bank_fr_fcztfrp1","FCT ZEBRE TWO","FCZTFRP1","base.fr"
"bank_fr_fczsfr21","FCT ZEUS (OSIRIS)","FCZSFR21","base.fr"
"bank_fr_fdpffrp1","FDP","FDPFFRP1","base.fr"
"bank_fr_fefifr21","FEDERAL FINANCE BANQUE SA","FEFIFR21","base.fr"
"bank_fr_fegefr21","FEDERAL GESTION","FEGEFR21","base.fr"
"bank_fr_fefbfrp1","FEDERATION FRANCAISE DU BATIMENT","FEFBFRP1","base.fr"
"bank_fr_fndmfrp1","FEDERATION NATIONALE DE LA MUTUALITE FRANCAISE","FNDMFRP1","base.fr"
"bank_fr_fepsfrp1","FEDERIS EPARGNE SALARIALE SA","FEPSFRP1","base.fr"
"bank_fr_fegafrp1","FEDERIS GESTION D'ACTIFS SA","FEGAFRP1","base.fr"
"bank_fr_feitfrp1","FERRI INTERMEDIATION","FEITFRP1","base.fr"
"bank_fr_ferifrp1","FERRIGESTION SA","FERIFRP1","base.fr"
"bank_fr_ffpffrp1","FFP","FFPFFRP1","base.fr"
"bank_fr_ficffr21","FIAT CREDIT FRANCE","FICFFR21","base.fr"
"bank_fr_filufr21","FIAT LEASE AUTO","FILUFR21","base.fr"
"bank_fr_filpfrp1","FIDEAS CAPITAL","FILPFRP1","base.fr"
"bank_fr_fiemfrp1","FIDEM SOCIETE ANONYME","FIEMFRP1","base.fr"
"bank_fr_fidgfrp1","FIDES GESTION SARL","FIDGFRP1","base.fr"
"bank_fr_fwabfrp1","FIDEURAM WARGNY ACTIVE BROKER","FWABFRP1","base.fr"
"bank_fr_fdivfrp1","FIDIAM INVEST","FDIVFRP1","base.fr"
"bank_fr_fdinfrp1","FIDINVEST","FDINFRP1","base.fr"
"bank_fr_uffifrpp","FIDUCIAL GERANCE","UFFIFRPP","base.fr"
"bank_fr_flmffr21","FILIA-MAIF SA","FLMFFR21","base.fr"
"bank_fr_fmmlfrp1","FIMALAC","FMMLFRP1","base.fr"
"bank_fr_fippfrp1","FIMIPAR SA","FIPPFRP1","base.fr"
"bank_fr_fidafrp1","FINADOU-FINANCIERE DE L'ADOU SA","FIDAFRP1","base.fr"
"bank_fr_fialfr21","FINALION","FIALFR21","base.fr"
"bank_fr_fnnlfrp1","FINALTIS","FNNLFRP1","base.fr"
"bank_fr_ficifrp1","FINAMA CREDIT SOCIETE ANONYME","FICIFRP1","base.fr"
"bank_fr_fimufr21","FINAMUR","FIMUFR21","base.fr"
"bank_fr_figtfrp1","FINANCE ET GESTION SA","FIGTFRP1","base.fr"
"bank_fr_fncnfrp1","FINANCE FI SARL","FNCNFRP1","base.fr"
"bank_fr_fiirfrp1","FINANCE INTERMEDIATION","FIIRFRP1","base.fr"
"bank_fr_fncafrp1","FINANCE SA","FNCAFRP1","base.fr"
"bank_fr_figvfr21","FINANCE SA GESTION PRIVEE","FIGVFR21","base.fr"
"bank_fr_fiomfrp1","FINANCECOM AM","FIOMFRP1","base.fr"
"bank_fr_fihafrp1","FINANCIAL CHAMPLAIN","FIHAFRP1","base.fr"
"bank_fr_fagafrp1","FINANCIERE AGACHE","FAGAFRP1","base.fr"
"bank_fr_ficvfrp1","FINANCIERE ARBEVEL SOCIETE ANONYME","FICVFRP1","base.fr"
"bank_fr_fitlfrp1","FINANCIERE ATLAS S.A.","FITLFRP1","base.fr"
"bank_fr_fiaxfrp1","FINANCIERE AUXO","FIAXFRP1","base.fr"
"bank_fr_fgbbfr2m","FINANCIERE BOURBON","FGBBFR2M","base.fr"
"bank_fr_fictfr21","FINANCIERE CENTRE-LOIRE","FICTFR21","base.fr"
"bank_fr_fiuzfrp1","FINANCIERE D'UZES","FIUZFRP1","base.fr"
"bank_fr_fchpfrp1","FINANCIERE DE CHAMPLAIN SAS","FCHPFRP1","base.fr"
"bank_fr_fcipfr21","FINANCIERE DE CREDIT IMMOBILIER DE PICARDIE-CHAMPAGNE-ARDENNE SA","FCIPFR21","base.fr"
"bank_fr_fgprfrp1","FINANCIERE DE GESTION PRIVEE","FGPRFRP1","base.fr"
"bank_fr_filrfr21","FINANCIERE DE L'ARC","FILRFR21","base.fr"
"bank_fr_fiehfrp1","FINANCIERE DE L'ECHIQUIER SA","FIEHFRP1","base.fr"
"bank_fr_fislfr21","FINANCIERE DE L'IMMOBILIER SUD ATLANTIQUE SA","FISLFR21","base.fr"
"bank_fr_filxfrp1","FINANCIERE DE L'OXER SAS","FILXFRP1","base.fr"
"bank_fr_fncifrp1","FINANCIERE DE LA CITE SAS","FNCIFRP1","base.fr"
"bank_fr_filyfr21","FINANCIERE DE LYON, LA","FILYFR21","base.fr"
"bank_fr_fpelfr21","FINANCIERE DES PAIEMENTS ELECTRONIQUES","FPELFR21","base.fr"
"bank_fr_fcmufrp1","FINANCIERE DU CREDIT MUTUEL","FCMUFRP1","base.fr"
"bank_fr_fmshfrp1","FINANCIERE DU MARCHE SAINT-HONORE SA","FMSHFRP1","base.fr"
"bank_fr_fiexfrp1","FINANCIERE EXPERT","FIEXFRP1","base.fr"
"bank_fr_ffnefrp1","FINANCIERE FRANCO NEERLANDAISE","FFNEFRP1","base.fr"
"bank_fr_fgedfrp1","FINANCIERE GEDEL SRL","FGEDFRP1","base.fr"
"bank_fr_fhbbfrp1","FINANCIERE HOCHE BAINS LES BAINS","FHBBFRP1","base.fr"
"bank_fr_fiidfr21","FINANCIERE IMMOBILIERE CREDIT AGRICOLE CIB","FIIDFR21","base.fr"
"bank_fr_ficlfrp1","FINANCIERE INTERREGION CREDIT IMMOBILIER","FICLFRP1","base.fr"
"bank_fr_fklefrp1","FINANCIERE KLEBER","FKLEFRP1","base.fr"
"bank_fr_fidffrp1","FINANCIERE LA DEFENSE","FIDFFRP1","base.fr"
"bank_fr_fimhfrp1","FINANCIERE MEESCHAERT SA","FIMHFRP1","base.fr"
"bank_fr_fnnofrp1","FINANCIERE OCEOR SA","FNNOFRP1","base.fr"
"bank_fr_firafrp1","FINANCIERE RAPHAEL","FIRAFRP1","base.fr"
"bank_fr_frhbfr21","FINANCIERE REG. HAB BOURGOGNE-F.C. ALLIER","FRHBFR21","base.fr"
"bank_fr_frsmfr21","FINANCIERE REGION SUD MASSIF CENTRAL","FRSMFR21","base.fr"
"bank_fr_frcofr21","FINANCIERE REGIONALE CREDITS IMMOBILIERS EST","FRCOFR21","base.fr"
"bank_fr_frcmfr21","FINANCIERE REGIONALE DE CREDIT IMMOBILIER DU NORD PAS-DE-CALAIS SA","FRCMFR21","base.fr"
"bank_fr_frhafr21","FINANCIERE REGIONALE POUR HABITAT ALDA","FRHAFR21","base.fr"
"bank_fr_sicxfrp1","FINANCIERE SICOMAX SOCIETE ANONYME","SICXFRP1","base.fr"
"bank_fr_fitpfrp1","FINANCIERE TIEPOLO","FITPFRP1","base.fr"
"bank_fr_fieyfrp1","FINANCIERE VAN EYCK SAS","FIEYFRP1","base.fr"
"bank_fr_finffr21","FINANCO SA","FINFFR21","base.fr"
"bank_fr_fnnvfrp1","FINAVEO","FNNVFRP1","base.fr"
"bank_fr_fingfrp1","FINERGIE","FINGFRP1","base.fr"
"bank_fr_fifffrp1","FINIFAC","FIFFFRP1","base.fr"
"bank_fr_fniffr21","FINIFAC SAS","FNIFFR21","base.fr"
"bank_fr_finjfrp1","FININFO SA","FINJFRP1","base.fr"
"bank_fr_famufrp1","FININFOR ET ASSOCIES MULTIGESTION","FAMUFRP1","base.fr"
"bank_fr_fiogfrp1","FINOGEST SA","FIOGFRP1","base.fr"
"bank_fr_fipbfrp1","FIP BOURSE S.A.","FIPBFRP1","base.fr"
"bank_fr_nbadfrpp","FIRST ABU DHABI BANK PJSC PARIS","NBADFRPP","base.fr"
"bank_fr_fftafrp1","FISCHER FRANCIS TREES AND WATTS","FFTAFRP1","base.fr"
"bank_fr_fripfrp1","FITCH FRANCE SA","FRIPFRP1","base.fr"
"bank_fr_fvvafrp1","FIVAL SA","FVVAFRP1","base.fr"
"bank_fr_auoofr21","FL AUTO","AUOOFR21","base.fr"
"bank_fr_flfifrp1","FLEMING FINANCE","FLFIFRP1","base.fr"
"bank_fr_flinfrp1","FLINVEST SAS","FLINFRP1","base.fr"
"bank_fr_fdvffr22","FLORIMOND DESPREZ VEUVE ET FILS","FDVFFR22","base.fr"
"bank_fr_flagfrp1","FLORNOY ET ASSOCIES GESTION","FLAGFRP1","base.fr"
"bank_fr_fonafrp1","FONCARIS SOCIETE ANONYME","FONAFRP1","base.fr"
"bank_fr_fassfrp1","FONCIER ASSURANCE","FASSFRP1","base.fr"
"bank_fr_fonefrp1","FONCIER-BAIL","FONEFRP1","base.fr"
"bank_fr_fopifrp1","FONCIERE DE PARIS","FOPIFRP1","base.fr"
"bank_fr_fdrefr22","FONCIERES DES REGIONS","FDREFR22","base.fr"
"bank_fr_fobefrp1","FONDATION BETTENCOURT","FOBEFRP1","base.fr"
"bank_fr_fonffrp1","FONDATION DE FRANCE","FONFFRP1","base.fr"
"bank_fr_fogcfrp1","FONDATION DES GUEULES CASSEES","FOGCFRP1","base.fr"
"bank_fr_foenfrp1","FONDATION ENTREPRENDRE","FOENFRP1","base.fr"
"bank_fr_fhhafr21","FONDATION HANS HARTUNG ANNA-EVA BERGMA","FHHAFR21","base.fr"
"bank_fr_fomafr21","FONDATION MAIF","FOMAFR21","base.fr"
"bank_fr_fomlfrp1","FONDATION MEDERIC ALZHEIMER","FOMLFRP1","base.fr"
"bank_fr_fopbfrp1","FONDATION PAUL BENNETOT","FOPBFRP1","base.fr"
"bank_fr_fdgafrp1","FONDS DE GARANTIE DES DEPOTS","FDGAFRP1","base.fr"
"bank_fr_fdeofrp1","FONDS DEVELOPPEMENT ECONOMIQUE ET SOCIAL","FDEOFRP1","base.fr"
"bank_fr_fdmlfrp1","FONDS DOTATION MUSEE DU LOUVRE","FDMLFRP1","base.fr"
"bank_fr_fgaofr21","FONDS GARANTIE ASSUR OBL DOMMAGES","FGAOFR21","base.fr"
"bank_fr_fogtfr21","FONDS GARANTIE TERRORISME","FOGTFR21","base.fr"
"bank_fr_fpspfrp1","FONDS PARITAIRES DE SECURISATION DES PARCOURS PROF","FPSPFRP1","base.fr"
"bank_fr_frgnfr21","FONDS REGIONAL GARANT NORD PAS DE CALAIS","FRGNFR21","base.fr"
"bank_fr_ftrnfrp1","FONGECFA TRANSPORT","FTRNFRP1","base.fr"
"bank_fr_fogffrp1","FONGEPAR GESTION FINANCIERE SAS","FOGFFRP1","base.fr"
"bank_fr_foitfrp1","FORINTER SARL","FOITFRP1","base.fr"
"bank_fr_foacfrp1","FORTIS ASSURANCES SA","FOACFRP1","base.fr"
"bank_fr_fcfsfrp1","FORTIS COMMERCIAL FINANCE SAS","FCFSFRP1","base.fr"
"bank_fr_foerfrp1","FORTIS EBANKING FRANCE","FOERFRP1","base.fr"
"bank_fr_fogpfrp1","FORTIS GESTION PRIVEE","FOGPFRP1","base.fr"
"bank_fr_mefpfr21","FORTIS GESTION PRIVEE SA","MEFPFR21","base.fr"
"bank_fr_foiffrp1","FORTIS INVESTMENT FINANCE SA","FOIFFRP1","base.fr"
"bank_fr_figpfrp1","FORTIS INVESTMENT MANAGEMENT FRANCE SA","FIGPFRP1","base.fr"
"bank_fr_folefr21","FORTIS LEASE","FOLEFR21","base.fr"
"bank_fr_folffrp1","FORTIS LEASE FRANCE","FOLFFRP1","base.fr"
"bank_fr_fomifrp1","FORTIS MEDIACOM FINANCE","FOMIFRP1","base.fr"
"bank_fr_fosffrp1","FORTIS SECURITIES FRANCE","FOSFFRP1","base.fr"
"bank_fr_forffrp1","FORWARD FINANCE SOCIETE ANONYME","FORFFRP1","base.fr"
"bank_fr_fagffrp1","FRANCE ACTIVE GARANTIE FAG S A","FAGFFRP1","base.fr"
"bank_fr_frtefrp1","FRANCE TELECOM ENCAISSEMENTS SAS","FRTEFRP1","base.fr"
"bank_fr_ftemfrp1","FRANCE TELEVISION IMAGES2","FTEMFRP1","base.fr"
"bank_fr_frtvfrp1","FRANCE TELEVISIONS","FRTVFRP1","base.fr"
"bank_fr_sfftfrp1","FRANCETEL SOCIETE FRANCAISE DE FINANCEMENT DES TELECOMMUNICATIONS SA","SFFTFRP1","base.fr"
"bank_fr_franfr21","FRANFINANCE SA","FRANFR21","base.fr"
"bank_fr_frrcfrp1","FRANK RUSSELL COMPANY LTD","FRRCFRP1","base.fr"
"bank_fr_frtffrp1","FRANKLIN TEMPLETON ASSET MANAGEMENT SA","FRTFFRP1","base.fr"
"bank_fr_fraffrpp","FRANSABANK FRANCE S.A.","FRAFFRPP","base.fr"
"bank_fr_frgefrp1","FRIEDLAND GESTION","FRGEFRP1","base.fr"
"bank_fr_fbelfrpp","FROMAGERIES BEL","FBELFRPP","base.fr"
"bank_fr_frrffrp1","FRR","FRRFFRP1","base.fr"
"bank_fr_frucfrp1","FRUCTIBAIL SAS","FRUCFRP1","base.fr"
"bank_fr_frutfrp1","FRUCTICOMI SA","FRUTFRP1","base.fr"
"bank_fr_fruifrp1","FRUCTIFONDS IMMOBILIER SC","FRUIFRP1","base.fr"
"bank_fr_fsanfrp1","FSA INTERMEDIATION SARL","FSANFRP1","base.fr"
"bank_fr_fumffr21","FUND-MARKET FRANCE SAS","FUMFFR21","base.fr"
"bank_fr_fuaafrp1","FUNDLOGIC SAS","FUAAFRP1","base.fr"
"bank_fr_funqfrp1","FUNDQUEST","FUNQFRP1","base.fr"
"bank_fr_fusmfrp1","FUTUR ASSET MANAGEMENT","FUSMFRP1","base.fr"
"bank_fr_gpfifrp1","G.P.K. FINANCE SA","GPFIFRP1","base.fr"
"bank_fr_glssfrp1","GALAXY SAS","GLSSFRP1","base.fr"
"bank_fr_gagtfr21","GALIA GESTION","GAGTFR21","base.fr"
"bank_fr_gapafrp1","GALILEO PARTNERS","GAPAFRP1","base.fr"
"bank_fr_ganafrp1","GAN ASSURANCES","GANAFRP1","base.fr"
"bank_fr_gbslfrp1","GARBAN SECURITIES LIMITED","GBSLFRP1","base.fr"
"bank_fr_gasefrp1","GASELYS","GASEFRP1","base.fr"
"bank_fr_gasffrp1","GASPAL FINANCE","GASFFRP1","base.fr"
"bank_fr_gagefrp1","GASPAL GESTION","GAGEFRP1","base.fr"
"bank_fr_gcaffrp1","GCE AFFACTURAGE","GCAFFRP1","base.fr"
"bank_fr_gcasfrp1","GCE ASSURANCES","GCASFRP1","base.fr"
"bank_fr_gcblfrp1","GCE BAIL","GCBLFRP1","base.fr"
"bank_fr_ccdffr21","GE CAPITAL BANK LIMITED COMMERCIAL DISTRIBUTION FINANCE PARIS BRANCH","CCDFFR21","base.fr"
"bank_fr_caeffr21","GE CAPITAL EQUIPEMENT FINANCE SAS","CAEFFR21","base.fr"
"bank_fr_caoufrp1","GE CAPITAL EQUIPEMENT FINANCE SAS","CAOUFRP1","base.fr"
"bank_fr_ceqffr21","GE CAPITAL EQUIPEMENT FINANCE SAS","CEQFFR21","base.fr"
"bank_fr_sofbfrp1","GE CAPITAL FINANCE SOFIREC","SOFBFRP1","base.fr"
"bank_fr_cafofrp1","GE CAPITAL FINANCEMENTS IMMOBILIERS D'ENTREPRISE SAS","CAFOFRP1","base.fr"
"bank_fr_codffrp1","GE COMMERCIAL DISTRIBUTION FINANCE","CODFFRP1","base.fr"
"bank_fr_cotufrp1","GE CORPORATE BANKING EUROPE S A S","COTUFRP1","base.fr"
"bank_fr_rbfafrp1","GE FACTOR","RBFAFRP1","base.fr"
"bank_fr_moeyfrpp","GE MONEY BANK SCA","MOEYFRPP","base.fr"
"bank_fr_gecffrpp","GE SCF","GECFFRPP","base.fr"
"bank_fr_gccifrp1","GECINA","GCCIFRP1","base.fr"
"bank_fr_gedifr21","GEDEX DISTRIBUTION","GEDIFR21","base.fr"
"bank_fr_gfcofrpp","GEFCO","GFCOFRPP","base.fr"
"bank_fr_gmtofr22","GEMALTO TREASURY SERVICES","GMTOFR22","base.fr"
"bank_fr_gennfr22","GENEBANQUE SA","GENNFR22","base.fr"
"bank_fr_gencfr21","GENECAL","GENCFR21","base.fr"
"bank_fr_geeefr21","GENECOMI","GEEEFR21","base.fr"
"bank_fr_genffrp1","GENEFIM","GENFFRP1","base.fr"
"bank_fr_geeffrp1","GENEFIMMO","GEEFFRP1","base.fr"
"bank_fr_gecsfrp1","GENERAL ELECTRIC CAPITAL SAS","GECSFRP1","base.fr"
"bank_fr_gsdsfr21","GENERAL SERVICE DEVELOPPEMENT SARL","GSDSFR21","base.fr"
"bank_fr_gefvfrp1","GENERALE DE FINANCEMENTS ET DE SERVICES","GEFVFRP1","base.fr"
"bank_fr_gepgfrp1","GENERALE DE PATRIMOINE ET DE GESTION","GEPGFRP1","base.fr"
"bank_fr_gasrfrp1","GENERALI ASSURANCES IARD SA","GASRFRP1","base.fr"
"bank_fr_gasvfrp1","GENERALI ASSURANCES VIE","GASVFRP1","base.fr"
"bank_fr_gebefrp1","GENERALI BELGIUM","GEBEFRP1","base.fr"
"bank_fr_gefafrp1","GENERALI FINANCES","GEFAFRP1","base.fr"
"bank_fr_gefcfrpa","GENERALI FRANCE","GEFCFRPA","base.fr"
"bank_fr_gefhfrp1","GENERALI FRANCE HOLDING SA","GEFHFRP1","base.fr"
"bank_fr_gefcfrp1","GENERALI FRANCE SA","GEFCFRP1","base.fr"
"bank_fr_gegefrp1","GENERALI GESTION","GEGEFRP1","base.fr"
"bank_fr_gieufrp1","GENERALI INVESTMENT EUROPE","GIEUFRP1","base.fr"
"bank_fr_giopfrp1","GENERALI INVESTMENT OPERA","GIOPFRP1","base.fr"
"bank_fr_geagfrp1","GEORGE V ASSET MANAGEMENT","GEAGFRP1","base.fr"
"bank_fr_gceufr21","GEORGET COURTAGE EUROPEEN SA","GCEUFR21","base.fr"
"bank_fr_gpprfrp1","GERANCE PARISIENNE PRIVEE - G.P.P.","GPPRFRP1","base.fr"
"bank_fr_geclfrp1","GERER CONSEIL","GECLFRP1","base.fr"
"bank_fr_gerpfr21","GERPRO","GERPFR21","base.fr"
"bank_fr_gemmfrp1","GESMOB SA","GEMMFRP1","base.fr"
"bank_fr_gisgfrp1","GESTEPARGNE INVESTISSEMENTS SERVICES - GIS SA","GISGFRP1","base.fr"
"bank_fr_cccafr21","GESTION D'ENCOURS DE CREDITS IMMOBILIERS GECI SAS","CCCAFR21","base.fr"
"bank_fr_ergifr21","GESTION D'EQUIPEMENTS TOURISTIQUES INDUSTRIELS ET COMMERCIAUX-GETIC SA","ERGIFR21","base.fr"
"bank_fr_gfpgfrp1","GESTION FINANCIERE PRIVEE GEFIP SA","GFPGFRP1","base.fr"
"bank_fr_gsstfrp1","GESTION SA","GSSTFRP1","base.fr"
"bank_fr_gvgcfrp1","GESTION VALOR","GVGCFRP1","base.fr"
"bank_fr_gesffrp1","GESTOR FINANCE","GESFFRP1","base.fr"
"bank_fr_getyfrp1","GESTYS SA","GETYFRP1","base.fr"
"bank_fr_gfigfrp1","GFI SECURITIES LTD PARIS BRANCH","GFIGFRP1","base.fr"
"bank_fr_arrcfrp1","GIE AGIRC-ARRCO GROUPEMENT D'INTERET ECONOMIQUE","ARRCFRP1","base.fr"
"bank_fr_axaffrpp","GIE AXA FRANCE","AXAFFRPP","base.fr"
"bank_fr_gigffrp1","GIE GROUPE LA FRANCAISE","GIGFFRP1","base.fr"
"bank_fr_giepfrp1","GIE PSA TRESORERIE","GIEPFRP1","base.fr"
"bank_fr_giinfr21","GIFAO INVESTISSEMENT SA","GIINFR21","base.fr"
"bank_fr_gifsfrp1","GIMAR FINANCE SCA","GIFSFRP1","base.fr"
"bank_fr_gifnfrp1","GINALFI FINANCE","GIFNFRP1","base.fr"
"bank_fr_ginjfrp1","GINJER AM","GINJFRP1","base.fr"
"bank_fr_girafrp1","GIRARDET S.A.","GIRAFRP1","base.fr"
"bank_fr_euckfrp1","GLOBAL EQUITIES CAPITAL MARKETS","EUCKFRP1","base.fr"
"bank_fr_glgefrp1","GLOBAL GESTION SA","GLGEFRP1","base.fr"
"bank_fr_gosbfrp1","GLOBAL OVERSEAS BANK A.D.","GOSBFRP1","base.fr"
"bank_fr_glsnfr21","GLON SANDERS","GLSNFR21","base.fr"
"bank_fr_gmacfr21","GMAC BANQUE","GMACFR21","base.fr"
"bank_fr_gmasfrp1","GMF ASSURANCE","GMASFRP1","base.fr"
"bank_fr_asemfrp1","GO.FX ASSET MANAGEMENT","ASEMFRP1","base.fr"
"bank_fr_gofifr21","GOFFIN BANK NV","GOFIFR21","base.fr"
"bank_fr_goldfrpp","GOLDMAN SACHS PARIS INC. ET CIE","GOLDFRPP","base.fr"
"bank_fr_gopkfrp1","GORGEU PERQUEL KRUCKER","GOPKFRP1","base.fr"
"bank_fr_gohafrp1","GOY HAUVETTE, S.A.","GOHAFRP1","base.fr"
"bank_fr_gpavfrp1","GPA VIE","GPAVFRP1","base.fr"
"bank_fr_gpasfrp1","GPM ASSURANCE","GPASFRP1","base.fr"
"bank_fr_grcofrp1","GRAMONT CONTREPARTIE S.A.","GRCOFRP1","base.fr"
"bank_fr_grekfrp1","GRENKE 2","GREKFRP1","base.fr"
"bank_fr_grnkfr21","GRENKE 3","GRNKFR21","base.fr"
"bank_fr_legrfrp1","GRESHAM BANQUE","LEGRFRP1","base.fr"
"bank_fr_gpamfrpp","GROUPAMA ASSET MANAGEMENT","GPAMFRPP","base.fr"
"bank_fr_gcatfr21","GROUPAMA CENTRE ATLANTIQUE","GCATFR21","base.fr"
"bank_fr_grerfrp1","GROUPAMA EPARGNE SALARIALE SA","GRERFRP1","base.fr"
"bank_fr_grfpfrp1","GROUPAMA FUND PICKERS","GRFPFRP1","base.fr"
"bank_fr_grgefr21","GROUPAMA GRAND EST","GRGEFR21","base.fr"
"bank_fr_grhafr21","GROUPAMA RHONE-ALPES-AUVERGNE","GRHAFR21","base.fr"
"bank_fr_gpsifrp1","GROUPAMA SYSTEMES D'INFORMATION","GPSIFRP1","base.fr"
"bank_fr_grddfr22","GROUPE ADEO","GRDDFR22","base.fr"
"bank_fr_grddfrp1","GROUPE ADEO","GRDDFRP1","base.fr"
"bank_fr_gagrfrp1","GROUPE AG2R","GAGRFRP1","base.fr"
"bank_fr_gpanfrpp","GROUPE ARNAULT","GPANFRPP","base.fr"
"bank_fr_auchfr22","GROUPE AUCHAN","AUCHFR22","base.fr"
"bank_fr_grmafr21","GROUPE CAMACTE","GRMAFR21","base.fr"
"bank_fr_gcarfr21","GROUPE CARRE","GCARFR21","base.fr"
"bank_fr_grdofrp1","GROUPE DUMAS OREPA - D ET O","GRDOFRP1","base.fr"
"bank_fr_fcsafrp1","GROUPE FINANCIERE CENTURIA SAS","FCSAFRP1","base.fr"
"bank_fr_grhufrp1","GROUPE HUMANIS","GRHUFRP1","base.fr"
"bank_fr_grjcfr21","GROUPE JCB","GRJCFR21","base.fr"
"bank_fr_gmonfrp1","GROUPE MONCEAU","GMONFRP1","base.fr"
"bank_fr_grpmfrp1","GROUPE PASTEUR MUTUALITE","GRPMFRP1","base.fr"
"bank_fr_sipffr22","GROUPE PETIT FORESTIER","SIPFFR22","base.fr"
"bank_fr_grobfrp1","GROUPE ROBECO (FRANCE)","GROBFRP1","base.fr"
"bank_fr_segufr21","GROUPE SEGULA TECHNOLOGIES","SEGUFR21","base.fr"
"bank_fr_gsmafr21","GROUPE SMISO MUTUELLE DES CADRES","GSMAFR21","base.fr"
"bank_fr_gufgfrp1","GROUPE UFG-LFP","GUFGFRP1","base.fr"
"bank_fr_grvafr21","GROUPE VAUBAN","GRVAFR21","base.fr"
"bank_fr_gcbafrp1","GROUPEMENT DES CARTES BANCAIRES GIE","GCBAFRP1","base.fr"
"bank_fr_ggesfrp1","GSD GESTION SA","GGESFRP1","base.fr"
"bank_fr_gsmcfrp1","GSM CONSULTING","GSMCFRP1","base.fr"
"bank_fr_fneafrp1","GT FINANCE","FNEAFRP1","base.fr"
"bank_fr_fnnefrp1","GT FINANCE SA","FNNEFRP1","base.fr"
"bank_fr_iceffrp1","GT ICEFUND","ICEFFRP1","base.fr"
"bank_fr_muaefrp1","GT MULTI ALTERNATIVE RECOVERY","MUAEFRP1","base.fr"
"bank_fr_muasfrp1","GT MULTI ALTERNATIVE SELECT","MUASFRP1","base.fr"
"bank_fr_oblifrp1","GT OBLIPLUS","OBLIFRP1","base.fr"
"bank_fr_opptfrp1","GT OPPORTUNITIES","OPPTFRP1","base.fr"
"bank_fr_rhrofrp1","GT RHIN RHONE","RHROFRP1","base.fr"
"bank_fr_gufifrp1","GUARDIAN FINANCES","GUFIFRP1","base.fr"
"bank_fr_ecorfr22","GUY DAUPHIN ENVIRONNEMENT","ECORFR22","base.fr"
"bank_fr_hetafrp1","H ET ASSOCIES SA","HETAFRP1","base.fr"
"bank_fr_hgaafrp1","HAAS GESTION","HGAAFRP1","base.fr"
"bank_fr_habbfrpp","HABIBSONS BANK LIMITED","HABBFRPP","base.fr"
"bank_fr_hcmnfrp1","HALBIS CAPITAL MANAGEMENT (FRANCE) SA","HCMNFRP1","base.fr"
"bank_fr_hamtfrp1","HAMANT","HAMTFRP1","base.fr"
"bank_fr_hadifr21","HANDIMUT SA","HADIFR21","base.fr"
"bank_fr_haaafrp1","HAREWOOD ASSET MANAGEMENT","HAAAFRP1","base.fr"
"bank_fr_harmfr21","HARMONIE MUTUELLE","HARMFR21","base.fr"
"bank_fr_hinmfrp1","HAUSSMANN INVESTISSEMENT MANAGERS","HINMFRP1","base.fr"
"bank_fr_hvasfr21","HAVAS","HVASFR21","base.fr"
"bank_fr_hafifrp1","HAW FINANCE","HAFIFRP1","base.fr"
"bank_fr_hatifrp1","HAYAUX DU TILLY S.A. SOCIETE DE BOURSE","HATIFRP1","base.fr"
"bank_fr_ivvnfr21","HD INVESTISSEMENT","IVVNFR21","base.fr"
"bank_fr_hdfifrp1","HDF FINANCE SA","HDFIFRP1","base.fr"
"bank_fr_heptfrp1","HEDIOS PATRIMOINE","HEPTFRP1","base.fr"
"bank_fr_hegvfrp1","HENDERSON GLOBAL INVESTORS LIMITED","HEGVFRP1","base.fr"
"bank_fr_hennfrpp","HENNER SAS","HENNFRPP","base.fr"
"bank_fr_hechfr21","HENRY DE CHAMPSAVIN S.A.","HECHFR21","base.fr"
"bank_fr_heppfrp1","HEPPNER","HEPPFRP1","base.fr"
"bank_fr_hrmsfrpp","HERMES INTERNATIONAL","HRMSFRPP","base.fr"
"bank_fr_hertfrp1","HERMITAGE","HERTFRP1","base.fr"
"bank_fr_hcrefr21","HERVET CREDITERME","HCREFR21","base.fr"
"bank_fr_hpfffr21","HEWLETT PACKARD FRANCE FINANCE","HPFFFR21","base.fr"
"bank_fr_hglgfrp1","HGL GESTION","HGLGFRP1","base.fr"
"bank_fr_hicxfr21","HIGH CONNEXION","HICXFR21","base.fr"
"bank_fr_hixafrp1","HIXANCE","HIXAFRP1","base.fr"
"bank_fr_hmgffrpp","HMG FINANCE SA","HMGFFRPP","base.fr"
"bank_fr_hmygfr22","HMY GROUP SAS","HMYGFR22","base.fr"
"bank_fr_hhgpfrp1","HOGEP - HOCHE GESTION PRIVEE SA","HHGPFRP1","base.fr"
"bank_fr_holdfr22","HOLDHAM","HOLDFR22","base.fr"
"bank_fr_homefrp1","HOME 7/7","HOMEFRP1","base.fr"
"bank_fr_hostfrp1","HOSTA FI SA","HOSTFRP1","base.fr"
"bank_fr_hotefrp1","HOTELIM","HOTEFRP1","base.fr"
"bank_fr_hlhzfrp1","HOULIHAN LOKEY HOWARD ET ZUKIN EUROPE","HLHZFRP1","base.fr"
"bank_fr_hpcpfrp1","HPC SA","HPCPFRP1","base.fr"
"bank_fr_getnfrp1","HR GESTION SA","GETNFRP1","base.fr"
"bank_fr_midlfrpx","HSBC BANK PLC","MIDLFRPX","base.fr"
"bank_fr_hcamfr21","HSBC CCF ASSET MANAGEMENT GROUP","HCAMFR21","base.fr"
"bank_fr_hciffrp1","HSBC CCF INVESTMENT BANK (FRANCE)","HCIFFRP1","base.fr"
"bank_fr_heeffrp1","HSBC EPARGNE ENTREPRISE (FRANCE) SA","HEEFFRP1","base.fr"
"bank_fr_elfafrp1","HSBC FACTORING FRANCE","ELFAFRP1","base.fr"
"bank_fr_hfpffrp1","HSBC FINANCIAL PRODUCTS (FRANCE) SNC","HFPFFRP1","base.fr"
"bank_fr_ccfrfrpp","HSBC FRANCE","CCFRFRPP","base.fr"
"bank_fr_hsiffr21","HSBC GLOBAL ASSET MANAGEMENT (FRANCE)","HSIFFR21","base.fr"
"bank_fr_hamffrpp","HSBC GLOBAL ASSET MANAGEMENT FRANCE","HAMFFRPP","base.fr"
"bank_fr_samffrpp","HSBC GLOBAL ASSET MANAGEMENT FRANCE","SAMFFRPP","base.fr"
"bank_fr_hiblfrp1","HSBC INVESTMENT BANK PLC","HIBLFRP1","base.fr"
"bank_fr_hslffrp1","HSBC LEASING (FRANCE) S.A.S","HSLFFRP1","base.fr"
"bank_fr_logsfrp1","HSBC PRIVATE WEALTH MANAGERS","LOGSFRP1","base.fr"
"bank_fr_hrelfrp1","HSBC REAL ESTATE LEASING (FRANCE) SA","HRELFRP1","base.fr"
"bank_fr_hsfhfrpp","HSBC SFH (FRANCE)","HSFHFRPP","base.fr"
"bank_fr_hugefrp1","HUGAU GESTION SAS","HUGEFRP1","base.fr"
"bank_fr_fxbbfrpp","IBANFIRST","FXBBFRPP","base.fr"
"bank_fr_ibfffr21","IBM FRANCE FINANCEMENT SA","IBFFFR21","base.fr"
"bank_fr_aiamfrp1","ICMOS FRANCE","AIAMFRP1","base.fr"
"bank_fr_icpefr21","ICSO PRIVATE EQUITY","ICPEFR21","base.fr"
"bank_fr_idmufrp1","IDENTITES MUTUELLE","IDMUFRP1","base.fr"
"bank_fr_idlsfr21","IDES INVESTISSEMENTS SA","IDLSFR21","base.fr"
"bank_fr_idagfrp1","IDI ASSET MANAGEMENT","IDAGFRP1","base.fr"
"bank_fr_idiafrp1","IDIA","IDIAFRP1","base.fr"
"bank_fr_idiffrp1","IDIFINE SA","IDIFFRP1","base.fr"
"bank_fr_idpafrp1","IDINVEST PARTNERS","IDPAFRP1","base.fr"
"bank_fr_ifbofrp1","IFF BOURSE","IFBOFRP1","base.fr"
"bank_fr_iffifr21","IFN FINANCE SA","IFFIFR21","base.fr"
"bank_fr_maktfrp1","IG MARKETS","MAKTFRP1","base.fr"
"bank_fr_igfifrp1","IGEA FINANCE","IGFIFRP1","base.fr"
"bank_fr_imerfrpp","IMERYS SA","IMERFRPP","base.fr"
"bank_fr_imelfrp1","IMMOBILIER ELYBAIL","IMELFRP1","base.fr"
"bank_fr_imgefrp1","IMMOVALOR GESTION","IMGEFRP1","base.fr"
"bank_fr_imacfr21","IMPERIO ASSURANCES ET CAPITALISATION SA","IMACFR21","base.fr"
"bank_fr_ichcfr21","INCHCAPE FRANCE FINANCE","ICHCFR21","base.fr"
"bank_fr_inmpfrp1","INDEP'AM","INMPFRP1","base.fr"
"bank_fr_vpksfr21","INDIGO PARK","VPKSFR21","base.fr"
"bank_fr_icbkfrpp","INDUSTRIAL AND COMMERCIAL BANK OF CHINA (EUROPE) S.A., PARIS BRANCH","ICBKFRPP","base.fr"
"bank_fr_istafr22","INFOVISTA HOLDING","ISTAFR22","base.fr"
"bank_fr_ingbfrpp","ING BANK FRANCE COMMERCIAL BANKING","INGBFRPP","base.fr"
"bank_fr_ingbfr21","ING BANK FRANCE-RETAIL BANKING (ING DIRECT)","INGBFR21","base.fr"
"bank_fr_igfefr21","ING FERRI SA","IGFEFR21","base.fr"
"bank_fr_igiafr21","ING INVESTMENT MANAGEMENT","IGIAFR21","base.fr"
"bank_fr_inlffrp1","ING LEASE FRANCE S.A.","INLFFRP1","base.fr"
"bank_fr_igrefrp1","ING REAL ESTATE FINANCE (FRANCE)","IGREFRP1","base.fr"
"bank_fr_igenfr22","INGENICO","IGENFR22","base.fr"
"bank_fr_iogefrp1","INNOVACOM GESTION","IOGEFRP1","base.fr"
"bank_fr_iopafrp1","INNOVEN PARTENAIRES","IOPAFRP1","base.fr"
"bank_fr_iocpfrp1","INOCAP","IOCPFRP1","base.fr"
"bank_fr_isntfrp1","INSTINET FRANCE S.A.","ISNTFRP1","base.fr"
"bank_fr_inddfrp1","INSTITUT D'EMISSION D'OUTRE-MER","INDDFRP1","base.fr"
"bank_fr_idecfrp1","INSTITUT DE DEVELOPPEMENT COOPERATIF","IDECFRP1","base.fr"
"bank_fr_idpcfrp1","INSTITUT DE DEVELOPPEMENT ET DE PARTICIPATION AU CAPITAL SA","IDPCFRP1","base.fr"
"bank_fr_inpyfr21","INSTITUT DE PREVOYANCE VALMY","INPYFR21","base.fr"
"bank_fr_iesufrp1","INSTITUT ETUDE SECURITE UNION EUROPEEN","IESUFRP1","base.fr"
"bank_fr_ipasfrp1","INSTITUT PASTEUR","IPASFRP1","base.fr"
"bank_fr_ifcifrp1","INSTITUT POUR FINANCEMENT CINEMA ET INDUSTRIES CULTURELLES SA","IFCIFRP1","base.fr"
"bank_fr_inpzfrp1","INSTITUTION DE PREVOYANCE AUSTERLITZ","INPZFRP1","base.fr"
"bank_fr_irclfrp1","INSTITUTION DE RETRAITE COMPLEMENTAIRE DE L'ENSEIGNEMENT ET DE LA CREATION","IRCLFRP1","base.fr"
"bank_fr_iirsfr21","INSTITUTION INTERPROFESSIONNELLE RETRAITE SALARIES","IIRSFR21","base.fr"
"bank_fr_irpsfrp1","INSTITUTION RETRAITE ET PREVOYANCE SALARIES","IRPSFRP1","base.fr"
"bank_fr_idamfrp1","INTEGRAL DEVELOPMENT ASSET MANAGEMENT","IDAMFRP1","base.fr"
"bank_fr_inoofr21","INTER COOP","INOOFR21","base.fr"
"bank_fr_inelfrpp","INTER EUROPE CONSEIL","INELFRPP","base.fr"
"bank_fr_inepfr21","INTER EXPANSION SOCIETE ANONYME","INEPFR21","base.fr"
"bank_fr_ifiofr21","INTER FINANCES DE L'OUEST","IFIOFR21","base.fr"
"bank_fr_itiefrp1","INTER INVEST","ITIEFRP1","base.fr"
"bank_fr_itmafr21","INTER MUTUELLE ASSISTANCE SA","ITMAFR21","base.fr"
"bank_fr_iparfrpp","INTER PARFUMS S.A","IPARFRPP","base.fr"
"bank_fr_inrffrp1","INTERFIMO SOCIETE ANONYME","INRFFRP1","base.fr"
"bank_fr_ittifrp1","INTERIALE","ITTIFRP1","base.fr"
"bank_fr_inrmfrp1","INTERMEDIA BANQUE","INRMFRP1","base.fr"
"bank_fr_icagfrp1","INTERNATIONAL CAPITAL GESTION SA","ICAGFRP1","base.fr"
"bank_fr_icssfr21","INTERNATIONAL CREDIT SERVICES SAS","ICSSFR21","base.fr"
"bank_fr_idsrfr21","INTERNATIONAL DEVELOPEMENT SERVICES","IDSRFR21","base.fr"
"bank_fr_infffrp1","INTERNATIONAL FINANCE FUTURES - IFF SNC","INFFFRP1","base.fr"
"bank_fr_ingnfrp1","INTERNATIONALE NEDERLANDEN BOURSE S.A.","INGNFRP1","base.fr"
"bank_fr_icpofr22","INTERPOL SERVICE FINANCIER","ICPOFR22","base.fr"
"bank_fr_inudfr21","INTERSELECTION ACTUARIAT ADVISER","INUDFR21","base.fr"
"bank_fr_bcitfrpp","INTESA SANPAOLO SPA","BCITFRPP","base.fr"
"bank_fr_inrafrp1","INVESCO FRANCE SA","INRAFRP1","base.fr"
"bank_fr_invzfrpp","INVESCO GESTION","INVZFRPP","base.fr"
"bank_fr_imiffrp1","INVESCO MIM FRANCE","IMIFFRP1","base.fr"
"bank_fr_ievsfrp1","INVEST AM","IEVSFRP1","base.fr"
"bank_fr_inuufrp1","INVEST SECURITIES SOCIETE ANONYME","INUUFRP1","base.fr"
"bank_fr_iniufrp1","INVESTIMUR","INIUFRP1","base.fr"
"bank_fr_ideifrp1","INVESTISSEURS DANS L'ENTREPRISE SA","IDEIFRP1","base.fr"
"bank_fr_ivelfr21","INVESTLIFE","IVELFR21","base.fr"
"bank_fr_irevfrp1","INVISTA REAL ESTATE INVESTMENT MANAGEMENT","IREVFRP1","base.fr"
"bank_fr_orunfrpp","IPAGOO LLP FRENCH BRANCH","ORUNFRPP","base.fr"
"bank_fr_ippsfr21","IPANEVA PAYMENT ASSOCIATION","IPPSFR21","base.fr"
"bank_fr_ipbmfrp1","IPBM SA","IPBMFRP1","base.fr"
"bank_fr_ippefrp1","IPECA-PREVOYANCE","IPPEFRP1","base.fr"
"bank_fr_ipicfrp1","IPICAS","IPICFRP1","base.fr"
"bank_fr_ipsnfrpp","IPSEN","IPSNFRPP","base.fr"
"bank_fr_ipsofrpp","IPSOS","IPSOFRPP","base.fr"
"bank_fr_irprfr21","IRCEM PREVOYANCE","IRPRFR21","base.fr"
"bank_fr_irfnfrp1","IRIS FINANCE","IRFNFRP1","base.fr"
"bank_fr_irfifrp1","IRIS FINANCE SARL","IRFIFRP1","base.fr"
"bank_fr_isgefrp1","ISAI GESTION","ISGEFRP1","base.fr"
"bank_fr_isacfrp1","ISATIS CAPITAL","ISACFRP1","base.fr"
"bank_fr_isbkfrpp","ISBANK AG","ISBKFRPP","base.fr"
"bank_fr_isanfrp1","ISIS ASSET MANAGEMENT","ISANFRP1","base.fr"
"bank_fr_iskafrp1","ISKANDER","ISKAFRP1","base.fr"
"bank_fr_assgfrp1","IT ASSET MANAGEMENT SA","ASSGFRP1","base.fr"
"bank_fr_ttcefrpp","IT-CE","TTCEFRPP","base.fr"
"bank_fr_ivcafrp1","IVO CAPITAL PARTNERS","IVCAFRP1","base.fr"
"bank_fr_ixmifrp1","IXIS MIDCAPS","IXMIFRP1","base.fr"
"bank_fr_ixipfrp1","IXIS PCM","IXIPFRP1","base.fr"
"bank_fr_ipcmfrp1","IXIS PRIVATE CAPITAL MANAGEMENT","IPCMFRP1","base.fr"
"bank_fr_degtfrp1","J. DE DEMANDOLX GESTION SA","DEGTFRP1","base.fr"
"bank_fr_mmivfrp1","J.P. MORGAN MANSART INVESTMENTS","MMIVFRP1","base.fr"
"bank_fr_jcppfrp1","JAMES CAPEL S.A.","JCPPFRP1","base.fr"
"bank_fr_drahfrp1","JB DRAX HONORE SAS","DRAHFRP1","base.fr"
"bank_fr_jcdxfr22","JC DECAUX SA","JCDXFR22","base.fr"
"bank_fr_jclffrp1","JCL FINANCE","JCLFFRP1","base.fr"
"bank_fr_jcgrfrpp","JEAN CASSEGRAIN","JCGRFRPP","base.fr"
"bank_fr_jodcfr21","JOHN DEERE CREDIT SAS","JODCFR21","base.fr"
"bank_fr_jolifr21","JOHN LOCKE INVESTMENTS","JOLIFR21","base.fr"
"bank_fr_kleifr21","JP KLEIN INVESTISSEMENT SA","KLEIFR21","base.fr"
"bank_fr_jamffrp1","JPMORGAN ASSET MANAGEMENT FRANCE","JAMFFRP1","base.fr"
"bank_fr_chasfrpb","JPMORGAN CHASE BANK, N.A.","CHASFRPB","base.fr"
"bank_fr_chasfrpp","JPMORGAN CHASE BANK, N.A.","CHASFRPP","base.fr"
"bank_fr_jyskfr21","JYSKE BANK A/S","JYSKFR21","base.fr"
"bank_fr_karofrp1","KARAKORAM","KAROFRP1","base.fr"
"bank_fr_kaupfrp1","KAUPTHING BANK HF","KAUPFRP1","base.fr"
"bank_fr_kbbffr21","KBC BAIL FRANCE SAS","KBBFFR21","base.fr"
"bank_fr_kbiafrp1","KBC BAIL IMMOBILIER FRANCE","KBIAFRP1","base.fr"
"bank_fr_kbsefrp1","KBC SECURITIES SUCCURSALE FRANCAISE","KBSEFRP1","base.fr"
"bank_fr_kbfgfrpp","KBL FRANCE GESTION","KBFGFRPP","base.fr"
"bank_fr_kblxfrpp","KBL RICHELIEU BANQUE PRIVEE","KBLXFRPP","base.fr"
"bank_fr_koexfrpp","KEB HANA BANK","KOEXFRPP","base.fr"
"bank_fr_kmbffr21","KEMPF BFSC","KMBFFR21","base.fr"
"bank_fr_jbbrfrpp","KEPLER CAPITAL MARKETS","JBBRFRPP","base.fr"
"bank_fr_kerffrp1","KEREN FINANCE SAS","KERFFRP1","base.fr"
"bank_fr_pprffrpp","KERING FINANCE","PPRFFRPP","base.fr"
"bank_fr_rersfrp1","KIPLINK FINANCE","RERSFRP1","base.fr"
"bank_fr_kirafrp1","KIRAO","KIRAFRP1","base.fr"
"bank_fr_klelfrp1","KLELINE GESTION SAS","KLELFRP1","base.fr"
"bank_fr_klpifrpp","KLEPIERRE MANAGEMENT","KLPIFRPP","base.fr"
"bank_fr_kleafrp1","KLESIA","KLEAFRP1","base.fr"
"bank_fr_momufrp1","KLESIA FINANCES","MOMUFRP1","base.fr"
"bank_fr_kmgefrp1","KMS GESTION S A","KMGEFRP1","base.fr"
"bank_fr_kofffrp1","KOMATSU FINANCIAL FRANCE SAS","KOFFFRP1","base.fr"
"bank_fr_baskfr21","KUTXABANK","BASKFR21","base.fr"
"bank_fr_kyrifrpp","KYRIBA SAS","KYRIFRPP","base.fr"
"bank_fr_cpmgfrp1","L CAPITAL MANAGEMENT","CPMGFRP1","base.fr"
"bank_fr_lauxfr21","L'AUXILIAIRE","LAUXFR21","base.fr"
"bank_fr_acfffrp1","L'AUXILIAIRE DU CREDIT FONCIER DE FRANCE","ACFFFRP1","base.fr"
"bank_fr_lionfrp1","L'LIONE FINANCE CONSEIL","LIONFRP1","base.fr"
"bank_fr_lbgafrpp","LA BANQUE POSTALE ASSET MANAGEMENT","LBGAFRPP","base.fr"
"bank_fr_crnpfrp1","LA BANQUE POSTALE CREDIT ENTREPRISES","CRNPFRP1","base.fr"
"bank_fr_bpfifrp1","LA BANQUE POSTALE FINANCEMENT","BPFIFRP1","base.fr"
"bank_fr_hlbpfrpp","LA BANQUE POSTALE HOME LOAN SFH","HLBPFRPP","base.fr"
"bank_fr_stenfrp1","LA BANQUE POSTALE STRUCTURED ASSET MANAGEMENT","STENFRP1","base.fr"
"bank_fr_cobpfrp1","LA BANQUE PRIVEE 1818","COBPFRP1","base.fr"
"bank_fr_ccbifrp1","LA COMPAGNIE DE BANQUES INTERNATIONALES DE PARIS SA","CCBIFRP1","base.fr"
"bank_fr_cmftfrp1","LA COMPAGNIE FINANCIERE ALCATEL-LUCENT","CMFTFRP1","base.fr"
"bank_fr_crogfrp1","LA CROIX ROUGE","CROGFRP1","base.fr"
"bank_fr_fecofrp1","LA FEDERATION CONTINENTALE S.A.","FECOFRP1","base.fr"
"bank_fr_fnlufrp1","LA FINANCIERE DE L'EUROPE","FNLUFRP1","base.fr"
"bank_fr_fmagfrp1","LA FINANCIERE MAGELLAN SAS","FMAGFRP1","base.fr"
"bank_fr_frcifr21","LA FINANCIERE REGIONALE DE CREDIT IMMOBILIER DE BRETAGNE SA","FRCIFR21","base.fr"
"bank_fr_fipofrp1","LA FINANCIERE RESPONSABLE","FIPOFRP1","base.fr"
"bank_fr_fiolfr21","LA FINANCIERE TIEPOLO","FIOLFR21","base.fr"
"bank_fr_frncfrp1","LA FRANCAISE AM","FRNCFRP1","base.fr"
"bank_fr_mutffrp1","LA FRANCAISE AM","MUTFFRP1","base.fr"
"bank_fr_frupfrp1","LA FRANCAISE BANK SUCCURSALE DE PARIS","FRUPFRP1","base.fr"
"bank_fr_fjeufrp1","LA FRANCAISE DES JEUX","FJEUFRP1","base.fr"
"bank_fr_frplfrp1","LA FRANCAISE DES PLACEMENTS","FRPLFRP1","base.fr"
"bank_fr_fpgpfr21","LA FRANCAISE DES PLACEMENTS GESTION PRIVEE","FPGPFR21","base.fr"
"bank_fr_friofrp1","LA FRANCAISE INVESTMENT SOLUTIONS","FRIOFRP1","base.fr"
"bank_fr_frrmfrp1","LA FRANCAISE REM","FRRMFRP1","base.fr"
"bank_fr_fmutfrp1","LA FRANCE MUTUALISTE","FMUTFRP1","base.fr"
"bank_fr_agrgfrp1","LA MEDICALE DE FRANCE","AGRGFRP1","base.fr"
"bank_fr_mcaifr21","LA MONDIALE CASH GROUPEMENT D INTERET ECONOMIQUE","MCAIFR21","base.fr"
"bank_fr_mogdfr21","LA MONDIALE GESTION D'ACTIFS SA","MOGDFR21","base.fr"
"bank_fr_grmofrp1","LA MONDIALE GROUPE","GRMOFRP1","base.fr"
"bank_fr_moprfrp1","LA MONDIALE PARTENAIRE SA","MOPRFRP1","base.fr"
"bank_fr_mparfr21","LA MONDIALE PARTICIPATIONS SA","MPARFR21","base.fr"
"bank_fr_mudofrp1","LA MUTUELLE DES DOUANES","MUDOFRP1","base.fr"
"bank_fr_mutsfrp1","LA MUTUELLE DU TRESOR","MUTSFRP1","base.fr"
"bank_fr_mugefrp1","LA MUTUELLE GENERALE","MUGEFRP1","base.fr"
"bank_fr_nofcfrp1","LA NOUVELLE FINANCE","NOFCFRP1","base.fr"
"bank_fr_pfoafrp1","LA PERENNITE CIE FAVOR OPERAT ASSUR CA","PFOAFRP1","base.fr"
"bank_fr_scorfr21","LA RUCHE SA","SCORFR21","base.fr"
"bank_fr_lagefrpp","LABCO GESTION GIE","LAGEFRPP","base.fr"
"bank_fr_lathfr22","LABORATOIRES THEA","LATHFR22","base.fr"
"bank_fr_lafafrpp","LAFARGE SA","LAFAFRPP","base.fr"
"bank_fr_lcmafrp1","LAFFITTE CAPITAL MANAGEMENT SAS","LCMAFRP1","base.fr"
"bank_fr_lgspfrpp","LAGARDERE UNLIMITED","LGSPFRPP","base.fr"
"bank_fr_lagrfr21","LAMAZERE GESTION PRIVEE","LAGRFR21","base.fr"
"bank_fr_lmeffr22","LAMINES MARCHANDS EUROPEENS","LMEFFR22","base.fr"
"bank_fr_lasrfr21","LANDESBANK SAAR (SAARLB)","LASRFR21","base.fr"
"bank_fr_lacmfrp1","LATITUDE CAPITAL MANAGEMENT","LACMFRP1","base.fr"
"bank_fr_lazlfr21","LAZARD CAPITAL MARKETS","LAZLFR21","base.fr"
"bank_fr_lazpfrcc","LAZARD FRERES BANQUE","LAZPFRCC","base.fr"
"bank_fr_lazpfrpp","LAZARD FRERES BANQUE","LAZPFRPP","base.fr"
"bank_fr_lafgfrpp","LAZARD FRERES GESTION","LAFGFRPP","base.fr"
"bank_fr_lazlfrp1","LAZARD INSTRUMENTS FINANCIERS AND CIE","LAZLFRP1","base.fr"
"bank_fr_lbblfr21","LBS LANDESBAUSPARKASSE BADEN-WURTTEMBERG","LBBLFR21","base.fr"
"bank_fr_cppifrp1","LC CAPITAL","CPPIFRP1","base.fr"
"bank_fr_lclpfrp1","LCL BANQUE PRIVEE","LCLPFRP1","base.fr"
"bank_fr_megifrp1","LE MEIGNEN, RIVAUD ET CIE","MEGIFRP1","base.fr"
"bank_fr_soumfr21","LE SOU MEDICAL","SOUMFR21","base.fr"
"bank_fr_lepffr21","LEASE PLAN FINANCE SA","LEPFFR21","base.fr"
"bank_fr_labffrp1","LEBANESE ARAB BANK FRANCE SA","LABFFRP1","base.fr"
"bank_fr_lecrfrp1","LECUREUR S.A","LECRFRP1","base.fr"
"bank_fr_lgamfrp1","LEGAL AND GENERAL ASSET MANAGEMENT (FRANCE)","LGAMFRP1","base.fr"
"bank_fr_legdfr2l","LEGRAND FRANCE SA","LEGDFR2L","base.fr"
"bank_fr_lewafr21","LEMON WAY","LEWAFR21","base.fr"
"bank_fr_ifdyfr21","LES DOCKS LYONNAIS SA","IFDYFR21","base.fr"
"bank_fr_mboufr21","LES MUTUELLES BOURBONNAISES","MBOUFR21","base.fr"
"bank_fr_lecpfrpp","LESAFFRE ET COMPAGNIE","LECPFRPP","base.fr"
"bank_fr_levnfrp1","LEVEN CHAUSSIER S.A.","LEVNFRP1","base.fr"
"bank_fr_levefrp1","LEVEN S.A.","LEVEFRP1","base.fr"
"bank_fr_lfaafrp1","LFPI ASSET MANAGEMENT","LFAAFRP1","base.fr"
"bank_fr_lgiafrp1","LGA INVESTISSEMENT ASSOCIE","LGIAFRP1","base.fr"
"bank_fr_lieefr21","LIBEA","LIEEFR21","base.fr"
"bank_fr_lixbfrp1","LIXXBAIL SA","LIXBFRP1","base.fr"
"bank_fr_lixxfr21","LIXXCREDIT","LIXXFR21","base.fr"
"bank_fr_lodnfr21","LOCA DIN","LODNFR21","base.fr"
"bank_fr_loaafr21","LOCAM - LOCATION AUTOMOBILES MATERIELS SAS","LOAAFR21","base.fr"
"bank_fr_locufr21","LOCAMUR-SOFIGROS","LOCUFR21","base.fr"
"bank_fr_lcndfrp1","LOCINDUS","LCNDFRP1","base.fr"
"bank_fr_lofifrp1","LOISIRS FINANCE SA","LOFIFRP1","base.fr"
"bank_fr_locyfrpp","LOMBARD ODIER (EUROPE) SA SUCCURSALE EN FRANCE","LOCYFRPP","base.fr"
"bank_fr_lodhfrp1","LOMBARD ODIER DARIER HENTSCH GESTION SA","LODHFRP1","base.fr"
"bank_fr_lofpfrp1","LONDON FORFAITING A PARIS SA","LOFPFRP1","base.fr"
"bank_fr_loaefrp1","LONGCHAMP ASSET MANAGEMENT","LOAEFRP1","base.fr"
"bank_fr_lcmlfrp1","LOUIS CAPITAL MARKETS UK LLP","LCMLFRP1","base.fr"
"bank_fr_ldarfr22","LOUIS DREYFUS ARMATEURS","LDARFR22","base.fr"
"bank_fr_drlofrp1","LOUIS DREYFUS SAS","DRLOFRP1","base.fr"
"bank_fr_lourfrp1","LOURMEL","LOURFRP1","base.fr"
"bank_fr_louvfrpp","LOUVRE HOTELS GROUP","LOUVFRPP","base.fr"
"bank_fr_loxafr2x","LOXAM HOLDING","LOXAFR2X","base.fr"
"bank_fr_loxxfrp1","LOXXIA","LOXXFRP1","base.fr"
"bank_fr_locrfrp1","LOXXIA CREDIT","LOCRFRP1","base.fr"
"bank_fr_loslfr21","LOXXIABAIL SLIBAIL","LOSLFR21","base.fr"
"bank_fr_luddfr22","LUDENDO ENTREPRISES","LUDDFR22","base.fr"
"bank_fr_lutcfrp1","LUTETIA CAPITAL","LUTCFRP1","base.fr"
"bank_fr_lvmhfrpp","LVMH - MOET HENNESSY LOUIS VUITTON","LVMHFRPP","base.fr"
"bank_fr_lygpfr21","LYON GESTION PRIVEE SA","LYGPFR21","base.fr"
"bank_fr_lycofr21","LYRA COLLECT","LYCOFR21","base.fr"
"bank_fr_lyrefr22","LYRECO SAS","LYREFR22","base.fr"
"bank_fr_lamxfrpp","LYXOR ASSET MANAGEMENT","LAMXFRPP","base.fr"
"bank_fr_lyntfrp1","LYXOR INTERMEDIATION","LYNTFRP1","base.fr"
"bank_fr_limxfrpp","LYXOR INTERNATIONAL ASSET MANAGEMENT","LIMXFRPP","base.fr"
"bank_fr_cemnfr21","M COMME MUTUELLE CONSEILS","CEMNFR21","base.fr"
"bank_fr_compfrp1","M DE COMPIEGNE","COMPFRP1","base.fr"
"bank_fr_mffmfr21","M F F SAS","MFFMFR21","base.fr"
"bank_fr_mjepfrp1","M.A.J. (ELIS PANTIN)","MJEPFRP1","base.fr"
"bank_fr_maaufr21","MAAF ASSURANCES SA","MAAUFR21","base.fr"
"bank_fr_maagfrp1","MAAF GESTION SA","MAAGFRP1","base.fr"
"bank_fr_macgfrp1","MACIF GESTION SA","MACGFRP1","base.fr"
"bank_fr_mmfifrp1","MACIF MUTAVIE FINANCE","MMFIFRP1","base.fr"
"bank_fr_mapifrp1","MACIF PARTICIPATIONS","MAPIFRP1","base.fr"
"bank_fr_mccffrp1","MACIF-MUTUALITE","MCCFFRP1","base.fr"
"bank_fr_matafr21","MACIFILIA SA","MATAFR21","base.fr"
"bank_fr_mafffrp1","MACIFIMO SAS","MAFFFRP1","base.fr"
"bank_fr_mauofrp1","MACQUARIE CAPITAL (EUROPE) LIMITED","MAUOFRP1","base.fr"
"bank_fr_mcaufr21","MACSF ASSURANCES","MCAUFR21","base.fr"
"bank_fr_muerfr21","MACSF EPARGNE RETRAITE SOCIETE ANONYME","MUERFR21","base.fr"
"bank_fr_mafifrp1","MACSF FINANCEMENT SA","MAFIFRP1","base.fr"
"bank_fr_magpfr21","MAGENTA PATRIMOINE SA","MAGPFR21","base.fr"
"bank_fr_magnfrp1","MAGNIN S.A.","MAGNFRP1","base.fr"
"bank_fr_mmaifr21","MAIF-MUTUELLE ASSURANCE INSTITUTEUR FRANCE","MMAIFR21","base.fr"
"bank_fr_sonpfr21","MAISONS AND CITES SOGINORPA SAS","SONPFR21","base.fr"
"bank_fr_mfsafr21","MAN FINANCIAL SERVICES SAS","MFSAFR21","base.fr"
"bank_fr_mpcofrp1","MANAGEMENT PATRIMONIAL CONSEIL SAS","MPCOFRP1","base.fr"
"bank_fr_maotfrp1","MANDARINE GESTION","MAOTFRP1","base.fr"
"bank_fr_mpayfrp1","MANGOPAY FRANCE SA","MPAYFRP1","base.fr"
"bank_fr_maoufr21","MANITOU BF","MAOUFR21","base.fr"
"bank_fr_miaifrp1","MARCHES INTER ACTIONS (MIA)","MIAIFRP1","base.fr"
"bank_fr_magofrp1","MARIGNAN GESTION SA","MAGOFRP1","base.fr"
"bank_fr_makpfrp1","MARKET PAY","MAKPFRP1","base.fr"
"bank_fr_mrrkfrp1","MARKUS AM","MRRKFRP1","base.fr"
"bank_fr_maenfr21","MARSH FINANCES S.A.","MAENFR21","base.fr"
"bank_fr_mmgifrp1","MARTIN MAUREL - GESTION INSTITUTIONNELLE SA","MMGIFRP1","base.fr"
"bank_fr_mmacfrp1","MARTIN MAUREL COURTAGE","MMACFRP1","base.fr"
"bank_fr_mmgefr21","MARTIN MAUREL GESTION SA","MMGEFR21","base.fr"
"bank_fr_mmvifrp1","MARTIN MAUREL VIE","MMVIFRP1","base.fr"
"bank_fr_mgptfr21","MARVEYRE GESTION PRIVEE ENTR D INV","MGPTFR21","base.fr"
"bank_fr_mfgefrp1","MASSENA FINANCE GESTION SA","MFGEFRP1","base.fr"
"bank_fr_maoefrp1","MASSERAN GESTION","MAOEFRP1","base.fr"
"bank_fr_mabifrp1","MASSILIA BAIL 2 SAS","MABIFRP1","base.fr"
"bank_fr_matffrp1","MATIGNON FINANCES SA","MATFFRP1","base.fr"
"bank_fr_mavgfrp1","MATIGNON INVESTISSEMENT ET GESTION","MAVGFRP1","base.fr"
"bank_fr_mauafr21","MATMUT ASSURANCE","MAUAFR21","base.fr"
"bank_fr_eumufr21","MATMUT VIE SA","EUMUFR21","base.fr"
"bank_fr_maubfrpp","MAUBOUSSIN","MAUBFRPP","base.fr"
"bank_fr_mdfrfr21","MAVEN DERIVATIVES LTD-FRENCH BRANCH","MDFRFR21","base.fr"
"bank_fr_mbldfr21","MBLD","MBLDFR21","base.fr"
"bank_fr_mbwsfrpp","MBWS S.A.","MBWSFRPP","base.fr"
"bank_fr_mcaffr21","MCA FINANCE SOCIETE ANONYME","MCAFFR21","base.fr"
"bank_fr_mcbrfrpp","MCBRIDE SAS","MCBRFRPP","base.fr"
"bank_fr_medafrp1","MEDIATIS SA","MEDAFRP1","base.fr"
"bank_fr_menrfrp1","MEDICALE DE FRANCE","MENRFRP1","base.fr"
"bank_fr_megefrp1","MEDICIS GESTION SARL","MEGEFRP1","base.fr"
"bank_fr_msegfrp1","MEESCHAERT","MSEGFRP1","base.fr"
"bank_fr_meaefrp1","MEESCHAERT ASSET MANAGEMENT","MEAEFRP1","base.fr"
"bank_fr_merofrm1","MEESCHAERT ROUSSELLE ANJOU-COURTAGE","MEROFRM1","base.fr"
"bank_fr_merofr21","MEESCHAERT ROUSSELLE FUTURES","MEROFR21","base.fr"
"bank_fr_merofrp1","MEESCHAERT, ROUSSELLE S.A.","MEROFRP1","base.fr"
"bank_fr_meerfrp1","MEESCHAERT-ROUSSELLE","MEERFRP1","base.fr"
"bank_fr_icbcfrpp","MEGA INTERNATIONAL COMMERCIAL BANK CO. LTD.","ICBCFRPP","base.fr"
"bank_fr_mltsfrp1","MELANION CAPITAL SAS","MLTSFRP1","base.fr"
"bank_fr_mdesfrp1","MELENDES","MDESFRP1","base.fr"
"bank_fr_menffrp1","MENAFINANCE SA","MENFFRP1","base.fr"
"bank_fr_mebefr21","MERCEDES-BENZ BANK AG, SUCCURSALE EN FRANCE","MEBEFR21","base.fr"
"bank_fr_meckfrp1","MERCURY CAPITAL MARKETS","MECKFRP1","base.fr"
"bank_fr_mrlafrp1","MERRILL LYNCH CAPITAL MARKETS (FRANCE)SAS","MRLAFRP1","base.fr"
"bank_fr_mlibfrp1","MERRILL LYNCH INTERNATIONAL BANK","MLIBFRP1","base.fr"
"bank_fr_mlnafrp1","MERRILL LYNCH INVEST SAS","MLNAFRP1","base.fr"
"bank_fr_mlpefrp1","MERRILL LYNCH PIERCE FENNER SMITH SA","MLPEFRP1","base.fr"
"bank_fr_mlpmfrp1","MERRILL LYNCH PORTFOLIO MANAGERS LTD PARIS BRANCH","MLPMFRP1","base.fr"
"bank_fr_mehgfrp1","MESSIEURS HOTTINGUER ET CIE GESTION","MEHGFRP1","base.fr"
"bank_fr_meucfrp1","METAL SECURITIES","MEUCFRP1","base.fr"
"bank_fr_mrcmfrp1","METIER REGROUPEMENT DE CREDITS MRC","MRCMFRP1","base.fr"
"bank_fr_fitgfrp1","METIER REGROUPEMENT DE CREDITS-MRC SA","FITGFRP1","base.fr"
"bank_fr_metgfrp2","METROPOLE GESTION","METGFRP2","base.fr"
"bank_fr_metgfrp1","METROPOLE GESTION SA","METGFRP1","base.fr"
"bank_fr_mailfrp1","MF GLOBAL SA","MAILFRP1","base.fr"
"bank_fr_mfppfr21","MFP PREVOYANCE","MFPPFR21","base.fr"
"bank_fr_mfprfrp1","MFPREVIE","MFPRFRP1","base.fr"
"bank_fr_mfpsfr21","MFPS","MFPSFR21","base.fr"
"bank_fr_mgpafrp1","MGET PATRIMOINE","MGPAFRP1","base.fr"
"bank_fr_micxfr21","MICHAUX S.A.","MICXFR21","base.fr"
"bank_fr_mihhfr21","MICHELIN","MIHHFR21","base.fr"
"bank_fr_micsfr21","MICOS BANCA S.P.A","MICSFR21","base.fr"
"bank_fr_miclfr21","MIDI CAPITAL","MICLFR21","base.fr"
"bank_fr_biopfrp1","MILARIS SA","BIOPFRP1","base.fr"
"bank_fr_mcppfrp1","MILLENNIUM CAPITAL PARTNERS LLP","MCPPFRP1","base.fr"
"bank_fr_mipufr21","MIP AUDIT","MIPUFR21","base.fr"
"bank_fr_mipcfr21","MIP CONSEIL","MIPCFR21","base.fr"
"bank_fr_mirafrpp","MIRABAUD ET CIE (EUROPE), SUCCURSALE EN FRANCE","MIRAFRPP","base.fr"
"bank_fr_naeifrp1","MIROVA SA","NAEIFRP1","base.fr"
"bank_fr_mhcbfrpp","MIZUHO BANK, LTD. PARIS BRANCH","MHCBFRPP","base.fr"
"bank_fr_mmaffrp1","MMA FINANCE SOCIETE ANONYME","MMAFFRP1","base.fr"
"bank_fr_mmavfr21","MMA VIE S.A.","MMAVFR21","base.fr"
"bank_fr_mmmbfrp1","MMEI MUTUELLE BULL","MMMBFRP1","base.fr"
"bank_fr_mmncfrp1","MNCE MUT NAT CSSE EPARGNE","MMNCFRP1","base.fr"
"bank_fr_moisfr22","MOBILIS","MOISFR22","base.fr"
"bank_fr_cavnfr21","MOBILIS BANQUE","CAVNFR21","base.fr"
"bank_fr_mobqfr22","MOBILIS BANQUE","MOBQFR22","base.fr"
"bank_fr_mobgfr22","MOBILIS GESTION","MOBGFR22","base.fr"
"bank_fr_vizafrpp","MOBSAT","VIZAFRPP","base.fr"
"bank_fr_mosgfrp1","MODELES ET STRATEGIES","MOSGFRP1","base.fr"
"bank_fr_mogsfr21","MONCEAU GENERALE ASSURANCES","MOGSFR21","base.fr"
"bank_fr_moiofrp1","MONCEAU INVESTISSEMENTS IMMOBILIERS","MOIOFRP1","base.fr"
"bank_fr_moibfrp1","MONCEAU INVESTISSEMENTS MOBILIERS","MOIBFRP1","base.fr"
"bank_fr_mrepfrp1","MONCEAU RETRAITE ET EPARGNE","MREPFRP1","base.fr"
"bank_fr_moenfr21","MONDIALE ENTREPRISE","MOENFR21","base.fr"
"bank_fr_morcfrp1","MONDO TV FRANCE S.A.","MORCFRP1","base.fr"
"bank_fr_moagfrp1","MONETA ASSET MANAGEMENT","MOAGFRP1","base.fr"
"bank_fr_mofafrp1","MONEYGRAM FRANCE SA","MOFAFRP1","base.fr"
"bank_fr_motafrp1","MONTAIGNE CAPITAL SAS","MOTAFRP1","base.fr"
"bank_fr_motffrp1","MONTBLEU FINANCE","MOTFFRP1","base.fr"
"bank_fr_montfrpp","MONTE PASCHI BANQUE S.A.","MONTFRPP","base.fr"
"bank_fr_mopvfrp1","MONTE PASCHI INVEST","MOPVFRP1","base.fr"
"bank_fr_mosmfrp1","MONTMARTRE ASSET MANAGEMENT","MOSMFRP1","base.fr"
"bank_fr_mficfrp1","MONTPENSIER FINANCE SAS","MFICFRP1","base.fr"
"bank_fr_mvaefrp1","MONTPENSIER VALEURS EURO","MVAEFRP1","base.fr"
"bank_fr_mofnfrp1","MONTSEGUR FINANCE SAS","MOFNFRP1","base.fr"
"bank_fr_motpfr21","MONTUPET S.A.","MOTPFR21","base.fr"
"bank_fr_moodfrp1","MOODY'S FRANCE SAS","MOODFRP1","base.fr"
"bank_fr_mogifrp1","MOORE GLOBAL INVESTMENTS LIMITED","MOGIFRP1","base.fr"
"bank_fr_msfafrp1","MORGAN STANLEY (FRANCE)","MSFAFRP1","base.fr"
"bank_fr_msfrfrcp","MORGAN STANLEY (FRANCE) S.A.","MSFRFRCP","base.fr"
"bank_fr_msfrfrp1","MORGAN STANLEY (FRANCE) S.A.","MSFRFRP1","base.fr"
"bank_fr_msfrfr31","MORGAN STANLEY AND CO INT'L (PARIS BRANCH)","MSFRFR31","base.fr"
"bank_fr_mrngfr21","MORNING","MRNGFR21","base.fr"
"bank_fr_mosffrp1","MOSAIC FINANCE","MOSFFRP1","base.fr"
"bank_fr_mtlsfr2a","MOTEURS LEROY SOMER","MTLSFR2A","base.fr"
"bank_fr_morufr21","MOULIN ROUGE","MORUFR21","base.fr"
"bank_fr_mpogfr22","MPO INTERNATIONAL","MPOGFR22","base.fr"
"bank_fr_fmtsfrpp","MTS FRANCE SAS","FMTSFRPP","base.fr"
"bank_fr_mulffrp1","MULTIGESTIONS FINANCE","MULFFRP1","base.fr"
"bank_fr_muecfrp1","MUR ECUREUIL","MUECFRP1","base.fr"
"bank_fr_mtacfr21","MUTAC","MTACFR21","base.fr"
"bank_fr_mgarfrp1","MUTAME GARANTIES PARIS","MGARFRP1","base.fr"
"bank_fr_munofr21","MUTAME NORMANDIE","MUNOFR21","base.fr"
"bank_fr_muprfr21","MUTAME PROVENCE","MUPRFR21","base.fr"
"bank_fr_mmblfr21","MUTAME SAVOIE MONT BLANC","MMBLFR21","base.fr"
"bank_fr_mubefr21","MUTAME TERRITOIRE DE BELFORT","MUBEFR21","base.fr"
"bank_fr_mupafrp1","MUTAME UNION PARIS","MUPAFRP1","base.fr"
"bank_fr_muvffr21","MUTAME VAL DE FRANCE","MUVFFR21","base.fr"
"bank_fr_malifr21","MUTAVIE ACTI PLUS TRESORERIE","MALIFR21","base.fr"
"bank_fr_muacfr21","MUTAVIE ACTIPLUS","MUACFR21","base.fr"
"bank_fr_mutgfrp1","MUTLOG GARANTIES","MUTGFRP1","base.fr"
"bank_fr_murefrp1","MUTRE S.A.","MUREFRP1","base.fr"
"bank_fr_mufcfr21","MUTUALITE FRANCAISE CALVADOS","MUFCFR21","base.fr"
"bank_fr_mfdrfr21","MUTUALITE FRANCAISE DOUBS","MFDRFR21","base.fr"
"bank_fr_mufafr21","MUTUALITE FRANCAISE SARTHE","MUFAFR21","base.fr"
"bank_fr_mualfr21","MUTUELLE ALSACE LORRAINE","MUALFR21","base.fr"
"bank_fr_macffr21","MUTUELLE ASSUR. DES COMMERCANTS ET INDUSTRIELS DE FRANCE ET DES CADRES ET SALARIES DE L'IND. ET DU COMM.","MACFFR21","base.fr"
"bank_fr_mubufrp1","MUTUELLE BLEUE","MUBUFRP1","base.fr"
"bank_fr_muelfrp1","MUTUELLE CARCEPT PREV","MUELFRP1","base.fr"
"bank_fr_mucrfrp1","MUTUELLE CENTRALE DE REASSURANCE","MUCRFRP1","base.fr"
"bank_fr_mcmgfr21","MUTUELLE COMPLEMENTAIRE 403","MCMGFR21","base.fr"
"bank_fr_muiffrp1","MUTUELLE D'IVRY (LA FRATERNELLE)","MUIFFRP1","base.fr"
"bank_fr_mfrcfr21","MUTUELLE DE FRANCHE COMTE","MFRCFR21","base.fr"
"bank_fr_mupsfr21","MUTUELLE DE POITIERS ASSURANCES","MUPSFR21","base.fr"
"bank_fr_muaffrp1","MUTUELLE DES ARCHITECTES FRANCAIS","MUAFFRP1","base.fr"
"bank_fr_mcavfr21","MUTUELLE DES CADRES VAUBAN","MCAVFR21","base.fr"
"bank_fr_mucnfr21","MUTUELLE DES CHEMINOTS DE NORMANDIE","MUCNFR21","base.fr"
"bank_fr_mcerfrp1","MUTUELLE DES CLERCS ET EMPLOYES DE NOTAIRE","MCERFRP1","base.fr"
"bank_fr_mooefrp1","MUTUELLE DES OEUVRES CORPORATIVES DE L EDUCATION NATIONALE","MOOEFRP1","base.fr"
"bank_fr_mpirfrp1","MUTUELLE DU PERSONNEL DE L INDUSTRIE ET DE LA RECHERCHE","MPIRFRP1","base.fr"
"bank_fr_mpgsfrp1","MUTUELLE DU PERSONNEL GROUPE SOCIETE GENERALE","MPGSFRP1","base.fr"
"bank_fr_muexfr21","MUTUELLE EXISTENCE","MUEXFR21","base.fr"
"bank_fr_mfmgfrp1","MUTUELLE FAMILIALE MANDAT DE GESTION","MFMGFRP1","base.fr"
"bank_fr_mufsfr21","MUTUELLE FORCE SUD","MUFSFR21","base.fr"
"bank_fr_mglrfrp1","MUTUELLE GARCONS LIMONA RESTAU CIT","MGLRFRP1","base.fr"
"bank_fr_mgpnfrp1","MUTUELLE GENERALE DE LA POLICE","MGPNFRP1","base.fr"
"bank_fr_mugpfrp1","MUTUELLE GENERALE DE LA POLICE","MUGPFRP1","base.fr"
"bank_fr_mugrfrp1","MUTUELLE GENERALE DE PARIS","MUGRFRP1","base.fr"
"bank_fr_mughfrp1","MUTUELLE GENERALE DES CHEMINOTS","MUGHFRP1","base.fr"
"bank_fr_mgetfrp1","MUTUELLE GENERALE ENVIRONNEMENT ET TERRITOIRE","MGETFRP1","base.fr"
"bank_fr_mibmfr21","MUTUELLE IBM","MIBMFR21","base.fr"
"bank_fr_mimpfr21","MUTUELLE INTERP LES MENAGES PREVOYANTS","MIMPFR21","base.fr"
"bank_fr_mroufr21","MUTUELLE LA ROUSSILLONNAISE","MROUFR21","base.fr"
"bank_fr_mumefrp1","MUTUELLE MEDICIS","MUMEFRP1","base.fr"
"bank_fr_mmiufrp1","MUTUELLE MINISTERE INTERIEUR","MMIUFRP1","base.fr"
"bank_fr_mnamfrp1","MUTUELLE NATIONALE AVIATION MARINE","MNAMFRP1","base.fr"
"bank_fr_mnhpfr21","MUTUELLE NATIONALE DES HOSPITALIERS ET DES PERSONNELS DE SANTE","MNHPFR21","base.fr"
"bank_fr_muntfrp1","MUTUELLE NATIONALE TERRITORIALE","MUNTFRP1","base.fr"
"bank_fr_mprefr21","MUTUELLE PREVANOR","MPREFR21","base.fr"
"bank_fr_mpmgfr21","MUTUELLE PREVEA MANDAT DE GESTION","MPMGFR21","base.fr"
"bank_fr_relefr21","MUTUELLE RELEYA","RELEFR21","base.fr"
"bank_fr_musufr21","MUTUELLE SANTEVIE UMT","MUSUFR21","base.fr"
"bank_fr_mmasfrp1","MUTUELLE ST MARTIN ACTION SOCIAL","MMASFRP1","base.fr"
"bank_fr_muunfr21","MUTUELLE UNEO","MUUNFR21","base.fr"
"bank_fr_msavfr21","MUTUELLES SAVOYARDES","MSAVFR21","base.fr"
"bank_fr_mvmvfrp1","MV4 SARL","MVMVFRP1","base.fr"
"bank_fr_gssifrp1","MW GESTION S.A.","GSSIFRP1","base.fr"
"bank_fr_myrifrp1","MYRIA AM","MYRIFRP1","base.fr"
"bank_fr_nfmdfrp1","N F M D A","NFMDFRP1","base.fr"
"bank_fr_nancfrp1","NANO CAPITAL","NANCFRP1","base.fr"
"bank_fr_naamfrp1","NATEXIS ASSET MANAGEMENT","NAAMFRP1","base.fr"
"bank_fr_nasqfrp1","NATEXIS ASSET SQUARE","NASQFRP1","base.fr"
"bank_fr_nsfcfr21","NATIO ENERGIE SOCIETE POUR LE FINANCEMENT DES ECONOMIES D'ENERGIE SOFERGIE SA","NSFCFR21","base.fr"
"bank_fr_natofr21","NATIOBAIL 2 SA","NATOFR21","base.fr"
"bank_fr_natrfr21","NATIOCREDIBAIL SA","NATRFR21","base.fr"
"bank_fr_nsncfr21","NATIOCREDIMURS - SOCIETE EN NOM COLLECTIF","NSNCFR21","base.fr"
"bank_fr_nbokfrpp","NATIONAL BANK OF KUWAIT (INTERNATIONAL) PLC, PARIS BRANCH","NBOKFRPP","base.fr"
"bank_fr_nbpafrpp","NATIONAL BANK OF PAKISTAN","NBPAFRPP","base.fr"
"bank_fr_natxfrpp","NATIXIS","NATXFRPP","base.fr"
"bank_fr_ixirfrpp","NATIXIS (BROKERAGE DIVISION)","IXIRFRPP","base.fr"
"bank_fr_naarfrp1","NATIXIS ARBITRAGE SOCIETE EN NOM COLLECTIF","NAARFRP1","base.fr"
"bank_fr_ixiafrpp","NATIXIS ASSET MANAGEMENT","IXIAFRPP","base.fr"
"bank_fr_namifrp1","NATIXIS ASSET MANAGEMENT FINANCE","NAMIFRP1","base.fr"
"bank_fr_nasufrp1","NATIXIS ASSURANCES SA","NASUFRP1","base.fr"
"bank_fr_nabifrp1","NATIXIS BAIL","NABIFRP1","base.fr"
"bank_fr_cocnfrp1","NATIXIS COFICINE","COCNFRP1","base.fr"
"bank_fr_ncsofrp1","NATIXIS CORPORATE SOLUTIONS LIMITED","NCSOFRP1","base.fr"
"bank_fr_eneefrp1","NATIXIS ENERGECO","ENEEFRP1","base.fr"
"bank_fr_nfacfr21","NATIXIS FACTOR","NFACFR21","base.fr"
"bank_fr_cadffrp1","NATIXIS FINANCEMENT","CADFFRP1","base.fr"
"bank_fr_nafnfrp1","NATIXIS FUNDING","NAFNFRP1","base.fr"
"bank_fr_ngaafrp1","NATIXIS GLOBAL ASSET MANAGEMENT SA","NGAAFRP1","base.fr"
"bank_fr_ngaafrpp","NATIXIS GLOBAL ASSET MANAGEMENT SA","NGAAFRPP","base.fr"
"bank_fr_inrpfrp1","NATIXIS INTEREPARGNE","INRPFRP1","base.fr"
"bank_fr_nalefrp1","NATIXIS LEASE","NALEFRP1","base.fr"
"bank_fr_nmulfrp1","NATIXIS MULTIMANAGER","NMULFRP1","base.fr"
"bank_fr_natffrp1","NATIXIS TRANSPORT FINANCE","NATFFRP1","base.fr"
"bank_fr_ntrxfr2a","NATUREX","NTRXFR2A","base.fr"
"bank_fr_nwsefrp1","NATWEST SELLIER S.A.","NWSEFRP1","base.fr"
"bank_fr_necrfrpp","NCT - NECOTRANS","NECRFRPP","base.fr"
"bank_fr_nfaafr21","NEFILI SAS","NFAAFR21","base.fr"
"bank_fr_neosfrp1","NEOPOST SA","NEOSFRP1","base.fr"
"bank_fr_nestfr21","NESTOR","NESTFR21","base.fr"
"bank_fr_neipfrp1","NET IPO AG","NEIPFRP1","base.fr"
"bank_fr_neysfr21","NETSIZE PAYMENT SAS","NEYSFR21","base.fr"
"bank_fr_netvfrp1","NETVALOR SA","NETVFRP1","base.fr"
"bank_fr_nosgfrp1","NEUFLIZE OBC INVESTISSEMENTS","NOSGFRP1","base.fr"
"bank_fr_nepafrp1","NEUFLIZE PRIVATE ASSET","NEPAFRP1","base.fr"
"bank_fr_nevefrp1","NEUFLIZE VIE","NEVEFRP1","base.fr"
"bank_fr_negefrp1","NEVILLE GESTION SA","NEGEFRP1","base.fr"
"bank_fr_neahfrp1","NEW ALPHA AM","NEAHFRP1","base.fr"
"bank_fr_nehffrp1","NEW HOLLAND FINANCE","NEHFFRP1","base.fr"
"bank_fr_nwnpfrpp","NEW NP S.A.S.","NWNPFRPP","base.fr"
"bank_fr_nfssfr21","NEWCOURT FINANCE FRANCE","NFSSFR21","base.fr"
"bank_fr_negrfrp1","NEWEDGE GROUP","NEGRFRP1","base.fr"
"bank_fr_newgfrp1","NEWEDGE GROUP","NEWGFRP1","base.fr"
"bank_fr_ncgffrp1","NEWSCAPE CAPITAL GROUP FRANCE","NCGFFRP1","base.fr"
"bank_fr_nemmfrp1","NEXAM","NEMMFRP1","base.fr"
"bank_fr_nextfrp1","NEXSTAGE","NEXTFRP1","base.fr"
"bank_fr_nfsefrp1","NFINANCE SECURITIES","NFSEFRP1","base.fr"
"bank_fr_nfddfrp1","NFMDA","NFDDFRP1","base.fr"
"bank_fr_nihnfr21","NIDERA HANDELSCOMPAGNIE BV","NIHNFR21","base.fr"
"bank_fr_nikffrp1","NIKKO FRANCE S.A.","NIKFFRP1","base.fr"
"bank_fr_nombfrp1","NOMURA BOURSE S.A.","NOMBFRP1","base.fr"
"bank_fr_nonnfrp1","NORBAIL LOC","NONNFRP1","base.fr"
"bank_fr_nosofrp1","NORBAIL SOFERGIE SA","NOSOFRP1","base.fr"
"bank_fr_noimfrp1","NORBAIL-IMMOBILIER SA","NOIMFRP1","base.fr"
"bank_fr_ncpafrp1","NORD CAPITAL PARTENAIRES","NCPAFRP1","base.fr"
"bank_fr_noesfrp1","NORD EUROPE ASSURANCES","NOESFRP1","base.fr"
"bank_fr_nofifr21","NORD FINANCEMENT SA","NOFIFR21","base.fr"
"bank_fr_ngdafr21","NORFINANCE GILBERT DUPONT ET ASSOCIES SNC","NGDAFR21","base.fr"
"bank_fr_nofafrp1","NORRSKEN FINANCE SA","NOFAFRP1","base.fr"
"bank_fr_notafr21","NORTIA SAS","NOTAFR21","base.fr"
"bank_fr_norffrp1","NORWICH FINANCE (FRANCE)","NORFFRP1","base.fr"
"bank_fr_ncmgfrp1","NOUVEAU CREDIT MARTINIQUAIS GPE","NCMGFRP1","base.fr"
"bank_fr_novufr21","NOUVELLE VAGUE","NOVUFR21","base.fr"
"bank_fr_novcfrp1","NOVACREDIT SOCIETE ANONYME","NOVCFRP1","base.fr"
"bank_fr_novlfr21","NOVALIA","NOVLFR21","base.fr"
"bank_fr_novtfrp1","NOVALIS TAITBOUT","NOVTFRP1","base.fr"
"bank_fr_novwfrp1","NOVAWATT","NOVWFRP1","base.fr"
"bank_fr_nlnrfrp1","NRD LLC NATURAL RESOURCES RESEARCH","NLNRFRP1","base.fr"
"bank_fr_inwgfrp1","NS INVEST WARGNY GROUP","INWGFRP1","base.fr"
"bank_fr_nscgfr21","NSC GROUPE","NSCGFR21","base.fr"
"bank_fr_oafpfrp1","OAKS FIELD PARTNERS","OAFPFRP1","base.fr"
"bank_fr_obdsfrp1","OBERTHUR DEVELOPPEMENT SAS","OBDSFRP1","base.fr"
"bank_fr_otsafrpp","OBERTHUR TECHNOLOGIES","OTSAFRPP","base.fr"
"bank_fr_ocdefrp1","OCDE","OCDEFRP1","base.fr"
"bank_fr_ocirfrp1","OCIRP","OCIRFRP1","base.fr"
"bank_fr_ocfifrp1","OCTO FINANCES SA","OCFIFRP1","base.fr"
"bank_fr_odeqfrp1","ODB EQUITIES","ODEQFRP1","base.fr"
"bank_fr_odcffrp1","ODDO CORPORATE FINANCE SCA","ODCFFRP1","base.fr"
"bank_fr_oddofrpa","ODDO ET CIE","ODDOFRPA","base.fr"
"bank_fr_oddofrpp","ODDO ET CIE","ODDOFRPP","base.fr"
"bank_fr_oddofrcp","ODDO ET CIE ENTREPRISE D'INVESTISSEMENT","ODDOFRCP","base.fr"
"bank_fr_odpcfrp1","ODDO PINATTON CORPORATE","ODPCFRP1","base.fr"
"bank_fr_odyvfrp1","ODYSSEE VENTURE SAS","ODYVFRP1","base.fr"
"bank_fr_ofamfrp1","OFFICIUM ASSET MANAGEMENT","OFAMFRP1","base.fr"
"bank_fr_ofiffrpp","OFI ASSET MANAGEMENT","OFIFFRPP","base.fr"
"bank_fr_ofiafrp1","OFI ASSET MANAGEMENT SA","OFIAFRP1","base.fr"
"bank_fr_ofiifrp1","OFI INFRAVIA","OFIIFRP1","base.fr"
"bank_fr_ofitfrp1","OFI INSTIT","OFITFRP1","base.fr"
"bank_fr_ofisfrp1","OFI INVESTMENT SOLUTIONS","OFISFRP1","base.fr"
"bank_fr_ofimfrp1","OFI MANDATS SA","OFIMFRP1","base.fr"
"bank_fr_ofmgfrp1","OFI MGA","OFMGFRP1","base.fr"
"bank_fr_ofipfrp1","OFI PATRIMOINE SA","OFIPFRP1","base.fr"
"bank_fr_ofpefrp1","OFI PREMIUM","OFPEFRP1","base.fr"
"bank_fr_ofiefrp1","OFI PRIVATE EQUITY","OFIEFRP1","base.fr"
"bank_fr_ofiqfrp1","OFI QUANT","OFIQFRP1","base.fr"
"bank_fr_ofrifrp1","OFI REIM INVEST","OFRIFRP1","base.fr"
"bank_fr_ofsnfrp1","OFIM SNC","OFSNFRP1","base.fr"
"bank_fr_offufrp1","OFIMA FUTUR","OFFUFRP1","base.fr"
"bank_fr_ofmafrp1","OFIMALLIANCE","OFMAFRP1","base.fr"
"bank_fr_oficfrp1","OFIVALMO CAPITAL","OFICFRP1","base.fr"
"bank_fr_ofgefrp1","OFIVALMO GESTION","OFGEFRP1","base.fr"
"bank_fr_ofnefrp1","OFIVALMO NET EPARGNE","OFNEFRP1","base.fr"
"bank_fr_ofpafrp1","OFIVALMO PALMARES","OFPAFRP1","base.fr"
"bank_fr_ofvifrp1","OFIVALMO PARTENAIRES SA","OFVIFRP1","base.fr"
"bank_fr_ojhofrp1","OJH SOCIETE ANONYME","OJHOFRP1","base.fr"
"bank_fr_opspfr21","OLKY PAYMENT SERVICE PROVIDER","OPSPFR21","base.fr"
"bank_fr_olcgfrp1","OLYMPIA CAPITAL GESTION SOCIETE ANONYME","OLCGFRP1","base.fr"
"bank_fr_omcafrp1","OMNES CAPITAL","OMCAFRP1","base.fr"
"bank_fr_omnafrp1","OMNIANE","OMNAFRP1","base.fr"
"bank_fr_omnffrp1","OMNIFINANCE","OMNFFRP1","base.fr"
"bank_fr_omgpfrp1","OMNIUM DE GESTION PATRIMONIALE SOCIETE ANONYME","OMGPFRP1","base.fr"
"bank_fr_ompffr21","OMNIUM PARTICIPATION ET FINANCEMENT","OMPFFR21","base.fr"
"bank_fr_baccfr22","ONEY BANK","BACCFR22","base.fr"
"bank_fr_opacfr21","OPAC 38","OPACFR21","base.fr"
"bank_fr_otgcfrpp","OPERA TRADING CAPITAL","OTGCFRPP","base.fr"
"bank_fr_opmafrp1","OPHILIAM MANAGEMENT","OPMAFRP1","base.fr"
"bank_fr_opinfrp1","OPPENHEIM INVESTMENT MANAGERS","OPINFRP1","base.fr"
"bank_fr_opprfrp1","OPPORTUNITE SA","OPPRFRP1","base.fr"
"bank_fr_grotfrp1","OPS","GROTFRP1","base.fr"
"bank_fr_opvifrp1","OPTIMUM VIE","OPVIFRP1","base.fr"
"bank_fr_oviefr21","ORADEA VIE SA","OVIEFR21","base.fr"
"bank_fr_gpbafrpp","ORANGE BANK","GPBAFRPP","base.fr"
"bank_fr_ftelfrpp","ORANGE SA","FTELFRPP","base.fr"
"bank_fr_obpsfrp1","ORANGE-BNP PARIBAS SERVICES","OBPSFRP1","base.fr"
"bank_fr_otrafr22","ORCHESTRA PREMAMAN S.A.","OTRAFR22","base.fr"
"bank_fr_orchfrp1","ORCHIDEE","ORCHFRP1","base.fr"
"bank_fr_orpvfr21","OREADE PREVIFRANCE","ORPVFR21","base.fr"
"bank_fr_orprfrp1","OREPA PREVOYANCE","ORPRFRP1","base.fr"
"bank_fr_ocdcfrp1","ORGANISATION DE COOPERATION ET DE DEV ECONOMIQUES","OCDCFRP1","base.fr"
"bank_fr_orfifrp1","ORIENT FINANCE SAS","ORFIFRP1","base.fr"
"bank_fr_orcpfrp1","ORKOS CAPITAL","ORCPFRP1","base.fr"
"bank_fr_ompnfrp1","ORPHELINAT MUT POLICE NAT PREVOYANC","OMPNFRP1","base.fr"
"bank_fr_orssfrp1","ORSAY ASSET MANAGEMENT","ORSSFRP1","base.fr"
"bank_fr_orvafr21","ORVAL SARL","ORVAFR21","base.fr"
"bank_fr_osfmfrp1","OSCA FUND MANAGEMENT","OSFMFRP1","base.fr"
"bank_fr_sorefr21","OSEO GARANTIE REGIONS SA","SOREFR21","base.fr"
"bank_fr_ossafrp1","OSSIAM","OSSAFRP1","base.fr"
"bank_fr_otamfrp1","OTC ASSET MANAGEMENT","OTAMFRP1","base.fr"
"bank_fr_otexfrp1","OTC EXTEND","OTEXFRP1","base.fr"
"bank_fr_vatefrp1","OTCEX SA","VATEFRP1","base.fr"
"bank_fr_otcafrp1","OTEA CAPITAL","OTCAFRP1","base.fr"
"bank_fr_otvsfr22","OTV SA","OTVSFR22","base.fr"
"bank_fr_mougfrp1","OUDART GESTION SA","MOUGFRP1","base.fr"
"bank_fr_oudafrp1","OUDART SA","OUDAFRP1","base.fr"
"bank_fr_ovfffrp1","OVERLORD FRANCE FINANCE","OVFFFRP1","base.fr"
"bank_fr_oewbfrp1","OYENS VAN EEGHEN WHOLESALE BROKERAGE","OEWBFRP1","base.fr"
"bank_fr_papmfrp1","PACIFICA PLACEMENT","PAPMFRP1","base.fr"
"bank_fr_paipfrp1","PAI PARTNERS SAS","PAIPFRP1","base.fr"
"bank_fr_pagtfrp1","PAIERIE GENERALE DU TRESOR","PAGTFRP1","base.fr"
"bank_fr_palnfrp1","PALATINE AM","PALNFRP1","base.fr"
"bank_fr_pasgfrp1","PALATINE ASSET MANAGEMENT","PASGFRP1","base.fr"
"bank_fr_palffr21","PALLADIUM FINANCE","PALFFR21","base.fr"
"bank_fr_parefrpp","PAREL","PAREFRPP","base.fr"
"bank_fr_padgfrp1","PARIBAS DERIVES GARANTIS SNC","PADGFRP1","base.fr"
"bank_fr_parofrp1","PARICOMI SOCIETE ANONYME","PAROFRP1","base.fr"
"bank_fr_parrfrp1","PARIFERGIE SOCIETE ANONYME","PARRFRP1","base.fr"
"bank_fr_pailfrp1","PARILEASE SAS","PAILFRP1","base.fr"
"bank_fr_pifcfrp1","PARIS ILE DE FRANCE CAPITALE","PIFCFRP1","base.fr"
"bank_fr_pmrmfrp1","PARM","PMRMFRP1","base.fr"
"bank_fr_pamffr21","PARNASSE MAIF","PAMFFR21","base.fr"
"bank_fr_pacdfrp1","PARNASSIENNE DE CREDIT SA","PACDFRP1","base.fr"
"bank_fr_paelfrp1","PARTENAIRES ET SELECTION","PAELFRP1","base.fr"
"bank_fr_parufr21","PARUNION SOCIETE CIVILE","PARUFR21","base.fr"
"bank_fr_pcahfr21","PAS DE CALAIS HABITAT","PCAHFR21","base.fr"
"bank_fr_paoofrp1","PASTEL ET ASSOCIES","PAOOFRP1","base.fr"
"bank_fr_pawafrp1","PATRICE WARGNY S.A.","PAWAFRP1","base.fr"
"bank_fr_pagafrp1","PATRIMOINE MANAGEMENT ET ASSOCIES","PAGAFRP1","base.fr"
"bank_fr_patsfrp1","PATRIMOINES ET SELECTIONS SA","PATSFRP1","base.fr"
"bank_fr_pconfr21","PATRIVAL CONSEIL","PCONFR21","base.fr"
"bank_fr_patvfr21","PATRIVAL S.A.","PATVFR21","base.fr"
"bank_fr_paylfrp1","PAYPLUG","PAYLFRP1","base.fr"
"bank_fr_paotfrp1","PAYTOP","PAOTFRP1","base.fr"
"bank_fr_perffr21","PERGAM FINANCE SA","PERFFR21","base.fr"
"bank_fr_prgrfrpp","PERNOD RICARD","PRGRFRPP","base.fr"
"bank_fr_psagfrp1","PEUGEOT SA","PSAGFRP1","base.fr"
"bank_fr_pgagfrp1","PGA GESTION SGA","PGAGFRP1","base.fr"
"bank_fr_phamfrp1","PHILEAS ASSET MANAGEMENT","PHAMFRP1","base.fr"
"bank_fr_dadcfrp1","PHILIPPE (DIDIER) S.A.","DADCFRP1","base.fr"
"bank_fr_phgefrp1","PHILIPPE GESTION","PHGEFRP1","base.fr"
"bank_fr_phpafrp1","PHILIPPE PATRIMOINE SA","PHPAFRP1","base.fr"
"bank_fr_phnefrp1","PHILIPPINE NATIONAL BANK EUROPE","PHNEFRP1","base.fr"
"bank_fr_phllfrp1","PHILLIMORE","PHLLFRP1","base.fr"
"bank_fr_pibifr21","PICARDIE BAIL SA","PIBIFR21","base.fr"
"bank_fr_pictfrpp","PICTET AND CIE (EUROPE) S.A. SUCCURSALE DE PARIS","PICTFRPP","base.fr"
"bank_fr_pfsbfrpp","PIERRE FABRE SA","PFSBFRPP","base.fr"
"bank_fr_pigffrp1","PIM GESTION FRANCE","PIGFFRP1","base.fr"
"bank_fr_pififrp1","PINATTON FINANCE","PIFIFRP1","base.fr"
"bank_fr_picpfrp1","PINK CAPITAL","PICPFRP1","base.fr"
"bank_fr_plntfr21","PLANTUREUX SA","PLNTFR21","base.fr"
"bank_fr_plgefrp1","PLATINIUM GESTION SAS","PLGEFRP1","base.fr"
"bank_fr_pmamfr21","PMAV MUTUELLE","PMAMFR21","base.fr"
"bank_fr_pogefr21","PORTZAMPARC GESTION","POGEFR21","base.fr"
"bank_fr_porzfr21","PORTZAMPARC SOCIETE DE BOURSE SA","PORZFR21","base.fr"
"bank_fr_pownfrp1","POWERNEXT SA","POWNFRP1","base.fr"
"bank_fr_pegsfr21","PRADO EPARGNE GESTION SA","PEGSFR21","base.fr"
"bank_fr_pprefr21","PRADO PREVOYANCE","PPREFR21","base.fr"
"bank_fr_prgafrp1","PRAGMA CAPITAL","PRGAFRP1","base.fr"
"bank_fr_predfrp1","PREDICA","PREDFRP1","base.fr"
"bank_fr_prmufr21","PREMUT","PRMUFR21","base.fr"
"bank_fr_prnsfrp1","PREPAID FINANCIAL SERVICES LTD","PRNSFRP1","base.fr"
"bank_fr_pevifr21","PREPAR VIE","PEVIFR21","base.fr"
"bank_fr_prdufrp1","PRET D'UNION","PRDUFRP1","base.fr"
"bank_fr_prncfrp1","PREVAAL FINANCE","PRNCFRP1","base.fr"
"bank_fr_imadfr21","PREVADIES HARMONIE MUTUELLES","IMADFR21","base.fr"
"bank_fr_pviefrp1","PREVOIR-VIE GROUPE PREVOIR SOCIETE ANONYME","PVIEFRP1","base.fr"
"bank_fr_prpvfr21","PREVOYANCE DU PERSONNEL NAVIGUANT","PRPVFR21","base.fr"
"bank_fr_preofrp1","PREVOYANCE RE SA","PREOFRP1","base.fr"
"bank_fr_prgtfrp1","PRIGEST SA","PRGTFRP1","base.fr"
"bank_fr_prmffrp1","PRIM FINANCE","PRMFFRP1","base.fr"
"bank_fr_prmvfrp1","PRIM'ALTERNATIVE INVESTMENT SA","PRMVFRP1","base.fr"
"bank_fr_prfufrp1","PRIMONIAL FUNDQUEST","PRFUFRP1","base.fr"
"bank_fr_prsvfrp1","PRIMONIAL REAL ESTATE INVESTMENT MANAGEMENT","PRSVFRP1","base.fr"
"bank_fr_prrofr21","PRIORIS SAS","PRROFR21","base.fr"
"bank_fr_prbffrp1","PROBTP FINANCE","PRBFFRP1","base.fr"
"bank_fr_procfrpp","PROCAPITAL SA","PROCFRPP","base.fr"
"bank_fr_scmefr21","PROCIVIS NORD SA","SCMEFR21","base.fr"
"bank_fr_projfrp1","PROJEO SA","PROJFRP1","base.fr"
"bank_fr_prpcfr21","PROMELYS PARTICIPATIONS","PRPCFR21","base.fr"
"bank_fr_prgefrp1","PROMEPAR - GESTION SA","PRGEFRP1","base.fr"
"bank_fr_promfr22","PROMOD SAS","PROMFR22","base.fr"
"bank_fr_prvlfrp1","PROVALOR SA","PRVLFRP1","base.fr"
"bank_fr_prbnfrp1","PRUDENTIAL BACHE INTERNATIONAL LTD","PRBNFRP1","base.fr"
"bank_fr_pubgfrpp","PUBLICIS FINANCE SERVICES SA","PUBGFRPP","base.fr"
"bank_fr_pyivfrp1","PYTHAGORE INVESTISSEMENT","PYIVFRP1","base.fr"
"bank_fr_qnbafrpp","QATAR NATIONAL BANK","QNBAFRPP","base.fr"
"bank_fr_qcfsfrp1","QUAERO CAPITAL (FRANCE) S.A.S.","QCFSFRP1","base.fr"
"bank_fr_qunmfrp1","QUANTAM SA","QUNMFRP1","base.fr"
"bank_fr_quvifr21","QUANTISSIMA VIE","QUVIFR21","base.fr"
"bank_fr_quacfrp1","QUATREM ASSURANCES COLLECTIVES","QUACFRP1","base.fr"
"bank_fr_sifbfrp1","QUILVEST BANQUE PRIVEE SOCIETE ANONYME","SIFBFRP1","base.fr"
"bank_fr_qcfifrp1","QUILVEST COPAGEST FINANCE","QCFIFRP1","base.fr"
"bank_fr_copffrp1","QUILVEST COPAGEST FINANCE SA","COPFFRP1","base.fr"
"bank_fr_qagdfrp1","QUILVEST ET ASSOCIES GESTION D'ACTIFS","QAGDFRP1","base.fr"
"bank_fr_qugpfrp1","QUILVEST GESTION PRIVEE SA","QUGPFRP1","base.fr"
"bank_fr_semufrp1","QUILVEST GESTION SA","SEMUFRP1","base.fr"
"bank_fr_rcapfrp1","R CAPITAL MANAGEMENT","RCAPFRP1","base.fr"
"bank_fr_jaeefrp1","R JAMES","JAEEFRP1","base.fr"
"bank_fr_rabofrpp","RABOBANK PARIS","RABOFRPP","base.fr"
"bank_fr_ragtfrp1","RAPHAEL GESTION","RAGTFRP1","base.fr"
"bank_fr_rjeefrp1","RAYMOND JAMES EURO EQUITIES","RJEEFRP1","base.fr"
"bank_fr_rajifrp1","RAYMOND JAMES INTERNATIONAL SAS","RAJIFRP1","base.fr"
"bank_fr_rbdsfrp1","RBC CAPITAL MARKETS CORPORATION","RBDSFRP1","base.fr"
"bank_fr_roycfrpp","RBC EUROPE LIMITED PARIS BRANCH","ROYCFRPP","base.fr"
"bank_fr_disffrpp","RBC INVESTOR SERVICES BANK FRANCE S.A.","DISFFRPP","base.fr"
"bank_fr_rcinfrpp","RCIBANQUE S.A.","RCINFRPP","base.fr"
"bank_fr_reaffrp1","REASSURANCE ET FINANCES S.A. - REAFIN","REAFFRP1","base.fr"
"bank_fr_rgfpfrp1","RECETTE GENERALE DES FINANCES DE PARIS","RGFPFRP1","base.fr"
"bank_fr_recofrp1","REFCO S.A.","RECOFRP1","base.fr"
"bank_fr_refsfrp1","REFCO SECURITIES SA","REFSFRP1","base.fr"
"bank_fr_regrfrp1","REGARDBTP SA","REGRFRP1","base.fr"
"bank_fr_reaofrp1","REGENT ASSOCIATES LIMITED","REAOFRP1","base.fr"
"bank_fr_ratpfrp1","REGIE AUTONOME DES TRANSPORTS PARIS","RATPFRP1","base.fr"
"bank_fr_rnggfrp1","REGISTRE NATIONAL DE GESTION DES GES","RNGGFRP1","base.fr"
"bank_fr_remyfr21","REMY COINTREAU","REMYFR21","base.fr"
"bank_fr_renofrp1","RENAULT SA","RENOFRP1","base.fr"
"bank_fr_reanfr21","RENE ABALLEA FINANCE SA","REANFR21","base.fr"
"bank_fr_reuufrp1","REPUBLIC AM","REUUFRP1","base.fr"
"bank_fr_rteffrpp","RESEAU DE TRANSPORT D'ELECTRICITE","RTEFFRPP","base.fr"
"bank_fr_reinfrp1","RESTAURATION INVESTISSEMENT","REINFRP1","base.fr"
"bank_fr_reunfr21","REUNIBAIL","REUNFR21","base.fr"
"bank_fr_reuifr21","REUNICA GIE","REUIFR21","base.fr"
"bank_fr_rexsfrpp","REXEL SA","REXSFRPP","base.fr"
"bank_fr_reydfr22","REYDEL AUTOMOTIVE SAS","REYDFR22","base.fr"
"bank_fr_rfsgfrp1","RFS GESTION","RFSGFRP1","base.fr"
"bank_fr_rhgefr21","RHONE GESTION SA","RHGEFR21","base.fr"
"bank_fr_rifrfr21","RIA FRANCE","RIFRFR21","base.fr"
"bank_fr_rfgifrp1","RICHELIEU FINANCE GESTION PRIVEE SA","RFGIFRP1","base.fr"
"bank_fr_rififrp1","RICHELIEU FINANCE SA","RIFIFRP1","base.fr"
"bank_fr_rigsfrp1","RIVAGE GESTION","RIGSFRP1","base.fr"
"bank_fr_riinfrp1","RIVAGE INVESTMENT","RIINFRP1","base.fr"
"bank_fr_rcoufrp1","RIVOLI CONSERV FUND","RCOUFRP1","base.fr"
"bank_fr_rmaafrp1","RMA ASSET MANAGEMENT","RMAAFRP1","base.fr"
"bank_fr_robgfrp1","ROBECO GESTIONS SAS","ROBGFRP1","base.fr"
"bank_fr_rofffrp1","ROBERT FLEMING (FRANCE) S.A.","ROFFFRP1","base.fr"
"bank_fr_robtfr22","ROBERTET SA","ROBTFR22","base.fr"
"bank_fr_rorrfrp1","ROCHE BRUNE AM","RORRFRP1","base.fr"
"bank_fr_rocffrp1","ROCHEFORT - FINANCES","ROCFFRP1","base.fr"
"bank_fr_rcbpfrpp","ROTHSCHILD AND CIE BANQUE","RCBPFRPP","base.fr"
"bank_fr_rcbpfrpr","ROTHSCHILD AND CIE BANQUE","RCBPFRPR","base.fr"
"bank_fr_rcbpfrs1","ROTHSCHILD ET CIE GESTION","RCBPFRS1","base.fr"
"bank_fr_rocgfrp1","ROTHSCHILD ET COMPAGNIE GESTION","ROCGFRP1","base.fr"
"bank_fr_rhdffrp1","ROTHSCHILD HDF INVESTMENT SOLUTIONS","RHDFFRP1","base.fr"
"bank_fr_rouafrp1","ROUVIER ASSOCIES SAS","ROUAFRP1","base.fr"
"bank_fr_rscnfrp1","RSI CAISSE NATIONALE","RSCNFRP1","base.fr"
"bank_fr_rtetfrp1","RTE EDT TRANSPORT","RTETFRP1","base.fr"
"bank_fr_rinefrp1","RUSSELL INVESTMENTS FRANCE","RINEFRP1","base.fr"
"bank_fr_sircfr21","S I R C A M","SIRCFR21","base.fr"
"bank_fr_copofr21","S V COM I COOP","COPOFR21","base.fr"
"bank_fr_scmifrp1","S.A.C.I.C.A.P. AIPAL","SCMIFRP1","base.fr"
"bank_fr_matdfr21","S.A.S. MATMUT DEVELOPPEMENT","MATDFR21","base.fr"
"bank_fr_smanfrp1","S.G.I MANAGEMENT SA","SMANFRP1","base.fr"
"bank_fr_optgfrp1","SA OPTIGESTION","OPTGFRP1","base.fr"
"bank_fr_salafr2m","SAARLB FRANCE","SALAFR2M","base.fr"
"bank_fr_sadpfrp1","SACD PATRIMOINE SCI","SADPFRP1","base.fr"
"bank_fr_scmsfr21","SACICAP AISNE SOMME OISE","SCMSFR21","base.fr"
"bank_fr_sciqfr21","SACICAP AQUITAINE SUD SA","SCIQFR21","base.fr"
"bank_fr_scmgfr21","SACICAP DE LA GIRONDE","SCMGFR21","base.fr"
"bank_fr_sciffr21","SACICAP DE LA MANCHE SA","SCIFFR21","base.fr"
"bank_fr_slcifr21","SACICAP DE LORRAINE SA","SLCIFR21","base.fr"
"bank_fr_scosfr21","SACICAP DE SAVOIE","SCOSFR21","base.fr"
"bank_fr_scbifr21","SACICAP DES PYRENEES SCOP","SCBIFR21","base.fr"
"bank_fr_scoofr21","SACICAP DU CALVADOS","SCOOFR21","base.fr"
"bank_fr_sciofr21","SACICAP TOULOUSE","SCIOFR21","base.fr"
"bank_fr_sclafr21","SACICAP VALLEE DU RHONE","SCLAFR21","base.fr"
"bank_fr_scoafr21","SACICAP VAUCLUSE","SCOAFR21","base.fr"
"bank_fr_sfrafrpp","SAFRAN","SFRAFRPP","base.fr"
"bank_fr_sgfcfrp1","SAGARA FINANCIERE","SGFCFRP1","base.fr"
"bank_fr_sagpfrp1","SAGEP","SAGPFRP1","base.fr"
"bank_fr_sgisfrp1","SAGIS AM","SGISFRP1","base.fr"
"bank_fr_sdoffrp1","SAINT DOMINIQUE FINANCE","SDOFFRP1","base.fr"
"bank_fr_verafrpp","SAINT GOBAIN EMBALLAGE","VERAFRPP","base.fr"
"bank_fr_sartfr21","SAINT MARTIN SARL","SARTFR21","base.fr"
"bank_fr_sairfrp1","SAINTOIN ROULET","SAIRFRP1","base.fr"
"bank_fr_opkgfrp1","SAL OPPENHEIM JR ET CIE KGAA","OPKGFRP1","base.fr"
"bank_fr_slsgfrp1","SALAMANDRE ASSET MANAGEMENT","SLSGFRP1","base.fr"
"bank_fr_sasnfrp1","SALOMON SMITH BARNEY SA","SASNFRP1","base.fr"
"bank_fr_sdfffrp1","SAME DEUTZ-FAHR FINANCE SAS","SDFFFRP1","base.fr"
"bank_fr_samsfr22","SAMSIC","SAMSFR22","base.fr"
"bank_fr_sectfr21","SAMSUNG ELECTRONICS FRANCE S.A.","SECTFR21","base.fr"
"bank_fr_sanofrp1","SANEF CONSTRUCTION","SANOFRP1","base.fr"
"bank_fr_snoffr2l","SANOFI PASTEUR MSD S.N.C.","SNOFFR2L","base.fr"
"bank_fr_saavfrpp","SANOFI SA","SAAVFRPP","base.fr"
"bank_fr_sanxfrpp","SANOFI SA","SANXFRPP","base.fr"
"bank_fr_saalfrp1","SANPAOLO BAIL","SAALFRP1","base.fr"
"bank_fr_samrfrp1","SANPAOLO MUR SNC","SAMRFRP1","base.fr"
"bank_fr_saoffr21","SANTANDER CONSUMER FRANCE","SAOFFR21","base.fr"
"bank_fr_saiifrp1","SAPAR SA","SAIIFRP1","base.fr"
"bank_fr_sprrfrp1","SAPRR","SPRRFRP1","base.fr"
"bank_fr_seemfrp1","SARASIN EXPERTISE AM SA","SEEMFRP1","base.fr"
"bank_fr_srcafr21","SARCA - SOCIETE ANONYME REGIONALE DE CREDIT AUTOMOBILE","SRCAFR21","base.fr"
"bank_fr_sargfrp1","SARGEP","SARGFRP1","base.fr"
"bank_fr_sdbgfrp1","SAS DIAMANT BLEU GESTION","SDBGFRP1","base.fr"
"bank_fr_satefr22","SAS LE CREUSET","SATEFR22","base.fr"
"bank_fr_sasmfrp1","SAS PARVIM","SASMFRP1","base.fr"
"bank_fr_prrsfrp1","SAS PRETS ET SERVICES","PRRSFRP1","base.fr"
"bank_fr_sauffrp1","SAS UNOFI","SAUFFRP1","base.fr"
"bank_fr_satcfrp1","SATURNE CAPITAL","SATCFRP1","base.fr"
"bank_fr_sxfrfrp1","SAXO BANQUE FRANCE","SXFRFRP1","base.fr"
"bank_fr_sbmdfr22","SBM DEVELOPPEMENT","SBMDFR22","base.fr"
"bank_fr_imnvfrp1","SC IMMOBILIERE NATIO VIE","IMNVFRP1","base.fr"
"bank_fr_scfffr21","SCANIA FINANCE FRANCE SAS","SCFFFR21","base.fr"
"bank_fr_scpnfrp1","SCHELCHER PRINCE","SCPNFRP1","base.fr"
"bank_fr_scpffrp1","SCHELCHER PRINCE FINANCE","SCPFFRP1","base.fr"
"bank_fr_scpofrp1","SCHELCHER PRINCE GESTION","SCPOFRP1","base.fr"
"bank_fr_scpgfrp1","SCHELCHER PRINCE PATRIMOINE ET INVESTISSEMENTS","SCPGFRP1","base.fr"
"bank_fr_sceefr21","SCHNEIDER ELECTRIC INDUSTRIES","SCEEFR21","base.fr"
"bank_fr_scfrfrp1","SCHRODERS FRANCE SA","SCFRFRP1","base.fr"
"bank_fr_scclfrp1","SCI CLICHY-BARBUSSE","SCCLFRP1","base.fr"
"bank_fr_snhvfr21","SCI NEUILLY HOTEL DE VILLE","SNHVFR21","base.fr"
"bank_fr_scprfr21","SCI PROMUTA","SCPRFR21","base.fr"
"bank_fr_sscyfrp1","SCI SACCEF CHAMPS ELYSEES","SSCYFRP1","base.fr"
"bank_fr_scskfr21","SCI SOKOBURU","SCSKFR21","base.fr"
"bank_fr_sacmfr21","SCM AUTOMOBILE CYCLE MOTOCYCLE SOMERA","SACMFR21","base.fr"
"bank_fr_scagfr21","SCM CENTRE ATLANTIQUE NEGOCIANTS CEREALE","SCAGFR21","base.fr"
"bank_fr_scshfr21","SCM DES SPECIALISTES PHOX","SCSHFR21","base.fr"
"bank_fr_sirpfrp1","SCM IMMOBILIERE DE LA REGION PARISIENNE","SIRPFRP1","base.fr"
"bank_fr_sndnfrp1","SCM NATIONALE NEGOCE GRAINS","SNDNFRP1","base.fr"
"bank_fr_sngcfr21","SCM NEG GRAINS COLL AQUITAINE MIDI- PYREN.","SNGCFR21","base.fr"
"bank_fr_sngefr21","SCM NEGOC GRIANS 2 ESTUAIRES - BRETAGNE","SNGEFR21","base.fr"
"bank_fr_scgifrpp","SCOR GLOBAL INVESTMENTS","SCGIFRPP","base.fr"
"bank_fr_scrsfrp1","SCOR GLOBAL P ET C","SCRSFRP1","base.fr"
"bank_fr_scrrfrp1","SCOR SE","SCRRFRP1","base.fr"
"bank_fr_sctyfrpp","SCOR SE","SCTYFRPP","base.fr"
"bank_fr_scvifrp1","SCOR VIE","SCVIFRP1","base.fr"
"bank_fr_sopufr22","SCORPIUS S.A.S.","SOPUFR22","base.fr"
"bank_fr_sttefrp1","SDG TRECENTO ASSET MANAGEMENT","STTEFRP1","base.fr"
"bank_fr_sdrlfr21","SDR OUEST SA","SDRLFR21","base.fr"
"bank_fr_sksafr22","SEB S.A.","SKSAFR22","base.fr"
"bank_fr_sedffrp1","SEDEC FINANCE SAS","SEDFFRP1","base.fr"
"bank_fr_ssedfr21","SEDEV (SOCIETE EUROP. DE DEV. DU FINANCEMENT)","SSEDFR21","base.fr"
"bank_fr_sefafr21","SEFIA SAS","SEFAFR21","base.fr"
"bank_fr_secnfrp1","SEGESPAR CNCA","SECNFRP1","base.fr"
"bank_fr_segffrp1","SEGESPAR FINANCE SA","SEGFFRP1","base.fr"
"bank_fr_sebifr21","SELACO BAIL","SEBIFR21","base.fr"
"bank_fr_selefr21","SELECTIBAIL","SELEFR21","base.fr"
"bank_fr_sedrfr21","SELECTION DISC ORGANISATION","SEDRFR21","base.fr"
"bank_fr_seltfrp1","SELFTRADE","SELTFRP1","base.fr"
"bank_fr_sesgfrp1","SELLIER SUCHET GADALA","SESGFRP1","base.fr"
"bank_fr_semyfrp1","SEMYRHAMIS","SEMYFRP1","base.fr"
"bank_fr_snnafrp1","SENAT","SNNAFRP1","base.fr"
"bank_fr_smpifrp1","SEPIAM","SMPIFRP1","base.fr"
"bank_fr_snrmfrp1","SERNAM","SNRMFRP1","base.fr"
"bank_fr_rsasfrp1","SERVICES EN ASSURANCE REASSURANCE ET PREVOYANCE-SARP SAS","RSASFRP1","base.fr"
"bank_fr_sepifrp1","SERVICES ET PRETS IMMOBILIERS","SEPIFRP1","base.fr"
"bank_fr_sepnfrp1","SEVENTURE PARTNERS","SEPNFRP1","base.fr"
"bank_fr_sfilfrpp","SFIL","SFILFRPP","base.fr"
"bank_fr_sgeufrp1","SG EURO C T SA","SGEUFRP1","base.fr"
"bank_fr_opeufrp1","SG OPTION EUROPE SA","OPEUFRP1","base.fr"
"bank_fr_sgsefrp1","SG SECURITIES (PARIS)","SGSEFRP1","base.fr"
"bank_fr_sepafrp1","SG SECURITIES(ADMIN SERV.)","SEPAFRP1","base.fr"
"bank_fr_rplafrp1","SGAM BANQUE","RPLAFRP1","base.fr"
"bank_fr_sgfifr21","SGB FINANCE SA","SGFIFR21","base.fr"
"bank_fr_sgdefrp1","SGD EQUITIES","SGDEFRP1","base.fr"
"bank_fr_sgdpfrpp","SGD PARFUMERIE FRANCE","SGDPFRPP","base.fr"
"bank_fr_derifrp1","SGE DELAHAYE","DERIFRP1","base.fr"
"bank_fr_scavfrp1","SICAV A2G","SCAVFRP1","base.fr"
"bank_fr_silifrp1","SICAVONLINE SA","SILIFRP1","base.fr"
"bank_fr_sobofr21","SICOPIERRE - SOCIETE IMOBILIERE POUR LE COMMERCE ET L'INDUSTRIE SA","SOBOFR21","base.fr"
"bank_fr_sicnfrp1","SICOSNAY","SICNFRP1","base.fr"
"bank_fr_sfssfr21","SIEMENS FINANCIAL SERVICES SAS","SFSSFR21","base.fr"
"bank_fr_sigffr2a","SIG FRANCE SAS","SIGFFR2A","base.fr"
"bank_fr_sigefrp1","SIGMA GESTION","SIGEFRP1","base.fr"
"bank_fr_siapfrp1","SIGMALOG CAPITAL","SIAPFRP1","base.fr"
"bank_fr_sipifrp1","SIIC DE PARIS","SIPIFRP1","base.fr"
"bank_fr_bashfrp1","SIIC DE PARIS 8 EME SA","BASHFRP1","base.fr"
"bank_fr_sofqfr21","SIIC DE PARIS S.A","SOFQFR21","base.fr"
"bank_fr_sirefr21","SIMON MARIE AND CO LTD","SIREFR21","base.fr"
"bank_fr_sifsfrp1","SINOPIA FINANCIAL SERVICES SA","SIFSFRP1","base.fr"
"bank_fr_sisgfr21","SINOPIA SOCIETE DE GESTION","SISGFR21","base.fr"
"bank_fr_sjinfrp1","SJP INVEST","SJINFRP1","base.fr"
"bank_fr_skisfrp1","SKANDIA INVEST","SKISFRP1","base.fr"
"bank_fr_sklffr21","SKANDIA LINK SOCIEDAD ANONIMA DE SEGUROS Y REASEGUROS","SKLFFR21","base.fr"
"bank_fr_slmffrp1","SLAM MANDATS FRANCE","SLMFFRP1","base.fr"
"bank_fr_sliafrp1","SLIBAILAUTOS","SLIAFRP1","base.fr"
"bank_fr_slilfrp1","SLIBAILENERGIE","SLILFRP1","base.fr"
"bank_fr_sliufrp1","SLIBAILMURS","SLIUFRP1","base.fr"
"bank_fr_sliofrp1","SLIBIAL IMMOBILIER","SLIOFRP1","base.fr"
"bank_fr_slmpfrp1","SLIMPAY","SLMPFRP1","base.fr"
"bank_fr_smagfrp1","SMA GESTION","SMAGFRP1","base.fr"
"bank_fr_smbpfrp1","SMABTP","SMBPFRP1","base.fr"
"bank_fr_srarfrp1","SMAR","SRARFRP1","base.fr"
"bank_fr_smbtfrp1","SMAVIE BTP","SMBTFRP1","base.fr"
"bank_fr_smcsfrpp","SMCP SAS","SMCSFRPP","base.fr"
"bank_fr_sninfrpp","SNCF INTERSERVICES","SNINFRPP","base.fr"
"bank_fr_rffrfrpp","SNCF RESEAU","RFFRFRPP","base.fr"
"bank_fr_scicfrp1","SNCF-HABITAT SACICAP","SCICFRP1","base.fr"
"bank_fr_sobifrp1","SOBIAL","SOBIFRP1","base.fr"
"bank_fr_scmdfr21","SOC. ANONYME COOPERATIVE D'INTERET COLLECTIF POUR L'ACCESION A LA PROP. MIDI MEDIT.-SACICAP MIDI MEDITER.","SCMDFR21","base.fr"
"bank_fr_bicsfr21","SOCAMA-BICS","BICSFR21","base.fr"
"bank_fr_soasfrp1","SOCAMAB ASSURANCES SA","SOASFRP1","base.fr"
"bank_fr_sopmfrp1","SOCFIM","SOPMFRP1","base.fr"
"bank_fr_saddfr21","SOCIETE ALSACIENNE DE DEVELOPPEMENT ET D'EXPANSION SDR SADE SOCIETE DE DEVELOPPEMENT REGIONAL D'ALSACE SA","SADDFR21","base.fr"
"bank_fr_scilfr21","SOCIETE AN CREDIT IMMOBILIER DE L'ARMOR ET ARGOAD","SCILFR21","base.fr"
"bank_fr_scidfr21","SOCIETE AN CREDIT IMMOBILIER DOUBS HAUTE SAON FR C SUD","SCIDFR21","base.fr"
"bank_fr_scipfr21","SOCIETE AN CREDIT IMMOBILIER PICARDIE","SCIPFR21","base.fr"
"bank_fr_sioafr21","SOCIETE AN DE CIT IMMOB DE L'AIN","SIOAFR21","base.fr"
"bank_fr_scocfr21","SOCIETE AN DE CREDIT IMMOBILIER CREDIT IMMOBILIER D'ALSACE","SCOCFR21","base.fr"
"bank_fr_scmtfr21","SOCIETE AN DE CREDIT IMMOBILIER CREDIT IMMOBILIER FAMILIAL","SCMTFR21","base.fr"
"bank_fr_scigfr21","SOCIETE AN DE CREDIT IMMOBILIER DE BRETAGNE","SCIGFR21","base.fr"
"bank_fr_scihfr21","SOCIETE AN DE CREDIT IMMOBILIER DE CHAMPAGNE-ARDENNE","SCIHFR21","base.fr"
"bank_fr_scolfr21","SOCIETE AN DE CREDIT IMMOBILIER DE L'ARRDT DE BRIVE","SCOLFR21","base.fr"
"bank_fr_scmvfr21","SOCIETE AN DE CREDIT IMMOBILIER DE LA VENDEE","SCMVFR21","base.fr"
"bank_fr_scomfr21","SOCIETE AN DE CREDIT IMMOBILIER DU MAINE","SCOMFR21","base.fr"
"bank_fr_scbofrp1","SOCIETE AN DE CREDIT IMMOBILIER ET DE TRANSACTIONS","SCBOFRP1","base.fr"
"bank_fr_scohfr21","SOCIETE AN DE CREDIT IMMOBILIER HABITAT GROUPE 36","SCOHFR21","base.fr"
"bank_fr_scmffr21","SOCIETE ANONYME COOPERATIVE D INTERET COLLECTIF POUR L'ACCESSION A LA PROPRIETE DE FRANCHE-COMTE","SCMFFR21","base.fr"
"bank_fr_scmbfr21","SOCIETE ANONYME COOPERATIVE D'INTERET COLLECTIF POUR L'ACCESSION A LA PROP. DE PROVENCE-SACICAP DE PROV.","SCMBFR21","base.fr"
"bank_fr_scizfr21","SOCIETE ANONYME COOPERATIVE D'INTERET COLLECTIF POUR L'ACCESSION A LA PROPIETE FOREZ VELAY","SCIZFR21","base.fr"
"bank_fr_scmhfr21","SOCIETE ANONYME COOPERATIVE D'INTERET COLLECTIF POUR L'ACCESSION A LA PROPRIETE BERRY","SCMHFR21","base.fr"
"bank_fr_scmyfr21","SOCIETE ANONYME COOPERATIVE D'INTERET COLLECTIF POUR L'ACCESSION A LA PROPRIETE DU PUY DE DOME","SCMYFR21","base.fr"
"bank_fr_scovfr21","SOCIETE ANONYME COOPERATIVE D'INTERET COLLECTIF POUR L'ACCESSION A LA PROPRIETE DU VAR","SCOVFR21","base.fr"
"bank_fr_sciufr21","SOCIETE ANONYME COOPERATIVE D'INTERET COLLECTIF POUR L'ACCESSION DE LA PROPRIETE SUD MASSIF CENTRAL","SCIUFR21","base.fr"
"bank_fr_scmnfr21","SOCIETE ANONYME DE COOPERATION IMMOBILIERE DE LA MAYENNE PAR ABREVIATION SACI DE LA MAYENNE","SCMNFR21","base.fr"
"bank_fr_scbnfr21","SOCIETE ANONYME DE CREDIT A L'INDUSTRIE FRANCAISE","SCBNFR21","base.fr"
"bank_fr_scibfr21","SOCIETE ANONYME DE CREDIT IMMOBILIER CI BRETAGNE OUEST","SCIBFR21","base.fr"
"bank_fr_scodfr21","SOCIETE ANONYME DE CREDIT IMMOBILIER D'EURE-ET-LOIR, SA","SCODFR21","base.fr"
"bank_fr_scoufr21","SOCIETE ANONYME DE CREDIT IMMOBILIER DE HAUTE","SCOUFR21","base.fr"
"bank_fr_sciafr21","SOCIETE ANONYME DE CREDIT IMMOBILIER DE L'ANJOU ET DES PREVOYANTS DE L'AVENIR DE CHOLET","SCIAFR21","base.fr"
"bank_fr_sciefr21","SOCIETE ANONYME DE CREDIT IMMOBILIER DE L'EST","SCIEFR21","base.fr"
"bank_fr_scmofr21","SOCIETE ANONYME DE CREDIT IMMOBILIER DE L'ORNE","SCMOFR21","base.fr"
"bank_fr_scivfr21","SOCIETE ANONYME DE CREDIT IMMOBILIER DE LA HAUTE-VIENNE","SCIVFR21","base.fr"
"bank_fr_sconfr21","SOCIETE ANONYME DE CREDIT IMMOBILIER DE SAINT-NAZAIRE ET DE LA REGION DES PAYS DE LOIRE","SCONFR21","base.fr"
"bank_fr_scinfr21","SOCIETE ANONYME DE CREDIT IMMOBILIER DES ENVIRONS DE PARIS","SCINFR21","base.fr"
"bank_fr_srocfr21","SOCIETE ANONYME RURALE ET OUVRIERE DE CREDIT IMMOBILIER DE SAINE-ET-MARNE SA","SROCFR21","base.fr"
"bank_fr_saiafr21","SOCIETE AUX DES IND ALIMENTAIRES","SAIAFR21","base.fr"
"bank_fr_ivstfrp1","SOCIETE AUXILIAIRE D'ETUDES ET D'INVESTISSEMENTS MOBILIERS SA","IVSTFRP1","base.fr"
"bank_fr_sobcfr2p","SOCIETE BIC","SOBCFR2P","base.fr"
"bank_fr_sccifrp1","SOCIETE CENTRALE DE CREDIT IMMOBILIER SA","SCCIFRP1","base.fr"
"bank_fr_sccrfrp1","SOCIETE CENTRALE DE CREDIT MARITIME MUTUEL SCOP","SCCRFRP1","base.fr"
"bank_fr_scfifrp1","SOCIETE CENTRALE POUR LE FINANCEMENT DE L'IMMOBILIER SOCFIM SA","SCFIFRP1","base.fr"
"bank_fr_socvfrp1","SOCIETE CENTRALE PREVOIR SA","SOCVFRP1","base.fr"
"bank_fr_scnofrp1","SOCIETE CIVILE CENTRALE MONCEAU","SCNOFRP1","base.fr"
"bank_fr_scfnfrp1","SOCIETE CIVILE FONCIERE CENTRALE MONCEAU","SCFNFRP1","base.fr"
"bank_fr_scfdfrp1","SOCIETE CIVILE IMMOBILIERE OFI DOMUS","SCFDFRP1","base.fr"
"bank_fr_scaafr21","SOCIETE COOP. AUX. DE MATERIEL COOPAMAT","SCAAFR21","base.fr"
"bank_fr_scaofr21","SOCIETE COOPERATIVE AGRICOLE LE GOUESSANT","SCAOFR21","base.fr"
"bank_fr_screfrp1","SOCIETE COOPERATIVE POUR LA RENOVATION ET L'EQUIPEMENT DU COMMERCE","SCREFRP1","base.fr"
"bank_fr_sodffr21","SOCIETE D'ASSURANCE LE FINISTERE","SODFFR21","base.fr"
"bank_fr_ssacfrp1","SOCIETE D'ASSURANCES DE CONSOLIDATION DES RETRAITES DE L'ASSURANCE SA","SSACFRP1","base.fr"
"bank_fr_sddsfr21","SOCIETE D'ETUDES ET D'ASSISTANCE (SEA)","SDDSFR21","base.fr"
"bank_fr_silefrpp","SOCIETE D'IMPORTATION LECLERC SIPLEC","SILEFRPP","base.fr"
"bank_fr_sbexfrp1","SOCIETE DE BANQUE ET D'EXPANSION SA","SBEXFRP1","base.fr"
"bank_fr_saeufrp1","SOCIETE DE BANQUE PRIVEE","SAEUFRP1","base.fr"
"bank_fr_sbgdfrp1","SOCIETE DE BOURSE GILBERT DUPONT SNC","SBGDFRP1","base.fr"
"bank_fr_sobmfrp1","SOCIETE DE BOURSE J.P. MORGAN S.A.","SOBMFRP1","base.fr"
"bank_fr_sugdfr21","SOCIETE DE CAUTION MUTUELLE DE L'UNION GENERALE DE DISTRIBUTION","SUGDFR21","base.fr"
"bank_fr_satofrp1","SOCIETE DE CAUTION MUTUELLE DES ARTISANS DU TAXIS","SATOFRP1","base.fr"
"bank_fr_setmfrp1","SOCIETE DE CAUTION MUTUELLE DES ENTREPRISES DE TRAVAIL TEMPORAIRE S.C.O.P.","SETMFRP1","base.fr"
"bank_fr_snghfr21","SOCIETE DE CAUTION MUTUELLE DES NEGOCIANTS EN CEREALES/OLEAGINEUX ET PROTEAGINEUX","SNGHFR21","base.fr"
"bank_fr_spiffrp1","SOCIETE DE CAUTION MUTUELLE DES PROFESSIONS IMMOBILIERES ET FONCIERES","SPIFFRP1","base.fr"
"bank_fr_socmfrp1","SOCIETE DE CREDIT A LA CONSOMMATION SA","SOCMFRP1","base.fr"
"bank_fr_scsdfr21","SOCIETE DE CREDIT DES STES D'ASS. CAR MUT","SCSDFR21","base.fr"
"bank_fr_socgfrp1","SOCIETE DE CREDIT POUR LE LOGEMENT SA","SOCGFRP1","base.fr"
"bank_fr_sdrnfr21","SOCIETE DE DEVELOPPEMENT REG DU CENTRE-EST","SDRNFR21","base.fr"
"bank_fr_sdrafr21","SOCIETE DE DEVELOPPEMENT REG DU LANGUEDOC-ROUSSILLON","SDRAFR21","base.fr"
"bank_fr_sdrofr21","SOCIETE DE DEVELOPPEMENT REG DU NORD ET PAS DE CALAIS","SDROFR21","base.fr"
"bank_fr_sdrbfr21","SOCIETE DE DEVELOPPEMENT REGION BRETAGNE - SDR BRETAGNE","SDRBFR21","base.fr"
"bank_fr_soiefr21","SOCIETE DE FINANCEMENT DE LA MEUNERIE SA","SOIEFR21","base.fr"
"bank_fr_sraufrp1","SOCIETE DE FINANCEMENT DES RESEAUX AUTOMOBILES-SOFIRA","SRAUFRP1","base.fr"
"bank_fr_sfpafr21","SOCIETE DE FINANCEMENT PUR LE MASSIF CENTRAL","SFPAFR21","base.fr"
"bank_fr_scuefr21","SOCIETE DE GARANTIE COOPERATIVE ET MUTUELLE DE LA REGION PACA SA","SCUEFR21","base.fr"
"bank_fr_ssgefrp1","SOCIETE DE GARANTIE DES ENTREPRISES LAITIERES AGRICOLES ET ALIMENTAIRES-SOGAL SA","SSGEFRP1","base.fr"
"bank_fr_sogifrp1","SOCIETE DE GERANCE D'INTERETS PRIVES 'SOGIP'","SOGIFRP1","base.fr"
"bank_fr_sgfdfr21","SOCIETE DE GESTION DES FONDS D'INVESTISSEMENT DE BRETAGNE","SGFDFR21","base.fr"
"bank_fr_sgdffrp1","SOCIETE DE GESTION ET D'ETU FIRES FIGEST","SGDFFRP1","base.fr"
"bank_fr_sppcfrp1","SOCIETE DE PROMOTION ET DE PARTICIPATION POUR LA COOPERATION ECONOMIQUE SA","SPPCFRP1","base.fr"
"bank_fr_sacdfrp1","SOCIETE DES AUTEURS ET COMPOSITEURS DRAMATIQUES SOCIETE CIVILE A CAPITAL VARIABLE","SACDFRP1","base.fr"
"bank_fr_saphfr21","SOCIETE DES AUTOROUTES PARIS-RHIN-RHONE SA","SAPHFR21","base.fr"
"bank_fr_sarpfr21","SOCIETE DES AUTOROUTES RHONE ALPES","SARPFR21","base.fr"
"bank_fr_mlapfrp1","SOCIETE DES PRODUITS MARNIER LAPOSTOLLE SA","MLAPFRP1","base.fr"
"bank_fr_fiadfrp1","SOCIETE DES S.D.R. 'FINANSDER' SA","FIADFRP1","base.fr"
"bank_fr_sgsofrp1","SOCIETE DU GRAND SUD OUEST","SGSOFRP1","base.fr"
"bank_fr_xfmnfrp1","SOCIETE DU NOUVEAU MARCHE","XFMNFRP1","base.fr"
"bank_fr_sseefr21","SOCIETE EUROPEENNE DE DEVELOPPEMENT DU FINANCEMENT","SSEEFR21","base.fr"
"bank_fr_sfemfr21","SOCIETE FEDERATIVE EUROP MONETIQ FINT","SFEMFR21","base.fr"
"bank_fr_sfhpfr21","SOCIETE FINANCIER HABITAT PROVENCE-ALPES-COTE AZU R","SFHPFR21","base.fr"
"bank_fr_sfiafrp1","SOCIETE FINANCIERE AUXILIAIRE","SFIAFRP1","base.fr"
"bank_fr_sfcnfr21","SOCIETE FINANCIERE COMIT INTERPROF LOGEMENT","SFCNFR21","base.fr"
"bank_fr_sfbsfrp1","SOCIETE FINANCIERE DE BANQUE - SOFIB SA","SFBSFRP1","base.fr"
"bank_fr_sofdfrp1","SOCIETE FINANCIERE DE DEPOTS ET DE PLACEMENTS - SOFIDEP","SOFDFRP1","base.fr"
"bank_fr_ssfdfr21","SOCIETE FINANCIERE DE DEVELOPPEMENT SA","SSFDFR21","base.fr"
"bank_fr_sfgmfrp1","SOCIETE FINANCIERE DE GRANDS MAGASINS SA","SFGMFRP1","base.fr"
"bank_fr_sfptfr21","SOCIETE FINANCIERE DE PAIEMENTS","SFPTFR21","base.fr"
"bank_fr_sfegfr21","SOCIETE FINANCIERE DES ENTREPRISES DU GARD SOFIGARD SA","SFEGFR21","base.fr"
"bank_fr_soiofr21","SOCIETE FINANCIERE DES MIROIRS","SOIOFR21","base.fr"
"bank_fr_sfpdfr21","SOCIETE FINANCIERE DES PAYS DE L' ADOUR SEBADOUR","SFPDFR21","base.fr"
"bank_fr_sfpmfrp1","SOCIETE FINANCIERE DU PORTE MONNAIE ELECTRONIQUE INTERBANCAIRE","SFPMFRP1","base.fr"
"bank_fr_sofhfr21","SOCIETE FINANCIERE ET CHARBONNIERE","SOFHFR21","base.fr"
"bank_fr_soibfrp1","SOCIETE FINANCIERE ET MOBILIERE SA","SOIBFRP1","base.fr"
"bank_fr_sfgefr21","SOCIETE FINANCIERE GESTION ET EXPLOITATION THIN","SFGEFR21","base.fr"
"bank_fr_sfhbfr21","SOCIETE FINANCIERE HABITAT BRETAGNE ATLANTIQUE","SFHBFR21","base.fr"
"bank_fr_sfiifrp1","SOCIETE FINANCIERE HR","SFIIFRP1","base.fr"
"bank_fr_sofmfrp1","SOCIETE FINANCIERE MALAKOFF - SOFIMAL","SOFMFRP1","base.fr"
"bank_fr_sfedfrp1","SOCIETE FINANCIERE POUR EXPANSION DE LA DISTRIBUTION","SFEDFRP1","base.fr"
"bank_fr_ssfafrp1","SOCIETE FINANCIERE POUR L'ACCESSION A LA PROPRIETE SA","SSFAFRP1","base.fr"
"bank_fr_sffbfrp1","SOCIETE FINANCIERE POUR LE FINANCEMENT DE BUREAUX ET D'USINES-SOFIBUS SA","SFFBFRP1","base.fr"
"bank_fr_sflpfrp1","SOCIETE FINANCIERE POUR LES PAYS D'OUTRE MER","SFLPFRP1","base.fr"
"bank_fr_sfrifr21","SOCIETE FINANCIERE REDEPLOIEMENT INDUSTRIEL","SFRIFR21","base.fr"
"bank_fr_sordfrp1","SOCIETE FRANCAISE DU RADIOTELEPHONE","SORDFRP1","base.fr"
"bank_fr_sgcufr21","SOCIETE GARANTIE COOP MUTUEL DES IND. MET. ELECT.","SGCUFR21","base.fr"
"bank_fr_sgecfrp1","SOCIETE GARANTIE ETUDE DES CREDIT DES C E FRANCE","SGECFRP1","base.fr"
"bank_fr_sgcefrp1","SOCIETE GENERALE - CTM FOR EQUITY","SGCEFRP1","base.fr"
"bank_fr_sgaafr21","SOCIETE GENERALE ASSET MANAGEMENT BANQUE SA","SGAAFR21","base.fr"
"bank_fr_sgcnfrp1","SOCIETE GENERALE CORPORATE INVESTMENT BANKING","SGCNFRP1","base.fr"
"bank_fr_soegfrp1","SOCIETE GENERALE GESTION","SOEGFRP1","base.fr"
"bank_fr_sgdcfrp1","SOCIETE GENERALE POUR LE DEVELOPPEMENT DES OPERATIONS DE CREDIT-BAIL IMMOBILIER SOGEBAIL SA","SGDCFRP1","base.fr"
"bank_fr_sogtfrp1","SOCIETE GENERALE PRIVATE BANKING","SOGTFRP1","base.fr"
"bank_fr_sgrefr21","SOCIETE GENERALE RETIREMENT SERVICES","SGREFR21","base.fr"
"bank_fr_gscffr22","SOCIETE GENERALE SCF","GSCFFR22","base.fr"
"bank_fr_gsfhfrpp","SOCIETE GENERALE SFH","GSFHFRPP","base.fr"
"bank_fr_sgfgfrp1","SOCIETE GESTION FONDS GARANTIE TOM","SGFGFRP1","base.fr"
"bank_fr_scoefr21","SOCIETE HAVRAISE DE CREDIT IMMOBILIER, SOCIETE ANONYME DE CREDIT IMMOBILIER SOCIETE ANONYME","SCOEFR21","base.fr"
"bank_fr_siamfr21","SOCIETE IMMOBILIERE POUR L'AUTOMOBILE ET LA MECANIQUE","SIAMFR21","base.fr"
"bank_fr_cosafr21","SOCIETE IMMOBILIERE POUR LE COMMERCE ET L'INDUSTRIE CORSABAIL SA","COSAFR21","base.fr"
"bank_fr_siaafrp1","SOCIETE INTERPROF ARTISANALE GARANTIE INVEST","SIAAFRP1","base.fr"
"bank_fr_soldfr21","SOCIETE LORRAINE D'EQUIPEMENT","SOLDFR21","base.fr"
"bank_fr_smatfr21","SOCIETE MUTUELLE DES ACCIDENTS CORPORELS (SMAC)","SMATFR21","base.fr"
"bank_fr_sncffrp1","SOCIETE NATIONALE DES CHEMINS DE FER FRANCAIS S N F EURL","SNCFFRP1","base.fr"
"bank_fr_stnmfrp1","SOCIETE NATIONALE IMMOBILIERE","STNMFRP1","base.fr"
"bank_fr_scrafr21","SOCIETE NOUVELLE AGME SA","SCRAFR21","base.fr"
"bank_fr_spfcfrp1","SOCIETE PARISIENNE DE FINANCE ET DE CREDIT","SPFCFRP1","base.fr"
"bank_fr_sopgfrp1","SOCIETE PARISIENNE DE GESTION SA","SOPGFRP1","base.fr"
"bank_fr_sdicfrp1","SOCIETE POUR DEVELOPPEMENT INT DU COMMERCE ET INDUSTRIE","SDICFRP1","base.fr"
"bank_fr_sfapfrp1","SOCIETE POUR FAVOR ACCES A PROP IMMOB","SFAPFRP1","base.fr"
"bank_fr_sdecfr21","SOCIETE POUR LE DEVELOPPEMENT ECO CENTRE ET CENTRE-OUEST","SDECFR21","base.fr"
"bank_fr_sodefr21","SOCIETE POUR LE DEVELOPPEMENT REGIONAL","SODEFR21","base.fr"
"bank_fr_sfdifr21","SOCIETE POUR LE FINANCEMENT DU DEVELOPPEMENT INDUSTRIEL EN POITOU CHARENTES SOFINDI SA","SFDIFR21","base.fr"
"bank_fr_grvgfrp1","SOCIETE PRIVEE D'ETUDES SARL","GRVGFRP1","base.fr"
"bank_fr_spgpfrp1","SOCIETE PRIVEE DE GESTION DE PATRIMOINE","SPGPFRP1","base.fr"
"bank_fr_spgcfrp1","SOCIETE PRIVEE DE GESTION ET DE CONSEIL SA","SPGCFRP1","base.fr"
"bank_fr_sosmfrp1","SOCIETE SOVAC IMMOBILIER","SOSMFRP1","base.fr"
"bank_fr_sosffrp1","SOCIETE SUISSE BANQUE FRANCE","SOSFFRP1","base.fr"
"bank_fr_socdfr21","SOCODEI","SOCDFR21","base.fr"
"bank_fr_sormfr2n","SOCRAM BANQUE","SORMFR2N","base.fr"
"bank_fr_sscafr21","SOCREDPAR SOCIETE DE CREDITS AUX PARTICULIERS","SSCAFR21","base.fr"
"bank_fr_sscufrp1","SOCREPAR STE DE CREDITS AUX PARTICULIERS","SSCUFRP1","base.fr"
"bank_fr_sodlfr21","SODELEM SA","SODLFR21","base.fr"
"bank_fr_sodmfr21","SODERMUR","SODMFR21","base.fr"
"bank_fr_sdalfrpp","SODIAAL INTERNATIONAL","SDALFRPP","base.fr"
"bank_fr_sfaxfrpp","SOFAX BANQUE","SFAXFRPP","base.fr"
"bank_fr_ssgifr21","SOFEFINERG - SOCIETE GALE POUR FINANCEMENT INV ECO ENER","SSGIFR21","base.fr"
"bank_fr_bfcmfr21","SOFEMO","BFCMFR21","base.fr"
"bank_fr_ssfefrp1","SOFERBAIL - SOCIETE FINANCIERE ENERGIE EQUIP PUB ENV","SSFEFRP1","base.fr"
"bank_fr_sofafrp1","SOFICARTE SAS","SOFAFRP1","base.fr"
"bank_fr_sofufr21","SOFIMURS","SOFUFR21","base.fr"
"bank_fr_ssfrfr21","SOFINABAIL - SOCIETE FINANCIERE POUR LE CREDIT BAIL","SSFRFR21","base.fr"
"bank_fr_sofnfr21","SOFINAUTO","SOFNFR21","base.fr"
"bank_fr_sondfr21","SOFINEDIS S.A.","SONDFR21","base.fr"
"bank_fr_sfigfrp1","SOFINGEST - SOCIETE FINANCIERE D'INVESTISSEMENT ET DE GESTION","SFIGFRP1","base.fr"
"bank_fr_sofrfrpp","SOFIPROTEOL","SOFRFRPP","base.fr"
"bank_fr_scfefr21","SOFIRIF - COOPERATIVE FINANCIERE DE LA REGION ILE DE FRANCE SA","SCFEFR21","base.fr"
"bank_fr_sofsfr21","SOFISCOP SOCIETE ANONYME COOPERATIVE A CAPITAL VARIABLE","SOFSFR21","base.fr"
"bank_fr_sostfr21","SOFISCOP SUD EST SA","SOSTFR21","base.fr"
"bank_fr_sofefrp1","SOFRACEM SA","SOFEFRP1","base.fr"
"bank_fr_sorrfr21","SOFRAFI SA","SORRFR21","base.fr"
"bank_fr_ssgffrp1","SOGAFI - SOCIETE DE GARANTIE FINANCIERE","SSGFFRP1","base.fr"
"bank_fr_socifrp1","SOGAMA - CREDIT ASSOCIATIF SA","SOCIFRP1","base.fr"
"bank_fr_afdofr22","SOGEA-SATOM","AFDOFR22","base.fr"
"bank_fr_sogcfr21","SOGECAP SOCIETE ANONYME","SOGCFR21","base.fr"
"bank_fr_sgaifr22","SOGECLAIR S.A.","SGAIFR22","base.fr"
"bank_fr_sogffrp1","SOGEFIMUR SOCIETE ANONYME","SOGFFRP1","base.fr"
"bank_fr_sosnfr21","SOGEFINANCEMENT SAS","SOSNFR21","base.fr"
"bank_fr_sorafr21","SOGELEASE FRANCE SA","SORAFR21","base.fr"
"bank_fr_sogufrp1","SOGESSUR SOCIETE ANONYME","SOGUFRP1","base.fr"
"bank_fr_soglfr21","SOGEVAL SOCIETE ANONYME","SOGLFR21","base.fr"
"bank_fr_sogxfr21","SOGEXIA","SOGXFR21","base.fr"
"bank_fr_soecfr22","SOITEC SA","SOECFR22","base.fr"
"bank_fr_sdirfrpp","SOLAIRE DIRECT","SDIRFRPP","base.fr"
"bank_fr_soexfrp1","SOLENDI EXPANSION SA","SOEXFRP1","base.fr"
"bank_fr_sopjfr21","SOLUCIA PROTECTION JURIDIQUE","SOPJFR21","base.fr"
"bank_fr_solefr21","SOLYCREDIT SA","SOLEFR21","base.fr"
"bank_fr_somgfrp1","SOMANGEST","SOMGFRP1","base.fr"
"bank_fr_soiafr21","SONAUTO FINANCEMENT","SOIAFR21","base.fr"
"bank_fr_sonrfrpp","SONEPAR SA","SONRFRPP","base.fr"
"bank_fr_sppafr21","SOPARFIL SAS","SPPAFR21","base.fr"
"bank_fr_soblfrp1","SOPHIA BAIL SA","SOBLFRP1","base.fr"
"bank_fr_sopifrp1","SOPHIA S.A.","SOPIFRP1","base.fr"
"bank_fr_sterfr22","SOPRA STERIA GROUP","STERFR22","base.fr"
"bank_fr_soprfrp1","SOPROFINANCE","SOPRFRP1","base.fr"
"bank_fr_soicfrp1","SORIA FINANCE SA","SOICFRP1","base.fr"
"bank_fr_fsoufr22","SOUFFLET FINANCES","FSOUFR22","base.fr"
"bank_fr_tellfrp1","SOULIE-TELLIER S.A.","TELLFRP1","base.fr"
"bank_fr_spptfrp1","SPEF TECHNOLOGY","SPPTFRP1","base.fr"
"bank_fr_spasfrp1","SPGP ASSURANCE SARL","SPASFRP1","base.fr"
"bank_fr_spvifr21","SPHERIA VIE","SPVIFR21","base.fr"
"bank_fr_sptifr21","SPIE BATIGNOLLES","SPTIFR21","base.fr"
"bank_fr_siirfrp1","SPIRICA","SIIRFRP1","base.fr"
"bank_fr_srrefr21","SPREAD RESEARCH","SRREFR21","base.fr"
"bank_fr_stdpfrpp","ST DUPONT","STDPFRPP","base.fr"
"bank_fr_mggnfrp1","ST MAGENTA","MGGNFRP1","base.fr"
"bank_fr_stsofrp1","STANDARD AND POOR'S CREDIT MARKET SERVICES FRANCE SAS","STSOFRP1","base.fr"
"bank_fr_stlefrp1","STAR LEASE SA","STLEFRP1","base.fr"
"bank_fr_sbinfrpp","STATE BANK OF INDIA, PARIS BRANCH","SBINFRPP","base.fr"
"bank_fr_sbosfrp1","STATE STREET BANQUE SA","SBOSFRP1","base.fr"
"bank_fr_sbosfrp2","STATE STREET BANQUE SA PARIS","SBOSFRP2","base.fr"
"bank_fr_ssgvfrp1","STATE STREET GLOBAL ADVISORS FRANCE SA","SSGVFRP1","base.fr"
"bank_fr_sdgmfr21","STE D'AMENAGEMENT DE GESTION DU MIN DE STRASBOURG","SDGMFR21","base.fr"
"bank_fr_scrmfr21","STE DU CANAL DE PROVENCE ET D'AMENAGEMENT REG PROV","SCRMFR21","base.fr"
"bank_fr_stfefr21","STE FIRE DE LA NEF","STFEFR21","base.fr"
"bank_fr_stfffrp1","STE FIRE DES S D R FINANSDER","STFFFRP1","base.fr"
"bank_fr_sfpofrp1","STE FIRE DU PORTE MONNAIE ELECTR INTERBA","SFPOFRP1","base.fr"
"bank_fr_stfrfrp1","STE FIRE HR","STFRFRP1","base.fr"
"bank_fr_sgftfrp1","STE GESTION FONDS GARANTIE OUTRE MER","SGFTFRP1","base.fr"
"bank_fr_shaufr21","STE HOSPITALIERE D ASSURANCES MUTUELLES","SHAUFR21","base.fr"
"bank_fr_scssfr22","STEELCASE SAS","SCSSFR22","base.fr"
"bank_fr_tfexfrpp","STEF TFE","TFEXFRPP","base.fr"
"bank_fr_sttsfrp1","STET SAS","STTSFRP1","base.fr"
"bank_fr_stfcfrp1","STRATEGE FINANCE","STFCFRP1","base.fr"
"bank_fr_stfafrp1","STRATEGIE FINANCIERE SARL","STFAFRP1","base.fr"
"bank_fr_sucsfrp1","SUCRES ET DENREES S.A","SUCSFRP1","base.fr"
"bank_fr_swrefrp1","SUISSE DE REASSURANCES (FRANCE)","SWREFRP1","base.fr"
"bank_fr_sulyfrp1","SULLY AM","SULYFRP1","base.fr"
"bank_fr_smbcfrpp","SUMITOMO MITSUI BANKING CORPORATION EUROPE LIMITED","SMBCFRPP","base.fr"
"bank_fr_suagfrp1","SUNNY ASSET MANAGEMENT","SUAGFRP1","base.fr"
"bank_fr_suamfrp1","SUPERFUND ASSET MANAGEMENT","SUAMFRP1","base.fr"
"bank_fr_sufdfr21","SURAVENIR FIDELITY","SUFDFR21","base.fr"
"bank_fr_survfr21","SURAVENIR SOCIETE ANONYME","SURVFR21","base.fr"
"bank_fr_svitfrp1","SV INTERNATIONAL SA","SVITFRP1","base.fr"
"bank_fr_handfrpp","SVENSKA HANDELSBANKEN AB","HANDFRPP","base.fr"
"bank_fr_swcafrp1","SWAN CAPITAL MANAGEMENT SA","SWCAFRP1","base.fr"
"bank_fr_swaafrp1","SWELL ASSET MANAGEMENT","SWAAFRP1","base.fr"
"bank_fr_slamfrpp","SWISS LIFE ASSET MANAGEMENT (FRANCE)","SLAMFRPP","base.fr"
"bank_fr_slarfrp1","SWISS LIFE ASSURANCE ET PATRIMOINE","SLARFRP1","base.fr"
"bank_fr_swilfrpp","SWISSLIFE BANQUE PRIVEE","SWILFRPP","base.fr"
"bank_fr_swgpfrp1","SWISSLIFE GESTION PRIVEE","SWGPFRP1","base.fr"
"bank_fr_syaafrp1","SYCOMORE ASSET MANAGEMENT SA","SYAAFRP1","base.fr"
"bank_fr_sygpfrp1","SYCOMORE GESTION PRIVEE SGP SA","SYGPFRP1","base.fr"
"bank_fr_sygmfrp1","SYGMA BANQUE SA","SYGMFRP1","base.fr"
"bank_fr_syfsfr21","SYGMA FINANCE SNC","SYFSFR21","base.fr"
"bank_fr_synafrp1","SYNALGEST SA","SYNAFRP1","base.fr"
"bank_fr_sniufrp1","SYND NAL INDUSTRIE NUTRITION ANIMAL","SNIUFRP1","base.fr"
"bank_fr_snpmfr21","SYNDICAT NATIONAL PERSONNELS MINISTERE AGRICULTURE","SNPMFR21","base.fr"
"bank_fr_syfgfr21","SYNERGIE FINANCE GESTION","SYFGFR21","base.fr"
"bank_fr_taclfrp1","TAILOR CAPITAL","TACLFRP1","base.fr"
"bank_fr_tagefrp1","TALENCE GESTION","TAGEFRP1","base.fr"
"bank_fr_taanfrp1","TARGET ASSET MANAGERS","TAANFRP1","base.fr"
"bank_fr_tagpfr21","TAURUS GESTION PRIVEE","TAGPFR21","base.fr"
"bank_fr_tcdafrp1","TCM DROITS AUDIOVISUELS","TCDAFRP1","base.fr"
"bank_fr_tccafrp1","TCR CAPITAL SAS","TCCAFRP1","base.fr"
"bank_fr_tchifrpp","TECHNICOLOR","TCHIFRPP","base.fr"
"bank_fr_tpsafrpp","TELEPERFORMANCE","TPSAFRPP","base.fr"
"bank_fr_telvfrp1","TELEVIE","TELVFRP1","base.fr"
"bank_fr_tefrfr21","TELEVISION FRANCAISE 1 SA","TEFRFR21","base.fr"
"bank_fr_tfrsfrp1","TEMPO FRANCE SAS","TFRSFRP1","base.fr"
"bank_fr_temrfr21","TEMPRO SA","TEMRFR21","base.fr"
"bank_fr_terefr22","TEREOS","TEREFR22","base.fr"
"bank_fr_ttssfr21","TETHYS SAS","TTSSFR21","base.fr"
"bank_fr_tfixfr21","TF1 SA","TFIXFR21","base.fr"
"bank_fr_tcsffrpp","THALES SA","TCSFFRPP","base.fr"
"bank_fr_eibcfrpp","THE EXPORT-IMPORT BANK OF CHINA, PARIS BRANCH","EIBCFRPP","base.fr"
"bank_fr_exikfrp1","THE EXPORT-IMPORT BANK OF KOREA","EXIKFRP1","base.fr"
"bank_fr_goirfrp1","THE GOVERNOR AND CY OF THE BANK IRELAND","GOIRFRP1","base.fr"
"bank_fr_kglmfrp1","THE KYTE GROUP LIMITED","KGLMFRP1","base.fr"
"bank_fr_themfrp1","THEMA","THEMFRP1","base.fr"
"bank_fr_theifrp1","THEMIS","THEIFRP1","base.fr"
"bank_fr_thgefrp1","THIRIET GESTION","THGEFRP1","base.fr"
"bank_fr_tiiafrp1","TIKEHAU INVEST MANAGEMENT","TIIAFRP1","base.fr"
"bank_fr_tocffrp1","TOCQUEVILLE FINANCE SA","TOCFFRP1","base.fr"
"bank_fr_tokhfr21","TOKHEIM GROUP SERVICES","TOKHFR21","base.fr"
"bank_fr_topgfrp1","TOP GESTION SARL","TOPGFRP1","base.fr"
"bank_fr_totdfrp1","TOP TRADES","TOTDFRP1","base.fr"
"bank_fr_ttrefrpp","TOTAL TREASURY","TTREFRPP","base.fr"
"bank_fr_toexfrp1","TOURISME EXPANSION","TOEXFRP1","base.fr"
"bank_fr_tkgtfr21","TOYOTA KREDITBANK GMBH","TKGTFR21","base.fr"
"bank_fr_tmhffr21","TOYOTA MATERIAL HANDLING FINANCIAL","TMHFFR21","base.fr"
"bank_fr_trdpfrp1","TRADINGPAD","TRDPFRP1","base.fr"
"bank_fr_treffrp2","TRADITION SECURITIES AND FUTURES","TREFFRP2","base.fr"
"bank_fr_treffrp1","TRADITION SECURITIES AND FUTURES GIE","TREFFRP1","base.fr"
"bank_fr_tserfr22","TRANSALLIANCE SERVICE","TSERFR22","base.fr"
"bank_fr_tcfffrp1","TRANSAMERICA COMMERCIAL FINANCE FRANCE","TCFFFRP1","base.fr"
"bank_fr_tslffrpp","TRANSATEL SA","TSLFFRPP","base.fr"
"bank_fr_trfnfrp1","TRANSATLANTIQUE FINANCE","TRFNFRP1","base.fr"
"bank_fr_trabfrp1","TRANSBANQUE","TRABFRP1","base.fr"
"bank_fr_tdevfrp1","TRANSDEV","TDEVFRP1","base.fr"
"bank_fr_trfifr21","TRANSOLVER FINANCE SA","TRFIFR21","base.fr"
"bank_fr_trsvfr21","TRANSVALOR SA","TRSVFR21","base.fr"
"bank_fr_trpsfrp1","TRAVELEX PARIS SAS","TRPSFRP1","base.fr"
"bank_fr_boxxfrpp","TREASURYXPRESS","BOXXFRPP","base.fr"
"bank_fr_treifrp1","TREILHARD GESTION S.N.C.","TREIFRP1","base.fr"
"bank_fr_trpufrp1","TRESOR PUBLIC","TRPUFRP1","base.fr"
"bank_fr_trglfrp1","TRESORERIE GENERALE POUR L'ETRANGER","TRGLFRP1","base.fr"
"bank_fr_tregfrpp","TREVES SA","TREGFRPP","base.fr"
"bank_fr_tscgfrpp","TREVES TEXTILES AND SEAT COMPONENTS","TSCGFRPP","base.fr"
"bank_fr_trgafr21","TRICOIRE GESTION ET ASSOCIES","TRGAFR21","base.fr"
"bank_fr_trtifrp1","TRUFFLE CAPITAL","TRTIFRP1","base.fr"
"bank_fr_tfinfrp1","TRUSTEAM FINANCE SCA","TFINFRP1","base.fr"
"bank_fr_vitdfrp1","TSAF OTC SA","VITDFRP1","base.fr"
"bank_fr_tpcmfrp1","TULLET PREBON CAPITAL MARKETS FRANCE","TPCMFRP1","base.fr"
"bank_fr_tupffrp1","TULLETT PREBON FRANCE","TUPFFRP1","base.fr"
"bank_fr_utubfrpp","TUNISIAN FOREIGN BANK","UTUBFRPP","base.fr"
"bank_fr_tcpafrp1","TURENNE CAPITAL PARTENAIRES SA","TCPAFRP1","base.fr"
"bank_fr_tuamfrp1","TURGOT ASSET MANAGEMENT","TUAMFRP1","base.fr"
"bank_fr_tugefr21","TURGOT GESTION","TUGEFR21","base.fr"
"bank_fr_emlafrp1","TURKIYE EMLAK BANKASI","EMLAFRP1","base.fr"
"bank_fr_twfcfrp1","TWENTY FIRST CAPITAL","TWFCFRP1","base.fr"
"bank_fr_uedpfr21","U ETABLISSEMENT DE PAIEMENT","UEDPFR21","base.fr"
"bank_fr_uafufrp1","UAF","UAFUFRP1","base.fr"
"bank_fr_uboffr22","UBISOFT ENTERTAINMENT S.A.","UBOFFR22","base.fr"
"bank_fr_ubswfrpp","UBS (FRANCE) SA","UBSWFRPP","base.fr"
"bank_fr_ubswfr31","UBS SECURITIES FRANCE SA","UBSWFR31","base.fr"
"bank_fr_ucimfr21","UCABAIL IMMOBILIER","UCIMFR21","base.fr"
"bank_fr_ucbifrp1","UCB - BAIL SAS","UCBIFRP1","base.fr"
"bank_fr_ucenfrp1","UCB ENTREPRISES SAS","UCENFRP1","base.fr"
"bank_fr_uclifrp1","UCB LOCABAIL IMMOBILIER SAS","UCLIFRP1","base.fr"
"bank_fr_uflofrp1","UFB LOCABAIL","UFLOFRP1","base.fr"
"bank_fr_ufalfrp1","UFG ALTERAM","UFALFRP1","base.fr"
"bank_fr_ufcofrp1","UFG COURTAGE","UFCOFRP1","base.fr"
"bank_fr_ufpefrp1","UFG PRIVATE EQUITY","UFPEFRP1","base.fr"
"bank_fr_uremfrp1","UFG REAL ESTATE MANAGERS","UREMFRP1","base.fr"
"bank_fr_ufsnfr21","UFINVEST SNC","UFSNFR21","base.fr"
"bank_fr_ugalfr21","UGECAM ALSACE","UGALFR21","base.fr"
"bank_fr_ubfcfr21","UGECAM BOURGOGNE FRANCHE COMTE","UBFCFR21","base.fr"
"bank_fr_ugrcfrp1","UGRC","UGRCFRP1","base.fr"
"bank_fr_ugrrfrp1","UGRR-ISICA","UGRRFRP1","base.fr"
"bank_fr_uimmfrp1","UIMM","UIMMFRP1","base.fr"
"bank_fr_uufsfrp1","UIS UNI POUR FINANCEMENT IMMOBILIER SOCIETE-GE CAPITAL UIS","UUFSFRP1","base.fr"
"bank_fr_ujaafrpp","UJA","UJAAFRPP","base.fr"
"bank_fr_ulysfrp1","ULYSSE ASSURANCES","ULYSFRP1","base.fr"
"bank_fr_ulypfrp1","ULYSSE PATRIMOINE","ULYPFRP1","base.fr"
"bank_fr_umsufrp1","UMS","UMSUFRP1","base.fr"
"bank_fr_uccifrp1","UNCIA AM","UCCIFRP1","base.fr"
"bank_fr_unesfrpp","UNESCO (UNITED NATIONS EDUCATIONAL, SCIENTIFIC AND CULTURAL ORGANIZATION)","UNESFRPP","base.fr"
"bank_fr_ungifr21","UNI GESTION SARL","UNGIFR21","base.fr"
"bank_fr_unbifrp1","UNIBAIL","UNBIFRP1","base.fr"
"bank_fr_unbdfrpp","UNIBAIL-RODAMCO","UNBDFRPP","base.fr"
"bank_fr_uniqfrp1","UNIBANQUE","UNIQFRP1","base.fr"
"bank_fr_uncifr21","UNICEFI 63","UNCIFR21","base.fr"
"bank_fr_unmbfrp1","UNICREDIT BANCA MOBILIARE","UNMBFRP1","base.fr"
"bank_fr_uncrfrpp","UNICREDITO ITALIANO SPA - SUCCURSALE DE PARIS","UNCRFRPP","base.fr"
"bank_fr_uufefrp1","UNIFERGIE","UUFEFRP1","base.fr"
"bank_fr_uamffrp1","UNIGESTION ASSET MANAGEMENT (FRANCE) SA","UAMFFRP1","base.fr"
"bank_fr_ungrfrp1","UNIGRAINS","UNGRFRP1","base.fr"
"bank_fr_unmufr21","UNILIA MUTUELLE","UNMUFR21","base.fr"
"bank_fr_umrmfr21","UNILIA MUTUELLES REALISATIONS MUTUALISTES","UMRMFR21","base.fr"
"bank_fr_unmmfr21","UNIMAT","UNMMFR21","base.fr"
"bank_fr_unfafr21","UNIMIE FINANCEMENTS","UNFAFR21","base.fr"
"bank_fr_ubgifrp1","UNION BANCAIRE GESTION INSTITUTIONNELLE (FRANCE) SA","UBGIFRP1","base.fr"
"bank_fr_ubaffrpp","UNION DE BANQUES ARABES ET FRANCAISES","UBAFFRPP","base.fr"
"bank_fr_unctfrp1","UNION DE CREDIT POUR LE BATIMENT","UNCTFRP1","base.fr"
"bank_fr_ugdpfr21","UNION DES GROUPEMENTS D'ACHATS PUBLICS (UGAP)","UGDPFR21","base.fr"
"bank_fr_umrefr21","UNION DES MUTUELLE DE LA REUNION","UMREFR21","base.fr"
"bank_fr_unmsfr21","UNION DES MUTUELLES SANTE 63","UNMSFR21","base.fr"
"bank_fr_ufiffrp1","UNION FINANCIERE DE FRANCE","UFIFFRP1","base.fr"
"bank_fr_ufdcfrp1","UNION FINANCIERE POUR LE DEVELOPPEMENT DE L'ECONOMIE CEREALIERE UNIGRAINS","UFDCFRP1","base.fr"
"bank_fr_uicrfrp1","UNION INDUSTRIELLE DE CREDIT","UICRFRP1","base.fr"
"bank_fr_uicffrp1","UNION INDUSTRIELLE ET COMMERCIALE DE FRANCE UNINCOFRA SA","UICFFRP1","base.fr"
"bank_fr_umurfrp1","UNION MUTUALISTE RETRAITE R1","UMURFRP1","base.fr"
"bank_fr_umamfrp1","UNION MUTUELLES ASSURANCES MONCEAU","UMAMFRP1","base.fr"
"bank_fr_unfcfrp1","UNION NOTARIALE FINANCIERE DE CREDIT-UNOFI-CREDIT SA","UNFCFRP1","base.fr"
"bank_fr_unsnfrp1","UNION SOLIDARITE UNIVERSITAIRE","UNSNFRP1","base.fr"
"bank_fr_unppfrp1","UNIPREVOYANCE","UNPPFRP1","base.fr"
"bank_fr_unbufr22","UNITED BISCUITS","UNBUFR22","base.fr"
"bank_fr_usdsfrp1","UNITED STATES DEPARTMENT OF STATE","USDSFRP1","base.fr"
"bank_fr_unarfrp1","UNOFI ASSURANCES","UNARFRP1","base.fr"
"bank_fr_utsifrp1","UTSIT","UTSIFRP1","base.fr"
"bank_fr_uzgefrp1","UZES GESTION","UZGEFRP1","base.fr"
"bank_fr_vaeofrpp","VALEO SA","VAEOFRPP","base.fr"
"bank_fr_valrfrp1","VALINTER 11 S.A.","VALRFRP1","base.fr"
"bank_fr_valofr21","VALLOUREC","VALOFR21","base.fr"
"bank_fr_vlrcfr22","VALLOUREC TUBES","VLRCFR22","base.fr"
"bank_fr_mofefrp1","VAN DER MOOLEN FINANCIAL SERVICES SAS","MOFEFRP1","base.fr"
"bank_fr_vacrfrp1","VARENNE CAPITAL PARTNERS","VACRFRP1","base.fr"
"bank_fr_vatcfrp1","VATEL CAPITAL","VATCFRP1","base.fr"
"bank_fr_velefrp1","VENDOME LEASE SAS","VELEFRP1","base.fr"
"bank_fr_veoefrpp","VEOLIA ENVIRONNEMENT SA","VEOEFRPP","base.fr"
"bank_fr_veagfrp1","VERMEER ASSET MANAGEMENT","VEAGFRP1","base.fr"
"bank_fr_vecsfrp1","VERMEER CAPITAL PARTNERS","VECSFRP1","base.fr"
"bank_fr_veenfrp1","VERNIER ENTREPRENDRE SAS","VEENFRP1","base.fr"
"bank_fr_vetofr2l","VETOQUINOL SA","VETOFR2L","base.fr"
"bank_fr_vffffrp1","VFS FINANCE FRANCE SAS","VFFFFRP1","base.fr"
"bank_fr_vibifrp1","VIA BAIL","VIBIFRP1","base.fr"
"bank_fr_vibofrp1","VIA BOURSE S.A.","VIBOFRP1","base.fr"
"bank_fr_viapfr21","VIAPAY.FR","VIAPFR21","base.fr"
"bank_fr_vccafr21","VICAT","VCCAFR21","base.fr"
"bank_fr_viplfr21","VIE PLUS","VIPLFR21","base.fr"
"bank_fr_vcomfrpp","VILMORIN ET CIE","VCOMFRPP","base.fr"
"bank_fr_vibpfr21","VINCENT BRAC DE LA PERRIERE S.A.","VIBPFR21","base.fr"
"bank_fr_vincfrpp","VINCI","VINCFRPP","base.fr"
"bank_fr_vcfrfrpp","VINCI CONSTRUCTION FRANCE","VCFRFRPP","base.fr"
"bank_fr_vcgpfr22","VINCI CONSTRUCTION GRANDS PROJETS","VCGPFR22","base.fr"
"bank_fr_viiefrp1","VITALIA VIE","VIIEFRP1","base.fr"
"bank_fr_fiflfrpp","VIVARTE","FIFLFRPP","base.fr"
"bank_fr_vivefrpp","VIVENDI","VIVEFRPP","base.fr"
"bank_fr_vivmfr21","VIVERIS MANAGEMENT SAS","VIVMFR21","base.fr"
"bank_fr_vivrfr21","VIVERIS REIM","VIVRFR21","base.fr"
"bank_fr_viinfr21","VIVIENNE INVESTISSEMENT","VIINFR21","base.fr"
"bank_fr_vicffr21","VIZILLE CAPITAL FINANCE SA","VICFFR21","base.fr"
"bank_fr_vowafr21","VOLKSWAGEN BANK","VOWAFR21","base.fr"
"bank_fr_vofifr21","VOLKSWAGEN FINANCE SA","VOFIFR21","base.fr"
"bank_fr_vltafrpp","VOLTALIA S.A.","VLTAFRPP","base.fr"
"bank_fr_vafffr21","VOLVO AUTOMOBILES FINANCE FRANCE SAS","VAFFFR21","base.fr"
"bank_fr_vtfffr21","VOLVO TRUCK FINANCE FRANCE","VTFFFR21","base.fr"
"bank_fr_fgesfrp1","VP FINANCE GESTION SAS","FGESFRP1","base.fr"
"bank_fr_fnaafrp1","VP FINANCE SA","FNAAFRP1","base.fr"
"bank_fr_eurofrpp","VTB BANK (FRANCE) S.A.","EUROFRPP","base.fr"
"bank_fr_fnnafrp1","W FINANCE","FNNAFRP1","base.fr"
"bank_fr_figifrp1","W FINANCE GESTION","FIGIFRP1","base.fr"
"bank_fr_whawfr21","W-HA","WHAWFR21","base.fr"
"bank_fr_wterfr22","WATERS S.A.S","WTERFR22","base.fr"
"bank_fr_wavcfr21","WAVECOM","WAVCFR21","base.fr"
"bank_fr_fdfrfr22","WEBHELP PAYMENT SERVICES FRANCE SAS","FDFRFR22","base.fr"
"bank_fr_wendfrp1","WENDEL INVESTISSEMENT","WENDFRP1","base.fr"
"bank_fr_wendfrpp","WENDEL SE","WENDFRPP","base.fr"
"bank_fr_wuigfrp1","WESTERN UNION INTERNATIONAL BANK GMBH-FRENCH BRANCH","WUIGFRP1","base.fr"
"bank_fr_winmfrp1","WINDERMERE","WINMFRP1","base.fr"
"bank_fr_wndhfrp1","WINDHURST INDUSTRIES SAS","WNDHFRP1","base.fr"
"bank_fr_wisafrp1","WISEAM","WISAFRP1","base.fr"
"bank_fr_witafr21","WITAM SARL","WITAFR21","base.fr"
"bank_fr_wofgfrp1","WORMSER FRERES GESTION","WOFGFRP1","base.fr"
"bank_fr_aprqfrp1","XANGE PRIVATE EQUITY","APRQFRP1","base.fr"
"bank_fr_yamffrp1","YAMAICHI FRANCE S.A.","YAMFFRP1","base.fr"
"bank_fr_ycapfrp1","YCAP","YCAPFRP1","base.fr"
"bank_fr_ymgifrpp","YMAGIS S.A.","YMGIFRPP","base.fr"
"bank_fr_yomofrp1","YOMONI","YOMOFRP1","base.fr"
"bank_fr_ypiifr2n","YPI","YPIIFR2N","base.fr"
"bank_fr_yvlcfrp1","YVES LEVEN CAPITAL","YVLCFRP1","base.fr"
"bank_fr_yrocfrpp","YVES ROCHER","YROCFRPP","base.fr"
"bank_fr_zavofrpp","Z AND V SAS","ZAVOFRPP","base.fr"
"bank_fr_zarcfr21","ZARIFI AND ASSOCIES SA","ZARCFR21","base.fr"
"bank_fr_zebkfrp1","ZEBANK","ZEBKFRP1","base.fr"
"bank_fr_zencfrp1","ZENCAP AM","ZENCFRP1","base.fr"
"bank_fr_zodcfr22","ZODIAC AEROSPACE","ZODCFR22","base.fr"
"bank_fr_zuiffr21","ZURICH INTERNATIONAL FRANCE","ZUIFFR21","base.fr"

```

## File: data\tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.fr"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="integer_rounding">HALF-UP</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_montant_op_realisees" model="account.report.line">
                <field name="name">A. Amount of operations carried out</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_op_imposables_ht" model="account.report.line">
                        <field name="name">Taxable transactions (excl. VAT)</field>
                        <field name="children_ids">
                            <record id="tax_report_A1" model="account.report.line">
                                <field name="name">A1 - Sales, provision of services</field>
                                <field name="code">box_A1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_A1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">A1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_A2" model="account.report.line">
                                <field name="name">A2 - Other taxable transactions</field>
                                <field name="code">box_A2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_A2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">A2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_A3" model="account.report.line">
                                <field name="name">A3 - Intra-Community purchases of services</field>
                                <field name="code">box_A3</field>
                                <field name="expression_ids">
                                    <record id="tax_report_A3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">A3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_A4" model="account.report.line">
                                <field name="name">A4 - Imports (other than petroleum products)</field>
                                <field name="code">box_A4</field>
                                <field name="expression_ids">
                                    <record id="tax_report_A4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">A4</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_A5" model="account.report.line">
                                <field name="name">A5 - Removal from suspensive tax regime (other than petroleum products)</field>
                                <field name="code">box_A5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_A5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">A5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_B1" model="account.report.line">
                                <field name="name">B1 - Releases for consumption of petroleum products</field>
                                <field name="code">box_B1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_B1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">B1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_B2" model="account.report.line">
                                <field name="name">B2 - Intra-Community acquisitions</field>
                                <field name="code">box_B2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_B2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">B2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_B3" model="account.report.line">
                                <field name="name">B3 - Taxable supplies of electricity, natural gas, heat or cooling in France</field>
                                <field name="code">box_B3</field>
                                <field name="expression_ids">
                                    <record id="tax_report_B3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">B3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_B4" model="account.report.line">
                                <field name="name">B4 - Purchases of goods or services from a taxable person not established in France</field>
                                <field name="code">box_B4</field>
                                <field name="expression_ids">
                                    <record id="tax_report_B4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">B4</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_B5" model="account.report.line">
                                <field name="name">B5 - Regularisations</field>
                                <field name="code">box_B5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_B5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">B5</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_op_non_imposables" model="account.report.line">
                        <field name="name">Untaxed operations</field>
                        <field name="children_ids">
                            <record id="tax_report_E1" model="account.report.line">
                                <field name="name">E1 - Exports outside the EU</field>
                                <field name="code">box_E1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_E1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">E1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_E2" model="account.report.line">
                                <field name="name">E2 - Other non-taxable transactions</field>
                                <field name="code">box_E2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_E2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">E2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_E3" model="account.report.line">
                                <field name="name">E3 - Distance selling taxable in another Member State to non-taxable persons</field>
                                <field name="code">box_E3</field>
                                <field name="expression_ids">
                                    <record id="tax_report_E3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">E3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_E4" model="account.report.line">
                                <field name="name">E4 - Imports (other than petroleum products)</field>
                                <field name="code">box_E4</field>
                                <field name="expression_ids">
                                    <record id="tax_report_E4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">E4</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_E5" model="account.report.line">
                                <field name="name">E5 - Removal from suspensive tax regime (other than petroleum products)</field>
                                <field name="code">box_E5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_E5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">E5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_E6" model="account.report.line">
                                <field name="name">E6 - Imports under suspensive tax arrangements (other than petroleum products)</field>
                                <field name="code">box_E6</field>
                                <field name="expression_ids">
                                    <record id="tax_report_E6_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">E6</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F1" model="account.report.line">
                                <field name="name">F1 - Intra-Community acquisitions</field>
                                <field name="code">box_F1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F2" model="account.report.line">
                                <field name="name">F2 - Intra-Community supplies to a taxable person</field>
                                <field name="code">box_F2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F3" model="account.report.line">
                                <field name="name">F3 - Non-taxable supplies of electricity, natural gas, heat or cooling in France</field>
                                <field name="code">box_F3</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F4" model="account.report.line">
                                <field name="name">F4 - Releases for consumption of petroleum products</field>
                                <field name="code">box_F4</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F4</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F5" model="account.report.line">
                                <field name="name">F5 - Imports of petroleum products under a suspensive tax regime</field>
                                <field name="code">box_F5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F6" model="account.report.line">
                                <field name="name">F6 - Franchise purchases</field>
                                <field name="code">box_F6</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F6_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F6</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F7" model="account.report.line">
                                <field name="name">F7 - Sales of goods or services by a taxable person not established in France</field>
                                <field name="code">box_F7</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F7</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F8" model="account.report.line">
                                <field name="name">F8 - Accruals</field>
                                <field name="code">box_7B</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">7B</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_F9" model="account.report.line">
                                <field name="name">F9 - Internal transactions between members of a single taxable person</field>
                                <field name="code">box_F9</field>
                                <field name="expression_ids">
                                    <record id="tax_report_F9_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">F9</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_decompte_tva" model="account.report.line">
                <field name="name">B. Settlement of VAT to be paid</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_tva_brute" model="account.report.line">
                        <field name="name">Gross VAT</field>
                    </record>
                    <record id="tax_report_tva_brute_metropo" model="account.report.line">
                        <field name="name">Operations carried out in mainland France</field>
                        <field name="children_ids">
                            <record id="tax_report_08_base" model="account.report.line">
                                <field name="name">08 - Standard rate 20% (base)</field>
                                <field name="code">box_08_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_08_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">08_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_08_taxe" model="account.report.line">
                                <field name="name">08 - Standard rate 20% (tax)</field>
                                <field name="code">box_08_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_08_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_08_base.balance * 0.2</field>
                                    </record>
                                    <record id="tax_report_08_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">08_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_09_base" model="account.report.line">
                                <field name="name">09 - Reduced rate 5.5% (base)</field>
                                <field name="code">box_09_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_09_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">09_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_09_taxe" model="account.report.line">
                                <field name="name">09 - Reduced rate 5.5% (tax)</field>
                                <field name="code">box_09_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_09_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_09_base.balance * 0.055</field>
                                    </record>
                                    <record id="tax_report_09_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">09_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_9B_base" model="account.report.line">
                                <field name="name">9B - Reduced rate 10% (base)</field>
                                <field name="code">box_9B_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_9B_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">9B_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_9B_taxe" model="account.report.line">
                                <field name="name">9B - Reduced rate 10% (tax)</field>
                                <field name="code">box_9B_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_9B_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_9B_base.balance * 0.1</field>
                                    </record>
                                    <record id="tax_report_9B_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">9B_taxe</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tva_brute_dom" model="account.report.line">
                        <field name="name">Operations carried out in the DOM</field>
                        <field name="children_ids">
                            <record id="tax_report_10_base" model="account.report.line">
                                <field name="name">10 - Standard rate 8.5% (base)</field>
                                <field name="code">box_10_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_10_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">10_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_10_taxe" model="account.report.line">
                                <field name="name">10 - Standard rate 8.5% (tax)</field>
                                <field name="code">box_10_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_10_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_10_base.balance * 0.085</field>
                                    </record>
                                    <record id="tax_report_10_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">10_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_11_base" model="account.report.line">
                                <field name="name">11 - Reduced rate 2.1% (base)</field>
                                <field name="code">box_11_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_11_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">11_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_11_taxe" model="account.report.line">
                                <field name="name">11 - Reduced rate 2.1% (tax)</field>
                                <field name="code">box_11_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_11_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_11_base.balance * 0.021</field>
                                    </record>
                                    <record id="tax_report_11_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">11_taxe</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tva_brute_autre" model="account.report.line">
                        <field name="name">Transactions taxable at another rate (metropolitan France or DOM)</field>
                        <field name="children_ids">
                            <record id="tax_report_T1_base" model="account.report.line">
                                <field name="name">T1 - Transactions carried out in the French overseas departments and taxable at 1.75% (base)</field>
                                <field name="code">box_T1_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T1_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T1_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T1_taxe" model="account.report.line">
                                <field name="name">T1 - Transactions carried out in the French overseas departments and taxable at 1.75% (tax)</field>
                                <field name="code">box_T1_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T1_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_T1_base.balance * 0.0175</field>
                                    </record>
                                    <record id="tax_report_T1_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T1_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T2_base" model="account.report.line">
                                <field name="name">T2 - Transactions carried out in the French overseas departments and taxable at 1.05% (base)</field>
                                <field name="code">box_T2_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T2_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T2_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T2_taxe" model="account.report.line">
                                <field name="name">T2 - Transactions carried out in the French overseas departments and taxable at 1,05% (tax)</field>
                                <field name="code">box_T2_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T2_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_T2_base.balance * 0.0105</field>
                                    </record>
                                    <record id="tax_report_T2_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T2_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T3_base" model="account.report.line">
                                <field name="name">T3 - Transactions carried out in Corsica and taxable at the rate of 10% (base)</field>
                                <field name="code">box_T3_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T3_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T3_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T3_taxe" model="account.report.line">
                                <field name="name">T3 - Transactions carried out in Corsica and taxable at the rate of 10% (tax)</field>
                                <field name="code">box_T3_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T3_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_T3_base.balance * 0.1</field>
                                    </record>
                                    <record id="tax_report_T3_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T3_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T4_base" model="account.report.line">
                                <field name="name">T4 - Transactions carried out in Corsica and taxable at the rate of 2,1% (base)</field>
                                <field name="code">box_T4_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T4_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T4_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T4_taxe" model="account.report.line">
                                <field name="name">T4 - Transactions carried out in Corsica and taxable at the rate of 2,1% (tax)</field>
                                <field name="code">box_T4_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T4_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_T4_base.balance * 0.021</field>
                                    </record>
                                    <record id="tax_report_T4_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T4_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T5_base" model="account.report.line">
                                <field name="name">T5 - Transactions carried out in Corsica and taxable at the rate of 0,9% (base)</field>
                                <field name="code">box_T5_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T5_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T5_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T5_taxe" model="account.report.line">
                                <field name="name">T5 - Transactions carried out in Corsica and taxable at the rate of 0,9% (tax)</field>
                                <field name="code">box_T5_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T5_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_T5_base.balance * 0.009</field>
                                    </record>
                                    <record id="tax_report_T5_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T5_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T6_base" model="account.report.line">
                                <field name="name">T6 - Transactions carried out in mainland France at the rate of 2,1% (base)</field>
                                <field name="code">box_T6_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T6_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T6_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T6_taxe" model="account.report.line">
                                <field name="name">T6 - Transactions carried out in mainland France at the rate of 2,1% (tax)</field>
                                <field name="code">box_T6_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T6_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_T6_base.balance * 0.021</field>
                                    </record>
                                    <record id="tax_report_T6_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T6_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T7_base" model="account.report.line">
                                <field name="name">T7 - Withholding of VAT on copyright (base)</field>
                                <field name="code">box_T7_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T7_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T7_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_T7_taxe" model="account.report.line">
                                <field name="name">T7 - Withholding of VAT on copyright (tax)</field>
                                <field name="code">box_T7_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_T7_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">T7_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_13_base" model="account.report.line">
                                <field name="name">13 - Former rates (base)</field>
                                <field name="code">box_13_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_13_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">13_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_13_taxe" model="account.report.line">
                                <field name="name">13 - Former rates (tax)</field>
                                <field name="code">box_13_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_13_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">13_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_14_base" model="account.report.line">
                                <field name="name">14 - Transactions taxable at a particular rate (base)</field>
                                <field name="code">box_14_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_14_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">14_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_14_taxe" model="account.report.line">
                                <field name="name">14 - Transactions taxable at a particular rate (tax)</field>
                                <field name="code">box_14_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_14_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">14_taxe</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tva_brute_petrolier" model="account.report.line">
                        <field name="name">Oil products</field>
                        <field name="children_ids">
                            <record id="tax_report_P1_base" model="account.report.line">
                                <field name="name">P1 - Standard rate 20% (base)</field>
                                <field name="code">box_P1_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_P1_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">P1_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_P1_taxe" model="account.report.line">
                                <field name="name">P1 - Standard rate 20% (tax)</field>
                                <field name="code">box_P1_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_P1_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_P1_base.balance * 0.2</field>
                                    </record>
                                    <record id="tax_report_P1_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">P1_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_P2_base" model="account.report.line">
                                <field name="name">P2 - Reduced rate 13% (base)</field>
                                <field name="code">box_P2_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_P2_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">P2_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_P2_taxe" model="account.report.line">
                                <field name="name">P2 - Reduced rate 13% (tax)</field>
                                <field name="code">box_P2_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_P2_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_P2_base.balance * 0.13</field>
                                    </record>
                                    <record id="tax_report_P2_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">P2_taxe</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tva_brute_import" model="account.report.line">
                        <field name="name">Imports</field>
                        <field name="children_ids">
                            <record id="tax_report_I1_base" model="account.report.line">
                                <field name="name">I1 - Standard rate 20% (base)</field>
                                <field name="code">box_I1_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I1_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I1_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I1_taxe" model="account.report.line">
                                <field name="name">I1 - Standard rate 20% (tax)</field>
                                <field name="code">box_I1_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I1_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_I1_base.balance * 0.2</field>
                                    </record>
                                    <record id="tax_report_I1_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I1_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I2_base" model="account.report.line">
                                <field name="name">I2 - Reduced rate 10% (base)</field>
                                <field name="code">box_I2_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I2_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I2_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I2_taxe" model="account.report.line">
                                <field name="name">I2 - Reduced rate 10% (tax)</field>
                                <field name="code">box_I2_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I2_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_I2_base.balance * 0.1</field>
                                    </record>
                                    <record id="tax_report_I2_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I2_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I3_base" model="account.report.line">
                                <field name="name">I3 - Reduced rate 8.5% (base)</field>
                                <field name="code">box_I3_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I3_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I3_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I3_taxe" model="account.report.line">
                                <field name="name">I3 - Reduced rate 8.5% (tax)</field>
                                <field name="code">box_I3_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I3_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_I3_base.balance * 0.085</field>
                                    </record>
                                    <record id="tax_report_I3_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I3_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I4_base" model="account.report.line">
                                <field name="name">I4 - Reduced rate 5.5% (base)</field>
                                <field name="code">box_I4_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I4_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I4_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I4_taxe" model="account.report.line">
                                <field name="name">I4 - Reduced rate 5.5% (tax)</field>
                                <field name="code">box_I4_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I4_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_I4_base.balance * 0.055</field>
                                    </record>
                                    <record id="tax_report_I4_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I4_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I5_base" model="account.report.line">
                                <field name="name">I5 - Reduced rate 2.1% (base)</field>
                                <field name="code">box_I5_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I5_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I5_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I5_taxe" model="account.report.line">
                                <field name="name">I5 - Reduced rate 2.1% (tax)</field>
                                <field name="code">box_I5_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I5_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_I5_base.balance * 0.021</field>
                                    </record>
                                    <record id="tax_report_I5_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I5_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I6_base" model="account.report.line">
                                <field name="name">I6 - Reduced rate 1.05% (base)</field>
                                <field name="code">box_I6_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I6_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I6_base</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_I6_taxe" model="account.report.line">
                                <field name="name">I6 - Reduced rate 1.05% (tax)</field>
                                <field name="code">box_I6_taxe</field>
                                <field name="expression_ids">
                                    <record id="tax_report_I6_taxe_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_I6_base.balance * 0.0105</field>
                                    </record>
                                    <record id="tax_report_I6_taxe_tag_no_rounding" model="account.report.expression">
                                        <field name="label">balance_from_tags</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I6_taxe</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_15" model="account.report.line">
                                <field name="name">15 - Previously deducted VAT to be repaid</field>
                                <field name="code">box_15</field>
                                <field name="expression_ids">
                                    <record id="tax_report_15_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">15</field>
                                    </record>
                                </field>
                                <field name="children_ids">
                                    <record id="tax_report_15_1" model="account.report.line">
                                        <field name="name">of which VAT on petroleum products</field>
                                        <field name="code">box_15_1</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_15_1_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">15_1</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_15_2" model="account.report.line">
                                        <field name="name">of which VAT on imported products excluding petroleum products</field>
                                        <field name="code">box_15_2</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_15_2_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">15_2</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_5B" model="account.report.line">
                                <field name="name">5B - Amounts to be added, including advance holiday pay</field>
                                <field name="code">box_5B</field>
                                <field name="expression_ids">
                                    <record id="tax_report_5B_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">5B</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_16" model="account.report.line">
                                <field name="name">16 - Total gross VAT due</field>
                                <field name="code">box_16</field>
                                <field name="aggregation_formula">box_08_taxe.balance_from_tags + box_09_taxe.balance_from_tags + box_9B_taxe.balance_from_tags + box_10_taxe.balance_from_tags + box_11_taxe.balance_from_tags + box_13_taxe.balance + box_14_taxe.balance + box_T1_taxe.balance_from_tags + box_T2_taxe.balance_from_tags + box_T3_taxe.balance_from_tags + box_T4_taxe.balance_from_tags + box_T5_taxe.balance_from_tags + box_T6_taxe.balance_from_tags + box_T7_taxe.balance + box_P1_taxe.balance_from_tags + box_P2_taxe.balance_from_tags + box_I1_taxe.balance_from_tags + box_I2_taxe.balance_from_tags + box_I3_taxe.balance_from_tags + box_I4_taxe.balance_from_tags + box_I5_taxe.balance_from_tags + box_I6_taxe.balance_from_tags + box_15.balance + box_5B.balance</field>
                            </record>
                            <record id="tax_report_17" model="account.report.line">
                                <field name="name">17 - Of which VAT on intra-Community acquisitions</field>
                                <field name="code">box_17</field>
                                <field name="expression_ids">
                                    <record id="tax_report_17_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">17</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_18" model="account.report.line">
                                <field name="name">18 - Of which VAT on transactions to Monaco</field>
                                <field name="code">box_18</field>
                                <field name="expression_ids">
                                    <record id="tax_report_18_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">18</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tva_deductible" model="account.report.line">
                        <field name="name">Deductible VAT</field>
                        <field name="children_ids">
                            <record id="tax_report_19" model="account.report.line">
                                <field name="name">19 - Assets constituting fixed assets</field>
                                <field name="code">box_19</field>
                                <field name="expression_ids">
                                    <record id="tax_report_19_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">19</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_20" model="account.report.line">
                                <field name="name">20 - Other goods and services</field>
                                <field name="code">box_20</field>
                                <field name="expression_ids">
                                    <record id="tax_report_20_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">20</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_21" model="account.report.line">
                                <field name="name">21 - Other deductible VAT</field>
                                <field name="code">box_21</field>
                                <field name="expression_ids">
                                    <record id="tax_report_21_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">21</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_22" model="account.report.line">
                                <field name="name">22 - Carry-over of credit from line 27 of the previous return</field>
                                <field name="code">box_22</field>
                                <field name="expression_ids">
                                    <record id="tax_report_22_applied_carryover" model="account.report.expression">
                                        <field name="label">_applied_carryover_balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="date_scope">previous_tax_period</field>
                                    </record>
                                    <record id="tax_report_22_tag" model="account.report.expression">
                                        <field name="label">tag</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">22</field>
                                    </record>
                                    <record id="tax_report_22_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">box_22.tag + box_22._applied_carryover_balance</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_2C" model="account.report.line">
                                <field name="name">2C - Amounts to be charged, including advance holiday pay</field>
                                <field name="code">box_2C</field>
                                <field name="expression_ids">
                                    <record id="tax_report_2C_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">2C</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_22A" model="account.report.line">
                                <field name="name">22A - Enter the single tax rate applicable for the period if different from 100%</field>
                                <field name="code">box_22A</field>
                                <field name="expression_ids">
                                    <record id="tax_report_22A_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">22A</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_23" model="account.report.line">
                                <field name="name">23 - Total deductible VAT</field>
                                <field name="code">box_23</field>
                                <field name="aggregation_formula">box_19.balance + box_20.balance + box_21.balance + box_22.balance + box_2C.balance</field>
                            </record>
                            <record id="tax_report_24" model="account.report.line">
                                <field name="name">24 - Of which deductible VAT on imports</field>
                                <field name="code">box_24</field>
                                <field name="expression_ids">
                                    <record id="tax_report_24_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">24</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_2E" model="account.report.line">
                                <field name="name">2E - Of which deductible VAT on petroleum products</field>
                                <field name="code">box_2E</field>
                                <field name="expression_ids">
                                    <record id="tax_report_2E_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">2E</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_credit" model="account.report.line">
                <field name="name">Credit</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_25" model="account.report.line">
                        <field name="name">25 - VAT credit</field>
                        <field name="code">box_25</field>
                        <field name="expression_ids">
                            <record id="tax_report_25_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">box_23.balance - box_16.balance</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_TD" model="account.report.line">
                        <field name="name">TD - VAT Due</field>
                        <field name="code">box_TD</field>
                        <field name="expression_ids">
                            <record id="tax_report_td_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">box_16.balance - box_23.balance</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_regularisation" model="account.report.line">
                <field name="name">Regularisation of domestic consumption taxes (TIC)</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_credit_constate" model="account.report.line">
                        <field name="name">Recognised credit</field>
                        <field name="children_ids">
                            <record id="tax_report_TICFE" model="account.report.line">
                                <field name="name">TICFE</field>
                                <field name="code">box_TICFE</field>
                                <field name="expression_ids">
                                    <record id="tax_report_TICFE_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TICFE_constate</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_TICGN" model="account.report.line">
                                <field name="name">TICGN</field>
                                <field name="code">box_TICGN</field>
                                <field name="expression_ids">
                                    <record id="tax_report_TICGN_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TICGN_constate</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_TICC" model="account.report.line">
                                <field name="name">TICC</field>
                                <field name="code">box_TICC</field>
                                <field name="expression_ids">
                                    <record id="tax_report_TICC_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">TICC_constate</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_TIC_total" model="account.report.line">
                                <field name="name">Total</field>
                                <field name="code">box_TIC_total</field>
                                <field name="aggregation_formula">box_TICFE.balance + box_TICGN.balance + box_TICC.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_credit_impute" model="account.report.line">
                        <field name="name">Credit charged</field>
                        <field name="children_ids">
                            <record id="tax_report_X1" model="account.report.line">
                                <field name="name">X1</field>
                                <field name="code">box_X1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_X1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">X1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_X2" model="account.report.line">
                                <field name="name">X2</field>
                                <field name="code">box_X2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_X2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">X2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_X3" model="account.report.line">
                                <field name="name">X3</field>
                                <field name="code">box_X3</field>
                                <field name="expression_ids">
                                    <record id="tax_report_X3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">X3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_X4" model="account.report.line">
                                <field name="name">X4</field>
                                <field name="code">box_X4</field>
                                <field name="aggregation_formula">box_X1.balance + box_X2.balance + box_X3.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_reliquat" model="account.report.line">
                        <field name="name">Outstanding credit</field>
                        <field name="children_ids">
                            <record id="tax_report_Y1" model="account.report.line">
                                <field name="name">Y1</field>
                                <field name="code">box_Y1</field>
                                <field name="aggregation_formula">box_TICFE.balance - box_X1.balance</field>
                            </record>
                            <record id="tax_report_Y2" model="account.report.line">
                                <field name="name">Y2</field>
                                <field name="code">box_Y2</field>
                                <field name="aggregation_formula">box_TICGN.balance - box_X2.balance</field>
                            </record>
                            <record id="tax_report_Y3" model="account.report.line">
                                <field name="name">Y3</field>
                                <field name="code">box_Y3</field>
                                <field name="aggregation_formula">box_TICC.balance - box_X3.balance</field>
                            </record>
                            <record id="tax_report_Y4" model="account.report.line">
                                <field name="name">Y4</field>
                                <field name="code">box_Y4</field>
                                <field name="aggregation_formula">box_Y1.balance + box_Y2.balance + box_Y3.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tic_tax" model="account.report.line">
                        <field name="name">Tax due</field>
                        <field name="children_ids">
                            <record id="tax_report_Z1" model="account.report.line">
                                <field name="name">Z1</field>
                                <field name="code">box_Z1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_Z1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Z1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_Z2" model="account.report.line">
                                <field name="name">Z2</field>
                                <field name="code">box_Z2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_Z2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Z2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_Z3" model="account.report.line">
                                <field name="name">Z3</field>
                                <field name="code">box_Z3</field>
                                <field name="expression_ids">
                                    <record id="tax_report_Z3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Z3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_Z4" model="account.report.line">
                                <field name="name">Z4</field>
                                <field name="code">box_Z4</field>
                                <field name="aggregation_formula">box_Z1.balance + box_Z2.balance + box_Z3.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_determination" model="account.report.line">
                <field name="name">Determining the amount to be paid and/or VAT and/or TIC credits</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_26_external" model="account.report.line">
                        <field name="name">26 - Repayment of credit requested on form n°3519 attached</field>
                        <field name="code">box_26_external</field>
                        <field name="expression_ids">
                            <record id="tax_report_26_external_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_AA" model="account.report.line">
                        <field name="name">AA - VAT credit transferred to the head company on the recapitulative return 3310-CA3G</field>
                        <field name="code">box_AA</field>
                        <field name="expression_ids">
                            <record id="tax_report_AA_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">AA</field>
                            </record>
                        </field>
                    </record>

                    <record id="tax_report_27" model="account.report.line">
                        <field name="name">27 - Credit to be carried forward</field>
                        <field name="code">box_27</field>
                        <field name="expression_ids">
                            <record id="tax_report_27_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">box_25.balance - box_26_external.balance - box_AA.balance</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                            <record id="tax_report_27_carryover" model="account.report.expression">
                                <field name="label">_carryover_balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">box_27.balance</field>
                                <field name="carryover_target">box_22._applied_carryover_balance</field>
                                <field name="subformula" eval="False"/>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_Y5" model="account.report.line">
                        <field name="name">Y5 - Refund of TIC balance requested (carried forward from line Y4)</field>
                        <field name="code">box_Y5</field>
                        <field name="aggregation_formula">box_Y4.balance</field>
                    </record>
                    <record id="tax_report_Y6" model="account.report.line">
                        <field name="name">Y6 - TIC credit transferred to the head company on the 3310-CA3G recapitulative return (carried forward from line Y4)</field>
                        <field name="code">box_Y6</field>
                    </record>
                    <record id="tax_report_X5" model="account.report.line">
                        <field name="name">X5 - TIC credit offset against VAT (carried forward from line X4)</field>
                        <field name="code">box_X5</field>
                        <field name="aggregation_formula">box_X4.balance</field>
                    </record>
                    <record id="tax_report_28" model="account.report.line">
                        <field name="name">28 - Net VAT due</field>
                        <field name="code">box_28</field>
                        <field name="expression_ids">
                            <record id="tax_report_28_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">box_TD.balance - box_X5.balance</field>
                                <field name="subformula">if_above(EUR(0))</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_29" model="account.report.line">
                        <field name="name">29 - Similar taxes calculated on schedule n°3310-A-SD</field>
                        <field name="code">box_29</field>
                        <field name="expression_ids">
                            <record id="tax_report_29_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">29</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_Z5" model="account.report.line">
                        <field name="name">Z5 - Total domestic consumption tax due (carried forward from line Z4)</field>
                        <field name="code">box_Z5</field>
                        <field name="aggregation_formula">box_Z4.balance</field>
                    </record>
                    <record id="tax_report_AB" model="account.report.line">
                        <field name="name">AB - Total to be paid by the head company on the recapitulative declaration 3310-CA3G</field>
                        <field name="code">box_AB</field>
                    </record>
                    <record id="tax_report_32" model="account.report.line">
                        <field name="name">32 - Total payable</field>
                        <field name="code">box_32</field>
                        <field name="aggregation_formula">box_28.balance + box_29.balance + box_Z5.balance</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\template\account.account-fr.csv

```csv
"id","name","code","account_type","tag_ids","reconcile","name@fr"
"pcg_1011","Subscribed capital - uncalled","101100","equity","","False","Capital souscrit - non appelé"
"pcg_1012","Subscribed capital - called up, unpaid","101200","equity","","False","Capital souscrit - appelé non versé"
"pcg_1013","Subscribed capital - called up, paid","101300","equity","","False","Capital souscrit appelé, versé"
"pcg_10131","Capital not written off","101310","equity","","False","Capital non amorti"
"pcg_10132","Capital written off","101320","equity","","False","Capital amorti"
"pcg_1018","Subscribed capital subject to particular regulations","101800","equity","","False","Capital souscrit soumis à des réglementations particulières"
"pcg_102","Trust funds","102000","equity","","False","Fonds fiduciaires"
"pcg_1041","Share premiums","104100","equity","","False","Primes d'émission"
"pcg_1042","Merger premiums","104200","equity","","False","Primes de fusion"
"pcg_1043","Contribution premiums","104300","equity","","False","Primes d'apport"
"pcg_1044","Premiums on conversion of bonds into shares","104400","equity","","False","Primes de conversion d'obligations en actions"
"pcg_1045","Equity warrants","104500","equity","","False","Bons de souscription d'actions"
"pcg_105_account","Revaluation differences","105000","equity","","False","Ecarts de réévaluation"
"pcg_1061","Legal reserve","106100","equity","","False","Réserve légale"
"pcg_1062","Undistributable reserves","106200","equity","","False","Réserves indisponibles"
"pcg_1063","Statutory or contractual reserves","106300","equity","","False","Réserves statutaires ou contractuelles"
"pcg_1064","Tax-regulated reserves","106400","equity","","False","Réserves réglementées"
"pcg_1068","Other reserves","106800","equity","","False","Autres réserves"
"pcg_107","Difference on equity accounted investments","107000","equity","","False","Écarts d'équivalence"
"pcg_108","Drawings account","108000","equity","","False","Compte de l'exploitant"
"pcg_109","Shareholders: Subscribed capital uncalled","109000","equity","","False","Actionnaires : capital souscrit - non appelé"
"pcg_110","Profit carried forward","110000","equity","","False","Report à nouveau (solde créditeur)"
"pcg_119","Loss carried forward","119000","equity","","False","Report à nouveau (solde débiteur)"
"pcg_120","Profit for the financial year","120000","equity","","False","Résultat de l'exercice (bénéfice)"
"pcg_1209","Withholding tax on dividends","120900","equity","","False","Accomptes sur les dividendes"
"pcg_129","Loss for the financial year","129000","equity","","False","Résultat de l'exercice (perte)"
"pcg_131_account","Equipment grants","131000","equity","","False","Subventions d'équipement"
"pcg_139_account","Investment grants recognised in the income statement","139000","equity","","False","Subventions d'investissement inscrites au compte de résultat"
"pcg_14_account","Regulated provisions","140000","liability_non_current","","False","Provisions réglementées"
"pcg_143_account","Regulated provisions for inventories - Price increases","143000","liability_non_current","","False","Provisions réglementées relatives aux stocks - Hausse de prix"
"pcg_145","Depreciation by derogation","145000","liability_non_current","","False","Amortissements dérogatoires"
"pcg_148","Other tax-regulated provisions","148000","liability_non_current","","False","Autres provisions réglementées"
"pcg_1511","Provisions for litigation","151100","liability_non_current","","False","Provisions pour litiges"
"pcg_1512","Provisions for customer warranties","151200","liability_non_current","","False","Provisions pour garanties données aux clients"
"pcg_1514","Provisions for fines and penalties","151400","liability_non_current","","False","Provisions pour amendes et pénalités"
"pcg_1515","Provisions for foreign exchange losses","151500","liability_non_current","","False","Provisions pour pertes de change"
"pcg_1516","Provisions for losses on contracts","151600","liability_non_current","","False","Provisions pour pertes sur contrats"
"pcg_1518","Other provisions for liabilities","151800","liability_non_current","","False","Autres provisions pour risques"
"pcg_1521_account","Provisions for pensions and similar obligations","152100","liability_non_current","","False","Provisions pour pensions et obligations similaires"
"pcg_1522_account","Provisions for restructuring","152200","liability_non_current","","False","Provisions pour restructurations"
"pcg_1523_account","Provisions for taxation","152300","liability_non_current","","False","Provisions pour impôts"
"pcg_1524_account","Provisions for fixed asset renewal - concession entities","152400","liability_non_current","","False","Provisions pour renouvellement des immobilisations - entreprises concessionnaires"
"pcg_1525_account","Provisions for major maintenance and repairs","152500","liability_non_current","","False","Provisions pour gros entretien ou grandes révisions"
"pcg_1526_account","Provisions for repairs","152600","liability_non_current","","False","Provisions pour remises en état"
"pcg_1527_account","Other provisions for charges","152700","liability_non_current","","False","Autres provisions pour charges"
"pcg_161","Convertible debenture loans","161000","liability_non_current","","False","Emprunts obligataires convertibles"
"pcg_1618","Accrued interest on convertible bonds","161800","liability_non_current","","False","Intérêts courus sur emprunts obligataires convertibles"
"pcg_162","Bonds representing net liabilities given in trust","162000","liability_non_current","","False","Obligations représentatives de passifs nets remis en fiducie"
"pcg_163","Other debenture loans","163000","liability_non_current","","False","Autres emprunts obligataires"
"pcg_1638","Accrued interest on other bonds","163800","liability_non_current","","False","Intérêts courus sur autres emprunts obligataires"
"pcg_164","Loans from credit institutions","164000","liability_non_current","","False","Emprunts auprès des établissements de crédit"
"pcg_1648","Accrued interest on bank loans","164800","liability_non_current","","False","Intérêts courus sur emprunts auprès des établissements"
"pcg_1651","Deposits","165100","liability_non_current","","False","Dépôts"
"pcg_1655","Sureties","165500","liability_non_current","","False","Cautionnements"
"pcg_1658","Accrued interest on deposits and guarantees received","165800","liability_non_current","","False","Intérêts courus sur dépôts et cautionnements reçus"
"pcg_1661","Employee profit sharing - Blocked accounts","166100","liability_non_current","","False","Participation des salariés aux résultats - Comptes bloqués"
"pcg_1662","Employee profit sharing - Profit share funds","166200","liability_non_current","","False","Participation des salariés aux résultats - Fonds de participation"
"pcg_1668","Accrued interest on employee profit-sharing","166800","liability_non_current","","False","Intérêts courus sur participation des salariés aux résultats"
"pcg_1671","Loans and debts with special conditions - Issues of non-voting shares","167100","liability_non_current","","False","Emprunts et dettes assortis de conditions particulières - Emissions de titres participatifs"
"pcg_16718","Accrued interest on redeemable shares","167180","liability_non_current","","False","Intérêts courus sur titres participatifs"
"pcg_1674","Loans and debts with special conditions - Advances by the state subject to conditions","167400","liability_non_current","","False","Emprunts et dettes assortis de conditions particulières - Avances conditionnées de l'État"
"pcg_16748","Accrued interest on conditional advances","167480","liability_non_current","","False","Intérêts courus sur avances conditionnées"
"pcg_1675","Loans and debts with special conditions - Participating loans","167500","liability_non_current","","False","Prêts participatifs"
"pcg_16758","Accrued interest on participating loans","167580","liability_non_current","","False","Intérêts courus sur emprunts participatifs"
"pcg_1681","Other loans and similar debts payable - Other loans","168100","liability_non_current","","False","Autres emprunts et dettes assimilées - Autres emprunts"
"pcg_1685","Other loans and similar debts payable - Capitalised life annuities","168500","liability_non_current","","False","Autres emprunts et dettes assimilées - Rentes viagères capitalisées"
"pcg_1687","Other loans and similar debts payable - Other debts payable","168700","liability_non_current","","False","Autres emprunts et dettes assimilées - Autres dettes"
"pcg_1688","Accrued interest","168800","liability_non_current","","False","Intérêts courus"
"pcg_169","Debt redemption premiums","169000","asset_current","","False","Primes de remboursement des obligations"
"pcg_171","Debts payable related to participating interests (group)","171000","liability_non_current","","False","Dettes rattachées à des participations (groupe)"
"pcg_174","Debts payable related to participating interests (apart from group)","174000","liability_non_current","","False","Dettes rattachées à des participations (hors groupe)"
"pcg_178_account","Debts payable related to joint ventures","178000","liability_non_current","","False","Dettes rattachées à des sociétés en participation"
"pcg_181","Reciprocal branch accounts","181000","liability_non_current","","False","Comptes de liaison des établissements"
"pcg_186","Goods and services exchanged between establishments (expenses)","186000","liability_non_current","","False","Biens et prestations de services échangés entre établissements (charges)"
"pcg_187","Goods and services exchanged between establishments (income)","187000","liability_non_current","","False","Biens et prestations de services échangés entre établissements (produits)"
"pcg_188","Reciprocal joint venture accounts","188000","liability_non_current","","False","Comptes de liaison des sociétés en participation"
"pcg_2011","Incorporation costs","201100","asset_fixed","","False","Frais de constitution"
"pcg_2012","Start-up costs","201200","asset_fixed","","False","Frais de premier établissement"
"pcg_20121","Commercial assessment costs","201210","asset_fixed","","False","Frais de prospection"
"pcg_20122","Marketing costs","201220","asset_fixed","","False","Frais de publicité"
"pcg_2013","Capital increase and sundry transaction costs (mergers, demergers, restructurations)","201300","asset_fixed","","False","Frais d'augmentation de capital et d'opérations diverses (fusions, scissions, transformations)"
"pcg_203","Research and development costs","203000","asset_fixed","","False","Frais de recherche et de développement"
"pcg_205","Concessions and similar rights, patents, licences, trade marks, processes, software, rights and similar assets","205000","asset_fixed","","False","Concessions et droits similaires, brevets, licences, marques, procédés, logiciels, droits et valeurs similaires"
"pcg_206","Lease premium","206000","asset_fixed","","False","Droit au bail"
"pcg_207","Goodwill","207000","asset_fixed","","False","Fonds commercial"
"pcg_208","Other intangible fixed assets","208000","asset_fixed","","False","Dépréciations des autres immobilisations incorporelles"
"pcg_2111","Undeveloped land","211100","asset_fixed","","False","Terrains nus"
"pcg_2112","Serviced land","211200","asset_fixed","","False","Terrains aménagés"
"pcg_2113","Basements and sub-basements","211300","asset_fixed","","False","Sous-sols et sur-sols"
"pcg_2114","Mining sites","211400","asset_fixed","","False","Terrains de carrières (Tréfonds)"
"pcg_2115","Developed land","211500","asset_fixed","","False","Terrains bâtis"
"pcg_212","Site development (same allocation as for Account 211)","212000","asset_fixed","","False","Agencements et aménagements de terrains (même ventilation que celle du compte 211)"
"pcg_2131","Buildings","213100","asset_fixed","","False","Bâtiments"
"pcg_2135","Building fixtures and fittings (same allocation as for Account 2131)","213500","asset_fixed","","False","Installations générales, agencements, aménagements des constructions (même ventilation que celle du compte 2131)"
"pcg_2138_account","Infrastructure works","213800","asset_fixed","","False","Ouvrages d'infrastructure"
"pcg_214","Constructions on third-party sites (same allocation as for Account 213)","214000","asset_fixed","","False","Constructions sur sol d'autrui (même ventilation que celle du compte 213)"
"pcg_21511","Specialised complex installations on own site","215110","asset_fixed","","False","Installations complexes spécialisées sur sol propre"
"pcg_21514","Specialised complex installations on third-party site","215140","asset_fixed","","False","Installations complexes spécialisées sur sol d'autrui"
"pcg_21531","Installations of specific nature on own site","215310","asset_fixed","","False","Installations à caractère spécifique sur sol propre"
"pcg_21534","Installations of specific nature on third-party site","215340","asset_fixed","","False","Installations à caractère spécifique sur sol d'autrui"
"pcg_2154","Plant and machinery","215400","asset_fixed","","False","Matériels industriels"
"pcg_2155","Equipment and fixtures","215500","asset_fixed","","False","Outillage industriel"
"pcg_2157","Fixtures and fittings for plant and machinery, equipment and fixtures","215700","asset_fixed","","False","Agencements et aménagements des matériels et outillage industriels"
"pcg_2181","Sundry general fixtures and fittings","218100","asset_fixed","","False","Installations générales agencements aménagements divers"
"pcg_2182","Transport equipment","218200","asset_fixed","","False","Matériel de transport"
"pcg_2183","Office and computing equipment","218300","asset_fixed","","False","Matériel de bureau et matériel informatique"
"pcg_2184","Furnishings","218400","asset_fixed","","False","Mobilier"
"pcg_2185","Livestock","218500","asset_fixed","","False","Cheptel"
"pcg_2186","Recoverable packaging","218600","asset_fixed","","False","Emballages récupérables"
"pcg_2187","Merger malpractice on tangible assets","218700","asset_fixed","","False","Mali de fusions sur actifs corporels"
"pcg_22","Fixed assets in concession","220000","asset_fixed","","False","Immobilisations mises en concession"
"pcg_231_account","Property, plant and equipment in progress","231000","asset_fixed","","False","Immobilisations corporelles en cours"
"pcg_232","Intangible fixed assets in progress","232000","asset_fixed","","False","Dépréciations des immobilisations incorporelles en cours"
"pcg_237","Payments on account on intangible fixed assets","237000","asset_fixed","","False","Avances et acomptes versés sur commandes d'immobilisations incorporelles"
"pcg_238_account","Payments on account on orders for tangible fixed assets","238000","asset_fixed","","False","Avances et acomptes versés sur commandes d'immobilisations corporelles"
"pcg_2611","Long-term equity interests - Shares","261100","asset_fixed","","False","Titres de participation - Actions"
"pcg_2618","Other securities","261800","asset_fixed","","False","Autres valeurs mobilières de placement"
"pcg_262","Equity-valued securities","262000","asset_fixed","","False","Titres évalués par équivalence"
"pcg_266","Other categories of participating interest","266000","asset_fixed","","False","Autres formes de participation"
"pcg_2661","Rights to net assets placed in trust","266100","asset_fixed","","False","Droits représentatifs d’actifs nets remis en fiducie"
"pcg_2671","Debts receivable related to participating interests - group","267100","asset_fixed","","False","Créances rattachées à des participations - groupe"
"pcg_2674","Debts receivable related to participating interests - apart from group","267400","asset_fixed","","False","Créances rattachées à des participations - hors groupe"
"pcg_2675","Payments representing non-capitalised contributions - call for funds","267500","asset_fixed","","False","Versements représentatifs d'apports non capitalisés - appel de fonds"
"pcg_2676","Long-term capital advances","267600","asset_fixed","","False","Avances consolidables"
"pcg_2677","Other debts receivable related to participating interests","267700","asset_fixed","","False","Autres créances rattachées à des participations"
"pcg_2678","Accrued interest","267800","asset_fixed","","False","Intérêts courus"
"pcg_2681","Debts receivable related to joint ventures - Principal","268100","asset_fixed","","False","Créances rattachées à des sociétés en participation - Principal"
"pcg_2688","Debts receivable related to joint ventures - Accrued interest","268800","asset_fixed","","False","Créances rattachées à des sociétés en participation - Intérêts courus"
"pcg_269","Unpaid instalments on unpaid long-term equity interests","269000","asset_fixed","","False","Versements restant à effectuer sur titres de participation non libérés"
"pcg_2711","Shares","271100","asset_fixed","","False","Actions"
"pcg_2718","Other securities","271800","asset_fixed","","False","Autres titres"
"pcg_2721","Bonds","272100","asset_fixed","","False","Obligations"
"pcg_2722","Warrants","272200","asset_fixed","","False","Bons"
"pcg_273","Portfolio long-term investment securities","273000","asset_fixed","","False","Titres immobilisés de l'activité de portefeuille"
"pcg_2741","Participating loans","274100","asset_fixed","","True","Prêts participatifs"
"pcg_2742","Loans to partners/associates","274200","asset_fixed","","True","Prêts aux associés"
"pcg_2743","Loans to personnel","274300","asset_fixed","","True","Prêts au personnel"
"pcg_2748","Other loans","274800","asset_fixed","","True","Autres prêts"
"pcg_2751","Deposits","275100","asset_fixed","","True","Dépôts"
"pcg_2755","Sureties","275500","asset_fixed","","True","Cautionnements"
"pcg_2761","Sundry debts receivable","276100","asset_fixed","","False","Créances diverses"
"pcg_27682","Accrued interest on long-term investment debt securities","276820","asset_fixed","","False","Intérêts courus sur titres immobilisés (droits de créance)"
"pcg_27684","Accrued interest on loans","276840","asset_fixed","","False","Intérêts courus sur prêts"
"pcg_27685","Accrued interest on deposits and sureties","276850","asset_fixed","","False","Intérêts courus sur dépôts et cautionnements"
"pcg_27688","Accrued interest on sundry debts receivable","276880","asset_fixed","","False","Intérêts courus sur créances diverses"
"pcg_2771","Own shares","277100","asset_fixed","","False","Actions propres ou parts propres"
"pcg_2772","Own shares in process of cancellation","277200","asset_fixed","","False","Actions propres ou parts propres en voie d'annulation"
"pcg_278","Merger loss on financial assets","278000","asset_fixed","","False","Mali de fusion sur actifs financiers"
"pcg_279","Unpaid instalments on unpaid long-term investment securities","279000","asset_fixed","","False","Versements restant à effectuer sur titres immobilisés non libérés"
"pcg_2801","Establishment costs (same allocation as for Account 201)","280100","asset_fixed","","False","Frais d'établissement (même ventilation que celle du compte 201)"
"pcg_2803","Research and development costs","280300","asset_fixed","","False","Frais de recherche et de développement"
"pcg_2805","Concessions and similar rights, patents, licences, software, rights and similar assets","280500","asset_fixed","","False","Concessions et droits similaires, brevets, licences, logiciels, droits et valeurs similaires"
"pcg_2806","Tenancy law","280600","asset_fixed","","False","Droit du bail"
"pcg_2807","Goodwill","280700","asset_fixed","","False","Fonds commercial"
"pcg_2808","Other intangible fixed assets","280800","asset_fixed","","False","Dépréciations des autres immobilisations incorporelles"
"pcg_28081","Amortisation of merger loss on intangible assets","280810","asset_fixed","","False","Amortissements du mali de fusion sur actifs incorporels"
"pcg_2812","Site development (same allocation as for Account 212)","281200","asset_fixed","","False","Agencements aménagements de terrains (même ventilation que celle du compte 212)"
"pcg_2813","Constructions (same allocation as for Account 213)","281300","asset_fixed","","False","Constructions (même ventilation que celle du compte 213)"
"pcg_2814","Constructions on third-party site (same allocation as for Account 214)","281400","asset_fixed","","False","Constructions sur sol d'autrui (même ventilation que celle du compte 214)"
"pcg_2815","Installations matériel et outillage industriels (même ventilation que celle du compte 215)","281500","asset_fixed","","False","Installations matériel et outillage industriels (même ventilation que celle du compte 215)"
"pcg_2818","Other tangible fixed assets (same allocation as for Account 218)","281800","asset_fixed","","False","Amortissements des autres immobilisations corporelles (même ventilation que celle du compte 218)"
"pcg_28187","Amortization of merger loss on tangible assets","281870","asset_fixed","","False","Amortissement du mali de fusion sur actifs corporels"
"pcg_282","Depreciation on fixed assets in concession","282000","asset_fixed","","False","Amortissements des immobilisations mises en concession"
"pcg_2901","Establishment costs","290100","asset_fixed","","False","Frais d’établissement"
"pcg_2903","Development costs","290300","asset_fixed","","False","Frais de développement"
"pcg_2905","Trade marks, processes, rights and similar assets","290500","asset_fixed","","False","Marques, procédés, droits et valeurs similaires"
"pcg_2906","Lease premium","290600","asset_fixed","","False","Droit au bail"
"pcg_2907","Goodwill","290700","asset_fixed","","False","Fonds commercial"
"pcg_2908","Other intangible fixed assets","290800","asset_fixed","","False","Dépréciations des autres immobilisations incorporelles"
"pcg_29081","Impairment of merger loss on intangible assets","290810","asset_fixed","","False","Dépréciation du mali de fusion sur actifs incorporels"
"pcg_29187","Impairment of merger loss on tangible assets","291870","asset_fixed","","False","Dépréciation du mali de fusion sur actifs corporels"
"pcg_292","Provisions for diminution in value of fixed assets in concession","292000","asset_fixed","","False","Dépréciations des immobilisations mises en concession"
"pcg_2931","Tangible fixed assets in progress","293100","asset_fixed","","False","Immobilisations corporelles en cours"
"pcg_2932","Intangible fixed assets in progress","293200","asset_fixed","","False","Dépréciations des immobilisations incorporelles en cours"
"pcg_2961","Provisions for depreciation of long-term equity interests","296100","asset_fixed","","False","Provisions pour dépréciation des titres de participation"
"pcg_2962","Equity-valued securities","296200","asset_fixed","","False","Titres évalués par équivalence"
"pcg_2966","Provisions for depreciation of other forms of participation","296600","asset_fixed","","False","Provisions pour dépréciation des autres formes de participation"
"pcg_2967","Provisions for depreciation of debts receivable related to participating interests (same allocation as for Account 267)","296700","asset_fixed","","False","Provisions pour dépréciation des créances rattachées à des participations (même ventilation que celle du compte 267)"
"pcg_2968","Provisions for depreciation of debts receivable related to joint ventures (same allocation as for Account 268)","296800","asset_fixed","","False","Provisions pour dépréciation des créances rattachées à des sociétés en participation (même ventilation que celle du compte 268)"
"pcg_2971","Provisions for depreciation of long-term investment equity securities other than portfolio long-term equity investment securities (same allocation as for Account 271)","297100","asset_fixed","","False","Provisions pour dépréciation des titres immobilisés autres que les titres immobilisés de l'activité de portefeuille - droit de propriété (ventilation : 271)"
"pcg_2972","Provisions for depreciation of long-term investment debt securities (same allocation as for Account 272)","297200","asset_fixed","","False","Provisions pour dépréciation des titres immobilisés - droit de créance (même ventilation que celle du compte 272)"
"pcg_2973","Provisions for depreciation of portfolio long-term investment securities","297300","asset_fixed","","False","Provisions pour dépréciation des titres immobilisés de l'activité de portefeuille"
"pcg_2974","Provisions for depreciation of loans (same allocation as for Account 274)","297400","asset_fixed","","False","Provisions pour dépréciation des prêts (même ventilation que celle du compte 274)"
"pcg_2975","Provisions for depreciation of deposits and sureties advanced (same allocation as for Account 275)","297500","asset_fixed","","False","Provisions pour dépréciation des dépôts et cautionnements versés (même ventilation que celle du compte 275)"
"pcg_2976","Provisions for depreciation of Other debts receivable (same allocation as for Account 276)","297600","asset_fixed","","False","Provisions pour dépréciation des autres créances immobilisées (même ventilation que celle du compte 276)"
"pcg_29787","Impairment of merger loss on financial assets","297870","asset_fixed","","False","Dépréciation du mali de fusion sur actifs financiers"
"pcg_31_account","Raw materials and supplies","310000","asset_current","","False","Matières premières et fournitures"
"pcg_321_account","Consumable materials","321000","asset_current","","False","Matières consommables"
"pcg_3221","Fuels","322100","asset_current","","False","Combustibles"
"pcg_3222","Cleaning products","322200","asset_current","","False","Produits d'entretien"
"pcg_3223","Workshop and factory supplies","322300","asset_current","","False","Fournitures d'atelier et d'usine"
"pcg_3224","Store supplies","322400","asset_current","","False","Fournitures de magasin"
"pcg_3225","Office supplies","322500","asset_current","","False","Fournitures de bureau"
"pcg_3261","Non-returnable packaging","326100","asset_current","","False","Emballages perdus"
"pcg_3265","Unidentifiable recoverable packaging","326500","asset_current","","False","Emballages récupérables non identifiables"
"pcg_3267","Mixed usage packaging","326700","asset_current","","False","Emballages à usage mixte"
"pcg_331_account","Products in progress","331000","asset_current","","False","Produits en cours"
"pcg_335_account","Works in progress","335000","asset_current","","False","Travaux en cours"
"pcg_341_account","Project studies in progress","341000","asset_current","","False","Études en cours"
"pcg_345_account","Supply of services in progress","34000","asset_current","","False","Prestations de services en cours"
"pcg_351_account","Semi-finished products","351000","asset_current","","False","Produits intermédiaires"
"pcg_355_account","Finished products","355000","asset_current","","False","Ventes de produits finis"
"pcg_3581","Waste","358100","asset_current","","False","Déchets"
"pcg_3585","Refuse","358500","asset_current","","False","Rebuts"
"pcg_3586","Recoverable materials","358600","asset_current","","False","Matières de récupération"
"pcg_36","Stocks from fixed assets","360000","asset_current","","False","Stocks provenant d'immobilisations"
"pcg_37_account","Stocks of goods","370000","asset_current","","False","Stocks de marchandises"
"pcg_38","Stocks in transit","380000","asset_current","","False","Stocks en voie d'acheminement"
"pcg_391_account","Provisions for diminution in value of raw materials and supplies","391000","asset_current","","False","Dépréciations des matières premières et fournitures"
"pcg_392_account","Provisions for diminution in value of other consumables","392000","asset_current","","False","Dépréciations des autres approvisionnements"
"pcg_393_account","Provisions for diminution in value of work in progress (goods)","393000","asset_current","","False","Dépréciations des en cours de production de biens"
"pcg_394_account","Provisions for diminution in value of work in progress (services)","394000","asset_current","","False","Dépréciations des en cours de production de services"
"pcg_395_account","Provisions for diminution in value of product stocks","395000","asset_current","","False","Dépréciations des stocks de produits"
"pcg_397_account","Provisions for diminution in value of stocks of goods for resales","397000","asset_current","","False","Dépréciations des stocks de marchandises"
"pcg_400","Suppliers and related accounts","400000","liability_payable","","True","Fournisseurs et comptes rattachés"
"fr_pcg_pay","Suppliers - Purchase of goods and services","401100","liability_payable","","True","Fournisseurs - Achats de biens et prestations de services"
"pcg_4017","Suppliers - Contract performance holdbacks","401700","liability_payable","","True","Fournisseurs - Retenues de garantie"
"pcg_403","Suppliers - Bills payable","403000","liability_payable","","True","Fournisseurs - Effets à payer"
"pcg_4041","Suppliers - Fixed asset purchases","404100","liability_payable","","True","Fournisseurs - Achats d'immobilisations"
"pcg_4047","Fixed asset suppliers - Contract performance holdbacks","404700","liability_payable","","True","Fournisseurs d'immobilisations - Retenues de garantie"
"pcg_405","Fixed asset suppliers - Bills payable","405000","liability_payable","","True","Fournisseurs d'immobilisations - Effets à payer"
"pcg_4081","Suppliers - Invoices outstanding - Suppliers","408100","liability_current","","True","Factures non parvenues - Fournisseurs"
"pcg_4084","Suppliers - Invoices outstanding - Fixed asset suppliers","408400","liability_current","","True","Factures non parvenues - Fournisseurs d'immobilisations"
"pcg_4088","Suppliers - Invoices outstanding - Suppliers - Accrued interest","408800","liability_current","","True","Factures non parvenues - Fournisseurs - Intérêts courus"
"pcg_4091","Suppliers in debit - Payments on account on orders","409100","liability_current","","False","Fournisseurs avance et acomptes versés sur commandes"
"pcg_4096","Suppliers in debit - Debts receivable for returnable packaging andequipment","409600","asset_current","","True","Fournisseurs débiteurs - Créances pour emballages et matériel à rendre"
"pcg_40971","Suppliers in debit - Other debits","409710","asset_current","","True","Fournisseurs débiteurs - Autres avoirs des fournisseurs d'exploitation"
"pcg_40974","Suppliers in debit - Other debits of fixed asset suppliers","409740","asset_current","","True","Fournisseurs débiteurs - Autres avoirs des fournisseurs d'immobilisations"
"pcg_4098","Suppliers in debit - Purchase rebates, discounts, allowances and other outstanding debits","409800","asset_current","","True","Fournisseurs débiteurs - Rabais, remises, ristournes à obtenir et autres avoirs non encore reçus"
"pcg_410","Customers and related accounts","410000","asset_receivable","","True","Clients et comptes rattachés"
"fr_pcg_recv","Customers - Sales of goods or services","411100","asset_receivable","","True","Clients - Ventes de biens ou de prestations de services"
"fr_pcg_recv_pos","Customers - Sales of goods or services (PoS)","411101","asset_receivable","","True","Clients - Ventes de biens ou de prestations de services (PoS)"
"pcg_4117","Customers - Contract performance holdbacks","411700","asset_receivable","","True","Clients - Retenues de garantie"
"pcg_413","Customers - Bills receivable","413000","asset_receivable","","True","Clients - Effets à recevoir"
"pcg_416","Doubtful or contested customer accounts","416000","asset_current","","True","Clients douteux ou litigieux"
"pcg_4181","Customers - Invoices to be made out","418100","asset_current","","True","Clients - Factures à établir"
"pcg_4188","Customers - Accrued interest","418800","asset_current","","True","Clients - Intérêts courus non encore facturés"
"pcg_4191","Customers - Payments on account received on orders","419100","liability_current","","True","Clients créditeurs - Avances et acomptes reçus sur commandes"
"pcg_4196","Customers - Debts payable for returnable packaging and equipment","419600","liability_current","","True","Clients créditeurs - Dettes pour emballages et matériels consignés"
"pcg_4197","Customers - Other credits","419700","liability_current","","True","Clients créditeurs - Autres avoirs"
"pcg_4198","Sales rebates, discounts, allowances and other credits not yet issued","419800","liability_current","","True","Clients créditeurs - Rabais, remises, ristournes à accorder et autres avoirs à établir"
"pcg_421","Personnel - Remuneration payable","421000","liability_current","","True","Personnel - Rémunérations dues"
"pcg_422","Social and Economic Committee","422000","liability_current","","True","Comité social et économique"
"pcg_4246","Employee profit share - Special reserve","424600","liability_current","","True","Participation des salariés aux résultats - Réserve spéciale"
"pcg_4248","Employee profit share - Current accounts","424800","liability_current","","True","Participation des salariés aux résultats - Comptes courants"
"pcg_425","Personnel - Payments on account","425000","asset_current","","True","Personnel - Avances et acomptes"
"pcg_426","Personnel - Deposits","426000","liability_current","","True","Personnel - Dépôts"
"pcg_427","Personnel - Stoppages of payment","427000","liability_current","","True","Personnel - Oppositions"
"pcg_4282","Personnel - Accrued charges payable for holiday pay","428200","liability_current","","True","Personnel - Dettes provisionnées pour congés à payer"
"pcg_4284","Personnel - Accrued charges payable for employee profit share","428400","liability_current","","True","Personnel - Dettes provisionnées pour participation des salariés aux résultats"
"pcg_4286","Personnel - Other accrued charges payable","428600","liability_current","","True","Personnel - Autres charges à payer"
"pcg_431","Social security","431000","liability_current","","True","Sécurité Sociale"
"pcg_437","Other social agencies","437000","liability_current","","True","Autres organismes sociaux"
"pcg_4382","Contributions for holiday pay","438200","liability_current","","True","Charges sociales sur congés à payer"
"pcg_4386","Other accrued charges payable","438600","liability_current","","True","Organismes sociaux - Autres charges à payer"
"pcg_439","Social security - Accrued income","439000","asset_current","","True","Organismes sociaux - Produits à recevoir"
"pcg_441_account","State - Subsidies and grants receivable","441000","asset_current","","True","État - Subventions et aides à recevoir"
"pcg_4421","Withholding tax (Income tax)","442100","liability_current","","True","Prélèvements à la source (Impôt sur le revenu)"
"pcg_4422","Non-liberating flat-rate deductions","442200","liability_current","","True","Prélèvements forfaitaires non libératoires"
"pcg_4423","Deductions and withholdings on distributions","442300","liability_current","","True","Retenues et prélèvements sur les distributions"
"pcg_444","State - Income tax","444000","liability_current","","True","État - Impôts sur les bénéfices"
"pcg_4452","Value added tax due within the European Union","445200","liability_current","","False","TVA due sur acquisitions intracommunautaires"
"pcg_44521","Value added tax to be disbursed","445210","liability_current","","False","TVA due sur prestations intracommunautaires"
"pcg_4453","VAT due on imports (reverse charge)","445300","liability_current","","False","TVA due sur importations (autoliquidation)"
"pcg_44531","VAT due on non-EU supplies","445310","liability_current","","False","TVA due sur prestations hors UE"
"pcg_44551","VAT to be paid","445510","liability_current","","True","TVA à décaisser"
"pcg_44558","Taxes assimilated to VAT","445580","liability_current","","True","Taxes assimilées à la TVA"
"pcg_44562","Deductible VAT on fixed assets","445620","asset_current","","False","TVA déductible sur immobilisations"
"pcg_44563","VAT transferred by other entities","445630","asset_current","","True","TVA transférée par d'autres entités"
"pcg_44564","Deductible VAT on unsettled transactions","445640","asset_current","","True","TVA déductible sur opérations non réglées"
"pcg_44566","Deductible VAT on other goods and services","445660","asset_current","","False","TVA déductible sur autres biens et services"
"pcg_445662","Intra-Community deductible VAT","445662","asset_current","","False","TVA déductible intracommunautaire"
"pcg_445663","Deductible VAT outside the EU (reverse charge)","445663","asset_current","","False","TVA déductible hors UE (autoliquidation)"
"pcg_44567","VAT credit to be carried forward","445670","asset_current","","True","Crédit de TVA à reporter"
"pcg_445671","VAT claimed from the administration","445671","asset_current","","True","TVA réclamée à l'administration"
"pcg_44568","Deductible taxes assimilated to VAT","445680","asset_current","","True","Taxes déductibles assimilées à la TVA"
"pcg_44571","VAT collected","445710","liability_current","","False","TVA collectée"
"pcg_44574","VAT collected on unsettled transactions","445740","liability_current","","False","TVA collectée sur opérations non réglées"
"pcg_44578","Taxes collected as VAT","445780","liability_current","","True","Taxes collectées assimilées à la TVA"
"pcg_445800","Turnover taxes to be regularised or pending","445800","liability_current","","True","Taxes sur le chiffre d'affaires à régulariser ou en attente"
"pcg_44581","Advance payments - Simplified taxation system","445810","asset_current","","True","Acomptes - Régime simplifié d'imposition"
"pcg_44583","Turnover tax refunds claimed","445830","asset_current","","True","Remboursement de taxes sur le chiffre d'affaires demandé"
"pcg_44584","Prepaid VAT","445840","liability_current","","True","TVA récupérée d'avance"
"pcg_44586","Turnover taxes on non-received invoices","445860","asset_current","","True","Taxes sur le chiffre d'affaires sur factures non parvenues"
"pcg_44587","Turnover taxes on invoices to be issued","445870","liability_current","","True","Taxes sur le chiffre d'affaires sur factures à établir"
"pcg_446","Guaranteed bonds","446000","liability_current","","True","Obligations cautionnées"
"pcg_447","Other taxes, levies and similar payments","447000","liability_current","","True","Autres impôts, taxes et versements assimilés"
"pcg_4481","State - Accrued charges payable","448100","liability_current","","True","État - Charges à payer"
"pcg_44811","Tax on holiday pay","448110","liability_current","","True","Charges fiscales sur congés à payer"
"pcg_4412","Accrued expenses","448120","asset_current","","True","Charges à payer"
"pcg_4482","State - Accrued income receivable","448200","asset_current","","True","État - Produits à recevoir"
"pcg_449","Emission allowances to be surrendered to the State","449000","liability_current","","True","Quotas d'émission à restituer à l'État"
"pcg_451","Group","451000","liability_current","","True","Groupe"
"pcg_4551","Partners/associates - Current accounts - Principal","455100","liability_current","","True","Associés - Comptes courants - Principal"
"pcg_4558","Partners/associates - Current accounts - Accrued interest","455800","liability_current","","True","Associés - Comptes courants - Intérêts courus"
"pcg_45611","Partners/associates - Capital transactions - Contributions in kind","456110","asset_current","","True","Associés - Comptes d'apport en société - Apports en nature"
"pcg_45615","Partners/associates - Capital transactions - Contributions in money","456150","asset_current","","True","Associés - Comptes d'apport en société - Apports en numéraire"
"pcg_45621","Shareholders - Subscribed capital calledup, unpaid","456210","asset_current","","True","Actionnaires - Capital souscrit et appelé, non versé"
"pcg_45625","Partners/associates - Capital called up, unpaid","456250","asset_current","","True","Associés - Capital appelé, non versé"
"pcg_4563","Partners/associates - Payments received for capital increase","456300","asset_current","","True","Associés - Versements reçus sur augmentation de capital"
"pcg_4564","Partners/associates - Advance payments","456400","asset_current","","True","Associés - Versements anticipés"
"pcg_4566","Defaulting shareholders","456600","asset_current","","True","Actionnaires défaillants"
"pcg_4567","Partners/associates - Capital to be reimbursed","456700","asset_current","","True","Associés - Capital à rembourser"
"pcg_457","Partners/associates - Dividends payable","457000","liability_current","","True","Associés - Dividendes à payer"
"pcg_4581","Partners/associates - Joint and Economic Interest Group transaction - Current transactions","458100","asset_current","","True","Associés - Opérations faites en commun et en GIE - Opérations courantes"
"pcg_4588","Partners/associates - Joint and Economic Interest Group transaction - Accrued interest","458800","asset_current","","True","Associés - Opérations faites en commun et en GIE - Intérêts courus"
"pcg_462","Debts receivable on realisation of fixed assets","462000","asset_current","","True","Créances sur cessions d'immobilisations"
"pcg_464","Debts payable on purchases of short-term investment securities","464000","liability_current","","True","Dettes sur acquisitions de valeurs mobilières de placement"
"pcg_465","Debts receivable on realisation of short-term investment securities","465000","asset_current","","True","Créances sur cessions de valeurs mobilières de placement"
"pcg_467","Other accounts receivable and accrued income","467000","liability_current","","True","Divers comptes débiteurs et produits à recevoir"
"pcg_468_account","Other accounts payable and accrued liabilities","468000","liability_current","","False","Divers comptes créditeurs et charges à payer"
"pcg_471","Suspense accounts","471000","asset_current","","True","Compte d'attente"
"pcg_472","Suspense accounts","472000","asset_current","","True","Compte d'attente"
"pcg_473","Suspense accounts","473000","asset_current","","True","Compte d'attente"
"pcg_4741","Valuation differences on forward financial instruments - Assets","474100","asset_current","","True","Différences d'évaluation sur instruments financiers à terme - Actif"
"pcg_4742","Valuation difference on tokens held - Assets","474200","asset_current","","True","Différences d'évaluation sur jetons détenus - Actif"
"pcg_4746","Valuation differences of tokens on liabilities - Assets","474600","asset_current","","True","Différences d’évaluation de jetons sur des passifs - Actif"
"pcg_4751","Valuation differences on forward financial instruments - Liabilities","475100","liability_current","","True","Différences d'évaluation sur instruments financiers à terme - Passif"
"pcg_4752","Valuation differences on tokens held - Liabilities","475200","liability_current","","True","Différences d'évaluation sur jetons détenus - Passif"
"pcg_4756","Valuation differences of tokens on liabilities - Liabilities","475600","liability_current","","True","Différences d’évaluation de jetons sur des passifs - Passif"
"pcg_4761","Decrease in receivables","476100","asset_current","","True","Diminution des créances"
"pcg_4762","Increase in liabilities","476200","asset_current","","True","Augmentation des dettes"
"pcg_4768","Differences offset by currency hedging","476800","asset_current","","True","Différences compensées par couverture de change"
"pcg_4771","Increase in receivables","477100","liability_current","","True","Augmentation des créances"
"pcg_4772","Decrease in liabilities","477200","liability_current","","True","Diminution des dettes"
"pcg_4778","Differences offset by currency hedging","477800","liability_current","","True","Différences compensées par couverture de change"
"pcg_478","Other transitory accounts","478000","liability_current","","False","Autres comptes transitoires"
"pcg_4781","Merger loss on current assets","478100","liability_current","","False","Mali de fusion sur actif circulant"
"pcg_481_account","Debt issuance costs","481000","asset_current","","True","Frais d’émission des emprunts"
"pcg_486","Prepayments","486000","asset_current","","True","Charges constatées d'avance"
"pcg_487","Deferred income","487000","liability_current","","True","Produits constatés d'avance"
"pcg_4886","Periodic load balancing accounts","488600","asset_current","","True","Comptes de répartition périodique des charges"
"pcg_4887","Periodic Revenue Allocation Accounts","488700","liability_current","","True","Comptes de répartition périodique des produits"
"pcg_491","Provisions for impairment of trade receivables","491000","asset_current","","True","Provisions pour dépréciation des comptes de clients"
"pcg_4951","Provisions for impairment of group accounts","495100","asset_current","","True","Provisions pour dépréciation des comptes du groupe"
"pcg_4955","Provisions for depreciation of partners' current accounts","495500","asset_current","","True","Provisions pour dépréciation des comptes courants des associés"
"pcg_4958","Provisions for depreciation of joint and EIG operations","495800","asset_current","","True","Provisions pour dépréciation des opérations faites en commun et en GIE"
"pcg_4962","Provisions for impairment of receivables on disposals of fixed assets","496200","asset_current","","True","Provisions pour dépréciation des créances sur cessions d'immobilisations"
"pcg_4965","Provisions for impairment of receivables on sales of marketable securities","496500","asset_current","","True","Provisions pour dépréciation des créances sur cessions de valeurs mobilières de placement"
"pcg_4967","Provisions for depreciation - Other accounts receivable","496700","asset_current","","True","Provisions pour dépréciation - Autres comptes débiteurs"
"pcg_502","Marketable securities - Own shares","502000","asset_cash","","False","Valeurs mobilières de placement - Actions propres"
"pcg_5021","Shares to be allocated to employees and assigned to specific plans","502100","asset_cash","","False","Actons destinées à être attribuées aux employés et affectées à des plans déterminés"
"pcg_5022","Shares available for allocation to employees or for stock market adjustment","502200","asset_cash","","False","Actons disponibles pour être attribuées aux employés ou pour la régularisation des cours de la bourse"
"pcg_5031","Marketable securities - Listed securities","503100","asset_cash","","False","Valeurs mobilières de placement - Titres cotés"
"pcg_5035","Marketable securities - Unlisted securities","503500","asset_cash","","False","Valeurs mobilières de placement - Titres non cotés"
"pcg_504","Marketable securities - Other securities conferring a right of ownership","504000","asset_cash","","False","Valeurs mobilières de placement - Autres titres conférant un droit de propriété"
"pcg_505","Bonds and notes issued by the company and redeemed by it","505000","asset_cash","","False","Obligations et bons émis par la société et rachetés par elle"
"pcg_5061","Quoted bonds","506100","asset_cash","","False","Obligations cotés"
"pcg_5065","Unquoted bonds","506500","asset_cash","","False","Obligations non cotés"
"pcg_507","Treasury bills and short-term notes","507000","asset_cash","","False","Bons du Trésor et bons de caisse à court terme"
"pcg_5081","Other securities","508100","asset_cash","","False","Autres valeurs mobilières de placement"
"pcg_5082","Equity and bond warrants","508200","asset_cash","","False","Bons de souscription"
"pcg_5088","Accrued interest on bonds, warrants and similar securities","508800","asset_cash","","False","Intérêts courus sur obligations, bons et valeurs assimilées"
"pcg_509","Unpaid instalments on unpaid short-term investment securities","509000","asset_cash","","False","Versements restant à effectuer sur valeurs mobilières de placement non libérées"
"pcg_5111","Outstanding coupons for collection","511100","asset_cash","","False","Coupons échus à l'encaissement"
"pcg_5112","Cheques for collection","511200","asset_current","","True","Chèques à encaisser"
"pcg_5113","Bills for collection","511300","asset_cash","","False","Effets à l'encaissement"
"pcg_5114","Bills for discount","511400","asset_cash","","False","Effets à l'escompte"
"pcg_5121","Banks - Accounts in euros","512100","asset_cash","","False","Comptes en euros"
"pcg_5124","Banks - Accounts in foreign currencies","512400","asset_cash","","False","Banques - Comptes en devises"
"pcg_517","Other financial bodies","517000","asset_cash","","False","Autres organismes financiers"
"pcg_5181","Accrued interest payable","518100","asset_cash","","False","Intérêts courus à payer"
"pcg_5188","Accrued interest receivable","518800","asset_cash","","False","Intérêts courus à recevoir"
"pcg_5191","Current bank advances - Credit for assignment of commercial debts receivable","519100","asset_cash","","False","Concours bancaires courants - Crédit de mobilisation de créances commerciales (CMCC)"
"pcg_5193","Current bank advances - Assignment of debts receivable originating outside France","519300","asset_cash","","False","Concours bancaires courants - Mobilisation de créances nées à l'étranger"
"pcg_5198","Current bank advances - Accrued interest on current bank advances","519800","asset_cash","","False","Concours bancaires courants - Intérêts courus sur concours bancaires courants"
"pcg_52","Financial futures instruments and tokens held","520000","asset_cash","","False","Instruments financiers à terme et jetons détenus"
"pcg_53_account","Cashier","530000","asset_cash","","False","Caisse"
"pcg_58_account","Internal transfers","580000","asset_cash","","False","Virements internes"
"pcg_5903","Provisions for impairment of shares","590300","asset_current","","False","Provisions pour dépréciation des actions"
"pcg_5904","Provisions for depreciation of other securities conferring a right of ownership","590400","asset_current","","False","Provisions pour dépréciation des autres titres conférant un droit de propriété"
"pcg_5906","Provisions for impairment of bonds","590600","asset_current","","False","Provisions pour dépréciation des obligations"
"pcg_5908","Provisions for depreciation of other marketable securities and similar receivables (provisions)","590800","asset_current","","False","Provisions pour dépréciation des autres valeurs mobilières de placement et créances assimilées (provisions)"
"pcg_601_account","Inventory item purchases - Raw materials and supplies","601000","expense","account.account_tag_operating","False","Achats stockés - Matières premières et fournitures"
"pcg_6021_account","Inventory item purchases - Consumable Materials","602100","expense","account.account_tag_operating","False","Achats stockés - Matières consommables"
"pcg_60221","Inventory item purchases - Fuels","602210","expense","account.account_tag_operating","False","Achats stockés - Combustibles"
"pcg_60222","Inventory item purchases - Maintenance products","602220","expense","account.account_tag_operating","False","Achats stockés - Produits d'entretien"
"pcg_60223","Inventory item purchases - Workshop and factory supplies","602230","expense","account.account_tag_operating","False","Achats stockés - Fournitures d'atelier et d'usine"
"pcg_60224","Inventory item purchases - Store supplies","602240","expense","account.account_tag_operating","False","Achats stockés - Fournitures de magasin"
"pcg_60225","Inventory item purchases - Office supplies","602250","expense","account.account_tag_operating","False","Achats stockés - Fournitures de bureau"
"pcg_60261","Inventory item purchases - Non-returnable packaging","602610","expense","account.account_tag_operating","False","Achats stockés - Emballages perdus"
"pcg_60262","Inventory item purchases - Packaging waste","602620","expense","account.account_tag_operating","False","Achats stockés - Malis sur emballages"
"pcg_60265","Inventory item purchases - Unidentifiable recoverable packaging","602650","expense","account.account_tag_operating","False","Achats stockés - Emballages récupérables non identifiables"
"pcg_60267","Inventory item purchases - Mixed usage packaging","602670","expense","account.account_tag_operating","False","Achats stockés - Emballages à usage mixte"
"pcg_6031","Change in stocks of raw materials (and supplies)","603100","expense","account.account_tag_operating","False","Variation des stocks de matières premières (et fournitures)"
"pcg_6032","Change in stocks of other supplies","603200","expense","account.account_tag_operating","False","Variation des stocks des autres approvisionnements"
"pcg_6037","Change in inventories of goods","603700","expense","account.account_tag_operating","False","Variation des stocks de marchandises"
"pcg_604","Purchases of project studies and services","604000","expense","account.account_tag_operating","False","Achats d'études et prestations de services"
"pcg_605","Purchases of equipment, facilities and works","605000","expense","account.account_tag_operating","False","Achats de matériel équipements et travaux"
"pcg_6061","Non-inventoriable supplies (eg. water, energy)","606100","expense","account.account_tag_operating","False","Fournitures non stockables (eau, énergie...)"
"pcg_6063","Maintenance and minor equipment supplies","606300","expense","account.account_tag_operating","False","Fournitures d'entretien et de petit équipement"
"pcg_6064","Administrative supplies","606400","expense","account.account_tag_operating","False","Fournitures administratives"
"pcg_6068","Other materials and supplies","606800","expense","account.account_tag_operating","False","Achats autres matières et fournitures"
"pcg_607_account","Purchase of goods","607000","expense","account.account_tag_operating","False","Achats de marchandises"
"pcg_608","Incidental costs included in purchases","608000","expense","","False","Frais accessoires incorporés aux achats"
"pcg_6091","Discounts, rebates and discounts - Inventory item purchases - Raw materials (and supplies)","609100","expense","account.account_tag_operating","False","Achats stockés matières premières (et fournitures)"
"pcg_6092","Discounts, rebates and discounts - Inventory item purchases - Other inventory item consumables","609200","expense","account.account_tag_operating","False","Rabais, remises et ristournes obtenus sur achats d'autres approvisionnements stockés"
"pcg_6094","Discounts, rebates and discounts - Inventory item purchases - Project studies and services supplied","609400","expense","account.account_tag_operating","False","Rabais, remises et ristournes obtenus sur achats d'études et prestations de services"
"pcg_6095","Discounts, rebates and discounts - Inventory item purchases - Equipment, facilities and works","609500","expense","account.account_tag_operating","False","Rabais, remises et ristournes obtenus sur achats de matériel, équipements et travaux"
"pcg_6096","Discounts, rebates and discounts - Inventory item purchases - Non-inventory consumables","609600","expense","account.account_tag_operating","False","Rabais, remises et ristournes obtenus sur achats d'approvisionnements non stockés"
"pcg_6097","Discounts, rebates and discounts - Inventory item purchases - Goods for resale","609700","expense","account.account_tag_operating","False","Rabais, remises et ristournes obtenus sur achats de marchandises"
"pcg_6098","Unallocated rebates, discounts, allowances","609800","expense","account.account_tag_operating","False","Rabais, remises et ristournes non affectés"
"pcg_611","General subcontracting","611000","expense","account.account_tag_operating","False","Sous-traitance générale"
"pcg_6122","Movable property leases","612200","expense","account.account_tag_operating","False","Redevances de crédit-bail mobilier"
"pcg_6125","Real property leases","612500","expense","account.account_tag_operating","False","Redevances de crédit-bail immobilier"
"pcg_6132","Real property rental","613200","expense","account.account_tag_operating","False","Locations immobilières"
"pcg_6135","Movable property rental","613500","expense","account.account_tag_operating","False","Locations mobilières"
"pcg_614","Rental and joint ownership property costs","614000","expense","account.account_tag_operating","False","Charges locatives et de copropriété"
"pcg_6152","Maintenance and repairs on real property items","615200","expense","account.account_tag_operating","False","Entretien et réparations sur biens immobiliers"
"pcg_6155","Maintenance and repairs on movable property items","615500","expense","account.account_tag_operating","False","Entretien et réparations sur biens mobiliers"
"pcg_6156","Maintenance","615600","expense","account.account_tag_operating","False","Maintenance"
"pcg_6161","Comprehensive risk","616100","expense","account.account_tag_operating","False","Assurance multirisques"
"pcg_6162","Compulsory construction loss insurance","616200","expense","account.account_tag_operating","False","Assurance obligatoire dommage construction"
"pcg_61636","Transport insurance on purchases","616360","expense","account.account_tag_operating","False","Assurance transport sur achats"
"pcg_61637","Transport insurance on sales","616370","expense","account.account_tag_operating","False","Assurance transport sur ventes"
"pcg_61638","Transport insurance on other items","616380","expense","account.account_tag_operating","False","Assurance transport sur autres biens"
"pcg_6164","Operating risks","616400","expense","account.account_tag_operating","False","Assurance risques d'exploitation"
"pcg_6165","Customer insolvency","616500","expense","account.account_tag_operating","False","Assurance insolvabilité clients"
"pcg_617","Project studies, surveys, assessments","617000","expense","account.account_tag_operating","False","Études et recherches"
"pcg_6181","General documentation","618100","expense","account.account_tag_operating","False","Documentation générale"
"pcg_6183","Technical documentation","618300","expense","account.account_tag_operating","False","Documentation technique"
"pcg_6185","Colloquium, seminar, conference costs","618500","expense","account.account_tag_operating","False","Frais de colloques, séminaires, conférences"
"pcg_619","Purchase rebates, discounts, allowances on external services","619000","expense","account.account_tag_operating","False","Rabais, remises et ristournes obtenus sur services extérieurs"
"pcg_6211","Temporary personnel","621100","expense","account.account_tag_operating","False","Personnel intérimaire"
"pcg_6214","Personnel on secondment or loan to the entity","621400","expense","account.account_tag_operating","False","Personnel détaché ou prêté à l'entreprise"
"pcg_6221","Purchase commission and brokerage","622100","expense","account.account_tag_operating","False","Commissions et courtages sur achats"
"pcg_6222","Sales commission and brokerage","622200","expense","account.account_tag_operating","False","Commissions et courtages sur ventes"
"pcg_6224","Payments to forwarding agents","622400","expense","account.account_tag_operating","False","Rémunérations des transitaires"
"pcg_6225","Payments for factoring","622500","expense","account.account_tag_operating","False","Rémunérations d'affacturage"
"pcg_6226","Fees","622600","expense","account.account_tag_operating","False","Honoraires"
"pcg_6227","Legal and litigation fees","622700","expense","account.account_tag_operating","False","Frais d'actes et de contentieux"
"pcg_6228","Remuneration of intermediaries and fees - Sundry","622800","expense","account.account_tag_operating","False","Rémunérations d'intermédiaires et honoraires - Divers"
"pcg_6231","Announcements and advertisements","623100","expense","account.account_tag_operating","False","Annonces et insertions"
"pcg_6232","Samples","623200","expense","account.account_tag_operating","False","Échantillons"
"pcg_6233","Fairs and exhibitions","623300","expense","account.account_tag_operating","False","Foires et expositions"
"pcg_6234","Gifts to customers","623400","expense","account.account_tag_operating","False","Cadeaux à la clientèle"
"pcg_6235","Premiums","623500","expense","account.account_tag_operating","False","Primes"
"pcg_6236","Catalogues and printed material","623600","expense","account.account_tag_operating","False","Catalogues et imprimés"
"pcg_6237","Publications","623700","expense","account.account_tag_operating","False","Publications"
"pcg_6238","Miscellaneous (tips, regular donations)","623800","expense","account.account_tag_operating","False","Divers (pourboires, dons courants)"
"pcg_6241","Transport on purchases","624100","expense","account.account_tag_operating","False","Transports sur achats"
"pcg_6242","Transport on sales","624200","expense","account.account_tag_operating","False","Transports sur ventes"
"pcg_6243","Transport between establishments or work sites","624300","expense","account.account_tag_operating","False","Transports entre établissements ou chantiers"
"pcg_6244","Administrative transport","624400","expense","account.account_tag_operating","False","Transports administratifs"
"pcg_6247","Collective staff transport","624700","expense","account.account_tag_operating","False","Transports collectifs du personnel"
"pcg_6248","Various transport","624800","expense","account.account_tag_operating","False","Transports divers"
"pcg_6251","Travels and journeys","625100","expense","account.account_tag_operating","False","Voyages et déplacements"
"pcg_6255","Relocation costs","625500","expense","account.account_tag_operating","False","Frais de déménagement"
"pcg_6256","Missions","625600","expense","account.account_tag_operating","False","Missions"
"pcg_6257","Receptions","625700","expense","account.account_tag_operating","False","Réceptions"
"pcg_626","Postal and telecommunication costs","626000","expense","account.account_tag_operating","False","Frais postaux et frais de télécommunications"
"pcg_6271","Securities costs (purchase, sale, safe custody)","627100","expense","account.account_tag_operating","False","Frais sur titres (achat, vente, garde)"
"pcg_6272","Commissions and loan issue costs","627200","expense","account.account_tag_operating","False","Commissions et frais sur émission d'emprunts"
"pcg_6275","Charges on bills","627500","expense","account.account_tag_operating","False","Frais sur effets"
"pcg_6276","Rental of safes","627600","expense","account.account_tag_operating","False","Location de coffres"
"pcg_6278","Other expenses and commissions on services supplied","627800","expense","account.account_tag_operating","False","Autres frais et commissions sur prestations de services"
"pcg_6281","Sundry assistance (eg. contributions)","628100","expense","account.account_tag_operating","False","Concours divers (cotisations...)"
"pcg_6284","Personnel recruitment costs","628400","expense","account.account_tag_operating","False","Frais de recrutement de personnel"
"pcg_629","Purchase rebates, discounts, allowances on other external services","629000","expense","account.account_tag_operating","False","Rabais, remises et ristournes obtenus sur autres services extérieurs"
"pcg_6311","Tax on salaries","631100","expense","account.account_tag_operating","False","Taxe sur les salaires"
"pcg_6314","Default contribution for compulsory investment in construction","631400","expense","account.account_tag_operating","False","Cotisation pour défaut d'investissement obligatoire dans la construction"
"pcg_6318","Other taxes and similar payments on remuneration (tax authorities)","631800","expense","account.account_tag_operating","False","Autres impôts, taxes et versements assimilés sur rémunérations (administrations des impôts)"
"pcg_6331","Transport expenditures","633100","expense","account.account_tag_operating","False","Versement de transport"
"pcg_6332","Accommodation allowances","633200","expense","account.account_tag_operating","False","Allocation logement"
"pcg_6333","Employers' contribution to professional training","633300","expense","account.account_tag_operating","False","Contribution unique des employeurs à la formation professionnelle"
"pcg_6334","Employer participation in construction projects","633400","expense","account.account_tag_operating","False","Participation des employeurs à l'effort de construction"
"pcg_6335","Discharge payments entitling exemption from apprenticeship tax","633500","expense","account.account_tag_operating","False","Versements libératoires ouvrant droit à l'exonération de la taxe d'apprentissage"
"pcg_6338","Other taxes and similar payments on salaries (other bodies)","633800","expense","account.account_tag_operating","False","Autres impôts, taxes et versements assimilés sur rémunérations (autres organismes)"
"pcg_63511","Territorial economic contribution","635110","expense","","False","Contribution économique territoriale"
"pcg_63512","Property taxes","635120","expense","account.account_tag_operating","False","Taxes foncières"
"pcg_63513","Other local rates and taxes","635130","expense","account.account_tag_operating","False","Autres impôts locaux"
"pcg_63514","Tax on company vehicles","635140","expense","account.account_tag_operating","False","Taxe sur les véhicules des sociétés"
"pcg_6352","Non-recoverable turnover tax","635200","expense","account.account_tag_operating","False","Taxes sur le chiffre d'affaires non récupérables"
"pcg_6353","Indirect taxes","635300","expense","account.account_tag_operating","False","Impôts indirects"
"pcg_63541","Transfer duty","635410","expense","account.account_tag_operating","False","Droits de mutation"
"pcg_6358","Other duties","635800","expense","account.account_tag_operating","False","Autres droits"
"pcg_6371","Social solidarity contribution chargeable to companies","637100","expense","account.account_tag_operating","False","Contribution sociale de solidarité à la charge des sociétés"
"pcg_6372","Taxes collected by international public bodies","637200","expense","account.account_tag_operating","False","Taxes perçues par les organismes publics internationaux"
"pcg_6374","Taxes and levies due for payment outside France","637400","expense","account.account_tag_operating","False","Impôts et taxes exigibles à l'étranger"
"pcg_6378","Sundry taxes","637800","expense","account.account_tag_operating","False","Taxes diverses (autres organismes)"
"pcg_638","Tax reminder (other than income tax)","638000","expense","account.account_tag_operating","False","Rappel d’impôts (autres qu’impôts sur les bénéfices)"
"pcg_6411","Salaries, emoluments","641100","expense","account.account_tag_operating","False","Salaires et appointements"
"pcg_6412","Holiday pay","641200","expense","account.account_tag_operating","False","Congés payés"
"pcg_6413","Premiums and bonuses","641300","expense","account.account_tag_operating","False","Primes et gratifications"
"pcg_6414","Allowances and sundry benefits","641400","expense","account.account_tag_operating","False","Indemnités et avantages divers"
"pcg_6415","Family income supplement","641500","expense","account.account_tag_operating","False","Supplément familial"
"pcg_644","Owner remuneration","644000","expense","account.account_tag_operating","False","Rémunération du travail de l'exploitant"
"pcg_6451","Social Security Collection Office (URSSAF) contributions","645100","expense","account.account_tag_operating","False","Cotisations à l'URSSAF"
"pcg_6452","Mutual organisation contributions","645200","expense","account.account_tag_operating","False","Cotisations aux mutuelles"
"pcg_6453","Pension fund contributions","645300","expense","account.account_tag_operating","False","Cotisations aux caisses de retraites"
"pcg_6454","Contributions to Pôle emploi","645400","expense","account.account_tag_operating","False","Cotisations à Pôle emploi"
"pcg_6458","Contributions to other social agencies","645800","expense","account.account_tag_operating","False","Cotisations aux autres organismes sociaux"
"pcg_646","Owner social security contributions","646000","expense","account.account_tag_operating","False","Cotisations sociales personnelles de l'exploitant"
"pcg_6471","Direct allowances","647100","expense","account.account_tag_operating","False","Prestations directes"
"pcg_6472","Payments to the social and economic committee","647200","expense","account.account_tag_operating","False","Versements au comité social et économique"
"pcg_6474","Payments to other company benefit schemes","647400","expense","account.account_tag_operating","False","Versements aux autres oeuvres sociales"
"pcg_6475","Occupational medicine, pharmacy","647500","expense","account.account_tag_operating","False","Médecine du travail, pharmacie"
"pcg_648","Other personnel costs","648000","expense","account.account_tag_operating","False","Autres charges de personnel"
"pcg_649","Reimbursement of staff costs (if staff costs are reimbursed)","649000","expense","account.account_tag_operating","False","Remboursements de charges de personnel (si rembourement de personnel)"
"pcg_6511","Concessions, patents, licences, trade marks, processes, software","651100","expense","account.account_tag_operating","False","Redevances pour concessions brevets, licences, marques, procédés, logiciels"
"pcg_6516","Author and reproduction royalties","651600","expense","account.account_tag_operating","False","Droits d'auteur et de reproduction"
"pcg_6518","Other royalties and similar assets","651800","expense","account.account_tag_operating","False","Redevances pour autres droits et valeurs similaires"
"pcg_653","Directors‘ and executive officers’ remuneration","653000","expense","account.account_tag_operating","False","Rémunérations de l’activité des administrateurs et des gérants"
"pcg_6541","Debts receivable for the financial year","654100","expense","account.account_tag_operating","False","Créances de l'exercice"
"pcg_6544","Debts receivable for previous financial years","654400","expense","account.account_tag_operating","False","Créances des exercices antérieurs"
"pcg_6551","Share of profit transferred (accounts of the managing entity)","655100","expense","","False","Quote-part de bénéfice transférée (comptabilité du gérant)"
"pcg_6555","Share of loss (accounts of non-managing partners/associates)","655500","expense","","False","Quote-part de perte supportée (comptabilité des associés non gérants)"
"pcg_657","Book value of intangible assets and property, plant and equipment sold","657000","expense","account.account_tag_investing","False","Valeurs comptables des immobilisations incorporelles et corporelles cédées"
"pcg_6581","Contract penalties (and discounts paid on purchases and sales)","658100","expense","account.account_tag_operating","False","Pénalités sur marchés (et dédits payés sur achats et ventes)"
"pcg_6582","Penalties, tax and criminal fines","658200","expense","account.account_tag_operating","False","Pénalités, amendes fiscales et pénales"
"pcg_6583","Losses resulting from indexation clauses","658300","expense","account.account_tag_operating","False","Malis provenant de clauses d'indexation"
"pcg_6584","Lots","658400","expense","account.account_tag_operating","False","Lots"
"pcg_6588","Setting up or winding up trusts","658800","expense","","False","Opérations de constitution ou liquidation des fiducies"
"pcg_66116","Loans and similar debts payable","661160","expense","account.account_tag_financing","False","Emprunts et dettes assimilées"
"pcg_66117","Debts payable related to participating interests","661170","expense","account.account_tag_financing","False","Dettes rattachées à des participations"
"pcg_6612","Trust expenses, result for the period","661200","expense","","False","Charges de la fiducie, résultat de la période"
"pcg_6615","Current account and credit deposit interest","661500","expense","account.account_tag_financing","False","Intérêts des comptes courants et des dépôts créditeurs"
"pcg_6616","Bank and financing transaction interest (eg. discounting)","661600","expense","account.account_tag_financing","False","Intérêts bancaires et sur opérations de financement (escompte, ...)"
"pcg_6617","Interest on guaranteed bonds","661700","expense","account.account_tag_financing","False","Intérêts des obligations cautionnées"
"pcg_66181","Commercial debts payable","661810","expense","account.account_tag_financing","False","Intérêts des dettes commerciales"
"pcg_66188","Sundry debts payable","661880","expense","account.account_tag_financing","False","Intérêts des dettes diverses"
"pcg_664","Losses on debts receivable related to participating interests","664000","expense","account.account_tag_financing","False","Pertes sur créances liées à des participations"
"pcg_665","Discounts allowed","665000","expense","account.account_tag_financing","False","Escomptes accordés"
"pcg_666","Exchange losses","666000","expense","account.account_tag_financing","False","Pertes de change"
"pcg_6671","Book value of financial assets sold","667100","expense","account.account_tag_financing","False","Valeurs comptables des immobilisations financières cédées"
"pcg_6672","Net expenses on disposals of portfolio securities","667200","expense","account.account_tag_financing","False","Charges nettes sur cessions de titres immobilisés de l’activité de portefeuille"
"pcg_6673","Net expenses on disposals of marketable securities","667300","expense","account.account_tag_financing","False","Charges nettes sur cessions de valeurs mobilières de placement"
"pcg_6674","Net expenses on disposal of tokens","667400","expense","account.account_tag_financing","False","Charges nettes sur cessions de jetons"
"pcg_668","Other financial charges","668000","expense","account.account_tag_financing","False","Autres charges financières"
"pcg_6683","Losses arising from the repurchase by the entity of shares and bonds issued by itself","668300","expense","account.account_tag_investing","False","Mali provenant du rachat par l’entité d’actions et obligations émises par elle-même"
"pcg_669","Transfers of financial expenses","669000","expense","account.account_tag_financing","False","Transferts de charges financières"
"pcg_672","Extraordinary expenses on previous years (during the year only)","672000","expense","","False","Charges exceptionnelles sur exercices antérieurs (en cours d'exercice seulement)"
"pcg_678_account","Other exceptional expenses","678000","expense","account.account_tag_investing","False","Autres charges exceptionnelles"
"pcg_6811","Amortisation of intangible and tangible fixed assets","681100","expense","","False","Dotations aux amortissements sur immobilisations incorporelles et corporelles"
"pcg_68111","Intangible fixed assets and formation expenses","681110","expense","account.account_tag_operating","False","Immobilisations incorporelles et frais d’établissement"
"pcg_68112","Appropriations to depreciation on Tangible fixed assets","681120","expense","account.account_tag_operating","False","Dotations aux amortissements sur immobilisations corporelles"
"pcg_6815","Appropriations to provisions for operating liabilities and charges","681500","expense","account.account_tag_operating","False","Dotations aux provisions pour risques et charges d'exploitation"
"pcg_68161","Impairment of intangible assets","681610","expense","account.account_tag_operating","False","Dotations aux dépréciations des immobilisations incorporelles"
"pcg_68162","Impairment of property, plant and equipment","681620","expense","account.account_tag_operating","False","Dotations aux dépréciations des immobilisations corporelles"
"pcg_68173","Impairment of inventories and work in progress","681730","expense","account.account_tag_operating","False","Dotations aux dépréciations des stocks et en-cours"
"pcg_68174","Impairment of receivables","681740","expense","account.account_tag_operating","False","Dotations aux dépréciations des créances"
"pcg_6861","Appropriations to amortisation of premiums on redemption of debt securities","686100","expense","account.account_tag_financing","False","Dotations aux amortissements des primes de remboursement des obligations"
"pcg_6862","Depreciation of loan issue expenses","686200","expense","account.account_tag_operating","False","Dotations aux amortissements des frais d'émission des emprunts"
"pcg_6865","Appropriations to provisions for financial liabilities and charges","686500","expense","account.account_tag_financing","False","Dotations aux provisions pour risques et charges financiers"
"pcg_68662","Appropriations to provisions for diminution in value of financial components - Financial fixed assets","686620","expense","account.account_tag_financing","False","Dotations aux dépréciations des immobilisations financières"
"pcg_68665","Appropriations to provisions for diminution in value of financial components - Short-term investment securities","686650","expense","account.account_tag_financing","False","Dotations aux dépréciations des valeurs mobilières de placement"
"pcg_6871","Appropriations to extraordinary fixed asset depreciation","687100","expense","account.account_tag_financing","False","Dotations aux amortissements exceptionnels des immobilisations"
"pcg_68725","Appropriations to tax-regulated provisions (fixed assets) - Depreciation by derogation","687250","expense","account.account_tag_financing","False","Dotations aux provisions réglementées exceptionnelles (immobilisations) - Amortissements dérogatoires"
"pcg_6873","Appropriations to tax-regulated provisions (stocks)","687300","expense","account.account_tag_financing","False","Dotations aux provisions réglementées exceptionnelles (stocks)"
"pcg_6874","Appropriations to other tax-regulated provisions","687400","expense","account.account_tag_financing","False","Dotations aux autres provisions réglementées exceptionnelles"
"pcg_6875","Appropriations to provisions for extraordinary liabilities and charges","687500","expense","account.account_tag_financing","False","Dotations aux provisions exceptionnelles"
"pcg_6876","Appropriations to provisions for extraordinary diminutionin value","687600","expense","account.account_tag_financing","False","Dotations aux dépréciations exceptionnelles"
"pcg_691","Employee profit share","691000","expense","","False","Participation des salariés aux résultats"
"pcg_6951","Income tax due in France","695100","expense","","False","Impôts sur les bénéfices dus en France"
"pcg_6952","Additional contribution to income tax","695200","expense","","False","Contribution additionnelle à l'impôt sur les bénéfices"
"pcg_6954","Income tax due outside France","695400","expense","","False","Impôts sur les bénéfices dus à l'étranger"
"pcg_696","Supplementary company tax related to profit distributions","696000","expense","","False","Supplément d'impôt sur les sociétés lié aux distributions"
"pcg_6981","Group tax - Charges","698100","expense","","False","Intégration fiscale - Charges"
"pcg_6989","Group tax - Income","698900","expense","","False","Intégration fiscale - Produits"
"pcg_699","Income - Carry-back of losses","699000","expense","","False","Produits, Reports en arrière des déficits"
"pcg_701_account","Sales of finished products","701000","income","account.account_tag_operating","False","Ventes de produits finis"
"pcg_702","Sales of semi-finished products","702000","income","account.account_tag_operating","False","Rabais, remises et ristournes sur ventes de produits intermédiaires"
"pcg_703","Sales of residual products","703000","income","account.account_tag_operating","False","Ventes de produits résiduels"
"pcg_704_account","Works","704000","income","account.account_tag_operating","False","Travaux"
"pcg_705","Project studies","705000","income","account.account_tag_operating","False","Ventes d'études"
"pcg_706","Services supplied","706000","income","account.account_tag_operating","False","Ventes de prestations de services"
"pcg_707_account","Sales of goods","707000","income","account.account_tag_operating","False","Ventes de marchandises"
"pcg_7081","Income from services operated in the interest of personne","708100","income","account.account_tag_operating","False","Produits des services exploités dans l'intérêt du personnel"
"pcg_7082","Commission and brokerage","708200","income","account.account_tag_operating","False","Commissions et courtages"
"pcg_7083","Sundry rentals","708300","income","account.account_tag_operating","False","Locations diverses"
"pcg_7084","Personnel charged out","708400","income","account.account_tag_operating","False","Mise à disposition de personnel facturée"
"pcg_7085","Carriage and ancillary costs invoiced","708500","income","account.account_tag_operating","False","Ports et frais accessoires facturés"
"pcg_7086","Surplus on recovery of returnable packaging","708600","income","account.account_tag_operating","False","Bonis sur reprises d'emballages consignés"
"pcg_7087","Bonuses obtained from customers and sales premiums","708700","income","account.account_tag_operating","False","Bonifications obtenues des clients et primes sur ventes"
"pcg_7088","Other income from ancillary activities (eg. disposal of consumables)","708800","income","account.account_tag_operating","False","Autres produits d'activités annexes (cessions d'approvisionnements...)"
"pcg_7091","Sales of finished products","709100","income","account.account_tag_operating","False","Ventes de produits finis"
"pcg_7092","Sales of semi-finished products","709200","income","account.account_tag_operating","False","Rabais, remises et ristournes sur ventes de produits intermédiaires"
"pcg_7094","Sales rebates, discounts, allowances granted by the entity on work","709400","income","account.account_tag_operating","False","Rabais, remises et ristournes sur travaux"
"pcg_7095","Sales rebates, discounts, allowances granted by the entity on Project studies","709500","income","account.account_tag_operating","False","Rabais, remises et ristournes sur études"
"pcg_7096","Sales rebates, discounts, allowances granted by the entity on Services supplied","709600","income","account.account_tag_operating","False","Rabais, remises et ristournes sur prestations de services"
"pcg_7097","Sales rebates, discounts, allowances granted by the entity on Sales of goods for resale","709700","income","account.account_tag_operating","False","Rabais, remises et ristournes sur ventes de marchandises"
"pcg_7098","Sales rebates, discounts, allowances granted by the entity on Income from ancillary activities","709800","income","","False","Rabais, remises et ristournes sur produits des activités annexes"
"pcg_71331","Change in work in progress (goods) - Products in progress","713310","income","account.account_tag_operating","False","Variation des en-cours de production de biens - Produits en cours"
"pcg_71335","Change in work in progress (goods) - Works in progress","713350","income","account.account_tag_operating","False","Variation des en-cours de production de biens - Travaux en cours"
"pcg_71341","Change in work in progress (services) - Project studies in progress","713410","income","account.account_tag_operating","False","Variation des en-cours de production de services - Études en cours"
"pcg_71345","Change in work in progress (services) - Supply of services in progress","713450","income","account.account_tag_operating","False","Variation des en-cours de production de services - Prestations de services en cours"
"pcg_71351","Change in product stocks Semi-finished products","713510","income","account.account_tag_operating","False","Variation des stocks de produits intermédiaires"
"pcg_71355","Change in product stocks Finished products","713550","income","account.account_tag_operating","False","Variation des stocks de produits finis"
"pcg_71358","Change in product stocks Residual products","713580","income","account.account_tag_operating","False","Variation des stocks de produits résiduels"
"pcg_721","Own work capitalised - Intangible fixed assets","721000","income","account.account_tag_operating","False","Production immobilisée - Immobilisations incorporelles"
"pcg_722","Own work capitalised - Tangible fixed assets","722000","income","account.account_tag_operating","False","Production immobilisée - Immobilisations corporelles"
"pcg_741","Operating subsidies","741000","income","account.account_tag_operating","False","Subventions d’exploitation"
"pcg_742","Balancing subsidies","742000","income","account.account_tag_investing","False","Subventions d’équilibre"
"pcg_747","Share of investment grants transferred to profit or loss for the year","747000","income","account.account_tag_investing","False","Quote-part des subventions d’investissement virée au résultat de l’exercice"
"pcg_7511","Royalties and licence fees for concessions, patents, licences, trade marks, processes, software","751100","income","account.account_tag_operating","False","Redevances pour concessions, brevets, licences, marques, procédés, logiciels"
"pcg_7516","Author and reproduction royalties","751600","income","account.account_tag_operating","False","Droits d'auteur et de reproduction"
"pcg_7518","Other royalties and similar assets","751800","income","account.account_tag_operating","False","Redevances pour autres droits et valeurs similaires"
"pcg_752","Revenues from buildings not allocated to professional activities","752000","income","account.account_tag_operating","False","Revenus des immeubles non affectés aux activités professionnelles"
"pcg_753","Remuneration of directors and executive managers","753000","income","account.account_tag_operating","False","Rémunérations de l’activité des administrateurs et des gérants"
"pcg_754","Rebates from cooperatives (resulting from surpluses)","754000","income","account.account_tag_operating","False","Ristournes perçues des coopératives (provenant des excédents)"
"pcg_7551","Share of loss transferred (accounts of the managing entity)","755100","income","","False","Quote-part de perte transférée (comptabilité du gérant)"
"pcg_7555","Share of profit (accounts of non-managing partners/associates)","755500","income","","False","Quote-part de bénéfice attribuée (comptabilité des associés non-gérants)"
"pcg_757","Proceeds from disposals of property, plant and equipment and intangible assets","757000","income","account.account_tag_investing","False","Produits des cessions d’immobilisations incorporelles et corporelles"
"pcg_7581","Deductions and penalties on purchases and sales","758100","income","account.account_tag_operating","False","Dédits et pénalités perçus sur achats et ventes"
"pcg_7582","Donations received","758200","income","account.account_tag_operating","False","Libéralités reçues"
"pcg_7583","Receipts on amortised receivables","758300","income","account.account_tag_operating","False",""
"pcg_7584","Tax relief other than income tax","758400","income","account.account_tag_operating","False","Dégrèvements d’impôts autres qu’impôts sur les bénéfices"
"pcg_7585","Bonuses from indexation clauses","758500","income","account.account_tag_operating","False","Bonis provenant de clauses d’indexation"
"pcg_7586","Lots","758600","income","account.account_tag_operating","False","Lots"
"pcg_7587","Insurance indemnities (if relating to insurance indemnities)","758700","income","account.account_tag_operating","False","Indemnités d’assurance (si relatif à des indemnités d'assurance)"
"pcg_7588","Setting up or liquidation of trusts","758800","income","","False","Opérations de constitution ou liquidation des fiducies"
"pcg_7611","Income from long-term equity interests","761100","income","account.account_tag_financing","False","Revenus des titres de participation"
"pcg_7612","Trust income, result for the period","761200","income","","False","Produits de la fiducie, résultat de la période"
"pcg_7616","Income from other forms of participating interests","761600","income","account.account_tag_financing","False","Revenus sur autres formes de participation"
"pcg_7617","Income from debts receivable related to participating interests","761700","income","account.account_tag_financing","False","Revenus des créances rattachées à des participations"
"pcg_7621","Income from long-term investment securities","762100","income","account.account_tag_financing","False","Revenus des titres immobilisés"
"pcg_7626","Income from loans","762600","income","account.account_tag_financing","False","Revenus des prêts"
"pcg_7627","Income from capitalised debts receivable","762700","income","account.account_tag_financing","False","Revenus des créances immobilisées"
"pcg_7631","Income from commercial debts receivable","763100","income","account.account_tag_financing","False","Revenus des créances commerciales"
"pcg_7638","Income from sundry debts receivable","763800","income","account.account_tag_financing","False","Revenus des créances diverses"
"pcg_764","Income from short-term investment securities","764000","income","account.account_tag_financing","False","Revenus des valeurs mobilières de placement"
"pcg_765","Discounts obtained","765000","income","account.account_tag_financing","False","Escomptes obtenus"
"pcg_766","Exchange gains","766000","income","account.account_tag_financing","False","Gains de change"
"pcg_7671","Income from disposals of financial assets","767100","income","account.account_tag_investing","False","Produits des cessions d’immobilisations financières"
"pcg_7672","Net income from disposals of portfolio securities","767200","income","account.account_tag_investing","False","Produits nets sur cessions de titres immobilisés de l’activité de portefeuille"
"pcg_7673","Net proceeds from disposals of marketable securities","767300","income","account.account_tag_financing","False","Produits nets sur cessions de valeurs mobilières de placement"
"pcg_7674","Net income on disposal of tokens","767400","income","account.account_tag_financing","False","Produits nets sur cessions de jetons"
"pcg_7683","Bonuses arising from the repurchase by the company of shares and bonds issued by itself","768300","income","account.account_tag_investing","False","Bonis provenant du rachat par l’entreprise d’actions et d’obligations émises par elle-même"
"pcg_772","Extraordinary income on previous years (during the year only)","772000","income","","False","Produits exceptionnels sur exercices antérieurs (en cours d'exercice seulement)"
"pcg_778_account","Other extraordinary income","778000","income","account.account_tag_investing","False","Autres produits exceptionnels"
"pcg_7811","Reversals of amortisation of intangible assets and property, plant and equipment","781100","income","","False","Reprises sur amortissements des immobilisations incorporelles et corporelles"
"pcg_78111","Reversal of amortisation of intangible assets","781110","income","account.account_tag_operating","False","Reprises sur amortissements des immobilisations incorporelles"
"pcg_78112","Reversals of depreciation on tangible fixed assets","781120","income","account.account_tag_operating","False","Reprises sur amortissements des immobilisations corporelles"
"pcg_7815","Provisions for operating liabilities and charges written back","781500","income","account.account_tag_operating","False","Reprises sur provisions d'exploitation"
"pcg_7816","Provisions for diminution in value of intangible and tangible fixed assets written back","781600","income","","False","Reprises sur dépréciations des immobilisations incorporelles et corporelles"
"pcg_78161","Reversals of impairment of intangible assets","781610","income","account.account_tag_operating","False","Reprises sur dépréciations des immobilisations incorporelles"
"pcg_78162","Reversals of impairment of property, plant and equipment","781620","income","account.account_tag_operating","False","Reprises sur dépréciations des immobilisations corporelles"
"pcg_7817","Reversals of impairment of current assets","781700","income","","False","Reprises sur dépréciations des actifs circulants"
"pcg_78173","Reversals of impairment of current assets - Inventories and work in progress","781730","income","account.account_tag_operating","False","Reprises sur dépréciations des actifs circulants - Stocks et en-cours"
"pcg_78174","Reversals of impairment of current assets - Receivables","781740","income","account.account_tag_operating","False","Reprises sur dépréciations des actifs circulants - Créances"
"pcg_7865","Reversals of financial provisions","786500","income","account.account_tag_financing","False","Reprises sur provisions financières"
"pcg_78662","Reversals of impairment of financial assets","786620","income","account.account_tag_financing","False","Reprises sur dépréciations des immobilisations financières"
"pcg_78665","Reversals of impairment of marketable securities","786650","income","account.account_tag_financing","False","Reprises sur dépréciations des valeurs mobilières de placement"
"pcg_78725","Reversals of regulated provisions (fixed assets) - Accelerated depreciation","787250","income","account.account_tag_investing","False","Reprises sur provisions réglementées (immobilisations) - Amortissements dérogatoires"
"pcg_7873","Reversals of regulated provisions (stocks)","787300","income","account.account_tag_investing","False","Reprises sur provisions réglementées (stocks)"
"pcg_7874","Reversals of other regulated provisions","787400","income","account.account_tag_investing","False","Reprises sur autres provisions réglementées"
"pcg_7875","Reversals of exceptional provisions","787500","income","account.account_tag_investing","False","Reprises sur provisions exceptionnelles"
"pcg_7876","Reversals of exceptional depreciation","787600","income","account.account_tag_investing","False","Reprises sur dépréciations exceptionnelles"

```

## File: data\template\account.fiscal.position-fr.csv

```csv
"id","sequence","name","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@fr"
"fiscal_position_template_domestic","1","Domestic - France","1","1","","l10n_fr.fr_and_mc","","","Domestique - France"
"fiscal_position_template_intraeub2c","2","EU private","1","","","base.europe","","","EU privé"
"fiscal_position_template_intraeub2b","3","Intra-EU B2B","1","1","","base.europe","tva_normale","tva_sale_good_intra_0","Intra-EU B2B"
"","","","","","","","tva_normale_ttc","tva_sale_good_intra_0",""
"","","","","","","","tva_intermediaire_encaissement","tva_sale_service_intra_0",""
"","","","","","","","tva_normale_encaissement","tva_sale_service_intra_0",""
"","","","","","","","tva_normale_encaissement_ttc","tva_sale_service_intra_0",""
"","","","","","","","tva_intermediaire_encaissement_ttc","tva_sale_service_intra_0",""
"","","","","","","","tva_specifique","tva_sale_good_intra_0",""
"","","","","","","","tva_specifique_ttc","tva_sale_good_intra_0",""
"","","","","","","","tva_intermediaire","tva_sale_good_intra_0",""
"","","","","","","","tva_intermediaire_ttc","tva_sale_good_intra_0",""
"","","","","","","","tva_reduite","tva_sale_good_intra_0",""
"","","","","","","","tva_reduite_ttc","tva_sale_good_intra_0",""
"","","","","","","","tva_reduite_encaissement_ttc","tva_sale_service_intra_0",""
"","","","","","","","tva_reduite_encaissement","tva_sale_service_intra_0",""
"","","","","","","","tva_super_reduite","tva_sale_good_intra_0",""
"","","","","","","","tva_super_reduite_ttc","tva_sale_good_intra_0",""
"","","","","","","","tva_super_reduite_encaissement_ttc","tva_sale_service_intra_0",""
"","","","","","","","tva_super_reduite_encaissement","tva_sale_service_intra_0",""
"","","","","","","","tva_acq_normale","tva_intra_normale_biens",""
"","","","","","","","tva_acq_specifique","tva_intra_specifique_biens",""
"","","","","","","","tva_acq_encaissement","tva_intra_normale_services",""
"","","","","","","","tva_acq_intermediaire_encaissement","tva_intra_intermediaire_services",""
"","","","","","","","tva_acq_intermediaire","tva_intra_intermediaire_biens",""
"","","","","","","","tva_acq_reduite","tva_intra_reduite_biens",""
"","","","","","","","tva_acq_encaissement_reduite","tva_intra_reduite_services",""
"","","","","","","","tva_acq_super_reduite","tva_intra_super_reduite_biens",""
"","","","","","","","tva_acq_encaissement_super_reduite","tva_intra_super_reduite_services",""
"fiscal_position_template_import_export","50","Import/Export Outside Europe + DOM-TOM","1","","","","tva_normale","tva_sale_good_export_0","Import/Export Hors Europe + DOM-TOM"
"","","","","","","","tva_normale_ttc","tva_sale_good_export_0",""
"","","","","","","","tva_normale_encaissement","tva_sale_service_export_0",""
"","","","","","","","tva_intermediaire_encaissement","tva_sale_service_export_0",""
"","","","","","","","tva_normale_encaissement_ttc","tva_sale_service_export_0",""
"","","","","","","","tva_intermediaire_encaissement_ttc","tva_sale_service_export_0",""
"","","","","","","","tva_specifique","tva_sale_good_export_0",""
"","","","","","","","tva_specifique_ttc","tva_sale_good_export_0",""
"","","","","","","","tva_intermediaire","tva_sale_good_export_0",""
"","","","","","","","tva_intermediaire_ttc","tva_sale_good_export_0",""
"","","","","","","","tva_reduite","tva_sale_good_export_0",""
"","","","","","","","tva_reduite_ttc","tva_sale_good_export_0",""
"","","","","","","","tva_reduite_encaissement_ttc","tva_sale_service_export_0",""
"","","","","","","","tva_reduite_encaissement","tva_sale_service_export_0",""
"","","","","","","","tva_super_reduite","tva_sale_good_export_0",""
"","","","","","","","tva_super_reduite_ttc","tva_sale_good_export_0",""
"","","","","","","","tva_super_reduite_encaissement_ttc","tva_sale_service_export_0",""
"","","","","","","","tva_super_reduite_encaissement","tva_sale_service_export_0",""
"","","","","","","","tva_acq_normale","tva_import_outside_eu_20",""
"","","","","","","","tva_acq_specifique","tva_import_outside_eu_8_5",""
"","","","","","","","tva_acq_intermediaire","tva_import_outside_eu_10",""
"","","","","","","","tva_acq_reduite","tva_import_outside_eu_5_5",""
"","","","","","","","tva_acq_super_reduite","tva_import_outside_eu_2_1",""

```

## File: data\template\account.group-fr.csv

```csv
"id","code_prefix_start","name","name@fr"
"pcg_10","10","Capital and reserves","Capital et réserves"
"pcg_101","101","Capital","Capital"
"pcg_104","104","Premiums on share capital","Primes liées au capital social"
"pcg_106","106","Reserves","Réserves"
"pcg_11","11","Retained earnings","Report à nouveau"
"pcg_12","12","Result for the year","Résultat de l'exercice"
"pcg_13","13","Investment grants","Subventions d'investissement"
"pcg_15","15","Provisions","Provisions"
"pcg_151","151","Provisions for risks","Provisions pour risques"
"pcg_158","158","Other provisions for charges","Autres provisions pour charges"
"pcg_16","16","Loans and similar debts payable","Emprunts et dettes assimilées"
"pcg_165","165","Deposits and sureties received","Dépôts et cautionnements reçus"
"pcg_166","166","Employee profit share","Participation des salariés aux résultats"
"pcg_167","167","Loans and debts payable subject to particular conditions","Emprunts et dettes assortis de conditions particulières"
"pcg_168","168","Other loans and similar debts payable","Autres emprunts et dettes assimilées"
"pcg_17","17","Debts payable related to participating interests","Dettes rattachées à des participations"
"pcg_18","18","Reciprocal branch and joint venture accounts","Comptes de liaison des établissements et sociétés en participation"
"pcg_20","20","Intangible fixed assets","Frais d'établissement"
"pcg_201","201","Intangible fixed assets","Frais d'établissement"
"pcg_21","21","Tangible fixed assets","Immobilisations corporelles"
"pcg_211","211","Land","Terrains"
"pcg_213","213","Constructions","Constructions"
"pcg_215","215","Technical installations, plant and machinery, equipment and fixtures","Installations techniques, matériels et outillage industriels"
"pcg_2151","2151","Specialised complex installations","Installations complexes spécialisées"
"pcg_2153","2153","Installations of specific nature","Installations à caractère spécifique"
"pcg_218","218","Other tangible fixed assets","Autres immobilisations corporelles"
"pcg_23","23","Fixed assets in progress","Immobilisations en cours"
"pcg_26","26","Participating interests and related debts receivable","Participations et créances rattachées à des participations"
"pcg_261","261","Long-term equity interests","Titres de participation"
"pcg_267","267","Debts receivable related to participating interests","Créances rattachées à des participations"
"pcg_268","268","Debts receivable related to joint ventures","Créances rattachées à des sociétés en participation"
"pcg_27","27","Other financial fixed asset","Autres immobilisations financières"
"pcg_271","271","Long-term investment equity securities other than portfolio long-term investment equity securities","Titres de participation d'investissement à long terme autres que les titres de participation d'investissement à long terme du portefeuille"
"pcg_272","272","Long-term investment debt securities","Titres immobilisés (droit de créance)"
"pcg_274","274","Loans","Prêts"
"pcg_275","275","Deposits and sureties advanced","Dépôts et cautionnements versés"
"pcg_276","276","Other capitalised debts receivable","Autres créances immobilisées"
"pcg_2768","2768","Accrued interest","Intérêts courus"
"pcg_277","277","(Own shares)","(Actions propres ou parts propres)"
"pcg_28","28","Cumulative depreciation on fixed assets","Amortissements des immobilisations incorporelles"
"pcg_281","281","Depreciation on tangible fixed assets","Amortissements des immobilisations corporelles"
"pcg_29","29","Provisions for diminution in value of fixed assets","Dépréciations des immobilisations incorporelles"
"pcg_291","291","Provisions for diminution in value of tangible fixed assets (same allocation as for Account 21)","Provisions pour dépréciation des immobilisations corporelles (même affectation que pour le compte 21)"
"pcg_293","293","Provisions for diminution in value of fixed assets in progress","Dépréciations des immobilisations en cours"
"pcg_296","296","Impairment of participating interests and receivables from participating interests","Dépréciations des participations et créances rattachées à des participations"
"pcg_297","297","Provisions for diminution in value of other financial fixed assets","Dépréciations des autres immobilisations financières"
"pcg_32","32","Other consumables","Autres approvisionnements"
"pcg_322","322","Consumable supplies","Fournitures consommables"
"pcg_326","326","Packaging","Emballages"
"pcg_33","33","Work in progress (goods)","En cours de production de biens"
"pcg_34","34","Work in progress (services)","En cours de production de services"
"pcg_35","35","Product stocks","Stocks de produits"
"pcg_358","358","Residual products (or recoverable materials)","Produits résiduels (ou matières de récupération)"
"pcg_39","39","Provisions for diminution in value of stocks and work in progress","Dépréciations des stocks et en cours"
"pcg_40","40","Suppliers and related accounts","Fournisseurs et comptes rattachés"
"pcg_401","401","Suppliers","Fournisseurs"
"pcg_404","404","Fixed asset suppliers","Fournisseurs d'immobilisations"
"pcg_408","408","Suppliers - Invoices outstanding","Fournisseurs factures non parvenues"
"pcg_409","409","Suppliers in debit","Fournisseurs débiteurs"
"pcg_4097","4097","Suppliers - Other debits","Fournisseurs autres avoirs"
"pcg_411","411","Customers","Clients"
"pcg_416_group","416","Doubtful customers","Clients douteux"
"pcg_418","418","Customers - Charges not yet invoiced","Clients produits non encore facturés"
"pcg_419","419","Customers in credit","Clients créditeurs"
"pcg_42","42","Personnel and related accounts","Personnel et comptes rattachés"
"pcg_424","424","Employee profit share","Participation des salariés aux résultats"
"pcg_428","428","Personnel - Accrued charges payable and income receivable","Personnel charges à payer et produits à recevoir"
"pcg_43","43","Social security and other social agencies","Sécurité sociale et autres organismes sociaux"
"pcg_438","438","Social agencies - Accrued charges payable and income receivable","Organismes sociaux - charges à payer et produits à recevoir"
"pcg_44","44","State and other public authorities","État et autres collectivités publiques"
"pcg_442","442","State - Taxes and levies recoverable from third parties","Etat impots et taxes recouvrables sur des tiers"
"pcg_445","445","State - Turnover tax","Etat taxes sur le chiffres d'affaires"
"pcg_4455","4455","Turnover tax payable","Taxes sur le chiffre d'affaires à décaisser"
"pcg_4456","4456","Turnover tax deductible","Taxes sur le chiffre d'affaires déductibles"
"pcg_4457","4457","Turnover tax collected by the entity","Taxes sur le chiffre d'affaires collectées par l'entreprise"
"pcg_448","448","State - Accrued charges payable and income receivable","Etat charges à payer et produits à recevoir"
"pcg_45","45","Group and partners/associates","Groupe et associés"
"pcg_455","455","Partners/associates - Current accounts","Associés comptes courants"
"pcg_456","456","Partners/associates - Capital transactions","Associés opérations sur le capital"
"pcg_4561","4561","Partners/associates - Company contribution accounts","Associés comptes d'appport en société"
"pcg_4562","4562","Contributors - Capital called up, unpaid","Apporteurs capital appelé, non versé"
"pcg_458","458","Partners/associates - Joint and Economic Interest Group transactions","Associés opérations faites en commun et en GIE"
"pcg_46","46","Sundry debts receivable and payable","Débiteurs divers et créditeurs divers"
"pcg_47","47","Provisional or suspense accounts","Comptes transitoires ou d'attente"
"pcg_474_group","474","Valuation differences - Assets","Différences d’évaluation – Actif"
"pcg_475_group","475","Valuation differences - Liabilities","Différences d’évaluation – Passif"
"pcg_476","476","Realisable currency exchange losses","Différence de conversion actif"
"pcg_477","477","Realisable currency exchange gains","Différences de conversion passif"
"pcg_48","48","Accrual accounts","Comptes de régularisation"
"pcg_488","488","Periodic allocation of charges and income","Comptes de répartition périodique des charges et des produits"
"pcg_49","49","Provisions for doubtful debts","Dépréciations des comptes de tiers"
"pcg_495","495","Provisions for group and partners/associates doubtful debts","Dépréciations des comptes du groupe et des associés"
"pcg_496","496","Provisions for sundry doubtful debts","Dépréciations des comptes de débiteurs divers"
"pcg_50","50","Short-term investment securities","Valeurs mobilières de placement"
"pcg_503","503","Shares","Actions"
"pcg_506","506","Bonds","Obligations"
"pcg_508","508","Other short-term investment securities and similar debts receivable","Autres valeurs mobilières de placement et autres créances assimilées"
"pcg_51","51","Banks, financial and similar institutions","Banques, établissements financiers et assimilés"
"pcg_511","511","Financial instruments for collection","Valeurs à l'encaissement"
"pcg_512","512","Banks","Banques"
"pcg_518","518","Accrued interest","Intérêts courus"
"pcg_519","519","Current bank advances","Concours bancaires courants"
"pcg_59","59","Provisions for diminution in value of financial assets","Dépréciations des valeurs mobilières de placement"
"pcg_60","60","Purchases (except 603)","Achats (sauf 603)"
"pcg_602","602","Inventory item purchases - Other consumables","Achats stockés autres approvisionnements"
"pcg_6026","6026","Packaging","Emballages"
"pcg_603","603","Changes in inventories (supplies and goods)","Variations des stocks (approvisionnements et marchandises)"
"pcg_606","606","Non-inventory materials and supplies","Achats non stockés de matière et fournitures"
"pcg_609","609","Purchase rebates, discounts, allowances","Rabais, remises et ristournes obtenus sur achats"
"pcg_61","61","External services","Services extérieurs"
"pcg_612","612","Lease instalments","Redevances de crédit bail"
"pcg_615","615","Maintenance and repairs","Entretien et réparations"
"pcg_616","616","Insurance premiums","Primes d'assurances"
"pcg_6163","6163","Transport insurance","Assurance transport"
"pcg_618","618","Sundry","Divers"
"pcg_62","62","Other external services","Autres services extérieurs"
"pcg_621","621","Personnel external to the entity","Personnel extérieur à l'entreprise"
"pcg_622","622","Agents remuneration and fees","Rémunérations d'intermédiaires et honoraires"
"pcg_623","623","Advertising, publications, public relations","Publicité, publications, relations publiques"
"pcg_624","624","Transport of goods and collective personnel transport","Transports de biens et transports collectifs du personnel"
"pcg_625","625","Business travel, missions and receptions","Déplacements, missions et réceptions"
"pcg_627","627","Banking and similar services","Services bancaires et assimilés"
"pcg_628","628","Sundry","Divers"
"pcg_63","63","Taxes, levies and similar payments","Impôts, taxes et versements assimilés"
"pcg_631","631","Taxes, levies and similar payments on wages and salaries (to the tax administration)","Impôts, taxes et versements assimilés sur rémunérations (administrations des impôts)"
"pcg_633","633","Taxes, levies and similar payments on wages and salaries (to otherbodies)","Impôts, taxes et versements assimilés sur rémunérations (autres organismes)"
"pcg_635","635","Other taxes, levies and similar payments (to the tax administration)","Autres impôts, taxes et versements assimilés (administrations des impôts)"
"pcg_6351","6351","Direct taxes (except income tax)","Impôts directs (sauf impôts sur les bénéfices)"
"pcg_6354","6354","Registration and stamp duties","Droits d'enregistrement et de timbre"
"pcg_637","637","Other taxes, levies and similar payments (to other bodies)","Autres impôts, taxes et versements assimilés (autres organismes)"
"pcg_64","64","Personnel costs","Charges de personnel"
"pcg_641","641","Personnel wages and salaries","Rémunérations du personnel"
"pcg_645","645","Social security and provident fund contributions","Charges de sécurité sociale et de prévoyance"
"pcg_647","647","Other welfare costs","Autres charges sociales"
"pcg_65","65","Other current operating charges","Autres charges de gestion courante"
"pcg_651","651","Royalties and licence fees for concessions, patents,licences, trade marks, processes, software, rights and similar assets","Redevances pour concessions, brevets, licences, marques, procédés, logiciels, droits et valeurs simulaires"
"pcg_654","654","Bad debts written off","Pertes sur créances irrécouvrables"
"pcg_655","655","Share of joint venture profit or loss","Quote part de résultat sur opérations faites en commun"
"pcg_658_group","658","Penalties and other expenses","Pénalités et autres charges"
"pcg_66","66","Financial charges","Charges financières"
"pcg_661","661","Interest charges","Charges d'intérêts"
"pcg_6611","6611","Loan and debt interest","Intérêts des emprunts et dettes"
"pcg_6618","6618","Interest on other debts payable","Intérêts des autres dettes"
"pcg_67","67","Extraordinary charges","Charges exceptionnelles"
"pcg_68","68","Appropriations to depreciation and provisions","Dotations aux amortissements, aux dépréciations et aux provisions"
"pcg_681","681","Appropriations to depreciation and provisions - Operating charges","Dotations aux amortissements, aux dépréciations et aux provisions"
"pcg_6816","6816","Impairment of intangible and tangible assets","Dotations pour dépréciations des immobilisations incorporelles et corporelles"
"pcg_6817","6817","Appropriations to provisions for diminution in value of current assets","Dotations pour dépréciations des actifs circulants"
"pcg_686","686","Appropriations to depreciation and provisions - Financial charges","Dotations aux amortissements, aux dépréciations et aux provisions"
"pcg_6866","6866","Appropriations to provisions for diminution in value of financial components","Dotations pour dépréciations des éléments financiers"
"pcg_687","687","Appropriations to depreciation and provisions - Extraordinarcharges","Dotations aux amortissements, aux dépréciations et aux provisions"
"pcg_6872","6872","Appropriations to tax-regulated provisions (fixed assets)","Dotations aux provisions réglementées (immobilisations)"
"pcg_69","69","Employee profit share - Income and similar taxes","Participation des salariés impots sur les bénéfices et assimilés"
"pcg_695","695","Income tax","Impôts sur les bénéfices"
"pcg_698","698","Group tax","Intégration fiscale"
"pcg_70","70","Sales of manufactured products, services, goods for resale","Ventes de produits fabriqués, prestations de services, marchandises"
"pcg_708","708","Income from related activities","Produits des activités annexes"
"pcg_709","709","Sales rebates, discounts, allowances granted by the entity","Rabais, remises et ristournes accordés par l'entreprise"
"pcg_71","71","Change in stocks of finished products and work in progress","Production stockée (ou déstockage)"
"pcg_713","713","Change in stocks (work in progress, products)","Variation des stocks (en cours de production, produits)"
"pcg_7133","7133","Change in work in progress (goods)","Variation des en cours de production de biens"
"pcg_7134","7134","Change in work in progress (services)","Variation des en cours de production de services"
"pcg_7135","7135","Change in product stocks","Variation des stocks de produits"
"pcg_72","72","Own work capitalised","Production immobilisée"
"pcg_74_group","74","Operating grants","Subventions d'exploitation"
"pcg_75","75","Other current operating income","Autres produits de gestion courante"
"pcg_751","751","Royalties and licence fees for concessions, patents, licences, trade marks, processes, software, rights and similar assets","Redevances et droits de licence pour concessions, brevets, licences, marques, procédés, logiciels, droits et actifs similaires"
"pcg_755","755","Share of joint venture profit or loss","Quote part de résultat sur opérations faites en commun"
"pcg_758_group","758","Compensation and other income","Indemnités et autres produits"
"pcg_76","76","Financial income","Produits financiers"
"pcg_761","761","Income from participating interests","Produits de participations"
"pcg_762","762","Income from other financial fixed assets","Produits des autres immobilisations financières"
"pcg_763","763","Income from other debts receivable","Revenus des autres créances"
"pcg_767_group","767","Income from disposal of financial assets","Produits sur cession d’éléments financiers"
"pcg_768_group","768","Other financial income","Autres produits financiers"
"pcg_77","77","Extraordinary income","Produits exceptionnels"
"pcg_78","78","Depreciation and provisions written back","Reprises sur amortissements, dépréciations et provisions"
"pcg_781","781","Depreciation and provisions written back (to be enteredin operating income)","Reprises sur amortissements, dépréciations et provisions (à inscrire dans les produits d'exploitation)"
"pcg_786","786","Provisions for liabilities written back (to be entered in financiaincome)","Reprises sur provisions pour risques et dépréciations (à inscrire dans les produits financiers)"
"pcg_7866","7866","Provisions for diminution in value of financial components written back","Reprises sur dépréciations des éléments financiers"
"pcg_787","787","Provisions written back (to be entered in extraordinary income)","Reprises sur provisions et dépréciations (à inscrire dans les produits exceptionnels)"
"pcg_7872","7872","Reprises sur provisions réglementées (immobilisations)","Reprises sur provisions réglementées (immobilisations)"

```

## File: data\template\account.tax-fr.csv

```csv
"id","name","description","invoice_label","amount","amount_type","sequence","type_tax_use","tax_scope","include_base_amount","tax_group_id","active","tax_exigibility","cash_basis_transition_account_id","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","repartition_line_ids/tag_ids","description@fr","price_include_override"
"tva_acq_normale","20% G","20% Goods","TVA 20%","20.0","percent","9","purchase","consu","1","tax_group_tva_20","","","","100","base","invoice","","","20% M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_specifique","8.5% G","8.5% Goods","TVA 8.5%","8.5","percent","10","purchase","consu","1","tax_group_tva_85","","","","100","base","invoice","","","8,5% M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_intermediaire","10% G","10% Goods","TVA 10%","10.0","percent","10","purchase","consu","1","tax_group_tva_10","","","","100","base","invoice","","","10% M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_reduite","5.5% G","5.5% Goods","TVA 5.5%","5.5","percent","10","purchase","consu","1","tax_group_tva_55","","","","100","base","invoice","","","5,5% M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_super_reduite","2.1% G","2.1% Goods","TVA 2.1%","2.1","percent","10","purchase","consu","1","tax_group_tva_21","","","","100","base","invoice","","","2,1% M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_purchase_good_fuel","20% F","20% Fuels","TVA 20%","20.0","percent","9","purchase","consu","1","tax_group_tva_20","","","","100","base","invoice","","","20% Carburant",""
"","","","","","","","","","","","","","","80","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","20","tax","invoice","","","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","80","tax","refund","pcg_44566","-20","",""
"","","","","","","","","","","","","","","20","tax","refund","","","",""
"tva_purchase_good_fuel_TTC","20% F INC","20% fuel tax incl.","TVA 20%","20.0","percent","9","purchase","consu","1","tax_group_tva_20","","","","100","base","invoice","","","20% Carburant TTC",""
"","","","","","","","","","","","","","","80","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","20","tax","invoice","","","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","80","tax","refund","pcg_44566","-20","",""
"","","","","","","","","","","","","","","20","tax","refund","","","",""
"tva_acq_normale_TTC","20% G INC","20% Goods tax incl.","TVA 20%","20.0","percent","10","purchase","consu","","tax_group_tva_20","","","","100","base","invoice","","","20% M TTC",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_specifique_TTC","8.5% G INC","8.5% Goods tax incl.","TVA 8.5%","8.5","percent","10","purchase","consu","","tax_group_tva_85","","","","100","base","invoice","","","8,5% M TTC",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_intermediaire_TTC","10% G INC","10% Goods tax incl.","TVA 10%","10.0","percent","10","purchase","consu","","tax_group_tva_10","","","","100","base","invoice","","","10% M TTC",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_reduite_TTC","5.5% G INC","5.5% Goods tax incl.","TVA 5.5%","5.5","percent","10","purchase","consu","","tax_group_tva_55","","","","100","base","invoice","","","5,5% M TTC",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_super_reduite_TTC","2.1% G INC","2.1% Goods tax incl.","TVA 2.1%","2.1","percent","10","purchase","consu","","tax_group_tva_21","","","","100","base","invoice","","","2,1% M TTC",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_imm_normale","20% R E ","20% real estate","TVA 20%","20.0","percent","10","purchase","consu","1","tax_group_tva_20","","","","100","base","invoice","","","20% Immo",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44562","+19","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44562","-19","",""
"tva_imm_specifique","8.5% R E","8.5% real estate","TVA 8.5%","8.5","percent","10","purchase","consu","1","tax_group_tva_85","0","","","100","base","invoice","","","8,5% Immo",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44562","+19","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44562","-19","",""
"tva_imm_intermediaire","10% R E","10% real estate","TVA 10%","10.0","percent","10","purchase","consu","1","tax_group_tva_10","0","","","100","base","invoice","","","10% immo",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44562","+19","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44562","-19","",""
"tva_imm_reduite","5.5% R E","5.5% real estate","TVA 5.5%","5.5","percent","10","purchase","consu","1","tax_group_tva_55","0","","","100","base","invoice","","","5,5% Immo",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44562","+19","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44562","-19","",""
"tva_imm_super_reduite","2.1% R E","2.1% real estate","TVA 2.1%","2.1","percent","10","purchase","consu","1","tax_group_tva_21","0","","","100","base","invoice","","","2,1% Immo",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44562","+19","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44562","-19","",""
"tva_import_outside_eu_20","20% EX O EU","20% import","TVA 20%","20.0","percent","11","purchase","consu","1","tax_group_tva_20","","","","100","base","invoice","","+A4||+I1_base","20% Import",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445663","+20||+24","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_4453","-I1_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A4||-I1_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445663","-20||-24","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_4453","+I1_taxe","",""
"tva_import_outside_eu_10","10% EX","10% import","TVA 10%","10.0","percent","11","purchase","consu","1","tax_group_tva_10","0","","","100","base","invoice","","+A4||+I2_base","10% Import",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445663","+20||+24","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_4453","-I2_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A4||-I2_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445663","-20||-24","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_4453","+I2_taxe","",""
"tva_import_outside_eu_8_5","8.5% EX","8.5% import","TVA 8.5%","8.5","percent","11","purchase","consu","1","tax_group_tva_85","0","","","100","base","invoice","","+A4||+I3_base","8,5% Import",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445663","+20||+24","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_4453","-I3_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A4||-I3_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445663","-20||-24","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_4453","+I3_taxe","",""
"tva_import_outside_eu_5_5","5.5% EX","5.5% import","TVA 5.5%","5.5","percent","11","purchase","consu","1","tax_group_tva_55","0","","","100","base","invoice","","+A4||+I4_base","5,5% Import",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445663","+20||+24","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_4453","-I4_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A4||-I4_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445663","-20||-24","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_4453","+I4_taxe","",""
"tva_import_outside_eu_2_1","2.1% EX","2.1% import","TVA 2.1%","2.1","percent","11","purchase","consu","1","tax_group_tva_21","0","","","100","base","invoice","","+A4||+I5_base","2,1% Import",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445663","+20||+24","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_4453","-I5_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A4||-I5_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445663","-20||-24","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_4453","+I5_taxe","",""
"tva_intra_normale_biens","20% EU G","20% EU G","TVA 20%","20.0","percent","10","purchase","consu","1","tax_group_tva_20","","","","100","base","invoice","","+B2||+08_base","20% EU M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_4452","-08_taxe||-17","",""
"","","","","","","","","","","","","","","100","base","refund","","-B2||-08_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_4452","+08_taxe||+17","",""
"tva_intra_normale_services","20% EU S","20% EU S","TVA 20%","20.0","percent","10","purchase","service","1","tax_group_tva_20","","","","100","base","invoice","","+A3||+08_base","20% EU S",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_44521","-08_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A3||-08_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_44521","+08_taxe","",""
"tva_purchase_service_20_import","20% EX","20% IMPORT","TVA 20%","20.0","percent","10","purchase","service","1","tax_group_tva_20","","","","100","base","invoice","","+B4||+08_base","20% Import",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445663","+20","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_44531","-08_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-B4||-08_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445663","-20","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_44531","+08_taxe","",""
"tva_purchase_service_0","0% EX","0% EXO","TVA 0%","0.0","percent","10","purchase","service","1","tax_group_tva_0","","on_invoice","pcg_44574","100","base","invoice","","","0% EXO",""
"","","","","","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","","","",""
"tva_acq_encaissement","20% S","20% Service","TVA 20%","20.0","percent","10","purchase","service","1","tax_group_tva_20","","on_payment","pcg_44564","100","base","invoice","","","20% Service",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_intermediaire_encaissement","10% S","10% Service","TVA 10%","10.0","percent","10","purchase","service","1","tax_group_tva_10","0","on_payment","pcg_44564","100","base","invoice","","","10% Service",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_encaissement_reduite","5.5% S","5.5% Service","TVA 5.5%","5.5","percent","10","purchase","service","1","tax_group_tva_55","0","on_payment","pcg_44564","100","base","invoice","","","5,5% Service",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_encaissement_super_reduite","2.1% S","2.1% Service","TVA 2.1%","2.1","percent","10","purchase","service","1","tax_group_tva_21","0","on_payment","pcg_44564","100","base","invoice","","","2,1% Service",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_encaissement_TTC","20% S INC","20% Service tax incl.","VAT 20%","20.0","percent","10","purchase","service","","tax_group_tva_20","","on_payment","pcg_44564","100","base","invoice","","","20% Service TTC",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_intermediaire_encaissement_TTC","10% S INC","10% Service tax incl.","TVA 10%","10.0","percent","10","purchase","service","","tax_group_tva_10","0","on_payment","pcg_44564","100","base","invoice","","","10% Service TTC",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_encaissement_reduite_TTC","5.5% S INC","5.5% Service tax incl.","TVA 5.5%","5.5","percent","10","purchase","service","","tax_group_tva_55","0","on_payment","pcg_44564","100","base","invoice","","","5,5 Service TTC",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_acq_encaissement_super_reduite_TTC","2.1% S INC","2.1% Service tax incl.","TVA 2.1%","2.1","percent","10","purchase","service","","tax_group_tva_21","0","on_payment","pcg_44564","100","base","invoice","","","2,1% Service TTC",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44566","+20","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44566","-20","",""
"tva_purchase_imm_normale","20% R E","20% real estate","TVA 20%","20.0","percent","10","purchase","service","1","tax_group_tva_20","","on_payment","pcg_44564","100","base","invoice","","","20% Immo",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44562","+19","",""
"","","","","","","","","","","","","","","100","base","refund","","","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44562","-19","",""
"tva_normale","20% G","20% Goods","TVA 20%","20.0","percent","10","sale","consu","1","tax_group_tva_20","","","","100","base","invoice","","+A1||+08_base","20% M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+08_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-08_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-08_taxe","",""
"tva_intermediaire","10% G","10% Goods","TVA 10%","10.0","percent","10","sale","consu","1","tax_group_tva_10","0","","","100","base","invoice","","+A1||+9B_base","10% M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+9B_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-9B_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-9B_taxe","",""
"tva_reduite","5.5% G","5.5% Goods","TVA 5.5%","5.5","percent","10","sale","consu","1","tax_group_tva_55","0","","","100","base","invoice","","+A1||+09_base","5,5% M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+09_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-09_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-09_taxe","",""
"tva_specifique","8.5% G","8.5% Goods","TVA 8.5%","8.5","percent","10","sale","consu","1","tax_group_tva_85","0","","","100","base","invoice","","+A1||+10_base","8,5% M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+10_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-10_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-10_taxe","",""
"tva_super_reduite","2.1% G","2.1% Goods","TVA 2.1%","2.1","percent","10","sale","consu","1","tax_group_tva_21","0","","","100","base","invoice","","+A1||+11_base","2.1% M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+11_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-11_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-11_taxe","",""
"tva_normale_ttc","20% G INC","20% Goods tax incl.","TVA 20%","20.0","percent","10","sale","consu","","tax_group_tva_20","0","","","100","base","invoice","","+A1||+08_base","20% M TTC","tax_included"
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+08_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-08_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-08_taxe","",""
"tva_intermediaire_ttc","10% G INC","10% Goods tax incl.","TVA 10%","10.0","percent","10","sale","consu","","tax_group_tva_10","0","","","100","base","invoice","","+A1||+9B_base","10% M TTC","tax_included"
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+9B_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-9B_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-9B_taxe","",""
"tva_specifique_ttc","8.5% G INC","8.5% Goods tax incl.","TVA 8.5%","8.5","percent","10","sale","consu","","tax_group_tva_85","0","","","100","base","invoice","","+A1||+10_base","8,5% M TTC","tax_included"
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+10_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-10_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-10_taxe","",""
"tva_reduite_ttc","5.5% G INC","5.5% Goods tax incl.","TVA 5.5%","5.5","percent","10","sale","consu","","tax_group_tva_55","0","","","100","base","invoice","","+A1||+09_base","5,5% M TTC","tax_included"
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+09_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-09_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-09_taxe","",""
"tva_super_reduite_ttc","2.1% G INC","2.1% Goods tax incl.","TVA 2.1%","2.1","percent","10","sale","consu","","tax_group_tva_21","0","","","100","base","invoice","","+A1||+11_base","2,1% M TTC","tax_included"
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+11_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-11_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-11_taxe","",""
"tva_sale_good_0","0% EXEMPT G","0% EXO","TVA 0%","0.0","percent","10","sale","consu","1","tax_group_tva_0","","","","100","base","invoice","","+E2","0% EXO",""
"","","","","","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","","","","","100","base","refund","","-E2","",""
"","","","","","","","","","","","","","","100","tax","refund","","","",""
"tva_sale_good_export_0","0% EX G","0% EXPORT","TVA 0%","0.0","percent","10","sale","consu","1","tax_group_tva_0","","","","100","base","invoice","","+E1","0% EXPORT",""
"","","","","","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","","","","","100","base","refund","","-E1","",""
"","","","","","","","","","","","","","","100","tax","refund","","","",""
"tva_sale_good_intra_0","0% EU G","0% EU G","TVA 0%","0.0","percent","10","sale","consu","1","tax_group_tva_0","","","","100","base","invoice","","+F2","0% EU M",""
"","","","","","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","","","","","100","base","refund","","-F2","",""
"","","","","","","","","","","","","","","100","tax","refund","","","",""
"tva_normale_encaissement","20% S","20% Service","TVA 20%","20.0","percent","10","sale","service","1","tax_group_tva_20","","on_payment","pcg_44574","100","base","invoice","","+A1||+08_base","20% Service",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+08_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-08_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-08_taxe","",""
"tva_intermediaire_encaissement","10% S","10% Service","TVA 10%","10.0","percent","10","sale","service","1","tax_group_tva_10","0","on_payment","pcg_44574","100","base","invoice","","+A1||+9B_base","10% Service",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+9B_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-9B_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-9B_taxe","",""
"tva_reduite_encaissement","5.5% S","5.5% Service","TVA 5.5%","5.5","percent","10","sale","service","1","tax_group_tva_55","0","on_payment","pcg_44574","100","base","invoice","","+A1||+09_base","5,5% Service",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+09_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-09_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-09_taxe","",""
"tva_super_reduite_encaissement","2.1% S","2.1% Service","TVA 2.1%","2.1","percent","10","sale","service","1","tax_group_tva_21","0","on_payment","pcg_44574","100","base","invoice","","+A1||+11_base","2,1% Service",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+11_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-11_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-11_taxe","",""
"tva_normale_encaissement_ttc","20% S INC","20% Service tax incl.","TVA 20%","20.0","percent","10","sale","service","","tax_group_tva_20","0","on_payment","pcg_445800","100","base","invoice","","+A1||+08_base","20% Service TTC","tax_included"
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+08_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-08_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-08_taxe","",""
"tva_intermediaire_encaissement_ttc","10% S INC","10% Service tax incl.","VAT 10%","10.0","percent","10","sale","service","","tax_group_tva_10","0","on_payment","pcg_445800","100","base","invoice","","+A1||+9B_base","10% Service TTC","tax_included"
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+9B_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-9B_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-9B_taxe","",""
"tva_reduite_encaissement_ttc","5.5% S INC","5.5% Service tax incl.","TVA 5.5%","5.5","percent","10","sale","service","","tax_group_tva_55","0","on_payment","pcg_445800","100","base","invoice","","+A1||+09_base","5,5% Service TTC","tax_included"
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+09_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-09_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-09_taxe","",""
"tva_super_reduite_encaissement_ttc","2.1% S INC","2.1% Service tax incl.","TVA 2.1%","2.1","percent","10","sale","service","","tax_group_tva_21","0","on_payment","pcg_445800","100","base","invoice","","+A1||+11_base","2,1% Service TTC","tax_included"
"","","","","","","","","","","","","","","100","tax","invoice","pcg_44571","+11_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A1||-11_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_44571","-11_taxe","",""
"tva_sale_service_0","0% EXEMPT S","0% EXEMPTS Service","TVA 0%","0.0","percent","10","sale","service","1","tax_group_tva_0","","on_payment","pcg_44574","100","base","invoice","","+E2","0% EXO",""
"","","","","","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","","","","","100","base","refund","","-E2","",""
"","","","","","","","","","","","","","","100","tax","refund","","","",""
"tva_sale_service_export_0","0% EX S","0% EXPORT Service","TVA 0%","0.0","percent","10","sale","service","1","tax_group_tva_0","","","pcg_44574","100","base","invoice","","+E2","0% EXPORT",""
"","","","","","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","","","","","100","base","refund","","-E2","",""
"","","","","","","","","","","","","","","100","tax","refund","","","",""
"tva_sale_service_intra_0","0% EU S","0% EU Service","TVA 0%","0.0","percent","10","sale","service","1","tax_group_tva_0","","on_payment","pcg_44574","100","base","invoice","","+E2","0% EU Service",""
"","","","","","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","","","","","100","base","refund","","-E2","",""
"","","","","","","","","","","","","","","100","tax","refund","","","",""
"tva_intra_specifique_biens","8.5% EU G","8.5% EU Goods","TVA 8.5%","8.5","percent","10","purchase","consu","1","tax_group_tva_85","0","","","100","base","invoice","","+B2||+10_base","TVA 8,5% EU M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_4452","-10_taxe||-17","",""
"","","","","","","","","","","","","","","100","base","refund","","-B2||-10_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_4452","+10_taxe||+17","",""
"tva_intra_specifique_services","8.5% EU S","8.5% EU Service","TVA 8.5%","8.5","percent","10","purchase","service","1","tax_group_tva_85","0","","","100","base","invoice","","+A3||+10_base","TVA 8,5 EU Service",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+17||+10_taxe","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_44521","-20","",""
"","","","","","","","","","","","","","","100","base","refund","","-A3||-10_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-17||-10_taxe","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_44521","+20","",""
"tva_intra_intermediaire_biens","10% EU G","10% EU Goods","TVA 10%","10.0","percent","10","purchase","consu","1","tax_group_tva_10","0","","","100","base","invoice","","+B2||+9B_base","10% EU M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_4452","-9B_taxe||-17","",""
"","","","","","","","","","","","","","","100","base","refund","","-B2||-9B_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_4452","+9B_taxe||+17","",""
"tva_intra_intermediaire_services","10% EU S","10% EU Service","TVA 10%","10.0","percent","10","purchase","service","1","tax_group_tva_10","0","","","100","base","invoice","","+A3||+9B_base","10% EU M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_44521","-9B_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A3||-9B_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_44521","+9B_taxe","",""
"tva_intra_reduite_biens","5.5% EU G","5.5% EU Goods","TVA 5.5%","5.5","percent","10","purchase","consu","1","tax_group_tva_55","0","","","100","base","invoice","","+B2||+09_base","5,5% EU M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_4452","-09_taxe||-17","",""
"","","","","","","","","","","","","","","100","base","refund","","-B2||-09_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_4452","+09_taxe||+17","",""
"tva_intra_reduite_services","5.5% EU S","5.5% EU Service","TVA 5.5%","5.5","percent","10","purchase","service","1","tax_group_tva_55","0","","","100","base","invoice","","+A3||+09_base","5,5% EU M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_44521","-09_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A3||-09_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_44521","+09_taxe","",""
"tva_intra_super_reduite_biens","2.1% EU G","2.1% EU Goods","TVA 2.1%","2.1","percent","10","purchase","consu","1","tax_group_tva_21","0","","","100","base","invoice","","+B2||+11_base","2,1% EU M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_4452","-11_taxe||-17","",""
"","","","","","","","","","","","","","","100","base","refund","","-B2||-11_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_4452","+11_taxe||+17","",""
"tva_intra_super_reduite_services","2.1% EU S","2.1% EU Service","TVA 2.1%","2.1","percent","10","purchase","service","1","tax_group_tva_21","0","","","100","base","invoice","","+A3||+11_base","2,1% EU M",""
"","","","","","","","","","","","","","","100","tax","invoice","pcg_445662","+20","",""
"","","","","","","","","","","","","","","-100","tax","invoice","pcg_44521","-11_taxe","",""
"","","","","","","","","","","","","","","100","base","refund","","-A3||-11_base","",""
"","","","","","","","","","","","","","","100","tax","refund","pcg_445662","-20","",""
"","","","","","","","","","","","","","","-100","tax","refund","pcg_44521","+11_taxe","",""

```

## File: data\template\account.tax.group-fr.csv

```csv
"id","name","country_id","tax_receivable_account_id","tax_payable_account_id","name@fr"
"tax_group_tva_0","VAT 0%","base.fr","pcg_44567","pcg_44551","TVA 0%"
"tax_group_tva_20","VAT 20%","base.fr","pcg_44567","pcg_44551","TVA 20%"
"tax_group_tva_85","VAT 8.5%","base.fr","pcg_44567","pcg_44551","TVA 8,5%"
"tax_group_tva_55","VAT 5.5%","base.fr","pcg_44567","pcg_44551","TVA 5,5%"
"tax_group_tva_10","VAT 10%","base.fr","pcg_44567","pcg_44551","TVA 10%"
"tax_group_tva_21","VAT 2.1%","base.fr","pcg_44567","pcg_44551","TVA 2,1%"

```

## File: migrations\2.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    for company in env['res.company'].search([('chart_template', '=', 'fr')], order="parent_path"):
        env['account.chart.template'].try_loading('fr', company)

```

## File: migrations\2.2\pre-migrate-add-bank-xmlid.py

```python
def migrate(cr, version):
    cr.execute(
        """
        WITH banks AS (
            SELECT bank.id,
                   LOWER(bank.bic) AS bic
              FROM res_bank bank
              JOIN ir_model_data d
                ON d.module = 'base'
               AND d.name = 'fr'
               AND d.res_id = bank.country
             WHERE bank.active
               AND bank.bic ~ '^[A-Z0-9]+$'
        )
        INSERT INTO ir_model_data(
                        model,
                        module,
                        name,
                        res_id,
                        noupdate
                    )
             SELECT 'res.bank',
                    'l10n_fr_account',
                    CONCAT('bank_fr_', banks.bic),
                    banks.id,
                    True
               FROM banks
        ON CONFLICT DO NOTHING
        """
    )

```

## File: models\account_move.py

```python
from odoo import fields, models, api


class AccountMove(models.Model):
    _inherit = "account.move"

    l10n_fr_is_company_french = fields.Boolean(compute='_compute_l10n_fr_is_company_french')

    @api.model
    def _get_view(self, view_id=None, view_type='form', **options):
        arch, view = super()._get_view(view_id, view_type, **options)
        company = self.env.company
        if view_type == 'form' and company.country_code in company._get_france_country_codes():
            shipping_field = arch.xpath("//field[@name='partner_shipping_id']")[0]
            shipping_field.attrib.pop("groups", None)
        return arch, view

    @api.depends('company_id.country_code')
    def _compute_l10n_fr_is_company_french(self):
        for record in self:
            record.l10n_fr_is_company_french = record.country_code in record.company_id._get_france_country_codes()

    @api.depends("country_code", "move_type")
    def _compute_show_delivery_date(self):
        # EXTEND 'account'
        super()._compute_show_delivery_date()
        for move in self.filtered(lambda m: m.country_code == 'FR'):
            move.show_delivery_date = move.is_sale_document()

    def _post(self, soft=True):
        # EXTEND 'account'
        res = super()._post(soft=soft)
        for move in self.filtered(lambda m: m.show_delivery_date and not m.delivery_date):
            move.delivery_date = move.invoice_date
        return res

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_fr_rounding_difference_loss_account_id = fields.Many2one('account.account', check_company=True)
    l10n_fr_rounding_difference_profit_account_id = fields.Many2one('account.account', check_company=True)

```

## File: models\template_fr.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models, Command
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('fr')
    def _get_fr_template_data(self):
        return {
            'code_digits': 6,
            'property_account_receivable_id': 'fr_pcg_recv',
            'property_account_payable_id': 'fr_pcg_pay',
            'property_account_expense_categ_id': 'pcg_607_account',
            'property_account_income_categ_id': 'pcg_707_account',
            'property_account_downpayment_categ_id': 'pcg_4191',
        }

    @template('fr', 'res.company')
    def _get_fr_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.fr',
                'bank_account_code_prefix': '512',
                'cash_account_code_prefix': '53',
                'transfer_account_code_prefix': '58',
                'account_default_pos_receivable_account_id': 'fr_pcg_recv_pos',
                'income_currency_exchange_account_id': 'pcg_766',
                'expense_currency_exchange_account_id': 'pcg_666',
                'account_journal_suspense_account_id': 'pcg_471',
                'account_journal_early_pay_discount_loss_account_id': 'pcg_665',
                'account_journal_early_pay_discount_gain_account_id': 'pcg_765',
                'deferred_expense_account_id': 'pcg_486',
                'deferred_revenue_account_id': 'pcg_487',
                'l10n_fr_rounding_difference_loss_account_id': 'pcg_4768',
                'l10n_fr_rounding_difference_profit_account_id': 'pcg_4778',
                'account_sale_tax_id': 'tva_normale',
                'account_purchase_tax_id': 'tva_acq_normale',
            },
        }

    @template('fr', 'account.journal')
    def _get_fr_account_journal(self):
        return {
            'sale': {'refund_sequence': True},
            'purchase': {'refund_sequence': True},
        }

    @template('fr', 'account.reconcile.model')
    def _get_fr_reconcile_model(self):
        return {
            'bank_charges_reconcile_model': {
                'name': 'Bank fees',
                'line_ids': [
                    Command.create({
                        'account_id': 'pcg_6278',
                        'amount_type': 'percentage',
                        'amount_string': '100',
                    }),
                ],
            },
        }

```

## File: models\template_mc.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('mc')
    def _get_mc_template_data(self):
        return {
            'code_digits': '6',
            'parent': 'fr',
        }

    def _deref_account_tags(self, template_code, tax_data):
        if template_code == 'mc':
            template_code = 'fr'
        return super()._deref_account_tags(template_code, tax_data)

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_move
from . import template_fr
from . import template_mc
from . import res_company

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_fr_fec_export_wizard,access_l10n_fr_fec_export_wizard,model_l10n_fr_fec_export_wizard,account.group_account_user,1,1,1,0

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document">
        <xpath expr="(//address)[1]" position="after">
            <div class="mb-0" t-if="o.l10n_fr_is_company_french and o.partner_id.commercial_partner_id.siret">
                SIRET: <t t-esc="o.partner_id.commercial_partner_id.siret"/>
            </div>
        </xpath>
        <xpath expr="(//address)[2]" position="after">
            <div class="mb-0" t-if="o.l10n_fr_is_company_french and o.partner_id.commercial_partner_id.siret">
                SIRET: <t t-esc="o.partner_id.commercial_partner_id.siret"/>
            </div>
        </xpath>
        <xpath expr="(//address)[3]" position="after">
            <div class="mb-0" t-if="o.l10n_fr_is_company_french and o.partner_id.commercial_partner_id.siret">
                SIRET: <t t-esc="o.partner_id.commercial_partner_id.siret"/>
            </div>
        </xpath>

        <xpath expr="//div[@id='informations']" position="inside">
            <t t-if="o.l10n_fr_is_company_french and o.partner_id.commercial_partner_id != o.partner_id and o.move_type.startswith('out_')">
                <t t-set="partner" t-value="o.partner_id.commercial_partner_id"/>
                <div t-attf-class="#{'col-auto col-3 mw-100' if report_type != 'html' else 'col'} mb-2" name="customer_address">
                    <strong>Customer Address:</strong>
                    <br/>
                    <address t-field="partner.self" class="m-0" t-options="{'widget': 'contact', 'fields': ['address'], 'no_marker': True}"/>
                </div>
            </t>
        </xpath>

        <xpath expr="//div[@id='informations']" position="inside">
            <t t-if="o.l10n_fr_is_company_french and o.move_type.startswith('out_')">
                <t t-set="tax_scopes" t-value="o.invoice_line_ids.mapped('tax_ids.tax_scope')"/>
                <t t-set="has_service" t-value="'service' in tax_scopes"/>
                <t t-set="has_consu" t-value="'consu' in tax_scopes"/>

                <div t-if="has_service or has_consu" t-attf-class="#{'col-auto col-3 mw-100' if report_type != 'html' else 'col'} mb-2" name="operation_type">
                    <strong>Operation Type:</strong>
                    <br/>
                    <span t-if="has_service and has_consu">
                        Mixed Operation
                    </span>
                    <span t-elif="has_service and not has_consu">
                        Service Delivery
                    </span>
                    <span t-else="">
                        Goods Delivery
                    </span>
                </div>
            </t>
        </xpath>

        <xpath expr="//div[@name='qr_code_placeholder']" position="before">
            <div class="mb-3">
                <p t-if="o.l10n_fr_is_company_french and o.move_type.startswith('out_') and 'on_invoice' in o.invoice_line_ids.mapped('tax_ids.tax_exigibility')">
                    Option to pay tax on debits
                </p>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_partner_form_l10n_fr" model="ir.ui.view">
        <field name="name">res.partner.form.l10n.fr</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="l10n_fr.res_partner_form_l10n_fr"/>
        <field name="arch" type="xml">
            <field name="siret" position="attributes">
                <attribute name="invisible">'FR' not in fiscal_country_codes or not is_company</attribute>
            </field>
        </field>
    </record>
</odoo>

```

## File: wizard\account_fr_fec_export_wizard.py

```python
# -*- coding: utf-8 -*-
# Copyright (C) 2013-2015 Akretion (http://www.akretion.com)
import csv
import io
from odoo.tools import float_is_zero, SQL
from odoo import fields, models, api
from odoo.tools.misc import get_lang
from stdnum.fr import siren


class FecExportWizard(models.TransientModel):
    _name = 'l10n_fr.fec.export.wizard'
    _description = 'Fichier Echange Informatise'

    date_from = fields.Date(string='Start Date', required=True, default=lambda self: self._context.get('report_dates', {}).get('date_from'))
    date_to = fields.Date(string='End Date', required=True, default=lambda self: self._context.get('report_dates', {}).get('date_to'))
    filename = fields.Char(string='Filename', size=256, readonly=True)
    test_file = fields.Boolean()
    exclude_zero = fields.Boolean(string="Exclude lines at 0")
    export_type = fields.Selection([
        ('official', 'Official FEC report (posted entries only)'),
        ('nonofficial', 'Non-official FEC report (posted and unposted entries)'),
    ], string='Export Type', required=True, default='official')
    excluded_journal_ids = fields.Many2many('account.journal', string="Excluded Journals",
                                            domain="[('company_id', 'parent_of', current_company_id)]")

    @api.onchange('test_file')
    def _onchange_export_file(self):
        if not self.test_file:
            self.export_type = 'official'

    def _get_base_domain(self):
        domain = [('company_id', 'in', tuple(self.env.company._accessible_branches().ids))]
        # For official report: only use posted entries
        if self.export_type == "official":
            domain.append(('parent_state', '=', 'posted'))
        if self.excluded_journal_ids:
            domain.append(('journal_id', 'not in', self.excluded_journal_ids.ids))
        if self.exclude_zero:
            domain.append(('balance', '!=', 0.0))
        return domain

    def _do_query_unaffected_earnings(self):
        """ Compute the sum of ending balances for all accounts that are of a type that does not bring forward the balance in new fiscal years.
            This is needed because we have to display only one line for the initial balance of all expense/revenue accounts in the FEC.
        """
        query = self.env['account.move.line']._search(self._get_base_domain() + [
            ('date', '<', self.date_from),
            ('account_id.include_initial_balance', '=', False),
        ])
        sql_query = query.select(SQL(
            """
                'OUV' AS JournalCode,
                'Balance initiale' AS JournalLib,
                'OUVERTURE/' || %(formatted_date_year)s AS EcritureNum,
                %(formatted_date_from)s AS EcritureDate,
                '120/129' AS CompteNum,
                'Benefice (perte) reporte(e)' AS CompteLib,
                '' AS CompAuxNum,
                '' AS CompAuxLib,
                '-' AS PieceRef,
                %(formatted_date_from)s AS PieceDate,
                '/' AS EcritureLib,
                replace(CASE WHEN COALESCE(sum(account_move_line.balance), 0) <= 0 THEN '0,00' ELSE to_char(SUM(account_move_line.balance), '000000000000000D99') END, '.', ',') AS Debit,
                replace(CASE WHEN COALESCE(sum(account_move_line.balance), 0) >= 0 THEN '0,00' ELSE to_char(-SUM(account_move_line.balance), '000000000000000D99') END, '.', ',') AS Credit,
                '' AS EcritureLet,
                '' AS DateLet,
                %(formatted_date_from)s AS ValidDate,
                '' AS Montantdevise,
                '' AS Idevise
            """,
            formatted_date_year=self.date_from.year,
            formatted_date_from=fields.Date.to_string(self.date_from).replace('-', ''),
        ))
        self.env.flush_all()
        self._cr.execute(sql_query)
        return list(self._cr.fetchone())

    def _get_company_legal_data(self, company):
        """
        Dom-Tom are excluded from the EU's fiscal territory
        Those regions do not have SIREN
        sources:
            https://www.service-public.fr/professionnels-entreprises/vosdroits/F23570
            http://www.douane.gouv.fr/articles/a11024-tva-dans-les-dom

        * Returns the siren if the company is french or an empty siren for dom-tom
        * For non-french companies -> returns the complete vat number
        """
        dom_tom_group = self.env.ref('l10n_fr.dom-tom')
        is_dom_tom = company.account_fiscal_country_id.code in dom_tom_group.country_ids.mapped('code')
        if not company.vat or is_dom_tom:
            return ''
        elif company.country_id.code == 'FR' and len(company.vat) >= 13 and siren.is_valid(company.vat[4:13]):
            return company.vat[4:13]
        else:
            return company.vat

    def generate_fec(self):
        # We choose to implement the flat file instead of the XML file for 2 reasons :
        # 1) the XSD file impose to have the label on the account.move, but Odoo has the label on the account.move.line,
        # so that's a  problem !
        # 2) CSV files are easier to read/use for a regular accountant. So it will be easier for the accountant to check
        # the file before sending it to the fiscal administration
        company = self.env.company
        company_legal_data = self._get_company_legal_data(company)

        header = [
            u'JournalCode',    # 0
            u'JournalLib',     # 1
            u'EcritureNum',    # 2
            u'EcritureDate',   # 3
            u'CompteNum',      # 4
            u'CompteLib',      # 5
            u'CompAuxNum',     # 6  We use partner.id
            u'CompAuxLib',     # 7
            u'PieceRef',       # 8
            u'PieceDate',      # 9
            u'EcritureLib',    # 10
            u'Debit',          # 11
            u'Credit',         # 12
            u'EcritureLet',    # 13
            u'DateLet',        # 14
            u'ValidDate',      # 15
            u'Montantdevise',  # 16
            u'Idevise',        # 17
            ]

        rows_to_write = [header]
        # INITIAL BALANCE
        unaffected_earnings_account = self.env['account.account'].search([
            *self.env['account.account']._check_company_domain(company),
            ('account_type', '=', 'equity_unaffected'),
        ], limit=1)
        unaffected_earnings_line = True  # used to make sure that we add the unaffected earning initial balance only once
        if unaffected_earnings_account:
            #compute the benefit/loss of last year to add in the initial balance of the current year earnings account
            unaffected_earnings_results = self._do_query_unaffected_earnings()
            unaffected_earnings_line = False

        aa_name = self.env['account.account']._field_to_sql('account_move_line__account_id', 'name')

        query = self.env['account.move.line']._search(self._get_base_domain() + [
            ('date', '<', self.date_from),
            ('account_id.include_initial_balance', '=', True),
            ('account_id.account_type', 'not in', ['asset_receivable', 'liability_payable']),
        ])
        aa_code = self.env['account.account']._field_to_sql('account_move_line__account_id', 'code', query)
        sql_query = query.select(SQL(
            """
                'OUV' AS JournalCode,
                'Balance initiale' AS JournalLib,
                'OUVERTURE/' || %(formatted_date_year)s AS EcritureNum,
                %(formatted_date_from)s AS EcritureDate,
                MIN(%(aa_code)s) AS CompteNum,
                replace(replace(MIN(%(aa_name)s), '|', '/'), '\t', '') AS CompteLib,
                '' AS CompAuxNum,
                '' AS CompAuxLib,
                '-' AS PieceRef,
                %(formatted_date_from)s AS PieceDate,
                '/' AS EcritureLib,
                replace(CASE WHEN sum(account_move_line.balance) <= 0 THEN '0,00' ELSE to_char(SUM(account_move_line.balance), '000000000000000D99') END, '.', ',') AS Debit,
                replace(CASE WHEN sum(account_move_line.balance) >= 0 THEN '0,00' ELSE to_char(-SUM(account_move_line.balance), '000000000000000D99') END, '.', ',') AS Credit,
                '' AS EcritureLet,
                '' AS DateLet,
                %(formatted_date_from)s AS ValidDate,
                '' AS Montantdevise,
                '' AS Idevise,
                MIN(account_move_line__account_id.id) AS CompteID
            """,
            formatted_date_year=self.date_from.year,
            formatted_date_from=fields.Date.to_string(self.date_from).replace('-', ''),
            aa_code=aa_code,
            aa_name=aa_name,
        ))
        self._cr.execute(SQL('%s GROUP BY account_move_line__account_id.id', sql_query))

        currency_digits = 2
        for row in self._cr.fetchall():
            listrow = list(row)
            account_id = listrow.pop()
            if not unaffected_earnings_line:
                account = self.env['account.account'].browse(account_id)
                if account.account_type == 'equity_unaffected':
                    #add the benefit/loss of previous fiscal year to the first unaffected earnings account found.
                    unaffected_earnings_line = True
                    current_amount = float(listrow[11].replace(',', '.')) - float(listrow[12].replace(',', '.'))
                    unaffected_earnings_amount = float(unaffected_earnings_results[11].replace(',', '.')) - float(unaffected_earnings_results[12].replace(',', '.'))
                    listrow_amount = current_amount + unaffected_earnings_amount
                    if float_is_zero(listrow_amount, precision_digits=currency_digits):
                        continue
                    if listrow_amount > 0:
                        listrow[11] = str(listrow_amount).replace('.', ',')
                        listrow[12] = '0,00'
                    else:
                        listrow[11] = '0,00'
                        listrow[12] = str(-listrow_amount).replace('.', ',')
            rows_to_write.append(listrow)

        #if the unaffected earnings account wasn't in the selection yet: add it manually
        if (not unaffected_earnings_line
            and unaffected_earnings_results
            and (unaffected_earnings_results[11] != '0,00'
                 or unaffected_earnings_results[12] != '0,00')):
            #search an unaffected earnings account
            unaffected_earnings_account = self.env['account.account'].search([
                ('account_type', '=', 'equity_unaffected')
            ], limit=1)
            if unaffected_earnings_account:
                unaffected_earnings_results[4] = unaffected_earnings_account.code
                unaffected_earnings_results[5] = unaffected_earnings_account.name
            rows_to_write.append(unaffected_earnings_results)

        # INITIAL BALANCE - receivable/payable
        query = self.env['account.move.line']._search(self._get_base_domain() + [
            ('date', '<', self.date_from),
            ('account_id.include_initial_balance', '=', True),
            ('account_id.account_type', 'in', ['asset_receivable', 'liability_payable']),
        ])
        query.left_join('account_move_line', 'partner_id', 'res_partner', 'id', 'partner_id')
        aa_code = self.env['account.account']._field_to_sql('account_move_line__account_id', 'code', query)
        sql_query = query.select(SQL(
            """
                'OUV' AS JournalCode,
                'Balance initiale' AS JournalLib,
                'OUVERTURE/' || %(formatted_date_year)s AS EcritureNum,
                %(formatted_date_from)s AS EcritureDate,
                MIN(%(aa_code)s) AS CompteNum,
                replace(MIN(%(aa_name)s), '|', '/') AS CompteLib,
                COALESCE(NULLIF(replace(account_move_line__partner_id.ref, '|', '/'), ''), account_move_line__partner_id.id::text) AS CompAuxNum,
                COALESCE(replace(account_move_line__partner_id.name, '|', '/'), '') AS CompAuxLib,
                '-' AS PieceRef,
                %(formatted_date_from)s AS PieceDate,
                '/' AS EcritureLib,
                replace(CASE WHEN sum(account_move_line.balance) <= 0 THEN '0,00' ELSE to_char(SUM(account_move_line.balance), '000000000000000D99') END, '.', ',') AS Debit,
                replace(CASE WHEN sum(account_move_line.balance) >= 0 THEN '0,00' ELSE to_char(-SUM(account_move_line.balance), '000000000000000D99') END, '.', ',') AS Credit,
                '' AS EcritureLet,
                '' AS DateLet,
                %(formatted_date_from)s AS ValidDate,
                '' AS Montantdevise,
                '' AS Idevise,
                MIN(account_move_line__account_id.id) AS CompteID
            """,
            formatted_date_year=self.date_from.year,
            formatted_date_from=fields.Date.to_string(self.date_from).replace('-', ''),
            aa_code=aa_code,
            aa_name=aa_name,
        ))
        self._cr.execute(SQL('%s GROUP BY account_move_line__partner_id.id, account_move_line__account_id.id', sql_query))

        for row in self._cr.fetchall():
            listrow = list(row)
            listrow.pop()
            rows_to_write.append(listrow)

        # LINES
        query_limit = int(self.env['ir.config_parameter'].sudo().get_param('l10n_fr_fec.batch_size', 500000)) # To prevent memory errors when fetching the results
        query = self.env['account.move.line']._search(
            domain=self._get_base_domain() + [
                ('date', '>=', self.date_from),
                ('date', '<=', self.date_to),
            ],
            limit=query_limit + 1,
            order='date, move_name, id',
        )
        account_alias = query.left_join('account_move_line', 'account_id', 'account_account', 'id', 'account_id')
        aa_code = self.env['account.account']._field_to_sql(account_alias, 'code', query)

        aj_name = self.env['account.journal']._field_to_sql('account_move_line__journal_id', 'name')
        columns = SQL(
            """
                REGEXP_REPLACE(replace(%(journal_alias)s.code, '|', '/'), '[\\t\\r\\n]', ' ', 'g') AS JournalCode,
                REGEXP_REPLACE(replace(%(aj_name)s, '|', '/'), '[\\t\\r\\n]', ' ', 'g') AS JournalLib,
                REGEXP_REPLACE(replace(%(move_alias)s.name, '|', '/'), '[\\t\\r\\n]', ' ', 'g') AS EcritureNum,
                TO_CHAR(%(move_alias)s.date, 'YYYYMMDD') AS EcritureDate,
                %(aa_code)s AS CompteNum,
                REGEXP_REPLACE(replace(%(aa_name)s, '|', '/'), '[\\t\\r\\n]', ' ', 'g') AS CompteLib,
                CASE WHEN %(account_alias)s.account_type IN ('asset_receivable', 'liability_payable')
                THEN
                    CASE WHEN %(partner_alias)s.ref IS null OR %(partner_alias)s.ref = ''
                    THEN %(partner_alias)s.id::text
                    ELSE replace(%(partner_alias)s.ref, '|', '/')
                    END
                ELSE ''
                END
                AS CompAuxNum,
                CASE WHEN %(account_alias)s.account_type IN ('asset_receivable', 'liability_payable')
                     THEN COALESCE(REGEXP_REPLACE(replace(%(partner_alias)s.name, '|', '/'), '[\\t\\r\\n]', ' ', 'g'), '')
                     ELSE ''
                END AS CompAuxLib,
                CASE WHEN %(move_alias)s.ref IS null OR %(move_alias)s.ref = ''
                     THEN '-'
                     ELSE REGEXP_REPLACE(replace(%(move_alias)s.ref, '|', '/'), '[\\t\\r\\n]', ' ', 'g')
                END AS PieceRef,
                TO_CHAR(COALESCE(%(move_alias)s.invoice_date, %(move_alias)s.date), 'YYYYMMDD') AS PieceDate,
                CASE WHEN account_move_line.name IS NULL OR account_move_line.name = '' THEN '/'
                     WHEN account_move_line.name SIMILAR TO '[\\t|\\s|\\n]*' THEN '/'
                     ELSE REGEXP_REPLACE(replace(account_move_line.name, '|', '/'), '[\\t\\n\\r]', ' ', 'g') END AS EcritureLib,
                replace(CASE WHEN account_move_line.debit = 0 THEN '0,00' ELSE to_char(account_move_line.debit, '000000000000000D99') END, '.', ',') AS Debit,
                replace(CASE WHEN account_move_line.credit = 0 THEN '0,00' ELSE to_char(account_move_line.credit, '000000000000000D99') END, '.', ',') AS Credit,
                CASE WHEN %(full_alias)s.id IS NULL THEN ''::text ELSE %(full_alias)s.id::text END AS EcritureLet,
                CASE WHEN account_move_line.full_reconcile_id IS NULL THEN '' ELSE TO_CHAR(%(full_alias)s.create_date, 'YYYYMMDD') END AS DateLet,
                TO_CHAR(%(move_alias)s.date, 'YYYYMMDD') AS ValidDate,
                CASE
                    WHEN account_move_line.amount_currency IS NULL OR account_move_line.amount_currency = 0 THEN ''
                    ELSE replace(to_char(account_move_line.amount_currency, '000000000000000D99'), '.', ',')
                END AS Montantdevise,
                CASE WHEN account_move_line.currency_id IS NULL THEN '' ELSE %(currency_alias)s.name END AS Idevise
            """,
            currency_alias=SQL.identifier(query.left_join('account_move_line', 'currency_id', 'res_currency', 'id', 'currency_id')),
            full_alias=SQL.identifier(query.left_join('account_move_line', 'full_reconcile_id', 'account_full_reconcile', 'id', 'full_reconcile_id')),
            journal_alias=SQL.identifier(query.left_join('account_move_line', 'journal_id', 'account_journal', 'id', 'journal_id')),
            move_alias=SQL.identifier(query.left_join('account_move_line', 'move_id', 'account_move', 'id', 'move_id')),
            partner_alias=SQL.identifier(query.left_join('account_move_line', 'partner_id', 'res_partner', 'id', 'partner_id')),
            account_alias=SQL.identifier(account_alias),
            aj_name=aj_name,
            aa_code=aa_code,
            aa_name=aa_name,
        )
        with io.StringIO() as fecfile:
            csv_writer = csv.writer(fecfile, delimiter='|', lineterminator='\r\n')

            # Write header and initial balances
            csv_writer.writerows(rows_to_write)

            # Write current period's data
            has_more_results = True
            while has_more_results:
                self._cr.execute(query.select(columns))
                query.offset += query_limit
                has_more_results = self._cr.rowcount > query_limit # we load one more result than the limit to check if there is more
                query_results = self._cr.fetchall()
                csv_writer.writerows(query_results[:query_limit])
            content = fecfile.getvalue()[:-2].encode()

        end_date = fields.Date.to_string(self.date_to).replace('-', '')
        suffix = ''
        if self.export_type == "nonofficial":
            suffix = '-NONOFFICIAL'

        # Set fiscal year lock date to the end date (not in test)
        fiscalyear_lock_date = self.env.company.fiscalyear_lock_date
        if not self.test_file and (not fiscalyear_lock_date or fiscalyear_lock_date < self.date_to):
            self.env.company.write({'fiscalyear_lock_date': self.date_to})

        return {
            'file_name': f"{company_legal_data}FEC{end_date}{suffix}.csv",
            'file_content': content,
            'file_type': 'csv'
        }

    def create_fec_report_action(self):
        # HOOK
        return

```

## File: wizard\account_fr_fec_export_wizard_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<record id="fec_export_wizard_view" model="ir.ui.view">
    <field name="name">l10n_fr.fec.export.wizard.view</field>
    <field name="model">l10n_fr.fec.export.wizard</field>
    <field name="arch" type="xml">
        <form string="FEC File Generation">
            <div class="alert alert-info" role="alert" invisible="test_file">
                When you download a FEC file, the lock date is set to the end date.
                If you want to test the FEC file generation, please tick the test file checkbox.
            </div>
            <div class="alert alert-info" role="alert" invisible="not test_file">
                You are in test mode. The FEC file generation will not set the lock date.
            </div>
            <notebook>
                <page string="Options" name="options">
                    <group>
                        <field name="date_from"/>
                        <field name="date_to"/>
                        <field name="test_file"/>
                        <field name="exclude_zero"/>
                        <field name="export_type" invisible="not test_file"/>
                        <field name="excluded_journal_ids" widget="many2many_tags" options="{'no_create': True}"/>
                    </group>
                </page>
                <page string="Technical Info" name="technical_info">
                    <group>
                        <div colspan="2">
                        The encoding of this text file is UTF-8. The structure of file is CSV separated by pipe '|'.
                        </div>
                    </group>
                    <group>
                        <table style="width:80%" colspan="2">
                            <tr>
                                <th>Technical Name</th>
                                <th>Column</th>
                                <th>Comment</th>
                            </tr>
                            <tr>
                                <td>JournalCode</td>
                                <td># 0</td>
                            </tr>
                            <tr>
                                <td>JournalLib</td>
                                <td># 1</td>
                            </tr>
                            <tr>
                                <td>EcritureNum</td>
                                <td># 2</td>
                            </tr>
                            <tr>
                                <td>EcritureDate</td>
                                <td># 3</td>
                            </tr>
                            <tr>
                                <td>CompteNum</td>
                                <td># 4</td>
                            </tr>
                            <tr>
                                <td>CompteLib</td>
                                <td># 5</td>
                            </tr>
                            <tr>
                                <td>CompAuxNum</td>
                                <td># 6</td>
                                <td>We use partner.id</td>
                            </tr>
                            <tr>
                                <td>CompAuxLib</td>
                                <td># 7</td>
                            </tr>
                            <tr>
                                <td>PieceRef</td>
                                <td># 8</td>
                            </tr>
                            <tr>
                                <td>PieceDate</td>
                                <td># 9</td>
                            </tr>
                            <tr>
                                <td>EcritureLib</td>
                                <td># 10</td>
                            </tr>
                            <tr>
                                <td>Debit</td>
                                <td># 11</td>
                            </tr>
                            <tr>
                                <td>Credit</td>
                                <td># 12</td>
                            </tr>
                            <tr>
                                <td>EcritureLet</td>
                                <td># 13</td>
                            </tr>
                            <tr>
                                <td>DateLet</td>
                                <td># 14</td>
                            </tr>
                            <tr>
                                <td>ValidDate</td>
                                <td># 15</td>
                            </tr>
                            <tr>
                                <td>Montantdevise</td>
                                <td># 16</td>
                            </tr>
                            <tr>
                                <td>Idevise</td>
                                <td># 17</td>
                            </tr>
                        </table>
                    </group>
                </page>
            </notebook>
            <footer>
                <button string="Generate" name="create_fec_report_action" type="object" class="oe_highlight" data-hotkey="q"/>
                <button string="Cancel" class="btn btn-secondary" special="cancel" data-hotkey="x"/>
            </footer>
        </form>
    </field>
</record>
</odoo>

```

## File: wizard\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_fr_fec_export_wizard

```

