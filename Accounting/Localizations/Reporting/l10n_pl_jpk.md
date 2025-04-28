# Odoo Module: l10n_pl_jpk

Category: Accounting/Localizations/Reporting

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Poland - JPK_VAT Community',
    'version': '1.0',
    'description': """
        All the fields needed for the JPK_VAT Export in Poland are available in Community
    """,
    'category': 'Accounting/Localizations/Reporting',
    'depends': [
        'l10n_pl',
    ],
    'data': [
        'security/ir.model.access.csv',
        'data/l10n_pl_tax_office.csv',
        'views/account_move_views.xml',
        'views/product_views.xml',
        'views/report_invoice.xml',
        'views/res_config_settings_views.xml',
        'views/res_partner_views.xml',
    ],
    'auto_install': True,
    'installable': True,
    'website': 'https://www.odoo.com/app/accounting',
    'license': 'LGPL-3',
}

```

## File: data\l10n_pl_tax_office.csv

```csv
"id","code","name"
"pl_tax_office_0202","0202","URZĄD SKARBOWY W BOLESŁAWCU"
"pl_tax_office_0203","0203","URZĄD SKARBOWY W BYSTRZYCY KŁODZKIEJ"
"pl_tax_office_0204","0204","URZĄD SKARBOWY W DZIERŻONIOWIE"
"pl_tax_office_0205","0205","URZĄD SKARBOWY W GŁOGOWIE"
"pl_tax_office_0206","0206","URZĄD SKARBOWY W JAWORZE"
"pl_tax_office_0207","0207","URZĄD SKARBOWY W JELENIEJ GÓRZE"
"pl_tax_office_0208","0208","URZĄD SKARBOWY W KAMIENNEJ GÓRZE"
"pl_tax_office_0209","0209","URZĄD SKARBOWY W KŁODZKU"
"pl_tax_office_0210","0210","URZĄD SKARBOWY W LEGNICY"
"pl_tax_office_0211","0211","URZĄD SKARBOWY W LUBANIU"
"pl_tax_office_0212","0212","URZĄD SKARBOWY W LUBINIE"
"pl_tax_office_0213","0213","URZĄD SKARBOWY W LWÓWKU ŚLĄSKIM"
"pl_tax_office_0214","0214","URZĄD SKARBOWY W MILICZU"
"pl_tax_office_0215","0215","URZĄD SKARBOWY W NOWEJ RUDZIE"
"pl_tax_office_0216","0216","URZĄD SKARBOWY W OLEŚNICY"
"pl_tax_office_0217","0217","URZĄD SKARBOWY W OŁAWIE"
"pl_tax_office_0218","0218","URZĄD SKARBOWY W STRZELINIE"
"pl_tax_office_0219","0219","URZĄD SKARBOWY W ŚRODZIE ŚLĄSKIEJ"
"pl_tax_office_0220","0220","URZĄD SKARBOWY W ŚWIDNICY"
"pl_tax_office_0221","0221","URZĄD SKARBOWY W TRZEBNICY"
"pl_tax_office_0222","0222","URZĄD SKARBOWY W WAŁBRZYCHU"
"pl_tax_office_0223","0223","URZĄD SKARBOWY W WOŁOWIE"
"pl_tax_office_0224","0224","URZĄD SKARBOWY WROCŁAW-FABRYCZNA"
"pl_tax_office_0225","0225","URZĄD SKARBOWY WROCŁAW-KRZYKI"
"pl_tax_office_0226","0226","URZĄD SKARBOWY WROCŁAW-PSIE POLE"
"pl_tax_office_0227","0227","URZĄD SKARBOWY WROCŁAW-STARE MIASTO"
"pl_tax_office_0228","0228","URZĄD SKARBOWY WROCŁAW-ŚRÓDMIEŚCIE"
"pl_tax_office_0229","0229","PIERWSZY URZĄD SKARBOWY WE WROCŁAWIU"
"pl_tax_office_0230","0230","URZĄD SKARBOWY W ZĄBKOWICACH ŚLĄSKICH"
"pl_tax_office_0231","0231","URZĄD SKARBOWY W ZGORZELCU"
"pl_tax_office_0232","0232","URZĄD SKARBOWY W ZŁOTORYI"
"pl_tax_office_0233","0233","URZĄD SKARBOWY W GÓRZE"
"pl_tax_office_0234","0234","URZĄD SKARBOWY W POLKOWICACH"
"pl_tax_office_0271","0271","DOLNOŚLĄSKI URZĄD SKARBOWY WE WROCŁAWIU"
"pl_tax_office_0402","0402","URZĄD SKARBOWY W ALEKSANDROWIE KUJAWSKIM"
"pl_tax_office_0403","0403","URZĄD SKARBOWY W BRODNICY"
"pl_tax_office_0404","0404","PIERWSZY URZĄD SKARBOWY W BYDGOSZCZY"
"pl_tax_office_0405","0405","DRUGI URZĄD SKARBOWY W BYDGOSZCZY"
"pl_tax_office_0406","0406","TRZECI URZĄD SKARBOWY W BYDGOSZCZY"
"pl_tax_office_0407","0407","URZĄD SKARBOWY W CHEŁMNIE"
"pl_tax_office_0408","0408","URZĄD SKARBOWY W GRUDZIĄDZU"
"pl_tax_office_0409","0409","URZĄD SKARBOWY W INOWROCŁAWIU"
"pl_tax_office_0410","0410","URZĄD SKARBOWY W LIPNIE"
"pl_tax_office_0411","0411","URZĄD SKARBOWY W MOGILNIE"
"pl_tax_office_0412","0412","URZĄD SKARBOWY W NAKLE NAD NOTECIĄ"
"pl_tax_office_0413","0413","URZĄD SKARBOWY W RADZIEJOWIE"
"pl_tax_office_0414","0414","URZĄD SKARBOWY W RYPINIE"
"pl_tax_office_0415","0415","URZĄD SKARBOWY W ŚWIECIU"
"pl_tax_office_0416","0416","PIERWSZY URZĄD SKARBOWY W TORUNIU"
"pl_tax_office_0417","0417","DRUGI URZĄD SKARBOWY W TORUNIU"
"pl_tax_office_0418","0418","URZĄD SKARBOWY W TUCHOLI"
"pl_tax_office_0419","0419","URZĄD SKARBOWY W WĄBRZEŹNIE"
"pl_tax_office_0420","0420","URZĄD SKARBOWY WE WŁOCŁAWKU"
"pl_tax_office_0421","0421","URZĄD SKARBOWY W ŻNINIE"
"pl_tax_office_0422","0422","URZĄD SKARBOWY W GOLUBIU-DOBRZYNIU"
"pl_tax_office_0423","0423","URZĄD SKARBOWY W SĘPÓLNIE KRAJEŃSKIM"
"pl_tax_office_0471","0471","KUJAWSKO-POMORSKI URZĄD SKARBOWY W BYDGOSZCZY"
"pl_tax_office_0602","0602","URZĄD SKARBOWY W BIAŁEJ PODLASKIEJ"
"pl_tax_office_0603","0603","URZĄD SKARBOWY W BIŁGORAJU"
"pl_tax_office_0604","0604","URZĄD SKARBOWY W CHEŁMIE"
"pl_tax_office_0605","0605","URZĄD SKARBOWY W HRUBIESZOWIE"
"pl_tax_office_0606","0606","URZĄD SKARBOWY W JANOWIE LUBELSKIM"
"pl_tax_office_0607","0607","URZĄD SKARBOWY W KRASNYMSTAWIE"
"pl_tax_office_0608","0608","URZĄD SKARBOWY W KRAŚNIKU"
"pl_tax_office_0609","0609","URZĄD SKARBOWY W LUBARTOWIE"
"pl_tax_office_0610","0610","PIERWSZY URZĄD SKARBOWY W LUBLINIE"
"pl_tax_office_0611","0611","DRUGI URZĄD SKARBOWY W LUBLINIE"
"pl_tax_office_0612","0612","TRZECI URZĄD SKARBOWY W LUBLINIE"
"pl_tax_office_0613","0613","URZĄD SKARBOWY W ŁUKOWIE"
"pl_tax_office_0614","0614","URZĄD SKARBOWY W OPOLU LUBELSKIM"
"pl_tax_office_0615","0615","URZĄD SKARBOWY W PARCZEWIE"
"pl_tax_office_0616","0616","URZĄD SKARBOWY W PUŁAWACH"
"pl_tax_office_0617","0617","URZĄD SKARBOWY W RADZYNIU PODLASKIM"
"pl_tax_office_0618","0618","URZĄD SKARBOWY W TOMASZOWIE LUBELSKIM"
"pl_tax_office_0619","0619","URZĄD SKARBOWY WE WŁODAWIE"
"pl_tax_office_0620","0620","URZĄD SKARBOWY W ZAMOŚCIU"
"pl_tax_office_0621","0621","URZĄD SKARBOWY W ŁĘCZNEJ"
"pl_tax_office_0622","0622","URZĄD SKARBOWY W RYKACH"
"pl_tax_office_0671","0671","LUBELSKI URZĄD SKARBOWY W LUBLINIE"
"pl_tax_office_0802","0802","URZĄD SKARBOWY W GORZOWIE WIELKOPOLSKIM"
"pl_tax_office_0803","0803","URZĄD SKARBOWY W KROŚNIE ODRZAŃSKIM"
"pl_tax_office_0804","0804","URZĄD SKARBOWY W MIĘDZYRZECZU"
"pl_tax_office_0805","0805","URZĄD SKARBOWY W NOWEJ SOLI"
"pl_tax_office_0806","0806","URZĄD SKARBOWY W SŁUBICACH"
"pl_tax_office_0807","0807","URZĄD SKARBOWY W ŚWIEBODZINIE"
"pl_tax_office_0808","0808","PIERWSZY URZĄD SKARBOWY W ZIELONEJ GÓRZE"
"pl_tax_office_0809","0809","DRUGI URZĄD SKARBOWY W ZIELONEJ GÓRZE"
"pl_tax_office_0810","0810","URZĄD SKARBOWY W ŻAGANIU"
"pl_tax_office_0811","0811","URZĄD SKARBOWY W ŻARACH"
"pl_tax_office_0812","0812","URZĄD SKARBOWY W DREZDENKU"
"pl_tax_office_0813","0813","URZĄD SKARBOWY W SULĘCINIE"
"pl_tax_office_0814","0814","URZĄD SKARBOWY WE WSCHOWIE"
"pl_tax_office_0871","0871","LUBUSKI URZĄD SKARBOWY W ZIELONEJ GÓRZE"
"pl_tax_office_1002","1002","URZĄD SKARBOWY W BEŁCHATOWIE"
"pl_tax_office_1003","1003","URZĄD SKARBOWY W BRZEZINACH"
"pl_tax_office_1004","1004","URZĄD SKARBOWY W GŁOWNIE"
"pl_tax_office_1005","1005","URZĄD SKARBOWY W KUTNIE"
"pl_tax_office_1006","1006","URZĄD SKARBOWY W ŁASKU"
"pl_tax_office_1007","1007","URZĄD SKARBOWY W ŁOWICZU"
"pl_tax_office_1008","1008","PIERWSZY URZĄD SKARBOWY ŁÓDŹ-BAŁUTY"
"pl_tax_office_1009","1009","DRUGI URZĄD SKARBOWY ŁÓDŹ-BAŁUTY"
"pl_tax_office_1010","1010","PIERWSZY URZĄD SKARBOWY ŁÓDŹ-GÓRNA"
"pl_tax_office_1011","1011","DRUGI URZĄD SKARBOWY ŁÓDŹ-GÓRNA"
"pl_tax_office_1012","1012","URZĄD SKARBOWY ŁÓDŹ-POLESIE"
"pl_tax_office_1013","1013","URZĄD SKARBOWY ŁÓDŹ-ŚRÓDMIEŚCIE"
"pl_tax_office_1014","1014","URZĄD SKARBOWY ŁÓDŹ-WIDZEW"
"pl_tax_office_1015","1015","URZĄD SKARBOWY W OPOCZNIE"
"pl_tax_office_1016","1016","URZĄD SKARBOWY W PABIANICACH"
"pl_tax_office_1017","1017","URZĄD SKARBOWY W PIOTRKOWIE TRYBUNALSKIM"
"pl_tax_office_1018","1018","URZĄD SKARBOWY W PODDĘBICACH"
"pl_tax_office_1019","1019","URZĄD SKARBOWY W RADOMSKU"
"pl_tax_office_1020","1020","URZĄD SKARBOWY W RAWIE MAZOWIECKIEJ"
"pl_tax_office_1021","1021","URZĄD SKARBOWY W SIERADZU"
"pl_tax_office_1022","1022","URZĄD SKARBOWY W SKIERNIEWICACH"
"pl_tax_office_1023","1023","URZĄD SKARBOWY W TOMASZOWIE MAZOWIECKIM"
"pl_tax_office_1024","1024","URZĄD SKARBOWY W WIELUNIU"
"pl_tax_office_1025","1025","URZĄD SKARBOWY W ZDUŃSKIEJ WOLI"
"pl_tax_office_1026","1026","URZĄD SKARBOWY W ZGIERZU"
"pl_tax_office_1027","1027","URZĄD SKARBOWY W WIERUSZOWIE"
"pl_tax_office_1028","1028","URZĄD SKARBOWY W ŁĘCZYCY"
"pl_tax_office_1029","1029","URZĄD SKARBOWY W PAJĘCZNIE"
"pl_tax_office_1071","1071","ŁÓDZKI URZĄD SKARBOWY W ŁODZI"
"pl_tax_office_1202","1202","URZĄD SKARBOWY W BOCHNI"
"pl_tax_office_1203","1203","URZĄD SKARBOWY W BRZESKU"
"pl_tax_office_1204","1204","URZĄD SKARBOWY W CHRZANOWIE"
"pl_tax_office_1205","1205","URZĄD SKARBOWY W DĄBROWIE TARNOWSKIEJ"
"pl_tax_office_1206","1206","URZĄD SKARBOWY W GORLICACH"
"pl_tax_office_1207","1207","PIERWSZY URZĄD SKARBOWY KRAKÓW"
"pl_tax_office_1208","1208","URZĄD SKARBOWY KRAKÓW-KROWODRZA"
"pl_tax_office_1209","1209","URZĄD SKARBOWY KRAKÓW-NOWA HUTA"
"pl_tax_office_1210","1210","URZĄD SKARBOWY KRAKÓW-PODGÓRZE"
"pl_tax_office_1211","1211","URZĄD SKARBOWY KRAKÓW-PRĄDNIK"
"pl_tax_office_1212","1212","URZĄD SKARBOWY KRAKÓW-STARE MIASTO"
"pl_tax_office_1213","1213","URZĄD SKARBOWY KRAKÓW-ŚRÓDMIEŚCIE"
"pl_tax_office_1214","1214","URZĄD SKARBOWY W LIMANOWEJ"
"pl_tax_office_1215","1215","URZĄD SKARBOWY W MIECHOWIE"
"pl_tax_office_1216","1216","URZĄD SKARBOWY W MYŚLENICACH"
"pl_tax_office_1217","1217","URZĄD SKARBOWY W NOWYM SĄCZU"
"pl_tax_office_1218","1218","URZĄD SKARBOWY W NOWYM TARGU"
"pl_tax_office_1219","1219","URZĄD SKARBOWY W OLKUSZU"
"pl_tax_office_1220","1220","URZĄD SKARBOWY W OŚWIĘCIMIU"
"pl_tax_office_1221","1221","URZĄD SKARBOWY W PROSZOWICACH"
"pl_tax_office_1222","1222","URZĄD SKARBOWY W SUCHEJ BESKIDZKIEJ"
"pl_tax_office_1223","1223","PIERWSZY URZĄD SKARBOWY W TARNOWIE"
"pl_tax_office_1224","1224","DRUGI URZĄD SKARBOWY W TARNOWIE"
"pl_tax_office_1225","1225","URZĄD SKARBOWY W WADOWICACH"
"pl_tax_office_1226","1226","URZĄD SKARBOWY W WIELICZCE"
"pl_tax_office_1227","1227","URZĄD SKARBOWY W ZAKOPANEM"
"pl_tax_office_1228","1228","DRUGI URZĄD SKARBOWY KRAKÓW"
"pl_tax_office_1271","1271","MAŁOPOLSKI URZĄD SKARBOWY W KRAKOWIE"
"pl_tax_office_1402","1402","URZĄD SKARBOWY W BIAŁOBRZEGACH"
"pl_tax_office_1403","1403","URZĄD SKARBOWY W CIECHANOWIE"
"pl_tax_office_1404","1404","URZĄD SKARBOWY W GARWOLINIE"
"pl_tax_office_1405","1405","URZĄD SKARBOWY W GOSTYNINIE"
"pl_tax_office_1406","1406","URZĄD SKARBOWY W GRODZISKU MAZOWIECKIM"
"pl_tax_office_1407","1407","URZĄD SKARBOWY W GRÓJCU"
"pl_tax_office_1408","1408","URZĄD SKARBOWY W KOZIENICACH"
"pl_tax_office_1409","1409","URZĄD SKARBOWY W LEGIONOWIE"
"pl_tax_office_1410","1410","URZĄD SKARBOWY W ŁOSICACH"
"pl_tax_office_1411","1411","URZĄD SKARBOWY W MAKOWIE MAZOWIECKIM"
"pl_tax_office_1412","1412","URZĄD SKARBOWY W MIŃSKU MAZOWIECKIM"
"pl_tax_office_1413","1413","URZĄD SKARBOWY W MŁAWIE"
"pl_tax_office_1414","1414","URZĄD SKARBOWY W NOWYM DWORZE MAZOWIECKIM"
"pl_tax_office_1415","1415","URZĄD SKARBOWY W OSTROŁĘCE"
"pl_tax_office_1416","1416","URZĄD SKARBOWY W OSTROWI MAZOWIECKIEJ"
"pl_tax_office_1417","1417","URZĄD SKARBOWY W OTWOCKU"
"pl_tax_office_1418","1418","URZĄD SKARBOWY W PIASECZNIE"
"pl_tax_office_1419","1419","URZĄD SKARBOWY W PŁOCKU"
"pl_tax_office_1420","1420","URZĄD SKARBOWY W PŁOŃSKU"
"pl_tax_office_1421","1421","URZĄD SKARBOWY W PRUSZKOWIE"
"pl_tax_office_1422","1422","URZĄD SKARBOWY W PRZASNYSZU"
"pl_tax_office_1423","1423","URZĄD SKARBOWY W PUŁTUSKU"
"pl_tax_office_1424","1424","PIERWSZY URZĄD SKARBOWY W RADOMIU"
"pl_tax_office_1425","1425","DRUGI URZĄD SKARBOWY W RADOMIU"
"pl_tax_office_1426","1426","URZĄD SKARBOWY W SIEDLCACH"
"pl_tax_office_1427","1427","URZĄD SKARBOWY W SIERPCU"
"pl_tax_office_1428","1428","URZĄD SKARBOWY W SOCHACZEWIE"
"pl_tax_office_1429","1429","URZĄD SKARBOWY W SOKOŁOWIE PODLASKIM"
"pl_tax_office_1430","1430","URZĄD SKARBOWY W SZYDŁOWCU"
"pl_tax_office_1431","1431","URZĄD SKARBOWY WARSZAWA-BEMOWO"
"pl_tax_office_1432","1432","URZĄD SKARBOWY WARSZAWA-BIELANY"
"pl_tax_office_1433","1433","URZĄD SKARBOWY WARSZAWA-MOKOTÓW"
"pl_tax_office_1434","1434","URZĄD SKARBOWY WARSZAWA-PRAGA"
"pl_tax_office_1435","1435","PIERWSZY URZĄD SKARBOWY WARSZAWA-ŚRÓDMIEŚCIE"
"pl_tax_office_1436","1436","DRUGI URZĄD SKARBOWY WARSZAWA-ŚRÓDMIEŚCIE"
"pl_tax_office_1437","1437","URZĄD SKARBOWY WARSZAWA-TARGÓWEK"
"pl_tax_office_1438","1438","URZĄD SKARBOWY WARSZAWA-URSYNÓW"
"pl_tax_office_1439","1439","URZĄD SKARBOWY WARSZAWA-WAWER"
"pl_tax_office_1440","1440","URZĄD SKARBOWY WARSZAWA-WOLA"
"pl_tax_office_1441","1441","URZĄD SKARBOWY W WĘGROWIE"
"pl_tax_office_1442","1442","URZĄD SKARBOWY W WOŁOMINIE"
"pl_tax_office_1443","1443","URZĄD SKARBOWY W WYSZKOWIE"
"pl_tax_office_1444","1444","URZĄD SKARBOWY W ZWOLENIU"
"pl_tax_office_1445","1445","URZĄD SKARBOWY W ŻUROMINIE"
"pl_tax_office_1446","1446","URZĄD SKARBOWY W ŻYRARDOWIE"
"pl_tax_office_1447","1447","URZĄD SKARBOWY W LIPSKU"
"pl_tax_office_1448","1448","URZĄD SKARBOWY W PRZYSUSZE"
"pl_tax_office_1449","1449","TRZECI URZĄD SKARBOWY WARSZAWA-ŚRÓDMIEŚCIE"
"pl_tax_office_1471","1471","PIERWSZY MAZOWIECKI URZĄD SKARBOWY W WARSZAWIE"
"pl_tax_office_1472","1472","DRUGI MAZOWIECKI URZĄD SKARBOWY W WARSZAWIE"
"pl_tax_office_1473","1473","TRZECI MAZOWIECKI URZĄD SKARBOWY W RADOMIU"
"pl_tax_office_1602","1602","URZĄD SKARBOWY W BRZEGU"
"pl_tax_office_1603","1603","URZĄD SKARBOWY W GŁUBCZYCACH"
"pl_tax_office_1604","1604","URZĄD SKARBOWY W KĘDZIERZYNIE-KOŹLU"
"pl_tax_office_1605","1605","URZĄD SKARBOWY W KLUCZBORKU"
"pl_tax_office_1606","1606","URZĄD SKARBOWY W NAMYSŁOWIE"
"pl_tax_office_1607","1607","URZĄD SKARBOWY W NYSIE"
"pl_tax_office_1608","1608","URZĄD SKARBOWY W OLEŚNIE"
"pl_tax_office_1609","1609","PIERWSZY URZĄD SKARBOWY W OPOLU"
"pl_tax_office_1610","1610","DRUGI URZĄD SKARBOWY W OPOLU"
"pl_tax_office_1611","1611","URZĄD SKARBOWY W PRUDNIKU"
"pl_tax_office_1612","1612","URZĄD SKARBOWY W STRZELCACH OPOLSKICH"
"pl_tax_office_1613","1613","URZĄD SKARBOWY W KRAPKOWICACH"
"pl_tax_office_1671","1671","OPOLSKI URZĄD SKARBOWY W OPOLU"
"pl_tax_office_1802","1802","URZĄD SKARBOWY W BRZOZOWIE"
"pl_tax_office_1803","1803","URZĄD SKARBOWY W DĘBICY"
"pl_tax_office_1804","1804","URZĄD SKARBOWY W JAROSŁAWIU"
"pl_tax_office_1805","1805","URZĄD SKARBOWY W JAŚLE"
"pl_tax_office_1806","1806","URZĄD SKARBOWY W KOLBUSZOWEJ"
"pl_tax_office_1807","1807","URZĄD SKARBOWY W KROŚNIE"
"pl_tax_office_1808","1808","URZĄD SKARBOWY W LESKU"
"pl_tax_office_1809","1809","URZĄD SKARBOWY W LEŻAJSKU"
"pl_tax_office_1810","1810","URZĄD SKARBOWY W LUBACZOWIE"
"pl_tax_office_1811","1811","URZĄD SKARBOWY W ŁAŃCUCIE"
"pl_tax_office_1812","1812","URZĄD SKARBOWY W MIELCU"
"pl_tax_office_1813","1813","URZĄD SKARBOWY W PRZEMYŚLU"
"pl_tax_office_1814","1814","URZĄD SKARBOWY W PRZEWORSKU"
"pl_tax_office_1815","1815","URZĄD SKARBOWY W ROPCZYCACH"
"pl_tax_office_1816","1816","PIERWSZY URZĄD SKARBOWY W RZESZOWIE"
"pl_tax_office_1817","1817","URZĄD SKARBOWY W SANOKU"
"pl_tax_office_1818","1818","URZĄD SKARBOWY W STALOWEJ WOLI"
"pl_tax_office_1819","1819","URZĄD SKARBOWY W STRZYŻOWIE"
"pl_tax_office_1820","1820","URZĄD SKARBOWY W TARNOBRZEGU"
"pl_tax_office_1821","1821","URZĄD SKARBOWY W USTRZYKACH DOLNYCH"
"pl_tax_office_1822","1822","DRUGI URZĄD SKARBOWY W RZESZOWIE"
"pl_tax_office_1823","1823","URZĄD SKARBOWY W NISKU"
"pl_tax_office_1871","1871","PODKARPACKI URZĄD SKARBOWY W RZESZOWIE"
"pl_tax_office_2002","2002","URZĄD SKARBOWY W AUGUSTOWIE"
"pl_tax_office_2003","2003","PIERWSZY URZĄD SKARBOWY W BIAŁYMSTOKU"
"pl_tax_office_2004","2004","DRUGI URZĄD SKARBOWY W BIAŁYMSTOKU"
"pl_tax_office_2005","2005","URZĄD SKARBOWY W BIELSKU PODLASKIM"
"pl_tax_office_2006","2006","URZĄD SKARBOWY W GRAJEWIE"
"pl_tax_office_2007","2007","URZĄD SKARBOWY W KOLNIE"
"pl_tax_office_2008","2008","URZĄD SKARBOWY W ŁOMŻY"
"pl_tax_office_2009","2009","URZĄD SKARBOWY W MOŃKACH"
"pl_tax_office_2010","2010","URZĄD SKARBOWY W SIEMIATYCZACH"
"pl_tax_office_2011","2011","URZĄD SKARBOWY W SOKÓŁCE"
"pl_tax_office_2012","2012","URZĄD SKARBOWY W SUWAŁKACH"
"pl_tax_office_2013","2013","URZĄD SKARBOWY W WYSOKIEM MAZOWIECKIEM"
"pl_tax_office_2014","2014","URZĄD SKARBOWY W ZAMBROWIE"
"pl_tax_office_2015","2015","URZĄD SKARBOWY W HAJNÓWCE"
"pl_tax_office_2071","2071","PODLASKI URZĄD SKARBOWY W BIAŁYMSTOKU"
"pl_tax_office_2202","2202","URZĄD SKARBOWY W BYTOWIE"
"pl_tax_office_2203","2203","URZĄD SKARBOWY W CHOJNICACH"
"pl_tax_office_2204","2204","URZĄD SKARBOWY W CZŁUCHOWIE"
"pl_tax_office_2205","2205","PIERWSZY URZĄD SKARBOWY W GDAŃSKU"
"pl_tax_office_2206","2206","DRUGI URZĄD SKARBOWY W GDAŃSKU"
"pl_tax_office_2207","2207","TRZECI URZĄD SKARBOWY W GDAŃSKU"
"pl_tax_office_2208","2208","PIERWSZY URZĄD SKARBOWY W GDYNI"
"pl_tax_office_2209","2209","DRUGI URZĄD SKARBOWY W GDYNI"
"pl_tax_office_2210","2210","URZĄD SKARBOWY W KARTUZACH"
"pl_tax_office_2211","2211","URZĄD SKARBOWY W KOŚCIERZYNIE"
"pl_tax_office_2212","2212","URZĄD SKARBOWY W KWIDZYNIE"
"pl_tax_office_2213","2213","URZĄD SKARBOWY W LĘBORKU"
"pl_tax_office_2214","2214","URZĄD SKARBOWY W MALBORKU"
"pl_tax_office_2215","2215","URZĄD SKARBOWY W PUCKU"
"pl_tax_office_2216","2216","URZĄD SKARBOWY W SŁUPSKU"
"pl_tax_office_2217","2217","URZĄD SKARBOWY W SOPOCIE"
"pl_tax_office_2218","2218","URZĄD SKARBOWY W STAROGARDZIE GDAŃSKIM"
"pl_tax_office_2219","2219","URZĄD SKARBOWY W TCZEWIE"
"pl_tax_office_2220","2220","URZĄD SKARBOWY W WEJHEROWIE"
"pl_tax_office_2221","2221","URZĄD SKARBOWY W PRUSZCZU GDAŃSKIM"
"pl_tax_office_2271","2271","POMORSKI URZĄD SKARBOWY W GDAŃSKU"
"pl_tax_office_2402","2402","URZĄD SKARBOWY W BĘDZINIE"
"pl_tax_office_2403","2403","PIERWSZY URZĄD SKARBOWY W BIELSKU-BIAŁEJ"
"pl_tax_office_2404","2404","DRUGI URZĄD SKARBOWY W BIELSKU-BIAŁEJ"
"pl_tax_office_2405","2405","URZĄD SKARBOWY W BYTOMIU"
"pl_tax_office_2406","2406","URZĄD SKARBOWY W CHORZOWIE"
"pl_tax_office_2407","2407","URZĄD SKARBOWY W CIESZYNIE"
"pl_tax_office_2408","2408","URZĄD SKARBOWY W CZECHOWICACH-DZIEDZICACH"
"pl_tax_office_2409","2409","PIERWSZY URZĄD SKARBOWY W CZĘSTOCHOWIE"
"pl_tax_office_2410","2410","DRUGI URZĄD SKARBOWY W CZĘSTOCHOWIE"
"pl_tax_office_2411","2411","URZĄD SKARBOWY W DĄBROWIE GÓRNICZEJ"
"pl_tax_office_2412","2412","PIERWSZY URZĄD SKARBOWY W GLIWICACH"
"pl_tax_office_2413","2413","DRUGI URZĄD SKARBOWY W GLIWICACH"
"pl_tax_office_2414","2414","URZĄD SKARBOWY W JASTRZĘBIU-ZDROJU"
"pl_tax_office_2415","2415","URZĄD SKARBOWY W JAWORZNIE"
"pl_tax_office_2416","2416","PIERWSZY URZĄD SKARBOWY W KATOWICACH"
"pl_tax_office_2417","2417","DRUGI URZĄD SKARBOWY W KATOWICACH"
"pl_tax_office_2418","2418","URZĄD SKARBOWY W KŁOBUCKU"
"pl_tax_office_2419","2419","URZĄD SKARBOWY W LUBLIŃCU"
"pl_tax_office_2420","2420","URZĄD SKARBOWY W MIKOŁOWIE"
"pl_tax_office_2421","2421","URZĄD SKARBOWY W MYSŁOWICACH"
"pl_tax_office_2422","2422","URZĄD SKARBOWY W MYSZKOWIE"
"pl_tax_office_2423","2423","URZĄD SKARBOWY W PIEKARACH ŚLĄSKICH"
"pl_tax_office_2424","2424","URZĄD SKARBOWY W PSZCZYNIE"
"pl_tax_office_2425","2425","URZĄD SKARBOWY W RACIBORZU"
"pl_tax_office_2426","2426","URZĄD SKARBOWY W RUDZIE ŚLĄSKIEJ"
"pl_tax_office_2427","2427","URZĄD SKARBOWY W RYBNIKU"
"pl_tax_office_2428","2428","URZĄD SKARBOWY W SIEMIANOWICACH ŚLĄSKICH"
"pl_tax_office_2429","2429","URZĄD SKARBOWY W SOSNOWCU"
"pl_tax_office_2430","2430","URZĄD SKARBOWY W TARNOWSKICH GÓRACH"
"pl_tax_office_2431","2431","URZĄD SKARBOWY W TYCHACH"
"pl_tax_office_2432","2432","URZĄD SKARBOWY W WODZISŁAWIU ŚLĄSKIM"
"pl_tax_office_2433","2433","URZĄD SKARBOWY W ZABRZU"
"pl_tax_office_2434","2434","URZĄD SKARBOWY W ZAWIERCIU"
"pl_tax_office_2435","2435","URZĄD SKARBOWY W ŻORACH"
"pl_tax_office_2436","2436","URZĄD SKARBOWY W ŻYWCU"
"pl_tax_office_2471","2471","PIERWSZY ŚLĄSKI URZĄD SKARBOWY W SOSNOWCU"
"pl_tax_office_2472","2472","DRUGI ŚLĄSKI URZĄD SKARBOWY W BIELSKU-BIAŁEJ"
"pl_tax_office_2602","2602","URZĄD SKARBOWY W BUSKU-ZDROJU"
"pl_tax_office_2603","2603","URZĄD SKARBOWY W JĘDRZEJOWIE"
"pl_tax_office_2604","2604","PIERWSZY URZĄD SKARBOWY W KIELCACH"
"pl_tax_office_2605","2605","DRUGI URZĄD SKARBOWY W KIELCACH"
"pl_tax_office_2606","2606","URZĄD SKARBOWY W KOŃSKICH"
"pl_tax_office_2607","2607","URZĄD SKARBOWY W OPATOWIE"
"pl_tax_office_2608","2608","URZĄD SKARBOWY W OSTROWCU ŚWIĘTOKRZYSKIM"
"pl_tax_office_2609","2609","URZĄD SKARBOWY W PIŃCZOWIE"
"pl_tax_office_2610","2610","URZĄD SKARBOWY W SANDOMIERZU"
"pl_tax_office_2611","2611","URZĄD SKARBOWY W SKARŻYSKU-KAMIENNEJ"
"pl_tax_office_2612","2612","URZĄD SKARBOWY W STARACHOWICACH"
"pl_tax_office_2613","2613","URZĄD SKARBOWY W STASZOWIE"
"pl_tax_office_2614","2614","URZĄD SKARBOWY W KAZIMIERZY WIELKIEJ"
"pl_tax_office_2615","2615","URZĄD SKARBOWY WE WŁOSZCZOWIE"
"pl_tax_office_2671","2671","ŚWIĘTOKRZYSKI URZĄD SKARBOWY W KIELCACH"
"pl_tax_office_2802","2802","URZĄD SKARBOWY W BARTOSZYCACH"
"pl_tax_office_2803","2803","URZĄD SKARBOWY W BRANIEWIE"
"pl_tax_office_2804","2804","URZĄD SKARBOWY W DZIAŁDOWIE"
"pl_tax_office_2805","2805","URZĄD SKARBOWY W ELBLĄGU"
"pl_tax_office_2806","2806","URZĄD SKARBOWY W EŁKU"
"pl_tax_office_2807","2807","URZĄD SKARBOWY W GIŻYCKU"
"pl_tax_office_2808","2808","URZĄD SKARBOWY W IŁAWIE"
"pl_tax_office_2809","2809","URZĄD SKARBOWY W KĘTRZYNIE"
"pl_tax_office_2810","2810","URZĄD SKARBOWY W NIDZICY"
"pl_tax_office_2811","2811","URZĄD SKARBOWY W NOWYM MIEŚCIE LUBAWSKIM"
"pl_tax_office_2812","2812","URZĄD SKARBOWY W OLECKU"
"pl_tax_office_2813","2813","URZĄD SKARBOWY W OLSZTYNIE"
"pl_tax_office_2814","2814","URZĄD SKARBOWY W OSTRÓDZIE"
"pl_tax_office_2815","2815","URZĄD SKARBOWY W PISZU"
"pl_tax_office_2816","2816","URZĄD SKARBOWY W SZCZYTNIE"
"pl_tax_office_2871","2871","WARMIŃSKO-MAZURSKI URZĄD SKARBOWY W OLSZTYNIE"
"pl_tax_office_3002","3002","URZĄD SKARBOWY W CZARNKOWIE"
"pl_tax_office_3003","3003","URZĄD SKARBOWY W GNIEŹNIE"
"pl_tax_office_3004","3004","URZĄD SKARBOWY W GOSTYNIU"
"pl_tax_office_3005","3005","URZĄD SKARBOWY W GRODZISKU WIELKOPOLSKIM"
"pl_tax_office_3006","3006","URZĄD SKARBOWY W JAROCINIE"
"pl_tax_office_3007","3007","PIERWSZY URZĄD SKARBOWY W KALISZU"
"pl_tax_office_3008","3008","DRUGI URZĄD SKARBOWY W KALISZU"
"pl_tax_office_3009","3009","URZĄD SKARBOWY W KĘPNIE"
"pl_tax_office_3010","3010","URZĄD SKARBOWY W KOLE"
"pl_tax_office_3011","3011","URZĄD SKARBOWY W KONINIE"
"pl_tax_office_3012","3012","URZĄD SKARBOWY W KOŚCIANIE"
"pl_tax_office_3013","3013","URZĄD SKARBOWY W KROTOSZYNIE"
"pl_tax_office_3014","3014","URZĄD SKARBOWY W LESZNIE"
"pl_tax_office_3015","3015","URZĄD SKARBOWY W MIĘDZYCHODZIE"
"pl_tax_office_3016","3016","URZĄD SKARBOWY W NOWYM TOMYŚLU"
"pl_tax_office_3017","3017","URZĄD SKARBOWY W OSTROWIE WIELKOPOLSKIM"
"pl_tax_office_3018","3018","URZĄD SKARBOWY W OSTRZESZOWIE"
"pl_tax_office_3019","3019","URZĄD SKARBOWY W PILE"
"pl_tax_office_3020","3020","URZĄD SKARBOWY POZNAŃ-GRUNWALD"
"pl_tax_office_3021","3021","URZĄD SKARBOWY POZNAŃ-JEŻYCE"
"pl_tax_office_3022","3022","URZĄD SKARBOWY POZNAŃ-NOWE MIASTO"
"pl_tax_office_3023","3023","PIERWSZY URZĄD SKARBOWY W POZNANIU"
"pl_tax_office_3025","3025","URZĄD SKARBOWY POZNAŃ-WINOGRADY"
"pl_tax_office_3026","3026","URZĄD SKARBOWY POZNAŃ-WILDA"
"pl_tax_office_3027","3027","URZĄD SKARBOWY W RAWICZU"
"pl_tax_office_3028","3028","URZĄD SKARBOWY W SŁUPCY"
"pl_tax_office_3029","3029","URZĄD SKARBOWY W SZAMOTUŁACH"
"pl_tax_office_3030","3030","URZĄD SKARBOWY W ŚREMIE"
"pl_tax_office_3031","3031","URZĄD SKARBOWY W ŚRODZIE WIELKOPOLSKIEJ"
"pl_tax_office_3032","3032","URZĄD SKARBOWY W TURKU"
"pl_tax_office_3033","3033","URZĄD SKARBOWY W WĄGROWCU"
"pl_tax_office_3034","3034","URZĄD SKARBOWY W WOLSZTYNIE"
"pl_tax_office_3035","3035","URZĄD SKARBOWY WE WRZEŚNI"
"pl_tax_office_3036","3036","URZĄD SKARBOWY W ZŁOTOWIE"
"pl_tax_office_3037","3037","URZĄD SKARBOWY W CHODZIEŻY"
"pl_tax_office_3038","3038","URZĄD SKARBOWY W OBORNIKACH"
"pl_tax_office_3039","3039","URZĄD SKARBOWY W PLESZEWIE"
"pl_tax_office_3071","3071","PIERWSZY WIELKOPOLSKI URZĄD SKARBOWY W POZNANIU"
"pl_tax_office_3072","3072","DRUGI WIELKOPOLSKI URZĄD SKARBOWY W KALISZU"
"pl_tax_office_3202","3202","URZĄD SKARBOWY W BIAŁOGARDZIE"
"pl_tax_office_3203","3203","URZĄD SKARBOWY W CHOSZCZNIE"
"pl_tax_office_3204","3204","URZĄD SKARBOWY W DRAWSKU POMORSKIM"
"pl_tax_office_3205","3205","URZĄD SKARBOWY W GOLENIOWIE"
"pl_tax_office_3206","3206","URZĄD SKARBOWY W GRYFICACH"
"pl_tax_office_3207","3207","URZĄD SKARBOWY W GRYFINIE"
"pl_tax_office_3208","3208","URZĄD SKARBOWY W KAMIENIU POMORSKIM"
"pl_tax_office_3209","3209","URZĄD SKARBOWY W KOŁOBRZEGU"
"pl_tax_office_3210","3210","PIERWSZY URZĄD SKARBOWY W KOSZALINIE"
"pl_tax_office_3211","3211","DRUGI URZĄD SKARBOWY W KOSZALINIE"
"pl_tax_office_3212","3212","URZĄD SKARBOWY W MYŚLIBORZU"
"pl_tax_office_3213","3213","URZĄD SKARBOWY W PYRZYCACH"
"pl_tax_office_3214","3214","URZĄD SKARBOWY W STARGARDZIE"
"pl_tax_office_3215","3215","PIERWSZY URZĄD SKARBOWY W SZCZECINIE"
"pl_tax_office_3216","3216","DRUGI URZĄD SKARBOWY W SZCZECINIE"
"pl_tax_office_3217","3217","TRZECI URZĄD SKARBOWY W SZCZECINIE"
"pl_tax_office_3218","3218","URZĄD SKARBOWY W SZCZECINKU"
"pl_tax_office_3219","3219","URZĄD SKARBOWY W ŚWINOUJŚCIU"
"pl_tax_office_3220","3220","URZĄD SKARBOWY W WAŁCZU"
"pl_tax_office_3271","3271","ZACHODNIOPOMORSKI URZĄD SKARBOWY W SZCZECINIE"

```

## File: models\account_move.py

```python
from odoo import api, fields, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_pl_vat_b_spv = fields.Boolean(
        string='B_SPV',
        help="Transfer of a single-purpose voucher effected by a taxable person acting on his/its own behalf",
        states={'draft': [('readonly', False)]},
    )
    l10n_pl_vat_b_spv_dostawa = fields.Boolean(
        string='B_SPV_Dostawa',
        help="Supply of goods and/or services covered by a single-purpose voucher to a taxpayer",
        states={'draft': [('readonly', False)]},
    )
    l10n_pl_vat_b_mpv_prowizja = fields.Boolean(
        string='B_MPV_Prowizja',
        help="Supply of agency and other services pertaining to the transfer of a single-purpose voucher",
        states={'draft': [('readonly', False)]},
    )
    l10n_pl_delivery_date = fields.Date(
        string='PL Delivery Date',
        copy=False,
        readonly=True,
        states={'draft': [('readonly', False)]},
    )
    l10n_pl_show_delivery_date = fields.Boolean(compute='_compute_l10n_pl_show_delivery_date')

    @api.depends('country_code', 'l10n_pl_delivery_date')
    def _compute_l10n_pl_show_delivery_date(self):
        for move in self:
            move.l10n_pl_show_delivery_date = move.l10n_pl_delivery_date and move.is_sale_document() and move.country_code == 'PL'

```

## File: models\l10n_pl_tax_office.py

```python
from odoo import fields, models


class TaxOffice(models.Model):
    _name = 'l10n_pl_tax_office'
    _description = 'Tax Office in Poland'
    _rec_names_search = ['name', 'code']
    _order = 'code'

    code = fields.Char('Code', required=True)
    name = fields.Char('Description', required=True)

    _sql_constraints = [
        ('code_company_uniq', 'unique (code)', 'The code of the tax office must be unique !')
    ]

    def name_get(self):
        result = []
        for tax_office in self:
            name = tax_office.code + ' ' + tax_office.name
            result.append((tax_office.id, name))
        return result

```

## File: models\product.py

```python
from odoo import models, fields


class ProductTemplate(models.Model):
    _inherit = 'product.template'

    l10n_pl_vat_gtu = fields.Selection(
        string='GTU Codes',
        selection=[
            ('GTU_01', 'GTU_01 - Alcoholic beverages'),
            ('GTU_02', 'GTU_02 - Goods referred to under Art. 103 sec 5aa'),
            ('GTU_03', 'GTU_03 - Fuel oil for excise duty, lubricating oils and other oils'),
            ('GTU_04', 'GTU_04 - Tobacco products, tobacco, e-liquid'),
            ('GTU_05', 'GTU_05 - Wastes'),
            ('GTU_06', 'GTU_06 - Electronic devices, their parts and materials'),
            ('GTU_07', 'GTU_07 - Vehicles and vehicle parts'),
            ('GTU_08', 'GTU_08 - Precious metals and base metals'),
            ('GTU_09', 'GTU_09 - Medicament and medical devices, medicinal products'),
            ('GTU_10', 'GTU_10 - Buildings, structures and land'),
            ('GTU_11', 'GTU_11 - Services related to the greenhouse gas emission allowance trading'),
            ('GTU_12', 'GTU_12 - Intangible services'),
            ('GTU_13', 'GTU_13 - Transport services and warehouse management services'),
        ],
        help='Codes for specific types of products, needed for VAT declaration'
    )

```

## File: models\res_company.py

```python
from odoo import fields, models


class Company(models.Model):
    _inherit = 'res.company'

    l10n_pl_reports_tax_office_id = fields.Many2one('l10n_pl_tax_office', string='Tax Office', groups="account.group_account_user")

```

## File: models\res_config_settings.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResConfigSettings(models.TransientModel):
    _inherit = 'res.config.settings'

    l10n_pl_reports_tax_office_id = fields.Many2one(related='company_id.l10n_pl_reports_tax_office_id', readonly=False)

```

## File: models\res_partner.py

```python
# coding: utf-8
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class ResPartner(models.Model):
    """Inherited to add the information needed for the JPK"""
    _inherit = 'res.partner'

    l10n_pl_links_with_customer = fields.Boolean(
        string='Links With Company',
        help='TP: Existing connection or influence between the customer and the supplier'
    )

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import product
from . import account_move
from . import res_partner
from . import res_company
from . import res_config_settings
from . import l10n_pl_tax_office

```

## File: security\ir.model.access.csv

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_l10n_pl_tax_office,access_l10n_pl_tax_office,model_l10n_pl_tax_office,account.group_account_user,1,1,1,0

```

## File: views\account_move_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="view_move_form_l10n_pl" model="ir.ui.view">
        <field name="name">account.move.form</field>
        <field name="model">account.move</field>
        <field name="inherit_id" ref="account.view_move_form"/>
        <field name="arch" type="xml">
            <field name="invoice_date" position="after">
                <field name="l10n_pl_show_delivery_date" invisible="1"/>
                <field name="l10n_pl_delivery_date" string="Delivery Date" attrs="{'invisible': [('l10n_pl_show_delivery_date', '=', False)]}"/>
            </field>
            <field name="qr_code_method" position="after">
                <field name="l10n_pl_delivery_date" string="Delivery Date" attrs="{'invisible': [('country_code', '!=', 'PL')]}"/>
            </field>
            <notebook position="inside">
                <page id="pl_extra" string="PL Extra" attrs="{'invisible': ['|', ('move_type', 'not in', ['out_invoice', 'out_refund']), ('country_code', '!=', 'PL')]}">
                    <group>
                        <group>
                            <field name="l10n_pl_vat_b_spv_dostawa"/>
                            <field name="l10n_pl_vat_b_mpv_prowizja"/>
                        </group>
                        <group>
                            <field name="l10n_pl_vat_b_spv"/>
                        </group>
                    </group>
                </page>
            </notebook>
        </field>
    </record>
</odoo>

```

## File: views\product_views.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <record id="product_template_form" model="ir.ui.view">
        <field name="name">product.template.form.inherit</field>
        <field name="model">product.template</field>
        <field name="inherit_id" ref="product.product_template_form_view"/>
        <field name="arch" type="xml">
            <xpath expr="//page[@name='invoicing']//group[@name='accounting']" position="inside">
                <group name="GTU" string="GTU">
                    <field name="l10n_pl_vat_gtu"/>
                </group>
            </xpath>
        </field>
    </record>
</odoo>

```

## File: views\report_invoice.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo>
    <template id="report_invoice_document" inherit_id="account.report_invoice_document">
        <!-- add sale date -->
        <xpath expr="//div[@name='due_date']" position="after">
            <div t-attf-class="#{'col-auto col-3 mw-100' if report_type != 'html' else 'col'} mb-2" t-if="o.l10n_pl_delivery_date" name="l10n_pl_delivery_date">
                <strong>Delivery Date:</strong>
                <p class="m-0" t-field="o.l10n_pl_delivery_date"/>
            </div>
        </xpath>
    </template>
</odoo>

```

## File: views\res_config_settings_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="res_config_settings_view_form" model="ir.ui.view">
        <field name="name">res.config.settings.view.form.inherit.l10n_pl_jpk</field>
        <field name="model">res.config.settings</field>
        <field name="inherit_id" ref="account.res_config_settings_view_form"/>
        <field name="arch" type="xml">
            <xpath expr="//div[@id='invoicing_settings']" position="after">
                <div attrs="{'invisible': [('country_code', '!=', 'PL')]}">
                    <div id="l10n_pl_jpk_title">
                        <h2>Polish Localization</h2>
                    </div>
                    <div id="l10n_pl_jpk_section" class="row mt16 o_settings_container">
                        <div class="col-12 col-lg-6 o_setting_box">
                            <div class="o_setting_left_pane"/>
                            <div class="o_setting_right_pane">
                                <div class="content-group">
                                    <div class="row mt16">
                                        <span>
                                            <label for="l10n_pl_reports_tax_office_id"/>
                                            <span class="fa fa-lg fa-building-o" title="Values set here are company-specific." aria-label="Values set here are company-specific." groups="base.group_multi_company" role="img"/>
                                            <field name="l10n_pl_reports_tax_office_id"/>
                                        </span>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </xpath>
        </field>
    </record>
</odoo>
```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <record  model="ir.ui.view" id="res_partner_account_pl_jpk_form">
            <field name="name">res.partner.account.pl.jpk.form</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="account.view_partner_property_form"/>
            <field name="arch" type="xml">
                <xpath expr="//group[@name='accounting_entries']" position="after">
                    <group string="PL Information">
                        <field name="l10n_pl_links_with_customer"/>
                    </group>
                </xpath>
            </field>
        </record>

    </data>
</odoo>

```

