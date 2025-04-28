# Odoo Module: l10n_it

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
    'name': 'Italy - Accounting',
    'version': '0.6',
    'depends': [
        'account',
        'base_iban',
        'base_vat',
    ],
    'author': 'OpenERP Italian Community',
    'description': """
Piano dei conti italiano di un'impresa generica.
================================================

Italian accounting chart and localization.
    """,
    'category': 'Accounting/Localizations/Account Charts',
    'website': 'http://www.odoo.com/',
    'data': [
        'data/account_account_tag.xml',
        'data/account_chart_template.xml',
        'data/account.account.template.csv',
        'data/account_chart_template_accounts.xml',
        'data/account.tax.group.csv',
        'data/account_tax_report_data.xml',
        'data/account_tax_template.xml',
        'data/account.fiscal.position.template.csv',
        'data/account_fiscal_position_tax_template_data.xml',
        'data/account_chart_template_data.xml',
        'data/report_invoice.xml'
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,code,name,account_type,reconcile,chart_template_id:id,tag_ids:id
1101,1101,Costi di impianto,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1106,1106,Software,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1108,1108,Avviamento,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1111,1111,Fondo ammortamento costi di impianto,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1116,1116,Fondo ammortamento software,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1118,1118,Fondo ammortamento avviamento,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1201,1201,Fabbricati,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1202,1202,Impianti e macchinari,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1204,1204,Attrezzature commerciali,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1205,1205,Macchine d'ufficio,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1206,1206,Arredamento,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1207,1207,Automezzi,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1208,1208,Imballaggi durevoli,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1211,1211,Fondo ammortamento fabbricati,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1212,1212,Fondo ammortamento impianti e macchinari,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1214,1214,Fondo ammortamento attrezzature commerciali,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1215,1215,Fondo ammortamento macchine d'ufficio,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1216,1216,Fondo ammortamento arredamento,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1217,1217,Fondo ammortamento automezzi,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1218,1218,Fondo ammortamento imballaggi durevoli,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1220,1220,Fornitori immobilizzazioni c/acconti,asset_non_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1301,1301,Mutui attivi,asset_fixed,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_ATT
1401,1401,Materie di consumo,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1404,1404,Merci,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1410,1410,Fornitori c/acconti,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1501,1501,Crediti v/clienti,asset_receivable,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1502,1502,Crediti commerciali diversi,asset_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1503,1503,Clienti c/spese anticipate,asset_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1505,1505,Cambiali attive,asset_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1506,1506,Cambiali allo sconto,asset_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1507,1507,Cambiali all'incasso,asset_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1508,1508,Crediti v/clienti (PoS),asset_receivable,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1509,1509,Fatture da emettere,asset_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1510,1510,Crediti insoluti,asset_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1511,1511,Cambiali insolute,asset_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1531,1531,Crediti da liquidare,asset_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1540,1540,Fondo svalutazione crediti,asset_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1541,1541,Fondo rischi su crediti,asset_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1601,1601,IVA n/credito,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1602,1602,IVA c/acconto,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1605,1605,Crediti per IVA,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1607,1607,Imposte c/acconto,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1608,1608,Crediti per imposte,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1609,1609,Crediti per ritenute subite,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1610,1610,Crediti per cauzioni,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1620,1620,Personale c/acconti,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1630,1630,Crediti v/istituti previdenziali,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1640,1640,Debitori diversi,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_ATT
1901,1901,Ratei attivi,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_ATT
1902,1902,Risconti attivi,asset_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_ATT
2101,2101,Patrimonio netto,equity,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PASS
2102,2102,Utile d'esercizio,income,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PASS
2103,2103,Perdita d'esercizio,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PASS
2104,2104,Prelevamenti extra gestione,income,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PASS
2105,2105,Titolare c/ritenute subite,equity,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PASS
2201,2201,Fondo per imposte,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PASS
2204,2204,Fondo responsabilità civile,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PASS
2205,2205,Fondo spese future,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PASS
2211,2211,Fondo manutenzioni programmate,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PASS
2301,2301,Debiti per TFRL,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_PASS
2410,2410,Mutui passivi,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2411,2411,Banche c/sovvenzioni,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2420,2420,Banche c/c passivi,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2421,2421,Banche c/RIBA all'incasso,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2422,2422,Banche c/cambiali all'incasso,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2423,2423,Banche c/anticipi su fatture,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2440,2440,Debiti v/altri finanziatori,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2501,2501,Debiti v/fornitori,liability_payable,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2503,2503,Cambiali passive,liability_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2520,2520,Fatture da ricevere,liability_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2521,2521,Debiti da liquidare,liability_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2530,2530,Clienti c/acconti,liability_current,TRUE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2601,2601,IVA n/debito,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2602,2602,Debiti per ritenute da versare,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2605,2605,Erario c/IVA,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2606,2606,Debiti per imposte,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2607,2607,Erario c/IVA Split Payment,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2608,2608,IVA c/Split Payment,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2619,2619,Debiti per cauzioni,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2620,2620,Personale c/retribuzioni,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2621,2621,Personale c/liquidazioni,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2622,2622,Clienti c/cessione,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2630,2630,Debiti v/istituti previdenziali,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2640,2640,Creditori diversi,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_D_PASS
2701,2701,Ratei passivi,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_E_PASS
2702,2702,Risconti passivi,liability_current,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_E_PASS
2801,2801,Bilancio di apertura,liability_current,FALSE,l10n_it_chart_template_generic,
2802,2802,Bilancio di chiusura,liability_current,FALSE,l10n_it_chart_template_generic,
2810,2810,IVA c/liquidazioni,liability_current,FALSE,l10n_it_chart_template_generic,
2811,2811,Istituti previdenziali,liability_current,FALSE,l10n_it_chart_template_generic,
2901,2901,Beni di terzi,off_balance,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_BENI
2902,2902,Depositanti beni,off_balance,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_BENI
2911,2911,Merci da ricevere,off_balance,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_IMPEGNI
2912,2912,Fornitori c/impegni,off_balance,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_IMPEGNI
2913,2913,Impegni per beni in leasing,off_balance,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_IMPEGNI
2914,2914,Creditori c/leasing,off_balance,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_IMPEGNI
2916,2916,Clienti c/impegni,off_balance,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_IMPEGNI
2917,2917,Merci da consegnare,off_balance,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_IMPEGNI
2921,2921,Rischi per effetti scontati,off_balance,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_RISCHI
2922,2922,Banche c/effetti scontati,off_balance,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_RISCHI
2926,2926,Rischi per fideiussioni,off_balance,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_RISCHI
2927,2927,Creditori per fideiussioni,off_balance,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_IMPEGNI
2931,2931,Rischi per avalli,off_balance,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_RISCHI
2932,2932,Creditori per avalli,off_balance,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_IMPEGNI
3101,3101,Merci c/vendite,income,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PL
3103,3103,Rimborsi spese di vendita,income,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PL
3110,3110,Resi su vendite,income,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PL
3111,3111,Ribassi e abbuoni passivi,income,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PL
3112,3112,Premi su vendite,income,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PL
3201,3201,Fitti attivi,income,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PL
3202,3202,Proventi vari,income,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PL
3210,3210,Arrotondamenti attivi,income,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PL
3220,3220,Plusvalenze ordinarie diverse,income,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PL
3230,3230,Sopravvenienze attive ordinarie diverse,income,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PL
3240,3240,Insussistenze attive ordinarie diverse,income,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_A_PL
4101,4101,Merci c/acquisti,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4102,4102,Materie di consumo c/acquisti,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4105,4105,Merci c/apporti,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4110,4110,Resi su acquisti,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4111,4111,Ribassi e abbuoni attivi,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4112,4112,Premi su acquisti,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4121,4121,Merci c/esistenze iniziali,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4122,4122,Materie di consumo c/esistenze iniziali,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4131,4131,Merci c/rimanenze finali,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4132,4132,Materie di consumo c/rimanenze finali,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4201,4201,Costi di trasporto,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4202,4202,Costi per energia,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4203,4203,Costi di pubblicità,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4204,4204,Costi di consulenze,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4205,4205,Costi postali,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4206,4206,Costi telefonici,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4207,4207,Costi di assicurazione,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4208,4208,Costi di vigilanza,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4209,4209,Costi per i locali,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4210,4210,Costi di esercizio automezzi,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4211,4211,Costi di manutenzione e riparazione,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4212,4212,Provvigioni passive,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4213,4213,Spese di incasso,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4301,4301,Fitti passivi,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4302,4302,Canoni di leasing,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4401,4401,Salari e stipendi,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4402,4402,Oneri sociali,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4403,4403,TFRL,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4404,4404,Altri costi per il personale,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4501,4501,Ammortamento costi di impianto,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4506,4506,Ammortamento software,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4508,4508,Ammortamento avviamento,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4601,4601,Ammortamento fabbricati,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4602,4602,Ammortamento impianti e macchinari,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4604,4604,Ammortamento attrezzature commerciali,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4605,4605,Ammortamento macchine d'ufficio,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4606,4606,Ammortamento arredamento,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4607,4607,Ammortamento automezzi,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4608,4608,Ammortamento imballaggi durevoli,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4701,4701,Svalutazioni immobilizzazioni immateriali,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4702,4702,Svalutazioni immobilizzazioni materiali,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4706,4706,Svalutazione crediti,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4814,4814,Accantonamento per responsabilità civile,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4821,4821,Accantonamento per spese future,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4823,4823,Accantonamento per manutenzioni programmate,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4901,4901,Oneri fiscali diversi,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4903,4903,Oneri vari,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4905,4905,Perdite su crediti,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4910,4910,Arrotondamenti passivi,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4920,4920,Minusvalenze ordinarie diverse,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4930,4930,Sopravvenienze passive ordinarie diverse,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
4940,4940,Insussistenze passive ordinarie diverse,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_B_PL
5110,5110,Interessi attivi v/clienti,income_other,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_PL
5115,5115,Interessi attivi bancari,income_other,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_PL
5116,5116,Interessi attivi postali,income_other,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_PL
5140,5140,Proventi finanziari diversi,income_other,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_PL
5201,5201,Interessi passivi v/fornitori,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_PL
5202,5202,Interessi passivi bancari,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_PL
5203,5203,Sconti passivi bancari,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_PL
5210,5210,Interessi passivi su mutui,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_PL
5240,5240,Oneri finanziari diversi,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_C_PL
8101,8101,Imposte dell'esercizio,expense,FALSE,l10n_it_chart_template_generic,l10n_it.account_tag_E_PL
9101,9101,Conto di risultato economico,expense,FALSE,l10n_it_chart_template_generic,
9102,9102,Stato patrimoniale,expense,FALSE,l10n_it_chart_template_generic,

```

## File: data\account.fiscal.position.template.csv

```csv
"name","chart_template_id:id","id","sequence","auto_apply","vat_required","country_id:id","country_group_id:id","note"
"Italia","l10n_it_chart_template_generic","it",1,1,1,base.it,,
"Regime Intra comunitario privato","l10n_it_chart_template_generic","intra_private",2,1,,,base.europe,
"Regime Intra comunitario","l10n_it_chart_template_generic","intra",3,1,1,,base.europe,"Fattura emessa ai sensi dell’art. 17, comma 2 del DPR 26/10/1972 n. 633, l’applicazione dell’IVA è a carico del destinatario."
"Regime Extra comunitario","l10n_it_chart_template_generic","extra",4,1,,,,
"Scissione dei Pagamenti","l10n_it_chart_template_generic","split_payment_fiscal_position",5,,,base.it,,"Operazione soggetta a split payment – il cedente non incassa l’Iva ai sensi dell’ex art.17-ter del D.P.R. 633/1972, l’acquirente è obbligato al versamento all’Agenzia delle Entrate."

```

## File: data\account.tax.group.csv

```csv
id,name,country_id/id,preceding_subtotal
tax_group_iva_2,IVA 2%,base.it,Imponibile
tax_group_iva_4,IVA 4%,base.it,Imponibile
tax_group_iva_5,IVA 5%,base.it,Imponibile
tax_group_iva_10,IVA 10%,base.it,Imponibile
tax_group_iva_12,IVA 12%,base.it,Imponibile
tax_group_iva_21,IVA 21%,base.it,Imponibile
tax_group_iva_20,IVA 20%,base.it,Imponibile
tax_group_iva_22,IVA 22%,base.it,Imponibile
tax_group_imp_esc_art_15,Imponibile Escluso Art.15,base.it,Imponibile
tax_group_fuori,Fuori Campo IVA,base.it,Imponibile
tax_group_split_payment,Scissione dei Pagamenti,base.it,Scissione dei Pagamenti Esclusa

```

## File: data\account_account_tag.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <!-- Account Tags Balance Sheet -->
        <record id="account_tag_A_ATT" model="account.account.tag">
            <field name="name">Crediti verso soci</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_B_ATT" model="account.account.tag">
            <field name="name">Immobilizzazioni</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_C_ATT" model="account.account.tag">
            <field name="name">Attivo circolante</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_D_ATT" model="account.account.tag">
            <field name="name">Ratei e risconti - Attivi</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_A_PASS" model="account.account.tag">
            <field name="name">Patrimonio Netto</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_B_PASS" model="account.account.tag">
            <field name="name">Fondi per rischi e oneri</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_C_PASS" model="account.account.tag">
            <field name="name">Trattamento di fine rapporto di lavoro subordinato</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_D_PASS" model="account.account.tag">
            <field name="name">Debiti</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_E_PASS" model="account.account.tag">
            <field name="name">Ratei e risconti - Passivi</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_RISCHI" model="account.account.tag">
            <field name="name">Rischi</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_IMPEGNI" model="account.account.tag">
            <field name="name">Impegni</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_BENI" model="account.account.tag">
            <field name="name">Beni di terzi</field>
            <field name="applicability">accounts</field>
        </record>
        <!-- Account Tags Profit & Loss -->
        <record id="account_tag_A_PL" model="account.account.tag">
            <field name="name">Valore della produzione</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_B_PL" model="account.account.tag">
            <field name="name">Costi della produzione</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_C_PL" model="account.account.tag">
            <field name="name">Proventi e oneri finanziari</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_D_PL" model="account.account.tag">
            <field name="name">Rettifiche di valore di attività e passività finanziarie</field>
            <field name="applicability">accounts</field>
        </record>
        <record id="account_tag_E_PL" model="account.account.tag">
            <field name="name">Proventi e oneri straordinari</field>
            <field name="applicability">accounts</field>
        </record>
    </data>
</odoo>

```

## File: data\account_chart_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="l10n_it_chart_template_generic" model="account.chart.template">
        <field name="name">Italy - Generic Chart of Accounts</field>
        <field name="cash_account_code_prefix">180</field>
        <field name="bank_account_code_prefix">182</field>
        <field name="transfer_account_code_prefix">183</field>
        <field name="currency_id" ref="base.EUR"/>
        <field name="country_id" ref="base.it"/>
    </record>
</odoo>

```

## File: data\account_chart_template_accounts.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_it_chart_template_generic" model="account.chart.template">
		<field name="property_account_receivable_id" ref="1501"/>
		<field name="property_account_payable_id" ref="2501"/>
		<field name="property_account_expense_categ_id" ref="4101"/>
		<field name="property_account_income_categ_id" ref="3101"/>
		<field name="income_currency_exchange_account_id" ref="3220"/>
		<field name="expense_currency_exchange_account_id" ref="4920"/>
		<field name="default_pos_receivable_account_id" ref="1508"/>
		<field name="property_tax_payable_account_id" ref="2605"/>
		<field name="property_tax_receivable_account_id" ref="2605"/>
		<field name="account_journal_early_pay_discount_loss_account_id" ref="4111"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="3111"/>
    </record>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_it.l10n_it_chart_template_generic')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_fiscal_position_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- account.fiscal.position.tax.template -->
    <record id="afpttn_it_intra_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="22v"/>
        <field name="tax_dest_id" ref="00eu"/>
    </record>
    <record id="afpttn_it_intra_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="10v"/>
        <field name="tax_dest_id" ref="00eu"/>
    </record>
    <record id="afpttn_it_intra_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="5v"/>
        <field name="tax_dest_id" ref="00eu"/>
    </record>
    <record id="afpttn_it_intra_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="4v"/>
        <field name="tax_dest_id" ref="00eu"/>
    </record>
    <record id="afpttn_it_intra_5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="00v"/>
        <field name="tax_dest_id" ref="00eu"/>
    </record>

    <record id="afpttn_it_intra_6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="22am"/>
        <field name="tax_dest_id" ref="22rcm"/>
    </record>
    <record id="afpttn_it_intra_7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="10am"/>
        <field name="tax_dest_id" ref="10rcm"/>
    </record>
    <record id="afpttn_it_intra_8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="5am"/>
        <field name="tax_dest_id" ref="5rcm"/>
    </record>
    <record id="afpttn_it_intra_9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="4am"/>
        <field name="tax_dest_id" ref="4rcm"/>
    </record>
    <record id="afpttn_it_intra_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="00am"/>
        <field name="tax_dest_id" ref="00rcm"/>
    </record>

    <record id="afpttn_it_intra_11" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="22as"/>
        <field name="tax_dest_id" ref="22rcs"/>
    </record>
    <record id="afpttn_it_intra_12" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="10as"/>
        <field name="tax_dest_id" ref="10rcs"/>
    </record>
    <record id="afpttn_it_intra_13" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="5as"/>
        <field name="tax_dest_id" ref="5rcs"/>
    </record>
    <record id="afpttn_it_intra_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="4as"/>
        <field name="tax_dest_id" ref="4rcs"/>
    </record>
    <record id="afpttn_it_intra_15" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="intra"/>
        <field name="tax_src_id"  ref="00as"/>
        <field name="tax_dest_id" ref="00rcs"/>
    </record>

    <record id="afpttn_it_split_payment_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="split_payment_fiscal_position"/>
        <field name="tax_src_id"  ref="22v"/>
        <field name="tax_dest_id" ref="22vsp_group"/>
    </record>
    <record id="afpttn_it_split_payment_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="split_payment_fiscal_position"/>
        <field name="tax_src_id"  ref="10v"/>
        <field name="tax_dest_id" ref="10vsp_group"/>
    </record>
    <record id="afpttn_it_split_payment_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="split_payment_fiscal_position"/>
        <field name="tax_src_id"  ref="5v"/>
        <field name="tax_dest_id" ref="5vsp_group"/>
    </record>
    <record id="afpttn_it_split_payment_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="split_payment_fiscal_position"/>
        <field name="tax_src_id"  ref="4v"/>
        <field name="tax_dest_id" ref="4vsp_group"/>
    </record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report_vat" model="account.report">
        <field name="name">VAT Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.it"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_vat_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_operazione_imponibile" model="account.report.line">
                <field name="name">Operazione Imponibile</field>
                <field name="code">h1</field>
                <field name="children_ids">
                    <record id="tax_report_line_vp2" model="account.report.line">
                        <field name="name">VP2 - Totale operazioni attive</field>
                        <field name="code">VP2</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">02</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp3" model="account.report.line">
                        <field name="name">VP3 - Totale operazioni passive</field>
                        <field name="code">VP3</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">03</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_iva" model="account.report.line">
                <field name="name">IVA</field>
                <field name="code">h2</field>
                <field name="children_ids">
                    <record id="tax_report_line_vp4" model="account.report.line">
                        <field name="name">VP4 - IVA esigibile</field>
                        <field name="code">VP4</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp4_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">4v</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp5" model="account.report.line">
                        <field name="name">VP5 - IVA detraibile</field>
                        <field name="code">VP5</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5v</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_saldi_riporti_e_interessi" model="account.report.line">
                <field name="name">Saldi, riporti e interessi</field>
                <field name="code">h3</field>
                <field name="children_ids">
                    <record id="tax_report_line_vp6" model="account.report.line">
                        <field name="name">VP6 - IVA dovuta</field>
                        <field name="code">VP6</field>
                        <field name="children_ids">
                            <record id="tax_report_line_vp6a" model="account.report.line">
                                <field name="name">VP6a - IVA dovuta (debito)</field>
                                <field name="code">VP6a</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_vp6a_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_vp6b" model="account.report.line">
                                <field name="name">VP6b - IVA dovuta (credito)</field>
                                <field name="code">VP6b</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_vp6b_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp7" model="account.report.line">
                        <field name="name">VP7 - Debito periodo precedente non superiore 25,82</field>
                        <field name="code">VP7</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp7_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp7</field>
                            </record>
                            <record id="tax_report_line_vp7_applied_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <record id="tax_report_line_vp7_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">VP7._applied_carryover_balance + VP7.tag</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp8" model="account.report.line">
                        <field name="name">VP8 - Credito periodo precedente</field>
                        <field name="code">VP8</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp8_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp8</field>
                            </record>
                            <record id="tax_report_line_vp8_applied_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <record id="tax_report_line_vp8_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">VP8._applied_carryover_balance + VP8.tag</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp9" model="account.report.line">
                        <field name="name">VP9 - Credito anno precedente</field>
                        <field name="code">VP9</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp9_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp9</field>
                            </record>
                            <record id="tax_report_line_vp9_applied_carryover" model="account.report.expression">
                                <field name="label">_applied_carryover_balance</field>
                                <field name="engine">external</field>
                                <field name="formula">most_recent</field>
                                <field name="date_scope">previous_tax_period</field>
                            </record>
                            <record id="tax_report_line_vp9_balance" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">VP9._applied_carryover_balance + VP9.tag</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp10" model="account.report.line">
                        <field name="name">VP10 - Versamenti auto UE</field>
                        <field name="code">VP10</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp10</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp11" model="account.report.line">
                        <field name="name">VP11 - Credito d'imposta</field>
                        <field name="code">VP11</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp11</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp12" model="account.report.line">
                        <field name="name">VP12 - Interessi dovuti per liquidazioni trimestrali</field>
                        <field name="code">VP12</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp12</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vp13" model="account.report.line">
                        <field name="name">VP13 - Acconto dovuto</field>
                        <field name="code">VP13</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vp13_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vp13</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_conto_corrente_iva" model="account.report.line">
                <field name="name">Conto corrente IVA</field>
                <field name="code">h4</field>
                <field name="children_ids">
                    <record id="tax_report_line_vp14" model="account.report.line">
                        <field name="name">VP14 - IVA da versare</field>
                        <field name="code">VP14</field>
                        <field name="children_ids">
                            <record id="tax_report_line_vp14a" model="account.report.line">
                                <field name="name">VP14a - IVA da versare (debito)</field>
                                <field name="code">VP14a</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_vp14a_vp4_vp5_dif_pos" model="account.report.expression">
                                        <field name="label">vp4_vp5_dif_pos</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_vp14a_vp4_vp5_dif_neg" model="account.report.expression">
                                        <field name="label">vp4_vp5_dif_neg</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_vp14a_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">(VP14a.vp4_vp5_dif_pos + VP7.balance + VP12.balance) - (-VP14a.vp4_vp5_dif_neg + VP8.balance + VP9.balance + VP10.balance + VP11.balance + VP13.balance)</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_vp14a_carryover" model="account.report.expression">
                                        <field name="label">_carryover_balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP14a.balance</field>
                                        <field name="subformula">if_between(EUR(0), EUR(25.82))</field>
                                        <field name="carryover_target">VP7._applied_carryover_balance</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_vp14b" model="account.report.line">
                                <field name="name">VP14b - IVA da versare (credito)</field>
                                <field name="code">VP14b</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_vp14b_vp4_vp5_dif_pos" model="account.report.expression">
                                        <field name="label">vp4_vp5_dif_pos</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_vp14b_vp4_vp5_dif_neg" model="account.report.expression">
                                        <field name="label">vp4_vp5_dif_neg</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP4.balance - VP5.balance</field>
                                        <field name="subformula">if_below(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_vp14b_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">(-VP14b.vp4_vp5_dif_neg + VP8.balance + VP9.balance + VP10.balance + VP11.balance + VP13.balance) - (VP14b.vp4_vp5_dif_pos + VP7.balance + VP12.balance)</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                    </record>
                                    <record id="tax_report_line_vp14b_carryover" model="account.report.expression">
                                        <field name="label">_carryover_balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">VP14b.balance</field>
                                        <field name="subformula">if_above(EUR(0))</field>
                                        <field name="carryover_target">VP8._applied_carryover_balance</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_turnover" model="account.report.line">
                <field name="name">Turnover</field>
                <field name="code">VE</field>
                <field name="children_ids">
                    <record id="tax_report_line_turnover_section_1" model="account.report.line">
                        <field name="name">Conferimenti di prodotti agricoli e cessioni da agricoltori esonerati (in caso di superamento di 1/3</field>
                        <field name="code">VE_section1</field>
                        <field name="children_ids">
                            <record id="tax_report_line_ve1" model="account.report.line">
                                <field name="name">VE1 - Passaggi a cooperative art.34 comma 2 con percentuale di compensazione 2%</field>
                                <field name="code">VE1</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve1</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve2" model="account.report.line">
                                <field name="name">VE2 - Passaggi a cooperative art.34 comma 2 con percentuale di compensazione 4%</field>
                                <field name="code">VE2</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve2_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve3" model="account.report.line">
                                <field name="name">VE3 - Passaggi a cooperative art.34 comma 2 con percentuale di compensazione 6,4%</field>
                                <field name="code">VE3</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve3_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve3</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve4" model="account.report.line">
                                <field name="name">VE4 - Passaggi a cooperative art.34 comma 2 con percentuale di compensazione 7,3%</field>
                                <field name="code">VE4</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve4</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve5" model="account.report.line">
                                <field name="name">VE5 - Passaggi a cooperative art.34 comma 2 con percentuale di compensazione 7,5%</field>
                                <field name="code">VE5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve6" model="account.report.line">
                                <field name="name">VE6 - Passaggi a cooperative art.34 comma 2 con percentuale di compensazione 8,3%</field>
                                <field name="code">VE6</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve6_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve6</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve7" model="account.report.line">
                                <field name="name">VE7 - Passaggi a cooperative art.34 comma 2 con percentuale di compensazione 8,5%</field>
                                <field name="code">VE7</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve7</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve8" model="account.report.line">
                                <field name="name">VE8 - Passaggi a cooperative art.34 comma 2 con percentuale di compensazione 8,8%</field>
                                <field name="code">VE8</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve8</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve9" model="account.report.line">
                                <field name="name">VE9 - Passaggi a cooperative art.34 comma 2 con percentuale di compensazione 9,5%</field>
                                <field name="code">VE9</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve9_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve9</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve10" model="account.report.line">
                                <field name="name">VE10 - Passaggi a cooperative art.34 comma 2 con percentuale di compensazione 10%</field>
                                <field name="code">VE10</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve10</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve11" model="account.report.line">
                                <field name="name">VE11 - Passaggi a cooperative art.34 comma 2 con percentuale di compensazione 12,3%</field>
                                <field name="code">VE11</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve11_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve11</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_turnover_section_2" model="account.report.line">
                        <field name="name">Operazioni imponibili agricole (art.34 comma 1) e operazioni imponibili commerciali e professional</field>
                        <field name="code">VE_section2</field>
                        <field name="children_ids">
                            <record id="tax_report_line_ve20" model="account.report.line">
                                <field name="name">VE20 - Operazioni imponibili aliquota 4%</field>
                                <field name="code">VE20</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve20_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve20</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve21" model="account.report.line">
                                <field name="name">VE21 - Operazioni imponibili aliquota 5%</field>
                                <field name="code">VE21</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve21_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve21</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve22" model="account.report.line">
                                <field name="name">VE22 - Operazioni imponibili aliquota 10%</field>
                                <field name="code">VE22</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve22_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve22</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve23" model="account.report.line">
                                <field name="name">VE23 - Operazioni imponibili aliquota 22%</field>
                                <field name="code">VE23</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve23_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve23</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_turnover_section_3" model="account.report.line">
                        <field name="name">Totale imponibile e imposta</field>
                        <field name="code">VE_section3</field>
                        <field name="children_ids">
                            <record id="tax_report_line_ve24" model="account.report.line">
                                <field name="name">VE24 - Totale righe da VE1 a VE11 e linee da VE20 a VE23</field>
                                <field name="code">VE24</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve24_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">
                                            VE1.balance + VE2.balance + VE3.balance + VE4.balance + VE5.balance
                                            + VE6.balance + VE7.balance + VE8.balance + VE9.balance + VE10.balance
                                            + VE11.balance + VE20.balance + VE21.balance + VE22.balance + VE23.balance
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve25" model="account.report.line">
                                <field name="name">VE25 - Variazioni e arrotondamenti (usare segno &#43;/&#8722;)</field>
                                <field name="code">VE25</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve25_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve25</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve26" model="account.report.line">
                                <field name="name">VE26 - Totale VE24 e VE25</field>
                                <field name="code">VE26</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve26_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">
                                            VE24.balance + VE25.balance
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_turnover_section_4" model="account.report.line">
                        <field name="name">Altre operazioni</field>
                        <field name="code">VE_section4</field>
                        <field name="children_ids">
                            <record id="tax_report_line_ve30" model="account.report.line">
                                <field name="name">VE30 - Operazioni che concorrono alla formazione del plafond</field>
                                <field name="code">VE30</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_ve30_I" model="account.report.line">
                                        <field name="name">VE30_I - Totale</field>
                                        <field name="code">VE30_I</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve30_i_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">
                                                    VE30_II.balance + VE30_III.balance + VE30_IV.balance + VE30_V.balance
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve30_ii" model="account.report.line">
                                        <field name="name">VE30_II - Esportazioni</field>
                                        <field name="code">VE30_II</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve30_ii_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve30_ii</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve30_iii" model="account.report.line">
                                        <field name="name">VE30_III - Cessioni intracomunitarie</field>
                                        <field name="code">VE30_III</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve30_iii_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve30_iii</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve30_iv" model="account.report.line">
                                        <field name="name">VE30_IV - Cessioni verso San Marino</field>
                                        <field name="code">VE30_IV</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve30_iv_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve30_iv</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve30_v" model="account.report.line">
                                        <field name="name">VE30_V - Operazioni assimilate</field>
                                        <field name="code">VE30_V</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve30_v_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve30_v</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve31" model="account.report.line">
                                <field name="name">VE31 - Operazioni non imponibili a seguito di dichiarazioni di intento</field>
                                <field name="code">VE31</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve31_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve31</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve32" model="account.report.line">
                                <field name="name">VE32 - Altre operazioni non imponibili</field>
                                <field name="code">VE32</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve32_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve32</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve33" model="account.report.line">
                                <field name="name">VE33 - Operazioni esenti (art.10</field>
                                <field name="code">VE33</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve33_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve33</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve34" model="account.report.line">
                                <field name="name">VE34 - Operazioni non soggette all’imposta ai sensi degli articoli da 7 a 7-septies</field>
                                <field name="code">VE34</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve34_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve34</field>
                                    </record>
                                </field>
                            </record>

                            <record id="tax_report_line_ve35" model="account.report.line">
                                <field name="name">VE35 - Operazioni con applicazione del reverse charge interno</field>
                                <field name="code">VE35</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_ve35_I" model="account.report.line">
                                        <field name="name">VE35_I - Total</field>
                                        <field name="code">VE35_I</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_i_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">aggregation</field>
                                                <field name="formula">
                                                    VE35_II.balance + VE35_III.balance + VE35_IV.balance
                                                    + VE35_V.balance + VE35_VI.balance + VE35_VII.balance
                                                    + VE35_VIII.balance + VE35_IX.balance
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_ii" model="account.report.line">
                                        <field name="name">VE35_II - Cessioni di rottami e altri materiali di recupero</field>
                                        <field name="code">VE35_II</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_ii_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_ii</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_iii" model="account.report.line">
                                        <field name="name">VE35_III - Cessioni di oro e argento puro</field>
                                        <field name="code">VE35_III</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_iii_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_iii</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_iv" model="account.report.line">
                                        <field name="name">VE35_IV - Subappalto nel settore edile</field>
                                        <field name="code">VE35_IV</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_iv_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_iv</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_v" model="account.report.line">
                                        <field name="name">VE35_V - Cessioni di fabbricati strumentali</field>
                                        <field name="code">VE35_V</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_v_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_v</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_vi" model="account.report.line">
                                        <field name="name">VE35_VI - Cessioni di telefoni cellulari</field>
                                        <field name="code">VE35_VI</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_vi_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_vi</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_vii" model="account.report.line">
                                        <field name="name">VE35_VII - Cessioni di prodotti elettronici</field>
                                        <field name="code">VE35_VII</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_vii_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_vii</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_viii" model="account.report.line">
                                        <field name="name">VE35_VIII - Prestazioni comparto edile e settori connessi</field>
                                        <field name="code">VE35_VIII</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_viii_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_viii</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve35_ix" model="account.report.line">
                                        <field name="name">VE35_IX - Operazioni settore energetico</field>
                                        <field name="code">VE35_IX</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve35_ix_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve35_ix</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve36" model="account.report.line">
                                <field name="name">VE36 - Operazioni non soggette all&quot;imposta effettuate nei confronti dei terremotati</field>
                                <field name="code">VE36</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve36_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve36</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve37" model="account.report.line">
                                <field name="name">VE37 - Operazioni effettuate nell&quot;anno ma con imposta esigibile negli anni successivi</field>
                                <field name="code">VE37</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_ve37_I" model="account.report.line">
                                        <field name="name">VE37_I - Total</field>
                                        <field name="code">VE37_I</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve37_i_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve37_i</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_ve37_ii" model="account.report.line">
                                        <field name="name">VE37_II - ex art. 32-bis, DL n. 83/2012</field>
                                        <field name="code">VE37_II</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_ve37_ii_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">ve37_ii</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve38" model="account.report.line">
                                <field name="name">VE38 - Operazioni nei confronti di soggetti di cui all&quot;art.17-ter</field>
                                <field name="code">VE38</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve38_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve38</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve39" model="account.report.line">
                                <field name="name">VE39 - (meno) Operazioni effettuate in anni precedenti ma con imposta esigibile nel 2022</field>
                                <field name="code">VE39</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve39_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve39</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_ve40" model="account.report.line">
                                <field name="name">VE40 - (meno) Cessioni di beni ammortizzabili e passaggi interni</field>
                                <field name="code">VE40</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_ve40_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">ve40</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_reverse_charge_iva" model="account.report.line">
                <field name="name">Reverse Charge</field>
                <field name="code">VJ</field>
                <field name="children_ids">
                    <record id="tax_report_line_vj1" model="account.report.line">
                        <field name="name">VJ1 - Acquisti di beni dalla Città del Vaticano e da San Marino</field>
                        <field name="code">VJ1</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj1</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj2" model="account.report.line">
                        <field name="name">VJ2 - Estrazione di beni da depositi Iva</field>
                        <field name="code">VJ2</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj2</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj3" model="account.report.line">
                        <field name="name">VJ3 - Acquisti di beni giá presenti in Italia o servizi, da soggetti non residenti</field>
                        <field name="code">VJ3</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj3</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj4" model="account.report.line">
                        <field name="name">VJ4 - Compensi corrisposti ai rivenditori di biglietti di viaggio ed ai rivenditori di documenti di sosta </field>
                        <field name="code">VJ4</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj4_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj4</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj5" model="account.report.line">
                        <field name="name">VJ5 - Provvigioni corrisposte dalle agenzie di viaggio ai propri intermediari</field>
                        <field name="code">VJ5</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj5</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj6" model="account.report.line">
                        <field name="name">VJ6 - Acquisti di rottami e altri materiali di recupero</field>
                        <field name="code">VJ6</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj6_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj6</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj7" model="account.report.line">
                        <field name="name">VJ7 - Acquisti di oro industriale e argento puro effettuati in Italia</field>
                        <field name="code">VJ7</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj7_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj7</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj8" model="account.report.line">
                        <field name="name">VJ8 - Acquisti di oro da investimento effettuati in Italia</field>
                        <field name="code">VJ8</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj8_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj8</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj9" model="account.report.line">
                        <field name="name">VJ9 - Acquisti intracomunitari di beni</field>
                        <field name="code">VJ9</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj9_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj9</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj10" model="account.report.line">
                        <field name="name">VJ10 - Importazioni di rottami e altri materiali di recupero</field>
                        <field name="code">VJ10</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj10</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj11" model="account.report.line">
                        <field name="name">VJ11 - Importazioni di oro industriale e argento puro</field>
                        <field name="code">VJ11</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj11</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj12" model="account.report.line">
                        <field name="name">VJ12 - Subappalto di servizi in campo edile</field>
                        <field name="code">VJ12</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj12</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj13" model="account.report.line">
                        <field name="name">VJ13 - Acquisti di fabbricati o porzioni di fabbricati strumentali</field>
                        <field name="code">VJ13</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj13_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj13</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj14" model="account.report.line">
                        <field name="name">VJ14 - Acquisti di telefoni cellulari</field>
                        <field name="code">VJ14</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj14_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj14</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj15" model="account.report.line">
                        <field name="name">VJ15 - Acquisti di prodotti elettronici</field>
                        <field name="code">VJ15</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj15_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj15</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj16" model="account.report.line">
                        <field name="name">VJ16 - Prestazioni di servizi in campo edile</field>
                        <field name="code">VJ16</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj16_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj16</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj17" model="account.report.line">
                        <field name="name">VJ17 - Acquiti di beni e servizi del settore energetico</field>
                        <field name="code">VJ17</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj17_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj17</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj18" model="account.report.line">
                        <field name="name">VJ18 - acquisti effettuati dalle pubbliche amministrazioni titolari di partita IVA</field>
                        <field name="code">VJ18</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_vj18_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">vj18</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_vj19" model="account.report.line">
                        <field name="name">VJ19 - Totale quadro VJ</field>
                        <field name="code">VJ19</field>
                        <field name="aggregation_formula">VJ1.balance + VJ2.balance + VJ3.balance + VJ4.balance + VJ5.balance + VJ6.balance + VJ7.balance + VJ8.balance + VJ9.balance + VJ10.balance + VJ11.balance + VJ12.balance + VJ13.balance + VJ14.balance + VJ15.balance + VJ16.balance + VJ17.balance + VJ18.balance</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\account_tax_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Sales (Vendita) ...................................................................... -->
    <record id="22v" model="account.tax.template">
        <field name="description">22v</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">22%</field>
        <field name="sequence">30</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp2_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp2_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="10v" model="account.tax.template">
        <field name="description">10v</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">10%</field>
        <field name="sequence">40</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp2_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp2_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="5v" model="account.tax.template">
        <field name="description">5v</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">5%</field>
        <field name="active">False</field>
        <field name="sequence">50</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp2_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp2_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="4v" model="account.tax.template">
        <field name="description">4v</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">4%</field>
        <field name="sequence">60</field>
        <field name="active">False</field>
        <field name="amount">4</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp2_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp2_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="00v" model="account.tax.template">
        <field name="description">00v</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">0%</field>
        <field name="sequence">70</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_fuori"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <!-- Purchase of Goods (Acquisti Merce) ................................................... -->
    <record id="22am" model="account.tax.template">
        <field name="description">22am</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">22% G</field>
        <field name="sequence">80</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
    </record>
    <record id="10am" model="account.tax.template">
        <field name="description">10am</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">10% G</field>
        <field name="sequence">90</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
    </record>
    <record id="5am" model="account.tax.template">
        <field name="description">5am</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">5% G</field>
        <field name="sequence">100</field>
        <field name="active">False</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
    </record>
    <record id="4am" model="account.tax.template">
        <field name="description">4am</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">4% G</field>
        <field name="sequence">110</field>
        <field name="active">False</field>
        <field name="amount">4</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
    </record>
    <record id="00am" model="account.tax.template">
        <field name="description">00am</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">0% G</field>
        <field name="sequence">120</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_fuori"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <!-- Purchase of Services (Acquisti Servizi) .............................................. -->
    <record id="22as" model="account.tax.template">
        <field name="description">22as</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">22% S</field>
        <field name="sequence">130</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
    </record>
    <record id="10as" model="account.tax.template">
        <field name="description">10as</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">10% S</field>
        <field name="sequence">140</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
    </record>
    <record id="5as" model="account.tax.template">
        <field name="description">5as</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">5% S</field>
        <field name="sequence">150</field>
        <field name="active">False</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
    </record>
    <record id="4as" model="account.tax.template">
        <field name="description">4as</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">4% S</field>
        <field name="sequence">160</field>
        <field name="active">False</field>
        <field name="amount">4</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
        ]"/>
    </record>
    <record id="00as" model="account.tax.template">
        <field name="description">00as</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">0% S</field>
        <field name="sequence">170</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_fuori"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <!-- Export ............................................................................... -->
    <record id="00eu" model="account.tax.template">
        <field name="description">00eu</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">0% EU</field>
        <field name="sequence">180</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_fuori"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp2_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp2_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <!-- No tax ............................................................................... -->
    <record id="00art15v" model="account.tax.template">
        <field name="description">00art15v</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">0% Art.15</field>
        <field name="sequence">190</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_imp_esc_art_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp2_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp2_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="00art15a" model="account.tax.template">
        <field name="description">00art15a</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">0% Art.15</field>
        <field name="sequence">200</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_imp_esc_art_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <!-- Reverse Charge Services .............................................................. -->
    <record id="22rcs" model="account.tax.template">
        <field name="description">22rcs</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">22% S RC</field>
        <field name="sequence">210</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="10rcs" model="account.tax.template">
        <field name="description">10rcs</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">10% S RC</field>
        <field name="sequence">220</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="5rcs" model="account.tax.template">
        <field name="description">5rcs</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">5% S RC</field>
        <field name="sequence">230</field>
        <field name="active">False</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="4rcs" model="account.tax.template">
        <field name="description">4rcs</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">4% S RC</field>
        <field name="sequence">240</field>
        <field name="active">False</field>
        <field name="amount">4</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="00rcs" model="account.tax.template">
        <field name="description">00rcs</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">0% S RC</field>
        <field name="sequence">250</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_fuori"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <!-- Reverse Charge Goods (Merci) ......................................................... -->
    <record id="22rcm" model="account.tax.template">
        <field name="description">22rcm</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">22% G RC</field>
        <field name="sequence">260</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="10rcm" model="account.tax.template">
        <field name="description">10rcm</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">10% G RC</field>
        <field name="sequence">270</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="5rcm" model="account.tax.template">
        <field name="description">5rcm</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">5% G RC</field>
        <field name="sequence">280</field>
        <field name="active">False</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="4rcm" model="account.tax.template">
        <field name="description">4rcm</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">4% G RC</field>
        <field name="sequence">290</field>
        <field name="active">False</field>
        <field name="amount">4</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="00rcm" model="account.tax.template">
        <field name="description">00rcm</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">0% G RC</field>
        <field name="sequence">300</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_fuori"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <!-- Reverse Charge 22% VAT deposit ....................................................... -->
    <record id="22rcd" model="account.tax.template">
        <field name="description">22rcd</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">22% G Deposit</field>
        <field name="sequence">310</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="10rcd" model="account.tax.template">
        <field name="description">10rcd</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">10% G Deposit</field>
        <field name="sequence">320</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="5rcd" model="account.tax.template">
        <field name="description">5rcd</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">5% G Deposit</field>
        <field name="sequence">330</field>
        <field name="active">False</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="4rcd" model="account.tax.template">
        <field name="description">4rcd</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">4% G Deposit</field>
        <field name="sequence">340</field>
        <field name="active">False</field>
        <field name="amount">4</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">consu</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>
    <record id="00rcd" model="account.tax.template">
        <field name="description">00rcd</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">0% G Deposit</field>
        <field name="sequence">350</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_scope">service</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_fuori"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp4_tag')]
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp3_tag'), ref('tax_report_line_vj3_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'minus_report_expression_ids': [ref('tax_report_line_vp5_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_expression_ids': [ref('tax_report_line_vp4_tag')],
            }),
        ]"/>
    </record>

    <!-- Split Payment (Scissione dei pagamenti) ......................................................... -->

    <record id="22vsp" model="account.tax.template">
        <field name="description">22% Scissione dei Pagamenti pos.</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">22% SP pos.</field>
        <field name="sequence">350</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">none</field>
        <field name="tax_group_id" ref="tax_group_iva_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'plus_report_expression_ids': [ref('tax_report_line_vp2_tag'), ref('tax_report_line_ve38_tag')],
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp2_tag'), ref('tax_report_line_ve38_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>
    <record id="22vsp_storno" model="account.tax.template">
        <field name="description">22% Scissione dei Pagamenti neg.</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">22% SP neg.</field>
        <field name="sequence">360</field>
        <field name="amount">-22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">none</field>
        <field name="tax_group_id" ref="tax_group_split_payment"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>
    <record id="22vsp_group" model="account.tax.template">
        <field name="description">22%</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">22% SP</field>
        <field name="sequence">370</field>
        <field name="amount">0</field>
        <field name="amount_type">group</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_split_payment"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('22vsp'),ref('22vsp_storno')])]"/>
    </record>

    <record id="10vsp" model="account.tax.template">
        <field name="description">10% Scissione dei Pagamenti pos.</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">10% SP pos.</field>
        <field name="sequence">350</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">none</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'plus_report_expression_ids': [ref('tax_report_line_vp2_tag'), ref('tax_report_line_ve38_tag')],
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp2_tag'), ref('tax_report_line_ve38_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>
    <record id="10vsp_storno" model="account.tax.template">
        <field name="description">10% Scissione dei Pagamenti neg.</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">10% SP neg.</field>
        <field name="sequence">360</field>
        <field name="amount">-10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">none</field>
        <field name="tax_group_id" ref="tax_group_split_payment"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>
    <record id="10vsp_group" model="account.tax.template">
        <field name="description">10%</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">10% SP</field>
        <field name="sequence">370</field>
        <field name="amount">0</field>
        <field name="amount_type">group</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_split_payment"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('10vsp'),ref('10vsp_storno')])]"/>
    </record>

    <record id="5vsp" model="account.tax.template">
        <field name="description">5% Scissione dei Pagamenti pos.</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">5% SP pos.</field>
        <field name="sequence">350</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">none</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'plus_report_expression_ids': [ref('tax_report_line_vp2_tag'), ref('tax_report_line_ve38_tag')],
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp2_tag'), ref('tax_report_line_ve38_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>
    <record id="5vsp_storno" model="account.tax.template">
        <field name="description">5% Scissione dei Pagamenti neg.</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">5% SP neg.</field>
        <field name="sequence">360</field>
        <field name="amount">-5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">none</field>
        <field name="tax_group_id" ref="tax_group_split_payment"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>
    <record id="5vsp_group" model="account.tax.template">
        <field name="description">5%</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">5% SP</field>
        <field name="sequence">370</field>
        <field name="amount">0</field>
        <field name="amount_type">group</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_split_payment"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('5vsp'),ref('5vsp_storno')])]"/>
    </record>

    <record id="4vsp" model="account.tax.template">
        <field name="description">4% Scissione dei Pagamenti pos.</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">4% SP pos.</field>
        <field name="sequence">350</field>
        <field name="amount">4</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">none</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'plus_report_expression_ids': [ref('tax_report_line_vp2_tag'), ref('tax_report_line_ve38_tag')],
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_line_vp2_tag'), ref('tax_report_line_ve38_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>
    <record id="4vsp_storno" model="account.tax.template">
        <field name="description">4% Scissione dei Pagamenti neg.</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">4% SP neg.</field>
        <field name="sequence">360</field>
        <field name="amount">-4</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">none</field>
        <field name="tax_group_id" ref="tax_group_split_payment"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('2608'),
                'use_in_tax_closing': False,
            }),
        ]"/>
    </record>
    <record id="4vsp_group" model="account.tax.template">
        <field name="description">4%</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">4% SP</field>
        <field name="sequence">370</field>
        <field name="amount">0</field>
        <field name="amount_type">group</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_split_payment"/>
        <field name="children_tax_ids" eval="[(6, 0, [ref('4vsp'),ref('4vsp_storno')])]"/>
    </record>
</odoo>

```

## File: data\report_invoice.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- TODO: remove view in master -->
    <template id="report_invoice_document" inherit_id="account.report_invoice_document">
    </template>
</odoo>

```

## File: migrations\0.5\post-update-tax-grids.py

```python
from odoo.addons.account.models.chart_template import update_taxes_from_templates

def migrate(cr, version):
    # Add the new tax tags to the credit note repartition lines
    update_taxes_from_templates(cr, 'l10n_it.l10n_it_chart_template_generic')

```

## File: migrations\0.6\post-update-tax-grids.py

```python
from odoo.addons.account.models.chart_template import update_taxes_from_templates

def migrate(cr, version):
    # Change tax tag ve38 from tax repartition lines to base repartition lines
    update_taxes_from_templates(cr, 'l10n_it.l10n_it_chart_template_generic')

```

## File: migrations\15.0.0.3\post-migrate.py

```python
# -*- coding: utf-8 -*-

def migrate(cr, version):

    cr.execute("""
        INSERT INTO account_account_account_tag
        SELECT DISTINCT account.id, template_tag.account_account_tag_id
        FROM account_account_template AS template
        JOIN account_account AS account
            ON account.code LIKE CONCAT(template.code, '%')
        JOIN account_account_template_account_tag AS template_tag
            ON template.id = template_tag.account_account_template_id
        JOIN res_company ON res_company.id = account.company_id
        JOIN res_country ON res_country.id = res_company.account_fiscal_country_id
            AND res_country.code = 'IT'
        ON CONFLICT DO NOTHING
    """)

```

## File: migrations\16.0.0.4\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.addons.account.models.chart_template import update_taxes_from_templates
from odoo import api, SUPERUSER_ID

import psycopg2
import logging

_logger = logging.getLogger(__name__)


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    account_templates = {account_code: env.ref(f'l10n_it.{account_code}') for account_code in ['2607', '2608']}
    companies = env['res.company'].search([('chart_template_id', '=', env.ref('l10n_it.l10n_it_chart_template_generic').id)])
    for company in companies:
        try:
            for account_code, account_template in account_templates.items():
                template_vals = [(account_template, company.chart_template_id._get_account_vals(company, account_template, account_code + '00', {}))]
                company.chart_template_id._create_records_with_xmlid('account.account', template_vals, company)
                _logger.info("Created split payment accounts for company: %s(%s).", company.name, company.id)
        except psycopg2.errors.UniqueViolation:
            _logger.error("Split payment accounts already exist for company: %s(%s).", company.name, company.id)

    update_taxes_from_templates(cr, 'l10n_it.l10n_it_chart_template_generic')

```

## File: models\account_chart_template.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    def _load(self, company):
        """ Set tax calculation rounding method required in Italian localization
        Also to avoid rounding errors when sent with FatturaPA"""
        res = super()._load(company)
        if company.account_fiscal_country_id.code == 'IT':
            company.write({'tax_calculation_rounding_method': 'round_globally'})
            vat_split_payment_account = self.env['account.account'].search([('company_id', '=', company.id), ('code', 'like', '2607%')])
            split_payment_tax_group = self.env.ref('l10n_it.tax_group_split_payment').with_company(company)
            split_payment_tax_group.property_tax_receivable_account_id = vat_split_payment_account
            split_payment_tax_group.property_tax_payable_account_id = vat_split_payment_account
        return res

```

## File: models\account_report.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountReportExpression(models.AbstractModel):
    _inherit = "account.report.expression"

    def _get_carryover_target_expression(self, options):
        if self.report_line_id.code == 'VP14b' and fields.Date.from_string(options['date']['date_to']).month == 12:
            # For this line, if we are between two years, we want to carry over to vp9 instead of the line set in the XML file (vp8)
            return self.env.ref('l10n_it.tax_report_line_vp9_applied_carryover')

        return super()._get_carryover_target_expression(options)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_report
from . import account_chart_template

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
      <image width="1500" height="1000" transform="translate(4.8 7.25) scale(0.03 0.03)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABdwAAAPICAYAAADDojnTAAAACXBIWXMAAUmSAAFJkgEjcqhqAAAfA0lEQVR4XuzYQRVAUAAAQZxVEUAkhUQSRBEi/MseeG/mvAl2ns79mQCAX3uOa5QAAB93r9soAQA+bhkFAAAAAADAmOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAIC3HTs0AACEARg2weN8vg/AVCa6FzRguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAAQMdwAAAAAACBjuAAAAAAAQMNwBAAAAACBguAMAAAAAQMBwBwAAAACAgOEOAAAAAAABwx0AAAAAAAKGOwAAAAAABAx3AAAAAAAIGO4AAAAAABAw3AEAAAAAIGC4AwAAAABAwHAHAAAAAICA4Q4AAAAAAAHDHQAAAAAAAoY7AAAAAAAEDHcAAAAAAAgY7gAAAAAAEDDcAQAAAAAgYLgDAAAAAEDAcAcAAAAAgIDhDgAAAAAAAcMdAAAAAAAChjsAAAAAAATOzNxfBAAAAAAAvC3cWA0qLsJhDgAAAABJRU5ErkJggg=="/>
    </g>
  </g>
</svg>

```

