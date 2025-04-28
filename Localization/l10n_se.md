# Odoo Module: l10n_se

Category: Localization

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
    "name": "Swedish - Accounting",
    "version": "1.0",
    "author": "XCLUDE",
    "category": "Localization",
    "depends": ["account", "base_vat"],
    "data": [
        "data/account_chart_template_before_accounts.xml",
        "data/account.account.template.csv",
        "data/account_chart_template_after_accounts.xml",
        "data/account_tax_group.xml",
        "data/account_tax_report_data.xml",
        "data/account_tax_template.xml",
        "data/account_fiscal_position_template.xml",
        "data/account_fiscal_position_account_template.xml",
        "data/account_fiscal_position_tax_template.xml",
        "data/account_chart_template_configuration.xml",
    ],
    'license': 'LGPL-3',
 }

```

## File: data\account.account.template.csv

```csv
"id","code","name","user_type_id/id","chart_template_id/id","reconcile"
"a1030","1030","Patent","account.data_account_type_non_current_assets","l10nse_chart_template","False"
"a1039","1039","Ackumulerade avskrivningar på patent","account.data_account_type_non_current_assets","l10nse_chart_template","False"
"a1060","1060","Hyresrätter, tomträtter och liknande","account.data_account_type_non_current_assets","l10nse_chart_template","False"
"a1069","1069","Ackumulerade avskrivningar på hyresrätter, tomträtter och liknande","account.data_account_type_non_current_assets","l10nse_chart_template","False"
"a1110","1110","Byggnader på egen mark","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1119","1119","Ackumulerade avskrivningar på byggnader","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1130","1130","Mark","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1150","1150","Markanläggningar","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1159","1159","Ackumulerade avskrivningar på markanläggningar","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1210","1210","Maskiner och andra tekniska anläggningar","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1219","1219","Ackumulerade avskrivningar på maskiner och andra tekniska anläggningar","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1220","1220","Inventarier och verktyg","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1229","1229","Ackumulerade avskrivningar på inventarier och verktyg","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1240","1240","Bilar och andra transportmedel","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1249","1249","Ackumulerade avskrivningar på bilar och andra transportmedel","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1250","1250","Datorer","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1259","1259","Ackumulerade avskrivningar på datorer","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1290","1290","Övriga materiella anläggningstillgångar","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1291","1291","Konst och liknande tillgångar","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1299","1299","Ackumulerade avskrivningar på övriga materiella anläggningstillgångar","account.data_account_type_fixed_assets","l10nse_chart_template","False"
"a1350","1350","Andelar och värdepapper i andra företag","account.data_account_type_non_current_assets","l10nse_chart_template","False"
"a1380","1380","Andra långfristiga fordringar","account.data_account_type_non_current_assets","l10nse_chart_template","False"
"a1410","1410","Lager av råvaror","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1419","1419","Förändring av lager av råvaror","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1440","1440","Produkter i arbete","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1449","1449","Förändring av produkter i arbete","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1450","1450","Lager av färdiga varor","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1459","1459","Förändring av lager av färdiga varor","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1460","1460","Lager av handelsvaror","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1469","1469","Förändring av lager av handelsvaror","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1470","1470","Pågående arbete","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1479","1479","Förändring av Pågående arbete","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1480","1480","Förskott för varor och tjänster","account.data_account_type_prepayments","l10nse_chart_template","False"
"a1490","1490","Övriga lagertillgångar","account.data_account_type_prepayments","l10nse_chart_template","False"
"a1510","1510","Kundfordringar","account.data_account_type_receivable","l10nse_chart_template","True"
"a1513","1513","Kundfordringar - delad faktura","account.data_account_type_receivable","l10nse_chart_template","True"
"a1519","1519","Nedskrivning av kundfordringar","account.data_account_type_receivable","l10nse_chart_template","True"
"a1580","1580","Fordringar för kontokort och kuponger","account.data_account_type_receivable","l10nse_chart_template","True"
"a1610","1610","Kortfristiga fordringar hos anställda","account.data_account_type_receivable","l10nse_chart_template","True"
"a1630","1630","Avräkning för skatter och avgifter (skattekonto)","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1640","1640","Skattefordringar","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1650","1650","Momsfordran","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1680","1680","Andra kortfristiga fordringar","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1710","1710","Förutbetalda hyreskostnader","account.data_account_type_prepayments","l10nse_chart_template","False"
"a1720","1720","Förutbetalda leasingavgifter, kortfristig del","account.data_account_type_prepayments","l10nse_chart_template","False"
"a1730","1730","Förutbetalda försäkringspremier","account.data_account_type_prepayments","l10nse_chart_template","False"
"a1740","1740","Förutbetalda räntekostnader","account.data_account_type_prepayments","l10nse_chart_template","False"
"a1750","1750","Upplupna hyresintäkter","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1760","1760","Upplupna ränteintäkter","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1790","1790","Övriga förutbetalda kostnader och upplupna intäkter","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1810","1810","Andel i börsnoterade företag","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1880","1880","Andra kortfristiga placeringar","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1890","1890","Nedskrivning av kortfristiga placceringar","account.data_account_type_current_assets","l10nse_chart_template","False"
"a1910","1910","Kassa","account.data_account_type_liquidity","l10nse_chart_template","False"
"a1920","1920","PlusGiro","account.data_account_type_liquidity","l10nse_chart_template","False"
"a1930","1930","Företagskonto/checkkonto/affärskonto","account.data_account_type_liquidity","l10nse_chart_template","True"
"a1940","1940","Övriga bankkonton","account.data_account_type_liquidity","l10nse_chart_template","False"
"a2070","2070","Ändamålsbestämda medel","account.data_account_type_equity","l10nse_chart_template","False"
"a2081","2081","Aktiekapital","account.data_account_type_equity","l10nse_chart_template","False"
"a2083","2083","Medlemsinsatser","account.data_account_type_equity","l10nse_chart_template","False"
"a2086","2086","Reservfond","account.data_account_type_equity","l10nse_chart_template","False"
"a2090","2090","Fritt eget kapital","account.data_account_type_equity","l10nse_chart_template","False"
"a2091","2091","Balanserad vinst eller förlust","account.data_account_type_equity","l10nse_chart_template","False"
"a2098","2098","Vinst eller förlust från föregående år","account.data_account_type_equity","l10nse_chart_template","False"
"a2099","2099","Årets resultat","account.data_account_type_equity","l10nse_chart_template","False"
"a2120","2120","Periodiseringsfond 2020","account.data_account_type_equity","l10nse_chart_template","False"
"a2121","2121","Periodiseringsfond 2021","account.data_account_type_equity","l10nse_chart_template","False"
"a2122","2122","Periodiseringsfond 2022","account.data_account_type_equity","l10nse_chart_template","False"
"a2123","2123","Periodiseringsfond 2023","account.data_account_type_equity","l10nse_chart_template","False"
"a2124","2124","Periodiseringsfond 2024","account.data_account_type_equity","l10nse_chart_template","False"
"a2125","2125","Periodiseringsfond 2015","account.data_account_type_equity","l10nse_chart_template","False"
"a2126","2126","Periodiseringsfond 2016","account.data_account_type_equity","l10nse_chart_template","False"
"a2127","2127","Periodiseringsfond 2017","account.data_account_type_equity","l10nse_chart_template","False"
"a2128","2128","Periodiseringsfond 2018","account.data_account_type_equity","l10nse_chart_template","False"
"a2129","2129","Periodiseringsfond 2019","account.data_account_type_equity","l10nse_chart_template","False"
"a2150","2150","Ackumulerade överavskrivningar","account.data_account_type_non_current_liabilities","l10nse_chart_template","False"
"a2210","2210","Avsättningar för pensioner enligt tryggandelagen","account.data_account_type_non_current_liabilities","l10nse_chart_template","False"
"a2220","2220","Avsättningar för garantier","account.data_account_type_non_current_liabilities","l10nse_chart_template","False"
"a2290","2290","Övriga avsättningar","account.data_account_type_non_current_liabilities","l10nse_chart_template","False"
"a2330","2330","Checkräkningskredit","account.data_account_type_non_current_liabilities","l10nse_chart_template","False"
"a2350","2350","Andra långfristiga skulder till kreditinstitut","account.data_account_type_non_current_liabilities","l10nse_chart_template","False"
"a2390","2390","Övriga långfristga skulder","account.data_account_type_non_current_liabilities","l10nse_chart_template","False"
"a2393","2393","Lån från närstående personer, långfristig del","account.data_account_type_non_current_liabilities","l10nse_chart_template","False"
"a2410","2410","Andra kortfrigstiga låneskulder till kreditinstitut","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2420","2420","Förskott från kunder","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2440","2440","Leverantörsskulder","account.data_account_type_payable","l10nse_chart_template","True"
"a2480","2480","Checkräkningskredit, kortfristig","account.data_account_type_payable","l10nse_chart_template","True"
"a2490","2490","Övriga kortfristiga skulder till kreditinstitut, kunder och leverantörer","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2510","2510","Skatteskulder","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2610","2610","Utgående moms, 25 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2611","2611","Utgående moms på försäljning inom Sverige, 25 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2612","2612","Utgående moms på egna uttag, 25 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2613","2613","Utgående moms för uthyrning, 25 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2614","2614","Utgående moms omvänd skattskyldighet, 25 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2615","2615","Utgående moms import av varor, 25 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2616","2616","Utgående moms VMB 25 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2620","2620","Utgående moms, 12 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2621","2621","Utgående moms på försäljning inom Sverige, 12 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2622","2622","Utgående moms på egna uttag, 12 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2623","2623","Utgående moms för uthyrning, 12 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2624","2624","Utgående moms omvänd skattskyldighet, 12 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2625","2625","Utgående moms import av varor, 12 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2626","2626","Utgående moms VMB, 12 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2630","2630","Utgående moms, 6 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2631","2631","Utgående moms på försäljning inom Sverige, 6 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2632","2632","Utgående moms på egna uttag, 6 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2633","2633","Utgående moms för uthyrning, 6 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2634","2634","Utgående moms omvänd skattskyldighet, 6 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2635","2635","Utgående moms import av varor, 6 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2636","2636","Utgående moms VMB 6 %","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2640","2640","Ingående moms","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2641","2641","Debiterad ingående moms","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2642","2642","Debiterad ingående moms i anslutning till frivillig skattskyldighet","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2645","2645","Beräknad ingående moms på förvärv från utlandet","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2646","2646","Ingående moms på uthyrning","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2647","2647","Ingående moms omvänd skattskyldighet varor och tjänster i Sverige","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2648","2648","Vilande ingående moms","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2649","2649","Ingående moms, blandad verksamhet","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2650","2650","Redovisningskonto för moms","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2710","2710","Personalskatt","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2730","2730","Lagstadgade sociala avgifter och särskild löneskatt","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2740","2740","Avtalade sociala avgifter","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2790","2790","Övriga löneavdrag","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2820","2820","Kortfristiga skulder till anställda","account.data_account_type_payable","l10nse_chart_template","True"
"a2840","2840","Kortfristig låneskulder","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2890","2890","Övriga kortfristiga skulder","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2910","2910","Upplupna löner","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2920","2920","Upplupna semesterlöner","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2930","2930","Upplupna pensionskostnader","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2940","2940","Upplupna lagstadgade sociala och andra avgifter","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2950","2950","Upplupna avtalade sociala avgifter","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2960","2960","Upplupna räntekostnader","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2970","2970","Förutbetalda intäkter","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2990","2990","Övriga upplupna kostnader och förutbetalda intäkter","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a2999","2999","OBS-konto","account.data_account_type_current_liabilities","l10nse_chart_template","False"
"a3000","3000","Försäljning inom Sverige","account.data_account_type_revenue","l10nse_chart_template","False"
"a3001","3001","Försäljning inom Sverige, 25 % moms","account.data_account_type_revenue","l10nse_chart_template","False"
"a3002","3002","Försäljning inom Sverige, 12 % moms","account.data_account_type_revenue","l10nse_chart_template","False"
"a3003","3003","Försäljning inom Sverige, 6 % moms","account.data_account_type_revenue","l10nse_chart_template","False"
"a3004","3004","Försäljning inom Sverige, momsfri","account.data_account_type_revenue","l10nse_chart_template","False"
"a3100","3100","Försäljning av varor utanför EU","account.data_account_type_revenue","l10nse_chart_template","False"
"a3105","3105","Försäljning varor till land utanför EU","account.data_account_type_revenue","l10nse_chart_template","False"
"a3106","3106","Försäljning varor till annat EU-land, momspliktig","account.data_account_type_revenue","l10nse_chart_template","False"
"a3108","3108","Försäljning varor till annat EU-land, momsfri","account.data_account_type_revenue","l10nse_chart_template","False"
"a3200","3200","Försäljning VMB och omvänd moms","account.data_account_type_revenue","l10nse_chart_template","False"
"a3211","3211","Försäljning positiv VMB 25 %","account.data_account_type_revenue","l10nse_chart_template","False"
"a3212","3212","Försäljning negativ VMB 25 %","account.data_account_type_revenue","l10nse_chart_template","False"
"a3231","3231","Försäljning inom byggsektorn, omvänd skatteskyldighet moms","account.data_account_type_revenue","l10nse_chart_template","False"
"a3300","3300","Försäljning av tjänster utanför Sverige","account.data_account_type_revenue","l10nse_chart_template","False"
"a3305","3305","Försäljning av tjänster till land utanför EU","account.data_account_type_revenue","l10nse_chart_template","False"
"a3308","3308","Försäljning av tjänster till annat EU-land","account.data_account_type_revenue","l10nse_chart_template","False"
"a3400","3400","Försäljning, egna uttag","account.data_account_type_revenue","l10nse_chart_template","False"
"a3401","3401","Egna uttag momspliktiga, 25 %","account.data_account_type_revenue","l10nse_chart_template","False"
"a3402","3402","Egna uttag momspliktiga, 12 %","account.data_account_type_revenue","l10nse_chart_template","False"
"a3403","3403","Egna uttag momspliktiga, 6 %","account.data_account_type_revenue","l10nse_chart_template","False"
"a3404","3404","Egna uttag, momsfria","account.data_account_type_revenue","l10nse_chart_template","False"
"a3500","3500","Fakturerade kostnader (gruppkonto)","account.data_account_type_revenue","l10nse_chart_template","False"
"a3510","3510","Fakturerat Emballage","account.data_account_type_revenue","l10nse_chart_template","False"
"a3520","3520","Fakturerade frakter","account.data_account_type_revenue","l10nse_chart_template","False"
"a3521","3521","Fakturerade frakter, EU-land","account.data_account_type_revenue","l10nse_chart_template","False"
"a3522","3522","Fakturerade frakter, export","account.data_account_type_revenue","l10nse_chart_template","False"
"a3530","3530","Fakturerad tull- och speditionskostnader m.m.","account.data_account_type_revenue","l10nse_chart_template","False"
"a3540","3540","Faktureringsavgifter","account.data_account_type_revenue","l10nse_chart_template","False"
"a3541","3541","Faktureringsavgifter, EU-land","account.data_account_type_revenue","l10nse_chart_template","False"
"a3542","3542","Faktureringsavgifter, export","account.data_account_type_revenue","l10nse_chart_template","False"
"a3600","3600","Rörelsens sidointäkter (gruppkonto)","account.data_account_type_revenue","l10nse_chart_template","False"
"a3730","3730","Lämnade rabatter","account.data_account_type_revenue","l10nse_chart_template","False"
"a3740","3740","Öres- och kronutjämning","account.data_account_type_revenue","l10nse_chart_template","False"
"a3800","3800","Aktiverat arbete för egen räkning (gruppkonto)","account.data_account_type_revenue","l10nse_chart_template","False"
"a3900","3900","Övriga rörelseintäkter (gruppkonto)","account.data_account_type_other_income","l10nse_chart_template","False"
"a3913","3913","Frivilligt momspliktiga hyresintäkter","account.data_account_type_other_income","l10nse_chart_template","False"
"a3960","3960","Valutakursvinster på fordringar och skulder av rörelsekaraktär","account.data_account_type_other_income","l10nse_chart_template","False"
"a3970","3970","Vinst vid avyttring av immateriella och materiella anläggningstillgångar","account.data_account_type_other_income","l10nse_chart_template","False"
"a3980","3980","Erhållna offentliga stöd m.m.","account.data_account_type_other_income","l10nse_chart_template","False"
"a4000","4000","Inköp av varor från Sverige","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4200","4200","Sålda varor VMB","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4211","4211","Sålda varor positiv VMB 25 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4212","4212","Sålda varor negativ VMB 25 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4400","4400","Momspliktiga inköp i Sverige","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4415","4415","Inköpta varor i Sverige, omvänd skattskyldighet, 25 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4425","4425","Inköp tjänster i Sverige, omvänd skattskyldighet, 25 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4426","4426","Inköp tjänster i Sverige, omvänd skattskyldighet, 12 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4427","4427","Inköp tjänster i Sverige, omvänd skattskyldighet, 6 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4500","4500","Övriga momsplitiga inköp","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4515","4515","Inköp av varor från annat EU-land, 25 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4516","4516","Inköp av varor från annat EU-land, 12 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4517","4517","Inköp av varor från annat EU-land, 6 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4518","4518","Inköp av varor från annat EU-land, momsfri","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4531","4531","Inköp av tjänster från ett land utanför EU, 25 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4532","4532","Inköp av tjänster från ett land utanför EU, 12 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4533","4533","Inköp av tjänster från ett land utanför EU, 6 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4535","4535","Inköp av tjänster från annat EU-land, 25 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4536","4536","Inköp av tjänster från annat EU-land, 12 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4537","4537","Inköp av tjänster från annat EU-land, 6 %","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4538","4538","Inköp av tjänster från annat EU-land, momsfri","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4545","4545","Import av varor, 25 % moms","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4546","4546","Import av varor, 12 % moms","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4547","4547","Import av varor, 6 % moms","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4600","4600","Legoarbeten och underentreprenader (gruppkonto)","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4700","4700","Reduktion av inköpspriser (gruppkonto)","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4900","4900","Förändring av lager (gruppkonto)","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4910","4910","Förändring av lager av råvaror","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4920","4920","Förändring av lager av tillsatsmaterial och förnödenheter","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4940","4940","Förändring produkter i arbete","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4950","4950","Förändring av lager av färdiga varor","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4960","4960","Förändring av lager av handelsvaror","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a4970","4970","Förändring pågående arbete, nedladgda kostnader","account.data_account_type_direct_costs","l10nse_chart_template","False"
"a5010","5010","Lokalhyra","account.data_account_type_expenses","l10nse_chart_template","False"
"a5020","5020","El för belusning","account.data_account_type_expenses","l10nse_chart_template","False"
"a5030","5030","Värme","account.data_account_type_expenses","l10nse_chart_template","False"
"a5040","5040","Vatten och avlopp","account.data_account_type_expenses","l10nse_chart_template","False"
"a5060","5060","Städning och renhållning","account.data_account_type_expenses","l10nse_chart_template","False"
"a5070","5070","Reparation och underhåll av lokaler","account.data_account_type_expenses","l10nse_chart_template","False"
"a5120","5120","El för belysning","account.data_account_type_expenses","l10nse_chart_template","False"
"a5130","5130","Värme","account.data_account_type_expenses","l10nse_chart_template","False"
"a5140","5140","Vatten och avlopp","account.data_account_type_expenses","l10nse_chart_template","False"
"a5160","5160","Städning och renhållning","account.data_account_type_expenses","l10nse_chart_template","False"
"a5170","5170","Reparation och underhåll av fastighet","account.data_account_type_expenses","l10nse_chart_template","False"
"a5200","5200","Hyra av anläggningstillgångar (gruppkonto)","account.data_account_type_expenses","l10nse_chart_template","False"
"a5300","5300","Energikostnader (gruppkonto)","account.data_account_type_expenses","l10nse_chart_template","False"
"a5410","5410","Förbrukningsinventarier","account.data_account_type_expenses","l10nse_chart_template","False"
"a5420","5420","Programvaror","account.data_account_type_expenses","l10nse_chart_template","False"
"a5460","5460","Förbrukningsmaterial","account.data_account_type_expenses","l10nse_chart_template","False"
"a5480","5480","Arbetskläder","account.data_account_type_expenses","l10nse_chart_template","False"
"a5500","5500","Reparation och underhåll (gruppkonto)","account.data_account_type_expenses","l10nse_chart_template","False"
"a5600","5600","Kostnader för transportmedel (gruppkonto)","account.data_account_type_expenses","l10nse_chart_template","False"
"a5611","5611","Drivmedel för personbilar","account.data_account_type_expenses","l10nse_chart_template","False"
"a5612","5612","Försäkring och skatt för personbilar ","account.data_account_type_expenses","l10nse_chart_template","False"
"a5613","5613","Reparation och underhåll av personbilar","account.data_account_type_expenses","l10nse_chart_template","False"
"a5615","5615","Leasing av personbilar","account.data_account_type_expenses","l10nse_chart_template","False"
"a5616","5616","Trängselskatt","account.data_account_type_expenses","l10nse_chart_template","False"
"a5700","5700","Frakter och transporter (gruppkonto)","account.data_account_type_expenses","l10nse_chart_template","False"
"a5800","5800","Resekostnader (gruppkonto)","account.data_account_type_expenses","l10nse_chart_template","False"
"a5810","5810","Biljetter","account.data_account_type_expenses","l10nse_chart_template","False"
"a5820","5820","Hyrbilskostnader","account.data_account_type_expenses","l10nse_chart_template","False"
"a5831","5831","Kost och logi i Sverige","account.data_account_type_expenses","l10nse_chart_template","False"
"a5832","5832","Kost och logi i utlandet","account.data_account_type_expenses","l10nse_chart_template","False"
"a5900","5900","Reklam och PR","account.data_account_type_expenses","l10nse_chart_template","False"
"a6071","6071","Representation, avdragsgill","account.data_account_type_expenses","l10nse_chart_template","False"
"a6072","6072","Representation, ej avdragsgill","account.data_account_type_expenses","l10nse_chart_template","False"
"a6090","6090","Övriga försäljningskostnader ","account.data_account_type_expenses","l10nse_chart_template","False"
"a6100","6100","Kontorsmateriel och trycksaker (gruppkonto)","account.data_account_type_expenses","l10nse_chart_template","False"
"a6210","6210","Telekommunikation","account.data_account_type_expenses","l10nse_chart_template","False"
"a6230","6230","Datakommunikation","account.data_account_type_expenses","l10nse_chart_template","False"
"a6250","6250","Postbefordran","account.data_account_type_expenses","l10nse_chart_template","False"
"a6310","6310","Företagsförsäkringar","account.data_account_type_expenses","l10nse_chart_template","False"
"a6350","6350","Förluster på kundfordringar","account.data_account_type_expenses","l10nse_chart_template","False"
"a6390","6390","Övriga riskkostnader","account.data_account_type_expenses","l10nse_chart_template","False"
"a6410","6410","Styrelsearvoden som inte är lön","account.data_account_type_expenses","l10nse_chart_template","False"
"a6420","6420","Ersättningar till revisor","account.data_account_type_expenses","l10nse_chart_template","False"
"a6530","6530","Redovisningstjänster","account.data_account_type_expenses","l10nse_chart_template","False"
"a6540","6540","IT-tjänster","account.data_account_type_expenses","l10nse_chart_template","False"
"a6550","6550","Konsultarvoden","account.data_account_type_expenses","l10nse_chart_template","False"
"a6560","6560","Serviceavgifter till branschorganisationer","account.data_account_type_expenses","l10nse_chart_template","False"
"a6570","6570","Bankkostnader","account.data_account_type_expenses","l10nse_chart_template","False"
"a6580","6580","Advokat- och rättegångskostnader","account.data_account_type_expenses","l10nse_chart_template","False"
"a6590","6590","Övriga externa tjänster","account.data_account_type_expenses","l10nse_chart_template","False"
"a6800","6800","Inhyrd personal (gruppkonto)","account.data_account_type_expenses","l10nse_chart_template","False"
"a6970","6970","Tidningar, tidskrifter och facklitteratur","account.data_account_type_expenses","l10nse_chart_template","False"
"a6980","6980","Föreningsavgifter","account.data_account_type_expenses","l10nse_chart_template","False"
"a6991","6991","Övriga externa kostnader, avdragsgilla","account.data_account_type_expenses","l10nse_chart_template","False"
"a6992","6992","Övriga externa kostnader, ej avdragsgilla","account.data_account_type_expenses","l10nse_chart_template","False"
"a7010","7010","Löner till kollektivanställda","account.data_account_type_expenses","l10nse_chart_template","False"
"a7090","7090","Förändring av semesterlöneskuld","account.data_account_type_expenses","l10nse_chart_template","False"
"a7210","7210","Löner till tjänstemän","account.data_account_type_expenses","l10nse_chart_template","False"
"a7220","7220","Löner till företagsledare","account.data_account_type_expenses","l10nse_chart_template","False"
"a7240","7240","Styrelsearvoden","account.data_account_type_expenses","l10nse_chart_template","False"
"a7290","7290","Förändring av semesterlöneskuld","account.data_account_type_expenses","l10nse_chart_template","False"
"a7310","7310","Kontanta extraersättningar","account.data_account_type_expenses","l10nse_chart_template","False"
"a7321","7321","Skattefria traktamenten, Sverige","account.data_account_type_expenses","l10nse_chart_template","False"
"a7322","7322","Skattepliktiga traktamenten, Sverige","account.data_account_type_expenses","l10nse_chart_template","False"
"a7323","7323","Skattefria traktamenten, utlandet","account.data_account_type_expenses","l10nse_chart_template","False"
"a7324","7324","Skattepliktiga traktamenten, utlandet","account.data_account_type_expenses","l10nse_chart_template","False"
"a7331","7331","Skattefria bilersättningar","account.data_account_type_expenses","l10nse_chart_template","False"
"a7332","7332","Skattepliktiga bilersättningar","account.data_account_type_expenses","l10nse_chart_template","False"
"a7333","7333","Ersättning för trängselskatt, skattefri","account.data_account_type_expenses","l10nse_chart_template","False"
"a7380","7380","Kostnader förmånder till anställda","account.data_account_type_expenses","l10nse_chart_template","False"
"a7385","7385","Kostnader för fri bil","account.data_account_type_expenses","l10nse_chart_template","False"
"a7390","7390","Övriga kostnadsersättningar och förmåner","account.data_account_type_expenses","l10nse_chart_template","False"
"a7410","7410","Pensionsförsäkringspremier","account.data_account_type_expenses","l10nse_chart_template","False"
"a7490","7490","Övriga pensionskostnader","account.data_account_type_expenses","l10nse_chart_template","False"
"a7511","7511","Arbetsgivaravgift för löner och ersättningar","account.data_account_type_expenses","l10nse_chart_template","False"
"a7512","7512","Arbetsgivaravgifter för förmånsvärden","account.data_account_type_expenses","l10nse_chart_template","False"
"a7519","7519","Arbetsgivaravgifter för semester- och löneskulder","account.data_account_type_expenses","l10nse_chart_template","False"
"a7520","7520","Avgifter 16,36 %","account.data_account_type_expenses","l10nse_chart_template","False"
"a7530","7530","Särskild Löneskatt","account.data_account_type_expenses","l10nse_chart_template","False"
"a7550","7550","Avkastningsskatt på pensionsmedel","account.data_account_type_expenses","l10nse_chart_template","False"
"a7570","7570","Premier för arbetsmarknadsförsäkringar","account.data_account_type_expenses","l10nse_chart_template","False"
"a7580","7580","Gruppförsäkringspremier","account.data_account_type_expenses","l10nse_chart_template","False"
"a7590","7590","Övriga sociala och andra avgifter enligt lag och avtal","account.data_account_type_expenses","l10nse_chart_template","False"
"a7600","7600","Övriga personalkostnader (gruppkonto)","account.data_account_type_expenses","l10nse_chart_template","False"
"a7610","7610","Utbildning","account.data_account_type_expenses","l10nse_chart_template","False"
"a7621","7621","Sjuk- och hälsovård, avdragsgill","account.data_account_type_expenses","l10nse_chart_template","False"
"a7622","7622","Sjuk- och hälsovård, ej avdragsgill","account.data_account_type_expenses","l10nse_chart_template","False"
"a7631","7631","Personalrepresentation, avdragsgill","account.data_account_type_expenses","l10nse_chart_template","False"
"a7632","7632","Personalrepresentation, ej avdragsgill","account.data_account_type_expenses","l10nse_chart_template","False"
"a7720","7720","Nedskrivningar av byggnader och mark","account.data_account_type_expenses","l10nse_chart_template","False"
"a7730","7730","Nedskrivningar av maskiner och inventarier","account.data_account_type_expenses","l10nse_chart_template","False"
"a7810","7810","Avskrivningar av immateriella anläggningstillgångar","account.data_account_type_depreciation","l10nse_chart_template","False"
"a7820","7820","Avskrivningar på byggnader och markanläggningar","account.data_account_type_depreciation","l10nse_chart_template","False"
"a7830","7830","Avskrivningar maskiner och inventarier","account.data_account_type_depreciation","l10nse_chart_template","False"
"a7970","7970","Förlust vid avyttring av immateriella och materiella anläggningstillgångar","account.data_account_type_expenses","l10nse_chart_template","False"
"a7990","7990","Övriga rörelsekostnader","account.data_account_type_expenses","l10nse_chart_template","False"
"a8210","8210","Utdelningar på andelar i andra företag","account.data_account_type_other_income","l10nse_chart_template","False"
"a8220","8220","Resultat vid försäljning av värdepapper i och långfristiga fordringar hos andra företag","account.data_account_type_other_income","l10nse_chart_template","False"
"a8250","8250","Ränteintäkter från långfristiga fordringar hos och värdepapper i andra företag","account.data_account_type_other_income","l10nse_chart_template","False"
"a8270","8270","Nedskrivningar av innehav av andelar i och långfristiga fordringar hos andra företag","account.data_account_type_expenses","l10nse_chart_template","False"
"a8310","8310","Ränteintäkter från omsättningstillgångar","account.data_account_type_other_income","l10nse_chart_template","False"
"a8314","8314","Skattefria ränteintäkter","account.data_account_type_other_income","l10nse_chart_template","False"
"a8330","8330","Valutakursdifferenser på kortfristiga fordringar och placeringar","account.data_account_type_other_income","l10nse_chart_template","False"
"a8340","8340","Utdelningar på kortfristiga placeringar","account.data_account_type_other_income","l10nse_chart_template","False"
"a8350","8350","Resultat vid försäljning av kortfristiga placeringar","account.data_account_type_other_income","l10nse_chart_template","False"
"a8390","8390","Övriga finansiella intäkter","account.data_account_type_other_income","l10nse_chart_template","False"
"a8410","8410","Räntekostnader för långfristiga skulder","account.data_account_type_expenses","l10nse_chart_template","False"
"a8420","8420","Räntekostnader för kortfristiga skulder","account.data_account_type_expenses","l10nse_chart_template","False"
"a8422","8422","Dröjsmålsräntor för leverantörsskulder","account.data_account_type_expenses","l10nse_chart_template","False"
"a8423","8423","Räntekostnader för skatter och avgifter","account.data_account_type_expenses","l10nse_chart_template","False"
"a8430","8430","Valutakursdifferenser på skulder","account.data_account_type_expenses","l10nse_chart_template","False"
"a8811","8811","Avsättning till periodiseringsfond","account.data_account_type_expenses","l10nse_chart_template","False"
"a8819","8819","Återföring från periodiseringsfond","account.data_account_type_expenses","l10nse_chart_template","False"
"a8850","8850","Förändring av överavskrivningar","account.data_account_type_expenses","l10nse_chart_template","False"
"a8910","8910","Skatt som belastar årets resultat","account.data_account_type_expenses","l10nse_chart_template","False"
"a8990","8990","Resultat","account.data_account_type_expenses","l10nse_chart_template","False"
"a8999","8999","Årets resultat","account.data_account_type_expenses","l10nse_chart_template","False"

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
            <field name="name">Swedish Chart of Account</field>
            <field name="currency_id" ref="base.SEK"/>
            <field name="bank_account_code_prefix">193</field>
            <field name="cash_account_code_prefix">191</field>
            <field name="transfer_account_code_prefix">194</field>
            <field name="code_digits">4</field>
            <field name="complete_tax_set" eval="True"/>
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

## File: data\account_tax_group.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_25" model="account.tax.group">
            <field name="name">VAT 25%</field>
        </record>
        <record id="tax_group_12" model="account.tax.group">
            <field name="name">VAT 12%</field>
        </record>
        <record id="tax_group_6" model="account.tax.group">
            <field name="name">VAT 6%</field>
        </record>
        <record id="tax_group_0" model="account.tax.group">
            <field name="name">VAT 0%</field>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <!-- -->
        <!--A-->
        <!-- -->
        <record id="tax_report_title_sales" model="account.tax.report.line">
            <field name="name">Block A – Momspliktig försäljning eller uttag exklusive moms</field>
            <field name="code">se_a</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">1</field>
        </record>

        <record id="tax_report_line_05" model="account.tax.report.line">
            <field name="name">Fält 05 – Momspliktig försäljning som inte ingår i fält 06, 07 eller 08</field>
            <field name="code">se_05</field>
            <field name="tag_name">se_05</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_sales"/>
        </record>

        <record id="tax_report_line_06" model="account.tax.report.line">
            <field name="name">Fält 06 – Momspliktiga uttag</field>
            <field name="code">se_06</field>
            <field name="tag_name">se_06</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_sales"/>
        </record>

        <record id="tax_report_line_07" model="account.tax.report.line">
            <field name="name">Fält 07 – Beskattningsunderlag vid vinstmarginalbeskattning</field>
            <field name="code">se_07</field>
            <field name="tag_name">se_07</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">3</field>
            <field name="parent_id" ref="tax_report_title_sales"/>
        </record>

        <record id="tax_report_line_08" model="account.tax.report.line">
            <field name="name">Fält 08 – Hyresinkomster vid frivillig skattskyldighet</field>
            <field name="code">se_08</field>
            <field name="tag_name">se_08</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">4</field>
            <field name="parent_id" ref="tax_report_title_sales"/>
        </record>

        <!-- -->
        <!--B-->
        <!-- -->
        <record id="tax_report_title_output_vat_sales" model="account.tax.report.line">
            <field name="name">Block B – Utgående moms på försäljning eller uttag i fält 05–08</field>
            <field name="code">se_b</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">2</field>
        </record>

        <record id="tax_report_line_10" model="account.tax.report.line">
            <field name="name">Fält 10 – Utgående moms 25 %</field>
            <field name="code">se_10</field>
            <field name="tag_name">se_10</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_output_vat_sales"/>
        </record>

        <record id="tax_report_line_11" model="account.tax.report.line">
            <field name="name">Fält 11 – Utgående moms 12 %</field>
            <field name="code">se_11</field>
            <field name="tag_name">se_11</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_output_vat_sales"/>
        </record>

        <record id="tax_report_line_12" model="account.tax.report.line">
            <field name="name">Fält 12 – Utgående moms 6 %</field>
            <field name="code">se_12</field>
            <field name="tag_name">se_12</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">3</field>
            <field name="parent_id" ref="tax_report_title_output_vat_sales"/>
        </record>

        <!-- -->
        <!--C-->
        <!-- -->
        <record id="tax_report_title_purchases" model="account.tax.report.line">
            <field name="name">Block C – Momspliktiga inköp vid omvänd skattskyldighet</field>
            <field name="code">se_c</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">3</field>
        </record>

        <record id="tax_report_line_20" model="account.tax.report.line">
            <field name="name">Fält 20 – Inköp av varor från annat EU-land</field>
            <field name="code">se_20</field>
            <field name="tag_name">se_20</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_purchases"/>
        </record>

        <record id="tax_report_line_21" model="account.tax.report.line">
            <field name="name">Fält 21 – Inköp av tjänster från ett annat EU-land, enligt huvudregeln</field>
            <field name="code">se_21</field>
            <field name="tag_name">se_21</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_purchases"/>
        </record>

        <record id="tax_report_line_22" model="account.tax.report.line">
            <field name="name">Fält 22 – Inköp av tjänster från länder utanför EU</field>
            <field name="code">se_22</field>
            <field name="tag_name">se_22</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">3</field>
            <field name="parent_id" ref="tax_report_title_purchases"/>
        </record>

        <record id="tax_report_line_23" model="account.tax.report.line">
            <field name="name">Fält 23 – Inköp av varor i Sverige</field>
            <field name="code">se_23</field>
            <field name="tag_name">se_23</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">3</field>
            <field name="parent_id" ref="tax_report_title_purchases"/>
        </record>

        <record id="tax_report_line_24" model="account.tax.report.line">
            <field name="name">Fält 24 – Övriga inköp av tjänster</field>
            <field name="code">se_24</field>
            <field name="tag_name">se_24</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">4</field>
            <field name="parent_id" ref="tax_report_title_purchases"/>
        </record>

        <!-- -->
        <!--D-->
        <!-- -->
        <record id="tax_report_title_output_vat_purchases" model="account.tax.report.line">
            <field name="name">Block D – Utgående moms på inköp i fält 20–24</field>
            <field name="code">se_d</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">4</field>
        </record>

        <record id="tax_report_line_30" model="account.tax.report.line">
            <field name="name">Fält 30 – Utgående moms 25 %</field>
            <field name="code">se_30</field>
            <field name="tag_name">se_30</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_output_vat_purchases"/>
        </record>

        <record id="tax_report_line_31" model="account.tax.report.line">
            <field name="name">Fält 31 – Utgående moms 12 %</field>
            <field name="code">se_31</field>
            <field name="tag_name">se_31</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_output_vat_purchases"/>
        </record>

        <record id="tax_report_line_32" model="account.tax.report.line">
            <field name="name">Fält 32 – Utgående moms 6 %</field>
            <field name="code">se_32</field>
            <field name="tag_name">se_32</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">3</field>
            <field name="parent_id" ref="tax_report_title_output_vat_purchases"/>
        </record>

        <!-- -->
        <!--H-->
        <!-- -->
        <record id="tax_report_title_imports" model="account.tax.report.line">
            <field name="name">Block H - moms vid import</field>
            <field name="code">se_h</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">5</field>
        </record>

        <record id="tax_report_line_50" model="account.tax.report.line">
            <field name="name">Fält 50 - Beskattningsunderlag vid import</field>
            <field name="code">se_50</field>
            <field name="tag_name">se_50</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_imports"/>
        </record>

        <!-- -->
        <!--I-->
        <!-- -->
        <record id="tax_report_title_output_vat_imports" model="account.tax.report.line">
            <field name="name">Block I - Utgående moms på import i fält 50</field>
            <field name="code">se_i</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">6</field>
        </record>

        <record id="tax_report_line_60" model="account.tax.report.line">
            <field name="name">Fält 60 – Utgående moms 25 %</field>
            <field name="code">se_60</field>
            <field name="tag_name">se_60</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_output_vat_imports"/>
        </record>

        <record id="tax_report_line_61" model="account.tax.report.line">
            <field name="name">Fält 61 – Utgående moms 12 %</field>
            <field name="code">se_61</field>
            <field name="tag_name">se_61</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_output_vat_imports"/>
        </record>

        <record id="tax_report_line_62" model="account.tax.report.line">
            <field name="name">Fält 62 – Utgående moms 6 %</field>
            <field name="code">se_62</field>
            <field name="tag_name">se_62</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">3</field>
            <field name="parent_id" ref="tax_report_title_output_vat_imports"/>
        </record>

        <!-- -->
        <!--E-->
        <!-- -->
        <record id="tax_report_title_exempt_sales" model="account.tax.report.line">
            <field name="name">Block E – Försäljning m.m. som är undantagen från moms</field>
            <field name="code">se_e</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">7</field>
        </record>

        <record id="tax_report_line_35" model="account.tax.report.line">
            <field name="name">Fält 35 – Försäljning av varor till ett annat EU-land</field>
            <field name="code">se_35</field>
            <field name="tag_name">se_35</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_exempt_sales"/>
        </record>

        <record id="tax_report_line_36" model="account.tax.report.line">
            <field name="name">Fält 36 – Försäljning av varor utanför EU</field>
            <field name="code">se_36</field>
            <field name="tag_name">se_36</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">2</field>
            <field name="parent_id" ref="tax_report_title_exempt_sales"/>
        </record>

        <record id="tax_report_line_37" model="account.tax.report.line">
            <field name="name">Fält 37 – Mellanmans inköp av varor vid trepartshandel</field>
            <field name="code">se_37</field>
            <field name="tag_name">se_37</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">3</field>
            <field name="parent_id" ref="tax_report_title_exempt_sales"/>
        </record>

        <record id="tax_report_line_38" model="account.tax.report.line">
            <field name="name">Fält 38 – Mellanmans försäljning av varor vid trepartshandel</field>
            <field name="code">se_38</field>
            <field name="tag_name">se_38</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">4</field>
            <field name="parent_id" ref="tax_report_title_exempt_sales"/>
        </record>

        <record id="tax_report_line_39" model="account.tax.report.line">
            <field name="name">Fält 39 – Försäljning av tjänster till en beskattningsbar person (näringsidkare) i ett
                annat EU-land, enligt huvudregeln
            </field>
            <field name="code">se_39</field>
            <field name="tag_name">se_39</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">5</field>
            <field name="parent_id" ref="tax_report_title_exempt_sales"/>
        </record>

        <record id="tax_report_line_40" model="account.tax.report.line">
            <field name="name">Fält 40 – Övrig försäljning av tjänster omsatta utanför Sverige</field>
            <field name="code">se_40</field>
            <field name="tag_name">se_40</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">6</field>
            <field name="parent_id" ref="tax_report_title_exempt_sales"/>
        </record>

        <record id="tax_report_line_41" model="account.tax.report.line">
            <field name="name">Fält 41 – Försäljning när köparen är skattskyldig i Sverige</field>
            <field name="code">se_41</field>
            <field name="tag_name">se_41</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">7</field>
            <field name="parent_id" ref="tax_report_title_exempt_sales"/>
        </record>

        <record id="tax_report_line_42" model="account.tax.report.line">
            <field name="name">Fält 42 – Övrig försäljning m.m.</field>
            <field name="code">se_42</field>
            <field name="tag_name">se_42</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">8</field>
            <field name="parent_id" ref="tax_report_title_exempt_sales"/>
        </record>

        <!-- -->
        <!--F-->
        <!-- -->
        <record id="tax_report_title_input_vat" model="account.tax.report.line">
            <field name="name">Block F – Ingående moms</field>
            <field name="code">se_f</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">8</field>
        </record>

        <record id="tax_report_line_48" model="account.tax.report.line">
            <field name="name">Fält 48 – Ingående moms att dra av</field>
            <field name="code">se_48</field>
            <field name="tag_name">se_48</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_input_vat"/>
        </record>

        <!-- -->
        <!--G-->
        <!-- -->
        <record id="tax_report_title_vat_debt_credit" model="account.tax.report.line">
            <field name="name">Block G – Moms att betala eller få tillbaka</field>
            <field name="code">se_g</field>
            <field name="formula">se_b+se_i+se_d+se_f</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">9</field>
        </record>

        <record id="tax_report_line_49" model="account.tax.report.line">
            <field name="name">Fält 49 – Moms att betala eller få tillbaka</field>
            <field name="code">se_49</field>
            <field name="formula">se_b+se_i+se_d+se_f</field>
            <field name="country_id" ref="base.se"/>
            <field name="sequence">1</field>
            <field name="parent_id" ref="tax_report_title_vat_debt_credit"/>
        </record>

    </data>
</odoo>

```

## File: data\account_tax_template.xml

```xml
<?xml version='1.0' encoding='UTF-8'?>
<odoo>
    <data noupdate="1">
        <record id="sale_tax_25_goods" model="account.tax.template">
            <field name="chart_template_id" ref="l10nse_chart_template"/>
            <field name="name">Utgående moms 25%</field>
            <field name="description">ST25</field>
            <field name="amount">25</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_25"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_05')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2611'),
                    'plus_report_line_ids': [ref('tax_report_line_10')]
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_05')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2611'),
                    'minus_report_line_ids': [ref('tax_report_line_10')]
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_05')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2611'),
                    'plus_report_line_ids': [ref('tax_report_line_10')]
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_05')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2611'),
                    'minus_report_line_ids': [ref('tax_report_line_10')]
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
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'plus_report_line_ids': [ref('tax_report_line_48')]
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
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'plus_report_line_ids': [ref('tax_report_line_48')]
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_05')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2621'),
                    'plus_report_line_ids': [ref('tax_report_line_11')]
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_05')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2621'),
                    'minus_report_line_ids': [ref('tax_report_line_11')]
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_05')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2621'),
                    'plus_report_line_ids': [ref('tax_report_line_11')]
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_05')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2621'),
                    'minus_report_line_ids': [ref('tax_report_line_11')]
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
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'plus_report_line_ids': [ref('tax_report_line_48')]
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
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'plus_report_line_ids': [ref('tax_report_line_48')]
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_05')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2631'),
                    'plus_report_line_ids': [ref('tax_report_line_12')]
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_05')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2631'),
                    'minus_report_line_ids': [ref('tax_report_line_12')]
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_05')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2631'),
                    'plus_report_line_ids': [ref('tax_report_line_12')]
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_05')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2631'),
                    'minus_report_line_ids': [ref('tax_report_line_12')]
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
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'plus_report_line_ids': [ref('tax_report_line_48')]
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
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2641'),
                    'plus_report_line_ids': [ref('tax_report_line_48')]
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_39')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax'
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_39')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax'
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_35')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax'
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_35')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax'
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_20')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_line_ids': [ref('tax_report_line_30')],
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_20')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_line_ids': [ref('tax_report_line_30')],
                    'plus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_20')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_line_ids': [ref('tax_report_line_31')],
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_20')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_line_ids': [ref('tax_report_line_31')],
                    'plus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_20')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_line_ids': [ref('tax_report_line_32')],
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_20')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_line_ids': [ref('tax_report_line_32')],
                    'plus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_21')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_line_ids': [ref('tax_report_line_30')],
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_21')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_line_ids': [ref('tax_report_line_30')],
                    'plus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_21')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_line_ids': [ref('tax_report_line_31')],
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_21')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_line_ids': [ref('tax_report_line_31')],
                    'plus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_21')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_line_ids': [ref('tax_report_line_32')],
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_21')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_line_ids': [ref('tax_report_line_32')],
                    'plus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_24')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2647'),
                    'plus_report_line_ids': [ref('tax_report_line_30')],
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_24')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2647'),
                    'minus_report_line_ids': [ref('tax_report_line_30')],
                    'plus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_24')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a4426'),
                    'plus_report_line_ids': [ref('tax_report_line_31')],
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_24')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a4426'),
                    'minus_report_line_ids': [ref('tax_report_line_31')],
                    'plus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_24')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a4427'),
                    'plus_report_line_ids': [ref('tax_report_line_32')],
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_24')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a4427'),
                    'minus_report_line_ids': [ref('tax_report_line_32')],
                    'plus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_39')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax'
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_39')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax'
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_36')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax'
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_36')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax'
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_50')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_line_ids': [ref('tax_report_line_60')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2615')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_50')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_line_ids': [ref('tax_report_line_60')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2615')
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_50')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_line_ids': [ref('tax_report_line_61')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2625')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_50')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_line_ids': [ref('tax_report_line_61')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2625')
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_50')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_line_ids': [ref('tax_report_line_62')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2635')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_50')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_line_ids': [ref('tax_report_line_62')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2635')
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_22')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_line_ids': [ref('tax_report_line_30')],
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_22')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_line_ids': [ref('tax_report_line_30')],
                    'plus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2614')
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_22')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_line_ids': [ref('tax_report_line_31')],
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_22')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_line_ids': [ref('tax_report_line_31')],
                    'plus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2624')
                })]"/>
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
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_line_22')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'plus_report_line_ids': [ref('tax_report_line_32')],
                    'minus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')
                })]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_line_22')]
                }),
                (0, 0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2645'),
                    'minus_report_line_ids': [ref('tax_report_line_32')],
                    'plus_report_line_ids': [ref('tax_report_line_48')]
                }),
                (0, 0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('a2634')
                })]"/>
        </record>
    </data>
</odoo>

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
            if company.country_id == self.env.ref('base.se') and company.vat:
                org_number = re.sub(r'\D', '', company.vat)[:-2]
                org_number = org_number[:6] + '-' + org_number[6:]

                company.org_number = org_number
            else:
                company.org_number = ''

```

## File: models\__init__.py

```python
from . import res_company

```

