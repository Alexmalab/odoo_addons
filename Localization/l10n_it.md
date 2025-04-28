# Odoo Module: l10n_it

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Italy - Accounting',
    'version': '0.2',
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
    'category': 'Localization',
    'website': 'http://www.odoo.com/',
    'data': [
        'data/l10n_it_chart_data.xml',
        'data/account.account.template.csv',
        'data/account.tax.group.csv',
        'data/account_tax_report_data.xml',
        'data/account_tax_template.xml',
        'data/account.fiscal.position.template.csv',
        'data/account.chart.template.csv',
        'data/account_chart_template_data.xml',
        ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,code,name,user_type_id:id,reconcile,chart_template_id:id
1101,1101,costi di impianto ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1106,1106,software ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1108,1108,avviamento ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1111,1111,fondo ammortamento costi di impianto ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1116,1116,fondo ammortamento software ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1118,1118,fondo ammortamento avviamento ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1201,1201,fabbricati ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1202,1202,impianti e macchinari ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1204,1204,attrezzature commerciali ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1205,1205,macchine d'ufficio ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1206,1206,arredamento ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1207,1207,automezzi ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1208,1208,imballaggi durevoli ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1211,1211,fondo ammortamento fabbricati ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1212,1212,fondo ammortamento impianti e macchinari ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1214,1214,fondo ammortamento attrezzature commerciali ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1215,1215,fondo ammortamento macchine d'ufficio ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1216,1216,fondo ammortamento arredamento ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1217,1217,fondo ammortamento automezzi ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1218,1218,fondo ammortamento imballaggi durevoli ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1220,1220,fornitori immobilizzazioni c/acconti ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1301,1301,mutui attivi ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1401,1401,materie di consumo ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1404,1404,merci ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1410,1410,fornitori c/acconti ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1501,1501,crediti v/clienti ,account.data_account_type_receivable,TRUE,l10n_it_chart_template_generic
1502,1502,crediti commerciali diversi ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1503,1503,clienti c/spese anticipate ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1505,1505,cambiali attive ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1506,1506,cambiali allo sconto ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1507,1507,cambiali all'incasso ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1508,1508,crediti v/clienti (PoS) ,account.data_account_type_receivable,TRUE,l10n_it_chart_template_generic
1509,1509,fatture da emettere ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1510,1510,crediti insoluti ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1511,1511,cambiali insolute ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1531,1531,crediti da liquidare ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1540,1540,fondo svalutazione crediti ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1541,1541,fondo rischi su crediti ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1601,1601,IVA n/credito ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1602,1602,IVA c/acconto ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1605,1605,crediti per IVA ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1607,1607,imposte c/acconto ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1608,1608,crediti per imposte ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1609,1609,crediti per ritenute subite ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1610,1610,crediti per cauzioni ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1620,1620,personale c/acconti ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1630,1630,crediti v/istituti previdenziali ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1640,1640,debitori diversi ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1901,1901,ratei attivi ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
1902,1902,risconti attivi ,account.data_account_type_current_assets,TRUE,l10n_it_chart_template_generic
2101,2101,patrimonio netto ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2102,2102,utile d'esercizio ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2103,2103,perdita d'esercizio ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2104,2104,prelevamenti extra gestione ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2105,2105,titolare c/ritenute subite ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2201,2201,fondo per imposte ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2204,2204,fondo responsabilità civile ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2205,2205,fondo spese future ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2211,2211,fondo manutenzioni programmate ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2301,2301,debiti per TFRL ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2410,2410,mutui passivi ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2411,2411,banche c/sovvenzioni ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2420,2420,banche c/c passivi ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2421,2421,banche c/RIBA all'incasso ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2422,2422,banche c/cambiali all'incasso ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2423,2423,banche c/anticipi su fatture ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2440,2440,debiti v/altri finanziatori ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2501,2501,debiti v/fornitori ,account.data_account_type_payable,TRUE,l10n_it_chart_template_generic
2503,2503,cambiali passive ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2520,2520,fatture da ricevere ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2521,2521,debiti da liquidare ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2530,2530,clienti c/acconti ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2601,2601,IVA n/debito ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2602,2602,debiti per ritenute da versare ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2605,2605,erario c/IVA ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2606,2606,debiti per imposte ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2619,2619,debiti per cauzioni ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2620,2620,personale c/retribuzioni ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2621,2621,personale c/liquidazioni ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2622,2622,clienti c/cessione ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2630,2630,debiti v/istituti previdenziali ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2640,2640,creditori diversi ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2701,2701,ratei passivi ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2702,2702,risconti passivi ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2801,2801,bilancio di apertura ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2802,2802,bilancio di chiusura ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2810,2810,IVA c/liquidazioni ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2811,2811,istituti previdenziali ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2901,2901,beni di terzi ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2902,2902,depositanti beni ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2911,2911,merci da ricevere ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2912,2912,fornitori c/impegni ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2913,2913,impegni per beni in leasing ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2914,2914,creditori c/leasing ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2916,2916,clienti c/impegni ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2917,2917,merci da consegnare ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2921,2921,rischi per effetti scontati ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2922,2922,banche c/effetti scontati ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2926,2926,rischi per fideiussioni ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2927,2927,creditori per fideiussioni ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2931,2931,rischi per avalli ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
2932,2932,creditori per avalli ,account.data_account_type_current_liabilities,TRUE,l10n_it_chart_template_generic
3101,3101,merci c/vendite ,account.data_account_type_revenue,TRUE,l10n_it_chart_template_generic
3103,3103,rimborsi spese di vendita ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
3110,3110,resi su vendite ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
3111,3111,ribassi e abbuoni passivi ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
3112,3112,premi su vendite ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
3201,3201,fitti attivi ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
3202,3202,proventi vari ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
3210,3210,arrotondamenti attivi ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
3220,3220,plusvalenze ordinarie diverse ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
3230,3230,sopravvenienze attive ordinarie diverse ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
3240,3240,insussistenze attive ordinarie diverse ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
4101,4101,merci c/acquisti ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4102,4102,materie di consumo c/acquisti ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4105,4105,merci c/apporti ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4110,4110,resi su acquisti ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4111,4111,ribassi e abbuoni attivi ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4112,4112,premi su acquisti ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4121,4121,merci c/esistenze iniziali ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4122,4122,materie di consumo c/esistenze iniziali ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4131,4131,merci c/rimanenze finali ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4132,4132,materie di consumo c/rimanenze finali ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4201,4201,costi di trasporto ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4202,4202,costi per energia ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4203,4203,costi di pubblicità ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4204,4204,costi di consulenze ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4205,4205,costi postali ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4206,4206,costi telefonici ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4207,4207,costi di assicurazione ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4208,4208,costi di vigilanza ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4209,4209,costi per i locali ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4210,4210,costi di esercizio automezzi ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4211,4211,costi di manutenzione e riparazione ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4212,4212,provvigioni passive ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4213,4213,spese di incasso ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4301,4301,fitti passivi ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4302,4302,canoni di leasing ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4401,4401,salari e stipendi ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4402,4402,oneri sociali ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4403,4403,TFRL ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4404,4404,altri costi per il personale ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4501,4501,ammortamento costi di impianto ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4506,4506,ammortamento software ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4508,4508,ammortamento avviamento ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4601,4601,ammortamento fabbricati ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4602,4602,ammortamento impianti e macchinari ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4604,4604,ammortamento attrezzature commerciali ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4605,4605,ammortamento macchine d'ufficio ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4606,4606,ammortamento arredamento ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4607,4607,ammortamento automezzi ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4608,4608,ammortamento imballaggi durevoli ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4701,4701,svalutazioni immobilizzazioni immateriali ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4702,4702,svalutazioni immobilizzazioni materiali ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4706,4706,svalutazione crediti ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4814,4814,accantonamento per responsabilità civile ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4821,4821,accantonamento per spese future ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4823,4823,accantonamento per manutenzioni programmate ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4901,4901,oneri fiscali diversi ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4903,4903,oneri vari ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4905,4905,perdite su crediti ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4910,4910,arrotondamenti passivi ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4920,4920,minusvalenze ordinarie diverse ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4930,4930,sopravvenienze passive ordinarie diverse ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
4940,4940,insussistenze passive ordinarie diverse ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
5110,5110,interessi attivi v/clienti ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
5115,5115,interessi attivi bancari ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
5116,5116,interessi attivi postali ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
5140,5140,proventi finanziari diversi ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
5201,5201,interessi passivi v/fornitori ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
5202,5202,interessi passivi bancari ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
5203,5203,sconti passivi bancari ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
5210,5210,interessi passivi su mutui ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
5240,5240,oneri finanziari diversi ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
7101,7101,plusvalenze straordinarie ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
7102,7102,sopravvenienze attive straordinarie ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
7103,7103,insussistenze attive straordinarie ,account.data_account_type_other_income,TRUE,l10n_it_chart_template_generic
7201,7201,minusvalenze straordinarie ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
7202,7202,sopravvenienze passive straordinarie ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
7203,7203,insussistenze passive straordinarie ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
7204,7204,imposte esercizi precedenti ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
8101,8101,imposte dell'esercizio ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
9101,9101,conto di risultato economico ,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic
9102,9102,stato patrimoniale,account.data_account_type_expenses,TRUE,l10n_it_chart_template_generic

```

## File: data\account.chart.template.csv

```csv
"id","property_account_receivable_id/id","property_account_payable_id/id","property_account_expense_categ_id/id","property_account_income_categ_id/id","income_currency_exchange_account_id/id","expense_currency_exchange_account_id/id","default_pos_receivable_account_id/id"
"l10n_it_chart_template_generic","1501","2501","4101","3101","3220","4920","1508"

```

## File: data\account.fiscal.position.template.csv

```csv
"name","chart_template_id:id","id","sequence","auto_apply","vat_required","country_id:id","country_group_id:id"
"Italia","l10n_it_chart_template_generic","it",1,1,1,base.it,
"Regime Extra comunitario","l10n_it_chart_template_generic","extra",4,1,,,
"Regime Intra comunitario privato","l10n_it_chart_template_generic","intra_private",2,1,,,base.europe
"Regime Intra comunitario","l10n_it_chart_template_generic","intra",3,1,1,,base.europe

```

## File: data\account.tax.group.csv

```csv
id,name
tax_group_iva_2,IVA 2%
tax_group_iva_4,IVA 4%
tax_group_iva_5,IVA 5%
tax_group_iva_10,IVA 10%
tax_group_iva_12,IVA 12%
tax_group_iva_21,IVA 21%
tax_group_iva_20,IVA 20%
tax_group_iva_22,IVA 22%
tax_group_iva_4_de_50,IVA 4% Detraibile 50%
tax_group_iva_10_de_50,IVA 10% Detraibile 50%
tax_group_iva_20_de_10,IVA 20% Detraibile 10%
tax_group_iva_20_de_15,IVA 20% Detraibile 15%
tax_group_iva_20_de_40,IVA 20% Detraibile 40%
tax_group_iva_20_de_50,IVA 20% Detraibile 50%
tax_group_iva_21_inde,IVA 21% Indetraibile
tax_group_iva_21_de_10,IVA 21% Detraibile 10%
tax_group_iva_21_de_15,IVA 21% Detraibile 15%
tax_group_iva_21_de_40,IVA 21% Detraibile 40%
tax_group_iva_21_de_50,IVA 21% Detraibile 50%
tax_group_iva_22_inde,IVA 22% Indetraibile
tax_group_iva_22_de_10,IVA 22% Detraibile 10%
tax_group_iva_22_de_15,IVA 22% Detraibile 15%
tax_group_iva_22_de_40,IVA 22% Detraibile 40%
tax_group_iva_22_de_50,IVA 22% Detraibile 50%
tax_group_imp_esc_art_15,Imponibile Escluso Art.15
tax_group_fuori,Fuori Campo IVA

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

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report_line_operazione_imponibile" model="account.tax.report.line">
        <field name="name">Operazione Imponibile</field>
        <field name="code">h1</field>
        <field name="sequence">1</field>
        <field name="formula">None</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_vp2" model="account.tax.report.line">
        <field name="name">VP2 - Totale operazioni attive</field>
        <field name="code">VP2</field>
        <field name="parent_id" ref="tax_report_line_operazione_imponibile"/>
        <field name="tag_name">02</field>
        <field name="sequence">1</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_vp3" model="account.tax.report.line">
        <field name="name">VP3 - Totale operazioni passive</field>
        <field name="code">VP3</field>
        <field name="parent_id" ref="tax_report_line_operazione_imponibile"/>
        <field name="tag_name">03</field>
        <field name="sequence">2</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_iva" model="account.tax.report.line">
        <field name="name">IVA</field>
        <field name="code">h2</field>
        <field name="sequence">2</field>
        <field name="formula">None</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_vp4" model="account.tax.report.line">
        <field name="name">VP4 - IVA esigibile</field>
        <field name="code">VP4</field>
        <field name="parent_id" ref="tax_report_line_iva"/>
        <field name="tag_name">4v</field>
        <field name="sequence">1</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_vp5" model="account.tax.report.line">
        <field name="name">VP5 - IVA detraibile</field>
        <field name="code">VP5</field>
        <field name="parent_id" ref="tax_report_line_iva"/>
        <field name="tag_name">5v</field>
        <field name="sequence">2</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_saldi_riporti_e_interessi" model="account.tax.report.line">
        <field name="name">Saldi, riporti e interessi</field>
        <field name="code">h3</field>
        <field name="sequence">3</field>
        <field name="formula">None</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_vp6" model="account.tax.report.line">
        <field name="name">VP6 - IVA dovuta</field>
        <field name="code">VP6</field>
        <field name="parent_id" ref="tax_report_line_saldi_riporti_e_interessi"/>
        <field name="sequence">1</field>
        <field name="formula">None</field>
        <field name="country_id" ref="base.it"/>
    </record>
    <record id="tax_report_line_vp6a" model="account.tax.report.line">
        <field name="name">VP6a - IVA dovuta (debito)</field>
        <field name="code">VP6a</field>
        <field name="parent_id" ref="tax_report_line_vp6"/>
        <field name="formula">VP4&gt;VP5 and VP4-VP5 or 0</field>
        <field name="sequence">1</field>
        <field name="country_id" ref="base.it"/>
    </record>
    <record id="tax_report_line_vp6b" model="account.tax.report.line">
        <field name="name">VP6b - IVA dovuta (credito)</field>
        <field name="code">VP6b</field>
        <field name="parent_id" ref="tax_report_line_vp6"/>
        <field name="formula">VP5&gt;VP4 and VP5-VP4 or 0</field>
        <field name="sequence">2</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_vp7" model="account.tax.report.line">
        <field name="name">VP7 - Debito periodo precedente non superiore 25,82</field>
        <field name="code">VP7</field>
        <field name="parent_id" ref="tax_report_line_saldi_riporti_e_interessi"/>
        <field name="sequence">2</field>
        <field name="tag_name">vp7</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_vp8" model="account.tax.report.line">
        <field name="name">VP8 - Credito periodo precedente</field>
        <field name="code">VP8</field>
        <field name="parent_id" ref="tax_report_line_saldi_riporti_e_interessi"/>
        <field name="sequence">3</field>
        <field name="tag_name">vp8</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_vp9" model="account.tax.report.line">
        <field name="name">VP9 - Credito anno precedente</field>
        <field name="code">VP9</field>
        <field name="parent_id" ref="tax_report_line_saldi_riporti_e_interessi"/>
        <field name="sequence">4</field>
        <field name="tag_name">vp9</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_vp10" model="account.tax.report.line">
        <field name="name">VP10 - Versamenti auto UE</field>
        <field name="code">VP10</field>
        <field name="parent_id" ref="tax_report_line_saldi_riporti_e_interessi"/>
        <field name="sequence">5</field>
        <field name="tag_name">vp10</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_vp11" model="account.tax.report.line">
        <field name="name">VP11 - Credito d'imposta</field>
        <field name="code">VP11</field>
        <field name="parent_id" ref="tax_report_line_saldi_riporti_e_interessi"/>
        <field name="sequence">6</field>
        <field name="tag_name">vp11</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_vp12" model="account.tax.report.line">
        <field name="name">VP12 - Interessi dovuti per liquidazioni trimestrali</field>
        <field name="code">VP12</field>
        <field name="parent_id" ref="tax_report_line_saldi_riporti_e_interessi"/>
        <field name="sequence">7</field>
        <field name="tag_name">vp12</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_vp13" model="account.tax.report.line">
        <field name="name">VP13 - Acconto dovuto</field>
        <field name="code">VP13</field>
        <field name="parent_id" ref="tax_report_line_saldi_riporti_e_interessi"/>
        <field name="sequence">8</field>
        <field name="tag_name">vp13</field>
        <field name="country_id" ref="base.it"/>
    </record>

    <record id="tax_report_line_conto_corrente_iva" model="account.tax.report.line">
        <field name="name">Conto corrente IVA</field>
        <field name="code">h4</field>
        <field name="sequence">4</field>
        <field name="formula">None</field>
        <field name="country_id" ref="base.it"/>
    </record>
    <record id="tax_report_line_vp14" model="account.tax.report.line">
        <field name="name">VP14 - IVA da versare</field>
        <field name="code">VP14</field>
        <field name="parent_id" ref="tax_report_line_conto_corrente_iva"/>
        <field name="sequence">1</field>
        <field name="formula">None</field>
        <field name="country_id" ref="base.it"/>
    </record>
    <record id="tax_report_line_vp14a" model="account.tax.report.line">
        <field name="name">VP14a - IVA da versare (debito)</field>
        <field name="code">VP14a</field>
        <field name="parent_id" ref="tax_report_line_vp14"/>
        <field name="sequence">1</field>
        <field name="formula">max(((VP4&gt;VP5 and VP4-VP5 or 0) + VP7 + VP12) - ((VP5&gt;VP4 and VP5-VP4 or 0) + VP8 + VP9 + VP10 + VP11 + VP13), 0)</field>
        <field name="country_id" ref="base.it"/>
    </record>
    <record id="tax_report_line_vp14b" model="account.tax.report.line">
        <field name="name">VP14b - IVA da versare (credito)</field>
        <field name="code">VP14b</field>
        <field name="parent_id" ref="tax_report_line_vp14"/>
        <field name="sequence">2</field>
        <field name="formula">max(((VP5&gt;VP4 and VP5-VP4 or 0) + VP8 + VP9 + VP10 + VP11 + VP13) - ((VP4&gt;VP5 and VP4-VP5 or 0) + VP7 + VP12), 0)</field>
        <field name="country_id" ref="base.it"/>
    </record>

</odoo>

```

## File: data\account_tax_template.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>

    <!-- IVA 22 -->
    <record id="22v" model="account.tax.template">
        <field name="description">22v</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">Iva al 22% (debito)</field>
        <field name="sequence">1</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_vp2')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_line_ids': [ref('tax_report_line_vp4')],
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
                'account_id': ref('2601'),
            }),
        ]"/>
    </record>

    <record id="22a" model="account.tax.template">
        <field name="description">22a</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">Iva al 22% (credito)</field>
        <field name="sequence">2</field>
        <field name="amount">22</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_vp3')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_line_ids': [ref('tax_report_line_vp5')],
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
                'account_id': ref('1601'),
            }),
        ]"/>
    </record>

    <!-- IVA 10 -->
    <record id="10v" model="account.tax.template">
        <field name="description">10v</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">Iva al 10% (debito)</field>
        <field name="sequence">5</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_vp2')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_line_ids': [ref('tax_report_line_vp4')],
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
                'account_id': ref('2601'),
            }),
        ]"/>
    </record>

    <record id="10a" model="account.tax.template">
        <field name="description">10a</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">Iva al 10% (credito)</field>
        <field name="sequence">6</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_vp3')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_line_ids': [ref('tax_report_line_vp5')],
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
                'account_id': ref('1601'),
            }),
        ]"/>
    </record>

    <!-- IVA 5 -->
    <record id="5v" model="account.tax.template">
        <field name="description">5v</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">Iva al 5% (debito)</field>
        <field name="sequence">5</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_vp2')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_line_ids': [ref('tax_report_line_vp4')],
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
                'account_id': ref('2601'),
            }),
        ]"/>
    </record>

    <record id="5a" model="account.tax.template">
        <field name="description">5a</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">Iva al 5% (credito)</field>
        <field name="sequence">6</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_vp3')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_line_ids': [ref('tax_report_line_vp5')],
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
                'account_id': ref('1601'),
            }),
        ]"/>
    </record>

    <!-- IVA 4 -->
    <record id="4v" model="account.tax.template">
        <field name="description">4v</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">Iva al 4% (debito)</field>
        <field name="sequence">17</field>
        <field name="amount">4</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_vp2')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('2601'),
                'plus_report_line_ids': [ref('tax_report_line_vp4')],
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
                'account_id': ref('2601'),
            }),
        ]"/>
    </record>

    <record id="4a" model="account.tax.template">
        <field name="description">4a</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">Iva al 4% (credito)</field>
        <field name="sequence">18</field>
        <field name="amount">4</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_iva_4"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_vp3')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('1601'),
                'plus_report_line_ids': [ref('tax_report_line_vp5')],
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
                'account_id': ref('1601'),
            }),
        ]"/>
    </record>

    <!-- IVA 0 -->
    <record id="00v" model="account.tax.template">
        <field name="description">00v</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">Fuori Campo IVA (debito)</field>
        <field name="sequence">22</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_fuori"/>
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

    <record id="00a" model="account.tax.template">
        <field name="description">00a</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">Fuori Campo IVA (credito)</field>
        <field name="sequence">23</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_fuori"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_vp2')],
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

    <record id="00art15v" model="account.tax.template">
        <field name="description">00art15v</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">Imponibile Escluso Art.15 (debito)</field>
        <field name="sequence">22</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_imp_esc_art_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_vp2')],
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

    <record id="00art15a" model="account.tax.template">
        <field name="description">00art15a</field>
        <field name="chart_template_id" ref="l10n_it_chart_template_generic"/>
        <field name="name">Imponibile Escluso Art.15 (credito)</field>
        <field name="sequence">23</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include">False</field>
        <field name="tax_group_id" ref="tax_group_imp_esc_art_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_line_vp3')],
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

</odoo>

```

## File: data\l10n_it_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- Chart Template -->
        <record id="l10n_it_chart_template_generic" model="account.chart.template">
            <field name="name">Italy - Generic Chart of Accounts</field>
            <field name="cash_account_code_prefix">180</field>
            <field name="bank_account_code_prefix">182</field>
            <field name="transfer_account_code_prefix">183</field>
            <field name="currency_id" ref="base.EUR"/>
        </record>

</odoo>

```

