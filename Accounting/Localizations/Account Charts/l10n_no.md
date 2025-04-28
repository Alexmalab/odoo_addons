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
"id","code","name","account_type","tag_ids/id","reconcile","chart_template_id/id"
"no_a_cash","1000","Forskning og utvikling","asset_current","","False","no_chart_template"
"chart1010","1010","Forskning og utvikling, egenutviklet","asset_current","","False","no_chart_template"
"chart1020","1020","Konsesjoner","asset_current","","False","no_chart_template"
"chart1030","1030","Patenter","asset_current","","False","no_chart_template"
"chart1040","1040","Lisenser","asset_current","","False","no_chart_template"
"chart1050","1050","Varemerker","asset_current","","False","no_chart_template"
"chart1060","1060","Andre rettigheter","asset_current","","False","no_chart_template"
"chart1070","1070","Utsatt skattefordel","asset_current","","False","no_chart_template"
"chart1080","1080","Goodwill","asset_current","","False","no_chart_template"
"chart1100","1100","Bygninger","asset_current","","False","no_chart_template"
"chart1120","1120","Bygningsmessige anlegg","asset_current","","False","no_chart_template"
"chart1130","1130","Anlegg under utførelse","asset_current","","False","no_chart_template"
"chart1140","1140","Jord- og skogbrukseiendommer","asset_current","","False","no_chart_template"
"chart1150","1150","Tomter og andre grunnarealer","asset_current","","False","no_chart_template"
"chart1160","1160","Boliger inklusive tomter","asset_current","","False","no_chart_template"
"chart1190","1190","Andre anleggsmidler","asset_current","","False","no_chart_template"
"chart1200","1200","Maskiner og anlegg","asset_current","","False","no_chart_template"
"chart1210","1210","Maskiner og anlegg under utførelse","asset_current","","False","no_chart_template"
"chart1220","1220","Skip, rigger, fly","asset_current","","False","no_chart_template"
"chart1230","1230","Biler","asset_current","","False","no_chart_template"
"chart1239","1239","Vogntog, lastebiler og busser","asset_current","","False","no_chart_template"
"chart1240","1240","Andre transportmidler","asset_current","","False","no_chart_template"
"chart1249","1249","Andre transportmidler","asset_current","","False","no_chart_template"
"chart1250","1250","Inventar","asset_current","","False","no_chart_template"
"chart1260","1260","Fast bygningsinventar med annen avskrivning","asset_current","","False","no_chart_template"
"chart1270","1270","Verktøy mv.","asset_current","","False","no_chart_template"
"chart1280","1280","Kontormaskiner","asset_current","","False","no_chart_template"
"chart1290","1290","Andre driftsmidler","asset_current","","False","no_chart_template"
"chart1300","1300","Investeringer i datterselskaper","asset_current","","False","no_chart_template"
"chart1310","1310","Investeringer annet foretak i samme konsern","asset_current","","False","no_chart_template"
"chart1320","1320","Lån til foretak samme konsern","asset_current","","False","no_chart_template"
"chart1330","1330","Investeringer i tilknyttede selskap","asset_current","","False","no_chart_template"
"chart1340","1340","Lån til tilknyttede selskap","asset_current","","False","no_chart_template"
"chart1350","1350","Investeringer i aksjer og andeler","asset_current","","False","no_chart_template"
"chart1360","1360","Obligasjoner","asset_current","","False","no_chart_template"
"chart1370","1370","Fordringer på eiere, styremedlemmer mv.","asset_current","","False","no_chart_template"
"chart1380","1380","Fordringer på ansatte","asset_current","","False","no_chart_template"
"chart1390","1390","Andre langsiktige fordringer","asset_current","","False","no_chart_template"
"chart1399","1399","Andre fordringer","asset_current","","False","no_chart_template"
"chart1400","1400","Råvarer og innkjøpte halvfabrikata","asset_current","","False","no_chart_template"
"chart1401","1401","Halvfabrikata","asset_current","","False","no_chart_template"
"chart1420","1420","Varer under tilvirkning","asset_current","","False","no_chart_template"
"chart1440","1440","Ferdige egentilvirkede varer","asset_current","","False","no_chart_template"
"chart1460","1460","Innkjøpte varer for videresalg","asset_current","","False","no_chart_template"
"chart1480","1480","Forskuddsbetaling til leverandører","asset_current","","False","no_chart_template"
"chart1481","1481","Kortsiktige fordringer hos leverandører","asset_current","","False","no_chart_template"
"chart1500","1500","Kundefordringer","asset_receivable","","True","no_chart_template"
"chart1501","1501","Kundefordringer (PoS)","asset_receivable","","True","no_chart_template"
"chart1509","1509","Ikke reskontroførte kundefordringer","asset_current","","False","no_chart_template"
"chart1513","1513","Ikke fakturerte kundefordringer","asset_current","","False","no_chart_template"
"chart1514","1514","Kundfordringer - tjenester","asset_current","","False","no_chart_template"
"chart1520","1520","Andre kortsiktige fordringer","asset_current","","False","no_chart_template"
"chart1530","1530","Opptjente, ikke fakturerte driftsinntekter","asset_current","","False","no_chart_template"
"chart1550","1550","Kundefordringer på selskap samme konsern","asset_current","","False","no_chart_template"
"chart1560","1560","Andre fordringer på selskap innen samme konsern","asset_current","","False","no_chart_template"
"chart1570","1570","Andre kortsiktige fordringer","asset_current","","False","no_chart_template"
"chart1571","1571","Lønnsforskudd","asset_current","","False","no_chart_template"
"chart1572","1572","Andre kortsiktige lån til ansatte","asset_current","","False","no_chart_template"
"chart1579","1579","Andre kortsiktige fordringer","asset_current","","False","no_chart_template"
"chart1580","1580","Avsetning tap på fordringer","asset_current","","False","no_chart_template"
"chart1600","1600","Utgående merverdiavgift","asset_current","","False","no_chart_template"
"chart1601","1601","Utgående merverdiavgift høy sats","asset_current","","False","no_chart_template"
"chart1602","1602","Utgående merverdiavgift kjøp tjen. fra utlandet","asset_current","","False","no_chart_template"
"chart1603","1603","Utgående merverdiavgift middels sats","asset_current","","False","no_chart_template"
"chart1604","1604","Utgående merverdiavgift lav sats","asset_current","","False","no_chart_template"
"chart1605","1605","Grunnlag utgående merverdiavgift, høy sats","asset_current","","False","no_chart_template"
"chart1606","1606","Grunnlag utgående merverdiavgift, middels sats","asset_current","","False","no_chart_template"
"chart1607","1607","Grunnlag utgående merverdiavgift, lav sats","asset_current","","False","no_chart_template"
"chart1610","1610","Inngående merverdiavgift","asset_current","","False","no_chart_template"
"chart1611","1611","Inngående merverdiavgift høy sats","asset_current","","False","no_chart_template"
"chart1612","1612","Inngående merverdiavgift kjøp tjen. fra utlandet","asset_current","","False","no_chart_template"
"chart1613","1613","Inngående merverdiavgift middels sats","asset_current","","False","no_chart_template"
"chart1614","1614","Inngående merverdiavgift lav sats","asset_current","","False","no_chart_template"
"chart1640","1640","Oppgjørskonto merverdiavgift","asset_current","","False","no_chart_template"
"chart1670","1670","Krav på offentlige tilskudd","asset_current","","False","no_chart_template"
"chart1700","1700","Forskuddsbetalt leie","asset_current","","False","no_chart_template"
"chart1710","1710","Forskuddsbetalt rente","asset_current","","False","no_chart_template"
"chart1720","1720","Forskuddsbetalt lønn","asset_current","","False","no_chart_template"
"chart1740","1740","Forskuddsbetalt, ikke påløpt lønn","asset_current","","False","no_chart_template"
"chart1749","1749","Andre forskuddbetalte kostnader","asset_current","","False","no_chart_template"
"chart1750","1750","Påløpte leieinntekter","asset_current","","False","no_chart_template"
"chart1760","1760","Påløpte renteinntekter","asset_current","","False","no_chart_template"
"chart1780","1780","Krav på innbetaling av selskapskapital","asset_current","","False","no_chart_template"
"chart1790","1790","Interimskonto","asset_current","","False","no_chart_template"
"chart1800","1800","Aksjer & andeler i foretak i samme kons.","asset_current","","False","no_chart_template"
"chart1810","1810","Markesdbaserte aksjer","asset_current","","False","no_chart_template"
"chart1820","1820","Andre aksjer","asset_current","","False","no_chart_template"
"chart1830","1830","Markedsbaserte obligasjoner","asset_current","","False","no_chart_template"
"chart1840","1840","Andre obligasjoner","asset_current","","False","no_chart_template"
"chart1850","1850","Markedsbaserte sertifikater","asset_current","","False","no_chart_template"
"chart1860","1860","Andre sertifikater","asset_current","","False","no_chart_template"
"chart1870","1870","Andre markedsbaserte finansielle instrumenter","asset_current","account.account_tag_financing","False","no_chart_template"
"chart1880","1880","Andre finansielle instrumenter","asset_current","account.account_tag_financing","False","no_chart_template"
"chart1890","1890","Andeler utenfor konsern","asset_current","","False","no_chart_template"
"chart1905","1905","Kontanter Euro","asset_current","","False","no_chart_template"
"chart1908","1908","Kontanter, annen valuta","asset_current","","False","no_chart_template"
"chart1910","1910","Kasse","asset_current","","False","no_chart_template"
"chart1918","1918","Kassedifferenser","asset_current","","False","no_chart_template"
"chart1919","1919","Kasse i transfer","asset_current","","False","no_chart_template"
"chart1921","1921","Bankinnskudd 2","asset_current","","False","no_chart_template"
"chart1942","1942","Bank ikke allokert innbetaling","asset_current","","False","no_chart_template"
"chart1944","1944","Bank ikke identifisert innbetaling","asset_current","","False","no_chart_template"
"chart1950","1950","Bankinnskudd for skattetrekk","asset_current","","False","no_chart_template"
"chart1980","1980","Valutakonto Euro","asset_current","","False","no_chart_template"
"chart2000","2000","Aksjekapital","equity","","False","no_chart_template"
"chart2010","2010","Egne aksjer","equity","","False","no_chart_template"
"chart2020","2020","Overkursfond","equity","","False","no_chart_template"
"chart2030","2030","Annen innskutt egenkapital","equity","","False","no_chart_template"
"chart2036","2036","Stiftelsesutgifter","equity","","False","no_chart_template"
"chart2040","2040","Fond for vurderingsforskjeller","equity","","False","no_chart_template"
"chart2050","2050","Annen egenkapital","equity","","False","no_chart_template"
"chart2055","2055","Innskudd kontanter","equity","","False","no_chart_template"
"chart2060","2060","Privatuttak","equity","","False","no_chart_template"
"chart2070","2070","Forskuddsskatt","equity","","False","no_chart_template"
"chart2080","2080","Udekket tap","equity","","False","no_chart_template"
"chart2098","2090","Gevinst eller tap fra foregående år","equity","","False","no_chart_template"
"chart2100","2100","Pensjonsforpliktelser","liability_current","","False","no_chart_template"
"chart2120","2120","Utsatt skatt","liability_current","","False","no_chart_template"
"chart2140","2140","Avsetn. for garanti- & serviceforpl.","liability_current","","False","no_chart_template"
"chart2160","2160","Uopptjent inntekt","liability_current","","False","no_chart_template"
"chart2180","2180","Andre avsetninger for forpiktelser","liability_current","","False","no_chart_template"
"chart2188","2188","Avsetning for skyldig merverdiavgift","liability_current","","False","no_chart_template"
"chart2200","2200","Konvertible lån","liability_current","","False","no_chart_template"
"chart2210","2210","Obligsjonslån","liability_current","","False","no_chart_template"
"chart2220","2220","Gjeld til kredittinstitusjoner","liability_current","","False","no_chart_template"
"chart2240","2240","Pantelån","liability_current","","False","no_chart_template"
"chart2251","2251","Gjeld til ansatte","liability_current","","False","no_chart_template"
"chart2255","2255","Gjeld til eiere","liability_current","","False","no_chart_template"
"chart2260","2260","Gjeld til selskap i samme konsern","liability_current","","False","no_chart_template"
"chart2270","2270","Andre valutalån","liability_current","","False","no_chart_template"
"chart2280","2280","Stille interessentinnskudd og ansv. lånekapital","liability_current","","False","no_chart_template"
"chart2299","2299","Annen langsiktig gjeld","liability_current","","False","no_chart_template"
"chart2300","2300","Konvertible lån","liability_current","","False","no_chart_template"
"chart2320","2320","Sertifikatlån","liability_current","","False","no_chart_template"
"chart2340","2340","Andre valutalån","liability_current","","False","no_chart_template"
"chart2360","2360","Byggelån","liability_current","","False","no_chart_template"
"chart2380","2380","Kassakreditt","liability_current","","True","no_chart_template"
"chart2390","2390","Annen gjeld til kredittinstitusjon","liability_current","","False","no_chart_template"
"chart2400","2400","Leverandørgjeld","liability_payable","","True","no_chart_template"
"chart2409","2409","Ikke reskontroført leverandørgjeld","liability_current","","False","no_chart_template"
"chart2441","2441","Leverandørgjeld - tjenester","liability_payable","","True","no_chart_template"
"chart2442","2442","Ikke fakturerte mottatte varer","liability_current","","False","no_chart_template"
"chart2460","2460","Leverandørgjeld til selskap i samme konsern","liability_payable","","True","no_chart_template"
"chart2490","2490","Påløpt lev.gjeld, mottatte varer","liability_current","","False","no_chart_template"
"chart2491","2491","Vekselgjeld","liability_current","","False","no_chart_template"
"chart2500","2500","Avsatt betalbar skatt, ikke utlignet","liability_current","","False","no_chart_template"
"chart2510","2510","Betalbar skatt, utlignet","liability_current","","False","no_chart_template"
"chart2530","2530","Refusjon skatt etter Skatteloven §31 5. ledd","liability_current","","False","no_chart_template"
"chart2540","2540","Forhåndsskatt","liability_current","","False","no_chart_template"
"chart2600","2600","Forskuddstrekk","liability_current","","False","no_chart_template"
"chart2610","2610","Påleggstrekk","liability_current","","False","no_chart_template"
"chart2620","2620","Bidragstrekk","liability_current","","False","no_chart_template"
"chart2630","2630","Trygdetrekk","liability_current","","False","no_chart_template"
"chart2640","2640","Forsikringstrekk","liability_current","","False","no_chart_template"
"chart2650","2650","Trukket fagforeningskontigent","liability_current","","False","no_chart_template"
"chart2690","2690","Andre trekk","liability_current","","False","no_chart_template"
"chart2700","2700","Utgående merverdiavgift","liability_current","","False","no_chart_template"
"chart2701","2701","Utgående merverdiavgift høy sats","liability_current","","False","no_chart_template"
"chart2702","2702","Utgående merverdiavgift kjøp tjen. fra utlandet","liability_current","","False","no_chart_template"
"chart2703","2703","Utgående merverdiavgift middels sats","liability_current","","False","no_chart_template"
"chart2704","2704","Utgående merverdiavgift lav sats","liability_current","","False","no_chart_template"
"chart2710","2710","Inngående merverdiavgift","liability_current","","False","no_chart_template"
"chart2711","2711","Inngående merverdiavgift høy sats","liability_current","","False","no_chart_template"
"chart2712","2712","Inngående merverdiavgift kjøp tjen. fra utlandet","liability_current","","False","no_chart_template"
"chart2713","2713","Inngående merverdiavgift middels sats","liability_current","","False","no_chart_template"
"chart2714","2714","Inngående merverdiavgift lav sats","liability_current","","False","no_chart_template"
"chart2727","2727","Utgående mva, kjøp varer fra utlandet, høy sats","liability_current","","False","no_chart_template"
"chart2728","2728","Utgående mva, kjøp varer fra utlandet, middels sats","liability_current","","False","no_chart_template"
"chart2740","2740","Oppgjørskonto merverdiavgift","liability_current","","False","no_chart_template"
"chart2741","2741","Inngående mva, kjøp varer fra utlandet, høy sats","liability_current","","False","no_chart_template"
"chart2742","2742","Inngående mva, kjøp varer fra utlandet, middels sats","liability_current","","False","no_chart_template"
"chart2745","2745","Grunnlag utgående mva kjøp av tjenester utland","liability_current","","False","no_chart_template"
"chart2746","2746","Motkonto grunnlag kjøp av tjenester utland","liability_current","","False","no_chart_template"
"chart2761","2761","Grunnlag innførsel varer, høy sats","liability_current","","False","no_chart_template"
"chart2762","2762","Motkonto grunnlag innførsel varer, høy sats","liability_current","","False","no_chart_template"
"chart2763","2763","Grunnlag innførsel varer, middels sats","liability_current","","False","no_chart_template"
"chart2764","2764","Motkonto grunnlag innførsel varer, middels sats","liability_current","","False","no_chart_template"
"chart2765","2765","Grunnlag innførsel varer, ingen mva","liability_current","","False","no_chart_template"
"chart2766","2766","Motkonto grunnlag innførsel varer, ingen mva","liability_current","","False","no_chart_template"
"chart2770","2770","Skyldig arbeidsgiveravgift","liability_current","","False","no_chart_template"
"chart2780","2780","Påløpt arbeidsgiveravgift","liability_current","","False","no_chart_template"
"chart2781","2781","Arb.giv.avg. pål. feriep.","liability_current","","False","no_chart_template"
"chart2785","2785","Påløpt arbeidsgiveravgift ferielønn","liability_current","","False","no_chart_template"
"chart2790","2790","Andre offentlige avgifter","liability_current","","False","no_chart_template"
"chart2800","2800","Avsatt utbytte","liability_current","","False","no_chart_template"
"chart2900","2900","Forskudd fra kunder","liability_current","","False","no_chart_template"
"chart2902","2902","Forskudd kunder gavekort","liability_current","","False","no_chart_template"
"chart2903","2903","Forskudd kunder tilgodelapp","liability_current","","False","no_chart_template"
"chart2910","2910","Gjeld til ansatte og eiere","liability_current","","False","no_chart_template"
"chart2920","2920","Gjeld til selskap i samme konsern","liability_current","","False","no_chart_template"
"chart2930","2930","Lønn","liability_current","","False","no_chart_template"
"chart2940","2940","Feriepenger","liability_current","","False","no_chart_template"
"chart2950","2950","Påløpte renter","liability_current","","False","no_chart_template"
"chart2960","2960","Påløpte kostn. og forskuddsbet. inskudd","liability_current","","False","no_chart_template"
"chart2970","2970","Uopptjent inntekt , avsetning","liability_current","","False","no_chart_template"
"chart2980","2980","Avsetninger og forpliktelser","liability_current","","False","no_chart_template"
"chart2990","2990","Annen kortsiktig gjeld","liability_current","","False","no_chart_template"
"chart3000","3000","Salgsinntekt handelsvarer avgiftspl. høy sats","income","account.account_tag_operating","False","no_chart_template"
"chart3010","3010","Salgsinntekt egentilv. varer avgiftspl. høy sats","income","account.account_tag_operating","False","no_chart_template"
"chart3020","3020","Salgsinntekt tjenester avgiftspl. høy sats","income","","False","no_chart_template"
"chart3030","3030","Salgsinntekt handelsvarer avgiftpl. middels sats","income","","False","no_chart_template"
"chart3035","3035","Salgsinnt. handelsvarer, avgiftspliktig, lav sats","income","","False","no_chart_template"
"chart3040","3040","Salgsinntekt egentilv. varer avgiftpl middels sats","income","","False","no_chart_template"
"chart3050","3050","Salgsinntekter tjenester avgiftspl. lav sats","income","","False","no_chart_template"
"chart3060","3060","Uttak av varer avgiftspliktig høy sats","income","","False","no_chart_template"
"chart3061","3061","Uttak av varer, avgiftspliktig, høy sats","income","","False","no_chart_template"
"chart3062","3062","Uttak av varer, avg.pliktig, middels sats","income","","False","no_chart_template"
"chart3063","3063","Uttak av varer avgiftspliktig middels sats","income","","False","no_chart_template"
"chart3070","3070","Uttak av tjenester avgiftspliktig høy sats","income","","False","no_chart_template"
"chart3071","3071","Uttak av tjenester, avgiftspliktig, høy sats","income","","False","no_chart_template"
"chart3074","3074","Uttak av tjenester avgiftspliktig lav sats","income","","False","no_chart_template"
"chart3080","3080","Rabatter og annen salgsinntektsred., avgiftspl.","income","","False","no_chart_template"
"chart3090","3090","Refunderbare utlegg for kjøpers regning, avgiftspl","income","","False","no_chart_template"
"chart3095","3095","Ikke fakturert omsetning","income","","False","no_chart_template"
"chart3096","3096","Fakturert men ikke levert","income","","False","no_chart_template"
"chart3100","3100","Salgsinntekt handelsvarer avgiftsfri","income","account.account_tag_operating","False","no_chart_template"
"chart3105","3105","Salgsinntekt handelsvarer, utførsel, avgiftsfri","income","","False","no_chart_template"
"chart3110","3110","Salgsinntekt egentilvirkede varer avgiftsfri","income","account.account_tag_operating","False","no_chart_template"
"chart3120","3120","Salgsinntekt tjenester avgiftsfri","income","account.account_tag_operating","False","no_chart_template"
"chart3125","3125","Salgsinntekt tjenester, utførsel, avgiftsfri","income","","False","no_chart_template"
"chart3160","3160","Uttak av varer avgiftsfritt","income","account.account_tag_operating","False","no_chart_template"
"chart3170","3170","Uttak av tjenester, avgiftsfritt","income","","False","no_chart_template"
"chart3180","3180","Rabatter og andre salgsinntektsreduksjon avgiftsfri","income","","False","no_chart_template"
"chart3190","3190","Refunderbare utlegg for kjøpers regning, avg.fri","income","","False","no_chart_template"
"chart3200","3200","Salgsinntekt handelsvarervarer utenfor avg.omr","income","","False","no_chart_template"
"chart3210","3210","Salgsinntekt egentilvirkede varer utenfor avg.omr","income","","False","no_chart_template"
"chart3220","3220","Salgsinntekt tjenester utenfor avg.omr","income","","False","no_chart_template"
"chart3260","3260","Uttak av varer utenfor avgiftsområdet","income","","False","no_chart_template"
"chart3280","3280","Rabatter og annen salgsinntektsreduksjon","income","","False","no_chart_template"
"chart3281","3281","Kasserabatt, git kunde","income","","False","no_chart_template"
"chart3300","3300","Spes. offent. avg. tilvirk./solgte varer","income","","False","no_chart_template"
"chart3301","3301","Spesiell off. avg. tilv./solgte varer fritt","income","","False","no_chart_template"
"chart3302","3302","Miljø avgift for tilv./solgte varer avgiftspliktig","income","","False","no_chart_template"
"chart3303","3303","Miljø avgift for tilv./solgte varer avgiftsfritt","income","","False","no_chart_template"
"chart3400","3400","Spes. offent. avg. tilvirk./solgte varer","income","","False","no_chart_template"
"chart3440","3440","Spes. offentlige tilskudd for tjenester","income","","False","no_chart_template"
"chart3500","3500","Uopptjente inntekter garanti","income","","False","no_chart_template"
"chart3510","3510","Uopptjente inntekter service","income","","False","no_chart_template"
"chart3600","3600","Leieinntekter fast eiendom","income","","False","no_chart_template"
"chart3610","3610","Leieinntekter andre varige driftsmidler","income","","False","no_chart_template"
"chart3620","3620","Andre leieinntekter","income","","False","no_chart_template"
"chart3700","3700","Provisjonsinntekter","income","","False","no_chart_template"
"chart3800","3800","Gevinst ved avgang av anleggsmidler","income","","False","no_chart_template"
"chart3900","3900","Andre driftsrelaterte inntekter, avgiftspliktig","income","","False","no_chart_template"
"chart3910","3910","Utgående porto, avgiftspliktig","income","","False","no_chart_template"
"chart3920","3920","Utgående gebyrer, avgiftspliktig","income","","False","no_chart_template"
"chart3950","3950","Annen driftsrelatert inntekt, avgiftsfritt","income","","False","no_chart_template"
"chart3960","3960","Utgående porto, avgiftsfritt","income","","False","no_chart_template"
"chart3970","3970","Utgående gebyrer, avgiftsfritt","income","","False","no_chart_template"
"chart4000","4000","Innkjøp av råvarer og halvfabrikata høy sats","expense","","False","no_chart_template"
"chart4030","4030","Innkjøp av råvarer og halvfabrikata middels sats","expense","","False","no_chart_template"
"chart4035","4035","Innkjøp varer og halvfabrikata, lav avgiftssats","expense","","False","no_chart_template"
"chart4060","4060","Frakt, toll og spedisjon","expense","","False","no_chart_template"
"chart4070","4070","Innkjøpsprisreduksjon","expense","","False","no_chart_template"
"chart4090","4090","Beholdningsendring","expense","","False","no_chart_template"
"chart4100","4100","Innkjøp varer under tilvirkning høy sats","expense","","False","no_chart_template"
"chart4130","4130","Innkjøp varer under tilvirkning middels sats","expense","","False","no_chart_template"
"chart4160","4160","Frakt, toll og spedisjon","expense","","False","no_chart_template"
"chart4170","4170","Innkjøpsprisreduksjon","expense","","False","no_chart_template"
"chart4190","4190","Beholdningsendring","expense","","False","no_chart_template"
"chart4200","4200","Innkjøp ferdig egentilvirkede varer høy sats","expense","","False","no_chart_template"
"chart4230","4230","Innkjøp ferdig egentilvirkede varer middels sats","expense","","False","no_chart_template"
"chart4260","4260","Frakt, toll og spedisjon","expense","","False","no_chart_template"
"chart4270","4270","Innkjøpsprisreduksjon, avgiftspliktig","expense","","False","no_chart_template"
"chart4290","4290","Beholdningsendring","expense","","False","no_chart_template"
"chart4300","4300","Innkjøp varer for videresalg høy sats","expense","","False","no_chart_template"
"chart4330","4330","Innkjøp varer for videresalg middels sats","expense","","False","no_chart_template"
"chart4360","4360","Frakt, toll m.m. vedr. innkjøp av varer for videresalg","expense","","False","no_chart_template"
"chart4370","4370","Rabatter m.m. vedr. innkjøp av varer for videresalg","expense","","False","no_chart_template"
"chart4380","4380","Varekostnad","expense","","False","no_chart_template"
"chart4390","4390","Beholdningsendring varer for videresalg","expense","","False","no_chart_template"
"chart4400","4400","Fritt Kjøp","expense","","False","no_chart_template"
"chart4470","4470","Innkjøpsprisreduksjoner, fritt","expense","","False","no_chart_template"
"chart4500","4500","Fremmedytelser og underentreprise","expense","","False","no_chart_template"
"chart4590","4590","Beholdningsendring","expense","","False","no_chart_template"
"chart4600","4600","Emballasjematerialer ","expense","","False","no_chart_template"
"chart4690","4690","Beholdningsendring 2","expense","","False","no_chart_template"
"chart4800","4800","Eksp. gebyr pliktig, vareinnkjøp","expense","","False","no_chart_template"
"chart4810","4810","Frakt og porto pliktig, vareinnkjøp","expense","","False","no_chart_template"
"chart4860","4860","Eksp. gebyr fritt, vareinnkjøp","expense","","False","no_chart_template"
"chart4900","4900","Annen periodisering","expense","","False","no_chart_template"
"chart4910","4910","Justering av lager","expense","","False","no_chart_template"
"chart4990","4990","Beholdningsendring","expense","","False","no_chart_template"
"chart5000","5000","Lønn til ansatte","expense","","False","no_chart_template"
"chart5005","5005","Avtalte tariffgodtgjørelser","expense","","False","no_chart_template"
"chart5090","5090","Periodiseringskonto lønn","expense","","False","no_chart_template"
"chart5091","5091","Påløpt, ikke utbetalt lønn","expense","","False","no_chart_template"
"chart5092","5092","Feriepenger","expense","","False","no_chart_template"
"chart5099","5099","Andre lønnsposteringer","expense","","False","no_chart_template"
"chart5100","5100","Lønn til ansatte, timeansatte","expense","","False","no_chart_template"
"chart5105","5105","Avtalte tariffgodtgjørelser, timeansatte","expense","","False","no_chart_template"
"chart5180","5180","Feriepenger beregnet","expense","","False","no_chart_template"
"chart5182","5182","Arbeidsgiveravgift påløpte feriepenger","expense","","False","no_chart_template"
"chart5191","5191","Påløpt, ikke utbetalt lønn timeansatte","expense","","False","no_chart_template"
"chart5192","5192","Feriepenger, timeaansatte","expense","","False","no_chart_template"
"chart5199","5199","Andre lønnsposteringer, timeansatte","expense","","False","no_chart_template"
"chart5200","5200","Fri bil","expense","","False","no_chart_template"
"chart5210","5210","Fri telefon","expense","","False","no_chart_template"
"chart5220","5220","Fri avis","expense","","False","no_chart_template"
"chart5230","5230","Fri losji og bolig","expense","","False","no_chart_template"
"chart5240","5240","Rentefordel","expense","","False","no_chart_template"
"chart5251","5251","Gruppelivsforsikring","expense","","False","no_chart_template"
"chart5252","5252","Ulykkesforsikring","expense","","False","no_chart_template"
"chart5260","5260","Smusstillegg","expense","","False","no_chart_template"
"chart5280","5280","Andre fordeler i arbeidsforhold","expense","","False","no_chart_template"
"chart5290","5290","Motkonto for gruppe 52","expense","","False","no_chart_template"
"chart5300","5300","Tantieme","expense","","False","no_chart_template"
"chart5330","5330","Godtgj. til styre- og bedriftsforsamling","expense","","False","no_chart_template"
"chart5390","5390","Annen oppgavepliktig godtgjørelse","expense","","False","no_chart_template"
"chart5395","5395","Annen oppgavepliktig godtgjørelse, trekkfri","expense","","False","no_chart_template"
"chart5400","5400","Arbeidsgiveravgift","expense","","False","no_chart_template"
"chart5405","5405","Arbeidsgiveravgift av påløpte feriepenger","expense","","False","no_chart_template"
"chart5411","5411","Arb.giv.avg. pål. feriep.","expense","","False","no_chart_template"
"chart5420","5420","Innberetningspliktige pensjonskostnader","expense","","False","no_chart_template"
"chart5430","5430","Premie pensjonsordning","expense","","False","no_chart_template"
"chart5500","5500","Andre kostnadsgodtgjørelser","expense","","False","no_chart_template"
"chart5510","5510","Overtidsmat etter regning","expense","","False","no_chart_template"
"chart5520","5520","Kantinekostnader","expense","","False","no_chart_template"
"chart5600","5600","Arbeidsgodtgjørelse til eiere i DLS","expense","","False","no_chart_template"
"chart5700","5700","Lærlingtilskudd","expense","","False","no_chart_template"
"chart5800","5800","Refusjon av sykepenger","expense","","False","no_chart_template"
"chart5820","5820","Refusjon av arbeidsgiveravgift","expense","","False","no_chart_template"
"chart5890","5890","Annen refusjon","expense","","False","no_chart_template"
"chart5900","5900","Gaver til ansatte","expense","","False","no_chart_template"
"chart5910","5910","Kantinekostnader","expense","","False","no_chart_template"
"chart5920","5920","Yrkesskadeforsikring","expense","","False","no_chart_template"
"chart5930","5930","Andre ikke arb.giv.avg.pliktige forsikr.","expense","","False","no_chart_template"
"chart5940","5940","Øvrige personalkostnader","expense","","False","no_chart_template"
"chart5950","5950","Obligatorisk tjenestepensjon (OTP)","expense","","False","no_chart_template"
"chart5990","5990","Annen personalkostnad","expense","","False","no_chart_template"
"chart6000","6000","Avskrivning på bygn. & annen fast eiend.","expense","","False","no_chart_template"
"chart6010","6010","Avskrivning på transportmidler, maskiner","expense","","False","no_chart_template"
"chart6020","6020","Avskrivning på immaterielle eiendeler","expense","","False","no_chart_template"
"chart6050","6050","Nedskr. varige driftsmidl. & imat. eiend","expense","","False","no_chart_template"
"chart6100","6100","Frakter, transportkostnader og forsikring","expense","","False","no_chart_template"
"chart6110","6110","Toll og spedisjonskostnader ved forsend","expense","","False","no_chart_template"
"chart6200","6200","Elektrisitet","expense","","False","no_chart_template"
"chart6210","6210","Gass","expense","","False","no_chart_template"
"chart6220","6220","Fyringsolje","expense","","False","no_chart_template"
"chart6230","6230","Kull, koks","expense","","False","no_chart_template"
"chart6240","6240","Ved","expense","","False","no_chart_template"
"chart6250","6250","Bensin, dieselolje","expense","","False","no_chart_template"
"chart6260","6260","Vann","expense","","False","no_chart_template"
"chart6290","6290","Annen brensel","expense","","False","no_chart_template"
"chart6300","6300","Leie lokaler","expense","","False","no_chart_template"
"chart6320","6320","Renovasjon, vann, avløp mv.","expense","","False","no_chart_template"
"chart6340","6340","Lys, varme","expense","","False","no_chart_template"
"chart6360","6360","Renhold","expense","","False","no_chart_template"
"chart6390","6390","Annen kostnad lokaler","expense","","False","no_chart_template"
"chart6400","6400","Leie av driftsmidler","expense","","False","no_chart_template"
"chart6410","6410","Leie av inventar","expense","","False","no_chart_template"
"chart6420","6420","Leie datasystemer","expense","","False","no_chart_template"
"chart6430","6430","Leie andre kontormaskiner","expense","","False","no_chart_template"
"chart6440","6440","Leie transportmidler","expense","","False","no_chart_template"
"chart6490","6490","Annen leiekostnad","expense","","False","no_chart_template"
"chart6500","6500","Motordrevet verktøy","expense","","False","no_chart_template"
"chart6510","6510","Håndverktøy","expense","","False","no_chart_template"
"chart6520","6520","Hjelpeverktøy","expense","","False","no_chart_template"
"chart6530","6530","Spesialverktøy","expense","","False","no_chart_template"
"chart6540","6540","Inventar","expense","","False","no_chart_template"
"chart6550","6550","Driftsmaterialer","expense","","False","no_chart_template"
"chart6560","6560","Rekvisita","expense","","False","no_chart_template"
"chart6570","6570","Arbeidsklær og verneutstyr","expense","","False","no_chart_template"
"chart6590","6590","Annet driftsmateriel","expense","","False","no_chart_template"
"chart6600","6600","Reparasjoner og vedlikehold bygninger","expense","","False","no_chart_template"
"chart6620","6620","Reparasjoner og vedlikehold utstyr","expense","","False","no_chart_template"
"chart6690","6690","Reparasjon og vedlikehold annet","expense","","False","no_chart_template"
"chart6700","6700","Revisjonshonorar","expense","","False","no_chart_template"
"chart6720","6720","Honorar for økonomisk & juridisk bistand","expense","","False","no_chart_template"
"chart6750","6750","Honorar regnskapsfører","expense","","False","no_chart_template"
"chart6785","6785","Kjøp av tjenester fra utlandet","expense","","False","no_chart_template"
"chart6800","6800","Kontorrekvisita","expense","","False","no_chart_template"
"chart6810","6810","Datakostnad","expense","","False","no_chart_template"
"chart6820","6820","Trykksaker","expense","","False","no_chart_template"
"chart6840","6840","Aviser, tidsskrifter, bøker mv.","expense","","False","no_chart_template"
"chart6860","6860","Møter, kurs, oppdatering mv.","expense","","False","no_chart_template"
"chart6890","6890","Annen kontorkostnad","expense","","False","no_chart_template"
"chart6900","6900","Telefon","expense","","False","no_chart_template"
"chart6901","6901","Telefon fritt","expense","","False","no_chart_template"
"chart6940","6940","Porto","expense","","False","no_chart_template"
"chart7000","7000","Drivstoff","expense","","False","no_chart_template"
"chart7020","7020","Vedlikehold","expense","","False","no_chart_template"
"chart7040","7040","Forsikringer","expense","","False","no_chart_template"
"chart7080","7080","Bilkostnader, bruk av privat bil i næring","expense","","False","no_chart_template"
"chart7090","7090","Annen kostnad transportmidler","expense","","False","no_chart_template"
"chart7100","7100","Bilgodtgjørelse, oppgavepliktig","expense","","False","no_chart_template"
"chart7130","7130","Reisekostnader, oppgavepliktige","expense","","False","no_chart_template"
"chart7140","7140","Reisekostnader, ikke oppgavepliktig","expense","","False","no_chart_template"
"chart7150","7150","Diettkostnader, oppgaveplikig","expense","","False","no_chart_template"
"chart7160","7160","Diettkostnader, ikke oppgavepliktig","expense","","False","no_chart_template"
"chart7190","7190","Annen kostnadsgodtgjørelse","expense","","False","no_chart_template"
"chart7200","7200","Provisjonskostnader, oppgavepliktige","expense","","False","no_chart_template"
"chart7210","7210","Provisjonskostnader, ikke oppgavepliktig","expense","","False","no_chart_template"
"chart7300","7300","Salgskostnader","expense","","False","no_chart_template"
"chart7320","7320","Reklamekostnader","expense","","False","no_chart_template"
"chart7350","7350","Representasjon, fradragsberettiget","expense","","False","no_chart_template"
"chart7360","7360","Representasjon, ikke fradragsberettiget","expense","","False","no_chart_template"
"chart7390","7390","MVA-øreavrunding","expense","","False","no_chart_template"
"chart7395","7395","Øreavrunding","expense","","False","no_chart_template"
"chart7400","7400","Kontingenter og gaver","expense","","False","no_chart_template"
"chart7410","7410","Kontingenter, ikke fradragsberettiget","expense","","False","no_chart_template"
"chart7420","7420","Gaver, fradragsberettigede","expense","","False","no_chart_template"
"chart7430","7430","Gaver, ikke fradrag","expense","","False","no_chart_template"
"chart7500","7500","Forsikringspremier","expense","","False","no_chart_template"
"chart7550","7550","Garanti- og servicekostnader","expense","","False","no_chart_template"
"chart7560","7560","Servicekostnader","expense","","False","no_chart_template"
"chart7600","7600","Lisenesavgifter og royalties","expense","","False","no_chart_template"
"chart7610","7610","Patentkostnad ved egen patent","expense","","False","no_chart_template"
"chart7620","7620","Kostnader ved varemerker o.l.","expense","","False","no_chart_template"
"chart7630","7630","Kontroll-,prøve- og stempelavgifter","expense","","False","no_chart_template"
"chart7700","7700","Styre- og bedriftsforsamlingsmøter","expense","","False","no_chart_template"
"chart7710","7710","Generalforsamling","expense","","False","no_chart_template"
"chart7730","7730","Kostnader ved egne aksjer","expense","","False","no_chart_template"
"chart7740","7740","Øreavrunding, MVA - oppgjør","expense","","False","no_chart_template"
"chart7745","7745","Øreavrunding, avgiftspliktig","expense","","False","no_chart_template"
"chart7746","7746","Øreavrunding, avgiftsfritt","expense","","False","no_chart_template"
"chart7750","7750","Eiendoms- og festeavgift","expense","","False","no_chart_template"
"chart7770","7770","Bank og kortgebyrer","expense","","False","no_chart_template"
"chart7780","7780","Renter og gebyrer inkasso","expense","","False","no_chart_template"
"chart7798","7798","Annen kostnad, fradragsberettiget","expense","","False","no_chart_template"
"chart7799","7799","Annen kostnad, ikke fradragsberettiget","expense","","False","no_chart_template"
"chart7800","7800","Tap ved avgang driftsmidler","expense","","False","no_chart_template"
"chart7820","7820","Innkommet på tidligere nedskrevne fordri","expense","","False","no_chart_template"
"chart7830","7830","Tap på fordringer","expense","","False","no_chart_template"
"chart7850","7850","Tap pga. brannskade","expense","","False","no_chart_template"
"chart7860","7860","Tap på kontrakter","expense","","False","no_chart_template"
"chart7900","7900","Beholdningsendring anlegg under utførelse","expense","","False","no_chart_template"
"chart7910","7910","Ukurante varer","expense","","False","no_chart_template"
"chart8000","8000","Inntekter på investeringer i datterselskap","expense","","False","no_chart_template"
"chart8010","8010","Inntekt på investering i annet foretak. s/ konsern","expense","","False","no_chart_template"
"chart8020","8020","Inntekt på investering i tilknyttet selskap","expense","","False","no_chart_template"
"chart8030","8030","Renteinntekt på foretak i  samme konsern","expense","","False","no_chart_template"
"chart8040","8040","Renteinntekter, skattefrie","expense","","False","no_chart_template"
"chart8050","8050","Annen renteinntekt","expense","","False","no_chart_template"
"chart8056","8056","Påminnelsesavgift","expense","","False","no_chart_template"
"chart8060","8060","Valutagevinst (agio)","expense","","False","no_chart_template"
"chart8065","8065","Valutakursvinster, revenue","expense","","False","no_chart_template"
"chart8070","8070","Annen finansinntekt","expense","account.account_tag_financing","False","no_chart_template"
"chart8071","8071","Aksjeutbytte","expense","","False","no_chart_template"
"chart8078","8078","Gevinst realisasjon av aksjer","expense","","False","no_chart_template"
"chart8080","8080","Verdiøkning finanseille omløpsmidler","expense","account.account_tag_financing","False","no_chart_template"
"chart8090","8090","Inntekt på andre investeringer","expense","","False","no_chart_template"
"chart8100","8100","Verdired. av markedsbas.finans. omløps.","expense","account.account_tag_financing","False","no_chart_template"
"chart8110","8110","Nedskrivn. av andre finansielle omløps.","expense","account.account_tag_financing","False","no_chart_template"
"chart8120","8120","Nedskrivning av finansielle anleggsmidl.","expense","account.account_tag_financing","False","no_chart_template"
"chart8130","8130","Rentekostnad foretak i samme konsern","expense","","False","no_chart_template"
"chart8140","8140","Rentekostnader, ikke fradragsberettigede","expense","","False","no_chart_template"
"chart8150","8150","Annen rentekostnad","expense","","False","no_chart_template"
"chart8160","8160","Valutatap (disagio)","expense","","False","no_chart_template"
"chart8178","8178","Tap ved realisasjon av aksjer","expense","","False","no_chart_template"
"chart8179","8179","Annen finanskostnad","expense","","False","no_chart_template"
"chart8300","8300","Betalbar skatt","expense","","False","no_chart_template"
"chart8320","8320","Utsatt skatt","expense","","False","no_chart_template"
"chart8350","8350","Skattekostnad","expense","","False","no_chart_template"
"chart8400","8400","Ekstraordinær inntekt","expense","","False","no_chart_template"
"chart8500","8500","Ekstraordinær kostnad","expense","","False","no_chart_template"
"chart8600","8600","Betalbar skatt, ekstraordinært resultat","expense","","False","no_chart_template"
"chart8620","8620","Utsatt skatt, ekstraordinært resultat","expense","","False","no_chart_template"
"chart8800","8800","Årsresultat","expense","","False","no_chart_template"
"chart8900","8900","Overføringer fond for vurderingsforskjel","expense","","False","no_chart_template"
"chart8910","8910","Overføringer felleseid andelskapital for","expense","","False","no_chart_template"
"chart8920","8920","Avsatt utbytte/renter på grunnfondsbevis","expense","","False","no_chart_template"
"chart8922","8922","Avsatt tilleggsutbytte","expense","","False","no_chart_template"
"chart8923","8923","Avsatt ekstraordinært utbytte","expense","","False","no_chart_template"
"chart8930","8930","Konsernbidrag","expense","","False","no_chart_template"
"chart8940","8940","Aksjonærbidrag","expense","","False","no_chart_template"
"chart8950","8950","Fondsemisjon","expense","","False","no_chart_template"
"chart8960","8960","Overføringer annen egenkapital","expense","","False","no_chart_template"
"chart8980","8980","Avsatt til fri egenkapital","expense","","False","no_chart_template"
"chart8990","8990","Udekket tap","expense","","False","no_chart_template"
"chart9970","9970","Expense account on product template","expense","","False","no_chart_template"
"chart9990","9990","Income account on product template","income","","False","no_chart_template"

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
            <field name="default_pos_receivable_account_id" ref="chart1501"/>
            <field name="account_journal_early_pay_discount_loss_account_id" ref="chart4370"/>
            <field name="account_journal_early_pay_discount_gain_account_id" ref="chart3080"/>
            <field name="property_tax_payable_account_id" ref="chart2740"/>
            <field name="property_tax_receivable_account_id" ref="chart2740"/>
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
                (0,0, {'repartition_type': 'base'}),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_1_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_1_tag')],
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
                (0,0, {'repartition_type': 'base'}),

                (0,0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),

                (0,0, {'repartition_type': 'tax'}),
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_3_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2701'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_3_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_3_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2701'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_3_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_5_tag')],
                }),

                (0,0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_5_tag')],
                }),

                (0,0, {'repartition_type': 'tax'}),
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_6_tag')],
                }),

                (0,0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_6_tag')],
                }),

                (0,0, {'repartition_type': 'tax'}),
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
                (0,0, {'repartition_type': 'base'}),

                (0,0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),

                (0,0, {'repartition_type': 'tax'}),
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
                (0,0, {'repartition_type': 'base'}),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2713'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_11_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2713'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_11_tag')],
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
                (0,0, {'repartition_type': 'base'}),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2710'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_12_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2710'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_12_tag')],
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
                (0,0, {'repartition_type': 'base'}),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2714'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_13_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2714'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_13_tag')],
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
                (0,0, {'repartition_type': 'base'}),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_14_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_14_tag')],
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
                (0,0, {'repartition_type': 'base'}),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2713'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_15_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2713'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_15_tag')],
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
                (0,0, {'repartition_type': 'base'}),

                (0,0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),

                (0,0, {'repartition_type': 'tax'}),
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
                (0,0, {'repartition_type': 'base'}),

                (0,0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),

                (0,0, {'repartition_type': 'tax'}),
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
                (0,0, {'repartition_type': 'base'}),

                (0,0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {'repartition_type': 'base'}),

                (0,0, {'repartition_type': 'tax'}),
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_31_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2703'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_31_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_31_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2703'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_31_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_32_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2700'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_32_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_32_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2700'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_32_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_33_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2704'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_33_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_33_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2704'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_33_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_51_tag')],
                }),

                (0,0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_51_tag')],
                }),

                (0,0, {'repartition_type': 'tax'}),
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_52_tag')],
                }),

                (0,0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_52_tag')],
                }),

                (0,0, {'repartition_type': 'tax'}),
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_81_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2741'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_81_tax_tag')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2727'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_81_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_81_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2741'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_81_tax_tag')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2727'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_81_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_82_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2741'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_82_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_82_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2741'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_82_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_83_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2742'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_83_tax_tag')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2728'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_83_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_83_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2742'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_83_tax_tag')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2728'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_83_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_84_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2742'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_84_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_84_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2742'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_84_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_85_tag')],
                }),

                (0,0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_85_tag')],
                }),

                (0,0, {'repartition_type': 'tax'}),
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_86_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_86_tax_tag')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2702'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_86_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_86_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_86_tax_tag')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2702'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_86_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_87_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_87_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_87_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_87_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_88_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_88_tax_tag')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2702'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_88_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_88_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_88_tax_tag')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2702'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_88_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_89_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_89_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_89_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2712'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_89_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_91_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_91_tax_tag')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2701'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_91_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_91_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_91_tax_tag')],
                }),

                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('chart2701'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_91_tax_tag')],
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
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_line_code_92_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'plus_report_expression_ids': [ref('tax_report_line_code_92_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('tax_report_line_code_92_tag')],
                }),

                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart2711'),
                    'minus_report_expression_ids': [ref('tax_report_line_code_92_tax_tag')],
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
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.no"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
                <field name="sequence" eval="1"/>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_sales_goods_services_homeland" model="account.report.line">
                <field name="name">Salg av varer og tjenester i Norge</field>
                <field name="sequence" eval="1"/>
                <field name="children_ids">
                    <record id="tax_report_line_code_3" model="account.report.line">
                        <field name="name">3 Salg og uttak av varer og tjenester (høy sats 25%) - grunnlag</field>
                        <field name="code">BASE_3</field>
                        <field name="sequence" eval="2"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_3_tax" model="account.report.line">
                        <field name="name">3 Salg og uttak av varer og tjenester (høy sats 25%) - avgift</field>
                        <field name="code">TAX_3</field>
                        <field name="sequence" eval="3"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_3_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">3 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_31" model="account.report.line">
                        <field name="name">31 Salg og uttak av varer og tjenester (middels sats 15%) - grunnlag</field>
                        <field name="code">BASE_31</field>
                        <field name="sequence" eval="4"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_31_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">31 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_31_tax" model="account.report.line">
                        <field name="name">31 Salg og uttak av varer og tjenester (middels sats 15%) - avgift</field>
                        <field name="code">TAX_31</field>
                        <field name="sequence" eval="5"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_31_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">31 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_33" model="account.report.line">
                        <field name="name">33 Salg og uttak av varer og tjenester (lav sats 12%) - grunnlag</field>
                        <field name="code">BASE_33</field>
                        <field name="sequence" eval="6"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_33_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">33 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_33_tax" model="account.report.line">
                        <field name="name">33 Salg og uttak av varer og tjenester (lav sats 12%) - avgift</field>
                        <field name="code">TAX_33</field>
                        <field name="sequence" eval="7"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_33_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">33 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_5" model="account.report.line">
                        <field name="name">5 Salg og uttak av varer og tjenester som er fritatt for merverdiavgift (0%)</field>
                        <field name="code">BASE_5</field>
                        <field name="sequence" eval="8"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">5 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_6" model="account.report.line">
                        <field name="name">6 Salg av varer og tjenester som er unntatt merverdiavgiftsloven (0%)</field>
                        <field name="code">BASE_6</field>
                        <field name="sequence" eval="9"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_6_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">6 Base</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_sales_goods_services_abroad" model="account.report.line">
                <field name="name">Salg av varer og tjenester til utlandet</field>
                <field name="sequence" eval="10"/>
                <field name="children_ids">
                    <record id="tax_report_line_code_52" model="account.report.line">
                        <field name="name">52 Salg av varer og tjenester til utlandet som er fritatt for merverdiavgift (0%)</field>
                        <field name="code">BASE_52</field>
                        <field name="sequence" eval="11"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_52_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">52 Base</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_purchases_goods_services_homeland" model="account.report.line">
                <field name="name">Kjøp av varer og tjenester i Norge</field>
                <field name="sequence" eval="12"/>
                <field name="children_ids">
                    <record id="tax_report_line_code_1" model="account.report.line">
                        <field name="name">1 Kjøp av varer og tjenester med fradragsrett (høy sats 25%)</field>
                        <field name="code">TAX_1</field>
                        <field name="sequence" eval="13"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">1 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_11" model="account.report.line">
                        <field name="name">11 Kjøp av varer og tjenester med fradragsrett (middels sats 15%)</field>
                        <field name="code">TAX_11</field>
                        <field name="sequence" eval="14"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_13" model="account.report.line">
                        <field name="name">13 Kjøp av varer og tjenester med fradragsrett (lav sats 12%)</field>
                        <field name="code">TAX_13</field>
                        <field name="sequence" eval="15"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_13_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">13 Tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_purchases_goods_abroad" model="account.report.line">
                <field name="name">Kjøp av varer fra utlandet (import)</field>
                <field name="sequence" eval="16"/>
                <field name="children_ids">
                    <record id="tax_report_line_code_14" model="account.report.line">
                        <field name="name">14 Fradrag på kjøp av varer fra utlandet (merverdiavgift betalt ved innførsel, høy sats 25%)</field>
                        <field name="code">TAX_14</field>
                        <field name="sequence" eval="17"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_14_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">14 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_15" model="account.report.line">
                        <field name="name">15 Fradrag på kjøp av varer fra utlandet (merverdiavgift betalt ved innførsel, middels sats 15%)</field>
                        <field name="code">TAX_15</field>
                        <field name="sequence" eval="18"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_15_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">15 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_81" model="account.report.line">
                        <field name="name">81 Kjøp av varer fra utlandet med fradragsrett (høy sats 25%) - grunnlag</field>
                        <field name="code">BASE_81</field>
                        <field name="sequence" eval="19"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_81_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">81 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_81_tax" model="account.report.line">
                        <field name="name">81 Kjøp av varer fra utlandet med fradragsrett (høy sats 25%) - avgift</field>
                        <field name="code">TAX_81</field>
                        <field name="sequence" eval="20"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_81_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">81 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_82" model="account.report.line">
                        <field name="name">82 Kjøp av varer fra utlandet uten fradragsrett (høy sats 25%) - grunnlag</field>
                        <field name="code">BASE_82</field>
                        <field name="sequence" eval="21"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_82_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">82 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_82_tax" model="account.report.line">
                        <field name="name">82 Kjøp av varer fra utlandet uten fradragsrett (høy sats 25%) - avgift</field>
                        <field name="code">TAX_82</field>
                        <field name="sequence" eval="22"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_82_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">82 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_83" model="account.report.line">
                        <field name="name">83 Kjøp av varer fra utlandet med fradragsrett (middels sats 15%) - grunnlag</field>
                        <field name="code">BASE_83</field>
                        <field name="sequence" eval="23"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_83_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">83 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_83_tax" model="account.report.line">
                        <field name="name">83 Kjøp av varer fra utlandet med fradragsrett (middels sats 15%) - avgift</field>
                        <field name="code">TAX_83</field>
                        <field name="sequence" eval="24"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_83_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">83 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_84" model="account.report.line">
                        <field name="name">84 Kjøp av varer fra utlandet som er uten fradragsrett (middels sats 15%) - grunnlag</field>
                        <field name="code">BASE_84</field>
                        <field name="sequence" eval="25"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_84_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">84 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_84_tax" model="account.report.line">
                        <field name="name">84 Kjøp av varer fra utlandet som er uten fradragsrett (middels sats 15%) - avgift</field>
                        <field name="code">TAX_84</field>
                        <field name="sequence" eval="26"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_84_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">84 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_85" model="account.report.line">
                        <field name="name">85 Kjøp av varer fra utlandet som det ikke skal beregnes merverdiavgift på (nullsats 0%)</field>
                        <field name="code">BASE_85</field>
                        <field name="sequence" eval="27"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_85_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">85 Base</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_purchases_services_abroad" model="account.report.line">
                <field name="name">Kjøp av tjenester fra utlandet (import)</field>
                <field name="sequence" eval="28"/>
                <field name="children_ids">
                    <record id="tax_report_line_code_86" model="account.report.line">
                        <field name="name">86 Kjøp av tjenester fra utlandet med fradragsrett (høy sats 25%) - grunnlag</field>
                        <field name="code">BASE_86</field>
                        <field name="sequence" eval="29"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_86_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">86 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_86_tax" model="account.report.line">
                        <field name="name">86 Kjøp av tjenester fra utlandet med fradragsrett (høy sats 25%) - avgift</field>
                        <field name="code">TAX_86</field>
                        <field name="sequence" eval="30"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_86_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">86 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_87" model="account.report.line">
                        <field name="name">87 Kjøp av tjenester fra utlandet uten fradragsrett (høy sats 25%) - grunnlag</field>
                        <field name="code">BASE_87</field>
                        <field name="sequence" eval="31"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_87_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">87 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_87_tax" model="account.report.line">
                        <field name="name">87 Kjøp av tjenester fra utlandet uten fradragsrett (høy sats 25%) - avgift</field>
                        <field name="code">TAX_87</field>
                        <field name="sequence" eval="32"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_87_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">87 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_88" model="account.report.line">
                        <field name="name">88 Kjøp av tjenester fra utlandet med fradragsrett (lav sats 12%) - grunnlag</field>
                        <field name="code">BASE_88</field>
                        <field name="sequence" eval="33"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_88_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">88 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_88_tax" model="account.report.line">
                        <field name="name">88 Kjøp av tjenester fra utlandet med fradragsrett (lav sats 12%) - avgift</field>
                        <field name="code">TAX_88</field>
                        <field name="sequence" eval="34"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_88_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">88 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_89" model="account.report.line">
                        <field name="name">89 Kjøp av tjenester fra utlandet uten fradragsrett (lav sats 12%) - grunnlag</field>
                        <field name="code">BASE_89</field>
                        <field name="sequence" eval="35"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_89_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">89 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_89_tax" model="account.report.line">
                        <field name="name">89 Kjøp av tjenester fra utlandet uten fradragsrett (lav sats 12%) - avgift</field>
                        <field name="code">TAX_89</field>
                        <field name="sequence" eval="36"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_89_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">89 Tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_fish_etc" model="account.report.line">
                <field name="name">Fisk mv.</field>
                <field name="sequence" eval="37"/>
                <field name="children_ids">
                    <record id="tax_report_line_code_12" model="account.report.line">
                        <field name="name">12 Kjøp av fisk og andre marine viltlevende ressurser (11,11%)</field>
                        <field name="code">TAX_12</field>
                        <field name="sequence" eval="38"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">12 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_32" model="account.report.line">
                        <field name="name">32 Salg av fisk og andre marine viltlevende ressurser (11,11%) - grunnlag</field>
                        <field name="code">BASE_32</field>
                        <field name="sequence" eval="39"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_32_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">32 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_32_tax" model="account.report.line">
                        <field name="name">32 Salg av fisk og andre marine viltlevende ressurser (11,11%) - avgift</field>
                        <field name="code">TAX_32</field>
                        <field name="sequence" eval="40"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_32_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">32 Tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_emission_and_gold" model="account.report.line">
                <field name="name">Klimakvoter og gull</field>
                <field name="sequence" eval="41"/>
                <field name="children_ids">
                    <record id="tax_report_line_code_51" model="account.report.line">
                        <field name="name">51 Salg av klimakvoter og gull til næringsdrivende (0%)</field>
                        <field name="code">BASE_51</field>
                        <field name="sequence" eval="42"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_51_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">51 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_91" model="account.report.line">
                        <field name="name">91 Kjøp av klimakvoter og gull med fradragsrett (høy sats 25%) - grunnlag</field>
                        <field name="code">BASE_91</field>
                        <field name="sequence" eval="43"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_91_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">91 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_91_tax" model="account.report.line">
                        <field name="name">91 Kjøp av klimakvoter og gull med fradragsrett (høy sats 25%) - avgift</field>
                        <field name="code">TAX_91</field>
                        <field name="sequence" eval="44"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_91_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">91 Tax</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_92" model="account.report.line">
                        <field name="name">92 Kjøp av klimakvoter og gull uten fradragsrett (høy sats 25%) - grunnlag</field>
                        <field name="code">BASE_92</field>
                        <field name="sequence" eval="45"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_92_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">92 Base</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_code_92_tax" model="account.report.line">
                        <field name="name">92 Kjøp av klimakvoter og gull uten fradragsrett (høy sats 25%) - avgift</field>
                        <field name="code">TAX_92</field>
                        <field name="sequence" eval="46"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_code_92_tax_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">92 Tax</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_sum" model="account.report.line">
                <field name="name">Sum</field>
                <field name="sequence" eval="47"/>
                <field name="children_ids">
                    <record id="tax_report_line_to_be_paid" model="account.report.line">
                        <field name="name">Avgift å betale</field>
                        <field name="code">SUM</field>
                        <field name="sequence" eval="48"/>
                        <field name="expression_ids">
                            <record id="tax_report_line_to_be_paid_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">TAX_3.balance+TAX_31.balance+TAX_32.balance+TAX_33.balance+TAX_81.balance+TAX_82.balance+TAX_83.balance+TAX_84.balance+TAX_86.balance+TAX_87.balance+TAX_88.balance+TAX_89.balance+TAX_91.balance+TAX_92.balance-TAX_1.balance-TAX_11.balance-TAX_12.balance-TAX_13.balance-TAX_14.balance-TAX_15.balance</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_no_chart_data.xml

```xml
﻿<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

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

