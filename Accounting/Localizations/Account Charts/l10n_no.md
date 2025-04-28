# Odoo Module: l10n_no

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models


def _preserve_tag_on_taxes(cr, registry):
    from odoo.addons.account.models.chart_template import preserve_existing_tags_on_taxes
    preserve_existing_tags_on_taxes(cr, registry, 'l10n_no')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    "name" : "Norway - Accounting",
    "version" : "2.1",
    "author" : "Rolv Råen",
    'category': 'Accounting/Localizations/Account Charts',
    "description": """This is the module to manage the accounting chart for Norway in Odoo.

Updated for Odoo 9 by Bringsvor Consulting AS <www.bringsvor.com>
""",
    "depends" : [
        "account",
        "base_iban",
        "base_vat",
    ],
    "data": ['data/l10n_no_chart_data.xml',
             'data/account_tax_group_data.xml',
             'data/account_tax_report_data.xml',
             'data/account.account.template.csv',
             'data/account_tax_data.xml',
             'data/account_chart_template_data.xml',
             'views/res_partner_views.xml',
             'views/res_company_views.xml',
             ],
     'demo': [
         'demo/demo_company.xml',
     ],
    'post_init_hook': '_preserve_tag_on_taxes',
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","code","name","user_type_id/id","tag_ids/id","reconcile","chart_template_id/id"
"no_a_cash","1000","Forskning og utvikling","account.data_account_type_current_assets","","False","no_chart_template"
"chart1010","1010","Forskning og utvikling, egenutviklet","account.data_account_type_current_assets","","False","no_chart_template"
"chart1020","1020","Konsesjoner","account.data_account_type_current_assets","","False","no_chart_template"
"chart1030","1030","Patenter","account.data_account_type_current_assets","","False","no_chart_template"
"chart1040","1040","Lisenser","account.data_account_type_current_assets","","False","no_chart_template"
"chart1050","1050","Varemerker","account.data_account_type_current_assets","","False","no_chart_template"
"chart1060","1060","Andre rettigheter","account.data_account_type_current_assets","","False","no_chart_template"
"chart1070","1070","Utsatt skattefordel","account.data_account_type_current_assets","","False","no_chart_template"
"chart1080","1080","Goodwill","account.data_account_type_current_assets","","False","no_chart_template"
"chart1100","1100","Bygninger","account.data_account_type_current_assets","","False","no_chart_template"
"chart1120","1120","Bygningsmessige anlegg","account.data_account_type_current_assets","","False","no_chart_template"
"chart1130","1130","Anlegg under utførelse","account.data_account_type_current_assets","","False","no_chart_template"
"chart1140","1140","Jord- og skogbrukseiendommer","account.data_account_type_current_assets","","False","no_chart_template"
"chart1150","1150","Tomter og andre grunnarealer","account.data_account_type_current_assets","","False","no_chart_template"
"chart1160","1160","Boliger inklusive tomter","account.data_account_type_current_assets","","False","no_chart_template"
"chart1190","1190","Andre anleggsmidler","account.data_account_type_current_assets","","False","no_chart_template"
"chart1200","1200","Maskiner og anlegg","account.data_account_type_current_assets","","False","no_chart_template"
"chart1210","1210","Maskiner og anlegg under utførelse","account.data_account_type_current_assets","","False","no_chart_template"
"chart1220","1220","Skip, rigger, fly","account.data_account_type_current_assets","","False","no_chart_template"
"chart1230","1230","Biler","account.data_account_type_current_assets","","False","no_chart_template"
"chart1239","1239","Vogntog, lastebiler og busser","account.data_account_type_current_assets","","False","no_chart_template"
"chart1240","1240","Andre transportmidler","account.data_account_type_current_assets","","False","no_chart_template"
"chart1249","1249","Andre transportmidler","account.data_account_type_current_assets","","False","no_chart_template"
"chart1250","1250","Inventar","account.data_account_type_current_assets","","False","no_chart_template"
"chart1260","1260","Fast bygningsinventar med annen avskrivning","account.data_account_type_current_assets","","False","no_chart_template"
"chart1270","1270","Verktøy mv.","account.data_account_type_current_assets","","False","no_chart_template"
"chart1280","1280","Kontormaskiner","account.data_account_type_current_assets","","False","no_chart_template"
"chart1290","1290","Andre driftsmidler","account.data_account_type_current_assets","","False","no_chart_template"
"chart1300","1300","Investeringer i datterselskaper","account.data_account_type_current_assets","","False","no_chart_template"
"chart1310","1310","Investeringer annet foretak i samme konsern","account.data_account_type_current_assets","","False","no_chart_template"
"chart1320","1320","Lån til foretak samme konsern","account.data_account_type_current_assets","","False","no_chart_template"
"chart1330","1330","Investeringer i tilknyttede selskap","account.data_account_type_current_assets","","False","no_chart_template"
"chart1340","1340","Lån til tilknyttede selskap","account.data_account_type_current_assets","","False","no_chart_template"
"chart1350","1350","Investeringer i aksjer og andeler","account.data_account_type_current_assets","","False","no_chart_template"
"chart1360","1360","Obligasjoner","account.data_account_type_current_assets","","False","no_chart_template"
"chart1370","1370","Fordringer på eiere, styremedlemmer mv.","account.data_account_type_current_assets","","False","no_chart_template"
"chart1380","1380","Fordringer på ansatte","account.data_account_type_current_assets","","False","no_chart_template"
"chart1390","1390","Andre langsiktige fordringer","account.data_account_type_current_assets","","False","no_chart_template"
"chart1399","1399","Andre fordringer","account.data_account_type_current_assets","","False","no_chart_template"
"chart1400","1400","Råvarer og innkjøpte halvfabrikata","account.data_account_type_current_assets","","False","no_chart_template"
"chart1401","1401","Halvfabrikata","account.data_account_type_current_assets","","False","no_chart_template"
"chart1420","1420","Varer under tilvirkning","account.data_account_type_current_assets","","False","no_chart_template"
"chart1440","1440","Ferdige egentilvirkede varer","account.data_account_type_current_assets","","False","no_chart_template"
"chart1460","1460","Innkjøpte varer for videresalg","account.data_account_type_current_assets","","False","no_chart_template"
"chart1480","1480","Forskuddsbetaling til leverandører","account.data_account_type_current_assets","","False","no_chart_template"
"chart1481","1481","Kortsiktige fordringer hos leverandører","account.data_account_type_current_assets","","False","no_chart_template"
"chart1500","1500","Kundefordringer","account.data_account_type_receivable","","True","no_chart_template"
"chart1501","1501","Kundefordringer (PoS)","account.data_account_type_receivable","","True","no_chart_template"
"chart1509","1509","Ikke reskontroførte kundefordringer","account.data_account_type_current_assets","","False","no_chart_template"
"chart1513","1513","Ikke fakturerte kundefordringer","account.data_account_type_current_assets","","False","no_chart_template"
"chart1514","1514","Kundfordringer - tjenester","account.data_account_type_current_assets","","False","no_chart_template"
"chart1520","1520","Andre kortsiktige fordringer","account.data_account_type_current_assets","","False","no_chart_template"
"chart1530","1530","Opptjente, ikke fakturerte driftsinntekter","account.data_account_type_current_assets","","False","no_chart_template"
"chart1550","1550","Kundefordringer på selskap samme konsern","account.data_account_type_current_assets","","False","no_chart_template"
"chart1560","1560","Andre fordringer på selskap innen samme konsern","account.data_account_type_current_assets","","False","no_chart_template"
"chart1570","1570","Andre kortsiktige fordringer","account.data_account_type_current_assets","","False","no_chart_template"
"chart1571","1571","Lønnsforskudd","account.data_account_type_current_assets","","False","no_chart_template"
"chart1572","1572","Andre kortsiktige lån til ansatte","account.data_account_type_current_assets","","False","no_chart_template"
"chart1579","1579","Andre kortsiktige fordringer","account.data_account_type_current_assets","","False","no_chart_template"
"chart1580","1580","Avsetning tap på fordringer","account.data_account_type_current_assets","","False","no_chart_template"
"chart1600","1600","Utgående merverdiavgift","account.data_account_type_current_assets","","False","no_chart_template"
"chart1601","1601","Utgående merverdiavgift høy sats","account.data_account_type_current_assets","","False","no_chart_template"
"chart1602","1602","Utgående merverdiavgift kjøp tjen. fra utlandet","account.data_account_type_current_assets","","False","no_chart_template"
"chart1603","1603","Utgående merverdiavgift middels sats","account.data_account_type_current_assets","","False","no_chart_template"
"chart1604","1604","Utgående merverdiavgift lav sats","account.data_account_type_current_assets","","False","no_chart_template"
"chart1605","1605","Grunnlag utgående merverdiavgift, høy sats","account.data_account_type_current_assets","","False","no_chart_template"
"chart1606","1606","Grunnlag utgående merverdiavgift, middels sats","account.data_account_type_current_assets","","False","no_chart_template"
"chart1607","1607","Grunnlag utgående merverdiavgift, lav sats","account.data_account_type_current_assets","","False","no_chart_template"
"chart1610","1610","Inngående merverdiavgift","account.data_account_type_current_assets","","False","no_chart_template"
"chart1611","1611","Inngående merverdiavgift høy sats","account.data_account_type_current_assets","","False","no_chart_template"
"chart1612","1612","Inngående merverdiavgift kjøp tjen. fra utlandet","account.data_account_type_current_assets","","False","no_chart_template"
"chart1613","1613","Inngående merverdiavgift middels sats","account.data_account_type_current_assets","","False","no_chart_template"
"chart1614","1614","Inngående merverdiavgift lav sats","account.data_account_type_current_assets","","False","no_chart_template"
"chart1640","1640","Oppgjørskonto merverdiavgift","account.data_account_type_current_assets","","False","no_chart_template"
"chart1670","1670","Krav på offentlige tilskudd","account.data_account_type_current_assets","","False","no_chart_template"
"chart1700","1700","Forskuddsbetalt leie","account.data_account_type_current_assets","","False","no_chart_template"
"chart1710","1710","Forskuddsbetalt rente","account.data_account_type_current_assets","","False","no_chart_template"
"chart1720","1720","Forskuddsbetalt lønn","account.data_account_type_current_assets","","False","no_chart_template"
"chart1740","1740","Forskuddsbetalt, ikke påløpt lønn","account.data_account_type_current_assets","","False","no_chart_template"
"chart1749","1749","Andre forskuddbetalte kostnader","account.data_account_type_current_assets","","False","no_chart_template"
"chart1750","1750","Påløpte leieinntekter","account.data_account_type_current_assets","","False","no_chart_template"
"chart1760","1760","Påløpte renteinntekter","account.data_account_type_current_assets","","False","no_chart_template"
"chart1780","1780","Krav på innbetaling av selskapskapital","account.data_account_type_current_assets","","False","no_chart_template"
"chart1790","1790","Interimskonto","account.data_account_type_current_assets","","False","no_chart_template"
"chart1800","1800","Aksjer & andeler i foretak i samme kons.","account.data_account_type_current_assets","","False","no_chart_template"
"chart1810","1810","Markesdbaserte aksjer","account.data_account_type_current_assets","","False","no_chart_template"
"chart1820","1820","Andre aksjer","account.data_account_type_current_assets","","False","no_chart_template"
"chart1830","1830","Markedsbaserte obligasjoner","account.data_account_type_current_assets","","False","no_chart_template"
"chart1840","1840","Andre obligasjoner","account.data_account_type_current_assets","","False","no_chart_template"
"chart1850","1850","Markedsbaserte sertifikater","account.data_account_type_current_assets","","False","no_chart_template"
"chart1860","1860","Andre sertifikater","account.data_account_type_current_assets","","False","no_chart_template"
"chart1870","1870","Andre markedsbaserte finansielle instrumenter","account.data_account_type_current_assets","account.account_tag_financing","False","no_chart_template"
"chart1880","1880","Andre finansielle instrumenter","account.data_account_type_current_assets","account.account_tag_financing","False","no_chart_template"
"chart1890","1890","Andeler utenfor konsern","account.data_account_type_current_assets","","False","no_chart_template"
"chart1905","1905","Kontanter Euro","account.data_account_type_current_assets","","False","no_chart_template"
"chart1908","1908","Kontanter, annen valuta","account.data_account_type_current_assets","","False","no_chart_template"
"chart1910","1910","Kasse","account.data_account_type_current_assets","","False","no_chart_template"
"chart1918","1918","Kassedifferenser","account.data_account_type_current_assets","","False","no_chart_template"
"chart1919","1919","Kasse i transfer","account.data_account_type_current_assets","","False","no_chart_template"
"chart1921","1921","Bankinnskudd 2","account.data_account_type_current_assets","","False","no_chart_template"
"chart1942","1942","Bank ikke allokert innbetaling","account.data_account_type_current_assets","","False","no_chart_template"
"chart1944","1944","Bank ikke identifisert innbetaling","account.data_account_type_current_assets","","False","no_chart_template"
"chart1950","1950","Bankinnskudd for skattetrekk","account.data_account_type_current_assets","","False","no_chart_template"
"chart1980","1980","Valutakonto Euro","account.data_account_type_current_assets","","False","no_chart_template"
"chart2000","2000","Aksjekapital","account.data_account_type_equity","","False","no_chart_template"
"chart2010","2010","Egne aksjer","account.data_account_type_equity","","False","no_chart_template"
"chart2020","2020","Overkursfond","account.data_account_type_equity","","False","no_chart_template"
"chart2030","2030","Annen innskutt egenkapital","account.data_account_type_equity","","False","no_chart_template"
"chart2036","2036","Stiftelsesutgifter","account.data_account_type_equity","","False","no_chart_template"
"chart2040","2040","Fond for vurderingsforskjeller","account.data_account_type_equity","","False","no_chart_template"
"chart2050","2050","Annen egenkapital","account.data_account_type_equity","","False","no_chart_template"
"chart2055","2055","Innskudd kontanter","account.data_account_type_equity","","False","no_chart_template"
"chart2060","2060","Privatuttak","account.data_account_type_equity","","False","no_chart_template"
"chart2070","2070","Forskuddsskatt","account.data_account_type_equity","","False","no_chart_template"
"chart2080","2080","Udekket tap","account.data_account_type_equity","","False","no_chart_template"
"chart2098","2090","Gevinst eller tap fra foregående år","account.data_account_type_equity","","False","no_chart_template"
"chart2100","2100","Pensjonsforpliktelser","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2120","2120","Utsatt skatt","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2140","2140","Avsetn. for garanti- & serviceforpl.","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2160","2160","Uopptjent inntekt","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2180","2180","Andre avsetninger for forpiktelser","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2188","2188","Avsetning for skyldig merverdiavgift","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2200","2200","Konvertible lån","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2210","2210","Obligsjonslån","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2220","2220","Gjeld til kredittinstitusjoner","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2240","2240","Pantelån","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2251","2251","Gjeld til ansatte","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2255","2255","Gjeld til eiere","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2260","2260","Gjeld til selskap i samme konsern","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2270","2270","Andre valutalån","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2280","2280","Stille interessentinnskudd og ansv. lånekapital","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2299","2299","Annen langsiktig gjeld","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2300","2300","Konvertible lån","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2320","2320","Sertifikatlån","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2340","2340","Andre valutalån","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2360","2360","Byggelån","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2380","2380","Kassakreditt","account.data_account_type_current_liabilities","","True","no_chart_template"
"chart2390","2390","Annen gjeld til kredittinstitusjon","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2400","2400","Leverandørgjeld","account.data_account_type_payable","","True","no_chart_template"
"chart2409","2409","Ikke reskontroført leverandørgjeld","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2441","2441","Leverandørgjeld - tjenester","account.data_account_type_payable","","True","no_chart_template"
"chart2442","2442","Ikke fakturerte mottatte varer","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2460","2460","Leverandørgjeld til selskap i samme konsern","account.data_account_type_payable","","True","no_chart_template"
"chart2490","2490","Påløpt lev.gjeld, mottatte varer","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2491","2491","Vekselgjeld","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2500","2500","Avsatt betalbar skatt, ikke utlignet","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2510","2510","Betalbar skatt, utlignet","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2530","2530","Refusjon skatt etter Skatteloven §31 5. ledd","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2540","2540","Forhåndsskatt","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2600","2600","Forskuddstrekk","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2610","2610","Påleggstrekk","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2620","2620","Bidragstrekk","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2630","2630","Trygdetrekk","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2640","2640","Forsikringstrekk","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2650","2650","Trukket fagforeningskontigent","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2690","2690","Andre trekk","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2700","2700","Utgående merverdiavgift","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2701","2701","Utgående merverdiavgift høy sats","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2702","2702","Utgående merverdiavgift kjøp tjen. fra utlandet","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2703","2703","Utgående merverdiavgift middels sats","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2704","2704","Utgående merverdiavgift lav sats","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2710","2710","Inngående merverdiavgift","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2711","2711","Inngående merverdiavgift høy sats","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2712","2712","Inngående merverdiavgift kjøp tjen. fra utlandet","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2713","2713","Inngående merverdiavgift middels sats","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2714","2714","Inngående merverdiavgift lav sats","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2727","2727","Utgående mva, kjøp varer fra utlandet, høy sats","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2728","2728","Utgående mva, kjøp varer fra utlandet, middels sats","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2740","2740","Oppgjørskonto merverdiavgift","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2741","2741","Inngående mva, kjøp varer fra utlandet, høy sats","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2742","2742","Inngående mva, kjøp varer fra utlandet, middels sats","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2745","2745","Grunnlag utgående mva kjøp av tjenester utland","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2746","2746","Motkonto grunnlag kjøp av tjenester utland","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2761","2761","Grunnlag innførsel varer, høy sats","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2762","2762","Motkonto grunnlag innførsel varer, høy sats","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2763","2763","Grunnlag innførsel varer, middels sats","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2764","2764","Motkonto grunnlag innførsel varer, middels sats","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2765","2765","Grunnlag innførsel varer, ingen mva","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2766","2766","Motkonto grunnlag innførsel varer, ingen mva","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2770","2770","Skyldig arbeidsgiveravgift","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2780","2780","Påløpt arbeidsgiveravgift","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2781","2781","Arb.giv.avg. pål. feriep.","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2785","2785","Påløpt arbeidsgiveravgift ferielønn","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2790","2790","Andre offentlige avgifter","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2800","2800","Avsatt utbytte","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2900","2900","Forskudd fra kunder","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2902","2902","Forskudd kunder gavekort","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2903","2903","Forskudd kunder tilgodelapp","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2910","2910","Gjeld til ansatte og eiere","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2920","2920","Gjeld til selskap i samme konsern","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2930","2930","Lønn","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2940","2940","Feriepenger","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2950","2950","Påløpte renter","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2960","2960","Påløpte kostn. og forskuddsbet. inskudd","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2970","2970","Uopptjent inntekt , avsetning","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2980","2980","Avsetninger og forpliktelser","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart2990","2990","Annen kortsiktig gjeld","account.data_account_type_current_liabilities","","False","no_chart_template"
"chart3000","3000","Salgsinntekt handelsvarer avgiftspl. høy sats","account.data_account_type_revenue","account.account_tag_operating","False","no_chart_template"
"chart3010","3010","Salgsinntekt egentilv. varer avgiftspl. høy sats","account.data_account_type_revenue","account.account_tag_operating","False","no_chart_template"
"chart3020","3020","Salgsinntekt tjenester avgiftspl. høy sats","account.data_account_type_revenue","","False","no_chart_template"
"chart3030","3030","Salgsinntekt handelsvarer avgiftpl. middels sats","account.data_account_type_revenue","","False","no_chart_template"
"chart3035","3035","Salgsinnt. handelsvarer, avgiftspliktig, lav sats","account.data_account_type_revenue","","False","no_chart_template"
"chart3040","3040","Salgsinntekt egentilv. varer avgiftpl middels sats","account.data_account_type_revenue","","False","no_chart_template"
"chart3050","3050","Salgsinntekter tjenester avgiftspl. lav sats","account.data_account_type_revenue","","False","no_chart_template"
"chart3060","3060","Uttak av varer avgiftspliktig høy sats","account.data_account_type_revenue","","False","no_chart_template"
"chart3061","3061","Uttak av varer, avgiftspliktig, høy sats","account.data_account_type_revenue","","False","no_chart_template"
"chart3062","3062","Uttak av varer, avg.pliktig, middels sats","account.data_account_type_revenue","","False","no_chart_template"
"chart3063","3063","Uttak av varer avgiftspliktig middels sats","account.data_account_type_revenue","","False","no_chart_template"
"chart3070","3070","Uttak av tjenester avgiftspliktig høy sats","account.data_account_type_revenue","","False","no_chart_template"
"chart3071","3071","Uttak av tjenester, avgiftspliktig, høy sats","account.data_account_type_revenue","","False","no_chart_template"
"chart3074","3074","Uttak av tjenester avgiftspliktig lav sats","account.data_account_type_revenue","","False","no_chart_template"
"chart3080","3080","Rabatter og annen salgsinntektsred., avgiftspl.","account.data_account_type_revenue","","False","no_chart_template"
"chart3090","3090","Refunderbare utlegg for kjøpers regning, avgiftspl","account.data_account_type_revenue","","False","no_chart_template"
"chart3095","3095","Ikke fakturert omsetning","account.data_account_type_revenue","","False","no_chart_template"
"chart3096","3096","Fakturert men ikke levert","account.data_account_type_revenue","","False","no_chart_template"
"chart3100","3100","Salgsinntekt handelsvarer avgiftsfri","account.data_account_type_revenue","account.account_tag_operating","False","no_chart_template"
"chart3105","3105","Salgsinntekt handelsvarer, utførsel, avgiftsfri","account.data_account_type_revenue","","False","no_chart_template"
"chart3110","3110","Salgsinntekt egentilvirkede varer avgiftsfri","account.data_account_type_revenue","account.account_tag_operating","False","no_chart_template"
"chart3120","3120","Salgsinntekt tjenester avgiftsfri","account.data_account_type_revenue","account.account_tag_operating","False","no_chart_template"
"chart3125","3125","Salgsinntekt tjenester, utførsel, avgiftsfri","account.data_account_type_revenue","","False","no_chart_template"
"chart3160","3160","Uttak av varer avgiftsfritt","account.data_account_type_revenue","account.account_tag_operating","False","no_chart_template"
"chart3170","3170","Uttak av tjenester, avgiftsfritt","account.data_account_type_revenue","","False","no_chart_template"
"chart3180","3180","Rabatter og andre salgsinntektsreduksjon avgiftsfri","account.data_account_type_revenue","","False","no_chart_template"
"chart3190","3190","Refunderbare utlegg for kjøpers regning, avg.fri","account.data_account_type_revenue","","False","no_chart_template"
"chart3200","3200","Salgsinntekt handelsvarervarer utenfor avg.omr","account.data_account_type_revenue","","False","no_chart_template"
"chart3210","3210","Salgsinntekt egentilvirkede varer utenfor avg.omr","account.data_account_type_revenue","","False","no_chart_template"
"chart3220","3220","Salgsinntekt tjenester utenfor avg.omr","account.data_account_type_revenue","","False","no_chart_template"
"chart3260","3260","Uttak av varer utenfor avgiftsområdet","account.data_account_type_revenue","","False","no_chart_template"
"chart3280","3280","Rabatter og annen salgsinntektsreduksjon","account.data_account_type_revenue","","False","no_chart_template"
"chart3281","3281","Kasserabatt, git kunde","account.data_account_type_revenue","","False","no_chart_template"
"chart3300","3300","Spes. offent. avg. tilvirk./solgte varer","account.data_account_type_revenue","","False","no_chart_template"
"chart3301","3301","Spesiell off. avg. tilv./solgte varer fritt","account.data_account_type_revenue","","False","no_chart_template"
"chart3302","3302","Miljø avgift for tilv./solgte varer avgiftspliktig","account.data_account_type_revenue","","False","no_chart_template"
"chart3303","3303","Miljø avgift for tilv./solgte varer avgiftsfritt","account.data_account_type_revenue","","False","no_chart_template"
"chart3400","3400","Spes. offent. avg. tilvirk./solgte varer","account.data_account_type_revenue","","False","no_chart_template"
"chart3440","3440","Spes. offentlige tilskudd for tjenester","account.data_account_type_revenue","","False","no_chart_template"
"chart3500","3500","Uopptjente inntekter garanti","account.data_account_type_revenue","","False","no_chart_template"
"chart3510","3510","Uopptjente inntekter service","account.data_account_type_revenue","","False","no_chart_template"
"chart3600","3600","Leieinntekter fast eiendom","account.data_account_type_revenue","","False","no_chart_template"
"chart3610","3610","Leieinntekter andre varige driftsmidler","account.data_account_type_revenue","","False","no_chart_template"
"chart3620","3620","Andre leieinntekter","account.data_account_type_revenue","","False","no_chart_template"
"chart3700","3700","Provisjonsinntekter","account.data_account_type_revenue","","False","no_chart_template"
"chart3800","3800","Gevinst ved avgang av anleggsmidler","account.data_account_type_revenue","","False","no_chart_template"
"chart3900","3900","Andre driftsrelaterte inntekter, avgiftspliktig","account.data_account_type_revenue","","False","no_chart_template"
"chart3910","3910","Utgående porto, avgiftspliktig","account.data_account_type_revenue","","False","no_chart_template"
"chart3920","3920","Utgående gebyrer, avgiftspliktig","account.data_account_type_revenue","","False","no_chart_template"
"chart3950","3950","Annen driftsrelatert inntekt, avgiftsfritt","account.data_account_type_revenue","","False","no_chart_template"
"chart3960","3960","Utgående porto, avgiftsfritt","account.data_account_type_revenue","","False","no_chart_template"
"chart3970","3970","Utgående gebyrer, avgiftsfritt","account.data_account_type_revenue","","False","no_chart_template"
"chart4000","4000","Innkjøp av råvarer og halvfabrikata høy sats","account.data_account_type_expenses","","False","no_chart_template"
"chart4030","4030","Innkjøp av råvarer og halvfabrikata middels sats","account.data_account_type_expenses","","False","no_chart_template"
"chart4035","4035","Innkjøp varer og halvfabrikata, lav avgiftssats","account.data_account_type_expenses","","False","no_chart_template"
"chart4060","4060","Frakt, toll og spedisjon","account.data_account_type_expenses","","False","no_chart_template"
"chart4070","4070","Innkjøpsprisreduksjon","account.data_account_type_expenses","","False","no_chart_template"
"chart4090","4090","Beholdningsendring","account.data_account_type_expenses","","False","no_chart_template"
"chart4100","4100","Innkjøp varer under tilvirkning høy sats","account.data_account_type_expenses","","False","no_chart_template"
"chart4130","4130","Innkjøp varer under tilvirkning middels sats","account.data_account_type_expenses","","False","no_chart_template"
"chart4160","4160","Frakt, toll og spedisjon","account.data_account_type_expenses","","False","no_chart_template"
"chart4170","4170","Innkjøpsprisreduksjon","account.data_account_type_expenses","","False","no_chart_template"
"chart4190","4190","Beholdningsendring","account.data_account_type_expenses","","False","no_chart_template"
"chart4200","4200","Innkjøp ferdig egentilvirkede varer høy sats","account.data_account_type_expenses","","False","no_chart_template"
"chart4230","4230","Innkjøp ferdig egentilvirkede varer middels sats","account.data_account_type_expenses","","False","no_chart_template"
"chart4260","4260","Frakt, toll og spedisjon","account.data_account_type_expenses","","False","no_chart_template"
"chart4270","4270","Innkjøpsprisreduksjon, avgiftspliktig","account.data_account_type_expenses","","False","no_chart_template"
"chart4290","4290","Beholdningsendring","account.data_account_type_expenses","","False","no_chart_template"
"chart4300","4300","Innkjøp varer for videresalg høy sats","account.data_account_type_expenses","","False","no_chart_template"
"chart4330","4330","Innkjøp varer for videresalg middels sats","account.data_account_type_expenses","","False","no_chart_template"
"chart4360","4360","Frakt, toll m.m. vedr. innkjøp av varer for videresalg","account.data_account_type_expenses","","False","no_chart_template"
"chart4370","4370","Rabatter m.m. vedr. innkjøp av varer for videresalg","account.data_account_type_expenses","","False","no_chart_template"
"chart4380","4380","Varekostnad","account.data_account_type_expenses","","False","no_chart_template"
"chart4390","4390","Beholdningsendring varer for videresalg","account.data_account_type_expenses","","False","no_chart_template"
"chart4400","4400","Fritt Kjøp","account.data_account_type_expenses","","False","no_chart_template"
"chart4470","4470","Innkjøpsprisreduksjoner, fritt","account.data_account_type_expenses","","False","no_chart_template"
"chart4500","4500","Fremmedytelser og underentreprise","account.data_account_type_expenses","","False","no_chart_template"
"chart4590","4590","Beholdningsendring","account.data_account_type_expenses","","False","no_chart_template"
"chart4600","4600","Emballasjematerialer ","account.data_account_type_expenses","","False","no_chart_template"
"chart4690","4690","Beholdningsendring 2","account.data_account_type_expenses","","False","no_chart_template"
"chart4800","4800","Eksp. gebyr pliktig, vareinnkjøp","account.data_account_type_expenses","","False","no_chart_template"
"chart4810","4810","Frakt og porto pliktig, vareinnkjøp","account.data_account_type_expenses","","False","no_chart_template"
"chart4860","4860","Eksp. gebyr fritt, vareinnkjøp","account.data_account_type_expenses","","False","no_chart_template"
"chart4900","4900","Annen periodisering","account.data_account_type_expenses","","False","no_chart_template"
"chart4910","4910","Justering av lager","account.data_account_type_expenses","","False","no_chart_template"
"chart4990","4990","Beholdningsendring","account.data_account_type_expenses","","False","no_chart_template"
"chart5000","5000","Lønn til ansatte","account.data_account_type_expenses","","False","no_chart_template"
"chart5005","5005","Avtalte tariffgodtgjørelser","account.data_account_type_expenses","","False","no_chart_template"
"chart5090","5090","Periodiseringskonto lønn","account.data_account_type_expenses","","False","no_chart_template"
"chart5091","5091","Påløpt, ikke utbetalt lønn","account.data_account_type_expenses","","False","no_chart_template"
"chart5092","5092","Feriepenger","account.data_account_type_expenses","","False","no_chart_template"
"chart5099","5099","Andre lønnsposteringer","account.data_account_type_expenses","","False","no_chart_template"
"chart5100","5100","Lønn til ansatte, timeansatte","account.data_account_type_expenses","","False","no_chart_template"
"chart5105","5105","Avtalte tariffgodtgjørelser, timeansatte","account.data_account_type_expenses","","False","no_chart_template"
"chart5180","5180","Feriepenger beregnet","account.data_account_type_expenses","","False","no_chart_template"
"chart5182","5182","Arbeidsgiveravgift påløpte feriepenger","account.data_account_type_expenses","","False","no_chart_template"
"chart5191","5191","Påløpt, ikke utbetalt lønn timeansatte","account.data_account_type_expenses","","False","no_chart_template"
"chart5192","5192","Feriepenger, timeaansatte","account.data_account_type_expenses","","False","no_chart_template"
"chart5199","5199","Andre lønnsposteringer, timeansatte","account.data_account_type_expenses","","False","no_chart_template"
"chart5200","5200","Fri bil","account.data_account_type_expenses","","False","no_chart_template"
"chart5210","5210","Fri telefon","account.data_account_type_expenses","","False","no_chart_template"
"chart5220","5220","Fri avis","account.data_account_type_expenses","","False","no_chart_template"
"chart5230","5230","Fri losji og bolig","account.data_account_type_expenses","","False","no_chart_template"
"chart5240","5240","Rentefordel","account.data_account_type_expenses","","False","no_chart_template"
"chart5251","5251","Gruppelivsforsikring","account.data_account_type_expenses","","False","no_chart_template"
"chart5252","5252","Ulykkesforsikring","account.data_account_type_expenses","","False","no_chart_template"
"chart5260","5260","Smusstillegg","account.data_account_type_expenses","","False","no_chart_template"
"chart5280","5280","Andre fordeler i arbeidsforhold","account.data_account_type_expenses","","False","no_chart_template"
"chart5290","5290","Motkonto for gruppe 52","account.data_account_type_expenses","","False","no_chart_template"
"chart5300","5300","Tantieme","account.data_account_type_expenses","","False","no_chart_template"
"chart5330","5330","Godtgj. til styre- og bedriftsforsamling","account.data_account_type_expenses","","False","no_chart_template"
"chart5390","5390","Annen oppgavepliktig godtgjørelse","account.data_account_type_expenses","","False","no_chart_template"
"chart5395","5395","Annen oppgavepliktig godtgjørelse, trekkfri","account.data_account_type_expenses","","False","no_chart_template"
"chart5400","5400","Arbeidsgiveravgift","account.data_account_type_expenses","","False","no_chart_template"
"chart5405","5405","Arbeidsgiveravgift av påløpte feriepenger","account.data_account_type_expenses","","False","no_chart_template"
"chart5411","5411","Arb.giv.avg. pål. feriep.","account.data_account_type_expenses","","False","no_chart_template"
"chart5420","5420","Innberetningspliktige pensjonskostnader","account.data_account_type_expenses","","False","no_chart_template"
"chart5430","5430","Premie pensjonsordning","account.data_account_type_expenses","","False","no_chart_template"
"chart5500","5500","Andre kostnadsgodtgjørelser","account.data_account_type_expenses","","False","no_chart_template"
"chart5510","5510","Overtidsmat etter regning","account.data_account_type_expenses","","False","no_chart_template"
"chart5520","5520","Kantinekostnader","account.data_account_type_expenses","","False","no_chart_template"
"chart5600","5600","Arbeidsgodtgjørelse til eiere i DLS","account.data_account_type_expenses","","False","no_chart_template"
"chart5700","5700","Lærlingtilskudd","account.data_account_type_expenses","","False","no_chart_template"
"chart5800","5800","Refusjon av sykepenger","account.data_account_type_expenses","","False","no_chart_template"
"chart5820","5820","Refusjon av arbeidsgiveravgift","account.data_account_type_expenses","","False","no_chart_template"
"chart5890","5890","Annen refusjon","account.data_account_type_expenses","","False","no_chart_template"
"chart5900","5900","Gaver til ansatte","account.data_account_type_expenses","","False","no_chart_template"
"chart5910","5910","Kantinekostnader","account.data_account_type_expenses","","False","no_chart_template"
"chart5920","5920","Yrkesskadeforsikring","account.data_account_type_expenses","","False","no_chart_template"
"chart5930","5930","Andre ikke arb.giv.avg.pliktige forsikr.","account.data_account_type_expenses","","False","no_chart_template"
"chart5940","5940","Øvrige personalkostnader","account.data_account_type_expenses","","False","no_chart_template"
"chart5950","5950","Obligatorisk tjenestepensjon (OTP)","account.data_account_type_expenses","","False","no_chart_template"
"chart5990","5990","Annen personalkostnad","account.data_account_type_expenses","","False","no_chart_template"
"chart6000","6000","Avskrivning på bygn. & annen fast eiend.","account.data_account_type_expenses","","False","no_chart_template"
"chart6010","6010","Avskrivning på transportmidler, maskiner","account.data_account_type_expenses","","False","no_chart_template"
"chart6020","6020","Avskrivning på immaterielle eiendeler","account.data_account_type_expenses","","False","no_chart_template"
"chart6050","6050","Nedskr. varige driftsmidl. & imat. eiend","account.data_account_type_expenses","","False","no_chart_template"
"chart6100","6100","Frakter, transportkostnader og forsikring","account.data_account_type_expenses","","False","no_chart_template"
"chart6110","6110","Toll og spedisjonskostnader ved forsend","account.data_account_type_expenses","","False","no_chart_template"
"chart6200","6200","Elektrisitet","account.data_account_type_expenses","","False","no_chart_template"
"chart6210","6210","Gass","account.data_account_type_expenses","","False","no_chart_template"
"chart6220","6220","Fyringsolje","account.data_account_type_expenses","","False","no_chart_template"
"chart6230","6230","Kull, koks","account.data_account_type_expenses","","False","no_chart_template"
"chart6240","6240","Ved","account.data_account_type_expenses","","False","no_chart_template"
"chart6250","6250","Bensin, dieselolje","account.data_account_type_expenses","","False","no_chart_template"
"chart6260","6260","Vann","account.data_account_type_expenses","","False","no_chart_template"
"chart6290","6290","Annen brensel","account.data_account_type_expenses","","False","no_chart_template"
"chart6300","6300","Leie lokaler","account.data_account_type_expenses","","False","no_chart_template"
"chart6320","6320","Renovasjon, vann, avløp mv.","account.data_account_type_expenses","","False","no_chart_template"
"chart6340","6340","Lys, varme","account.data_account_type_expenses","","False","no_chart_template"
"chart6360","6360","Renhold","account.data_account_type_expenses","","False","no_chart_template"
"chart6390","6390","Annen kostnad lokaler","account.data_account_type_expenses","","False","no_chart_template"
"chart6400","6400","Leie av driftsmidler","account.data_account_type_expenses","","False","no_chart_template"
"chart6410","6410","Leie av inventar","account.data_account_type_expenses","","False","no_chart_template"
"chart6420","6420","Leie datasystemer","account.data_account_type_expenses","","False","no_chart_template"
"chart6430","6430","Leie andre kontormaskiner","account.data_account_type_expenses","","False","no_chart_template"
"chart6440","6440","Leie transportmidler","account.data_account_type_expenses","","False","no_chart_template"
"chart6490","6490","Annen leiekostnad","account.data_account_type_expenses","","False","no_chart_template"
"chart6500","6500","Motordrevet verktøy","account.data_account_type_expenses","","False","no_chart_template"
"chart6510","6510","Håndverktøy","account.data_account_type_expenses","","False","no_chart_template"
"chart6520","6520","Hjelpeverktøy","account.data_account_type_expenses","","False","no_chart_template"
"chart6530","6530","Spesialverktøy","account.data_account_type_expenses","","False","no_chart_template"
"chart6540","6540","Inventar","account.data_account_type_expenses","","False","no_chart_template"
"chart6550","6550","Driftsmaterialer","account.data_account_type_expenses","","False","no_chart_template"
"chart6560","6560","Rekvisita","account.data_account_type_expenses","","False","no_chart_template"
"chart6570","6570","Arbeidsklær og verneutstyr","account.data_account_type_expenses","","False","no_chart_template"
"chart6590","6590","Annet driftsmateriel","account.data_account_type_expenses","","False","no_chart_template"
"chart6600","6600","Reparasjoner og vedlikehold bygninger","account.data_account_type_expenses","","False","no_chart_template"
"chart6620","6620","Reparasjoner og vedlikehold utstyr","account.data_account_type_expenses","","False","no_chart_template"
"chart6690","6690","Reparasjon og vedlikehold annet","account.data_account_type_expenses","","False","no_chart_template"
"chart6700","6700","Revisjonshonorar","account.data_account_type_expenses","","False","no_chart_template"
"chart6720","6720","Honorar for økonomisk & juridisk bistand","account.data_account_type_expenses","","False","no_chart_template"
"chart6750","6750","Honorar regnskapsfører","account.data_account_type_expenses","","False","no_chart_template"
"chart6785","6785","Kjøp av tjenester fra utlandet","account.data_account_type_expenses","","False","no_chart_template"
"chart6800","6800","Kontorrekvisita","account.data_account_type_expenses","","False","no_chart_template"
"chart6810","6810","Datakostnad","account.data_account_type_expenses","","False","no_chart_template"
"chart6820","6820","Trykksaker","account.data_account_type_expenses","","False","no_chart_template"
"chart6840","6840","Aviser, tidsskrifter, bøker mv.","account.data_account_type_expenses","","False","no_chart_template"
"chart6860","6860","Møter, kurs, oppdatering mv.","account.data_account_type_expenses","","False","no_chart_template"
"chart6890","6890","Annen kontorkostnad","account.data_account_type_expenses","","False","no_chart_template"
"chart6900","6900","Telefon","account.data_account_type_expenses","","False","no_chart_template"
"chart6901","6901","Telefon fritt","account.data_account_type_expenses","","False","no_chart_template"
"chart6940","6940","Porto","account.data_account_type_expenses","","False","no_chart_template"
"chart7000","7000","Drivstoff","account.data_account_type_expenses","","False","no_chart_template"
"chart7020","7020","Vedlikehold","account.data_account_type_expenses","","False","no_chart_template"
"chart7040","7040","Forsikringer","account.data_account_type_expenses","","False","no_chart_template"
"chart7080","7080","Bilkostnader, bruk av privat bil i næring","account.data_account_type_expenses","","False","no_chart_template"
"chart7090","7090","Annen kostnad transportmidler","account.data_account_type_expenses","","False","no_chart_template"
"chart7100","7100","Bilgodtgjørelse, oppgavepliktig","account.data_account_type_expenses","","False","no_chart_template"
"chart7130","7130","Reisekostnader, oppgavepliktige","account.data_account_type_expenses","","False","no_chart_template"
"chart7140","7140","Reisekostnader, ikke oppgavepliktig","account.data_account_type_expenses","","False","no_chart_template"
"chart7150","7150","Diettkostnader, oppgaveplikig","account.data_account_type_expenses","","False","no_chart_template"
"chart7160","7160","Diettkostnader, ikke oppgavepliktig","account.data_account_type_expenses","","False","no_chart_template"
"chart7190","7190","Annen kostnadsgodtgjørelse","account.data_account_type_expenses","","False","no_chart_template"
"chart7200","7200","Provisjonskostnader, oppgavepliktige","account.data_account_type_expenses","","False","no_chart_template"
"chart7210","7210","Provisjonskostnader, ikke oppgavepliktig","account.data_account_type_expenses","","False","no_chart_template"
"chart7300","7300","Salgskostnader","account.data_account_type_expenses","","False","no_chart_template"
"chart7320","7320","Reklamekostnader","account.data_account_type_expenses","","False","no_chart_template"
"chart7350","7350","Representasjon, fradragsberettiget","account.data_account_type_expenses","","False","no_chart_template"
"chart7360","7360","Representasjon, ikke fradragsberettiget","account.data_account_type_expenses","","False","no_chart_template"
"chart7390","7390","MVA-øreavrunding","account.data_account_type_expenses","","False","no_chart_template"
"chart7395","7395","Øreavrunding","account.data_account_type_expenses","","False","no_chart_template"
"chart7400","7400","Kontingenter og gaver","account.data_account_type_expenses","","False","no_chart_template"
"chart7410","7410","Kontingenter, ikke fradragsberettiget","account.data_account_type_expenses","","False","no_chart_template"
"chart7420","7420","Gaver, fradragsberettigede","account.data_account_type_expenses","","False","no_chart_template"
"chart7430","7430","Gaver, ikke fradrag","account.data_account_type_expenses","","False","no_chart_template"
"chart7500","7500","Forsikringspremier","account.data_account_type_expenses","","False","no_chart_template"
"chart7550","7550","Garanti- og servicekostnader","account.data_account_type_expenses","","False","no_chart_template"
"chart7560","7560","Servicekostnader","account.data_account_type_expenses","","False","no_chart_template"
"chart7600","7600","Lisenesavgifter og royalties","account.data_account_type_expenses","","False","no_chart_template"
"chart7610","7610","Patentkostnad ved egen patent","account.data_account_type_expenses","","False","no_chart_template"
"chart7620","7620","Kostnader ved varemerker o.l.","account.data_account_type_expenses","","False","no_chart_template"
"chart7630","7630","Kontroll-,prøve- og stempelavgifter","account.data_account_type_expenses","","False","no_chart_template"
"chart7700","7700","Styre- og bedriftsforsamlingsmøter","account.data_account_type_expenses","","False","no_chart_template"
"chart7710","7710","Generalforsamling","account.data_account_type_expenses","","False","no_chart_template"
"chart7730","7730","Kostnader ved egne aksjer","account.data_account_type_expenses","","False","no_chart_template"
"chart7740","7740","Øreavrunding, MVA - oppgjør","account.data_account_type_expenses","","False","no_chart_template"
"chart7745","7745","Øreavrunding, avgiftspliktig","account.data_account_type_expenses","","False","no_chart_template"
"chart7746","7746","Øreavrunding, avgiftsfritt","account.data_account_type_expenses","","False","no_chart_template"
"chart7750","7750","Eiendoms- og festeavgift","account.data_account_type_expenses","","False","no_chart_template"
"chart7770","7770","Bank og kortgebyrer","account.data_account_type_expenses","","False","no_chart_template"
"chart7780","7780","Renter og gebyrer inkasso","account.data_account_type_expenses","","False","no_chart_template"
"chart7798","7798","Annen kostnad, fradragsberettiget","account.data_account_type_expenses","","False","no_chart_template"
"chart7799","7799","Annen kostnad, ikke fradragsberettiget","account.data_account_type_expenses","","False","no_chart_template"
"chart7800","7800","Tap ved avgang driftsmidler","account.data_account_type_expenses","","False","no_chart_template"
"chart7820","7820","Innkommet på tidligere nedskrevne fordri","account.data_account_type_expenses","","False","no_chart_template"
"chart7830","7830","Tap på fordringer","account.data_account_type_expenses","","False","no_chart_template"
"chart7850","7850","Tap pga. brannskade","account.data_account_type_expenses","","False","no_chart_template"
"chart7860","7860","Tap på kontrakter","account.data_account_type_expenses","","False","no_chart_template"
"chart7900","7900","Beholdningsendring anlegg under utførelse","account.data_account_type_expenses","","False","no_chart_template"
"chart7910","7910","Ukurante varer","account.data_account_type_expenses","","False","no_chart_template"
"chart8000","8000","Inntekter på investeringer i datterselskap","account.data_account_type_expenses","","False","no_chart_template"
"chart8010","8010","Inntekt på investering i annet foretak. s/ konsern","account.data_account_type_expenses","","False","no_chart_template"
"chart8020","8020","Inntekt på investering i tilknyttet selskap","account.data_account_type_expenses","","False","no_chart_template"
"chart8030","8030","Renteinntekt på foretak i  samme konsern","account.data_account_type_expenses","","False","no_chart_template"
"chart8040","8040","Renteinntekter, skattefrie","account.data_account_type_expenses","","False","no_chart_template"
"chart8050","8050","Annen renteinntekt","account.data_account_type_expenses","","False","no_chart_template"
"chart8056","8056","Påminnelsesavgift","account.data_account_type_expenses","","False","no_chart_template"
"chart8060","8060","Valutagevinst (agio)","account.data_account_type_expenses","","False","no_chart_template"
"chart8065","8065","Valutakursvinster, revenue","account.data_account_type_expenses","","False","no_chart_template"
"chart8070","8070","Annen finansinntekt","account.data_account_type_expenses","account.account_tag_financing","False","no_chart_template"
"chart8071","8071","Aksjeutbytte","account.data_account_type_expenses","","False","no_chart_template"
"chart8078","8078","Gevinst realisasjon av aksjer","account.data_account_type_expenses","","False","no_chart_template"
"chart8080","8080","Verdiøkning finanseille omløpsmidler","account.data_account_type_expenses","account.account_tag_financing","False","no_chart_template"
"chart8090","8090","Inntekt på andre investeringer","account.data_account_type_expenses","","False","no_chart_template"
"chart8100","8100","Verdired. av markedsbas.finans. omløps.","account.data_account_type_expenses","account.account_tag_financing","False","no_chart_template"
"chart8110","8110","Nedskrivn. av andre finansielle omløps.","account.data_account_type_expenses","account.account_tag_financing","False","no_chart_template"
"chart8120","8120","Nedskrivning av finansielle anleggsmidl.","account.data_account_type_expenses","account.account_tag_financing","False","no_chart_template"
"chart8130","8130","Rentekostnad foretak i samme konsern","account.data_account_type_expenses","","False","no_chart_template"
"chart8140","8140","Rentekostnader, ikke fradragsberettigede","account.data_account_type_expenses","","False","no_chart_template"
"chart8150","8150","Annen rentekostnad","account.data_account_type_expenses","","False","no_chart_template"
"chart8160","8160","Valutatap (disagio)","account.data_account_type_expenses","","False","no_chart_template"
"chart8178","8178","Tap ved realisasjon av aksjer","account.data_account_type_expenses","","False","no_chart_template"
"chart8179","8179","Annen finanskostnad","account.data_account_type_expenses","","False","no_chart_template"
"chart8300","8300","Betalbar skatt","account.data_account_type_expenses","","False","no_chart_template"
"chart8320","8320","Utsatt skatt","account.data_account_type_expenses","","False","no_chart_template"
"chart8350","8350","Skattekostnad","account.data_account_type_expenses","","False","no_chart_template"
"chart8400","8400","Ekstraordinær inntekt","account.data_account_type_expenses","","False","no_chart_template"
"chart8500","8500","Ekstraordinær kostnad","account.data_account_type_expenses","","False","no_chart_template"
"chart8600","8600","Betalbar skatt, ekstraordinært resultat","account.data_account_type_expenses","","False","no_chart_template"
"chart8620","8620","Utsatt skatt, ekstraordinært resultat","account.data_account_type_expenses","","False","no_chart_template"
"chart8800","8800","Årsresultat","account.data_account_type_expenses","","False","no_chart_template"
"chart8900","8900","Overføringer fond for vurderingsforskjel","account.data_account_type_expenses","","False","no_chart_template"
"chart8910","8910","Overføringer felleseid andelskapital for","account.data_account_type_expenses","","False","no_chart_template"
"chart8920","8920","Avsatt utbytte/renter på grunnfondsbevis","account.data_account_type_expenses","","False","no_chart_template"
"chart8922","8922","Avsatt tilleggsutbytte","account.data_account_type_expenses","","False","no_chart_template"
"chart8923","8923","Avsatt ekstraordinært utbytte","account.data_account_type_expenses","","False","no_chart_template"
"chart8930","8930","Konsernbidrag","account.data_account_type_expenses","","False","no_chart_template"
"chart8940","8940","Aksjonærbidrag","account.data_account_type_expenses","","False","no_chart_template"
"chart8950","8950","Fondsemisjon","account.data_account_type_expenses","","False","no_chart_template"
"chart8960","8960","Overføringer annen egenkapital","account.data_account_type_expenses","","False","no_chart_template"
"chart8980","8980","Avsatt til fri egenkapital","account.data_account_type_expenses","","False","no_chart_template"
"chart8990","8990","Udekket tap","account.data_account_type_expenses","","False","no_chart_template"
"chart9970","9970","Expense account on product template","account.data_account_type_expenses","","False","no_chart_template"
"chart9990","9990","Income account on product template","account.data_account_type_revenue","","False","no_chart_template"

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="no_chart_template" model="account.chart.template">
            <field name="property_account_receivable_id" ref="chart1500"/>
            <field name="property_account_payable_id" ref="chart2400"/>
            <field name="property_account_expense_categ_id" ref="chart4000"/>
            <field name="property_account_income_categ_id" ref="chart3000"/>
            <field name="property_account_expense_id" ref="chart4300"/>
            <field name="property_account_income_id" ref="chart3000"/>
            <field name="income_currency_exchange_account_id" ref="chart8060"/>
            <field name="expense_currency_exchange_account_id" ref="chart8160"/>
            <field name="default_pos_receivable_account_id" ref="chart1501" />
        </record>
    </data>

    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_no.no_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="tax2" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">1 Inngående mva høy sats 25%</field>
            <field name="sequence">0</field>
            <field name="description">Fradrag for inngående mva</field>
            <field name="amount">25</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'plus_report_line_ids': [ref('tax_report_line_code_1')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'minus_report_line_ids': [ref('tax_report_line_code_1')],
                }),
            ]"/>
        </record>

       <record id="tax1" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">0 Ingen mvabehandling 0%</field>
            <field name="description">Ingen mvabehandling(anskaffelser)</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
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

        <record id="tax3" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">3 Utgående mva høy sats 25%</field>
            <field name="description">Utgående mva</field>
            <field name="amount">25</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_3')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2701'),
                    'plus_report_line_ids': [ref('tax_report_line_code_3_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_3')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2701'),
                    'minus_report_line_ids': [ref('tax_report_line_code_3_tax')],
                }),
            ]"/>
        </record>

        <record id="tax4" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">5 Mvafritt salg 0%</field>
            <field name="description">Mvafritt salg</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_5')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_5')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="tax5" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">6 Omsetning utenfor mvaloven 0%</field>
            <field name="description">Omsetning utenfor merverdiavgiftsloven</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_6')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_6')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="tax6" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">7 Ingen mvabehandling(inntekter) 0%</field>
            <field name="description">Ingen mvabehandling(inntekter)</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
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

        <record id="tax7" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">11 Inngående mva middel sats 15%</field>
            <field name="description">Fradrag for inngående mva</field>
            <field name="amount">15</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_15"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2713'),
                    'plus_report_line_ids': [ref('tax_report_line_code_11')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2713'),
                    'minus_report_line_ids': [ref('tax_report_line_code_11')],
                }),
            ]"/>
        </record>

        <record id="tax8" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">12 Inngående mva råfisk 11%</field>
            <field name="description">Fradrag for inngående mva</field>
            <field name="amount">11.11</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_15"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2710'),
                    'plus_report_line_ids': [ref('tax_report_line_code_12')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2710'),
                    'minus_report_line_ids': [ref('tax_report_line_code_12')],
                }),
            ]"/>
        </record>

        <record id="tax9" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">13 Inngående mva lav sats 12%</field>
            <field name="description">Fradrag for inngående mva</field>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2714'),
                    'plus_report_line_ids': [ref('tax_report_line_code_13')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2714'),
                    'minus_report_line_ids': [ref('tax_report_line_code_13')],
                }),
            ]"/>
        </record>

        <record id="tax10" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">14 Innførselsmva høy sats 25%</field>
            <field name="description">Fradrag for innførselsmva</field>
            <field name="amount">25</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'plus_report_line_ids': [ref('tax_report_line_code_14')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'minus_report_line_ids': [ref('tax_report_line_code_14')],
                }),
            ]"/>
        </record>

        <record id="tax11" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">15 Innførselsmva middel sats 15%</field>
            <field name="description">Fradrag for innførselsmva</field>
            <field name="amount">15</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_15"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2713'),
                    'plus_report_line_ids': [ref('tax_report_line_code_15')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2713'),
                    'minus_report_line_ids': [ref('tax_report_line_code_15')],
                }),
            ]"/>
        </record>

        <record id="tax12" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">20 Grunnlag ved innførsel av varer nullsats 0%</field>
            <field name="description">Grunnlag ved innførsel av varer</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
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

        <record id="tax13" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">21 Grunnlag ved innførsel av varer høy sats 25%</field>
            <field name="description">Grunnlag ved innførsel av varer</field>
            <field name="amount">25</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
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

        <record id="tax14" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">22 Grunnlag ved innførsel av varer middel sats 15%</field>
            <field name="description">Grunnlag ved innførsel av varer</field>
            <field name="amount">15</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_15"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
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

        <record id="tax15" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">31 Utgående mva middel sats 15%</field>
            <field name="description">Utgående mva</field>
            <field name="amount">15</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_15"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_31')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2703'),
                    'plus_report_line_ids': [ref('tax_report_line_code_31_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_31')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2703'),
                    'minus_report_line_ids': [ref('tax_report_line_code_31_tax')],
                }),
            ]"/>
        </record>

        <record id="tax16" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">32 Utgående mva råfisk 11%</field>
            <field name="description">Utgående mva</field>
            <field name="amount">11.11</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_15"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_32')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2700'),
                    'plus_report_line_ids': [ref('tax_report_line_code_32_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_32')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2700'),
                    'minus_report_line_ids': [ref('tax_report_line_code_32_tax')],
                }),
            ]"/>
        </record>

        <record id="tax17" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">33 Utgående mva lav sats 12%</field>
            <field name="description">Utgående mva</field>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_33')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2704'),
                    'plus_report_line_ids': [ref('tax_report_line_code_33_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_33')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2704'),
                    'minus_report_line_ids': [ref('tax_report_line_code_33_tax')],
                }),
            ]"/>
        </record>

        <record id="tax18" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">51 Salg av klimakvoter og gull til næringsdrivende 0%</field>
            <field name="description">Salg av klimakvoter og gull til næringsdrivende</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_51')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_51')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="tax19" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">52 Utførsel av varer og tjenester nullsats 0%</field>
            <field name="description">Utførsel av varer og tjenester</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_52')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_52')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="tax20" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">81 Innførsel av varer med fradrag for innførselsmva høy sats 25%</field>
            <field name="description">Innførsel av varer med fradrag for innførselsmva</field>
            <field name="amount">25</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_81')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2741'),
                    'plus_report_line_ids': [ref('tax_report_line_code_81_tax')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2727'),
                    'plus_report_line_ids': [ref('tax_report_line_code_81_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_81')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2741'),
                    'minus_report_line_ids': [ref('tax_report_line_code_81_tax')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2727'),
                    'minus_report_line_ids': [ref('tax_report_line_code_81_tax')],
                }),
            ]"/>
        </record>

        <record id="tax21" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">82 Innførsel av varer uten fradrag for innførselsmva høy sats 25%</field>
            <field name="description">Innførsel av varer uten fradrag for innførselsmva</field>
            <field name="amount">25</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_82')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2741'),
                    'plus_report_line_ids': [ref('tax_report_line_code_82_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_82')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2741'),
                    'minus_report_line_ids': [ref('tax_report_line_code_82_tax')],
                }),
            ]"/>
        </record>

        <record id="tax22" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">83 Innførsel av varer med fradrag for innførselsmva middel sats 15%</field>
            <field name="description">Innførsel av varer med fradrag for innførselsmva</field>
            <field name="amount">15</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_15"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_83')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2742'),
                    'plus_report_line_ids': [ref('tax_report_line_code_83_tax')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2728'),
                    'plus_report_line_ids': [ref('tax_report_line_code_83_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_83')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2742'),
                    'minus_report_line_ids': [ref('tax_report_line_code_83_tax')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2728'),
                    'minus_report_line_ids': [ref('tax_report_line_code_83_tax')],
                }),
            ]"/>
        </record>

        <record id="tax23" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">84 Innførsel av varer uten fradrag for innførselsmva middel sats 15%</field>
            <field name="description">Innførsel av varer uten fradrag for innførselsmva</field>
            <field name="amount">15</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_15"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_84')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2742'),
                    'plus_report_line_ids': [ref('tax_report_line_code_84_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_84')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2742'),
                    'minus_report_line_ids': [ref('tax_report_line_code_84_tax')],
                }),
            ]"/>
        </record>

        <record id="tax24" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">85 Innførsel av varer uten mva beregning 0%</field>
            <field name="description">Innførsel av varer som det ikke skal beregnes mervediavgift av</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_85')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_85')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <record id="tax25" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">86 Tjenester kjøpt fra utlandet med fradrag for mva høy sats 25%</field>
            <field name="description">Tjenester kjøpt fra utlandet med fradrag for mva</field>
            <field name="amount">25</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_86')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'plus_report_line_ids': [ref('tax_report_line_code_86_tax')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2702'),
                    'plus_report_line_ids': [ref('tax_report_line_code_86_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_86')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'minus_report_line_ids': [ref('tax_report_line_code_86_tax')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2702'),
                    'minus_report_line_ids': [ref('tax_report_line_code_86_tax')],
                }),
            ]"/>
        </record>

        <record id="tax26" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">87 Tjenester kjøpt fra utlandet uten fradrag for mva høy sats 25%</field>
            <field name="description">Tjenester kjøpt fra utlandet uten fradrag for mva</field>
            <field name="amount">25</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_87')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'plus_report_line_ids': [ref('tax_report_line_code_87_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_87')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'minus_report_line_ids': [ref('tax_report_line_code_87_tax')],
                }),
            ]"/>
        </record>

        <record id="tax27" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">88 Tjenester kjøpt fra utlandet med fradrag for mva lav sats 12%</field>
            <field name="description">Tjenester kjøpt fra utlandet med fradrag for mva</field>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_12"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_88')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'plus_report_line_ids': [ref('tax_report_line_code_88_tax')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2702'),
                    'plus_report_line_ids': [ref('tax_report_line_code_88_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_88')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'minus_report_line_ids': [ref('tax_report_line_code_88_tax')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2702'),
                    'minus_report_line_ids': [ref('tax_report_line_code_88_tax')],
                }),
            ]"/>
        </record>

        <record id="tax28" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">89 Tjenester kjøpt fra utlandet uten fradrag for mva lav sats 12%</field>
            <field name="description">Tjenester kjøpt fra utlandet uten fradrag for mva</field>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_12"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_89')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'plus_report_line_ids': [ref('tax_report_line_code_89_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_89')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'minus_report_line_ids': [ref('tax_report_line_code_89_tax')],
                }),
            ]"/>
        </record>

        <record id="tax29" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">91 Kjøp av klimakvoter eller gull med fradrag for mva høy sats 25%</field>
            <field name="description">Kjøp av klimakvoter eller gull med fradrag for mva</field>
            <field name="amount">25</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_91')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'plus_report_line_ids': [ref('tax_report_line_code_91_tax')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2701'),
                    'plus_report_line_ids': [ref('tax_report_line_code_91_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_91')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'minus_report_line_ids': [ref('tax_report_line_code_91_tax')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2701'),
                    'minus_report_line_ids': [ref('tax_report_line_code_91_tax')],
                }),
            ]"/>
        </record>

        <record id="tax30" model="account.tax.template">
            <field name="chart_template_id" ref="no_chart_template"/>
            <field name="name">92 Kjøp av klimakvoter eller gull uten fradrag for mva høy sats 25%</field>
            <field name="description">Kjøp av klimakvoter eller gull uten fradrag for mva</field>
            <field name="amount">25</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_code_92')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'plus_report_line_ids': [ref('tax_report_line_code_92_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_code_92')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'minus_report_line_ids': [ref('tax_report_line_code_92_tax')],
                }),
            ]"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_0" model="account.tax.group">
            <field name="name">MVA 0%</field>
            <field name="country_id" ref="base.no"/>
        </record>
        <record id="tax_group_10" model="account.tax.group">
            <field name="name">MVA 10%</field>
            <field name="country_id" ref="base.no"/>
        </record>
        <record id="tax_group_12" model="account.tax.group">
            <field name="name">MVA 12%</field>
            <field name="country_id" ref="base.no"/>
        </record>
        <record id="tax_group_15" model="account.tax.group">
            <field name="name">MVA 15%</field>
            <field name="country_id" ref="base.no"/>
        </record>
        <record id="tax_group_25" model="account.tax.group">
            <field name="name">MVA 25%</field>
            <field name="country_id" ref="base.no"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- Tax report for Norway (January 2022)
         Tax codes are related to the SAF-T codes  -->

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.no"/>
    </record>

    <record id="tax_report_line_sales_goods_services_homeland" model="account.tax.report.line">
        <!-- Sales of goods and services in Norway -->
        <field name="name">Salg av varer og tjenester i Norge</field>
        <field name="sequence" eval="0"/>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_line_code_3" model="account.tax.report.line">
        <!-- Sales and withdrawals of goods and services (high rate 25%) - base -->
        <field name="name">3 Salg og uttak av varer og tjenester (høy sats 25%) - grunnlag</field>
        <field name="tag_name">3 Base</field>
        <field name="code">BASE_3</field>
        <field name="sequence" eval="0"/>
        <field name="parent_id" ref="tax_report_line_sales_goods_services_homeland"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_3_tax" model="account.tax.report.line">
        <!-- Sales and withdrawals of goods and services (high rate 25%) - tax -->
        <field name="name">3 Salg og uttak av varer og tjenester (høy sats 25%) - avgift</field>
        <field name="tag_name">3 Tax</field>
        <field name="code">TAX_3</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_line_sales_goods_services_homeland"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_31" model="account.tax.report.line">
        <!-- Sales and withdrawals of goods and services (middle rate 15%) - base -->
        <field name="name">31 Salg og uttak av varer og tjenester (middels sats 15%) - grunnlag</field>
        <field name="tag_name">31 Base</field>
        <field name="code">BASE_31</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_line_sales_goods_services_homeland"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_31_tax" model="account.tax.report.line">
        <!-- Sales and withdrawals of goods and services (middle rate 15%) - tax -->
        <field name="name">31 Salg og uttak av varer og tjenester (middels sats 15%) - avgift</field>
        <field name="tag_name">31 Tax</field>
        <field name="code">TAX_31</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_line_sales_goods_services_homeland"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_33" model="account.tax.report.line">
        <!-- Sales and withdrawals of goods and services (low rate 12%) - base -->
        <field name="name">33 Salg og uttak av varer og tjenester (lav sats 12%) - grunnlag</field>
        <field name="tag_name">33 Base</field>
        <field name="code">BASE_33</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="tax_report_line_sales_goods_services_homeland"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_33_tax" model="account.tax.report.line">
        <!-- Sales and withdrawals of goods and services (low rate 12%) - tax -->
        <field name="name">33 Salg og uttak av varer og tjenester (lav sats 12%) - avgift</field>
        <field name="tag_name">33 Tax</field>
        <field name="code">TAX_33</field>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="tax_report_line_sales_goods_services_homeland"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_5" model="account.tax.report.line">
        <!-- Sales and withdrawals of goods and services that are exempt from VAT (0%) -->
        <field name="name">5 Salg og uttak av varer og tjenester som er fritatt for merverdiavgift (0%)</field>
        <field name="tag_name">5 Base</field>
        <field name="code">BASE_5</field>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="tax_report_line_sales_goods_services_homeland"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_6" model="account.tax.report.line">
        <!-- Sale of goods and services that are exempt from the VAT Act (0%) -->
        <field name="name">6 Salg av varer og tjenester som er unntatt merverdiavgiftsloven (0%)</field>
        <field name="tag_name">6 Base</field>
        <field name="code">BASE_6</field>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="tax_report_line_sales_goods_services_homeland"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_sales_goods_services_abroad" model="account.tax.report.line">
        <!-- Sales of goods and services to other countries -->
        <field name="name">Salg av varer og tjenester til utlandet</field>
        <field name="sequence" eval="2"/>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_line_code_52" model="account.tax.report.line">
        <!-- Sales of goods and services abroad that are exempt from VAT (0%) -->
        <field name="name">52 Salg av varer og tjenester til utlandet som er fritatt for merverdiavgift (0%)</field>
        <field name="tag_name">52 Base</field>
        <field name="code">BASE_52</field>
        <field name="sequence" eval="0"/>
        <field name="parent_id" ref="tax_report_line_sales_goods_services_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_purchases_goods_services_homeland" model="account.tax.report.line">
        <!-- Purchases of goods and services in Norway -->
        <field name="name">Kjøp av varer og tjenester i Norge</field>
        <field name="sequence" eval="3"/>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_line_code_1" model="account.tax.report.line">
        <!-- Purchase of goods and services with a right to deduct (high rate 25%) -->
        <field name="name">1 Kjøp av varer og tjenester med fradragsrett (høy sats 25%)</field>
        <field name="tag_name">1 Tax</field>
        <field name="code">TAX_1</field>
        <field name="sequence" eval="0"/>
        <field name="parent_id" ref="tax_report_line_purchases_goods_services_homeland"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_11" model="account.tax.report.line">
        <!-- Purchase of goods and services with a right to deduct (middle rate 15%) -->
        <field name="name">11 Kjøp av varer og tjenester med fradragsrett (middels sats 15%)</field>
        <field name="tag_name">11 Tax</field>
        <field name="code">TAX_11</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_line_purchases_goods_services_homeland"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_13" model="account.tax.report.line">
        <!-- Purchase of goods and services with a right to deduct (low rate 12%) -->
        <field name="name">13 Kjøp av varer og tjenester med fradragsrett (lav sats 12%)</field>
        <field name="tag_name">13 Tax</field>
        <field name="code">TAX_13</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_line_purchases_goods_services_homeland"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_purchases_goods_abroad" model="account.tax.report.line">
        <!-- Purchases of goods from abroad (import) -->
        <field name="name">Kjøp av varer fra utlandet (import)</field>
        <field name="sequence" eval="4"/>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_line_code_14" model="account.tax.report.line">
        <!-- Deduction on purchases of goods from abroad (VAT paid on import, high rate 25%) -->
        <field name="name">14 Fradrag på kjøp av varer fra utlandet (merverdiavgift betalt ved innførsel, høy sats 25%)</field>
        <field name="tag_name">14 Tax</field>
        <field name="code">TAX_14</field>
        <field name="sequence" eval="100"/>
        <field name="parent_id" ref="tax_report_line_purchases_goods_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_15" model="account.tax.report.line">
        <!-- Deduction on purchases of goods from abroad (VAT paid on import, middle rate 15%) -->
        <field name="name">15 Fradrag på kjøp av varer fra utlandet (merverdiavgift betalt ved innførsel, middels sats 15%)</field>
        <field name="tag_name">15 Tax</field>
        <field name="code">TAX_15</field>
        <field name="sequence" eval="200"/>
        <field name="parent_id" ref="tax_report_line_purchases_goods_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_81" model="account.tax.report.line">
        <!-- Purchase of goods from abroad with a right to deduct (high rate 25%) - base -->
        <field name="name">81 Kjøp av varer fra utlandet med fradragsrett (høy sats 25%) - grunnlag</field>
        <field name="tag_name">81 Base</field>
        <field name="code">BASE_81</field>
        <field name="sequence" eval="0"/>
        <field name="parent_id" ref="tax_report_line_purchases_goods_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_81_tax" model="account.tax.report.line">
        <!-- Purchase of goods from abroad with a right to deduct (high rate 25%) - tax -->
        <field name="name">81 Kjøp av varer fra utlandet med fradragsrett (høy sats 25%) - avgift</field>
        <field name="tag_name">81 Tax</field>
        <field name="code">TAX_81</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_line_purchases_goods_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_82" model="account.tax.report.line">
        <!-- Purchase of goods from abroad without the right to deduct (high rate 25%) - base -->
        <field name="name">82 Kjøp av varer fra utlandet uten fradragsrett (høy sats 25%) - grunnlag</field>
        <field name="tag_name">82 Base</field>
        <field name="code">BASE_82</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_line_purchases_goods_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_82_tax" model="account.tax.report.line">
        <!-- Purchase of goods from abroad without the right to deduct (high rate 25%) - tax -->
        <field name="name">82 Kjøp av varer fra utlandet uten fradragsrett (høy sats 25%) - avgift</field>
        <field name="tag_name">82 Tax</field>
        <field name="code">TAX_82</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_line_purchases_goods_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_83" model="account.tax.report.line">
        <!-- Purchase of goods from abroad with a right to deduct (middle rate 15%) - base -->
        <field name="name">83 Kjøp av varer fra utlandet med fradragsrett (middels sats 15%) - grunnlag</field>
        <field name="tag_name">83 Base</field>
        <field name="code">BASE_83</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="tax_report_line_purchases_goods_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_83_tax" model="account.tax.report.line">
        <!-- Purchase of goods from abroad with a right to deduct (middle rate 15%) - tax -->
        <field name="name">83 Kjøp av varer fra utlandet med fradragsrett (middels sats 15%) - avgift</field>
        <field name="tag_name">83 Tax</field>
        <field name="code">TAX_83</field>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="tax_report_line_purchases_goods_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_84" model="account.tax.report.line">
        <!-- Purchase of goods from abroad without the right to deduct (middle rate 15%) - base -->
        <field name="name">84 Kjøp av varer fra utlandet som er uten fradragsrett (middels sats 15%) - grunnlag</field>
        <field name="tag_name">84 Base</field>
        <field name="code">BASE_84</field>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="tax_report_line_purchases_goods_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_84_tax" model="account.tax.report.line">
        <!-- Purchase of goods from abroad without the right to deduct (middle rate 15%) - tax -->
        <field name="name">84 Kjøp av varer fra utlandet som er uten fradragsrett (middels sats 15%) - avgift</field>
        <field name="tag_name">84 Tax</field>
        <field name="code">TAX_84</field>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="tax_report_line_purchases_goods_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_85" model="account.tax.report.line">
        <!-- Purchase of goods from abroad for which VAT is not to be calculated (zero rate 0%) -->
        <field name="name">85 Kjøp av varer fra utlandet som det ikke skal beregnes merverdiavgift på (nullsats 0%)</field>
        <field name="tag_name">85 Base</field>
        <field name="code">BASE_85</field>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="tax_report_line_purchases_goods_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_purchases_services_abroad" model="account.tax.report.line">
        <!-- Purchases of services from abroad (import) -->
        <field name="name">Kjøp av tjenester fra utlandet (import)</field>
        <field name="sequence" eval="5"/>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_line_code_86" model="account.tax.report.line">
        <!-- Purchase of services from abroad with a right to deduct (high rate 25%) - base -->
        <field name="name">86 Kjøp av tjenester fra utlandet med fradragsrett (høy sats 25%) - grunnlag</field>
        <field name="tag_name">86 Base</field>
        <field name="code">BASE_86</field>
        <field name="sequence" eval="0"/>
        <field name="parent_id" ref="tax_report_line_purchases_services_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_86_tax" model="account.tax.report.line">
        <!-- Purchase of services from abroad with a right to deduct (high rate 25%) - tax -->
        <field name="name">86 Kjøp av tjenester fra utlandet med fradragsrett (høy sats 25%) - avgift</field>
        <field name="tag_name">86 Tax</field>
        <field name="code">TAX_86</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_line_purchases_services_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_87" model="account.tax.report.line">
        <!-- Purchase of services from abroad without the right to deduct (high rate 25%) - base -->
        <field name="name">87 Kjøp av tjenester fra utlandet uten fradragsrett (høy sats 25%) - grunnlag</field>
        <field name="tag_name">87 Base</field>
        <field name="code">BASE_87</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_line_purchases_services_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_87_tax" model="account.tax.report.line">
        <!-- Purchase of services from abroad without the right to deduct (high rate 25%) - tax -->
        <field name="name">87 Kjøp av tjenester fra utlandet uten fradragsrett (høy sats 25%) - avgift</field>
        <field name="tag_name">87 Tax</field>
        <field name="code">TAX_87</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_line_purchases_services_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_88" model="account.tax.report.line">
        <!-- Purchase of services from abroad with a right to deduct (low rate 12%) - base -->
        <field name="name">88 Kjøp av tjenester fra utlandet med fradragsrett (lav sats 12%) - grunnlag</field>
        <field name="tag_name">88 Base</field>
        <field name="code">BASE_88</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="tax_report_line_purchases_services_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_88_tax" model="account.tax.report.line">
        <!-- Purchase of services from abroad with a right to deduct (low rate 12%) - tax -->
        <field name="name">88 Kjøp av tjenester fra utlandet med fradragsrett (lav sats 12%) - avgift</field>
        <field name="tag_name">88 Tax</field>
        <field name="code">TAX_88</field>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="tax_report_line_purchases_services_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_89" model="account.tax.report.line">
        <!-- Purchase of services from abroad without the right to deduct (low rate 12%) - base -->
        <field name="name">89 Kjøp av tjenester fra utlandet uten fradragsrett (lav sats 12%) - grunnlag</field>
        <field name="tag_name">89 Base</field>
        <field name="code">BASE_89</field>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="tax_report_line_purchases_services_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_89_tax" model="account.tax.report.line">
        <!-- Purchase of services from abroad without the right to deduct (low rate 12%) - tax -->
        <field name="name">89 Kjøp av tjenester fra utlandet uten fradragsrett (lav sats 12%) - avgift</field>
        <field name="tag_name">89 Tax</field>
        <field name="code">TAX_89</field>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="tax_report_line_purchases_services_abroad"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_fish_etc" model="account.tax.report.line">
        <!-- Fish etc. -->
        <field name="name">Fisk mv.</field>
        <field name="sequence" eval="6"/>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_line_code_12" model="account.tax.report.line">
        <!-- Purchase of fish and other marine wildlife resources (11.11%) -->
        <field name="name">12 Kjøp av fisk og andre marine viltlevende ressurser (11,11%)</field>
        <field name="tag_name">12 Tax</field>
        <field name="code">TAX_12</field>
        <field name="sequence" eval="0"/>
        <field name="parent_id" ref="tax_report_line_fish_etc"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_32" model="account.tax.report.line">
        <!-- Sales of fish and other marine wildlife resources (11.11%) - base -->
        <field name="name">32 Salg av fisk og andre marine viltlevende ressurser (11,11%) - grunnlag</field>
        <field name="tag_name">32 Base</field>
        <field name="code">BASE_32</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_line_fish_etc"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_32_tax" model="account.tax.report.line">
        <!-- Sales of fish and other marine wildlife resources (11.11%) - tax -->
        <field name="name">32 Salg av fisk og andre marine viltlevende ressurser (11,11%) - avgift</field>
        <field name="tag_name">32 Tax</field>
        <field name="code">TAX_32</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_line_fish_etc"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_emission_and_gold" model="account.tax.report.line">
        <!-- Emission allowances and gold -->
        <field name="name">Klimakvoter og gull</field>
        <field name="sequence" eval="7"/>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_line_code_51" model="account.tax.report.line">
        <!-- Sale of climate quotas and gold to businesses (0%) -->
        <field name="name">51 Salg av klimakvoter og gull til næringsdrivende (0%)</field>
        <field name="tag_name">51 Base</field>
        <field name="code">BASE_51</field>
        <field name="sequence" eval="0"/>
        <field name="parent_id" ref="tax_report_line_emission_and_gold"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_91" model="account.tax.report.line">
        <!-- Purchase of climate quotas and gold with a right to deduct (high rate 25%) - base -->
        <field name="name">91 Kjøp av klimakvoter og gull med fradragsrett (høy sats 25%) - grunnlag</field>
        <field name="tag_name">91 Base</field>
        <field name="code">BASE_91</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_line_emission_and_gold"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_91_tax" model="account.tax.report.line">
        <!-- Purchase of climate quotas and gold with a right to deduct (high rate 25%) - tax -->
        <field name="name">91 Kjøp av klimakvoter og gull med fradragsrett (høy sats 25%) - avgift</field>
        <field name="tag_name">91 Tax</field>
        <field name="code">TAX_91</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_line_emission_and_gold"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_92" model="account.tax.report.line">
        <!-- Purchase of climate quotas and gold without the right to deduct (high rate 25%) - base -->
        <field name="name">92 Kjøp av klimakvoter og gull uten fradragsrett (høy sats 25%) - grunnlag</field>
        <field name="tag_name">92 Base</field>
        <field name="code">BASE_92</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_line_emission_and_gold"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_code_92_tax" model="account.tax.report.line">
        <!-- Purchase of climate quotas and gold without the right to deduct (high rate 25%) - base -->
        <field name="name">92 Kjøp av klimakvoter og gull uten fradragsrett (høy sats 25%) - avgift</field>
        <field name="tag_name">92 Tax</field>
        <field name="code">TAX_92</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="tax_report_line_emission_and_gold"/>
        <field name="report_id" ref="tax_report"/>
    </record>

    <record id="tax_report_line_sum" model="account.tax.report.line">
        <field name="name">Sum</field>
        <field name="sequence" eval="8"/>
        <field name="report_id" ref="tax_report"/>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_line_to_be_paid" model="account.tax.report.line">
        <!-- Tax to be paid -->
        <field name="name">Avgift å betale</field>
        <field name="formula">TAX_3+TAX_31+TAX_32+TAX_33+TAX_81+TAX_82+TAX_83+TAX_84+TAX_86+TAX_87+TAX_88+TAX_89+TAX_91+TAX_92-TAX_1-TAX_11-TAX_12-TAX_13-TAX_14-TAX_15</field>
        <field name="code">SUM</field>
        <field name="sequence" eval="0"/>
        <field name="parent_id" ref="tax_report_line_sum"/>
        <field name="report_id" ref="tax_report"/>
    </record>

</odoo>

```

## File: data\l10n_no_chart_data.xml

```xml
﻿<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_no_statements_menu" name="Norway" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>

    <data>

	<menuitem id="account_reports_no_statements_menu" name="Norwegian Statements" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>

    <!-- COA Templates -->
    <record id="no_chart_template" model="account.chart.template">
        <field name="name">Norway's Chart of Accounts</field>
        <field name="cash_account_code_prefix">1900</field>
        <field name="bank_account_code_prefix">1920</field>
        <field name="transfer_account_code_prefix">1940</field>
        <field name="code_digits">4</field>
        <field name="currency_id" ref="base.NOK"/>
        <field name="country_id" ref="base.no"/>
    </record>

    </data>
</odoo>

```

## File: migrations\2.1\post-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.addons.account.models.chart_template import update_taxes_from_templates


def migrate(cr, version):
    update_taxes_from_templates(cr, 'l10n_no.no_chart_template')

```

## File: migrations\9.0.2.0\pre-set_tags_and_taxes_updatable.py

```python
from openerp.modules.registry import RegistryManager


def migrate(cr, version):
    registry = RegistryManager.get(cr.dbname)
    from openerp.addons.account.models.chart_template import migrate_set_tags_and_taxes_updatable
    migrate_set_tags_and_taxes_updatable(cr, registry, 'l10n_no')

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import fields, models


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    invoice_reference_model = fields.Selection(selection_add=[
        ('no', 'Norway')
    ], ondelete={'no': lambda recs: recs.write({'invoice_reference_model': 'odoo'})})

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models
from stdnum import luhn


class AccountMove(models.Model):
    _inherit = "account.move"

    def _get_invoice_reference_no_invoice(self):
        """ This computes the reference based on the Odoo format.
            We calculat reference using invoice number and
            partner id and added control digit at last.
        """
        return self._get_kid_number()

    def _get_invoice_reference_no_partner(self):
        """ This computes the reference based on the Odoo format.
            We calculat reference using invoice number and
            partner id and added control digit at last.
        """
        return self._get_kid_number()

    def _get_kid_number(self):
        self.ensure_one()
        invoice_name = ''.join([i for i in self.name if i.isdigit()]).zfill(7)
        ref = (str(self.partner_id.id).zfill(7)[-7:] + invoice_name[-7:])
        return ref + luhn.calc_check_digit(ref)

```

## File: models\res_company.py

```python
# coding: utf-8
from odoo import fields, models


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_no_bronnoysund_number = fields.Char(related='partner_id.l10n_no_bronnoysund_number', readonly=False)

```

## File: models\res_partner.py

```python
# coding: utf-8
from odoo import fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_no_bronnoysund_number = fields.Char(string='Register of Legal Entities (Brønnøysund Register Center)', size=9)

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_journal
from . import account_move
from . import res_partner
from . import res_company

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
      <image width="1100" height="800" transform="translate(4.8 7.25) scale(0.05 0.04)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABEwAAALGCAYAAABf+gQ+AAAACXBIWXMAAPGaAADxmgGrOWmbAAASA0lEQVR4Xu3bsW1bMRhGUb5AA6SJB4gmcZldonSpDa8QDeNag2gBV96A6S8CMIXwYEHn1F//AxfkNse3OQDuwXxfLe7CdjytJuxoXs+ryX3YnlYLAAD+3+XLagEAAADwaAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAAiMN4+b3aAACfnXsOAHBT25xzrkYA3M52PK0m7Ghez6sJAACP5+JLDgAAAEAIJgAAAAAhmAAAAACEYAIAAAAQggkAAABACCYAAAAAIZgAAAAAhGACAAAAEIIJAAAAQAgmAAAAACGYAAAAAIRgAgAAABCCCQAAAEAIJgAAAAAhmAAAAACEYAIAAAAQggkAAABACCYAAAAAIZgAAAAAhGACAAAAEIIJAAAAQAgmAAAAACGYAAAAAIRgAgAAABCCCQAAAEAIJgAAAAAhmAAAAACEYAIAAAAQggkAAABACCYAAAAAIZgAAAAAhGACAAAAEIIJAAAAQAgmAAAAACGYAAAAAIRgAgAAABCCCQAAAEAIJgAAAAAhmAAAAACEYAIAAAAQggkAAABACCYAAAAAIZgAAAAAxDa+/5yrEQAAAMADuXhhAgAAABCCCQAAAEAIJgAAAAAhmAAAAACEYAIAAAAQggkAAABACCYAAAAAIZgAAAAAhGACAAAAEIIJAAAAQAgmAAAAACGYAAAAAIRgAgAAABCCCQAAAEAIJgAAAAAhmAAAAACEYAIAAAAQggkAAABACCYAAAAAIZgAAAAAhGACAAAAEIIJAAAAQAgmAAAAACGYAAAAAIRgAgAAABCCCQAAAEAIJgAAAAAhmAAAAACEYAIAAAAQggkAAABACCYAAAAAIZgAAAAAhGACAAAAEIIJAAAAQAgmAAAAACGYAAAAAIRgAgAAABCCCQAAAEAIJgAAAAAhmAAAAACEYAIAAAAQggkAAABACCYAAAAAIZgAAAAAhGACAAAAEIIJAAAAQAgmAAAAACGYAAAAAIRgAgAAABCCCQAAAEAIJgAAAAAhmAAAAACEYAIAAAAQggkAAABACCYAAAAAIZgAAAAAhGACAAAAEIIJAAAAQAgmAAAAACGYAAAAAIRgAgAAABCCCQAAAEAIJgAAAAAhmAAAAACEYAIAAAAQggkAAABACCYAAAAAIZgAAAAAhGACAAAAEIIJAAAAQAgmAAAAACGYAAAAAIRgAgAAABCCCQAAAEAIJgAAAAAhmAAAAACEYAIAAAAQggkAAABACCYAAAAAIZgAAAAAhGACAAAAEIIJAAAAQAgmAAAAACGYAAAAAIRgAgAAABCCCQAAAEAIJgAAAABxePn1Y7UB4IZe/7ytJuzIHQQA4F+2OedcjQC4ne14Wk3Y0byeVxMAAB7PxZccAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAACIw9ieVhuAz2G+rxbwuNxzAICb8sIEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIAQTAAAAgBBMAAAAAEIwAQAAAAjBBAAAACAEEwAAAIAQTAAAAABCMAEAAAAIwQQAAAAgBBMAAACAEEwAAAAAQjABAAAACMEEAAAAIA5jjOfVCABgBx9jjK+rEQDADj7+AvZ8KZOr8M+JAAAAAElFTkSuQmCC"/>
    </g>
  </g>
</svg>

```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_company_form_inherit_no" model="ir.ui.view">
            <field name="name">res.company.form.inherit.l10n.no</field>
            <field name="model">res.company</field>
            <field name="inherit_id" ref="base.view_company_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='vat']" position="after">
                    <field name="l10n_no_bronnoysund_number" attrs="{'invisible': [('country_code', '!=', 'NO')]}"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\res_partner_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="view_partner_form_inherit_l10n_no" model="ir.ui.view">
        <field name="name">res.partner.form.inherit.l10n_no</field>
        <field name="model">res.partner</field>
        <field name="inherit_id" ref="base.view_partner_form"/>
        <field name="arch" type="xml">
            <xpath expr="//field[@name='vat']" position="after">
                <field name="l10n_no_bronnoysund_number" attrs="{'invisible': ['|', ('country_code', '!=', 'NO'), ('is_company', '=', False)]}"/>
            </xpath>
        </field>
    </record>
</odoo>

```

