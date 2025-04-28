# Odoo Module: l10n_se

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
    "name": "Sweden - Accounting",
    "version": "1.1",
    "author": "XCLUDE, Odoo SA",
    "category": "Accounting/Localizations/Account Charts",
    'description': """
Swedish Accounting
------------------

This is the base module to manage the accounting chart for Sweden in Odoo.
It also includes the invoice OCR payment reference handling.
    """,
    "depends": ["account", "base_vat"],
    "data": [
        'data/account.account.tag.csv',
        "data/account_chart_template_before_accounts.xml",
        "data/account.account.template-K3.csv",
        "data/account.account.template-K2.csv",
        "data/account.account.template.csv",
        "data/account_chart_template_after_accounts.xml",
        "data/account_tax_group_data.xml",
        "data/account_tax_report_data.xml",
        "data/account_tax_template.xml",
        "data/account_fiscal_position_template.xml",
        "data/account_fiscal_position_account_template.xml",
        "data/account_fiscal_position_tax_template.xml",
        "data/account_chart_template_configuration.xml",
        "data/res_country_data.xml",
        "views/partner_view.xml",
        "views/account_journal_view.xml",
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
 }

```

## File: data\account.account.tag.csv

```csv
id,name,applicability
account_tag_0,Subscribed but unpaid capital,accounts
account_tag_1,Intangible non-current assets,accounts
account_tag_2,Tangible non-current assets,accounts
account_tag_3,Financial assets,accounts
account_tag_4,Inventory,accounts
account_tag_5,Receivables,accounts
account_tag_6,Short-term investment,accounts
account_tag_7,Cash and bank accounts,accounts
account_tag_8,Share capital,accounts
account_tag_9,Share premium funds,accounts
account_tag_10,Reevaluation funds,accounts
account_tag_11,Misc. equity funds,accounts
account_tag_12,Balanced gain/loss,accounts
account_tag_13,Equity at the start of the financial year,accounts
account_tag_14,Deposits or withdrawals during the year,accounts
account_tag_15,Changes in the equity funds,accounts
account_tag_16,Changes in the fair value reserves,accounts
account_tag_17,Year-end results,accounts
account_tag_18,Untaxed reserves,accounts
account_tag_19,Provisions,accounts
account_tag_20,Bond loans,accounts
account_tag_21,Liabilities to credit institutions,accounts
account_tag_22,Advances from customers,accounts
account_tag_23,Account payables,accounts
account_tag_24,Liabilities to group companies,accounts
account_tag_25,Exchange liabilities,accounts
account_tag_26,Liabilities to associated companies and jointly controlled companies,accounts
account_tag_27,Liabilities to other companies in which there is an ownership interest,accounts
account_tag_28,Tax liabilities,accounts
account_tag_29,Other liabilities,accounts
account_tag_30,Accrued expenses and prepaid incomes,accounts
account_tag_31,"Operating income, stock changes, etc.",accounts
account_tag_32,Operating costs,accounts
account_tag_33,Financial items,accounts
account_tag_34,Year-end appropriations,accounts
account_tag_35,Taxes on results,accounts
account_tag_36,Results,accounts
account_tag_37,Dedicated funds,accounts
account_tag_38,Reserve funds,accounts
account_tag_39,Other equity,accounts
account_tag_40,Paid-in and issued contributions,accounts

```

## File: data\account.account.template-K2.csv

```csv
id,code,name,account_type,chart_template_id/id,tag_ids/id,reconcile
a1020,1020,Koncessioner m.m.,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1028,1028,Ackumulerade nedskrivningar på koncessioner m.m.,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1029,1029,Ackumulerade avskrivningar på koncessioner m.m.,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1038,1038,Ackumulerade nedskrivningar på patent,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1040,1040,Licenser,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1048,1048,Ackumulerade nedskrivningar på licenser,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1049,1049,Ackumulerade avskrivningar på licenser,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1050,1050,Varumärken,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1058,1058,Ackumulerade nedskrivningar på varumärken,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1059,1059,Ackumulerade avskrivningar på varumärken,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1060,1060,"Hyresrätter, tomträtter och liknande",asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1068,1068,"Ackumulerade nedskrivningar på hyresrätter, tomträtter och liknande",asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1070,1070,Goodwill,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1078,1078,Ackumulerade nedskrivningar på goodwill,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1079,1079,Ackumulerade avskrivningar på goodwill,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1080,1080,Förskott för immateriella anläggningstillgångar,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_1,False
a1111,1111,Byggnader på annans mark,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1112,1112,Byggnader på egen mark,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1118,1118,Ackumulerade nedskrivningar på byggnader,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1120,1120,Förbättringsutgifter på annans fastighet,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1129,1129,Ackumulerade avskrivningar på förbättringsutgifter på annans fastighet,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1140,1140,Tomter och obebyggda markområden,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1158,1158,Ackumulerade nedskrivningar på markanläggningar,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1180,1180,"Pågående ny-,till- och ombyggnad",asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1188,1188,Förskott för byggnader och mark,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1211,1211,Maskiner,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1213,1213,Andra tekniska anläggningar,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1218,1218,Ackumulerade nedskrivningar på maskiner och andra tekniska anläggningar,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1221,1221,Inventarier,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1222,1222,Byggnadsinventarier,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1223,1223,Markinventarier,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1225,1225,Verktyg,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1228,1228,Ackumulerade nedskrivningar på inventarier och verktyg,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1229,1229,Ackumulerade avskrivningar på inventarier och verktyg,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1230,1230,Installationer,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1231,1231,Installationer på egen fastighet,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1232,1232,Installationer på annans fastighet,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1238,1238,Ackumulerade nedskrivningar på installationer,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1239,1239,Ackumulerade avskrivningar på installationer,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1240,1240,Bilar och andra transportmedel,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1241,1241,Personbilar,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1242,1242,Lastbilar,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1243,1243,Truckar,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1244,1244,Arbetsmaskiner,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1245,1245,Traktorer,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1246,1246,"Motorcyklar, mopeder och skotrar",asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1247,1247,"Båtar, flygplan och helikoptrar",asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1248,1248,Ackumulerade nedskrivningar på bilar och andra transportmedel,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1251,1251,"Datorer, företaget",asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1257,1257,"Datorer, personal",asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1258,1258,Ackumulerade nedskrivningar på datorer,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1280,1280,Pågående nyanläggningar och förskott för maskiner och inventarier,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1281,1281,"Pågående nyanläggningar, maskiner och inventarier",asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1288,1288,Förskott för maskiner och inventarier,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1292,1292,Djur som klassificeras som anläggningstillgång,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1298,1298,Ackumulerade nedskrivningar på övriga materiella anläggningstillgångar,asset_fixed,l10nse_chart_template_K2,l10n_se.account_tag_2,False
a1310,1310,Andelar i koncernföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1311,1311,Aktier i noterade svenska koncernföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1312,1312,Aktier i onoterade svenska koncernföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1313,1313,Aktier i noterade ütlandska koncernföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1314,1314,Aktier i onoterade ütlandska koncernföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1316,1316,Andra andelar i svenska koncernföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1317,1317,Andra andelar i ütlandska koncernföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1318,1318,Ackumulerade nedskrivningar av andelar i koncernföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1320,1320,Långfristiga fordringar hos koncernföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1321,1321,Långfristiga fordringar hos moderföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1322,1322,Långfristiga fordringar hos dotterföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1323,1323,Långfristiga fordringar hos andra koncernföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1328,1328,Ackumulerade nedskrivningar av långfristiga fordringar hos koncernföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1330,1330,"Andelar i intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i",asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1331,1331,Andelar i intresseföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1332,1332,Ackumulerade nedskrivningar av andelar i intresseföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1333,1333,Andelar i gemensamt styrda företag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1334,1334,Ackumulerade nedskrivningar av andelar i gemensamt styrda företag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1336,1336,Andelar i övriga företag som det finns ett ägarintresse i,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1337,1337,Ackumulerade nedskrivningar av andelar i övriga företag som det finns ett ägarintresse i,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1338,1338,"Ackumulerade nedskrivningar av andelar i intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i",asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1340,1340,"Långfristiga fordringar hos intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i",asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,True
a1341,1341,Långfristiga fordringar hos intresseföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,True
a1342,1342,Ackumulerade nedskrivningar av långfristiga fordringar hos intresseföretag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,True
a1343,1343,Långfristiga fordringar hos gemensamt styrda företag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,True
a1344,1344,Ackumulerade nedskrivningar av långfristiga fordringar hos gemensamt styrda företag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,True
a1346,1346,Långfristiga fordringar hos övriga företag som det finns ett ägarintresse i,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,True
a1347,1347,Ackumulerade nedskrivningar av långfristiga fordringar hos övriga företag som det finns ett ägarintresse i,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,True
a1348,1348,"Ackumulerade nedskrivningar av långfristiga fordringar hos intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i",asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,True
a1351,1351,Andelar i noterade företag,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1352,1352,Andra andelar,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1353,1353,Andelar i bostadsrättsföreningar,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1354,1354,Obligationer,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1356,1356,"Andelar i ekonomiska föreningar, övriga företag",asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1357,1357,"Andelar i handelsbolag, andra företag",asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1358,1358,Ackumulerade nedskrivningar av andra andelar och värdepapper,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1360,1360,"Ackumulerade nedskrivningar av lån till delägare eller närstående enligt ABL, långfristig del",asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1369,1369,"Lån till delägare eller närstående enligt ABL, långfristig del",asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1381,1381,Långfristiga reversfordringar,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1382,1382,Långfristiga fordringar hos anställda,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1383,1383,"Lämnade depositioner, långfristiga",asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1384,1384,Derivat,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1385,1385,Värde av kapitalförsäkring,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1387,1387,Långfristiga kontraktsfordringar,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1388,1388,Långfristiga kundfordringar,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1389,1389,Ackumulerade nedskrivningar av andra långfristiga fordringar,asset_non_current,l10nse_chart_template_K2,l10n_se.account_tag_3,False
a1420,1420,Lager av tillsatsmaterial och förnödenheter,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a1429,1429,Förändring av lager av tillsatsmaterial och förnödenheter,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a1465,1465,Lager av varor VMB,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a1466,1466,Nedskrivning av varor VMB,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a1467,1467,Lager av varor VMB förenklad,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a1471,1471,"Pågående arbeten, nedlagda kostnader",asset_current,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a1478,1478,"Pågående arbeten, fakturering",asset_current,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a1480,1480,Förskott för varor och tjänster,asset_prepayments,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a1481,1481,Remburser,asset_prepayments,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a1489,1489,Övriga förskott till leverantörer,asset_prepayments,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a1491,1491,Lager av värdepapper,asset_prepayments,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a1492,1492,Lager av fastigheter,asset_prepayments,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a1493,1493,Djur som klassificeras som omsättningstillgång,asset_prepayments,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a1511,1511,Kundfordringar,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1512,1512,Belånade kundfordringar (factoring),asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1516,1516,Tvistiga kundfordringar,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1518,1518,Ej reskontrafördra kundfordringar,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1520,1520,Växelfordringar,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1525,1525,Osäkra växelfordringar,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1529,1529,Nedskrivning av växelfordringar,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1530,1530,Kontraktsfordringar,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1531,1531,Kontraktsfordringar,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1532,1532,Belånade kontraktsfordringar,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1536,1536,Tvistiga kontraktsfordringar,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1539,1539,Nedskrivning av kontraktsfordringar,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1550,1550,Konsignationsfordringar,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1560,1560,Kundfordringar hos koncernföretag,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1561,1561,Kundfordringar hos moderföretag,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1562,1562,Kundfordringar hos dotterföretag,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1563,1563,Kundfordringar hos andra koncernföretag,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1568,1568,Ej reskontrafördra kundfordringar hos koncernföretag,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1569,1569,Nedskrivning av kundfordringar hos koncernföretag,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1570,1570,"Kundfordringar hos intresseföretag, gemensamt styrda företag och övriga företag som det finns et ägarintresse i",asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1571,1571,Kundfordringar hos intresseföretag,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1572,1572,Kundfordringar hos gemensamt styrda företag,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1573,1573,Kundfordringar hos övriga företag som det finns ett ägarintresse i,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1611,1611,Reseförskott,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1612,1612,Kassaförskott,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1613,1613,Övriga förskott,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1614,1614,Tillfälliga lån till anställda,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1619,1619,Övriga fordringar hos anställda,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1620,1620,Upparbetad men ej fakturerad intäkt,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_5,True
a1660,1660,Kortfristiga fordringar hos koncernföretag,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1661,1661,Kortfristiga fordringar hos moderföretag,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1662,1662,Kortfristiga fordringar hos dotterföretag,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1663,1663,Kortfristiga fordringar hos andra koncernföretag,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1670,1670,"Kortfristiga fordringar hos intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i",asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1671,1671,Kortfristiga fordringar hos intresseföretag,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1672,1672,Kortfristiga fordringar hos gemensamt styrda företag,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1673,1673,Kortfristiga fordringar hos övriga företag som det finns ett ägarintresse i,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1681,1681,Utlägg för kunder,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1682,1682,Kortfristiga lånefordringar,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1683,1683,Derivat,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1684,1684,Kortfristiga fordringar hos leverantörer,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1685,1685,Kortfristiga fordringar hos delägare eller närstående,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1687,1687,Kortfristiga del av långfristiga fordringar,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1688,1688,Fordran arbetsmarknadsförsäkringar,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1689,1689,Övriga kortfristiga fordringar,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1690,1690,Fordringar för tecknat men ej inbetalt aktiekapital,asset_receivable,l10nse_chart_template_K2,l10n_se.account_tag_0,True
a1770,1770,Tillgångar av kostnadsnatur,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1780,1780,Upplupna avtalsintäkter,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_5,False
a1820,1820,Obligationer,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_6,False
a1830,1830,Konvertibla skuldebrev,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_6,False
a1860,1860,"Andelar i koncernföretag, kortfristigt",asset_current,l10nse_chart_template_K2,l10n_se.account_tag_6,False
a1886,1886,Derivat,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_6,False
a1889,1889,Andelar i övriga företag,asset_current,l10nse_chart_template_K2,l10n_se.account_tag_6,False
a1911,1911,Huvudkassa,asset_cash,l10nse_chart_template_K2,l10n_se.account_tag_7,False
a1912,1912,Kassa 2,asset_cash,l10nse_chart_template_K2,l10n_se.account_tag_7,False
a1913,1913,Kassa 3,asset_cash,l10nse_chart_template_K2,l10n_se.account_tag_7,False
a1950,1950,Bankcertifikat,asset_cash,l10nse_chart_template_K2,l10n_se.account_tag_7,False
a1960,1960,Koncernkonto moderföretag,asset_cash,l10nse_chart_template_K2,l10n_se.account_tag_7,False
a1970,1970,Särskilda bankkonton,asset_cash,l10nse_chart_template_K2,l10n_se.account_tag_7,False
a1972,1972,Upphovsmannakonto,asset_cash,l10nse_chart_template_K2,l10n_se.account_tag_7,False
a1973,1973,Skogskonto,asset_cash,l10nse_chart_template_K2,l10n_se.account_tag_7,False
a1974,1974,Spärrade bankmedel,asset_cash,l10nse_chart_template_K2,l10n_se.account_tag_7,False
a1979,1979,Särskilda bankkonton,asset_cash,l10nse_chart_template_K2,l10n_se.account_tag_7,False
a1980,1980,Valutakonton,asset_cash,l10nse_chart_template_K2,l10n_se.account_tag_7,False
a1990,1990,Redovisningsmedel,asset_cash,l10nse_chart_template_K2,l10n_se.account_tag_7,False
a2050,2050,Avsättning till expansionsfond,equity,l10nse_chart_template_K2,l10n_se.account_tag_19,False
a2061,2061,Eget kapital / stiftelsekapital / grundkapital,equity,l10nse_chart_template_K2,"l10n_se.account_tag_13",False
a2065,2065,Förändring i fond för verkligt värde,equity,l10nse_chart_template_K2,"l10n_se.account_tag_16",False
a2066,2066,Värdesäkringsfond,equity,l10nse_chart_template_K2,"l10n_se.account_tag_11",False
a2067,2067,Balanserad vinst eller förlust / Balanserad kapital,equity,l10nse_chart_template_K2,"l10n_se.account_tag_12",False
a2068,2068,Vinst eller förlust fran föregående år,equity,l10nse_chart_template_K2,"l10n_se.account_tag_12",False
a2069,2069,Årets resultat,equity,l10nse_chart_template_K2,"l10n_se.account_tag_17",False
a2071,2071,Ändamål 1,equity,l10nse_chart_template_K2,l10n_se.account_tag_37,False
a2072,2072,Ändamål 2,equity,l10nse_chart_template_K2,l10n_se.account_tag_37,False
a2080,2080,Bundet eget kapital,equity,l10nse_chart_template_K2,"l10n_se.account_tag_14",False
a2082,2082,Ej registrerat aktiekapital,equity,l10nse_chart_template_K2,l10n_se.account_tag_15,False
a2084,2084,Förlagsinsatser,equity,l10nse_chart_template_K2,l10n_se.account_tag_40,False
a2085,2085,Uppskrivningsfond,equity,l10nse_chart_template_K2,"l10n_se.account_tag_10",False
a2087,2087,Bunden överkursfond / Insatsemission,equity,l10nse_chart_template_K2,"l10n_se.account_tag_9",False
a2088,2088,Fond för yttre underhåll,equity,l10nse_chart_template_K2,"l10n_se.account_tag_11",False
a2089,2089,Fund för utvecklingsutgifter,equity,l10nse_chart_template_K2,"l10n_se.account_tag_11",False
a2093,2093,Erhålla aktieägartillskott,equity,l10nse_chart_template_K2,"l10n_se.account_tag_40",False
a2094,2094,Egna aktier,equity,l10nse_chart_template_K2,"l10n_se.account_tag_40",False
a2095,2095,Fusionsresultat,equity,l10nse_chart_template_K2,"l10n_se.account_tag_39",False
a2097,2097,Fri överkursfond,equity,l10nse_chart_template_K2,"l10n_se.account_tag_9",False
a2110,2110,Periodiseringsfond,equity,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2130,2130,Periodiseringsfond 2020 - nr 2,equity,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2131,2131,Periodiseringsfond 2021 - nr 2,equity,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2132,2132,Periodiseringsfond 2022 - nr 2,equity,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2133,2133,Periodiseringsfond 2023 - nr 2,equity,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2134,2134,Periodiseringsfond 2024 - nr 2,equity,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2135,2135,Periodiseringsfond 2015 - nr 2,equity,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2136,2136,Periodiseringsfond 2016 - nr 2,equity,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2137,2137,Periodiseringsfond 2017 - nr 2,equity,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2138,2138,Periodiseringsfond 2018 - nr 2,equity,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2139,2139,Periodiseringsfond 2019 - nr 2,equity,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2151,2151,Ackumulerade överavskrivningar på immateriella anläggningstillgångar,liability_non_current,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2152,2152,Ackumulerade överavskrivningar på byggnader lch markanläggningar,liability_non_current,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2153,2153,Ackumulerade överavskrivningar på maskiner och inventarier,liability_non_current,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2160,2160,Ersättningsfond,liability_non_current,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2161,2161,Ersättningsfond maskiner och inventarier,liability_non_current,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2162,2162,Ersättningsfond byggnader och markanläggningar,liability_non_current,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2164,2164,Ersättningsfond for djurlager i jordbruk och renskötsel,liability_non_current,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2190,2190,Övriga obeskattade reserver,liability_non_current,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2196,2196,Lagerreserv,liability_non_current,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2199,2199,Övriga obeskattade reserver,liability_non_current,l10nse_chart_template_K2,"l10n_se.account_tag_18",False
a2230,2230,Övriga avsättningar för pensioner och liknande förpliktelser,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_19,False
a2250,2250,Övriga avsättningar för skatter,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_19,False
a2252,2252,Avsättningar för tvistiga skatter,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_19,False
a2253,2253,Avsättningar särskild löneskatt deklarationspost,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_19,False
a2290,2290,Övriga avsättningar,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_19,False
a2310,2310,Obligations och förlagslån,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_20,False
a2320,2320,Konvertibla lån och liknade,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_20,False
a2321,2321,Konvertibla lån,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_20,False
a2322,2322,Lån förenade med optionsrätt,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_20,False
a2323,2323,Vinstandelslån,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_20,False
a2324,2324,Kapitalandelslån,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_20,False
a2331,2331,Utnyttjad checkräkningskredit 1,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_21,False
a2332,2332,Utnyttjad checkräkningskredit 2,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_21,False
a2335,2335,Beviljad checkräkningskredit 1,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_21,False
a2336,2336,Beviljad checkräkningskredit 2,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_21,False
a2340,2340,Byggnadskreditiv,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_21,False
a2351,2351,"Fastighetslån, långfristig del",liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_21,False
a2355,2355,Långfristiga lån i utländsk valuta från kreditinstitut,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_21,False
a2359,2359,Övriga långfristiga lån från kreditinstitut,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_21,False
a2360,2360,Långfristiga skulder till koncernföretag,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_24,False
a2361,2361,Långfristiga skulder till moderföretag,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_24,False
a2362,2362,Långfristiga skulder till dotterföretag,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_24,False
a2363,2363,Långfristiga skulder till andra koncernföretag,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_24,False
a2370,2370,"Långfristiga skulder till intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i",liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_26,False
a2371,2371,Långfristiga skulder till intresseföretag,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_26,False
a2372,2372,Långfristiga skulder till gemensamt styrda företag,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_26,False
a2373,2373,Långfristiga skulder till intresseföretag övriga företag som det finns ett ägarintresse i,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_27,False
a2391,2391,"Avbetalningskontrakt, långfristiga del",liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2392,2392,Villkorliga långfristiga skulder,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2394,2394,Långfristiga leverantörskrediter,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2395,2395,Andra långfristiga lån i utländsk valuta,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2396,2396,Derivat,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2397,2397,"Mottagna depositioner, långfristiga",liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2399,2399,Övriga långfristiga skulder,liability_non_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2411,2411,Kortfristiga lån från kreditinstitut,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_21,False
a2412,2412,"Byggnadskreditiv, kortfristig del",liability_current,l10nse_chart_template_K2,l10n_se.account_tag_21,False
a2417,2417,Kortfristiga del av långfristiga skulder till kreditinstitut,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_21,False
a2419,2419,Övriga kortfristiga skulder till kreditinstitut,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_21,False
a2421,2421,Ej inlösta presentkort,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_22,False
a2429,2429,Övriga förskott från kunder,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_22,False
a2430,2430,Pågående arbeten,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a2431,2431,"Pågående arbeten, fakturering",liability_current,l10nse_chart_template_K2,l10n_se.account_tag_4,True
a2438,2438,"Pågående arbeten, nedlagda kostnader",liability_current,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a2439,2439,Beräknad förändring av pågående arbeten,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_4,False
a2441,2441,Leverantörsskulder,liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_23,True
a2443,2443,Konsignationsskulder,liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_23,True
a2445,2445,Tvistiga leverantörsskulder,liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_23,True
a2448,2448,Ej reskontrafördra leverantörsskulder,liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_23,True
a2450,2450,Fakturerad men ej upparbetad intäkt,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_23",False
a2460,2460,Leverantörsskulder till koncernföretag,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_24",False
a2461,2461,Leverantörsskulder till moderföretag,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_24",False
a2462,2462,Leverantörsskulder till dotterföretag,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_24",False
a2463,2463,Leverantörsskulder till andra koncernföretag,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_24",False
a2470,2470,"Leverantörsskulder till intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i",liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_26",False
a2471,2471,Leverantörsskulder till intresseföretag,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_26",False
a2472,2472,Leverantörsskulder till gemensamt styrda företag,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_26",False
a2473,2473,Leverantörsskulder till övriga företag som det finns ett ägarintresse i,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_27",False
a2491,2491,Avräkning spelarrangörer,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_21,False
a2492,2492,Växelskulder,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_25",False
a2499,2499,Andra övriga kortfristiga skulder,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_29",False
a2512,2512,Beräknad inkomstskatt,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_28",False
a2513,2513,Beräknad fastighetsskatt/fastighetsavgift,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_28",False
a2514,2514,Beräknad särskild löneskatt på pensionskostnader,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_28",False
a2515,2515,Beräknad avkastningsskatt,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_28",False
a2517,2517,Beräknad utländsk skatt,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_28",False
a2518,2518,Betald F-skatt,liability_current,l10nse_chart_template_K2,"l10n_se.account_tag_28",False
a2618,2618,"Vilande utgående moms, 25 %",liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2628,2628,"Vilande utgående moms, 12 %%",liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2638,2638,"Vilande utgående moms, 6 %",liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2660,2660,Särskilda punktskatter,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2661,2661,Reklamskatt,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2669,2669,Övriga punktskatter,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2731,2731,Avräkning lagstadgade sociala avgifter,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2732,2732,Avräkning särskild löneskatt,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2750,2750,Utmätning i lön m.m.,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2760,2760,Utmätning i lön m.m.,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2761,2761,Avräkning semesterlöner,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2762,2762,Semesterlönekassa,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2791,2791,Personalens intressekonto,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2792,2792,Lönsparande,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2793,2793,Gruppförsäkringspremier,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2794,2794,Fackföreningsavgifter,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2795,2795,Mätnings- och granskningsarvoden,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2799,2799,Övriga löneavdrag,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2810,2810,Avräkning för factoring och belånade kontraktsfordringar,liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_29,True
a2811,2811,Avräkning för factoring,liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_29,True
a2812,2812,Avräkning för belånade kontraktsfordringar,liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_29,True
a2821,2821,Löneskulder,liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_29,True
a2822,2822,Reseräkningar,liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_29,True
a2823,2823,"Tantiem, gratifikationer",liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_29,True
a2829,2829,Övriga kortfristiga skulder till anställda,liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_29,True
a2830,2830,Avräkning för annans räkning,liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_29,True
a2841,2841,Kortfristig del av långfristiga skulder,liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_29,True
a2849,2849,Övriga kortfristiga låneskulder,liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_29,True
a2850,2850,Avräkning för skatter och avgifter (skattekonto),liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_29,True
a2852,2852,"Anståndsbelopp för moms, arbetsgivaravgifter och personalskatt",liability_payable,l10nse_chart_template_K2,l10n_se.account_tag_29,True
a2860,2860,Kortfristiga skulder till koncernföretag,liability_payable,l10nse_chart_template_K2,"l10n_se.account_tag_24",True
a2861,2861,Kortfristiga skulder till moderföretag,liability_payable,l10nse_chart_template_K2,"l10n_se.account_tag_24",True
a2862,2862,Kortfristiga skulder till dotterföretag,liability_payable,l10nse_chart_template_K2,"l10n_se.account_tag_24",True
a2863,2863,Kortfristiga skulder till andra koncernföretag,liability_payable,l10nse_chart_template_K2,"l10n_se.account_tag_24",True
a2870,2870,"Kortfristiga skulder till intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i",liability_payable,l10nse_chart_template_K2,"l10n_se.account_tag_26",True
a2871,2871,Kortfristiga skulder till intresseföretag,liability_payable,l10nse_chart_template_K2,"l10n_se.account_tag_26",True
a2872,2872,Kortfristiga skulder till gemensamt styrda företag,liability_payable,l10nse_chart_template_K2,"l10n_se.account_tag_26",True
a2873,2873,Kortfristiga skulder till övriga företag som det finns ett ägarintresse i,liability_payable,l10nse_chart_template_K2,"l10n_se.account_tag_27",True
a2880,2880,Skuld erhållna bidrag,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2891,2891,Skulder under indrivning,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2892,2892,Inre reparationsfond/underhållsfond,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2893,2893,"Skulder till närstående personer, kortfristig del",liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2895,2895,Derivat (kortfristiga skulder),liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2897,2897,"Mottagna depositioner, kortfristiga",liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2898,2898,Outtagen vinstutdelning,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2899,2899,Övriga kortfristiga skulder,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_29,False
a2911,2911,Löneskulder,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2912,2912,Ackordsöverskott,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2919,2919,Övriga Upplupna löner,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2930,2930,Upplupna pensionskostnader,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2931,2931,Upplupna pensionsutbetalningar,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2941,2941,Upplupna upplupna lagstadgade sociala avgifter,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2942,2942,Beräknad upplupen särskild löneskatt,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2943,2943,"Beräknad upplupen särskild löneskatt på pensionskostnader, deklarationspost",liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2944,2944,Beräknad upplupen avkastningsskatt på pensionskostnader,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2951,2951,Upplupna avtalade arbetsmarknadsförsäkringar,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2959,2959,"Upplupna avtalade pensionsförsäkringsavgifter, deklarationspost",liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2971,2971,Förutbetalda hyresintäkter,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2972,2972,Förutbetalda medlemsavgifter,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2979,2979,Övriga förutbetalda intäkter,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2980,2980,Upplupna avtalskostnader,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2991,2991,Beräknat arvode för bokslut,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2992,2992,Beräknat arvode för revision,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2993,2993,Ospecificerad skuld till leverantörer,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a2998,2998,Övriga upplupna kostnader och förutbetalda intäkter,liability_current,l10nse_chart_template_K2,l10n_se.account_tag_30,False
a3511,3511,Fakturerat emballage,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3518,3518,Returnerat emballage,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3560,3560,Fakturerad kostnader till koncernföretag,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3561,3561,Fakturerad kostnader till moderföretag,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3562,3562,Fakturerad kostnader till dotterföretag,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3563,3563,Fakturerad kostnader till andra koncernföretag,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3570,3570,"Fakturerad kostnader till intresseföretag, gemensamt styrda företag och övriga företag som det finns ägarintresse i",income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3590,3590,Övriga fakturerade kostnader,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3610,3610,Försäljning av material,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3611,3611,Försäljning av råmaterial,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3612,3612,Försäljning av skrot,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3613,3613,Försäljning av förbrukningsmaterial,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3619,3619,Försäljning av övrigt material,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3620,3620,Tillfällig uthyrning av personal,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3630,3630,Tillfällig av transportmedel,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3670,3670,Intäkter från värdepapper,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3671,3671,Försäljning av värdepapper,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3672,3672,Utdelning från värdepapper,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3679,3679,övriga intäkter från värdepapper,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3680,3680,Management fees,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3690,3690,Övriga sidointäkter,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3700,3700,Intäktskorrigeringar (gruppkonto),income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3710,3710,Oförelade intäktsreduktioner,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3731,3731,Lämnade kassarabatter,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3732,3732,Lämnade mängdrabatter,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3750,3750,Punktskatter,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3751,3751,Intäktsförda punktskatter (kreditkonto),income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3752,3752,Skuldförda punktskatter (debetkonto),income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3790,3790,övriga intäktskorrigeringar,income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3840,3840,Aktiverat arbete (material),income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3850,3850,Aktiverat arbete (omkostnader),income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3870,3870,Aktiverat arbete (personal),income,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3910,3910,Hyres- och arrendeintäkter,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3911,3911,Hyresintäkter,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3912,3912,Arrendeintäkter,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3914,3914,Övriga momspliktiga hyresintäkter,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3920,3920,"Provisionsintäkter, licensintäkter och royalties",income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3921,3921,Provisionsintäkter,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3922,3922,Licensintäkter och royalties,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3925,3925,Franchiseintäkter,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3950,3950,"Återvunna, tidigare, avskrivna kundfordringar",income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3971,3971,Vinst vid avyttring av immateriella anläggningstillgångar,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3972,3972,Vinst vid avyttring av byggnader och mark,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3973,3973,Vinst vid avyttring av maskiner och inventarier,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3981,3981,Erhållna EU-bidrag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3985,3985,Erhållna statliga bidrag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3987,3987,Erhållna kommunala bidrag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3988,3988,Erhållna bidrag och ersättningar för,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3989,3989,Övriga erhållna bidrag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3990,3990,Övriga ersättningar och intäkter,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3991,3991,Konfliktersättning,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3992,3992,Erhållna skadestånd,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3993,3993,Erhållna donationer och gåvor,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3994,3994,Försäkringsersättningar,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3995,3995,Erhållet ackord på skulder av rörelsekaraktär,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3996,3996,Erhållna reklambidrag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3997,3997,Sjuklöneersättning,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a3999,3999,Övriga rörelseintäkter,income_other,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a4416,4416,"Inköpta varor i Sverige, omvänd skattskyldighet, 12 %",expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a4417,4417,"Inköpta varor i Sverige, omvänd skattskyldighet, 6 %",expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a4425,4425,"Inköp tjänster i Sverige, omvänd skattskyldighet, 25 %",expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a4730,4730,Erhållna rabatter,expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a4731,4731,Erhållna kassarabatter,expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a4732,4732,Erhållna mängdrabatter (inkl. bonus),expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a4733,4733,Erhållna aktivitetsstöd,expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a4790,4790,Övriga reduktion av inköpspriser,expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a4944,4944,"Förändring produkter i arbete, material och utlägg",expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a4945,4945,"Förändring produkter i arbete, omkostnader",expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a4947,4947,"Förändring produkter i arbete, personalkostnader",expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a4974,4974,"Förändring pågående arbete, material och utlägg",expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a4975,4975,"Förändring pågående arbete, omkostnader",expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a4977,4977,"Förändring pågående arbete, personalkostnader",expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a4980,4980,Förändring av lager av värdepapper,expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a4981,4981,Sålda värdepappers anskaffningsvärde,expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a4987,4987,Nedskrivning av värdepapper,expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a4988,4988,Återföring av nedskrivning av värdepapper,expense_direct_cost,l10nse_chart_template_K2,l10n_se.account_tag_31,False
a5000,5000,Lokalkostnader (gruppkonto),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5011,5011,Hyra för kontorslokaler,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5012,5012,Hyra för garage,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5013,5013,Hyra för lagerlokaler,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5050,5050,Lokaltillbehör,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5061,5061,Städning,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5062,5062,Sophämtning,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5063,5063,Hyra för sopcontainer,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5064,5064,Snöröjning,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5065,5065,Trädgårdsskötsel,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5090,5090,Övriga lokalkostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5098,5098,"Övriga lokalkostnader, avdragsgilla",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5099,5099,"Övriga lokalkostnader, ej avdragsgilla",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5110,5110,Tomträttsavgäld/arrende,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5131,5131,Uppvärmning,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5132,5132,Sotning,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5161,5161,Städning,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5162,5162,Sophämtning,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5163,5163,Hyra för sopcontainer,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5164,5164,Snöröjning,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5165,5165,Trädgårdsskötsel,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5190,5190,Övriga fastighetskostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5191,5191,Fastighetsskatt/fastighetsavgift,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5192,5192,Fastighetsförsäkringspremier,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5193,5193,Fastighetsskötsel och förvaltning,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5198,5198,"Övriga fastighetskostnader, avdragsgilla",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5199,5199,"Övriga fastighetskostnader, ej avdragsgilla",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5210,5210,Hyra av maskiner och andra tekniska anläggningar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5211,5211,Korttidshyra av maskiner och andra tekniska anläggningar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5212,5212,Leasing av maskiner och andra tekniska anläggningar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5220,5220,Hyra av datorer,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5221,5221,Korttidshyra av inventarier och verktyg,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5222,5222,Leasing av inventarier och verktyg,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5250,5250,Hyra av datorer,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5251,5251,Korttidshyra av datorer,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5252,5252,Leasing av datorer,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5290,5290,Övriga hyreskostnader för anläggningstillgångar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5310,5310,El för drift,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5320,5320,Gas,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5330,5330,Eldningsolja,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5340,5340,Stenkol och koks,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5350,5350,"Torv, träkol, ved och annat träbränsle",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5360,5360,"Bensin, fotogen och motorbrännolja",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5370,5370,"Fjärrvärme, kyla och ånga",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5380,5380,Vatten,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5390,5390,Övriga energikostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5411,5411,Förbrukningsinventarier med en livslängd på mer än ett år,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5412,5412,Förbrukningsinventarier led en livslängd på ett år eller mindre,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5430,5430,Transportinventarier,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5440,5440,Förbrukningsemballage,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5480,5480,Arbetskläder och skyddsmaterial,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5490,5490,Övriga förbrukningsinventarier och förbrukningsmaterial,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5491,5491,Övriga förbrukningsinventarier med en livslängd på mer än ett år,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5492,5492,Övriga förbrukningsinventarier med en livslängd på ett år eller mindre,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5493,5493,Övrigt förbrukningsmaterial,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5510,5510,Reparation och underhåll av maskiner och andra tekniska anläggningar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5520,5520,"Reparation och underhåll av inventarier, verktyg och datorer m.m",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5530,5530,Reparation och underhåll av installationer,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5550,5550,Reparation och underhåll av förbrukningsinventarier,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5580,5580,Underhåll och tvätt av arbetskläder,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5590,5590,Övriga kostnader för reparation och underhålla,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5610,5610,Personbilar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5616,5616,"Trängselskatt, avdragsgill",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5619,5619,Övriga personbilskostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5620,5620,Lastbilskostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5630,5630,Truckkostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5640,5640,Kostnader för arbetsmaskiner,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5650,5650,Traktorkostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5660,5660,"Motorcykel-, moped-, och skoterkostnader",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5670,5670,"Båt-, flygplans- och helikopterkostnader",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5690,5690,Övriga kostnader för transportmedel,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5710,5710,"Frakter, transporter och försäkringar vid varudistribution",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5720,5720,Tull- och speditionskostnader m.m.,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5730,5730,Arbetstransporter,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5790,5790,Övriga kostnader för frakter och transporter,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5830,5830,Host och logi,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5890,5890,övriga resekostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5910,5910,Annonsering,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5920,5920,Utomhus- och trafikreklam,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5930,5930,Reklamtrycksaker och direktreklam,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5940,5940,Utställningar mässor,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5950,5950,Butiksreklam och återförsäljarreklam,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5960,5960,"Varuprover, reklamgåvor, presentreklam och tävlingar",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5970,5970,"Film-, radio-, TV- och Internetreklam",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5980,5980,"PR, institutionell reklam och sponsring",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a5990,5990,Övriga kostnader for reklam och PR,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6000,6000,Övriga försäljningskostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6010,6010,"Kataloger, prislistor m. m.",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6020,6020,Egna facktidskrifter,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6030,6030,Speciella orderkostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6040,6040,Kontokortsavgifter,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6050,6050,Försäljningsprovisioner,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6055,6055,Franchisekostnader o.dyl.,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6060,6060,Kreditförsäljningskostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6061,6061,Kreditupplysning,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6062,6062,Inkasso och KFM-avgifter,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6063,6063,Kreditförsäkringspremier,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6064,6064,Factoringsavgifter,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6069,6069,Övriga kreditförsäljningskostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6070,6070,Representation,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6080,6080,Bankgarantier,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6110,6110,Kontorsmateriel,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6150,6150,Trycksaker,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6200,6200,Tele och post (gruppkonto),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6211,6211,Fast telefoni,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6212,6212,Mobiltelefon,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6213,6213,Mobilsökning,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6214,6214,Fax,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6215,6215,Telex,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6230,6230,Datakommunikation,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6250,6250,Postbefordran,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6300,6300,Företagsförsäkringar och övriga riskkostnader (gruppkonto),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6320,6320,Självrisker vid skada,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6330,6330,Förluster i pågående arbeten,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6340,6340,Lämnade skadestånd,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6341,6341,"Lämnade skadestånd, avdragsgilla",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6342,6342,"Lämnade skadestånd, ej avdragsgilla",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6350,6350,Förluster på kundfordringar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6351,6351,Konstaterade förluster på kundfordringar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6352,6352,Befarade förluster på kundfordringar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6360,6360,Garantikostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6361,6361,Förändring av garantiavsättning,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6362,6362,Faktiska garantikostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6370,6370,Kostnader för bevakning och lam,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6380,6380,Förluster på övriga kortfristiga fordringar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6400,6400,Förvaltningskostnader (gruppkonto),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6421,6421,Revision,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6422,6422,Revisionsverksamhet utöver revision,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6423,6423,Skatterådgivning - revisor,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6424,6424,Övriga tjänster - revisor,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6430,6430,Management fees,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6440,6440,Årsredovisning och delårsrapporter,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6450,6450,Bolagsstämma/års- eller föreningsstämma,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6490,6490,Övriga förvaltningskostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6500,6500,Övriga externa tjänster (gruppkonto),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6510,6510,Mätningskostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6520,6520,Ritnings-och kopieringskostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6810,6810,Inhyrd produktionspersonal,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6820,6820,Inhyrd lagerpersonal,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6830,6830,Inhyrd transportpersonal,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6840,6840,Inhyrd kontors- och ekonomipersonal,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6850,6850,Inhyrd IT-personal,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6860,6860,Inhyrd marknads- och försäljningspersonal,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6870,6870,Inhyrd restaurang- och butikspersonal,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6880,6880,Inhyrda företagsledare,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6890,6890,Övriga inhyrd personal,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6900,6900,Övriga externa kostnader (gruppkonto),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6910,6910,Licensavgifter och royalties),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6920,6920,Kostnader för egna patent,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6930,6930,Kostnader för varumärken m.m.,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6940,6940,"Kontroll-, provnings- och stämpelavgifter",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6950,6950,Tillsynsavgifter myndigheter,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6981,6981,"Föreningsavgifter, avdragsgilla",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6982,6982,"Föreningsavgifter, ej avdragsgilla",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6990,6990,Övriga externa kostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6993,6993,Lämnade bidrag och gåvor,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6996,6996,Betald utländsk inkomstskatt,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6997,6997,Obetald utländsk inkomstskatt,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6998,6998,Utländsk moms,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a6999,6999,"Ingående moms, blandad verksamhet",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7000,7000,Löner till kollektivanställda (gruppkonto),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7010,7010,Löner till kollektivanställda,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7011,7011,Löner till kollektivanställda,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7012,7012,Vinstandelar till kollektivanställda,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7013,7013,Lön växa-stöd kollektivanställda 10.21%,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7017,7017,Avgångsvederlag till kollektivanställda,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7018,7018,"Bruttolöneavdrag, kollektivanställda",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7019,7019,Upplupna löner och vinstandelar till kollektivanställda,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7030,7030,Löner till kollektivanställda (utlandsanställda),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7031,7031,Löner till kollektivanställda (utlandsanställda),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7032,7032,Vinstandelar till kollektivanställda (utlandsanställda),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7037,7037,Avgångsvederlag till kollektivanställda (utlandsanställda),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7038,7038,"Bruttolöneavdrag, kollektivanställda (utlandsanställda)",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7039,7039,Upplupna löner och vinstandelar till kollektivanställda (utlandsanställda),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7080,7080,Löner till kollektivanställda for ej arbetad tid,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7081,7081,Sjuklöner till kollektivanställda,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7082,7082,Semesterlöner till kollektivanställda,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7083,7083,Föräldraersättning till kollektivanställda,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7089,7089,Övriga löner till kollektivanställda for ej arbetad tid,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7200,7200,Löner till tjänstemän och företagsledare (gruppkonto),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7211,7211,Löner till tjänstemän,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7212,7212,Vinstandelar till tjänstemän,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7213,7213,Lön växa-stöd tjänstemän 10.21%,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7217,7217,Avgångsvederlag till tjänstemän,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7218,7218,"Bruttolöneavdrag tjänstemän",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7219,7219,Upplupna löner och vinstandelar till tjänstemän,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7221,7221,Löner till företagsledare,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7222,7222,Tantiem till företagsledare,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7227,7227,Avgångsvederlag till företagsledare,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7228,7228,"Bruttolöneavdrag, företagsledare",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7229,7229,Upplupna löner och tantiem till företagsledare,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7230,7230,Löner till tjänstemän och ftgsledare (utlandsanställda),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7231,7231,Löner till tjänstemän och ftgsledare (utlandsanställda),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7232,7232,Vinstandelar till tjänstemän och ftgsledare (utlandsanställda),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7237,7237,Avgångsvederlag till tjänstemän (utlandsanställda),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7238,7238,"Bruttolöneavdrag, tjänstemän och ftgsledare (utlandsanställda)",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7239,7239,Upplupna löner och vinstandelar till tjänstemän och ftgsledare (utlandsanställda),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7280,7280,Löner till tjänstemän och företagsledare för ej arbetad tid,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7281,7281,Sjuklöner till tjänstemän,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7282,7282,Sjuklöner till företagsledare,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7283,7283,Föräldraersättning till tjänstemän,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7284,7284,Föräldraersättning till företagsledare,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7285,7285,Semesterlöner till tjänstemän,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7286,7286,Semesterlöner till företagsledare,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7288,7288,Övriga löner till tjänsterumän för ej arbetad tid,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7289,7289,Övriga löner till företagsledare för ej arbetad tid,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7291,7291,Förändring av semesterlöneskuld till tjänstemän,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7292,7292,Förändring av semesterlöneskuld till företagsledare,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7300,7300,MANUALLY ADDED Kostnadsersättningar och förmåner (gruppkonto),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7311,7311,Ersättningar för sammanträden m.m.,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7312,7312,Ersättningar för förslagsverksamhet och uppfinningar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7313,7313,Ersättningar för/bidrag till bostadskostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7314,7314,Ersättningar för/bidrag till måltidskostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7315,7315,Ersättningar för/bidrag till resor till och från arbetsplatsen,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7316,7316,Ersättningar för/bidrag till arbetskläder,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7317,7317,Ersättningar för/bidrag till arbetsmaterial och arbetsverktyg,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7318,7318,Felräkningspengar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7319,7319,Övriga kontanta extraersättningar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7320,7320,Traktamenten vid tjänsteresa,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7330,7330,Bilersättningar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7333,7333,"Ersättning för trängselskatt, skattefri",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7350,7350,Ersättningar för föreskrivna arbetskläder,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7370,7370,Representationsersättningar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7381,7381,Kostnader för fri bostad,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7382,7382,Kostnader för fria eller subventionerade måltider,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7383,7383,Kostnader för fria resor till och från arbetsplatsen,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7384,7384,Kostnader för fria eller subventionerade arbetskläder,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7386,7386,Subventionerad ränta,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7387,7387,Kostnader för lånedatorer,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7388,7388,Anställdas ersättning för erhållna förmåner,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7389,7389,Övriga kostnader för förmåner,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7391,7391,Kostnad för trängelskatterförmån,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7392,7392,Kostnad för förmån av hushållsnära tjänster,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7400,7400,Pensionskostnader (gruppkonto),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7411,7411,Premier för kollektiva pensionsförsäkringar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7412,7412,Premier för individuella pensionsförsäkringar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7418,7418,Återbäring från försäkringsföretag,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7420,7420,Förändring av pensionsskuld,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7430,7430,Avdrag för räntedel i pensionskostnad,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7440,7440,Förändring av pensionsstiftelse,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7441,7441,Avsättning till pensionsstiftelse,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7448,7448,Gottgörelse från pensionsstiftelse,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7460,7460,Pensionsutbetalningar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7461,7461,Pensionsutbetalningar till f.d. kollektivanställda,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7462,7462,Pensionsutbetalningar till f.d. tjänstemän,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7463,7463,Pensionsutbetalningar till f.d. företagsledare,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7470,7470,Förvaltnings-- och kreditförsäkringsavgift,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7500,7500,Sociala och andra avgifter enlight lag och avtal (gruppkonto),expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7510,7510,Arbetsgivaravgifter 31.42%,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7515,7515,Arbetsgivaravgifter på skattepliktiga kostnadsersättningar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7516,7516,Arbetsgivaravgifter på arvoden,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7518,7518,Arbetsgivaravgifter på bruttolöneavdrag,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7531,7531,"Särskild löneskatt för vissa försäkringsersättningar m.m.",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7532,7532,"Särskild löneskatt pensionskostnader, deklarationspost",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7533,7533,Särskild löneskatt för pensionskostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7551,7551,Avkastningsskatt 15% försäkringsföretag m. lf. samt avsatt till pensioner,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7552,7552,Avkastningsskatt 15% utländska pensionsförsäkringar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7553,7553,Avkastningsskatt 30% utländska försäkringsföretag m. fl.,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7554,7554,Avkastningsskatt 30% utländska kapitalförsäkringar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7571,7571,Arbetsmarknadsförsäkringar,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7572,7572,"Arbetsmarknadsförsäkringar pensionsförsäkringspremier, deklarationspost",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7581,7581,Grupplivförsäkringspremier,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7582,7582,Gruppjukförsäkringspremier,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7583,7583,Gruppolycksfallsförsäkringspremier,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7589,7589,Övriga gruppförsäkringspremier,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7620,7620,Sjuk- och hälsovård,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7623,7623,"Sjukvårdsförsäkring, ej avdragsgill",expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7630,7630,Personalrepresentation,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7650,7650,Sjuklöneförsäkring,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7670,7670,Förändring av personalstiftelsekapital,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7671,7671,Avsättning till personalstiftelse,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7678,7678,Gottgörelse från personalstiftelse av personalstiftelsekapital,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7690,7690,Övriga personalkostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7691,7691,Personalrekrytering,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7692,7692,Begravningshjälp,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7693,7693,Fritidsverksamhet,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7699,7699,Övriga personalkostnader,expense,l10nse_chart_template_K2,l10n_se.account_tag_32,False
a7710,7710,Nedskrivningar av immateriella anläggningstillgångar,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7740,7740,Nedskrivningar av vissa omsättningstillgångar,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7760,7760,Återföring av nedskrivningar av immateriella anläggningstillgångar,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7770,7770,Återföring av nedskrivningar av byggnader och mark,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7780,7780,Återföring av nedskrivningar av maskiner och inventarier,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7790,7790,Återföring av nedskrivningar av vissa omsättningstillgångar,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7811,7811,Avskrivningar på balanserade utgifter,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7812,7812,Avskrivningar på koncessioner m.m.,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7813,7813,Avskrivningar på patent,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7814,7814,Avskrivningar på licenser,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7815,7815,Avskrivningar på varumärken,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7816,7816,Avskrivningar på hyresrätter,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7817,7817,Avskrivningar på goodwill,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7819,7819,Avskrivningar på övriga immateriella anläggningstillgångar,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7821,7821,Avskrivningar på byggnader,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7824,7824,Avskrivningar på markanläggningar,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7829,7829,Avskrivningar på övriga byggnader,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7831,7831,Avskrivningar på maskiner och andra tekniska anläggningar,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7832,7832,Avskrivningar på inventarier och verktyg,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7833,7833,Avskrivningar på installationer,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7834,7834,Avskrivningar på bilar och nadra transportmedel,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7835,7835,Avskrivningar på datorer,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7836,7836,Avskrivningar på leasade tillgångar,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7839,7839,Avskrivningar på övriga maskiner och inventarier,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7840,7840,Avskrivningar på förbättringsutgifter på annans fastighet,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7960,7960,Valutakursförluster på fordringar och skulder av rörelsekaraktär,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7971,7971,Förlust vid avyttring av immateriella anläggningstillgångar,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7972,7972,Förlust vid avyttring av byggnader och mark,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a7973,7973,Förlust vid avyttring av maskiner och inventarier,expense,l10nse_chart_template_K2,"l10n_se.account_tag_32",False
a8010,8010,Utdelning på andelar i koncernföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8012,8012,Utdelning på andelar i dotterföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8016,8016,"Emissionsinsats, koncernföretag",income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8020,8020,Resultat vid försäljning av andelar i koncernföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8022,8022,Resultat vid försäljning av andelar i dotterföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8030,8030,Resultat från handelsbolag (dotterföretag),income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8070,8070,Nedskrivningar av andelar i och långfristiga fordringar hos koncernföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8072,8072,Nedskrivningar av andelar i dotterföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8076,8076,Nedskrivningar av långfristiga fordringar hos moderföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8077,8077,Nedskrivningar av långfristiga fordringar hos dotterföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8080,8080,Återföringar av nedskrivningar av andelar i och långfristiga fordringar hos koncernföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8082,8082,Återföringar av nedskrivningar av andelar i dotterföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8086,8086,Återföringar av nedskrivningar av långfristiga fordringar hos moderföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8087,8087,Återföringar av nedskrivningar av långfristiga fordringar hos dotterföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8110,8110,"Utdelningar på andelar i intresseföretag, gemensamt styrda företag och Övriga företag som det finns ett ägarintresse i",income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8111,8111,Utdelningar på andelar i intresseföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8112,8112,Utdelningar på andelar i gemensamt styrda företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8113,8113,Utdelningar på andelar i övriga företag som det finns ett ägarintresse i,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8116,8116,"Emissionsinsats, intresseföretag",income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8117,8117,"Emissionsinsats, gemensamt styrda företag",income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8118,8118,"Emissionsinsats, övriga företag som det finns ett ägarintresse i",income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8120,8120,"Resultat vid försäljning av andelar i intresseFöretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i",income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8121,8121,Resultat vid försäljning av andelar i intresseföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8122,8122,Resultat vid försäljning av andelar i gemensamt styrda företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8123,8123,Resultat vid försäljning av andelar i övriga företag som det finns et ägarintresse i,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8130,8130,"Resultatandelar från handelsbolag (intresseföretag, gemensamt styrda företag pcj övriga företag som det finns ett ägarintresse i)",income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8131,8131,Resultatandelar från handelsbolag (intresseföretag),income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8132,8132,Resultatandelar från handelsbolag (gemensamt styrda företag),income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8133,8133,Resultatandelar från handelsbolag (övriga företag som det finns ett ägarintresse i),income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8170,8170,"Nedskrivningar av andelar i och långfristiga fordringar hos intresseföretag, gemensamt styrda företag och övriga företag som det finns ett ägarintresse i",income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8171,8171,Nedskrivningar av andelar i intresseföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8172,8172,Nedskrivningar av långfristiga fordringar hos intresseföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8173,8173,Nedskrivningar av andelar i gemensamt styrda företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8174,8174,Nedskrivningar av långfristiga fordringar hos gemensamt styrda företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8176,8176,Nedskrivningar av andelar i övriga företag som det finns ett ägarintresse i,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8177,8177,Nedskrivningar av långfristiga fordringar hos övriga företag som det finns et ägarintresse i,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8180,8180,Återföringar av nedskrivningar av andelar i och långfristiga fordringar hos intresseföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8181,8181,Återföringar av nedskrivningar av andelar i intresseföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8182,8182,Återföringar av nedskrivningar av långfristiga fordringar hos intresseföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8183,8183,Återföringar av nedskrivningar av andelar i gemensamt styrda företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8184,8184,Återföringar av nedskrivningar av långfristiga fordringar hos gemensamt styrda företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8186,8186,Återföringar av nedskrivningar av andelar i övriga företag som det finns et ägarintresse i,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8187,8187,Återföringar av nedskrivningar av långfristiga fordringar hos övriga företag som det finns ett ägarintresse i,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8212,8212,"Utdelningar, övriga företag",income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8216,8216,"Insatsemissioner, övriga företag",income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8221,8221,Resultat vid försäljning av andelar i andra företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8222,8222,Resultat vid försäljning av långfristiga fordringar hos andra företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8223,8223,Resultat vid försäljning av derivat (långfristiga värdepappersinnehav),income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8230,8230,Valutakursdifferenser på långfristiga fordringar,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8231,8231,Valutakursvinster på långfristiga fordringar,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8236,8236,Valutakursförluster på långfristiga fordringar,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8240,8240,Resultatandelar från handelsbolag (andra företag),income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8251,8251,Ränteintäkter från långfristiga fordringar hos koncernföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8252,8252,0.00Ränteintäkter från Övriga värdepapper,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8254,8254,"Skattefria ränteintäkter, långfristiga tillgångar",income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8255,8255,Avkastningsskatt kapitalplacering,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8260,8260,Ränteintäkter från långfristiga fordringar hos koncernföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8261,8261,Ränteintäkter från långfristiga fordringar hos moderföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8262,8262,Ränteintäkter från långfristiga fordringar hos dotterföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8263,8263,Ränteintäkter från långfristiga fordringar hos andra koncernföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8271,8271,Nedskrivningar av andelar i andra företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8272,8272,Nedskrivningar av långfristiga fordringar hos andra företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8273,8273,Nedskrivningar av övriga värdepapper hos andra företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8280,8280,Återföringar av andelar i och långfristiga fordringar hos andra företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8281,8281,Återföringar av nedskrivningar av av andelar i andra företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8282,8282,Återföringar av nedskrivningar av långfristiga fordringar hos andra företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8283,8283,Återföringar av nedskrivningar av övriga värdepapper i andra företag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8311,8311,Ränteintäkter från bank,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8312,8312,Ränteintäkter från kortfristiga placeringar,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8313,8313,Ränteintäkter från kortfristiga fordringar,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8317,8317,Ränteintäkter för dold räntekompensation,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8319,8319,Ränteintäkter från omsättningstillgångar,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8331,8331,Valutakursvinster på kortfristiga fordringar och placeringar,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8336,8336,Valutakursförluster på kortfristiga fordringar och placeringar,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8360,8360,Övriga ränteintäkter från koncernföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8361,8361,Övriga ränteintäkter från moderföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8362,8362,Övriga ränteintäkter från dotterföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8363,8363,Övriga ränteintäkter från andra koncernföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8370,8370,Nedskrivningar av kortfristiga placeringar,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8380,8380,Återföringar av nedskrivningar av kortfristiga placeringar,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8400,8400,Räntekostnader (gruppkonto),income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8410,8410,Räntekostnader för långfristiga skulder,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8411,8411,"Räntekostnader för obligations-, förlags- och konvertibla lån",income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8412,8412,Räntedel i årets pensionskostnad,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8413,8413,Räntekostnader för checkräkningskredit,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8415,8415,Räntekostnader för andra skulder till kreditinstitut,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8417,8417,Räntekostnader för dold räntekompensation m. m.,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8418,8418,Räntekostnader för räntesubventioner,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8419,8419,Räntekostnader för långfristiga skulder,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8421,8421,Räntekostnader till kreditinstitut,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8424,8424,Räntekostnader byggnadskreditiv,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8429,8429,Övriga räntekostnader för kortfristiga skulder,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8431,8431,Valutakursvinster på skulder,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8436,8436,Valutakursförluster på skulder,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8440,8440,Erhållna räntebidrag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8460,8460,Räntekostnader till koncernföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8461,8461,Räntekostnader till moderföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8462,8462,Räntekostnader till dotterföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8463,8463,Räntekostnader till andra koncernföretag,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8490,8490,Övriga skuldrelaterade poster,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8491,8491,Erhållet ackord på skulder till kreditinstitut,income_other,l10nse_chart_template_K2,l10n_se.account_tag_33,False
a8810,8810,Förändring av periodiseringsfond,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8820,8820,Mottagna koncernbidrag,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8830,8830,Lämnade koncernbidrag,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8840,8840,Lämnade gottgörelser,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8851,8851,"Förändring av överavskrivningar, immateriella anläggningstillgångar",expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8852,8852,"Förändring av överskrivningar, byggnader och markanläggningar",expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8853,8853,"Förändring av överskrivningar, maskiner oh inventarier",expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8860,8860,Förändring av ersättningsfond,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8861,8861,Avsättning till ersättningsfond för inventarier,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8862,8862,Avsättning till ersättningsfond för byggnader och markanläggningar,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8864,8864,Avsättning till ersättningsfond för djurlager i jordbruk och renskötsel,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8865,8865,Ianspråktagande av ersättningsfond för avskrivningar,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8866,8866,Ianspråktagande av ersättningsfond fr annat än avskrivningar,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8869,8869,Återföring fran ersättningsfond,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8890,8890,Övriga bokslutsdispositioner,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8892,8892,Nedskrivningar av konsolideringskaraktär av anläggningstillgångar,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8896,8896,Förändring av lagerreserv,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8899,8899,Övriga bokslutsdispositioner,expense,l10nse_chart_template_K2,l10n_se.account_tag_34,False
a8920,8920,Skatt på grund av ändrad beskattning,expense,l10nse_chart_template_K2,l10n_se.account_tag_35,False
a8930,8930,Restituerad skatt,expense,l10nse_chart_template_K2,l10n_se.account_tag_35,False
a8980,8980,Övriga skatter,expense,l10nse_chart_template_K2,l10n_se.account_tag_35,False

```

## File: data\account.account.template-K3.csv

```csv
id,code,name,account_type,chart_template_id/id,tag_ids/id,reconcile
a1010,1010,Utvecklingsutgifter,asset_non_current,l10nse_chart_template_K3,l10n_se.account_tag_1,False
a1011,1011,Balanserade utgifter för utveckling,asset_non_current,l10nse_chart_template_K3,l10n_se.account_tag_1,False
a1012,1012,Balanserade utgifter för programvaror,asset_non_current,l10nse_chart_template_K3,l10n_se.account_tag_1,False
a1018,1018,Ackumulerade nedskrivningar på balanserade utgifter,asset_non_current,l10nse_chart_template_K3,l10n_se.account_tag_1,False
a1019,1019,Ackumulerade avskrivningar på balanserade utgifter,asset_non_current,l10nse_chart_template_K3,l10n_se.account_tag_1,False
a1081,1081,Pågående projekt för immateriella anläggningstillgångar,asset_non_current,l10nse_chart_template_K3,l10n_se.account_tag_1,False
a1088,1088,Förskott för immateriella anläggningstillgångar,asset_non_current,l10nse_chart_template_K3,l10n_se.account_tag_1,False
a1260,1260,Leasade tillgångar,asset_non_current,l10nse_chart_template_K3,l10n_se.account_tag_2,False
a1269,1269,Ackumulerade avskrivningar på leasade tillgångar,asset_non_current,l10nse_chart_template_K3,l10n_se.account_tag_2,False
a1370,1370,Uppskjuten skattefordran,asset_non_current,l10nse_chart_template_K3,l10n_se.account_tag_3,False
a2092,2092,Mottagna/Lämnade koncernbidrag,equity,l10nse_chart_template_K3,"l10n_se.account_tag_40",False
a2096,2096,Fond för verkligt värde,equity,l10nse_chart_template_K3,"l10n_se.account_tag_11",False
a2240,2240,Avsättningar för uppskjutna skatter,liability_non_current,l10nse_chart_template_K3,l10n_se.account_tag_19,False
a3940,3940,Orealiserade negativa/positiva värdeförändringar på säkringsinstrument,income_other,l10nse_chart_template_K3,l10n_se.account_tag_31,False
a7940,7940,Orealiserade positiva/negativa värdeförändringar på säkringsinstrument,expense,l10nse_chart_template_K3,"l10n_se.account_tag_32",False
a8290,8290,"Värdering till verkligt värde, anläggningstillgångar",income_other,l10nse_chart_template_K3,l10n_se.account_tag_33,False
a8291,8291,Orealiserade värdeförändringar på anläggningstillgångar,income_other,l10nse_chart_template_K3,l10n_se.account_tag_33,False
a8295,8295,Orealiserade värdeförändringar på derivatinstrument,income_other,l10nse_chart_template_K3,l10n_se.account_tag_33,False
a8320,8320,"Värdering till verkligt värde, omsättningstillgångar",income_other,l10nse_chart_template_K3,l10n_se.account_tag_33,False
a8321,8321,Orealiserade värdeförändringar på omsättningstillgångar,income_other,l10nse_chart_template_K3,l10n_se.account_tag_33,False
a8325,8325,Ränteintäkter från omsättningstillgångar,income_other,l10nse_chart_template_K3,l10n_se.account_tag_33,False
a8450,8450,Orealiserade värdeförändringar på skulder,income_other,l10nse_chart_template_K3,l10n_se.account_tag_33,False
a8451,8451,Orealiserade värdeförändringar på skulder,income_other,l10nse_chart_template_K3,l10n_se.account_tag_33,False
a8455,8455,Orealiserade värdeförändringar på säkringsinstrument,income_other,l10nse_chart_template_K3,l10n_se.account_tag_33,False
a8480,8480,Aktiverade ränteutgifter,income_other,l10nse_chart_template_K3,l10n_se.account_tag_33,False
a8940,8940,Uppskjuten skatt,expense,l10nse_chart_template_K3,l10n_se.account_tag_35,False

```

## File: data\account.account.template.csv

```csv
"id","code","name","account_type","chart_template_id/id",tag_ids/id,"reconcile"
a1030,1030,"Patent",asset_non_current,l10nse_chart_template,l10n_se.account_tag_1,False
a1039,1039,Ackumulerade avskrivningar på patent,asset_non_current,l10nse_chart_template,l10n_se.account_tag_1,False
a1060,1060,"Hyresrätter, tomträtter och liknande",asset_non_current,l10nse_chart_template,l10n_se.account_tag_1,False
a1069,1069,"Ackumulerade avskrivningar på hyresrätter, tomträtter och liknande",asset_non_current,l10nse_chart_template,l10n_se.account_tag_1,False
a1110,1110,Byggnader,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1119,1119,Ackumulerade avskrivningar på byggnader,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1130,1130,Mark,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1150,1150,Markanläggningar,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1159,1159,Ackumulerade avskrivningar på markanläggningar,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1210,1210,Maskiner och andra tekniska anläggningar,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1219,1219,Ackumulerade avskrivningar på maskiner och andra tekniska anläggningar,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1220,1220,Inventarier och verktyg,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1229,1229,Ackumulerade avskrivningar på inventarier och verktyg,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1240,1240,Bilar och andra transportmedel,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1249,1249,Ackumulerade avskrivningar på bilar och andra transportmedel,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1250,1250,Datorer,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1259,1259,Ackumulerade avskrivningar på datorer,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1290,1290,Övriga materiella anläggningstillgångar,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1291,1291,Konst och liknande tillgångar,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1299,1299,Ackumulerade avskrivningar på övriga materiella anläggningstillgångar,asset_fixed,l10nse_chart_template,l10n_se.account_tag_2,False
a1350,1350,Andelar och värdepapper i andra företag,asset_non_current,l10nse_chart_template,l10n_se.account_tag_3,False
a1380,1380,Andra långfristiga fordringar,asset_non_current,l10nse_chart_template,l10n_se.account_tag_3,False
a1410,1410,Lager av råvaror,asset_current,l10nse_chart_template,l10n_se.account_tag_4,False
a1419,1419,Förändring av lager av råvaror,asset_current,l10nse_chart_template,l10n_se.account_tag_4,False
a1440,1440,Produkter i arbete,asset_current,l10nse_chart_template,l10n_se.account_tag_4,False
a1449,1449,Förändring av produkter i arbete,asset_current,l10nse_chart_template,l10n_se.account_tag_4,False
a1450,1450,Lager av färdiga varor,asset_current,l10nse_chart_template,l10n_se.account_tag_4,False
a1459,1459,Förändring av lager av färdiga varor,asset_current,l10nse_chart_template,l10n_se.account_tag_4,False
a1460,1460,Lager av handelsvaror,asset_current,l10nse_chart_template,l10n_se.account_tag_4,False
a1469,1469,Förändring av lager av handelsvaror,asset_current,l10nse_chart_template,l10n_se.account_tag_4,False
a1470,1470,Pågående arbeten,asset_current,l10nse_chart_template,l10n_se.account_tag_4,False
a1479,1479,Förändring av Pågående arbete,asset_current,l10nse_chart_template,l10n_se.account_tag_4,False
a1480,1480,Förskott för varor och tjänster,asset_prepayments,l10nse_chart_template,l10n_se.account_tag_4,False
a1490,1490,Övriga lagertillgångar,asset_prepayments,l10nse_chart_template,l10n_se.account_tag_4,False
a1510,1510,Kundfordringar,asset_receivable,l10nse_chart_template,l10n_se.account_tag_5,True
a1513,1513,Kundfordringar - delad faktura,asset_receivable,l10nse_chart_template,l10n_se.account_tag_5,True
a1519,1519,Nedskrivning av kundfordringar,asset_receivable,l10nse_chart_template,l10n_se.account_tag_5,True
a1580,1580,Fordringar för kontokort och kuponger,asset_receivable,l10nse_chart_template,l10n_se.account_tag_5,True
a1610,1610,Kortfristiga fordringar hos anställda,asset_receivable,l10nse_chart_template,l10n_se.account_tag_5,True
a1630,1630,Avräkning för skatter och avgifter (skattekonto),asset_current,l10nse_chart_template,l10n_se.account_tag_5,False
a1640,1640,Skattefordringar,asset_current,l10nse_chart_template,l10n_se.account_tag_5,False
a1650,1650,Momsfordran,asset_current,l10nse_chart_template,l10n_se.account_tag_5,False
a1680,1680,Andra kortfristiga fordringar,asset_current,l10nse_chart_template,l10n_se.account_tag_5,False
a1710,1710,Förutbetalda hyreskostnader,asset_prepayments,l10nse_chart_template,l10n_se.account_tag_5,False
a1720,1720,"Förutbetalda leasingavgifter, kortfristig del",asset_prepayments,l10nse_chart_template,l10n_se.account_tag_5,False
a1730,1730,Förutbetalda försäkringspremier,asset_prepayments,l10nse_chart_template,l10n_se.account_tag_5,False
a1740,1740,Förutbetalda räntekostnader,asset_prepayments,l10nse_chart_template,l10n_se.account_tag_5,False
a1750,1750,Upplupna hyresintäkter,asset_current,l10nse_chart_template,l10n_se.account_tag_5,False
a1760,1760,Upplupna ränteintäkter,asset_current,l10nse_chart_template,l10n_se.account_tag_5,False
a1790,1790,Övriga förutbetalda kostnader och upplupna intäkter,asset_current,l10nse_chart_template,l10n_se.account_tag_5,False
a1810,1810,Andel i börsnoterade företag,asset_current,l10nse_chart_template,l10n_se.account_tag_6,False
a1880,1880,Andra kortfristiga placeringar,asset_current,l10nse_chart_template,l10n_se.account_tag_6,False
a1890,1890,Nedskrivning av kortfristiga placeringar,asset_current,l10nse_chart_template,l10n_se.account_tag_6,False
a1910,1910,Kassa,asset_cash,l10nse_chart_template,l10n_se.account_tag_7,False
a1920,1920,PlusGiro,asset_cash,l10nse_chart_template,l10n_se.account_tag_7,False
a1930,1930,Företagskonto/checkkonto/affärskonto,asset_cash,l10nse_chart_template,l10n_se.account_tag_7,False
a1940,1940,Övriga bankkonton,asset_cash,l10nse_chart_template,l10n_se.account_tag_7,False
a2010,2010,"Eget kapital, delägare 1",equity,l10nse_chart_template,l10n_se.account_tag_13,False
a2011,2011,Egna varuuttag,equity,l10nse_chart_template,l10n_se.account_tag_14,True
a2013,2013,Övriga egna uttag,equity,l10nse_chart_template,l10n_se.account_tag_14,True
a2017,2017,Årets kapitaltillskott,equity,l10nse_chart_template,l10n_se.account_tag_14,True
a2018,2018,Övriga egna insättningar,equity,l10nse_chart_template,l10n_se.account_tag_14,True
a2019,2019,"Årets resultat, delägare",equity,l10nse_chart_template,l10n_se.account_tag_17,False
a2020,2020,"Eget kapital, delägare 2",equity,l10nse_chart_template,l10n_se.account_tag_13,False
a2030,2030,"Eget kapital, delägare 3",equity,l10nse_chart_template,l10n_se.account_tag_13,False
a2040,2040,"Eget kapital, delägare 4",equity,l10nse_chart_template,l10n_se.account_tag_13,False
a2060,2060,"Eget kapital i ideella föreningar, stiftelser och registrerade trossamfund",equity,l10nse_chart_template,"l10n_se.account_tag_13",False
a2070,2070,Ändamålsbestämda medel,equity,l10nse_chart_template,l10n_se.account_tag_37,False
a2081,2081,Aktiekapital,equity,l10nse_chart_template,l10n_se.account_tag_8,False
a2083,2083,Medlemsinsatser,equity,l10nse_chart_template,l10n_se.account_tag_40,False
a2086,2086,Reservfond,equity,l10nse_chart_template,l10n_se.account_tag_38,False
a2090,2090,Fritt eget kapital,equity,l10nse_chart_template,l10n_se.account_tag_39,False
a2091,2091,Balanserad vinst eller förlust,equity,l10nse_chart_template,l10n_se.account_tag_12,False
a2098,2098,Vinst eller förlust från föregående år,equity,l10nse_chart_template,l10n_se.account_tag_11,False
a2099,2099,Årets resultat,equity,l10nse_chart_template,l10n_se.account_tag_17,False
a2120,2120,Periodiseringsfond 2020,equity,l10nse_chart_template,l10n_se.account_tag_18,False
a2121,2121,Periodiseringsfond 2021,equity,l10nse_chart_template,l10n_se.account_tag_18,False
a2122,2122,Periodiseringsfond 2022,equity,l10nse_chart_template,l10n_se.account_tag_18,False
a2123,2123,Periodiseringsfond 2023,equity,l10nse_chart_template,l10n_se.account_tag_18,False
a2124,2124,Periodiseringsfond 2024,equity,l10nse_chart_template,l10n_se.account_tag_18,False
a2125,2125,Periodiseringsfond 2015,equity,l10nse_chart_template,l10n_se.account_tag_18,False
a2126,2126,Periodiseringsfond 2016,equity,l10nse_chart_template,l10n_se.account_tag_18,False
a2127,2127,Periodiseringsfond 2017,equity,l10nse_chart_template,l10n_se.account_tag_18,False
a2128,2128,Periodiseringsfond 2018,equity,l10nse_chart_template,l10n_se.account_tag_18,False
a2129,2129,Periodiseringsfond 2019,equity,l10nse_chart_template,l10n_se.account_tag_18,False
a2150,2150,Ackumulerade överavskrivningar,liability_non_current,l10nse_chart_template,l10n_se.account_tag_18,False
a2210,2210,Avsättningar för pensioner enligt tryggandelagen,liability_non_current,l10nse_chart_template,l10n_se.account_tag_19,False
a2220,2220,Avsättningar för garantier,liability_non_current,l10nse_chart_template,l10n_se.account_tag_19,False
a2290,2290,Övriga avsättningar,liability_non_current,l10nse_chart_template,l10n_se.account_tag_19,False
a2330,2330,Checkräkningskredit,liability_non_current,l10nse_chart_template,l10n_se.account_tag_21,False
a2350,2350,Andra långfristiga skulder till kreditinstitut,liability_non_current,l10nse_chart_template,l10n_se.account_tag_21,False
a2390,2390,Övriga långfristiga skulder,liability_non_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2393,2393,"Lån från närstående personer, långfristiga del",liability_non_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2410,2410,Andra kortfristiga låneskulder till kreditinstitut,liability_current,l10nse_chart_template,l10n_se.account_tag_21,False
a2420,2420,Förskott från kunder,liability_current,l10nse_chart_template,l10n_se.account_tag_22,False
a2440,2440,Leverantörsskulder,liability_payable,l10nse_chart_template,l10n_se.account_tag_23,True
a2480,2480,"Checkräkningskredit, kortfristig",liability_payable,l10nse_chart_template,l10n_se.account_tag_21,True
a2490,2490,"Övriga kortfristiga skulder till kreditinstitut, kunder och leverantörer",liability_current,l10nse_chart_template,l10n_se.account_tag_21,False
a2510,2510,Skatteskulder,liability_current,l10nse_chart_template,l10n_se.account_tag_28,False
a2610,2610,"Utgående moms, 25 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2611,2611,"Utgående moms på försäljning inom Sverige, 25 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2612,2612,"Utgående moms på egna uttag, 25 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2613,2613,"Utgående moms för uthyrning, 25 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2614,2614,"Utgående moms omvänd skattskyldighet, 25 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2615,2615,"Utgående moms import av varor, 25 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2616,2616,Utgående moms VMB 25 %,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2620,2620,"Utgående moms, 12 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2621,2621,"Utgående moms på försäljning inom Sverige, 12 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2622,2622,"Utgående moms på egna uttag, 12 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2623,2623,"Utgående moms för uthyrning, 12 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2624,2624,"Utgående moms omvänd skattskyldighet, 12 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2625,2625,"Utgående moms import av varor, 12 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2626,2626,"Utgående moms VMB, 12 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2630,2630,"Utgående moms, 6 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2631,2631,"Utgående moms på försäljning inom Sverige, 6 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2632,2632,"Utgående moms på egna uttag, 6 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2633,2633,"Utgående moms för uthyrning, 6 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2634,2634,"Utgående moms omvänd skattskyldighet, 6 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2635,2635,"Utgående moms import av varor, 6 %",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2636,2636,Utgående moms VMB 6 %,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2640,2640,Ingående moms,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2641,2641,Debiterad ingående moms,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2642,2642,Debiterad ingående moms i anslutning till frivillig skattskyldighet,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2645,2645,Beräknad ingående moms på förvärv från utlandet,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2646,2646,Ingående moms på uthyrning,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2647,2647,Ingående moms omvänd skattskyldighet varor och tjänster i Sverige,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2648,2648,Vilande ingående moms,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2649,2649,"Ingående moms, blandad verksamhet",liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2650,2650,Redovisningskonto för moms,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2710,2710,Personalskatt,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2730,2730,Lagstadgade sociala avgifter och särskild löneskatt,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2740,2740,Avtalade sociala avgifter,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2790,2790,Övriga löneavdrag,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2820,2820,Kortfristiga skulder till anställda,liability_payable,l10nse_chart_template,l10n_se.account_tag_29,True
a2840,2840,Kortfristiga låneskulder,liability_payable,l10nse_chart_template,l10n_se.account_tag_29,True
a2890,2890,Övriga kortfristiga skulder,liability_current,l10nse_chart_template,l10n_se.account_tag_29,False
a2910,2910,Upplupna löner,liability_current,l10nse_chart_template,l10n_se.account_tag_30,False
a2920,2920,Upplupna semesterlöner,liability_current,l10nse_chart_template,l10n_se.account_tag_30,False
a2940,2940,Upplupna lagstadgade sociala och andra avgifter,liability_current,l10nse_chart_template,l10n_se.account_tag_30,False
a2950,2950,Upplupna avtalade sociala avgifter,liability_current,l10nse_chart_template,l10n_se.account_tag_30,False
a2960,2960,Upplupna räntekostnader,liability_current,l10nse_chart_template,l10n_se.account_tag_30,False
a2970,2970,Förutbetalda intäkter,liability_current,l10nse_chart_template,l10n_se.account_tag_30,False
a2990,2990,Övriga upplupna kostnader och förutbetalda intäkter,liability_current,l10nse_chart_template,l10n_se.account_tag_30,False
a2999,2999,OBS-konto,liability_current,l10nse_chart_template,l10n_se.account_tag_30,False
a3000,3000,Försäljning inom Sverige,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3001,3001,"Försäljning inom Sverige, 25 % moms",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3002,3002,"Försäljning inom Sverige, 12 % moms",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3003,3003,"Försäljning inom Sverige, 6 % moms",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3004,3004,"Försäljning inom Sverige, momsfri",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3100,3100,Försäljning av varor utanför EU,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3105,3105,Försäljning varor till land utanför EU,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3106,3106,"Försäljning varor till annat EU-land, momspliktig",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3108,3108,"Försäljning varor till annat EU-land, momsfri",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3200,3200,Försäljning VMB och omvänd moms,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3211,3211,Försäljning positiv VMB 25 %,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3212,3212,Försäljning negativ VMB 25 %,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3231,3231,"Försäljning inom byggsektorn, omvänd skatteskyldighet moms",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3300,3300,Försäljning av tjänster utanför Sverige,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3305,3305,Försäljning av tjänster till land utanför EU,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3308,3308,Försäljning av tjänster till annat EU-land,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3400,3400,"Försäljning, egna uttag",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3401,3401,"Egna uttag momspliktiga, 25 %",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3402,3402,"Egna uttag momspliktiga, 12 %",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3403,3403,"Egna uttag momspliktiga, 6 %",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3404,3404,"Egna uttag, momsfria",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3500,3500,Fakturerade kostnader (gruppkonto),income,l10nse_chart_template,l10n_se.account_tag_31,False
a3510,3510,Fakturerat emballage,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3520,3520,Fakturerade frakter,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3521,3521,"Fakturerade frakter, EU-land",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3522,3522,"Fakturerade frakter, export",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3530,3530,Fakturerad tull- och speditionskostnader m.m.,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3540,3540,Faktureringsavgifter,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3541,3541,"Faktureringsavgifter, EU-land",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3542,3542,"Faktureringsavgifter, export",income,l10nse_chart_template,l10n_se.account_tag_31,False
a3600,3600,Rörelsens sidointäkter (gruppkonto),income,l10nse_chart_template,l10n_se.account_tag_31,False
a3730,3730,Lämnade rabatter,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3740,3740,Öres- och kronutjämning,income,l10nse_chart_template,l10n_se.account_tag_31,False
a3800,3800,Aktiverat arbete för egen räkning (gruppkonto),income,l10nse_chart_template,l10n_se.account_tag_31,False
a3900,3900,Övriga rörelseintäkter (gruppkonto),income_other,l10nse_chart_template,l10n_se.account_tag_31,False
a3913,3913,Frivilligt momspliktiga hyresintäkter,income_other,l10nse_chart_template,l10n_se.account_tag_31,False
a3960,3960,Valutakursvinster på fordringar och skulder av rörelsekaraktär,income_other,l10nse_chart_template,l10n_se.account_tag_31,False
a3970,3970,Vinst vid avyttring av immateriella och materiella anläggningstillgångar,income_other,l10nse_chart_template,l10n_se.account_tag_31,False
a3980,3980,Erhållna offentliga stöd m.m.,income_other,l10nse_chart_template,l10n_se.account_tag_31,False
a4000,4000,Inköp av varor från Sverige,expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4200,4200,Sålda varor VMB,expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4211,4211,Sålda varor positiv VMB 25 %,expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4212,4212,Sålda varor negativ VMB 25 %,expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4400,4400,Momspliktiga inköp i Sverige,expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4415,4415,"Inköpta varor i Sverige, omvänd skattskyldighet, 25 %",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4426,4426,"Inköp tjänster i Sverige, omvänd skattskyldighet, 12 %",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4427,4427,"Inköp tjänster i Sverige, omvänd skattskyldighet, 6 %",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4500,4500,Övriga momspliktiga inköp,expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4515,4515,"Inköp av varor från annat EU-land, 25 %",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4516,4516,"Inköp av varor från annat EU-land, 12 %",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4517,4517,"Inköp av varor från annat EU-land, 6 %",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4518,4518,"Inköp av varor från annat EU-land, momsfri",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4531,4531,"Inköp av tjänster från ett land utanför EU, 25 %",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4532,4532,"Inköp av tjänster från ett land utanför EU, 12 %",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4533,4533,"Inköp av tjänster från ett land utanför EU, 6 %",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4535,4535,"Inköp av tjänster från annat EU-land, 25 %",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4536,4536,"Inköp av tjänster från annat EU-land, 12 %",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4537,4537,"Inköp av tjänster från annat EU-land, 6 %",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4538,4538,"Inköp av tjänster från annat EU-land, momsfri",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4545,4545,"Import av varor, 25 % moms",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4546,4546,"Import av varor, 12 % moms",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4547,4547,"Import av varor, 6 % moms",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4600,4600,Legoarbeten och underentreprenader (gruppkonto),expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4700,4700,Reduktion av inköpspriser (gruppkonto),expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4900,4900,Förändring av lager (gruppkonto),expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_31,False
a4910,4910,Förändring av lager av råvaror,expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4920,4920,Förändring av lager av tillsatsmaterial och förnödenheter,expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_32,False
a4940,4940,Förändring produkter i arbete,expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_31,False
a4950,4950,Förändring av lager av färdiga varor,expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_31,False
a4960,4960,Förändring av lager av handelsvaror,expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_31,False
a4970,4970,"Förändring pågående arbete, nedlagda kostnader",expense_direct_cost,l10nse_chart_template,l10n_se.account_tag_31,False
a5010,5010,Lokalhyra,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5020,5020,El för belysning,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5030,5030,Värme,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5040,5040,Vatten och avlopp,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5060,5060,Städning och renhållning,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5070,5070,Reparation och underhåll av lokaler,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5120,5120,El för belysning,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5130,5130,Värme,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5140,5140,Vatten och avlopp,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5160,5160,Städning och renhållning,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5170,5170,Reparation och underhåll av fastighet,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5200,5200,Hyra av anläggningstillgångar (gruppkonto),expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5300,5300,Energikostnader (gruppkonto),expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5410,5410,Förbrukningsinventarier,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5420,5420,Programvaror,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5460,5460,Förbrukningsmaterial,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5500,5500,Reparation och underhåll (gruppkonto),expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5600,5600,Kostnader för transportmedel (gruppkonto),expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5611,5611,Drivmedel för personbilar,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5612,5612,Försäkring och skatt för personbilar,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5613,5613,Reparation och underhåll av personbilar,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5615,5615,Leasing av personbilar,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5700,5700,Frakter och transporter (gruppkonto),expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5800,5800,Resekostnader (gruppkonto),expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5810,5810,Biljetter,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5820,5820,Hyrbilskostnader,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5831,5831,Kost och logi i Sverige,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5832,5832,Kost och logi i utlandet,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a5900,5900,"Reklam och PR","expense",l10nse_chart_template,l10n_se.account_tag_32,"False"
a6071,6071,"Representation, avdragsgill",expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6072,6072,"Representation, ej avdragsgill",expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6090,6090,Övriga försäljningskostnader,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6100,6100,Kontorsmateriel och trycksaker (gruppkonto),expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6210,6210,Telekommunikation,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6250,6250,Postbefordran,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6310,6310,Företagsförsäkringar,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6350,6350,Förluster på kundfordringar,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6390,6390,Övriga riskkostnader,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6410,6410,Styrelsearvoden som inte är lön,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6420,6420,Ersättningar till revisor,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6530,6530,Redovisningstjänster,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6540,6540,IT-tjänster,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6550,6550,Konsultarvoden,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6560,6560,Serviceavgifter till branschorganisationer,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6570,6570,Bankkostnader,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6580,6580,Advokat- och rättegångskostnader,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6590,6590,Övriga externa tjänster,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6800,6800,Inhyrd personal (gruppkonto),expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6970,6970,"Tidningar, tidskrifter och facklitteratur",expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6980,6980,Föreningsavgifter,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6991,6991,"Övriga externa kostnader, avdragsgilla",expense,l10nse_chart_template,l10n_se.account_tag_32,False
a6992,6992,"Övriga externa kostnader, ej avdragsgilla",expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7010,7010,Löner till kollektivanställda,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7090,7090,Förändring av semesterlöneskuld,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7210,7210,Löner till tjänstemän,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7220,7220,Löner till företagsledare,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7240,7240,Styrelsearvoden,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7290,7290,Förändring av semesterlöneskuld,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7310,7310,Kontanta extraersättningar,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7321,7321,"Skattefria traktamenten, Sverige",expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7322,7322,"Skattepliktiga traktamenten, Sverige",expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7323,7323,"Skattefria traktamenten, utlandet",expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7324,7324,"Skattepliktiga traktamenten, utlandet",expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7331,7331,Skattefria bilersättningar,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7332,7332,Skattepliktiga bilersättningar,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7380,7380,Kostnader förmåner till anställda,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7385,7385,Kostnader för fri bil,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7390,7390,Övriga kostnadsersättningar och förmåner,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7410,7410,Pensionsförsäkringspremier,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7490,7490,Övriga pensionskostnader,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7511,7511,Arbetsgivaravgift för löner och ersättningar,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7512,7512,Arbetsgivaravgifter för förmånsvärden,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7519,7519,Arbetsgivaravgifter för semester- och löneskulder,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7530,7530,Särskild Löneskatt,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7550,7550,Avkastningsskatt på pensionsmedel,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7570,7570,Premier för arbetsmarknadsförsäkringar,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7580,7580,Gruppförsäkringspremier,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7590,7590,Övriga sociala och andra avgifter enligt lag och avtal,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7600,7600,Övriga personalkostnader (gruppkonto),expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7610,7610,Utbildning,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7621,7621,"Sjuk- och hälsovård, avdragsgill",expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7622,7622,"Sjuk- och hälsovård, ej avdragsgill",expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7631,7631,"Personalrepresentation, avdragsgill",expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7632,7632,"Personalrepresentation, ej avdragsgill",expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7720,7720,Nedskrivningar av byggnader och mark,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7730,7730,Nedskrivningar av maskiner och inventarier,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7810,7810,Avskrivningar av immateriella anläggningstillgångar,expense_depreciation,l10nse_chart_template,l10n_se.account_tag_32,False
a7820,7820,Avskrivningar på byggnader och markanläggningar,expense_depreciation,l10nse_chart_template,l10n_se.account_tag_32,False
a7830,7830,Avskrivningar maskiner och inventarier,expense_depreciation,l10nse_chart_template,l10n_se.account_tag_32,False
a7970,7970,Förlust vid avyttring av immateriella och materiella anläggningstillgångar,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a7990,7990,Övriga rörelsekostnader,expense,l10nse_chart_template,l10n_se.account_tag_32,False
a8210,8210,Utdelningar på andelar i andra företag,income_other,l10nse_chart_template,l10n_se.account_tag_33,False
a8220,8220,Resultat vid försäljning av värdepapper i och långfristiga fordringar hos andra företag,income_other,l10nse_chart_template,l10n_se.account_tag_33,False
a8250,8250,Ränteintäkter från långfristiga fordringar hos och värdepapper i andra företag,income_other,l10nse_chart_template,l10n_se.account_tag_33,False
a8270,8270,Nedskrivningar av innehav av andelar i och långfristiga fordringar hos andra företag,expense,l10nse_chart_template,l10n_se.account_tag_33,False
a8310,8310,Ränteintäkter från omsättningstillgångar,income_other,l10nse_chart_template,l10n_se.account_tag_33,False
a8314,8314,Skattefria ränteintäkter,income_other,l10nse_chart_template,l10n_se.account_tag_33,False
a8330,8330,Valutakursdifferenser på kortfristiga fordringar och placeringar,income_other,l10nse_chart_template,l10n_se.account_tag_33,False
a8340,8340,Utdelningar på kortfristiga placeringar,income_other,l10nse_chart_template,l10n_se.account_tag_33,False
a8350,8350,Resultat vid försäljning av kortfristiga placeringar,income_other,l10nse_chart_template,l10n_se.account_tag_33,False
a8390,8390,Övriga finansiella intäkter,income_other,l10nse_chart_template,l10n_se.account_tag_33,False
a8410,8410,Räntekostnader för långfristiga skulder,expense,l10nse_chart_template,l10n_se.account_tag_33,False
a8420,8420,Räntekostnader för kortfristiga skulder,expense,l10nse_chart_template,l10n_se.account_tag_33,False
a8422,8422,Dröjsmålsräntor för leverantörsskulder,expense,l10nse_chart_template,l10n_se.account_tag_33,False
a8423,8423,Räntekostnader för skatter och avgifter,expense,l10nse_chart_template,l10n_se.account_tag_33,False
a8430,8430,Valutakursdifferenser på skulder,expense,l10nse_chart_template,l10n_se.account_tag_33,False
a8811,8811,Avsättning till periodiseringsfond,expense,l10nse_chart_template,l10n_se.account_tag_34,False
a8819,8819,Återföring från periodiseringsfond,expense,l10nse_chart_template,l10n_se.account_tag_34,False
a8850,8850,Förändring av överavskrivningar,expense,l10nse_chart_template,l10n_se.account_tag_34,False
a8910,8910,Skatt som belastar årets resultat,expense,l10nse_chart_template,l10n_se.account_tag_35,False
a8990,8990,Resultat,expense,l10nse_chart_template,l10n_se.account_tag_36,False
a8999,8999,Årets resultat,expense,l10nse_chart_template,l10n_se.account_tag_36,False
a9993,9993,Cash Discount Loss,expense,l10nse_chart_template,l10n_se.account_tag_36,False
a9994,9994,Cash Discount Gain,income_other,l10nse_chart_template,l10n_se.account_tag_33,False

```

## File: data\account_chart_template_after_accounts.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="l10nse_chart_template" model="account.chart.template">
            <field name="property_account_receivable_id" ref="a1510"/>
            <field name="property_account_payable_id" ref="a2440"/>
            <field name="property_account_expense_categ_id" ref="a4000"/>
            <field name="property_account_income_categ_id" ref="a3001"/>
            <field name="income_currency_exchange_account_id" ref="a3960"/>
            <field name="expense_currency_exchange_account_id" ref="a3960"/>
            <field name="property_stock_account_input_categ_id" ref="a4960"/>
            <field name="property_stock_account_output_categ_id" ref="a4960"/>
            <field name="property_stock_valuation_account_id" ref="a1410"/>
            <field name="default_pos_receivable_account_id" ref="a1910"/>
            <field name="account_journal_early_pay_discount_loss_account_id" ref="a9993"/>
            <field name="account_journal_early_pay_discount_gain_account_id" ref="a9994"/>
            <field name="property_tax_payable_account_id" ref="a2650"/>
            <field name="property_tax_receivable_account_id" ref="a1650"/>
        </record>
    </data>
</odoo>

```

## File: data\account_chart_template_before_accounts.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="l10nse_chart_template" model="account.chart.template">
            <field name="name">Swedish BAS Chart of Account Minimalist</field>
            <field name="currency_id" ref="base.SEK"/>
            <field name="bank_account_code_prefix">193</field>
            <field name="cash_account_code_prefix">191</field>
            <field name="transfer_account_code_prefix">194</field>
            <field name="code_digits">4</field>
            <field name="country_id" ref="base.se"/>
        </record>

        <record id="l10nse_chart_template_K2" model="account.chart.template">
            <field name="name">Swedish BAS Chart of Account complete K2</field>
            <field name="parent_id" ref="l10nse_chart_template"/>
            <field name="currency_id" ref="base.SEK"/>
            <field name="bank_account_code_prefix">193</field>
            <field name="cash_account_code_prefix">191</field>
            <field name="transfer_account_code_prefix">194</field>
            <field name="code_digits">4</field>
            <field name="country_id" ref="base.se"/>
        </record>

        <record id="l10nse_chart_template_K3" model="account.chart.template">
            <field name="name">Swedish BAS Chart of Account complete K3</field>
            <field name="parent_id" ref="l10nse_chart_template_K2"/>
            <field name="currency_id" ref="base.SEK"/>
            <field name="bank_account_code_prefix">193</field>
            <field name="cash_account_code_prefix">191</field>
            <field name="transfer_account_code_prefix">194</field>
            <field name="code_digits">4</field>
            <field name="country_id" ref="base.se"/>
        </record>
    </data>
</odoo>

```

## File: data\account_chart_template_configuration.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_se.l10nse_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_fiscal_position_account_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="fps_euro_25_goods_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_euro_b2b"/>
            <field name="account_src_id" ref="a3001"/>
            <field name="account_dest_id" ref="a3106"/>
        </record>
        <record id="fps_euro_12_goods_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_euro_b2b"/>
            <field name="account_src_id" ref="a3002"/>
            <field name="account_dest_id" ref="a3106"/>
        </record>
        <record id="fps_euro_6_goods_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_euro_b2b"/>
            <field name="account_src_id" ref="a3003"/>
            <field name="account_dest_id" ref="a3106"/>
        </record>
        <record id="fps_euro_0_goods_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_euro_b2b"/>
            <field name="account_src_id" ref="a3004"/>
            <field name="account_dest_id" ref="a3106"/>
        </record>
        <record id="fps_euro_25_service_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_euro_b2b"/>
            <field name="account_src_id" ref="a3001"/>
            <field name="account_dest_id" ref="a3308"/>
        </record>
        <record id="fps_euro_12_service_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_euro_b2b"/>
            <field name="account_src_id" ref="a3002"/>
            <field name="account_dest_id" ref="a3308"/>
        </record>
        <record id="fps_euro_6_service_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_euro_b2b"/>
            <field name="account_src_id" ref="a3003"/>
            <field name="account_dest_id" ref="a3308"/>
        </record>
        <record id="fps_euro_0_service_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_euro_b2b"/>
            <field name="account_src_id" ref="a3004"/>
            <field name="account_dest_id" ref="a3308"/>
        </record>        
        <record id="fps_outside_euro_25_goods_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_outside_euro"/>
            <field name="account_src_id" ref="a3001"/>
            <field name="account_dest_id" ref="a3105"/>
        </record>
        <record id="fps_outside_euro_12_goods_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_outside_euro"/>
            <field name="account_src_id" ref="a3002"/>
            <field name="account_dest_id" ref="a3105"/>
        </record>
        <record id="fps_outside_euro_6_goods_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_outside_euro"/>
            <field name="account_src_id" ref="a3003"/>
            <field name="account_dest_id" ref="a3105"/>
        </record>
        <record id="fps_outside_euro_0_goods_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_outside_euro"/>
            <field name="account_src_id" ref="a3004"/>
            <field name="account_dest_id" ref="a3105"/>
        </record>
        <record id="fps_outside_euro_25_service_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_outside_euro"/>
            <field name="account_src_id" ref="a3001"/>
            <field name="account_dest_id" ref="a3305"/>
        </record>
        <record id="fps_outside_euro_12_service_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_outside_euro"/>
            <field name="account_src_id" ref="a3002"/>
            <field name="account_dest_id" ref="a3305"/>
        </record>
        <record id="fps_outside_euro_6_service_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_outside_euro"/>
            <field name="account_src_id" ref="a3003"/>
            <field name="account_dest_id" ref="a3305"/>
        </record>
        <record id="fps_outside_euro_0_service_acc" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fp_outside_euro"/>
            <field name="account_src_id" ref="a3004"/>
            <field name="account_dest_id" ref="a3305"/>
        </record>  
    </data>
</odoo>

```

## File: data\account_fiscal_position_tax_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <!-- Fiscal Position Purchase Eurozone -->
        <record id="fpp_euro_25_services" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_euro_b2b" />
            <field name="tax_src_id" ref="purchase_tax_25_services" />
            <field name="tax_dest_id" ref="purchase_services_tax_25_EC" />
        </record>
        <record id="fpp_euro_25_goods" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_euro_b2b" />
            <field name="tax_src_id" ref="purchase_tax_25_goods" />
            <field name="tax_dest_id" ref="purchase_goods_tax_25_EC" />
        </record>
        <record id="fpp_euro_12_services" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_euro_b2b" />
            <field name="tax_src_id" ref="purchase_tax_12_services" />
            <field name="tax_dest_id" ref="purchase_services_tax_12_EC" />
        </record>
        <record id="fpp_euro_12_goods" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_euro_b2b" />
            <field name="tax_src_id" ref="purchase_tax_12_goods" />
            <field name="tax_dest_id" ref="purchase_goods_tax_12_EC" />
        </record>
        <record id="fpp_euro_6_services" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_euro_b2b" />
            <field name="tax_src_id" ref="purchase_tax_6_services" />
            <field name="tax_dest_id" ref="purchase_services_tax_6_EC" />
        </record>
        <record id="fpp_euro_6_goods" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_euro_b2b" />
            <field name="tax_src_id" ref="purchase_tax_6_goods" />
            <field name="tax_dest_id" ref="purchase_goods_tax_6_EC" />
        </record>
        <!-- Fiscal Position VAT on sales eurozone -->
        <record id="fps_euro_25_services" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_euro_b2b" />
            <field name="tax_src_id" ref="sale_tax_25_services" />
            <field name="tax_dest_id" ref="sale_tax_services_EC" />
        </record>
        <record id="fps_euro_25_goods" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_euro_b2b" />
            <field name="tax_src_id" ref="sale_tax_25_goods" />
            <field name="tax_dest_id" ref="sale_tax_goods_EC" />
        </record>
        <record id="fps_euro_12_services" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_euro_b2b" />
            <field name="tax_src_id" ref="sale_tax_12_services" />
            <field name="tax_dest_id" ref="sale_tax_services_EC" />
        </record>
        <record id="fps_euro_12_goods" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_euro_b2b" />
            <field name="tax_src_id" ref="sale_tax_12_goods" />
            <field name="tax_dest_id" ref="sale_tax_goods_EC" />
        </record>
        <record id="fps_euro_6_services" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_euro_b2b" />
            <field name="tax_src_id" ref="sale_tax_6_services" />
            <field name="tax_dest_id" ref="sale_tax_services_EC" />
        </record>
        <record id="fps_euro_6_goods" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_euro_b2b" />
            <field name="tax_src_id" ref="sale_tax_6_goods" />
            <field name="tax_dest_id" ref="sale_tax_goods_EC" />
        </record>
        <!-- Fiscal Position Purchase None Eurozone -->
        <record id="fpp_outside_25_services" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_outside_euro" />
            <field name="tax_src_id" ref="purchase_tax_25_services" />
            <field name="tax_dest_id" ref="purchase_services_tax_25_NEC" />
        </record>
        <record id="fpp_outside_25_goods" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_outside_euro" />
            <field name="tax_src_id" ref="purchase_tax_25_goods" />
            <field name="tax_dest_id" ref="purchase_goods_tax_25_NEC" />
        </record>
        <record id="fpp_outside_12_services" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_outside_euro" />
            <field name="tax_src_id" ref="purchase_tax_12_services" />
            <field name="tax_dest_id" ref="purchase_services_tax_12_NEC" />
        </record>
        <record id="fpp_outside_12_goods" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_outside_euro" />
            <field name="tax_src_id" ref="purchase_tax_12_goods" />
            <field name="tax_dest_id" ref="purchase_goods_tax_12_NEC" />
        </record>
        <record id="fpp_outside_6_services" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_outside_euro" />
            <field name="tax_src_id" ref="purchase_tax_6_services" />
            <field name="tax_dest_id" ref="purchase_services_tax_6_NEC" />
        </record>
        <record id="fpp_outside_6_goods" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_outside_euro" />
            <field name="tax_src_id" ref="purchase_tax_6_goods" />
            <field name="tax_dest_id" ref="purchase_goods_tax_6_NEC" />
        </record>
        <!-- Fiscal Position VAT on sales eurozone -->
        <record id="fps_outside_25_services" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_outside_euro" />
            <field name="tax_src_id" ref="sale_tax_25_services" />
            <field name="tax_dest_id" ref="sale_tax_services_NEC" />
        </record>
        <record id="fps_outside_25_goods" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_outside_euro" />
            <field name="tax_src_id" ref="sale_tax_25_goods" />
            <field name="tax_dest_id" ref="sale_tax_goods_NEC" />
        </record>
        <record id="fps_outside_12_services" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_outside_euro" />
            <field name="tax_src_id" ref="sale_tax_12_services" />
            <field name="tax_dest_id" ref="sale_tax_services_NEC" />
        </record>
        <record id="fps_outside_12_goods" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_outside_euro" />
            <field name="tax_src_id" ref="sale_tax_12_goods" />
            <field name="tax_dest_id" ref="sale_tax_goods_NEC" />
        </record>
        <record id="fps_outside_6_services" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_outside_euro" />
            <field name="tax_src_id" ref="sale_tax_6_services" />
            <field name="tax_dest_id" ref="sale_tax_services_NEC" />
        </record>
        <record id="fps_outside_6_goods" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fp_outside_euro" />
            <field name="tax_src_id" ref="sale_tax_6_goods" />
            <field name="tax_dest_id" ref="sale_tax_goods_NEC" />
        </record>
    </data>
</odoo>

```

## File: data\account_fiscal_position_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="fp_sweden" model="account.fiscal.position.template">
            <field name="name">Sverige</field>
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="country_id" ref="base.se"/>
            <field name="vat_required" eval="True"/>
            <field name="sequence">10</field>
        </record>
        <record id="fp_euro_b2c" model="account.fiscal.position.template">
            <field name="name">Europaunionen (B2C)</field>
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
            <field name="sequence">11</field>
        </record>
        <record id="fp_euro_b2b" model="account.fiscal.position.template">
            <field name="name">Europaunionen (B2B)</field>
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
            <field name="sequence">12</field>
        </record>
        <record id="fp_outside_euro" model="account.fiscal.position.template">
            <field name="name">Utanför Europaunionen</field>
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="sequence">13</field>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_25" model="account.tax.group">
            <field name="name">VAT 25%</field>
            <field name="country_id" ref="base.se"/>
        </record>
        <record id="tax_group_12" model="account.tax.group">
            <field name="name">VAT 12%</field>
            <field name="country_id" ref="base.se"/>
        </record>
        <record id="tax_group_6" model="account.tax.group">
            <field name="name">VAT 6%</field>
            <field name="country_id" ref="base.se"/>
        </record>
        <record id="tax_group_0" model="account.tax.group">
            <field name="name">VAT 0%</field>
            <field name="country_id" ref="base.se"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report" model="account.report">
        <field name="name">skatterapport</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.se"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_title_sales" model="account.report.line">
                <field name="name">Block A – Momspliktig försäljning eller uttag exklusive moms</field>
                <field name="code">se_a</field>
                <field name="aggregation_formula">se_05.balance + se_06.balance + se_07.balance + se_08.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_05" model="account.report.line">
                        <field name="name">Fält 05 – Momspliktig försäljning som inte ingår i fält 06, 07 eller 08</field>
                        <field name="code">se_05</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_05_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_05</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_06" model="account.report.line">
                        <field name="name">Fält 06 – Momspliktiga uttag</field>
                        <field name="code">se_06</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_06_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_06</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_07" model="account.report.line">
                        <field name="name">Fält 07 – Beskattningsunderlag vid vinstmarginalbeskattning</field>
                        <field name="code">se_07</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_07_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_07</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_08" model="account.report.line">
                        <field name="name">Fält 08 – Hyresinkomster vid frivillig skattskyldighet</field>
                        <field name="code">se_08</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_08_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_08</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_output_vat_sales" model="account.report.line">
                <field name="name">Block B – Utgående moms på försäljning eller uttag i fält 05–08</field>
                <field name="code">se_b</field>
                <field name="aggregation_formula">se_10.balance + se_11.balance + se_12.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_10" model="account.report.line">
                        <field name="name">Fält 10 – Utgående moms 25 %</field>
                        <field name="code">se_10</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_11" model="account.report.line">
                        <field name="name">Fält 11 – Utgående moms 12 %</field>
                        <field name="code">se_11</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_11</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_12" model="account.report.line">
                        <field name="name">Fält 12 – Utgående moms 6 %</field>
                        <field name="code">se_12</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_12</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_purchases" model="account.report.line">
                <field name="name">Block C – Momspliktiga inköp vid omvänd skattskyldighet</field>
                <field name="code">se_c</field>
                <field name="aggregation_formula">se_20.balance + se_21.balance + se_22.balance + se_23.balance + se_24.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_20" model="account.report.line">
                        <field name="name">Fält 20 – Inköp av varor från annat EU-land</field>
                        <field name="code">se_20</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_20_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_20</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_21" model="account.report.line">
                        <field name="name">Fält 21 – Inköp av tjänster från ett annat EU-land, enligt huvudregeln</field>
                        <field name="code">se_21</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_21_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_21</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_22" model="account.report.line">
                        <field name="name">Fält 22 – Inköp av tjänster från länder utanför EU</field>
                        <field name="code">se_22</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_22_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_22</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_23" model="account.report.line">
                        <field name="name">Fält 23 – Inköp av varor i Sverige</field>
                        <field name="code">se_23</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_23_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_23</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_24" model="account.report.line">
                        <field name="name">Fält 24 – Övriga inköp av tjänster</field>
                        <field name="code">se_24</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_24_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_24</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_output_vat_purchases" model="account.report.line">
                <field name="name">Block D – Utgående moms på inköp i fält 20–24</field>
                <field name="code">se_d</field>
                <field name="aggregation_formula">se_30.balance + se_31.balance + se_32.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_30" model="account.report.line">
                        <field name="name">Fält 30 – Utgående moms 25 %</field>
                        <field name="code">se_30</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_30_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_30</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_31" model="account.report.line">
                        <field name="name">Fält 31 – Utgående moms 12 %</field>
                        <field name="code">se_31</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_31_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_31</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_32" model="account.report.line">
                        <field name="name">Fält 32 – Utgående moms 6 %</field>
                        <field name="code">se_32</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_32_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_32</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_imports" model="account.report.line">
                <field name="name">Block H - moms vid import</field>
                <field name="code">se_h</field>
                <field name="aggregation_formula">se_50.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_50" model="account.report.line">
                        <field name="name">Fält 50 - Beskattningsunderlag vid import</field>
                        <field name="code">se_50</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_50_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_50</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_output_vat_imports" model="account.report.line">
                <field name="name">Block I - Utgående moms på import i fält 50</field>
                <field name="code">se_i</field>
                <field name="aggregation_formula">se_60.balance + se_61.balance + se_62.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_60" model="account.report.line">
                        <field name="name">Fält 60 – Utgående moms 25 %</field>
                        <field name="code">se_60</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_60_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_60</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_61" model="account.report.line">
                        <field name="name">Fält 61 – Utgående moms 12 %</field>
                        <field name="code">se_61</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_61_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_61</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_62" model="account.report.line">
                        <field name="name">Fält 62 – Utgående moms 6 %</field>
                        <field name="code">se_62</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_62_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_62</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_exempt_sales" model="account.report.line">
                <field name="name">Block E – Försäljning m.m. som är undantagen från moms</field>
                <field name="code">se_e</field>
                <field name="aggregation_formula">se_35.balance + se_36.balance + se_37.balance + se_38.balance + se_39.balance + se_40.balance + se_41.balance + se_42.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_35" model="account.report.line">
                        <field name="name">Fält 35 – Försäljning av varor till ett annat EU-land</field>
                        <field name="code">se_35</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_35_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_35</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_36" model="account.report.line">
                        <field name="name">Fält 36 – Försäljning av varor utanför EU</field>
                        <field name="code">se_36</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_36_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_36</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_37" model="account.report.line">
                        <field name="name">Fält 37 – Mellanmans inköp av varor vid trepartshandel</field>
                        <field name="code">se_37</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_37_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_37</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_38" model="account.report.line">
                        <field name="name">Fält 38 – Mellanmans försäljning av varor vid trepartshandel</field>
                        <field name="code">se_38</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_38_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_38</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_39" model="account.report.line">
                        <field name="name">Fält 39 – Försäljning av tjänster till en beskattningsbar person (näringsidkare) i ett annat EU-land, enligt huvudregeln </field>
                        <field name="code">se_39</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_39_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_39</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_40" model="account.report.line">
                        <field name="name">Fält 40 – Övrig försäljning av tjänster omsatta utanför Sverige</field>
                        <field name="code">se_40</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_40_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_40</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_41" model="account.report.line">
                        <field name="name">Fält 41 – Försäljning när köparen är skattskyldig i Sverige</field>
                        <field name="code">se_41</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_41_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_41</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_42" model="account.report.line">
                        <field name="name">Fält 42 – Övrig försäljning m.m.</field>
                        <field name="code">se_42</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_42_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_42</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_input_vat" model="account.report.line">
                <field name="name">Block F – Ingående moms</field>
                <field name="code">se_f</field>
                <field name="aggregation_formula">se_48.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_48" model="account.report.line">
                        <field name="name">Fält 48 – Ingående moms att dra av</field>
                        <field name="code">se_48</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_48_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">se_48</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_title_vat_debt_credit" model="account.report.line">
                <field name="name">Block G – Moms att betala eller få tillbaka</field>
                <field name="code">se_g</field>
                <field name="aggregation_formula">se_b.balance+se_i.balance+se_d.balance-se_f.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_49" model="account.report.line">
                        <field name="name">Fält 49 – Moms att betala eller få tillbaka</field>
                        <field name="code">se_49</field>
                        <field name="aggregation_formula">se_b.balance+se_i.balance+se_d.balance-se_f.balance</field>
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
    <data>
        <record id="sale_tax_25_goods" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Utgående moms 25%</field>
            <field name="description">ST25</field>
            <field name="amount">25</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_05_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2611'),
                    'plus_report_expression_ids': [ref('tax_report_line_10_tag')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_05_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2611'),
                    'minus_report_expression_ids': [ref('tax_report_line_10_tag')],
                })]"/>
        </record>
        <record id="sale_tax_25_services" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Utgående moms Tjänst 25%</field>
            <field name="description">ST25</field>
            <field name="amount">25</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_05_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2611'),
                    'plus_report_expression_ids': [ref('tax_report_line_10_tag')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_05_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2611'),
                    'minus_report_expression_ids': [ref('tax_report_line_10_tag')],
                })]"/>
        </record>
        <record id="purchase_tax_25_goods" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Ingående moms 25%</field>
            <field name="description">PT25</field>
            <field name="amount">25</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                })]"/>
        </record>
        <record id="purchase_tax_25_services" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Ingående moms Tjänst 25%</field>
            <field name="description">PT25</field>
            <field name="amount">25</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                })]"/>
        </record>
        <record id="sale_tax_12_goods" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Utgående moms 12%</field>
            <field name="description">ST12</field>
            <field name="amount">12</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_05_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2621'),
                    'plus_report_expression_ids': [ref('tax_report_line_11_tag')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_05_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2621'),
                    'minus_report_expression_ids': [ref('tax_report_line_11_tag')],
                })]"/>
        </record>
        <record id="sale_tax_12_services" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Utgående moms Tjänst 12%</field>
            <field name="description">ST12</field>
            <field name="amount">12</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_05_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2621'),
                    'plus_report_expression_ids': [ref('tax_report_line_11_tag')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_05_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2621'),
                    'minus_report_expression_ids': [ref('tax_report_line_11_tag')],
                })]"/>
        </record>
        <record id="purchase_tax_12_goods" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Ingående moms 12%</field>
            <field name="description">PT12</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                })]"/>
        </record>
        <record id="purchase_tax_12_services" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Ingående moms Tjänst 12%</field>
            <field name="description">PT12</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                })]"/>
        </record>
        <record id="sale_tax_6_goods" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Utgående moms 6%</field>
            <field name="description">ST6</field>
            <field name="amount">6</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_05_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2631'),
                    'plus_report_expression_ids': [ref('tax_report_line_12_tag')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_05_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2631'),
                    'minus_report_expression_ids': [ref('tax_report_line_12_tag')],
                })]"/>
        </record>
        <record id="sale_tax_6_services" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Utgående moms Tjänst 6%</field>
            <field name="description">ST6</field>
            <field name="amount">6</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_05_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2631'),
                    'plus_report_expression_ids': [ref('tax_report_line_12_tag')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_05_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2631'),
                    'minus_report_expression_ids': [ref('tax_report_line_12_tag')],
                })]"/>
        </record>
        <record id="purchase_tax_6_goods" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Ingående moms 6%</field>
            <field name="description">PT6</field>
            <field name="amount">6</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                })]"/>
        </record>
        <record id="purchase_tax_6_services" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Ingående moms Tjänst 6%</field>
            <field name="description">PT6</field>
            <field name="amount">6</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                })]"/>
        </record>
        <!-- Tax template VAT in EC goods -->
        <record id="sale_tax_services_EC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Momsfri försäljning av tjänst EU</field>
            <field name="description">SE0</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_39_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax'                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_39_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax'                 })]"/>
        </record>
        <record id="sale_tax_goods_EC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Momsfri Försäljning av varor EU</field>
            <field name="description">SE0</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_35_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax'                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_35_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax'                 })]"/>
        </record>
        <record id="purchase_goods_tax_25_EC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Inköp av varor EU moms 25%</field>
            <field name="description">PE25</field>
            <field name="amount">25</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_20_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_expression_ids': [ref('tax_report_line_30_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_20_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_expression_ids': [ref('tax_report_line_30_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')                 })]"/>
        </record>
        <record id="purchase_goods_tax_12_EC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Inköp av varor EU moms 12%</field>
            <field name="description">PE12</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_20_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_expression_ids': [ref('tax_report_line_31_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_20_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_expression_ids': [ref('tax_report_line_31_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')                 })]"/>
        </record>
        <record id="purchase_goods_tax_6_EC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Inköp av varor EU moms 6%</field>
            <field name="description">PE6</field>
            <field name="amount">6</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_20_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_expression_ids': [ref('tax_report_line_32_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_20_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_expression_ids': [ref('tax_report_line_32_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')                 })]"/>
        </record>
        <!-- Tax template VAT in EC services -->
        <record id="purchase_services_tax_25_EC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Inköp av tjänst EU moms 25%</field>
            <field name="description">PE25</field>
            <field name="amount">25</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_21_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_expression_ids': [ref('tax_report_line_30_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_21_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_expression_ids': [ref('tax_report_line_30_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')                 })]"/>
        </record>
        <record id="purchase_services_tax_12_EC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Inköp av tjänst EU moms 12%</field>
            <field name="description">PE12</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_21_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_expression_ids': [ref('tax_report_line_31_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_21_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_expression_ids': [ref('tax_report_line_31_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')                 })]"/>
        </record>
        <record id="purchase_services_tax_6_EC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Inköp av tjänst EU moms 6%</field>
            <field name="description">PE6</field>
            <field name="amount">6</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_21_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_expression_ids': [ref('tax_report_line_32_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_21_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_expression_ids': [ref('tax_report_line_32_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')                 })]"/>
        </record>
        <!-- Construction services -->
        <record id="purchase_construction_services_tax_25_EC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Inköpta tjänster i Sverige, omvändskattskyldighet, 25 %</field>
            <field name="description">PCS25</field>
            <field name="amount">25</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_24_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2647'),
                    'plus_report_expression_ids': [ref('tax_report_line_30_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_24_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2647'),
                    'minus_report_expression_ids': [ref('tax_report_line_30_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')                 })]"/>
        </record>
        <record id="purchase_construction_services_tax_12_EC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Inköpta tjänster i Sverige, omvändskattskyldighet, 12 %</field>
            <field name="description">PCS12</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_24_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a4426'),
                    'plus_report_expression_ids': [ref('tax_report_line_31_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_24_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a4426'),
                    'minus_report_expression_ids': [ref('tax_report_line_31_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')                 })]"/>
        </record>
        <record id="purchase_construction_services_tax_6_EC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Inköpta tjänster i Sverige, omvändskattskyldighet, 6 %</field>
            <field name="description">PCS6</field>
            <field name="amount">6</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_24_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a4427'),
                    'plus_report_expression_ids': [ref('tax_report_line_32_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_24_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a4427'),
                    'minus_report_expression_ids': [ref('tax_report_line_32_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')                 })]"/>
        </record>
        <!-- Tax template VAT Export -->
        <record id="sale_tax_services_NEC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Momsfri försäljning av tjänst utanför EU</field>
            <field name="description">SE0</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_39_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax'                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_39_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax'                 })]"/>
        </record>
        <record id="sale_tax_goods_NEC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Momsfri försäljning av varor utanför EU</field>
            <field name="description">SE0</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_36_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax'                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_36_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax'                 })]"/>
        </record>
        <record id="purchase_goods_tax_25_NEC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Beskattningsunderlag vid import 25%</field>
            <field name="description">PN25</field>
            <field name="amount">25</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_50_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_expression_ids': [ref('tax_report_line_60_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2615')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_50_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_expression_ids': [ref('tax_report_line_60_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2615')                 })]"/>
        </record>
        <record id="purchase_goods_tax_12_NEC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Beskattningsunderlag vid import 12%</field>
            <field name="description">PN12</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_50_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_expression_ids': [ref('tax_report_line_61_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2625')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_50_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_expression_ids': [ref('tax_report_line_61_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2625')                 })]"/>
        </record>
        <record id="purchase_goods_tax_6_NEC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Beskattningsunderlag vid import 6%</field>
            <field name="description">PN6</field>
            <field name="amount">6</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_50_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_expression_ids': [ref('tax_report_line_62_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2635')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_50_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_expression_ids': [ref('tax_report_line_62_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2635')                 })]"/>
        </record>
        <record id="purchase_services_tax_25_NEC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Inköp av tjänster utanför EU 25%</field>
            <field name="description">PN25</field>
            <field name="amount">25</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_22_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_expression_ids': [ref('tax_report_line_30_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_22_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_expression_ids': [ref('tax_report_line_30_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')                 })]"/>
        </record>
        <record id="purchase_services_tax_12_NEC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Inköp av tjänster utanför EU 12%</field>
            <field name="description">PN12</field>
            <field name="amount">12</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_22_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_expression_ids': [ref('tax_report_line_31_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_22_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_expression_ids': [ref('tax_report_line_31_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')                 })]"/>
        </record>
        <record id="purchase_services_tax_6_NEC" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Inköp av tjänster utanför EU 6%</field>
            <field name="description">PN6</field>
            <field name="amount">6</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_22_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_expression_ids': [ref('tax_report_line_32_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')                 })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_22_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_expression_ids': [ref('tax_report_line_32_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_48_tag')],
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')                 })]"/>
        </record>
        <!--Tax template in triangular trade-->
        <record id="triangular_tax_25_goods" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Trepartshandel - moms 25%</field>
            <field name="description">T25</field>
            <field name="amount">25</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_37_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_38_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2615'),
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_37_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_38_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2615'),
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                })]"/>
        </record>
        <record id="triangular_tax_12_goods" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Trepartshandel - moms 12%</field>
            <field name="description">T12</field>
            <field name="amount">12</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_37_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_38_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2625'),
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_37_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_38_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2625'),
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                })]"/>
        </record>
        <record id="triangular_tax_6_goods" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Trepartshandel - moms 6%</field>
            <field name="description">T6</field>
            <field name="amount">6</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_37_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_38_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2635'),
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_37_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_38_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('a2635'),
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                })]"/>
        </record>
        <record id="triangular_tax_0_goods" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Trepartshandel - momsfrei</field>
            <field name="description">T0</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_37_tag')],
                    'minus_report_expression_ids': [ref('tax_report_line_38_tag')],
                }),
                (0, 0, {
                    'repartition_type': 'tax',
            })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_37_tag')],
                    'plus_report_expression_ids': [ref('tax_report_line_38_tag')],
                }),
                (0, 0, {'repartition_type': 'tax'})]"/>
        </record>
    </data>
</odoo>

```

## File: data\res_country_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="se_partner_address_form" model="ir.ui.view">
        <field name="name">se.partner.form.address</field>
        <field name="model">res.partner</field>
        <field name="priority" eval="900"/>
        <field name="arch" type="xml">
            <form>
                <div class="o_address_format">
                    <field name="parent_id" invisible="1"/>
                    <field name="type" invisible="1"/>
                    <field name="street" placeholder="Street" class="o_address_street"
                            attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="street2" placeholder="Neighborhood" class="o_address_street"
                            attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="zip" placeholder="ZIP" class="o_address_zip"
                            attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="city" placeholder="City" class="o_address_city"
                            attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="state_id" class="o_address_state" placeholder="State..." options='{"no_open": True}'
                            attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="country_id" placeholder="Country" class="o_address_country" options='{"no_open": True, "no_create": True}'
                            attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                    <field name="state_id" class="o_address_state" placeholder="State..." options='{"no_open": True}'
                            attrs="{'readonly': [('type', '=', 'contact'),('parent_id', '!=', False)]}"/>
                </div>
            </form>
        </field>
    </record>
    <record id="base.se" model="res.country">
        <field name="address_view_id" ref="se_partner_address_form" />
        <field name="address_format" eval="'%(street)s\n%(street2)s\n%(zip)s %(city)s %(state_code)s\n%(country_name)s'"/>
    </record>
</odoo>

```

## File: migrations\1.1\end-migrate.py

```python
from odoo.addons.account.models.chart_template import update_taxes_from_templates


def migrate(cr, version):
    update_taxes_from_templates(cr, 'l10n_se.l10nse_chart_template')

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from odoo.exceptions import ValidationError


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    invoice_reference_model = fields.Selection(selection_add=[('se_ocr2', 'Sweden OCR Level 1 & 2'), ('se_ocr3', 'Sweden OCR Level 3'), ('se_ocr4', 'Sweden OCR Level 4')], ondelete={'se_ocr2': 'set default', 'se_ocr3': 'set default', 'se_ocr4': 'set default'})
    l10n_se_invoice_ocr_length = fields.Integer(string='OCR Number Length', help="Total length of OCR Reference Number including checksum.", default=6)

    @api.constrains('l10n_se_invoice_ocr_length')
    def _check_l10n_se_invoice_ocr_length(self):
        for journal in self:
            if journal.l10n_se_invoice_ocr_length < 6:
                raise ValidationError(_('OCR Reference Number length need to be greater than 5. Please correct settings under invoice journal settings.'))

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models, _
from odoo.exceptions import UserError, ValidationError
from stdnum import luhn


class AccountMove(models.Model):
    _inherit = 'account.move'

    def _get_invoice_reference_se_ocr2(self, reference):
        self.ensure_one()
        return reference + luhn.calc_check_digit(reference)

    def _get_invoice_reference_se_ocr3(self, reference):
        self.ensure_one()
        reference = reference + str(len(reference) + 2)[:1]
        return reference + luhn.calc_check_digit(reference)

    def _get_invoice_reference_se_ocr4(self, reference):
        self.ensure_one()

        ocr_length = self.journal_id.l10n_se_invoice_ocr_length

        if len(reference) + 1 > ocr_length:
            raise UserError(_("OCR Reference Number length is greater than allowed. Allowed length in invoice journal setting is %s.") % str(ocr_length))

        reference = reference.rjust(ocr_length - 1, '0')
        return reference + luhn.calc_check_digit(reference)


    def _get_invoice_reference_se_ocr2_invoice(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr2(str(self.id))

    def _get_invoice_reference_se_ocr3_invoice(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr3(str(self.id))

    def _get_invoice_reference_se_ocr4_invoice(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr4(str(self.id))

    def _get_invoice_reference_se_ocr2_partner(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr2(self.partner_id.ref if str(self.partner_id.ref).isdecimal() else str(self.partner_id.id))

    def _get_invoice_reference_se_ocr3_partner(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr3(self.partner_id.ref if str(self.partner_id.ref).isdecimal() else str(self.partner_id.id))

    def _get_invoice_reference_se_ocr4_partner(self):
        self.ensure_one()
        return self._get_invoice_reference_se_ocr4(self.partner_id.ref if str(self.partner_id.ref).isdecimal() else str(self.partner_id.id))

    @api.onchange('partner_id')
    def _onchange_partner_id(self):
        """ If Vendor Bill and Vendor OCR is set, add it. """
        if self.partner_id and self.move_type == 'in_invoice' and self.partner_id.l10n_se_default_vendor_payment_ref:
            self.payment_reference = self.partner_id.l10n_se_default_vendor_payment_ref
        return super(AccountMove, self)._onchange_partner_id()

    @api.constrains('payment_reference', 'state')
    def _l10n_se_check_payment_reference(self):
        for invoice in self:
            if (
                (invoice.payment_reference or invoice.state == 'posted')
                and invoice.partner_id
                and invoice.move_type == 'in_invoice'
                and invoice.partner_id.l10n_se_check_vendor_ocr
                and invoice.country_code == 'SE'
            ):
                try:
                    luhn.validate(invoice.payment_reference)
                except Exception:
                    raise ValidationError(_("Vendor require OCR Number as payment reference. Payment reference isn't a valid OCR Number."))

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, fields, models
import re


class ResCompany(models.Model):
    _inherit = 'res.company'

    org_number = fields.Char(compute='_compute_org_number')

    @api.depends('vat')
    def _compute_org_number(self):
        for company in self:
            if company.account_fiscal_country_id.code == "SE" and company.vat:
                org_number = re.sub(r'\D', '', company.vat)[:-2]
                org_number = org_number[:6] + '-' + org_number[6:]

                company.org_number = org_number
            else:
                company.org_number = ''

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models, api, _
from stdnum import luhn


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_se_check_vendor_ocr = fields.Boolean(string='Check Vendor OCR', help='This Vendor uses OCR Number on their Vendor Bills.')
    l10n_se_default_vendor_payment_ref = fields.Char(string='Default Vendor Payment Ref', help='If set, the vendor uses the same Default Payment Reference or OCR Number on all their Vendor Bills.')

    @api.onchange('l10n_se_default_vendor_payment_ref')
    def onchange_l10n_se_default_vendor_payment_ref(self):
        if not self.l10n_se_default_vendor_payment_ref == "" and self.l10n_se_check_vendor_ocr:
            reference = self.l10n_se_default_vendor_payment_ref
            try:
                luhn.validate(reference)
            except: 
                return {'warning': {'title': _('Warning'), 'message': _('Default vendor OCR number isn\'t a valid OCR number.')}}

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_company
from . import account_move
from . import account_journal
from . import res_partner

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="6.05" width="50.4" height="34.25" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.62" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="220" height="138" transform="translate(4.8 6.05) scale(0.23 0.25)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAANwAAACWCAYAAAC1meaLAAAACXBIWXMAADBKAAAwSgHCONjjAAACKUlEQVR4Xu3doVFDURRF0fcZdHwwlEAdqYUeEPSDRlADIlABhjgEmpmPY3DPfLYIa+lj98x1dxm3D+vgx35/HO+H+9lsM1ePd+N0upnNOBMXswGwHcFBSHAQEhyEBAchwUFIcBASHIQEByHBQUhwEBIchAQHIcFBSHAQEhyEBAchwUFIcBASHIQEByHBQUhwEBIchAQHIcFBSHAQEhyEBAchwUFIcBASHIQEByHBQUhwEBIchAQHIcFBSHAQEhyEBAchwUFIcBASHIQEByHBQUhwEBIchAQHIcFBSHAQEhyEBAchwUFIcBASHIQEByHBQUhwEFpensY6G/0nu8sxrnez1XbePsf4/JqtOBfL+iw4qDgpISQ4CAkOQoKDkOAgJDgICQ5CgoOQ4CAkOAgJDkKCg5DgICQ4CAkOQoKDkOAgJDgICQ5CgoOQ4CAkOAgJDkKCg5DgICQ4CAkOQpevH7PJ/+KZB39pGbcPnnn8st8fx/vhfjbbzNXj3TidbmYzzoSTEkKCg5DgICQ4CAkOQoKDkOAgJDgICQ5CgoOQ4CAkOAgJDkKCg5DgICQ4CAkOQoKDkOAgJDgICQ5CgoOQ4CAkOAgJDkKCg5DgICQ4CAkOQoKDkOAgJDgICQ5CgoOQ4CAkOAgJDkKCg5DgICQ4CAkOQoKDkOAgJDgICQ5CgoOQ4CAkOAgJDkKCg5DgICQ4CAkOQoKDkOAgJDgICQ5CgoPQsq7rbANs5Bs2+iNtsVkpdgAAAABJRU5ErkJggg=="/>
    </g>
  </g>
</svg>

```

## File: views\account_journal_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="view_account_journal_se_ocr_form" model="ir.ui.view">
            <field name="name">account.journal.se.ocr.form</field>
            <field name="model">account.journal</field>
            <field name="inherit_id" ref="account.view_account_journal_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='invoice_reference_model']" position="after">
                    <field name="l10n_se_invoice_ocr_length" attrs="{'invisible': [('invoice_reference_model', '!=', 'se_ocr4')]}"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\partner_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_partner_ocr_form" model="ir.ui.view">
            <field name="name">res.partner.ocr.form</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="account.view_partner_property_form"/>
            <field name="arch" type="xml">
                <group name="accounting_entries" position="after">
                    <group string="Payment Options Sweden" name="payment_options">
                        <field name="l10n_se_check_vendor_ocr"/>
                        <field name="l10n_se_default_vendor_payment_ref"/>
                    </group>
                </group>
            </field>
        </record>
    </data>
</odoo>

```

