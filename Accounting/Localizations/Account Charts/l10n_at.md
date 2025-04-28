# Odoo Module: l10n_at

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Austria - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['at'],
    'version': '3.2.1',
    'author': 'WT-IO-IT GmbH, Wolfgang Taferner (https://www.wt-io-it.at)',
    'website': 'https://www.odoo.com/documentation/master/applications/finance/fiscal_localizations.html',
    'category': 'Accounting/Localizations/Account Charts',
    'summary': 'Austrian Standardized Charts & Tax',
    'description': """

Austrian charts of accounts (Einheitskontenrahmen 2010).
==========================================================

    * Defines the following chart of account templates:
        * Austrian General Chart of accounts 2010
    * Defines templates for VAT on sales and purchases
    * Defines tax templates
    * Defines fiscal positions for Austrian fiscal legislation
    * Defines tax reports U1/U30

    """,
    'depends': [
        'account',
        'base_iban',
        'base_vat',
        'l10n_din5008',
    ],
    'auto_install': ['account'],
    'data': [
        'data/res.country.state.csv',
        'data/account_account_tag.xml',
        'data/account.account.tag.csv',
        'data/account_tax_report_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.tag.csv

```csv
"id","name","applicability","country_id/id"
"account_tag_external_code_0010","0010","accounts","base.at"
"account_tag_external_code_0011","0011","accounts","base.at"
"account_tag_external_code_0019","0019","accounts","base.at"
"account_tag_external_code_0020","0020","accounts","base.at"
"account_tag_external_code_0021","0021","accounts","base.at"
"account_tag_external_code_0029","0029","accounts","base.at"
"account_tag_external_code_0100","0100","accounts","base.at"
"account_tag_external_code_0101","0101","accounts","base.at"
"account_tag_external_code_0109","0109","accounts","base.at"
"account_tag_external_code_0110","0110","accounts","base.at"
"account_tag_external_code_0111","0111","accounts","base.at"
"account_tag_external_code_0112","0112","accounts","base.at"
"account_tag_external_code_0113","0113","accounts","base.at"
"account_tag_external_code_0119","0119","accounts","base.at"
"account_tag_external_code_0120","0120","accounts","base.at"
"account_tag_external_code_0121","0121","accounts","base.at"
"account_tag_external_code_0129","0129","accounts","base.at"
"account_tag_external_code_0130","0130","accounts","base.at"
"account_tag_external_code_0131","0131","accounts","base.at"
"account_tag_external_code_0132","0132","accounts","base.at"
"account_tag_external_code_0133","0133","accounts","base.at"
"account_tag_external_code_0134","0134","accounts","base.at"
"account_tag_external_code_0140","0140","accounts","base.at"
"account_tag_external_code_0141","0141","accounts","base.at"
"account_tag_external_code_0149","0149","accounts","base.at"
"account_tag_external_code_0150","0150","accounts","base.at"
"account_tag_external_code_0151","0151","accounts","base.at"
"account_tag_external_code_0159","0159","accounts","base.at"
"account_tag_external_code_0160","0160","accounts","base.at"
"account_tag_external_code_0169","0169","accounts","base.at"
"account_tag_external_code_0170","0170","accounts","base.at"
"account_tag_external_code_0179","0179","accounts","base.at"
"account_tag_external_code_0180","0180","accounts","base.at"
"account_tag_external_code_0200","0200","accounts","base.at"
"account_tag_external_code_0205","0205","accounts","base.at"
"account_tag_external_code_0210","0210","accounts","base.at"
"account_tag_external_code_0219","0219","accounts","base.at"
"account_tag_external_code_0220","0220","accounts","base.at"
"account_tag_external_code_0229","0229","accounts","base.at"
"account_tag_external_code_0300","0300","accounts","base.at"
"account_tag_external_code_0301","0301","accounts","base.at"
"account_tag_external_code_0310","0310","accounts","base.at"
"account_tag_external_code_0311","0311","accounts","base.at"
"account_tag_external_code_0320","0320","accounts","base.at"
"account_tag_external_code_0321","0321","accounts","base.at"
"account_tag_external_code_0330","0330","accounts","base.at"
"account_tag_external_code_0331","0331","accounts","base.at"
"account_tag_external_code_0340","0340","accounts","base.at"
"account_tag_external_code_0341","0341","accounts","base.at"
"account_tag_external_code_0349","0349","accounts","base.at"
"account_tag_external_code_0350","0350","accounts","base.at"
"account_tag_external_code_0351","0351","accounts","base.at"
"account_tag_external_code_0359","0359","accounts","base.at"
"account_tag_external_code_0360","0360","accounts","base.at"
"account_tag_external_code_0361","0361","accounts","base.at"
"account_tag_external_code_0370","0370","accounts","base.at"
"account_tag_external_code_0371","0371","accounts","base.at"
"account_tag_external_code_0379","0379","accounts","base.at"
"account_tag_external_code_0400","0400","accounts","base.at"
"account_tag_external_code_0401","0401","accounts","base.at"
"account_tag_external_code_0410","0410","accounts","base.at"
"account_tag_external_code_0411","0411","accounts","base.at"
"account_tag_external_code_0420","0420","accounts","base.at"
"account_tag_external_code_0421","0421","accounts","base.at"
"account_tag_external_code_0430","0430","accounts","base.at"
"account_tag_external_code_0431","0431","accounts","base.at"
"account_tag_external_code_0439","0439","accounts","base.at"
"account_tag_external_code_0440","0440","accounts","base.at"
"account_tag_external_code_0441","0441","accounts","base.at"
"account_tag_external_code_0500","0500","accounts","base.at"
"account_tag_external_code_0501","0501","accounts","base.at"
"account_tag_external_code_0510","0510","accounts","base.at"
"account_tag_external_code_0511","0511","accounts","base.at"
"account_tag_external_code_0520","0520","accounts","base.at"
"account_tag_external_code_0521","0521","accounts","base.at"
"account_tag_external_code_0530","0530","accounts","base.at"
"account_tag_external_code_0531","0531","accounts","base.at"
"account_tag_external_code_0540","0540","accounts","base.at"
"account_tag_external_code_0541","0541","accounts","base.at"
"account_tag_external_code_0550","0550","accounts","base.at"
"account_tag_external_code_0551","0551","accounts","base.at"
"account_tag_external_code_0555","0555","accounts","base.at"
"account_tag_external_code_0556","0556","accounts","base.at"
"account_tag_external_code_0559","0559","accounts","base.at"
"account_tag_external_code_0600","0600","accounts","base.at"
"account_tag_external_code_0601","0601","accounts","base.at"
"account_tag_external_code_0605","0605","accounts","base.at"
"account_tag_external_code_0606","0606","accounts","base.at"
"account_tag_external_code_0610","0610","accounts","base.at"
"account_tag_external_code_0611","0611","accounts","base.at"
"account_tag_external_code_0612","0612","accounts","base.at"
"account_tag_external_code_0613","0613","accounts","base.at"
"account_tag_external_code_0620","0620","accounts","base.at"
"account_tag_external_code_0621","0621","accounts","base.at"
"account_tag_external_code_0630","0630","accounts","base.at"
"account_tag_external_code_0631","0631","accounts","base.at"
"account_tag_external_code_0640","0640","accounts","base.at"
"account_tag_external_code_0641","0641","accounts","base.at"
"account_tag_external_code_0650","0650","accounts","base.at"
"account_tag_external_code_0651","0651","accounts","base.at"
"account_tag_external_code_0655","0655","accounts","base.at"
"account_tag_external_code_0659","0659","accounts","base.at"
"account_tag_external_code_0660","0660","accounts","base.at"
"account_tag_external_code_0661","0661","accounts","base.at"
"account_tag_external_code_0670","0670","accounts","base.at"
"account_tag_external_code_0671","0671","accounts","base.at"
"account_tag_external_code_0680","0680","accounts","base.at"
"account_tag_external_code_0681","0681","accounts","base.at"
"account_tag_external_code_0685","0685","accounts","base.at"
"account_tag_external_code_0686","0686","accounts","base.at"
"account_tag_external_code_0689","0689","accounts","base.at"
"account_tag_external_code_0700","0700","accounts","base.at"
"account_tag_external_code_0701","0701","accounts","base.at"
"account_tag_external_code_0702","0702","accounts","base.at"
"account_tag_external_code_0710","0710","accounts","base.at"
"account_tag_external_code_0711","0711","accounts","base.at"
"account_tag_external_code_0780","0780","accounts","base.at"
"account_tag_external_code_0781","0781","accounts","base.at"
"account_tag_external_code_0800","0800","accounts","base.at"
"account_tag_external_code_0810","0810","accounts","base.at"
"account_tag_external_code_0820","0820","accounts","base.at"
"account_tag_external_code_0830","0830","accounts","base.at"
"account_tag_external_code_0840","0840","accounts","base.at"
"account_tag_external_code_0850","0850","accounts","base.at"
"account_tag_external_code_0860","0860","accounts","base.at"
"account_tag_external_code_0862","0862","accounts","base.at"
"account_tag_external_code_0867","0867","accounts","base.at"
"account_tag_external_code_0870","0870","accounts","base.at"
"account_tag_external_code_0880","0880","accounts","base.at"
"account_tag_external_code_0900","0900","accounts","base.at"
"account_tag_external_code_0910","0910","accounts","base.at"
"account_tag_external_code_0920","0920","accounts","base.at"
"account_tag_external_code_0940","0940","accounts","base.at"
"account_tag_external_code_0950","0950","accounts","base.at"
"account_tag_external_code_0959","0959","accounts","base.at"
"account_tag_external_code_0980","0980","accounts","base.at"
"account_tag_external_code_0990","0990","accounts","base.at"
"account_tag_external_code_0995","0995","accounts","base.at"
"account_tag_external_code_0996","0996","accounts","base.at"
"account_tag_external_code_0997","0997","accounts","base.at"
"account_tag_external_code_1000","1000","accounts","base.at"
"account_tag_external_code_1001","1001","accounts","base.at"
"account_tag_external_code_1100","1100","accounts","base.at"
"account_tag_external_code_1101","1101","accounts","base.at"
"account_tag_external_code_1200","1200","accounts","base.at"
"account_tag_external_code_1201","1201","accounts","base.at"
"account_tag_external_code_1300","1300","accounts","base.at"
"account_tag_external_code_1301","1301","accounts","base.at"
"account_tag_external_code_1350","1350","accounts","base.at"
"account_tag_external_code_1351","1351","accounts","base.at"
"account_tag_external_code_1360","1360","accounts","base.at"
"account_tag_external_code_1361","1361","accounts","base.at"
"account_tag_external_code_1400","1400","accounts","base.at"
"account_tag_external_code_1401","1401","accounts","base.at"
"account_tag_external_code_1500","1500","accounts","base.at"
"account_tag_external_code_1501","1501","accounts","base.at"
"account_tag_external_code_1600","1600","accounts","base.at"
"account_tag_external_code_1601","1601","accounts","base.at"
"account_tag_external_code_1700","1700","accounts","base.at"
"account_tag_external_code_1800","1800","accounts","base.at"
"account_tag_external_code_1801","1801","accounts","base.at"
"account_tag_external_code_1803","1803","accounts","base.at"
"account_tag_external_code_1910","1910","accounts","base.at"
"account_tag_external_code_1920","1920","accounts","base.at"
"account_tag_external_code_1930","1930","accounts","base.at"
"account_tag_external_code_1940","1940","accounts","base.at"
"account_tag_external_code_1950","1950","accounts","base.at"
"account_tag_external_code_1960","1960","accounts","base.at"
"account_tag_external_code_1970","1970","accounts","base.at"
"account_tag_external_code_2000","2000","accounts","base.at"
"account_tag_external_code_2080","2080","accounts","base.at"
"account_tag_external_code_2090","2090","accounts","base.at"
"account_tag_external_code_2100","2100","accounts","base.at"
"account_tag_external_code_2130","2130","accounts","base.at"
"account_tag_external_code_2140","2140","accounts","base.at"
"account_tag_external_code_2150","2150","accounts","base.at"
"account_tag_external_code_2180","2180","accounts","base.at"
"account_tag_external_code_2190","2190","accounts","base.at"
"account_tag_external_code_2200","2200","accounts","base.at"
"account_tag_external_code_2201","2201","accounts","base.at"
"account_tag_external_code_2230","2230","accounts","base.at"
"account_tag_external_code_2240","2240","accounts","base.at"
"account_tag_external_code_2250","2250","accounts","base.at"
"account_tag_external_code_2251","2251","accounts","base.at"
"account_tag_external_code_2280","2280","accounts","base.at"
"account_tag_external_code_2290","2290","accounts","base.at"
"account_tag_external_code_2291","2291","accounts","base.at"
"account_tag_external_code_2292","2292","accounts","base.at"
"account_tag_external_code_2293","2293","accounts","base.at"
"account_tag_external_code_2294","2294","accounts","base.at"
"account_tag_external_code_2295","2295","accounts","base.at"
"account_tag_external_code_2300","2300","accounts","base.at"
"account_tag_external_code_2310","2310","accounts","base.at"
"account_tag_external_code_2320","2320","accounts","base.at"
"account_tag_external_code_2330","2330","accounts","base.at"
"account_tag_external_code_2340","2340","accounts","base.at"
"account_tag_external_code_2350","2350","accounts","base.at"
"account_tag_external_code_2470","2470","accounts","base.at"
"account_tag_external_code_2480","2480","accounts","base.at"
"account_tag_external_code_2490","2490","accounts","base.at"
"account_tag_external_code_2500","2500","accounts","base.at"
"account_tag_external_code_2501","2501","accounts","base.at"
"account_tag_external_code_2502","2502","accounts","base.at"
"account_tag_external_code_2504","2504","accounts","base.at"
"account_tag_external_code_2505","2505","accounts","base.at"
"account_tag_external_code_2506","2506","accounts","base.at"
"account_tag_external_code_2507","2507","accounts","base.at"
"account_tag_external_code_2509","2509","accounts","base.at"
"account_tag_external_code_2510","2510","accounts","base.at"
"account_tag_external_code_2511","2511","accounts","base.at"
"account_tag_external_code_2512","2512","accounts","base.at"
"account_tag_external_code_2513","2513","accounts","base.at"
"account_tag_external_code_2515","2515","accounts","base.at"
"account_tag_external_code_2517","2517","accounts","base.at"
"account_tag_external_code_2518","2518","accounts","base.at"
"account_tag_external_code_2519","2519","accounts","base.at"
"account_tag_external_code_2520","2520","accounts","base.at"
"account_tag_external_code_2530","2530","accounts","base.at"
"account_tag_external_code_2531","2531","accounts","base.at"
"account_tag_external_code_2532","2532","accounts","base.at"
"account_tag_external_code_2533","2533","accounts","base.at"
"account_tag_external_code_2534","2534","accounts","base.at"
"account_tag_external_code_2535","2535","accounts","base.at"
"account_tag_external_code_2536","2536","accounts","base.at"
"account_tag_external_code_2537","2537","accounts","base.at"
"account_tag_external_code_2538","2538","accounts","base.at"
"account_tag_external_code_2540","2540","accounts","base.at"
"account_tag_external_code_2541","2541","accounts","base.at"
"account_tag_external_code_2542","2542","accounts","base.at"
"account_tag_external_code_2543","2543","accounts","base.at"
"account_tag_external_code_2560","2560","accounts","base.at"
"account_tag_external_code_2565","2565","accounts","base.at"
"account_tag_external_code_2570","2570","accounts","base.at"
"account_tag_external_code_2573","2573","accounts","base.at"
"account_tag_external_code_2575","2575","accounts","base.at"
"account_tag_external_code_2577","2577","accounts","base.at"
"account_tag_external_code_2580","2580","accounts","base.at"
"account_tag_external_code_2582","2582","accounts","base.at"
"account_tag_external_code_2584","2584","accounts","base.at"
"account_tag_external_code_2586","2586","accounts","base.at"
"account_tag_external_code_2590","2590","accounts","base.at"
"account_tag_external_code_2600","2600","accounts","base.at"
"account_tag_external_code_2601","2601","accounts","base.at"
"account_tag_external_code_2602","2602","accounts","base.at"
"account_tag_external_code_2610","2610","accounts","base.at"
"account_tag_external_code_2612","2612","accounts","base.at"
"account_tag_external_code_2620","2620","accounts","base.at"
"account_tag_external_code_2630","2630","accounts","base.at"
"account_tag_external_code_2680","2680","accounts","base.at"
"account_tag_external_code_2690","2690","accounts","base.at"
"account_tag_external_code_2700","2700","accounts","base.at"
"account_tag_external_code_2730","2730","accounts","base.at"
"account_tag_external_code_2740","2740","accounts","base.at"
"account_tag_external_code_2750","2750","accounts","base.at"
"account_tag_external_code_2780","2780","accounts","base.at"
"account_tag_external_code_2790","2790","accounts","base.at"
"account_tag_external_code_2800","2800","accounts","base.at"
"account_tag_external_code_2880","2880","accounts","base.at"
"account_tag_external_code_2885","2885","accounts","base.at"
"account_tag_external_code_2890","2890","accounts","base.at"
"account_tag_external_code_2900","2900","accounts","base.at"
"account_tag_external_code_2940","2940","accounts","base.at"
"account_tag_external_code_2950","2950","accounts","base.at"
"account_tag_external_code_2960","2960","accounts","base.at"
"account_tag_external_code_2970","2970","accounts","base.at"
"account_tag_external_code_2980","2980","accounts","base.at"
"account_tag_external_code_2990","2990","accounts","base.at"
"account_tag_external_code_2999","2999","accounts","base.at"
"account_tag_external_code_3000","3000","accounts","base.at"
"account_tag_external_code_3010","3010","accounts","base.at"
"account_tag_external_code_3020","3020","accounts","base.at"
"account_tag_external_code_3030","3030","accounts","base.at"
"account_tag_external_code_3035","3035","accounts","base.at"
"account_tag_external_code_3040","3040","accounts","base.at"
"account_tag_external_code_3041","3041","accounts","base.at"
"account_tag_external_code_3042","3042","accounts","base.at"
"account_tag_external_code_3043","3043","accounts","base.at"
"account_tag_external_code_3044","3044","accounts","base.at"
"account_tag_external_code_3050","3050","accounts","base.at"
"account_tag_external_code_3051","3051","accounts","base.at"
"account_tag_external_code_3052","3052","accounts","base.at"
"account_tag_external_code_3060","3060","accounts","base.at"
"account_tag_external_code_3061","3061","accounts","base.at"
"account_tag_external_code_3070","3070","accounts","base.at"
"account_tag_external_code_3071","3071","accounts","base.at"
"account_tag_external_code_3080","3080","accounts","base.at"
"account_tag_external_code_3085","3085","accounts","base.at"
"account_tag_external_code_3090","3090","accounts","base.at"
"account_tag_external_code_3100","3100","accounts","base.at"
"account_tag_external_code_3105","3105","accounts","base.at"
"account_tag_external_code_3110","3110","accounts","base.at"
"account_tag_external_code_3180","3180","accounts","base.at"
"account_tag_external_code_3200","3200","accounts","base.at"
"account_tag_external_code_3201","3201","accounts","base.at"
"account_tag_external_code_3202","3202","accounts","base.at"
"account_tag_external_code_3203","3203","accounts","base.at"
"account_tag_external_code_3204","3204","accounts","base.at"
"account_tag_external_code_3205","3205","accounts","base.at"
"account_tag_external_code_3210","3210","accounts","base.at"
"account_tag_external_code_3300","3300","accounts","base.at"
"account_tag_external_code_3360","3360","accounts","base.at"
"account_tag_external_code_3370","3370","accounts","base.at"
"account_tag_external_code_3380","3380","accounts","base.at"
"account_tag_external_code_3390","3390","accounts","base.at"
"account_tag_external_code_3400","3400","accounts","base.at"
"account_tag_external_code_3401","3401","accounts","base.at"
"account_tag_external_code_3440","3440","accounts","base.at"
"account_tag_external_code_3441","3441","accounts","base.at"
"account_tag_external_code_3450","3450","accounts","base.at"
"account_tag_external_code_3455","3455","accounts","base.at"
"account_tag_external_code_3460","3460","accounts","base.at"
"account_tag_external_code_3470","3470","accounts","base.at"
"account_tag_external_code_3480","3480","accounts","base.at"
"account_tag_external_code_3481","3481","accounts","base.at"
"account_tag_external_code_3485","3485","accounts","base.at"
"account_tag_external_code_3486","3486","accounts","base.at"
"account_tag_external_code_3490","3490","accounts","base.at"
"account_tag_external_code_3491","3491","accounts","base.at"
"account_tag_external_code_3492","3492","accounts","base.at"
"account_tag_external_code_3493","3493","accounts","base.at"
"account_tag_external_code_3495","3495","accounts","base.at"
"account_tag_external_code_3500","3500","accounts","base.at"
"account_tag_external_code_3501","3501","accounts","base.at"
"account_tag_external_code_3502","3502","accounts","base.at"
"account_tag_external_code_3503","3503","accounts","base.at"
"account_tag_external_code_3504","3504","accounts","base.at"
"account_tag_external_code_3505","3505","accounts","base.at"
"account_tag_external_code_3506","3506","accounts","base.at"
"account_tag_external_code_3507","3507","accounts","base.at"
"account_tag_external_code_3509","3509","accounts","base.at"
"account_tag_external_code_3510","3510","accounts","base.at"
"account_tag_external_code_3511","3511","accounts","base.at"
"account_tag_external_code_3516","3516","accounts","base.at"
"account_tag_external_code_3517","3517","accounts","base.at"
"account_tag_external_code_3520","3520","accounts","base.at"
"account_tag_external_code_3530","3530","accounts","base.at"
"account_tag_external_code_3540","3540","accounts","base.at"
"account_tag_external_code_3541","3541","accounts","base.at"
"account_tag_external_code_3542","3542","accounts","base.at"
"account_tag_external_code_3550","3550","accounts","base.at"
"account_tag_external_code_3551","3551","accounts","base.at"
"account_tag_external_code_3554","3554","accounts","base.at"
"account_tag_external_code_3556","3556","accounts","base.at"
"account_tag_external_code_3560","3560","accounts","base.at"
"account_tag_external_code_3590","3590","accounts","base.at"
"account_tag_external_code_3600","3600","accounts","base.at"
"account_tag_external_code_3610","3610","accounts","base.at"
"account_tag_external_code_3620","3620","accounts","base.at"
"account_tag_external_code_3630","3630","accounts","base.at"
"account_tag_external_code_3640","3640","accounts","base.at"
"account_tag_external_code_3650","3650","accounts","base.at"
"account_tag_external_code_3700","3700","accounts","base.at"
"account_tag_external_code_3750","3750","accounts","base.at"
"account_tag_external_code_3760","3760","accounts","base.at"
"account_tag_external_code_3800","3800","accounts","base.at"
"account_tag_external_code_3900","3900","accounts","base.at"
"account_tag_external_code_3990","3990","accounts","base.at"
"account_tag_external_code_3999","3999","accounts","base.at"
"account_tag_external_code_4000","4000","accounts","base.at"
"account_tag_external_code_4010","4010","accounts","base.at"
"account_tag_external_code_4012","4012","accounts","base.at"
"account_tag_external_code_4014","4014","accounts","base.at"
"account_tag_external_code_4016","4016","accounts","base.at"
"account_tag_external_code_4019","4019","accounts","base.at"
"account_tag_external_code_4030","4030","accounts","base.at"
"account_tag_external_code_4031","4031","accounts","base.at"
"account_tag_external_code_4050","4050","accounts","base.at"
"account_tag_external_code_4052","4052","accounts","base.at"
"account_tag_external_code_4054","4054","accounts","base.at"
"account_tag_external_code_4060","4060","accounts","base.at"
"account_tag_external_code_4062","4062","accounts","base.at"
"account_tag_external_code_4064","4064","accounts","base.at"
"account_tag_external_code_4066","4066","accounts","base.at"
"account_tag_external_code_4068","4068","accounts","base.at"
"account_tag_external_code_4069","4069","accounts","base.at"
"account_tag_external_code_4070","4070","accounts","base.at"
"account_tag_external_code_4080","4080","accounts","base.at"
"account_tag_external_code_4084","4084","accounts","base.at"
"account_tag_external_code_4090","4090","accounts","base.at"
"account_tag_external_code_4092","4092","accounts","base.at"
"account_tag_external_code_4095","4095","accounts","base.at"
"account_tag_external_code_4096","4096","accounts","base.at"
"account_tag_external_code_4098","4098","accounts","base.at"
"account_tag_external_code_4100","4100","accounts","base.at"
"account_tag_external_code_4110","4110","accounts","base.at"
"account_tag_external_code_4111","4111","accounts","base.at"
"account_tag_external_code_4120","4120","accounts","base.at"
"account_tag_external_code_4200","4200","accounts","base.at"
"account_tag_external_code_4210","4210","accounts","base.at"
"account_tag_external_code_4290","4290","accounts","base.at"
"account_tag_external_code_4310","4310","accounts","base.at"
"account_tag_external_code_4350","4350","accounts","base.at"
"account_tag_external_code_4352","4352","accounts","base.at"
"account_tag_external_code_4353","4353","accounts","base.at"
"account_tag_external_code_4354","4354","accounts","base.at"
"account_tag_external_code_4356","4356","accounts","base.at"
"account_tag_external_code_4358","4358","accounts","base.at"
"account_tag_external_code_4360","4360","accounts","base.at"
"account_tag_external_code_4362","4362","accounts","base.at"
"account_tag_external_code_4364","4364","accounts","base.at"
"account_tag_external_code_4400","4400","accounts","base.at"
"account_tag_external_code_4401","4401","accounts","base.at"
"account_tag_external_code_4402","4402","accounts","base.at"
"account_tag_external_code_4403","4403","accounts","base.at"
"account_tag_external_code_4416","4416","accounts","base.at"
"account_tag_external_code_4419","4419","accounts","base.at"
"account_tag_external_code_4420","4420","accounts","base.at"
"account_tag_external_code_4421","4421","accounts","base.at"
"account_tag_external_code_4422","4422","accounts","base.at"
"account_tag_external_code_4423","4423","accounts","base.at"
"account_tag_external_code_4424","4424","accounts","base.at"
"account_tag_external_code_4425","4425","accounts","base.at"
"account_tag_external_code_4430","4430","accounts","base.at"
"account_tag_external_code_4449","4449","accounts","base.at"
"account_tag_external_code_4450","4450","accounts","base.at"
"account_tag_external_code_4451","4451","accounts","base.at"
"account_tag_external_code_4452","4452","accounts","base.at"
"account_tag_external_code_4453","4453","accounts","base.at"
"account_tag_external_code_4454","4454","accounts","base.at"
"account_tag_external_code_4455","4455","accounts","base.at"
"account_tag_external_code_4456","4456","accounts","base.at"
"account_tag_external_code_4457","4457","accounts","base.at"
"account_tag_external_code_4458","4458","accounts","base.at"
"account_tag_external_code_4459","4459","accounts","base.at"
"account_tag_external_code_4460","4460","accounts","base.at"
"account_tag_external_code_4461","4461","accounts","base.at"
"account_tag_external_code_4462","4462","accounts","base.at"
"account_tag_external_code_4466","4466","accounts","base.at"
"account_tag_external_code_4469","4469","accounts","base.at"
"account_tag_external_code_4470","4470","accounts","base.at"
"account_tag_external_code_4480","4480","accounts","base.at"
"account_tag_external_code_4481","4481","accounts","base.at"
"account_tag_external_code_4482","4482","accounts","base.at"
"account_tag_external_code_4486","4486","accounts","base.at"
"account_tag_external_code_4489","4489","accounts","base.at"
"account_tag_external_code_4490","4490","accounts","base.at"
"account_tag_external_code_4500","4500","accounts","base.at"
"account_tag_external_code_4510","4510","accounts","base.at"
"account_tag_external_code_4520","4520","accounts","base.at"
"account_tag_external_code_4580","4580","accounts","base.at"
"account_tag_external_code_4600","4600","accounts","base.at"
"account_tag_external_code_4605","4605","accounts","base.at"
"account_tag_external_code_4610","4610","accounts","base.at"
"account_tag_external_code_4630","4630","accounts","base.at"
"account_tag_external_code_4631","4631","accounts","base.at"
"account_tag_external_code_4660","4660","accounts","base.at"
"account_tag_external_code_4700","4700","accounts","base.at"
"account_tag_external_code_4701","4701","accounts","base.at"
"account_tag_external_code_4702","4702","accounts","base.at"
"account_tag_external_code_4705","4705","accounts","base.at"
"account_tag_external_code_4709","4709","accounts","base.at"
"account_tag_external_code_4800","4800","accounts","base.at"
"account_tag_external_code_4810","4810","accounts","base.at"
"account_tag_external_code_4820","4820","accounts","base.at"
"account_tag_external_code_4826","4826","accounts","base.at"
"account_tag_external_code_4829","4829","accounts","base.at"
"account_tag_external_code_4830","4830","accounts","base.at"
"account_tag_external_code_4831","4831","accounts","base.at"
"account_tag_external_code_4835","4835","accounts","base.at"
"account_tag_external_code_4840","4840","accounts","base.at"
"account_tag_external_code_4850","4850","accounts","base.at"
"account_tag_external_code_4851","4851","accounts","base.at"
"account_tag_external_code_4852","4852","accounts","base.at"
"account_tag_external_code_4855","4855","accounts","base.at"
"account_tag_external_code_4860","4860","accounts","base.at"
"account_tag_external_code_4865","4865","accounts","base.at"
"account_tag_external_code_4866","4866","accounts","base.at"
"account_tag_external_code_4867","4867","accounts","base.at"
"account_tag_external_code_4870","4870","accounts","base.at"
"account_tag_external_code_4871","4871","accounts","base.at"
"account_tag_external_code_4872","4872","accounts","base.at"
"account_tag_external_code_4875","4875","accounts","base.at"
"account_tag_external_code_4880","4880","accounts","base.at"
"account_tag_external_code_4881","4881","accounts","base.at"
"account_tag_external_code_4882","4882","accounts","base.at"
"account_tag_external_code_4885","4885","accounts","base.at"
"account_tag_external_code_4900","4900","accounts","base.at"
"account_tag_external_code_4901","4901","accounts","base.at"
"account_tag_external_code_4902","4902","accounts","base.at"
"account_tag_external_code_4903","4903","accounts","base.at"
"account_tag_external_code_4940","4940","accounts","base.at"
"account_tag_external_code_4941","4941","accounts","base.at"
"account_tag_external_code_4942","4942","accounts","base.at"
"account_tag_external_code_4943","4943","accounts","base.at"
"account_tag_external_code_4980","4980","accounts","base.at"
"account_tag_external_code_4981","4981","accounts","base.at"
"account_tag_external_code_4982","4982","accounts","base.at"
"account_tag_external_code_4983","4983","accounts","base.at"
"account_tag_external_code_4995","4995","accounts","base.at"
"account_tag_external_code_4999","4999","accounts","base.at"
"account_tag_external_code_5000","5000","accounts","base.at"
"account_tag_external_code_5001","5001","accounts","base.at"
"account_tag_external_code_5002","5002","accounts","base.at"
"account_tag_external_code_5003","5003","accounts","base.at"
"account_tag_external_code_5004","5004","accounts","base.at"
"account_tag_external_code_5005","5005","accounts","base.at"
"account_tag_external_code_5006","5006","accounts","base.at"
"account_tag_external_code_5007","5007","accounts","base.at"
"account_tag_external_code_5080","5080","accounts","base.at"
"account_tag_external_code_5084","5084","accounts","base.at"
"account_tag_external_code_5100","5100","accounts","base.at"
"account_tag_external_code_5200","5200","accounts","base.at"
"account_tag_external_code_5300","5300","accounts","base.at"
"account_tag_external_code_5310","5310","accounts","base.at"
"account_tag_external_code_5320","5320","accounts","base.at"
"account_tag_external_code_5330","5330","accounts","base.at"
"account_tag_external_code_5340","5340","accounts","base.at"
"account_tag_external_code_5350","5350","accounts","base.at"
"account_tag_external_code_5360","5360","accounts","base.at"
"account_tag_external_code_5370","5370","accounts","base.at"
"account_tag_external_code_5400","5400","accounts","base.at"
"account_tag_external_code_5410","5410","accounts","base.at"
"account_tag_external_code_5411","5411","accounts","base.at"
"account_tag_external_code_5412","5412","accounts","base.at"
"account_tag_external_code_5414","5414","accounts","base.at"
"account_tag_external_code_5415","5415","accounts","base.at"
"account_tag_external_code_5416","5416","accounts","base.at"
"account_tag_external_code_5417","5417","accounts","base.at"
"account_tag_external_code_5440","5440","accounts","base.at"
"account_tag_external_code_5441","5441","accounts","base.at"
"account_tag_external_code_5450","5450","accounts","base.at"
"account_tag_external_code_5451","5451","accounts","base.at"
"account_tag_external_code_5460","5460","accounts","base.at"
"account_tag_external_code_5461","5461","accounts","base.at"
"account_tag_external_code_5470","5470","accounts","base.at"
"account_tag_external_code_5471","5471","accounts","base.at"
"account_tag_external_code_5500","5500","accounts","base.at"
"account_tag_external_code_5501","5501","accounts","base.at"
"account_tag_external_code_5600","5600","accounts","base.at"
"account_tag_external_code_5601","5601","accounts","base.at"
"account_tag_external_code_5610","5610","accounts","base.at"
"account_tag_external_code_5615","5615","accounts","base.at"
"account_tag_external_code_5620","5620","accounts","base.at"
"account_tag_external_code_5621","5621","accounts","base.at"
"account_tag_external_code_5700","5700","accounts","base.at"
"account_tag_external_code_5750","5750","accounts","base.at"
"account_tag_external_code_5770","5770","accounts","base.at"
"account_tag_external_code_5772","5772","accounts","base.at"
"account_tag_external_code_5774","5774","accounts","base.at"
"account_tag_external_code_5780","5780","accounts","base.at"
"account_tag_external_code_5800","5800","accounts","base.at"
"account_tag_external_code_5801","5801","accounts","base.at"
"account_tag_external_code_5802","5802","accounts","base.at"
"account_tag_external_code_5803","5803","accounts","base.at"
"account_tag_external_code_5805","5805","accounts","base.at"
"account_tag_external_code_5806","5806","accounts","base.at"
"account_tag_external_code_5810","5810","accounts","base.at"
"account_tag_external_code_5811","5811","accounts","base.at"
"account_tag_external_code_5812","5812","accounts","base.at"
"account_tag_external_code_5820","5820","accounts","base.at"
"account_tag_external_code_5821","5821","accounts","base.at"
"account_tag_external_code_5822","5822","accounts","base.at"
"account_tag_external_code_5830","5830","accounts","base.at"
"account_tag_external_code_5831","5831","accounts","base.at"
"account_tag_external_code_5834","5834","accounts","base.at"
"account_tag_external_code_5835","5835","accounts","base.at"
"account_tag_external_code_5840","5840","accounts","base.at"
"account_tag_external_code_5842","5842","accounts","base.at"
"account_tag_external_code_5844","5844","accounts","base.at"
"account_tag_external_code_5850","5850","accounts","base.at"
"account_tag_external_code_5851","5851","accounts","base.at"
"account_tag_external_code_5852","5852","accounts","base.at"
"account_tag_external_code_5853","5853","accounts","base.at"
"account_tag_external_code_5855","5855","accounts","base.at"
"account_tag_external_code_5856","5856","accounts","base.at"
"account_tag_external_code_5857","5857","accounts","base.at"
"account_tag_external_code_5858","5858","accounts","base.at"
"account_tag_external_code_5860","5860","accounts","base.at"
"account_tag_external_code_5900","5900","accounts","base.at"
"account_tag_external_code_6000","6000","accounts","base.at"
"account_tag_external_code_6005","6005","accounts","base.at"
"account_tag_external_code_6010","6010","accounts","base.at"
"account_tag_external_code_6015","6015","accounts","base.at"
"account_tag_external_code_6020","6020","accounts","base.at"
"account_tag_external_code_6025","6025","accounts","base.at"
"account_tag_external_code_6030","6030","accounts","base.at"
"account_tag_external_code_6040","6040","accounts","base.at"
"account_tag_external_code_6050","6050","accounts","base.at"
"account_tag_external_code_6060","6060","accounts","base.at"
"account_tag_external_code_6070","6070","accounts","base.at"
"account_tag_external_code_6090","6090","accounts","base.at"
"account_tag_external_code_6100","6100","accounts","base.at"
"account_tag_external_code_6200","6200","accounts","base.at"
"account_tag_external_code_6205","6205","accounts","base.at"
"account_tag_external_code_6210","6210","accounts","base.at"
"account_tag_external_code_6220","6220","accounts","base.at"
"account_tag_external_code_6225","6225","accounts","base.at"
"account_tag_external_code_6230","6230","accounts","base.at"
"account_tag_external_code_6240","6240","accounts","base.at"
"account_tag_external_code_6250","6250","accounts","base.at"
"account_tag_external_code_6260","6260","accounts","base.at"
"account_tag_external_code_6270","6270","accounts","base.at"
"account_tag_external_code_6290","6290","accounts","base.at"
"account_tag_external_code_6400","6400","accounts","base.at"
"account_tag_external_code_6401","6401","accounts","base.at"
"account_tag_external_code_6402","6402","accounts","base.at"
"account_tag_external_code_6403","6403","accounts","base.at"
"account_tag_external_code_6404","6404","accounts","base.at"
"account_tag_external_code_6410","6410","accounts","base.at"
"account_tag_external_code_6411","6411","accounts","base.at"
"account_tag_external_code_6420","6420","accounts","base.at"
"account_tag_external_code_6421","6421","accounts","base.at"
"account_tag_external_code_6450","6450","accounts","base.at"
"account_tag_external_code_6451","6451","accounts","base.at"
"account_tag_external_code_6452","6452","accounts","base.at"
"account_tag_external_code_6453","6453","accounts","base.at"
"account_tag_external_code_6454","6454","accounts","base.at"
"account_tag_external_code_6455","6455","accounts","base.at"
"account_tag_external_code_6460","6460","accounts","base.at"
"account_tag_external_code_6461","6461","accounts","base.at"
"account_tag_external_code_6500","6500","accounts","base.at"
"account_tag_external_code_6560","6560","accounts","base.at"
"account_tag_external_code_6580","6580","accounts","base.at"
"account_tag_external_code_6590","6590","accounts","base.at"
"account_tag_external_code_6600","6600","accounts","base.at"
"account_tag_external_code_6601","6601","accounts","base.at"
"account_tag_external_code_6602","6602","accounts","base.at"
"account_tag_external_code_6603","6603","accounts","base.at"
"account_tag_external_code_6660","6660","accounts","base.at"
"account_tag_external_code_6661","6661","accounts","base.at"
"account_tag_external_code_6662","6662","accounts","base.at"
"account_tag_external_code_6663","6663","accounts","base.at"
"account_tag_external_code_6670","6670","accounts","base.at"
"account_tag_external_code_6680","6680","accounts","base.at"
"account_tag_external_code_6685","6685","accounts","base.at"
"account_tag_external_code_6690","6690","accounts","base.at"
"account_tag_external_code_6695","6695","accounts","base.at"
"account_tag_external_code_6700","6700","accounts","base.at"
"account_tag_external_code_6710","6710","accounts","base.at"
"account_tag_external_code_6720","6720","accounts","base.at"
"account_tag_external_code_6730","6730","accounts","base.at"
"account_tag_external_code_6731","6731","accounts","base.at"
"account_tag_external_code_6740","6740","accounts","base.at"
"account_tag_external_code_6750","6750","accounts","base.at"
"account_tag_external_code_6751","6751","accounts","base.at"
"account_tag_external_code_6790","6790","accounts","base.at"
"account_tag_external_code_6791","6791","accounts","base.at"
"account_tag_external_code_6800","6800","accounts","base.at"
"account_tag_external_code_7000","7000","accounts","base.at"
"account_tag_external_code_7005","7005","accounts","base.at"
"account_tag_external_code_7010","7010","accounts","base.at"
"account_tag_external_code_7011","7011","accounts","base.at"
"account_tag_external_code_7020","7020","accounts","base.at"
"account_tag_external_code_7021","7021","accounts","base.at"
"account_tag_external_code_7022","7022","accounts","base.at"
"account_tag_external_code_7030","7030","accounts","base.at"
"account_tag_external_code_7040","7040","accounts","base.at"
"account_tag_external_code_7041","7041","accounts","base.at"
"account_tag_external_code_7042","7042","accounts","base.at"
"account_tag_external_code_7050","7050","accounts","base.at"
"account_tag_external_code_7090","7090","accounts","base.at"
"account_tag_external_code_7100","7100","accounts","base.at"
"account_tag_external_code_7110","7110","accounts","base.at"
"account_tag_external_code_7120","7120","accounts","base.at"
"account_tag_external_code_7130","7130","accounts","base.at"
"account_tag_external_code_7140","7140","accounts","base.at"
"account_tag_external_code_7150","7150","accounts","base.at"
"account_tag_external_code_7160","7160","accounts","base.at"
"account_tag_external_code_7161","7161","accounts","base.at"
"account_tag_external_code_7162","7162","accounts","base.at"
"account_tag_external_code_7170","7170","accounts","base.at"
"account_tag_external_code_7171","7171","accounts","base.at"
"account_tag_external_code_7180","7180","accounts","base.at"
"account_tag_external_code_7200","7200","accounts","base.at"
"account_tag_external_code_7201","7201","accounts","base.at"
"account_tag_external_code_7202","7202","accounts","base.at"
"account_tag_external_code_7203","7203","accounts","base.at"
"account_tag_external_code_7204","7204","accounts","base.at"
"account_tag_external_code_7205","7205","accounts","base.at"
"account_tag_external_code_7206","7206","accounts","base.at"
"account_tag_external_code_7209","7209","accounts","base.at"
"account_tag_external_code_7210","7210","accounts","base.at"
"account_tag_external_code_7215","7215","accounts","base.at"
"account_tag_external_code_7216","7216","accounts","base.at"
"account_tag_external_code_7220","7220","accounts","base.at"
"account_tag_external_code_7225","7225","accounts","base.at"
"account_tag_external_code_7230","7230","accounts","base.at"
"account_tag_external_code_7235","7235","accounts","base.at"
"account_tag_external_code_7300","7300","accounts","base.at"
"account_tag_external_code_7320","7320","accounts","base.at"
"account_tag_external_code_7321","7321","accounts","base.at"
"account_tag_external_code_7322","7322","accounts","base.at"
"account_tag_external_code_7323","7323","accounts","base.at"
"account_tag_external_code_7324","7324","accounts","base.at"
"account_tag_external_code_7325","7325","accounts","base.at"
"account_tag_external_code_7326","7326","accounts","base.at"
"account_tag_external_code_7330","7330","accounts","base.at"
"account_tag_external_code_7332","7332","accounts","base.at"
"account_tag_external_code_7334","7334","accounts","base.at"
"account_tag_external_code_7335","7335","accounts","base.at"
"account_tag_external_code_7336","7336","accounts","base.at"
"account_tag_external_code_7340","7340","accounts","base.at"
"account_tag_external_code_7345","7345","accounts","base.at"
"account_tag_external_code_7350","7350","accounts","base.at"
"account_tag_external_code_7355","7355","accounts","base.at"
"account_tag_external_code_7360","7360","accounts","base.at"
"account_tag_external_code_7370","7370","accounts","base.at"
"account_tag_external_code_7380","7380","accounts","base.at"
"account_tag_external_code_7381","7381","accounts","base.at"
"account_tag_external_code_7382","7382","accounts","base.at"
"account_tag_external_code_7385","7385","accounts","base.at"
"account_tag_external_code_7390","7390","accounts","base.at"
"account_tag_external_code_7400","7400","accounts","base.at"
"account_tag_external_code_7401","7401","accounts","base.at"
"account_tag_external_code_7402","7402","accounts","base.at"
"account_tag_external_code_7410","7410","accounts","base.at"
"account_tag_external_code_7411","7411","accounts","base.at"
"account_tag_external_code_7412","7412","accounts","base.at"
"account_tag_external_code_7440","7440","accounts","base.at"
"account_tag_external_code_7480","7480","accounts","base.at"
"account_tag_external_code_7490","7490","accounts","base.at"
"account_tag_external_code_7500","7500","accounts","base.at"
"account_tag_external_code_7540","7540","accounts","base.at"
"account_tag_external_code_7580","7580","accounts","base.at"
"account_tag_external_code_7585","7585","accounts","base.at"
"account_tag_external_code_7600","7600","accounts","base.at"
"account_tag_external_code_7601","7601","accounts","base.at"
"account_tag_external_code_7610","7610","accounts","base.at"
"account_tag_external_code_7611","7611","accounts","base.at"
"account_tag_external_code_7630","7630","accounts","base.at"
"account_tag_external_code_7631","7631","accounts","base.at"
"account_tag_external_code_7650","7650","accounts","base.at"
"account_tag_external_code_7651","7651","accounts","base.at"
"account_tag_external_code_7652","7652","accounts","base.at"
"account_tag_external_code_7653","7653","accounts","base.at"
"account_tag_external_code_7654","7654","accounts","base.at"
"account_tag_external_code_7660","7660","accounts","base.at"
"account_tag_external_code_7661","7661","accounts","base.at"
"account_tag_external_code_7685","7685","accounts","base.at"
"account_tag_external_code_7690","7690","accounts","base.at"
"account_tag_external_code_7695","7695","accounts","base.at"
"account_tag_external_code_7696","7696","accounts","base.at"
"account_tag_external_code_7700","7700","accounts","base.at"
"account_tag_external_code_7710","7710","accounts","base.at"
"account_tag_external_code_7720","7720","accounts","base.at"
"account_tag_external_code_7740","7740","accounts","base.at"
"account_tag_external_code_7750","7750","accounts","base.at"
"account_tag_external_code_7755","7755","accounts","base.at"
"account_tag_external_code_7758","7758","accounts","base.at"
"account_tag_external_code_7760","7760","accounts","base.at"
"account_tag_external_code_7765","7765","accounts","base.at"
"account_tag_external_code_7770","7770","accounts","base.at"
"account_tag_external_code_7775","7775","accounts","base.at"
"account_tag_external_code_7780","7780","accounts","base.at"
"account_tag_external_code_7782","7782","accounts","base.at"
"account_tag_external_code_7785","7785","accounts","base.at"
"account_tag_external_code_7790","7790","accounts","base.at"
"account_tag_external_code_7800","7800","accounts","base.at"
"account_tag_external_code_7801","7801","accounts","base.at"
"account_tag_external_code_7804","7804","accounts","base.at"
"account_tag_external_code_7805","7805","accounts","base.at"
"account_tag_external_code_7806","7806","accounts","base.at"
"account_tag_external_code_7807","7807","accounts","base.at"
"account_tag_external_code_7808","7808","accounts","base.at"
"account_tag_external_code_7809","7809","accounts","base.at"
"account_tag_external_code_7810","7810","accounts","base.at"
"account_tag_external_code_7811","7811","accounts","base.at"
"account_tag_external_code_7815","7815","accounts","base.at"
"account_tag_external_code_7816","7816","accounts","base.at"
"account_tag_external_code_7820","7820","accounts","base.at"
"account_tag_external_code_7825","7825","accounts","base.at"
"account_tag_external_code_7830","7830","accounts","base.at"
"account_tag_external_code_7840","7840","accounts","base.at"
"account_tag_external_code_7841","7841","accounts","base.at"
"account_tag_external_code_7850","7850","accounts","base.at"
"account_tag_external_code_7860","7860","accounts","base.at"
"account_tag_external_code_7870","7870","accounts","base.at"
"account_tag_external_code_7890","7890","accounts","base.at"
"account_tag_external_code_7900","7900","accounts","base.at"
"account_tag_external_code_7910","7910","accounts","base.at"
"account_tag_external_code_7960","7960","accounts","base.at"
"account_tag_external_code_7970","7970","accounts","base.at"
"account_tag_external_code_7980","7980","accounts","base.at"
"account_tag_external_code_7990","7990","accounts","base.at"
"account_tag_external_code_7999","7999","accounts","base.at"
"account_tag_external_code_8000","8000","accounts","base.at"
"account_tag_external_code_8010","8010","accounts","base.at"
"account_tag_external_code_8020","8020","accounts","base.at"
"account_tag_external_code_8030","8030","accounts","base.at"
"account_tag_external_code_8040","8040","accounts","base.at"
"account_tag_external_code_8045","8045","accounts","base.at"
"account_tag_external_code_8050","8050","accounts","base.at"
"account_tag_external_code_8052","8052","accounts","base.at"
"account_tag_external_code_8055","8055","accounts","base.at"
"account_tag_external_code_8060","8060","accounts","base.at"
"account_tag_external_code_8070","8070","accounts","base.at"
"account_tag_external_code_8080","8080","accounts","base.at"
"account_tag_external_code_8100","8100","accounts","base.at"
"account_tag_external_code_8101","8101","accounts","base.at"
"account_tag_external_code_8110","8110","accounts","base.at"
"account_tag_external_code_8120","8120","accounts","base.at"
"account_tag_external_code_8121","8121","accounts","base.at"
"account_tag_external_code_8122","8122","accounts","base.at"
"account_tag_external_code_8125","8125","accounts","base.at"
"account_tag_external_code_8140","8140","accounts","base.at"
"account_tag_external_code_8150","8150","accounts","base.at"
"account_tag_external_code_8160","8160","accounts","base.at"
"account_tag_external_code_8170","8170","accounts","base.at"
"account_tag_external_code_8171","8171","accounts","base.at"
"account_tag_external_code_8180","8180","accounts","base.at"
"account_tag_external_code_8181","8181","accounts","base.at"
"account_tag_external_code_8190","8190","accounts","base.at"
"account_tag_external_code_8191","8191","accounts","base.at"
"account_tag_external_code_8200","8200","accounts","base.at"
"account_tag_external_code_8201","8201","accounts","base.at"
"account_tag_external_code_8205","8205","accounts","base.at"
"account_tag_external_code_8206","8206","accounts","base.at"
"account_tag_external_code_8210","8210","accounts","base.at"
"account_tag_external_code_8211","8211","accounts","base.at"
"account_tag_external_code_8220","8220","accounts","base.at"
"account_tag_external_code_8230","8230","accounts","base.at"
"account_tag_external_code_8231","8231","accounts","base.at"
"account_tag_external_code_8232","8232","accounts","base.at"
"account_tag_external_code_8260","8260","accounts","base.at"
"account_tag_external_code_8261","8261","accounts","base.at"
"account_tag_external_code_8270","8270","accounts","base.at"
"account_tag_external_code_8271","8271","accounts","base.at"
"account_tag_external_code_8280","8280","accounts","base.at"
"account_tag_external_code_8281","8281","accounts","base.at"
"account_tag_external_code_8290","8290","accounts","base.at"
"account_tag_external_code_8300","8300","accounts","base.at"
"account_tag_external_code_8310","8310","accounts","base.at"
"account_tag_external_code_8320","8320","accounts","base.at"
"account_tag_external_code_8340","8340","accounts","base.at"
"account_tag_external_code_8350","8350","accounts","base.at"
"account_tag_external_code_8360","8360","accounts","base.at"
"account_tag_external_code_8400","8400","accounts","base.at"
"account_tag_external_code_8410","8410","accounts","base.at"
"account_tag_external_code_8420","8420","accounts","base.at"
"account_tag_external_code_8440","8440","accounts","base.at"
"account_tag_external_code_8445","8445","accounts","base.at"
"account_tag_external_code_8450","8450","accounts","base.at"
"account_tag_external_code_8460","8460","accounts","base.at"
"account_tag_external_code_8470","8470","accounts","base.at"
"account_tag_external_code_8480","8480","accounts","base.at"
"account_tag_external_code_8490","8490","accounts","base.at"
"account_tag_external_code_8500","8500","accounts","base.at"
"account_tag_external_code_8510","8510","accounts","base.at"
"account_tag_external_code_8520","8520","accounts","base.at"
"account_tag_external_code_8530","8530","accounts","base.at"
"account_tag_external_code_8540","8540","accounts","base.at"
"account_tag_external_code_8550","8550","accounts","base.at"
"account_tag_external_code_8560","8560","accounts","base.at"
"account_tag_external_code_8600","8600","accounts","base.at"
"account_tag_external_code_8610","8610","accounts","base.at"
"account_tag_external_code_8700","8700","accounts","base.at"
"account_tag_external_code_8710","8710","accounts","base.at"
"account_tag_external_code_8720","8720","accounts","base.at"
"account_tag_external_code_8750","8750","accounts","base.at"
"account_tag_external_code_8760","8760","accounts","base.at"
"account_tag_external_code_8770","8770","accounts","base.at"
"account_tag_external_code_8800","8800","accounts","base.at"
"account_tag_external_code_8810","8810","accounts","base.at"
"account_tag_external_code_8900","8900","accounts","base.at"
"account_tag_external_code_8910","8910","accounts","base.at"
"account_tag_external_code_8920","8920","accounts","base.at"
"account_tag_external_code_8990","8990","accounts","base.at"
"account_tag_external_code_8991","8991","accounts","base.at"
"account_tag_external_code_9000","9000","accounts","base.at"
"account_tag_external_code_9001","9001","accounts","base.at"
"account_tag_external_code_9009","9009","accounts","base.at"
"account_tag_external_code_9010","9010","accounts","base.at"
"account_tag_external_code_9011","9011","accounts","base.at"
"account_tag_external_code_9030","9030","accounts","base.at"
"account_tag_external_code_9031","9031","accounts","base.at"
"account_tag_external_code_9040","9040","accounts","base.at"
"account_tag_external_code_9050","9050","accounts","base.at"
"account_tag_external_code_9060","9060","accounts","base.at"
"account_tag_external_code_9070","9070","accounts","base.at"
"account_tag_external_code_9080","9080","accounts","base.at"
"account_tag_external_code_9090","9090","accounts","base.at"
"account_tag_external_code_9130","9130","accounts","base.at"
"account_tag_external_code_9140","9140","accounts","base.at"
"account_tag_external_code_9150","9150","accounts","base.at"
"account_tag_external_code_9160","9160","accounts","base.at"
"account_tag_external_code_9161","9161","accounts","base.at"
"account_tag_external_code_9162","9162","accounts","base.at"
"account_tag_external_code_9163","9163","accounts","base.at"
"account_tag_external_code_9190","9190","accounts","base.at"
"account_tag_external_code_9191","9191","accounts","base.at"
"account_tag_external_code_9192","9192","accounts","base.at"
"account_tag_external_code_9200","9200","accounts","base.at"
"account_tag_external_code_9210","9210","accounts","base.at"
"account_tag_external_code_9220","9220","accounts","base.at"
"account_tag_external_code_9240","9240","accounts","base.at"
"account_tag_external_code_9300","9300","accounts","base.at"
"account_tag_external_code_9310","9310","accounts","base.at"
"account_tag_external_code_9320","9320","accounts","base.at"
"account_tag_external_code_9330","9330","accounts","base.at"
"account_tag_external_code_9340","9340","accounts","base.at"
"account_tag_external_code_9345","9345","accounts","base.at"
"account_tag_external_code_9350","9350","accounts","base.at"
"account_tag_external_code_9351","9351","accounts","base.at"
"account_tag_external_code_9360","9360","accounts","base.at"
"account_tag_external_code_9361","9361","accounts","base.at"
"account_tag_external_code_9370","9370","accounts","base.at"
"account_tag_external_code_9371","9371","accounts","base.at"
"account_tag_external_code_9380","9380","accounts","base.at"
"account_tag_external_code_9381","9381","accounts","base.at"
"account_tag_external_code_9385","9385","accounts","base.at"
"account_tag_external_code_9389","9389","accounts","base.at"
"account_tag_external_code_9390","9390","accounts","base.at"
"account_tag_external_code_9392","9392","accounts","base.at"
"account_tag_external_code_9393","9393","accounts","base.at"
"account_tag_external_code_9396","9396","accounts","base.at"
"account_tag_external_code_9397","9397","accounts","base.at"
"account_tag_external_code_9400","9400","accounts","base.at"
"account_tag_external_code_9410","9410","accounts","base.at"
"account_tag_external_code_9420","9420","accounts","base.at"
"account_tag_external_code_9430","9430","accounts","base.at"
"account_tag_external_code_9450","9450","accounts","base.at"
"account_tag_external_code_9501","9501","accounts","base.at"
"account_tag_external_code_9502","9502","accounts","base.at"
"account_tag_external_code_9550","9550","accounts","base.at"
"account_tag_external_code_9570","9570","accounts","base.at"
"account_tag_external_code_9580","9580","accounts","base.at"
"account_tag_external_code_9581","9581","accounts","base.at"
"account_tag_external_code_9600","9600","accounts","base.at"
"account_tag_external_code_9601","9601","accounts","base.at"
"account_tag_external_code_9610","9610","accounts","base.at"
"account_tag_external_code_9611","9611","accounts","base.at"
"account_tag_external_code_9612","9612","accounts","base.at"
"account_tag_external_code_9613","9613","accounts","base.at"
"account_tag_external_code_9614","9614","accounts","base.at"
"account_tag_external_code_9618","9618","accounts","base.at"
"account_tag_external_code_9620","9620","accounts","base.at"
"account_tag_external_code_9630","9630","accounts","base.at"
"account_tag_external_code_9640","9640","accounts","base.at"
"account_tag_external_code_9641","9641","accounts","base.at"
"account_tag_external_code_9644","9644","accounts","base.at"
"account_tag_external_code_9700","9700","accounts","base.at"
"account_tag_external_code_9710","9710","accounts","base.at"
"account_tag_external_code_9720","9720","accounts","base.at"
"account_tag_external_code_9730","9730","accounts","base.at"
"account_tag_external_code_9800","9800","accounts","base.at"
"account_tag_external_code_9810","9810","accounts","base.at"
"account_tag_external_code_9850","9850","accounts","base.at"
"account_tag_external_code_9880","9880","accounts","base.at"
"account_tag_external_code_9881","9881","accounts","base.at"
"account_tag_external_code_9882","9882","accounts","base.at"
"account_tag_external_code_9883","9883","accounts","base.at"
"account_tag_external_code_9884","9884","accounts","base.at"
"account_tag_external_code_9890","9890","accounts","base.at"
"account_tag_external_code_9891","9891","accounts","base.at"
"account_tag_external_code_9990","9990","accounts","base.at"
"account_tag_external_code_9991","9991","accounts","base.at"
"account_tag_external_code_9992","9992","accounts","base.at"
"account_tag_external_code_9993","9993","accounts","base.at"
"account_tag_external_code_9994","9994","accounts","base.at"

```

## File: data\account_account_tag.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record
            id="account_tag_l10n_at_AAI1"
            model="account.account.tag">
            <field name="name">Bilanz AAI1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AAI2"
            model="account.account.tag">
            <field name="name">Bilanz AAI2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AAI3"
            model="account.account.tag">
            <field name="name">Bilanz AAI3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AAII1"
            model="account.account.tag">
            <field name="name">Bilanz AAII1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AAII2"
            model="account.account.tag">
            <field name="name">Bilanz AAII2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AAII3"
            model="account.account.tag">
            <field name="name">Bilanz AAII3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AAII4"
            model="account.account.tag">
            <field name="name">Bilanz AAII4</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AAIII"
            model="account.account.tag">
            <field name="name">Bilanz AAIII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AAIII1"
            model="account.account.tag">
            <field name="name">Bilanz AAIII1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AAIII2"
            model="account.account.tag">
            <field name="name">Bilanz AAIII2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AAIII3"
            model="account.account.tag">
            <field name="name">Bilanz AAIII3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AAIII4"
            model="account.account.tag">
            <field name="name">Bilanz AAIII4</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AAIII5"
            model="account.account.tag">
            <field name="name">Bilanz AAIII5</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AAIII6"
            model="account.account.tag">
            <field name="name">Bilanz AAIII6</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_ABI1"
            model="account.account.tag">
            <field name="name">Bilanz ABI1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_ABI2"
            model="account.account.tag">
            <field name="name">Bilanz ABI2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_ABI3"
            model="account.account.tag">
            <field name="name">Bilanz ABI3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_ABI4"
            model="account.account.tag">
            <field name="name">Bilanz ABI4</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_ABI5"
            model="account.account.tag">
            <field name="name">Bilanz ABI5</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_ABII1"
            model="account.account.tag">
            <field name="name">Bilanz ABII1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_ABII2"
            model="account.account.tag">
            <field name="name">Bilanz ABII2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_ABII3"
            model="account.account.tag">
            <field name="name">Bilanz ABII3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_ABII4"
            model="account.account.tag">
            <field name="name">Bilanz ABII4</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_ABIII"
            model="account.account.tag">
            <field name="name">Bilanz ABIII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_ABIII1"
            model="account.account.tag">
            <field name="name">Bilanz ABIII1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_ABIII2"
            model="account.account.tag">
            <field name="name">Bilanz ABIII2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_ABIV"
            model="account.account.tag">
            <field name="name">Bilanz ABIV</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AC"
            model="account.account.tag">
            <field name="name">Bilanz AC</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_AD"
            model="account.account.tag">
            <field name="name">Bilanz AD</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PAI"
            model="account.account.tag">
            <field name="name">Bilanz PAI</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PAII"
            model="account.account.tag">
            <field name="name">Bilanz PAII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PAII1"
            model="account.account.tag">
            <field name="name">Bilanz PAII1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PAII2"
            model="account.account.tag">
            <field name="name">Bilanz PAII2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PAIII"
            model="account.account.tag">
            <field name="name">Bilanz PAIII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PAIII1"
            model="account.account.tag">
            <field name="name">Bilanz PAIII1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PAIII2"
            model="account.account.tag">
            <field name="name">Bilanz PAIII2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PAIII3"
            model="account.account.tag">
            <field name="name">Bilanz PAIII3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PAIV"
            model="account.account.tag">
            <field name="name">Bilanz PAIV</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PBI"
            model="account.account.tag">
            <field name="name">Bilanz PBI</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PBII"
            model="account.account.tag">
            <field name="name">Bilanz PBII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PBIII"
            model="account.account.tag">
            <field name="name">Bilanz PBIII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PBIV"
            model="account.account.tag">
            <field name="name">Bilanz PBIV</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PCI"
            model="account.account.tag">
            <field name="name">Bilanz PCI</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PCII"
            model="account.account.tag">
            <field name="name">Bilanz PCII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PCIII"
            model="account.account.tag">
            <field name="name">Bilanz PCIII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PCIV"
            model="account.account.tag">
            <field name="name">Bilanz PCIV</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PCV"
            model="account.account.tag">
            <field name="name">Bilanz PCV</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PCVI"
            model="account.account.tag">
            <field name="name">Bilanz PCVI</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PCVII"
            model="account.account.tag">
            <field name="name">Bilanz PCVII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PCVIII"
            model="account.account.tag">
            <field name="name">Bilanz PCVIII (deprecated)</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PCVIII1"
            model="account.account.tag">
            <field name="name">Bilanz PCVIII1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PCVIII2"
            model="account.account.tag">
            <field name="name">Bilanz PCVIII2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_PCVIII3"
            model="account.account.tag">
            <field name="name">Bilanz PCVIII3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PCVIII4"
            model="account.account.tag">
            <field name="name">Bilanz PCVIII4</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PD"
            model="account.account.tag">
            <field name="name">Bilanz PD</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>

        <record
            id="account_tag_l10n_at_EBIT1"
            model="account.account.tag">
            <field name="name">GuV EBIT1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_EBIT2"
            model="account.account.tag">
            <field name="name">GuV EBIT2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_EBIT3"
            model="account.account.tag">
            <field name="name">GuV EBIT3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_EBIT4"
            model="account.account.tag">
            <field name="name">GuV EBIT4</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_EBIT4I"
            model="account.account.tag">
            <field name="name">GuV EBIT4I</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_EBIT4II"
            model="account.account.tag">
            <field name="name">GuV EBIT4II</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_EBIT4III"
            model="account.account.tag">
            <field name="name">GuV EBIT4III</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_EBIT5I"
            model="account.account.tag">
            <field name="name">GuV EBIT5I</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_EBIT5II"
            model="account.account.tag">
            <field name="name">GuV EBIT5II</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_EBIT6I"
            model="account.account.tag">
            <field name="name">GuV EBIT6I</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_EBIT6II"
            model="account.account.tag">
            <field name="name">GuV EBIT6II</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_EBIT7I"
            model="account.account.tag">
            <field name="name">GuV EBIT7I</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_EBIT7II"
            model="account.account.tag">
            <field name="name">GuV EBIT7II</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_EBIT8"
            model="account.account.tag">
            <field name="name">GuV EBIT8</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_FIN10"
            model="account.account.tag">
            <field name="name">GuV FIN10</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_FIN11"
            model="account.account.tag">
            <field name="name">GuV FIN11</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_FIN12"
            model="account.account.tag">
            <field name="name">GuV FIN12</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_FIN13"
            model="account.account.tag">
            <field name="name">GuV FIN13</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_FIN14"
            model="account.account.tag">
            <field name="name">GuV FIN14</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_FIN15"
            model="account.account.tag">
            <field name="name">GuV FIN15</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_TAX"
            model="account.account.tag">
            <field name="name">GuV TAX</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_MTAX"
            model="account.account.tag">
            <field name="name">GuV MTAX</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
        <record
            id="account_tag_l10n_at_RCR"
            model="account.account.tag">
            <field name="name">GuV RCR</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_RRR"
            model="account.account.tag">
            <field name="name">GuV RRR</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ARR"
            model="account.account.tag">
            <field name="name">GuV ARR</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_RL"
            model="account.account.tag">
            <field name="name">GuV RL</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
            <field name="country_id" ref="base.at"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.at"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_l10n_at_non_tva_sale_report_title" model="account.report.line">
                <field name="name">3. Aufschlüsselung für die Zusammenfassende Meldung (ZM)</field>
                <field name="sequence" eval="5"/>
                <field name="aggregation_formula">(AT_ZM_IGL.balance + AT_ZM_IGL3.balance + AT_ZM_DL.balance)</field>
                <field name="children_ids">
                    <record id="tax_report_line_l10n_at_tva_line_3_zm_igl" model="account.report.line">
                        <field name="name">Innergemeinschaftliche Lieferungen</field>
                        <field name="code">AT_ZM_IGL</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_3_zm_igl_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">AT_ZM_IGL</field>
                            </record>
                        </field>
                        <field name="sequence" eval="10"/>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_3_zm_igl3" model="account.report.line">
                        <field name="name">Innergemeinschaftliche Lieferungen (Dreiecksgeschäfte)</field>
                        <field name="code">AT_ZM_IGL3</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_3_zm_igl3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">AT_ZM_IGL3</field>
                            </record>
                        </field>
                        <field name="sequence" eval="20"/>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_3_zm_dl" model="account.report.line">
                        <field name="name">Grenzüberschreitende Dienstleistungen (Sonstige Leistungen)</field>
                        <field name="code">AT_ZM_DL</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_3_zm_dl_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">AT_ZM_DL</field>
                            </record>
                        </field>
                        <field name="sequence" eval="30"/>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_l10n_at_tva_sale_report_title" model="account.report.line">
                <field name="name">4. VAT Computation (U1/U30)</field>
                <field name="aggregation_formula">(AT_022_tax.balance + AT_029_tax.balance + AT_006_tax.balance + AT_037_tax.balance + AT_052_tax.balance + AT_007_tax.balance + AT_056.balance + AT_057.balance + AT_048.balance + AT_044.balance + AT_032.balance + AT_072_tax.balance + AT_073_tax.balance + AT_008_tax.balance + AT_088_tax.balance)</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_l10n_at_tva_line_4_1" model="account.report.line">
                        <field name="name">4.1 Total amount of the taxable base for deliveries and other services [000]</field>
                        <field name="code">AT_000</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 000</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_2" model="account.report.line">
                        <field name="name">4.2 Plus own consumption (Section 1(1)(2), Section 3(2) and Section 3a(1a)) [001]</field>
                        <field name="code">AT_001</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 001</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_3" model="account.report.line">
                        <field name="name">4.3 Minus revenue for which the tax liability is the beneficiary according to Section 19(1) [021]</field>
                        <field name="code">AT_021</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 021</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_4" model="account.report.line">
                        <field name="name">4.4 Total</field>
                        <field name="aggregation_formula">AT_000.balance + AT_001.balance - AT_021.balance</field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_sale_01_report_title" model="account.report.line">
                        <field name="name">Tax-exempt WITH input tax deduction according to</field>
                        <field name="aggregation_formula">AT_011.balance + AT_012.balance + AT_015.balance + AT_017.balance + AT_018.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_5" model="account.report.line">
                                <field name="name">4.5 Section 6(1)(1) in conjunction with Section 7 (export deliveries) [011]</field>
                                <field name="code">AT_011</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 011</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_6" model="account.report.line">
                                <field name="name">4.6 Section 6(1)(1) in conjunction with Section 8 (contract processing) [012]</field>
                                <field name="code">AT_012</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_6_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 012</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_7" model="account.report.line">
                                <field name="name">4.7 Section 6(1)(2)-(6) and Section 23(5) (seafaring, aviation, ...) [015]</field>
                                <field name="code">AT_015</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 015</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_8" model="account.report.line">
                                <field name="name">4.8 Article 6(1) (Intra-Community deliveries excluding deliveries of vehicles to be specified separately below) [017]</field>
                                <field name="code">AT_017</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 017</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_9" model="account.report.line">
                                <field name="name">4.9 Article 6(1), if deliveries of new vehicles were made to customers without a VAT number or by vehicle suppliers pursuant to article. 2 [018]</field>
                                <field name="code">AT_018</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_9_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 018</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_sale_02_report_title" model="account.report.line">
                        <field name="name">Tax-exempt WITHOUT input tax deduction according to</field>
                        <field name="aggregation_formula">AT_019.balance + AT_016.balance + AT_020.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_10" model="account.report.line">
                                <field name="name">4.10 Section 6(1)(9)(a) (land sales) [019]</field>
                                <field name="code">AT_019</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 019</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_11" model="account.report.line">
                                <field name="name">4.11 Section 6(1)(27) (small business) [016]</field>
                                <field name="code">AT_016</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_11_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 016</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_12" model="account.report.line">
                                <field name="name">4.12 Section 6(1)(...) (other tax-exempt transactions without deduction of input tax) [020]</field>
                                <field name="code">AT_020</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_12_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 020</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_13" model="account.report.line">
                        <field name="name">4.13 Total amount of taxable deliveries, other services and own consumption (including taxable advance payments)</field>
                        <field name="aggregation_formula">(AT_000.balance + AT_001.balance - AT_021.balance) - AT_011.balance - AT_012.balance - AT_015.balance - AT_017.balance - AT_018.balance - AT_019.balance - AT_016.balance - AT_020.balance</field>
                    </record>
                    <record id="tax_report_line_at_base_title_umsatz_base_4_14_19" model="account.report.line">
                        <field name="name">Taxable base</field>
                        <field name="aggregation_formula">AT_022_base.balance + AT_029_base.balance + AT_006_base.balance + AT_037_base.balance + AT_052_base.balance + AT_007_base.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_14_base" model="account.report.line">
                                <field name="name">4.14 20% Standard rate [022]</field>
                                <field name="code">AT_022_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_14_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 022 Bemessungsgrundlage</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_15_base" model="account.report.line">
                                <field name="name">4.15 10% Reduced rate [029]</field>
                                <field name="code">AT_029_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_15_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 029 Bemessungsgrundlage</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_16_base" model="account.report.line">
                                <field name="name">4.16 13% Reduced rate [006]</field>
                                <field name="code">AT_006_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_16_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 006 Bemessungsgrundlage</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_17_base" model="account.report.line">
                                <field name="name">4.17 19% for Jungholz and Mittelberg [037]</field>
                                <field name="code">AT_037_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_17_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 037 Bemessungsgrundlage</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_18_base" model="account.report.line">
                                <field name="name">4.18 10% Additional tax for flat-rate agricultural and forestry holdings [052]</field>
                                <field name="code">AT_052_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_18_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 052 Bemessungsgrundlage</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_19_base" model="account.report.line">
                                <field name="name">4.19 7% Additional tax for flat-rate agricultural and forestry holdings [007]</field>
                                <field name="code">AT_007_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_19_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 007 Bemessungsgrundlage</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_at_tax_title_4_14_19" model="account.report.line">
                        <field name="name">VAT</field>
                        <field name="aggregation_formula">AT_022_tax.balance + AT_029_tax.balance + AT_006_tax.balance + AT_037_tax.balance + AT_052_tax.balance + AT_007_tax.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_14_tax" model="account.report.line">
                                <field name="name">4.14 20% Standard rate</field>
                                <field name="code">AT_022_tax</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_14_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 022 Umsatzsteuer</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_15_tax" model="account.report.line">
                                <field name="name">4.15 10% Reduced rate</field>
                                <field name="code">AT_029_tax</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_15_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 029 Umsatzsteuer</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_16_tax" model="account.report.line">
                                <field name="name">4.16 13% Reduced rate</field>
                                <field name="code">AT_006_tax</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_16_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 006 Umsatzsteuer</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_17_tax" model="account.report.line">
                                <field name="name">4.17 19% for Jungholz and Mittelberg</field>
                                <field name="code">AT_037_tax</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_17_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 037 Umsatzsteuer</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_18_tax" model="account.report.line">
                                <field name="name">4.18 10% Additional tax for flat-rate agricultural and forestry holdings</field>
                                <field name="code">AT_052_tax</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_18_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 052 Umsatzsteuer</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_19_tax" model="account.report.line">
                                <field name="name">4.19 7% Additional tax for flat-rate agricultural and forestry holdings</field>
                                <field name="code">AT_007_tax</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_19_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 007 Umsatzsteuer</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_20" model="account.report.line">
                        <field name="name">4.20 Tax liability pursuant to Section 11(12,14), Section 16(2) and pursuant to Art. 7(4) [056]</field>
                        <field name="code">AT_056</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_20_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 056</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_21" model="account.report.line">
                        <field name="name">4.21 Tax liability pursuant to Section 19(1) second sentence, Section 19(1c,1e) and pursuant to Art. 25(5) [057]</field>
                        <field name="code">AT_057</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_21_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 057</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_22" model="account.report.line">
                        <field name="name">4.22 Tax liability according to Section 19(1a) (construction services) [048]</field>
                        <field name="code">AT_048</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_22_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 048</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_23" model="account.report.line">
                        <field name="name">4.23 Tax liability under Section 19(1b) (security property, ...) [044]</field>
                        <field name="code">AT_044</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_23_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 044</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_24" model="account.report.line">
                        <field name="name">4.24 Tax liability under Section 19(1d) (scrap and waste materials) [032]</field>
                        <field name="code">AT_032</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_24_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 032</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_sale_03_report_title" model="account.report.line">
                        <field name="name">Intra-Community acquisition</field>
                        <field name="aggregation_formula">AT_070.balance + AT_071.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_25" model="account.report.line">
                                <field name="name">4.25 Total amount of taxable amounts for intra-Community acquisitions [070]</field>
                                <field name="code">AT_070</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_25_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 070</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_26" model="account.report.line">
                                <field name="name">4.26 Tax-exempt under Art. 6(2) [071]</field>
                                <field name="code">AT_071</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_26_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 071</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_27" model="account.report.line">
                                <field name="name">4.27 Total amount of taxable intra-Community acquisitions</field>
                                <field name="aggregation_formula">AT_070.balance - AT_071.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_sale_04_report_title" model="account.report.line">
                        <field name="name">Of which taxable with</field> <!-- TODO: -->
                        <field name="aggregation_formula">AT_072_base.balance + AT_073_base.balance + AT_008_base.balance + AT_088_base.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_at_base_title_umsatz_base_4_28_31" model="account.report.line">
                                <field name="name">Taxable base</field>
                                <field name="aggregation_formula">AT_072_base.balance + AT_073_base.balance + AT_008_base.balance + AT_088_base.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_28_base" model="account.report.line">
                                        <field name="name">4.28 20% Standard rate [072]</field>
                                        <field name="code">AT_072_base</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_28_base_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 072 Bemessungsgrundlage</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_l10n_at_tva_line_4_29_base" model="account.report.line">
                                        <field name="name">4.29 10% Reduced rate [073]</field>
                                        <field name="code">AT_073_base</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_29_base_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 073 Bemessungsgrundlage</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_l10n_at_tva_line_4_30_base" model="account.report.line">
                                        <field name="name">4.30 13% Reduced rate [008]</field>
                                        <field name="code">AT_008_base</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_30_base_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 008 Bemessungsgrundlage</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_l10n_at_tva_line_4_31_base" model="account.report.line">
                                        <field name="name">4.31 19% for Jungholz and Mittelberg [088]</field>
                                        <field name="code">AT_088_base</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_31_base_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 088 Bemessungsgrundlage</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_at_tax_title_4_28_31" model="account.report.line">
                                <field name="name">VAT</field>
                                <field name="aggregation_formula">AT_072_tax.balance + AT_073_tax.balance + AT_008_tax.balance + AT_088_tax.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_28_tax" model="account.report.line">
                                        <field name="name">4.28 20% Standard rate</field>
                                        <field name="code">AT_072_tax</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_28_tax_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 072 Umsatzsteuer</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_l10n_at_tva_line_4_29_tax" model="account.report.line">
                                        <field name="name">4.29 10% Reduced rate</field>
                                        <field name="code">AT_073_tax</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_29_tax_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 073 Umsatzsteuer</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_l10n_at_tva_line_4_30_tax" model="account.report.line">
                                        <field name="name">4.30 13% Reduced rate</field>
                                        <field name="code">AT_008_tax</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_30_tax_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 008 Umsatzsteuer</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_l10n_at_tva_line_4_31_tax" model="account.report.line">
                                        <field name="name">4.31 19% for Jungholz and Mittelberg</field>
                                        <field name="code">AT_088_tax</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_31_tax_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 088 Umsatzsteuer</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_sale_05_report_title" model="account.report.line">
                        <field name="name">Non-taxable acquisitions</field>
                        <field name="aggregation_formula">AT_076.balance + AT_077.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_32" model="account.report.line">
                                <field name="name">4.32 Acquisitions pursuant to Art. 3(8), second sentence, that have been taxed in the Member State of destination [076]</field>
                                <field name="code">AT_076</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_32_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 076</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_33" model="account.report.line">
                                <field name="name">4.33 Acquisitions under Art. 3(8), second sentence, deemed to be taxed domestically under Art. 25(2) [077]</field>
                                <field name="code">AT_077</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_33_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 077</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_l10n_at_tva_purchase_report_title" model="account.report.line">
                <field name="name">5. Deductible input tax computation</field>
                <field name="aggregation_formula">AT_060.balance + AT_061.balance + AT_083.balance + AT_065.balance + AT_066.balance + AT_082.balance + AT_087.balance + AT_089.balance + AT_064.balance + AT_062.balance + AT_063.balance + AT_067.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_l10n_at_tva_line_5_1" model="account.report.line">
                        <field name="name">5.1 Total amount of input taxes (excluding amounts to be shown separately below) [060]</field>
                        <field name="code">AT_060</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 060</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_2" model="account.report.line">
                        <field name="name">5.2 Input tax relating to import turnover tax paid (Section 12(1)(2)(a)) [061]</field>
                        <field name="code">AT_061</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 061</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_3" model="account.report.line">
                        <field name="name">5.3 Input tax concerning the import turnover tax owed and entered in the tax account (Section 12(1)(2)(b)) [083]</field>
                        <field name="code">AT_083</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 083</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_4" model="account.report.line">
                        <field name="name">5.4 Input taxes from intra-Community acquisition [065]</field>
                        <field name="code">AT_065</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_4_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 065</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_5" model="account.report.line">
                        <field name="name">5.5 Input taxes concerning the tax liability pursuant to Section 19(1) second sentence, Section 19(1c,1e) and pursuant to Art. 25(5) [066]</field>
                        <field name="code">AT_066</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 066</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_6" model="account.report.line">
                        <field name="name">5.6 Input taxes relating to tax liability pursuant to Section 19(1a) (construction services) [082]</field>
                        <field name="code">AT_082</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_6_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 082</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_7" model="account.report.line">
                        <field name="name">5.7 Input taxes relating to tax liability under Section 19(1b) (security property, ...) [087]</field>
                        <field name="code">AT_087</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_7_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 087</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_8" model="account.report.line">
                        <field name="name">5.8 Input taxes relating to tax liability under Section 19(1d) (scrap and waste materials) [089]</field>
                        <field name="code">AT_089</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_8_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 089</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_9" model="account.report.line">
                        <field name="name">5.9 Input taxes for intra-Community deliveries of new vehicles by vehicle suppliers under Art. 2 [064]</field>
                        <field name="code">AT_064</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_9_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 064</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_10" model="account.report.line">
                        <field name="name">5.10 Not deductible under Section 12(3) in conjunction with Subsections 4 and 5 [062]</field>
                        <field name="code">AT_062</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 062</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_11" model="account.report.line">
                        <field name="name">5.11 Correction pursuant to Section 12(10,11) [063]</field>
                        <field name="code">AT_063</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 063</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_12" model="account.report.line">
                        <field name="name">5.12 Correction pursuant to Section 16 [067]</field>
                        <field name="code">AT_067</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 067</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_13" model="account.report.line">
                        <field name="name">5.13 Total amount of deductible input tax</field>
                        <field name="aggregation_formula">AT_060.balance + AT_061.balance + AT_083.balance + AT_065.balance + AT_066.balance + AT_082.balance + AT_087.balance + AT_089.balance + AT_064.balance + AT_062.balance + AT_063.balance + AT_067.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_l10n_at_tva_final_report_title" model="account.report.line">
                <field name="name">6. Other corrections</field>
                <field name="aggregation_formula">AT_090.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_line_l10n_at_tva_line_6" model="account.report.line">
                        <field name="name">Other corrections [090]</field>
                        <field name="code">AT_090</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_6_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 090</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_l10n_at_tva_line_7" model="account.report.line">
                <field name="name">7. Prepayment/debit (-) or credit/surplus (+) [095]</field>
                <field name="aggregation_formula">(AT_022_tax.balance + AT_029_tax.balance + AT_006_tax.balance + AT_037_tax.balance + AT_052_tax.balance + AT_007_tax.balance + AT_056.balance + AT_057.balance + AT_048.balance + AT_044.balance + AT_032.balance + AT_072_tax.balance + AT_073_tax.balance + AT_008_tax.balance + AT_088_tax.balance + AT_060.balance + AT_061.balance + AT_083.balance + AT_065.balance + AT_066.balance + AT_082.balance + AT_087.balance + AT_089.balance + AT_064.balance + AT_062.balance + AT_063.balance + AT_067.balance) + AT_090.balance</field>
                <field name="hierarchy_level">0</field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\res.country.state.csv

```csv
"id","country_id:id","name","code"
state_at_1,base.at,"Burgenland","1"
state_at_2,base.at,"Kärnten","2"
state_at_3,base.at,"Niederösterreich","3"
state_at_4,base.at,"Oberösterreich","4"
state_at_5,base.at,"Salzburg","5"
state_at_6,base.at,"Steiermark","6"
state_at_7,base.at,"Tirol","7"
state_at_8,base.at,"Vorarlberg","8"
state_at_9,base.at,"Wien","9"

```

## File: data\template\account.account-at.csv

```csv
"id","name","code","reconcile","account_type","tag_ids","name@de"
"chart_at_template_transfer_288","Money in transit","2880","True","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2885","Schwebende Geldbewegungen"
"chart_at_template_0010","Start-up expenses","0010","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0010","Aufwendungen für das Ingangsetzen eines Betriebes"
"chart_at_template_0019","Accumulated depreciation of start-up expenses","0019","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0019","Kumulierte Abschreibungen für das Ingangsetzen eines Betriebes"
"chart_at_template_0020","Expansion expenses","0020","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0020","Aufwendungen für das Erweitern eines Betriebes"
"chart_at_template_0029","Accumulated depreciation of expansion expenses","0029","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0029","Kumulierte Abschreibungen für das Erweitern eines Betriebes"
"chart_at_template_0100","Concessions","0100","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0100","Konzessionen"
"chart_at_template_0110","Patent rights","0110","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0110","Patentrechte"
"chart_at_template_0112","Licensing rights","0112","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0112","Lizenzen"
"chart_at_template_0119","Accumulated deprecation of licensing and patent rights","0119","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0119","Kumulierte Abschreibungen für Lizenzen und Patentrechte"
"chart_at_template_0120","Software","0120","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0120","Software"
"chart_at_template_0129","Accumulated depreciation of software","0129","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0129","Kumulierte Abschreibungen für Software"
"chart_at_template_0130","Brands","0130","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0130","Marken"
"chart_at_template_0131","Trademarks","0131","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0131","Warenzeichen"
"chart_at_template_0132","Design rights","0132","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0132","Musterschutzrechte"
"chart_at_template_0133","Other copyrights","0133","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0133","Sonstige Urheberrechte"
"chart_at_template_0140","Leasing and rental rights","0140","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0140","Pacht- und Mietrechte"
"chart_at_template_0149","Accumlated depreciation of industrial property rights, similar","0149","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI1,l10n_at.account_tag_external_code_0149","Kumulierte Abschreibungen für gewerbliche Schutzrechte, ähnlich"
"chart_at_template_0150","Goodwill","0150","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI2,l10n_at.account_tag_external_code_0150","Geschäfts(Firmen)wert"
"chart_at_template_0159","Accumulated depreciation of goodwill","0159","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI2,l10n_at.account_tag_external_code_0159","Kumulierte Abschreibungen für Geschäfts(Firmen)wert"
"chart_at_template_0180","Prepayments for intangible fixed assets","0180","False","asset_non_current","l10n_at.account_tag_l10n_at_AAI3,l10n_at.account_tag_external_code_0180","Geleistete Anzahlungen für immaterielles Vermögen"
"chart_at_template_0200","Undeveloped land","0200","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII1,l10n_at.account_tag_external_code_0200","Unbebaute Grundstücke"
"chart_at_template_0210","Developed land (Land value)","0210","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII1,l10n_at.account_tag_external_code_0210","Bebaute Grundstücke (Grundwert)"
"chart_at_template_022","Land rights","0220","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII1,l10n_at.account_tag_external_code_0220","Grundstücksgleiche Rechte"
"chart_at_template_0300","Operating, commercial buildings on own land","0300","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII1,l10n_at.account_tag_external_code_0300","Betriebs- und Geschäftsgebäude auf eigenem Grund"
"chart_at_template_0310","Residential, social buildings on own land","0310","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII1,l10n_at.account_tag_external_code_0310","Wohn- und Sozialgebäude auf eigenem Grund"
"chart_at_template_0320","Operating, commercial buildings on third-party land","0320","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII1,l10n_at.account_tag_external_code_0320","Betriebs- und Geschäftsgebäude auf fremdem Grund"
"chart_at_template_0330","Residential, social buildings on third-party land","0330","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII1,l10n_at.account_tag_external_code_0330","Wohn- und Sozialgebäude auf fremdem Grund"
"chart_at_template_0340","Site improvements on own land","0340","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII1,l10n_at.account_tag_external_code_0340","Grundstückseinrichtungen auf eigenem Grund"
"chart_at_template_0349","Accumulated depreciation of buildings on own land","0349","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII1,l10n_at.account_tag_external_code_0349","Kumulierte Abschreibungen für Gebäude auf eigenem Grund"
"chart_at_template_0350","Site improvements on third-part land","0350","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII1,l10n_at.account_tag_external_code_0350","Grundstückseinrichtungen auf fremdem Grund"
"chart_at_template_0359","Accumulated depreciation of buildings on third-party land","0359","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII1,l10n_at.account_tag_external_code_0359","Kumulierte Abschreibungen für Gebäude auf fremden Grund"
"chart_at_template_0360","Structural improvements to third-party (leased) operating and commercial buildings","0360","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII1,l10n_at.account_tag_external_code_0360","Bauliche Investitionen in fremden (gepachteten) Betriebs- und Geschäftsgebäuden"
"chart_at_template_0370","Structural improvements to third-party (leased) residential and social buildings","0370","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII1,l10n_at.account_tag_external_code_0370","Bauliche Investitionen in fremden (gepachteten) Wohn- und Sozialgebäuden"
"chart_at_template_0379","Accumulated depreciation for fixtures in third-party buildings","0379","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII1,l10n_at.account_tag_external_code_0379","Kumulierte Abschreibungen für Einbauten in fremden Gebäuden"
"chart_at_template_0400","Production machinery","0400","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII2,l10n_at.account_tag_external_code_0400","Fertigungsmaschinen"
"chart_at_template_0410","Drive machinery","0410","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII2,l10n_at.account_tag_external_code_0410","Antriebsmaschinen"
"chart_at_template_0420","Power supply systems","0420","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII2,l10n_at.account_tag_external_code_0420","Energieversorgungsanlagen"
"chart_at_template_0430","Transportation systems","0430","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII2,l10n_at.account_tag_external_code_0430","Transportanlagen"
"chart_at_template_0500","Machine tools","0500","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII2,l10n_at.account_tag_external_code_0500","Maschinenwerkzeuge"
"chart_at_template_0510","General and hand tools","0510","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII2,l10n_at.account_tag_external_code_0510","Allgemeine Werkzeuge und Handwerkzeuge"
"chart_at_template_0520","Devices, dies and models","0520","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII2,l10n_at.account_tag_external_code_0520","Vorrichtungen, Formen und Modelle"
"chart_at_template_0530","Other manufacturing resources","0530","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII2,l10n_at.account_tag_external_code_0530","Andere Erzeugungshilfsmittel"
"chart_at_template_0540","Lifting devices and assembly systems","0540","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII2,l10n_at.account_tag_external_code_0540","Hebezeuge und Montageanlagen"
"chart_at_template_0550","Low-value assets used in the production process (machinery)","0550","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII2,l10n_at.account_tag_external_code_0550","Geringwertige Vermögensgegenstände, soweit im Erzeugungsprozeß verwendet (Maschinen)"
"chart_at_template_0600","Heating and lighting systems","0600","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII3,l10n_at.account_tag_external_code_0600","Beheizungs- und Beleuchtungsanlagen"
"chart_at_template_0610","Messaging and control devices","0610","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII3,l10n_at.account_tag_external_code_0610","Nachrichten- und Kontrollanlagen"
"chart_at_template_0620","Office machines, IT systems","0620","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII3,l10n_at.account_tag_external_code_0620","Büromaschinen, EDV-Anlagen"
"chart_at_template_0630","Cars","0630","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII3,l10n_at.account_tag_external_code_0630","PKW"
"chart_at_template_0640","Commercial vehicles","0640","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII3,l10n_at.account_tag_external_code_0640","LKW"
"chart_at_template_0650","Other transportation resources","0650","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII3,l10n_at.account_tag_external_code_0650","Andere Beförderungsmittel"
"chart_at_template_0660","Other operating and office equipment","0660","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII3,l10n_at.account_tag_external_code_0660","Andere Betriebs- und Geschäftsausstattung"
"chart_at_template_0670","Containers","0670","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII3,l10n_at.account_tag_external_code_0670","Gebinde"
"chart_at_template_0680","Low value office machines, IT systems","0680","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII3,l10n_at.account_tag_external_code_0680","Geringwertige Büromaschinen, EDV-Anlagen"
"chart_at_template_0681","Low value operating and office equipment","0681","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII3,l10n_at.account_tag_external_code_0680","Geringwertige Betriebs- und Geschäftsausstattung"
"chart_at_template_0692","Accumulated depreciation of office machines, IT systems","0692","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII3,l10n_at.account_tag_external_code_0659","Kumulierte Abschreibungen zu Büromaschinen, EDV-Anlagen"
"chart_at_template_0693","Accumulated depreciation of cars","0693","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII3,l10n_at.account_tag_external_code_0655","Kumulierte Abschreibungen zu PKW"
"chart_at_template_0694","Accumulated depreciation of commercial vehicles","0694","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII3,l10n_at.account_tag_external_code_0655","Kumulierte Abschreibungen zu LKW"
"chart_at_template_0696","Accumulated depreciation of operating and office equipment","0696","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII3,l10n_at.account_tag_external_code_0689","Kumulierte Abschreibungen zur Betriebs- und Geschäftsausstattung"
"chart_at_template_0700","Prepayments for tangible fixed assets 20 %","0700","False","asset_prepayments","l10n_at.account_tag_l10n_at_AAII4,l10n_at.account_tag_external_code_0700","Anzahlungen für Sachanlagen 20%"
"chart_at_template_0701","Prepayments for tangible fixed assets 10 %","0701","False","asset_prepayments","l10n_at.account_tag_l10n_at_AAII4,l10n_at.account_tag_external_code_0701","Anzahlungen für Sachanlagen 10%"
"chart_at_template_0702","Prepayments for tangible fixed assets 0 %","0702","False","asset_prepayments","l10n_at.account_tag_l10n_at_AAII4,l10n_at.account_tag_external_code_0702","Anzahlungen für Sachanlagen 0%"
"chart_at_template_0710","Assets under construction","0710","False","asset_fixed","l10n_at.account_tag_l10n_at_AAII4,l10n_at.account_tag_external_code_0710","Anlagen in Bau"
"chart_at_template_0800","Shares in affiliated companies","0800","False","asset_non_current","l10n_at.account_tag_l10n_at_AAIII1,l10n_at.account_tag_external_code_0800","Anteile an verbundenen Unternehmen"
"chart_at_template_0810","Investments in joint ventures","0810","False","asset_non_current","l10n_at.account_tag_l10n_at_AAIII1,l10n_at.account_tag_external_code_0810","Beteiligungen an Gemeinschaftsunternehmen"
"chart_at_template_0820","Investments in associates","0820","False","asset_non_current","l10n_at.account_tag_l10n_at_AAIII1,l10n_at.account_tag_external_code_0820","Beteiligungen an angeschlossenen (assoziierten) Unternehmen"
"chart_at_template_0830","Other investments","0830","False","asset_non_current","l10n_at.account_tag_l10n_at_AAIII1,l10n_at.account_tag_external_code_0830","Sonstige Beteiligungen"
"chart_at_template_0840","Loans to affiliated companies","0840","False","asset_non_current","l10n_at.account_tag_l10n_at_AAIII2,l10n_at.account_tag_external_code_0840","Ausleihungen an verbundene Unternehmen"
"chart_at_template_0850","Loans to other long-term investees and investors","0850","False","asset_non_current","l10n_at.account_tag_l10n_at_AAIII4,l10n_at.account_tag_external_code_0850","Ausleihungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht"
"chart_at_template_0860","Other Loans","0860","False","asset_non_current","l10n_at.account_tag_l10n_at_AAIII6,l10n_at.account_tag_external_code_0860","Sonstige Ausleihungen"
"chart_at_template_0870","Shares in corporations of a non-participating nature","0870","False","asset_non_current","l10n_at.account_tag_l10n_at_AAIII5,l10n_at.account_tag_external_code_0870","Anteile an Kapitalgesellschaften ohne Beteiligungscharakter"
"chart_at_template_0880","Shares in partnerships of a non-participating nature","0880","False","asset_non_current","l10n_at.account_tag_l10n_at_AAIII5,l10n_at.account_tag_external_code_0880","Anteile an Personengesellschaften ohne Beteiligungscharakter"
"chart_at_template_0900","Cooperative shares of a non-participating nature","0900","False","asset_non_current","l10n_at.account_tag_l10n_at_AAIII5,l10n_at.account_tag_external_code_0900","Genossenschaftsanteile ohne Beteiligungscharakter"
"chart_at_template_0910","Shares in investment funds","0910","False","asset_non_current","l10n_at.account_tag_l10n_at_AAIII5,l10n_at.account_tag_external_code_0910","Anteile an Investmentfonds"
"chart_at_template_0980","Prepayments for financial assets","0980","False","asset_prepayments","l10n_at.account_tag_l10n_at_AAIII5,l10n_at.account_tag_external_code_0980","Geleistete Anzahlungen für Finanzanlagen"
"chart_at_template_0990","Accumulated Depreciation","0990","False","asset_non_current","l10n_at.account_tag_l10n_at_AAIII5,l10n_at.account_tag_external_code_0990","Kumulierte Abschreibungen"
"chart_at_template_1600","Merchandise","1600","False","asset_current","l10n_at.account_tag_l10n_at_ABI3,l10n_at.account_tag_external_code_1600","Handelswaren"
"chart_at_template_1800","Prepayments for inventories 20%","1800","False","asset_prepayments","l10n_at.account_tag_l10n_at_ABI5,l10n_at.account_tag_external_code_1800","Geleistete Anzahlungen auf Vorräte 20 %"
"chart_at_template_1801","Prepayments for inventories 10%","1801","False","asset_prepayments","l10n_at.account_tag_l10n_at_ABI5,l10n_at.account_tag_external_code_1801","Geleistete Anzahlungen auf Vorräte 10 %"
"chart_at_template_1803","Prepayments for inventories 0%","1803","False","asset_prepayments","l10n_at.account_tag_l10n_at_ABI5,l10n_at.account_tag_external_code_1803","Geleistete Anzahlungen auf Vorräte 0 %"
"chart_at_template_2000","Trade receivables, domestic","2000","True","asset_receivable","l10n_at.account_tag_l10n_at_ABII1,l10n_at.account_tag_external_code_2000","Forderungen aus Lieferungen und Leistungen Inland"
"chart_at_template_2099","Trade receivables, domestic (point of sale)","2099","True","asset_receivable","l10n_at.account_tag_l10n_at_ABII1,l10n_at.account_tag_external_code_2000","Forderungen aus Lieferungen und Leistungen Inland (Point Of Sale)"
"chart_at_template_2080","Specific valuation allowances on trade receivables, domestic","2080","False","asset_current","l10n_at.account_tag_l10n_at_ABII1,l10n_at.account_tag_external_code_2080","Einzelwertberichtigungen zu Forderungen aus Lieferungen und Leistungen Inland"
"chart_at_template_2090","Global valuation allowances on trade receivables, domestic","2090","False","asset_current","l10n_at.account_tag_l10n_at_ABII1,l10n_at.account_tag_external_code_2090","Pauschalwertberichtigungen zu Forderungen aus Lieferungen und Leistungen Inland"
"chart_at_template_2100","Trade receivables, EU area","2100","True","asset_receivable","l10n_at.account_tag_l10n_at_ABII1,l10n_at.account_tag_external_code_2100","Forderungen aus Lieferungen und Leistungen EU-Raum"
"chart_at_template_2130","Specific valuation allowances on trade receivables, EU area","2130","False","asset_current","l10n_at.account_tag_l10n_at_ABII1,l10n_at.account_tag_external_code_2130","Einzelwertberichtigungen zu Forderungen aus Lieferungen und Leistungen EU-Raum"
"chart_at_template_2140","Global valuation allowances on trade receivables, EU area","2140","False","asset_current","l10n_at.account_tag_l10n_at_ABII1,l10n_at.account_tag_external_code_2140","Pauschalwertberichtigungen zu Forderungen aus Lieferungen und Leistungen EU-Raum"
"chart_at_template_2150","Trade receivables, international","2150","True","asset_receivable","l10n_at.account_tag_l10n_at_ABII1,l10n_at.account_tag_external_code_2150","Forderungen aus Lieferungen und Leistungen sonstiges Ausland"
"chart_at_template_2180","Specific valuation allowances on trade receivables, international","2180","False","asset_current","l10n_at.account_tag_l10n_at_ABII1,l10n_at.account_tag_external_code_2180","Einzelwertberichtigungen zu Forderungen aus Lieferungen und Leistungen sonstiges Ausland"
"chart_at_template_2190","Global valuation allowances on trade receivables, international","2190","False","asset_current","l10n_at.account_tag_l10n_at_ABII1,l10n_at.account_tag_external_code_2190","Pauschalwertberichtigungen zu Forderungen aus Lieferungen und Leistungen sonstiges Ausland"
"chart_at_template_2230","Specific valuation allowances on receivables from affiliated companies","2230","False","asset_current","l10n_at.account_tag_l10n_at_ABII2,l10n_at.account_tag_external_code_2230","Einzelwertberichtigungen zu Forderungen gegenüber verbundenen Unternehmen"
"chart_at_template_2240","Global valuation allowances on receivables from affiliated companies","2240","False","asset_current","l10n_at.account_tag_l10n_at_ABII2,l10n_at.account_tag_external_code_2240","Pauschalwertberichtigungen zu Forderungen gegenüber verbundenen Unternehmen"
"chart_at_template_2280","Specific valuation allowances on receivables from other long-term investees and investors","2280","False","asset_current","l10n_at.account_tag_l10n_at_ABII3,l10n_at.account_tag_external_code_2280","Einzelwertberichtigungen zu Forderungen gegenüber Unternehmen, mit denen ein Beteiligungsverhältnis besteht"
"chart_at_template_2290","Global valuation allowances on receivables from other long-term investees and investors","2290","False","asset_current","l10n_at.account_tag_l10n_at_ABII3,l10n_at.account_tag_external_code_2290","Pauschalwertberichtigungen zu Forderungen gegenüber Unternehmen, mit denen ein Beteiligungsverhältnis besteht"
"chart_at_template_2300","Other receivables and other assets","2300","False","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2300","Sonstige Forderungen und Vermögensgegenstände"
"chart_at_template_2470","Unpaid called capital contributions","2470","False","asset_current","l10n_at.account_tag_l10n_at_ABII3,l10n_at.account_tag_external_code_2470","Eingeforderte, aber noch nicht eingezahlte Einlagen"
"chart_at_template_2480","Specific valuation allowances on other receivables and other assets","2480","False","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2480","Einzelwertberichtigungen zu sonstigen Forderungen und Vermögensgegenständen"
"chart_at_template_2490","Global valuation allowances on other receivables and other assets","2490","False","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2490","Pauschalwertberichtigungen zu sonstigen Forderungen und Vermögensgegenständen"
"chart_at_template_2500","Input tax 20%","2500","False","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2500","Vorsteuern 20%"
"chart_at_template_2501","Input tax 10%","2501","False","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2500","Vorsteuern 10%"
"chart_at_template_2502","Input tax 13%","2502","False","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2500","Vorsteuern 13%"
"chart_at_template_2505","Other tax 13%","2505","False","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2500","Sonstige Vorsteuern"
"chart_at_template_2506","Input tax RC 20%","2506","False","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2502","Vorsteuern RC 20%"
"chart_at_template_2507","Input tax RC 10%","2507","False","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2502","Vorsteuern RC 10%"
"chart_at_template_2510","Input tax RC EU 20%","2510","False","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2502","Vorsteuern RC EU-Raum 20%"
"chart_at_template_2511","Input tax on intra-Community acquisitions 20%","2511","False","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2501","Vorsteuern IGE 20%"
"chart_at_template_2512","Input tax on intra-Community acquisitions 10%","2512","False","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2501","Vorsteuern IGE 10%"
"chart_at_template_2513","Input tax on intra-Community acquisitions 13%","2513","False","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2501","Vorsteuern IGE 13%"
"chart_at_template_2515","Input tax 20% (from import VAT)","2515","False","asset_current","l10n_at.account_tag_l10n_at_ABII4,l10n_at.account_tag_external_code_2509","Vorsteuern 20% (aus EUSt.)"
"chart_at_template_2610","Shares in affiliated companies","2610","False","asset_current","l10n_at.account_tag_l10n_at_ABIII1,l10n_at.account_tag_external_code_2610","Anteile an verbundenen Unternehmen"
"chart_at_template_2620","Other shares","2620","False","asset_current","l10n_at.account_tag_l10n_at_ABIII2,l10n_at.account_tag_external_code_2620","Sonstige Anteile"
"chart_at_template_2680","Bills of exchange where the entity is entitled to the underlying receivables","2680","False","asset_current","l10n_at.account_tag_l10n_at_ABIII2,l10n_at.account_tag_external_code_2680","Besitzwechsel, soweit dem Unternehmen nicht die der Ausstellung zugrundeliegenden Forderungen zustehen"
"chart_at_template_2690","Valuation allowances on long-term securities","2690","False","asset_current","l10n_at.account_tag_l10n_at_ABIII2,l10n_at.account_tag_external_code_2690","Wertberichtigungen Wertpapiere des Umlaufvermögens"
"chart_at_template_2730","Postage stamps","2730","False","asset_current","l10n_at.account_tag_l10n_at_ABIV,l10n_at.account_tag_external_code_2730","Postwertzeichen"
"chart_at_template_2740","Stamps","2740","False","asset_current","l10n_at.account_tag_l10n_at_ABIV,l10n_at.account_tag_external_code_2740","Stempelmarken"
"chart_at_template_2780","Cheques in foreign currency","2780","False","asset_current","l10n_at.account_tag_l10n_at_ABIV,l10n_at.account_tag_external_code_2780","Schecks in Inlandswährung"
"chart_at_template_2890","Valuation allowances","2890","False","asset_current","l10n_at.account_tag_l10n_at_ABIV,l10n_at.account_tag_external_code_2890","Wertberichtigungen"
"chart_at_template_2900","Prepaid Expenses","2900","False","asset_current","l10n_at.account_tag_l10n_at_AC,l10n_at.account_tag_external_code_2900","Aktive Rechnungsabgrenzungsposten"
"chart_at_template_2950","Discount","2950","False","asset_current","l10n_at.account_tag_l10n_at_AC,l10n_at.account_tag_external_code_2950","Disagio"
"chart_at_template_2960","Difference to required pension provisions","2960","False","asset_current","l10n_at.account_tag_l10n_at_PBI,l10n_at.account_tag_external_code_2960","Unterschiedsbetrag zur gebotenen Pensionsrückstellung"
"chart_at_template_2970","Difference under section XII Pensionskassengesetz","2970","False","asset_current","l10n_at.account_tag_l10n_at_PBII,l10n_at.account_tag_external_code_2970","Unterschiedsbetrag gem. Abschnitt XII Pensionskassengesetz"
"chart_at_template_2980","Deferred tax assets","2980","False","asset_current","l10n_at.account_tag_l10n_at_AC,l10n_at.account_tag_external_code_2980","Steuerabgrenzung"
"chart_at_template_3000","Provisions for termination benefits","3000","False","liability_non_current","l10n_at.account_tag_l10n_at_PBI,l10n_at.account_tag_external_code_3000","Rückstellungen für Abfertigungen"
"chart_at_template_3010","Provisions for pensions","3010","False","liability_non_current","l10n_at.account_tag_l10n_at_PBII,l10n_at.account_tag_external_code_3010","Rückstellungen für Pensionen"
"chart_at_template_3100","Bonds (including convertible ones)","3100","True","liability_current","l10n_at.account_tag_l10n_at_PCI,l10n_at.account_tag_external_code_3100","Anleihen (einschließlich konvertibler)"
"chart_at_template_3200","Payments received on account of orders 20%","3200","False","liability_current","l10n_at.account_tag_l10n_at_PCIII,l10n_at.account_tag_external_code_3200","Erhaltene Anzahlungen auf Bestellungen 20 %"
"chart_at_template_3201","Payments received on account of orders 10%","3201","False","liability_current","l10n_at.account_tag_l10n_at_PCIII,l10n_at.account_tag_external_code_3201","Erhaltene Anzahlungen auf Bestellungen 10 %"
"chart_at_template_3202","Payments received on account of orders 0%","3202","False","liability_current","l10n_at.account_tag_l10n_at_PCIII,l10n_at.account_tag_external_code_3202","Erhaltene Anzahlungen auf Bestellungen 0 %"
"chart_at_template_3210","VAT contingent/unrecognised transaction account for payments received on account of orders","3210","True","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3210","Umsatzsteuer-Evidenzkonto für erhaltene Anzahlungen auf Bestellungen"
"chart_at_template_3300","Trade payables, domestic","3300","True","liability_payable","l10n_at.account_tag_l10n_at_PCIV,l10n_at.account_tag_external_code_3300","Lieferverbindlichkeiten Inland"
"chart_at_template_3360","Trade payables, EU area","3360","True","liability_payable","l10n_at.account_tag_l10n_at_PCIV,l10n_at.account_tag_external_code_3360","Lieferverbindlichkeiten EU-Raum"
"chart_at_template_3370","Trade payables, other countries","3370","True","liability_payable","l10n_at.account_tag_l10n_at_PCIV,l10n_at.account_tag_external_code_3370","Lieferverbindlichkeiten sonstiges Ausland"
"chart_at_template_3480","Liabilities to partners/shareholders","3480","False","liability_current","l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3480","Verbindlichkeiten gegenüber Gesellschaftern"
"chart_at_template_3500","VAT 20%","3500","False","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3500","Umsatzsteuer 20%"
"chart_at_template_3501","VAT 10%","3501","False","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3500","Umsatzsteuer 10%"
"chart_at_template_3502","VAT 13%","3502","False","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3500","Umsatzsteuer 13%"
"chart_at_template_3505","VAT, other","3505","False","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3500","Sonstige Umsatzsteuer"
"chart_at_template_3510","VAT RC EU 20%","3510","False","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3502","Umsatzsteuer RC EU-Raum 20%"
"chart_at_template_3511","VAT on intra-Community acquisitions 20%","3511","False","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3501","Umsatzsteuer IGE 20%"
"chart_at_template_3512","VAT on intra-Community acquisitions 10%","3512","False","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3501","Umsatzsteuer IGE 10%"
"chart_at_template_3513","VAT on intra-Community acquisitions 13%","3513","False","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3501","Umsatzsteuer IGE 13%"
"chart_at_template_3515","Import VAT 20%","3515","False","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3509","Einfuhrumsatzsteuer 20%"
"chart_at_template_3520","VAT payable","3520","False","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3520","Ust. Zahllast"
"chart_at_template_3530","Allocation account for tax authorities","3530","True","liability_payable","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3530","Verrechnungskonto Finanzamt"
"chart_at_template_3540","Allocation for wage tax","3540","True","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3540","Verrechnung Lohnsteuer"
"chart_at_template_3541","Allocation for employer contributions","3541","True","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3541","Verrechnung Dienstgeberbeitrag"
"chart_at_template_3542","Allocation for supplement to employer contributions","3542","True","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3542","Verrechnung Dienstgeberzuschlag"
"chart_at_template_3550","Allocation for municipal taxes","3550","False","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3550","Verrechnung Kommunalsteuer"
"chart_at_template_3551","Allocation for Vienna employers' tax","3551","False","liability_current","l10n_at.account_tag_l10n_at_PCVIII1,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3551","Verrechnung Wiener Dienstgeberabgabe"
"chart_at_template_3600","Allocation account for social security","3600","True","liability_payable","l10n_at.account_tag_l10n_at_PCVIII2,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3600","Verrechnungskonto Sozialversicherung"
"chart_at_template_3610","Allocation account for municipality","3610","True","liability_payable","l10n_at.account_tag_l10n_at_PCVIII2,l10n_at.account_tag_l10n_at_PCVIII3,l10n_at.account_tag_external_code_3600","Verrechnungskonto Magistrat/Gemeinde (KoSt, U-Bahn, etc.)"
"chart_at_template_3740","Allocation account for goods/invoice receipt","3740","True","liability_payable","l10n_at.account_tag_l10n_at_PD,l10n_at.account_tag_external_code_3700","WERE Verrechnungskonto"
"chart_at_template_4000","Revenue 20%","4000","False","income","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT1,l10n_at.account_tag_external_code_4000","Brutto-Umsatzerlöse im Inland (20%)"
"chart_at_template_4001","Revenue 10%","4001","False","income","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT1,l10n_at.account_tag_external_code_4010","Brutto-Umsatzerlöse im Inland (10%)"
"chart_at_template_4100","Revenue euro-zone RC 20%","4100","False","income","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT1,l10n_at.account_tag_external_code_4100","Brutto-Umsatzerlöse im EU-Raum (RC 20%)"
"chart_at_template_4110","Revenue euro-zone RC 10%","4110","False","income","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT1,l10n_at.account_tag_external_code_4100","Brutto-Umsatzerlöse im EU-Raum (RC 10%)"
"chart_at_template_4200","Revenue international 0%","4200","False","income","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT1,l10n_at.account_tag_external_code_4050","Brutto-Umsatzerlöse in Drittstaaten (0%)"
"chart_at_template_4860","Exchange rate gains from foreign currency transactions","4860","False","income_other","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT1,l10n_at.account_tag_external_code_4860","Kursgewinne aus Fremdwährungstransaktionen"
"chart_at_template_5000","Cost of goods sold","5000","False","expense_direct_cost","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT5I,l10n_at.account_tag_external_code_5000","Wareneinsatz"
"chart_at_template_5010","Purchased merchandise 20%","5010","False","expense_direct_cost","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT5I,l10n_at.account_tag_external_code_5410","Wareneinkauf 20%"
"chart_at_template_5011","Purchased merchandise 10%","5011","False","expense_direct_cost","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT5I,l10n_at.account_tag_external_code_5411","Wareneinkauf 10%"
"chart_at_template_5050","Purchased merchandise 20% (intra-Community acquisitions)","5050","False","expense_direct_cost","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT5I,l10n_at.account_tag_external_code_5320","Wareneinkauf ig. Erwerb 20%"
"chart_at_template_5051","Purchased merchandise 10% (intra-Community acquisitions)","5051","False","expense_direct_cost","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT5I,l10n_at.account_tag_external_code_5310","Wareneinkauf ig. Erwerb 10%"
"chart_at_template_5052","Purchased merchandise 0% (intra-Community acquisitions) according to article 6 (2)","5052","False","expense_direct_cost","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT5I,l10n_at.account_tag_external_code_5330","Wareneinkauf ig. Erwerb 0% nach Art. 6 Abs. 2"
"chart_at_template_5090","Purchased merchandise 0%","5090","False","expense_direct_cost","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT5I,l10n_at.account_tag_external_code_5417","Wareneinkauf 0%"
"chart_at_template_5800","Cash discount income 20%","5800","False","expense_direct_cost","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT5I,l10n_at.account_tag_external_code_5800","Skontoertrag Materialaufwand 20 %"
"chart_at_template_5801","Cash discount income 10%","5801","False","expense_direct_cost","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT5I,l10n_at.account_tag_external_code_5801","Skontoertrag Materialaufwand 10 %"
"chart_at_template_5805","Cash discount income 0%","5805","False","expense_direct_cost","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT5I,l10n_at.account_tag_external_code_5805","Skontoertrag Materialaufwand 0%"
"chart_at_template_5810","Cash discount income 20% purchased services","5810","False","expense_direct_cost","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT5II,l10n_at.account_tag_external_code_5830","Skontoertrag bezogene Leistungen 20 %"
"chart_at_template_5811","Cash discount income 10% purchased services","5811","False","expense_direct_cost","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT5II,l10n_at.account_tag_external_code_5831","Skontoertrag bezogene Leistungen 10 %"
"chart_at_template_5812","Cash discount income 0% purchased services","5812","False","expense_direct_cost","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT5II,l10n_at.account_tag_external_code_5805","Skontoertrag bezogene Leistungen 0 %"
"chart_at_template_5900","Expense items list","5900","False","expense_direct_cost","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT5I,l10n_at.account_tag_external_code_5900","Aufwandsstellenrechnung"
"chart_at_template_6200","Salaries - salaried employees","6200","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6I,l10n_at.account_tag_external_code_6200","Gehälter - Angestellte"
"chart_at_template_6205","Salaries - managing directors","6205","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6I,l10n_at.account_tag_external_code_6205","Geschäftsführerbezug"
"chart_at_template_6220","Non-performance salaries","6220","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6I,l10n_at.account_tag_external_code_6220","Nichtleistungsgehälter"
"chart_at_template_6225","Additional allowances - salaried employees","6225","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6I,l10n_at.account_tag_external_code_6225","Zulagen - Angestellte"
"chart_at_template_6230","Bonuses and commissions - salaried employees","6230","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6I,l10n_at.account_tag_external_code_6230","Prämien und Provisionen - Angestellte"
"chart_at_template_6240","Special payments - salaried employees","6240","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6I,l10n_at.account_tag_external_code_6240","Sonderzahlungen - Angestellte"
"chart_at_template_6242","Vacation payments - salaried employees","6242","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6I,l10n_at.account_tag_external_code_6290","Urlaubsabfindung - Angestellte"
"chart_at_template_6255","Anniversary payments - salaried employees","6255","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6I,l10n_at.account_tag_external_code_6250","Jubiläumsaufwendungen - Angestellte"
"chart_at_template_6260","Voluntary travel and meal allowances","6260","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6I,l10n_at.account_tag_external_code_6260","Freiwillige Fahrt- und Verpflegungszuschüsse - Angestellte"
"chart_at_template_6270","Non-cash benefits - salaried employees","6270","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6I,l10n_at.account_tag_external_code_6270","Sachbezug - Angestellte"
"chart_at_template_6271","Non-cash benefits - managing directors","6271","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6I,l10n_at.account_tag_external_code_7585","Sachbezug - Geschäftsführer"
"chart_at_template_6310","Overtime - salaried employees","6310","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6I,l10n_at.account_tag_external_code_6210","Überstunden - Angestellte"
"chart_at_template_6340","Changes in provisions for vacations","6340","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6I,l10n_at.account_tag_external_code_6421","Veränderung Urlaubsrückstellung - Angestellte"
"chart_at_template_6400","Employees' occupational pension fund","6400","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6II,l10n_at.account_tag_external_code_6400","Mitarbeitervorsorgekasse - Angestellte"
"chart_at_template_6560","Statutory social welfare expenses - salaried employees","6560","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6II,l10n_at.account_tag_external_code_6560","Gesetzlicher Sozialaufwand - Angestellte"
"chart_at_template_6660","Municipal taxes","6660","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6II,l10n_at.account_tag_external_code_6662","Kommunalsteuer (KoSt) - Angestellte"
"chart_at_template_6661","Employer contributions","6661","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6II,l10n_at.account_tag_external_code_6660","Dienstgeberbeitrag zum Familienlastenausgleichsfonds (DB) - Angestellte"
"chart_at_template_6662","Supplement to employer contributions","6662","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6II,l10n_at.account_tag_external_code_6661","Zuschlag zum Dienstnehmerbeitrag (DZ) - Angestellte"
"chart_at_template_6640","Vienna employers' tax","6663","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6II,l10n_at.account_tag_external_code_6663","Dienstgeberabgabe der Gemeinde Wien (U-Bahn Steuer) - Angestellte"
"chart_at_template_6700","Voluntary social welfare expenses","6700","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT6II,l10n_at.account_tag_external_code_6790","Sonstiger freiwilliger Sozialaufwand"
"chart_at_template_6900","Expense items list","6900","False","expense","l10n_at.account_tag_l10n_at_EBIT6I,l10n_at.account_tag_external_code_7990","Aufwandsstellenrechnung"
"chart_at_template_7000","Amortization of capitalised business start-up and expansion expenses","7000","False","expense_depreciation","l10n_at.account_tag_l10n_at_EBIT7I,l10n_at.account_tag_external_code_7000","Abschreibungen auf aktivierte Aufwendungen für das Ingangsetzen und Erweitern eines Betriebes"
"chart_at_template_7090","Write-downs of capitalised business start-up and expansion expenses","7090","False","expense_depreciation","l10n_at.account_tag_l10n_at_EBIT7II,l10n_at.account_tag_external_code_7090","Abschreibungen vom Umlaufvermögen, soweit diese die im Unternehmen üblichen Abschreibungen übersteigen"
"chart_at_template_7600","Office supplies and printed forms","7600","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7600","Büromaterial und Drucksorten"
"chart_at_template_763","Specialist literature and newspapers","7630","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7630","Fachliteratur und Zeitungen"
"chart_at_template_7690","Donations and tips","7690","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7690","Spenden und Trinkgelder"
"chart_at_template_7770","Vocational training and continuing professional development","7770","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7770","Aus- und Fortbildung"
"chart_at_template_7780","Membership contributions","7780","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7780","Mitgliedsbeiträge"
"chart_at_template_7790","Money transfer charges","7790","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7790","Spesen des Geldverkehrs"
"chart_at_template_7820","Carrying amount of disposed assets, excluding financial assets","7820","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7820","Buchwert abgegangener Anlagen, ausgenommen Finanzanlagen"
"chart_at_template_7830","Loss on disposal of fixed assets, excluding financial assets (Carrying amount of sold assets (-))","7830","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7830","Verlust aus dem Abgang von Anlagevermögen, ausgenommen Finanzanlagen (Buchwert verkaufter Anlagen (-))"
"chart_at_template_7860","Exchange rate loss from foreign currency transactions","7860","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7860","Kursverluste aus Fremdwährungstransaktionen"
"chart_at_template_7890","Cash discount income on other operating expenses","7890","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7890","Skontoerträge auf sonstige betriebliche Aufwendungen"
"chart_at_template_7900","Expense items list","7900","False","expense","l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7900","Aufwandsstellenrechnung"
"chart_at_template_7960","Cost of sales","7960","False","expense","l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7960","Herstellungskosten der zur Erzielung der Umsatzerlöse erbrachten Leistungen"
"chart_at_template_7970","Selling expenses","7970","False","expense","l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7970","Vertriebskosten"
"chart_at_template_7980","Administrative expenses","7980","False","expense","l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7980","Verwaltungskosten"
"chart_at_template_7990","Other operating expenses","7990","False","expense","account.account_tag_operating,l10n_at.account_tag_l10n_at_EBIT8,l10n_at.account_tag_external_code_7990","Sonstige betriebliche Aufwendungen"
"chart_at_template_8140","Revenue from disposal of long-term equity investments (-)","8140","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN10,l10n_at.account_tag_external_code_8140","Erlöse aus dem Abgang von Beteiligungen (-)"
"chart_at_template_8150","Revenue from disposal of other long-term financial assets (-)","8150","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN10,l10n_at.account_tag_external_code_8150","Erlöse aus dem Abgang von sonstigen Finanzanlagen (-)"
"chart_at_template_8160","Revenue from disposal of long-term securities (-)","8160","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN10,l10n_at.account_tag_external_code_8160","Erlöse aus dem Abgang von Wertpapieren des Umlaufvermögens (-)"
"chart_at_template_8170","Carrying amount of disposed long-term equity investments (+)","8170","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN10,l10n_at.account_tag_external_code_8170","Buchwert abgegangener Beteiligungen (+)"
"chart_at_template_8171","Carrying amount of disposed long-term equity investments (-)","8171","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN10,l10n_at.account_tag_external_code_8171","Buchwert abgegangener Beteiligungen (-)"
"chart_at_template_8180","Carrying amount of disposed other long-term financial assets (+)","8180","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN11,l10n_at.account_tag_external_code_8180","Buchwert abgegangener sonstiger Finanzanlagen (+)"
"chart_at_template_8181","Carrying amount of disposed other long-term financial assets (-)","8181","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN11,l10n_at.account_tag_external_code_8181","Buchwert abgegangener sonstiger Finanzanlagen (-)"
"chart_at_template_8190","Carrying amount of disposed long-term securities (+)","8190","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN11,l10n_at.account_tag_external_code_8190","Buchwert abgegangener Wertpapiere des Umlaufvermögens (+)"
"chart_at_template_8191","Carrying amount of disposed long-term securities (-)","8191","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN11,l10n_at.account_tag_external_code_8191","Buchwert abgegangener Wertpapiere des Umlaufvermögens (-)"
"chart_at_template_8200","Revenue from disposal of long-term equity investments (+)","8200","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN10,l10n_at.account_tag_external_code_8200","Erlöse aus dem Abgang von Beteiligungen (+)"
"chart_at_template_8205","Revenue from disposal of other long-term financial assets (+)","8205","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN10,l10n_at.account_tag_external_code_8205","Erlöse aus dem Abgang von sonstigen Finanzanlagen (+)"
"chart_at_template_8206","Revenue from appreciation of other long-term financial assets","8206","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN10,l10n_at.account_tag_external_code_8206","Erlöse aus Zuschreibung sonstige Finanzanlagen"
"chart_at_template_8210","Revenue from disposal of long-term securities (+)","8210","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN11,l10n_at.account_tag_external_code_8210","Erlöse aus dem Abgang von Wertpapieren des Umlaufvermögens (+)"
"chart_at_template_8211","Appreciation of other long-term securities","8211","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN11,l10n_at.account_tag_external_code_8211","Zuschreibung Wertpapiere des Umlaufvermögens"
"chart_at_template_8350","Unused supplier cash discount","8350","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_FIN12,l10n_at.account_tag_external_code_8350","Nicht ausgenützte Lieferantenskonti"
"chart_at_template_8900","Profit transfer from profit and loss transfer","8990","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_RL,l10n_at.account_tag_external_code_8030","Gewinnüberrechnung Ergebnisabführung"
"chart_at_template_8901","Loss transfer from profit and loss transfer","8901","False","income_other","account.account_tag_financing,l10n_at.account_tag_l10n_at_RL,l10n_at.account_tag_external_code_8230","Verlustübernahme Ergebnisabführung"
"chart_at_template_9190","Unpaid uncalled contributions","9190","False","equity","account.account_tag_financing,l10n_at.account_tag_l10n_at_PAI,l10n_at.account_tag_external_code_9190","Nicht eingeforderte ausstehende Einlagen"
"chart_at_template_9260","Treasury shares","9260","False","equity","l10n_at.account_tag_l10n_at_PAI,l10n_at.account_tag_external_code_2600","Eigene Anteile"
"chart_at_template_9390","Balance sheet profit (loss)","9390","False","equity","account.account_tag_financing,l10n_at.account_tag_l10n_at_PAIV,l10n_at.account_tag_external_code_9390","Bilanzgewinn (-verlust)"
"chart_at_template_9800","Opening balance","9800","False","equity","l10n_at.account_tag_external_code_9800","Eröffnungsbilanz"
"chart_at_template_9850","Closing balance","9850","False","equity","l10n_at.account_tag_external_code_9850","Schlussbilanz"
"chart_at_template_9890","Profit and loss statement","9890","False","equity_unaffected","l10n_at.account_tag_l10n_at_RL,l10n_at.account_tag_external_code_9350","Gewinn- und Verlustrechnung"

```

## File: data\template\account.fiscal.position-at.csv

```csv
"id","name","auto_apply","country_id","vat_required","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id"
"fiscal_position_template_national","National + EU (ohne UID)","1","","0","base.europe","","","",""
"fiscal_position_template_national_w_uid","National","1","base.at","1","","","","",""
"fiscal_position_template_eu","Europäische Union","1","","1","base.europe","account_tax_template_sales_20_code022","account_tax_template_sales_eu_0_code017","",""
"","","","","","","account_tax_template_sales_20_katalog022","account_tax_template_sales_eu_0_services","",""
"","","","","","","account_tax_template_sales_10_code029","account_tax_template_sales_eu_0_code017","",""
"","","","","","","account_tax_template_sales_add7_code007","account_tax_template_sales_eu_0_code017","",""
"","","","","","","account_tax_template_purchase_20_code060","account_tax_template_purchase_eu_20","",""
"","","","","","","account_tax_template_purchase_20_misc_code060","account_tax_template_purchase_rev_charge_19_2_25_5","",""
"","","","","","","account_tax_template_purchase_10_code060","account_tax_template_purchase_eu_10","",""
"","","","","","","account_tax_template_purchase_19_code060","account_tax_template_purchase_eu_19","",""
"","","","","","","","","chart_at_template_4000","chart_at_template_4100"
"","","","","","","","","chart_at_template_4001","chart_at_template_4110"
"","","","","","","","","chart_at_template_2000","chart_at_template_2100"
"","","","","","","","","chart_at_template_5010","chart_at_template_5050"
"","","","","","","","","chart_at_template_5011","chart_at_template_5051"
"fiscal_position_template_non_eu","Drittstaaten","1","","","","account_tax_template_sales_20_code022","account_tax_template_sales_non_eu_0_code011","",""
"","","","","","","account_tax_template_sales_10_code029","account_tax_template_sales_non_eu_0_services","",""
"","","","","","","account_tax_template_sales_add7_code007","account_tax_template_sales_non_eu_0_code011","",""
"","","","","","","","","chart_at_template_4000","chart_at_template_4200"
"","","","","","","","","chart_at_template_2000","chart_at_template_2150"
"","","","","","","","","chart_at_template_5010","chart_at_template_5090"
"","","","","","","","","chart_at_template_5011","chart_at_template_5090"

```

## File: data\template\account.tax-at.csv

```csv
"id","name","description","invoice_label","sequence","type_tax_use","tax_scope","amount","amount_type","active","tax_group_id","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","repartition_line_ids/factor_percent","description@de"
"account_tax_template_sales_rev_charge_0_code021","0% Ust L","UST_021 Section 19 (1) second sentence (Tax liability concerns service recipient)","0%","400","sale","","0.0","percent","","tax_group_0","base","invoice","+KZ 000||+KZ 021","","","UST_021 § 19 Abs. 1 zweiter Satz (Steuerschuld betrifft Leistungsempfänger)"
"","","","","","","","","","","","tax","invoice","+KZ 021","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 000||-KZ 021","","",""
"","","","","","","","","","","","tax","refund","-KZ 021","chart_at_template_3505","",""
"account_tax_template_sales_rev_charge_0_code021_1a","0% Ust L 1a","UST_021 Section 19 (1a) (Tax liability concerns service recipient)","0%","400","sale","","0.0","percent","","tax_group_0","base","invoice","+KZ 000||+KZ 021","","","UST_021 § 19 Abs. 1a (Steuerschuld betrifft Leistungsempfänger)"
"","","","","","","","","","","","tax","invoice","+KZ 021","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 000||-KZ 021","","",""
"","","","","","","","","","","","tax","refund","-KZ 021","chart_at_template_3505","",""
"account_tax_template_sales_rev_charge_0_code021_1b","0% Ust L 1b","UST_021 Section 19 (1b) (Tax liability concerns service recipient)","0%","400","sale","","0.0","percent","","tax_group_0","base","invoice","+KZ 000||+KZ 021","","","UST_021 § 19 Abs. 1b (Steuerschuld betrifft Leistungsempfänger)"
"","","","","","","","","","","","tax","invoice","+KZ 021","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 000||-KZ 021","","",""
"","","","","","","","","","","","tax","refund","-KZ 021","chart_at_template_3505","",""
"account_tax_template_sales_rev_charge_0_code021_1c","0% Ust L 1c","UST_021 Section 19 (1c) (Tax liability concerns service recipient)","0%","400","sale","","0.0","percent","","tax_group_0","base","invoice","+KZ 000||+KZ 021","","","UST_021 § 19 Abs. 1c (Steuerschuld betrifft Leistungsempfänger)"
"","","","","","","","","","","","tax","invoice","+KZ 021","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 000||-KZ 021","","",""
"","","","","","","","","","","","tax","refund","-KZ 021","chart_at_template_3505","",""
"account_tax_template_sales_rev_charge_0_code021_1d","0% Ust L 1d","UST_021 Section 19 (1d) (Tax liability concerns service recipient)","0%","400","sale","","0.0","percent","","tax_group_0","base","invoice","+KZ 000||+KZ 021","","","UST_021 § 19 Abs. 1d (Steuerschuld betrifft Leistungsempfänger)"
"","","","","","","","","","","","tax","invoice","+KZ 021","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 000||-KZ 021","","",""
"","","","","","","","","","","","tax","refund","-KZ 021","chart_at_template_3505","",""
"account_tax_template_sales_rev_charge_0_code021_1e","0% Ust L 1e","UST_021 Section 19 (1e) (Tax liability concerns service recipient)","0%","400","sale","","0.0","percent","False","","base","invoice","+KZ 000||+KZ 021","","","UST_021 § 19 Abs. 1e (Steuerschuld betrifft Leistungsempfänger)"
"","","","","","","","","","","","tax","invoice","+KZ 021","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 000||-KZ 021","","",""
"","","","","","","","","","","","tax","refund","-KZ 021","chart_at_template_3505","",""
"account_tax_template_sales_non_eu_0_code011","0% USt EX","UST_011 Export 0%","0%","300","sale","consu","0.0","percent","","tax_group_0","base","invoice","+KZ 011||+KZ 000","","","UST_011 Export 0%"
"","","","","","","","","","","","tax","invoice","","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 011||-KZ 000","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_3505","",""
"account_tax_template_sales_non_eu_0_code012","0% Ust Sub","UST_012 Subcontracting 0%","0%","200","sale","","0.0","percent","","tax_group_0","base","invoice","+KZ 012||+KZ 000","","","UST_012 Lohnveredelung 0%"
"","","","","","","","","","","","tax","invoice","","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 012||-KZ 000","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_3505","",""
"account_tax_template_sales_non_eu_0_code015","0% Ust EX art6","UST_015 Export 0% (§ 6 Abs. 1 Z 2 bis 6)","0%","300","sale","","0.0","percent","","tax_group_0","base","invoice","+KZ 015||+KZ 000","","","UST_015 Export 0% (§ 6 Abs. 1 Z 2 bis 6)"
"","","","","","","","","","","","tax","invoice","","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 015||-KZ 000","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_3505","",""
"account_tax_template_sales_eu_0_code017","0% Ust IGL Nart6","UST_017 IGL 0% (without art. 6 par. 1)","0%","200","sale","consu","0.0","percent","","tax_group_0","base","invoice","+KZ 017||+KZ 000||+AT_ZM_IGL","","","UST_017 IGL 0% (ohne Art. 6 Abs. 1)"
"","","","","","","","","","","","tax","invoice","","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 017||-KZ 000||-AT_ZM_IGL","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_3505","",""
"account_tax_template_sales_eu_0_code018","0% Ust IGL art6","UST_018 IGL 0% (Art. 6 Abs. 1)","0%","200","sale","consu","0.0","percent","","tax_group_0","base","invoice","+KZ 018||+KZ 000||+AT_ZM_IGL","","","UST_018 IGL 0% (Art. 6 Abs. 1)"
"","","","","","","","","","","","tax","invoice","","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 018||-KZ 000||-AT_ZM_IGL","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_3505","",""
"account_tax_template_sales_0_code019","0% Ust R E","UST_019 Real estate sales 0% (§ 6 Abs. 1 Z 9 lit. a)","0%","100","sale","","0.0","percent","","tax_group_0","base","invoice","+KZ 019||+KZ 000","","","UST_019 Grundstücksumsätze 0% (§ 6 Abs. 1 Z 9 lit. a)"
"","","","","","","","","","","","tax","invoice","","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 019||-KZ 000","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_3505","",""
"account_tax_template_sales_0_code016","0% Ust S B","UST_016 Small business 0% (§ 6 Abs. 1 Z 27)","0%","100","sale","","0.0","percent","","tax_group_0","base","invoice","+KZ 016||+KZ 000","","","UST_016 Kleinunternehmer 0% (§ 6 Abs. 1 Z 27)"
"","","","","","","","","","","","tax","invoice","","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 016||-KZ 000","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_3505","",""
"account_tax_template_sales_0_code020","0% Ust O Exempt","UST_020 Other tax-exempt sales 0%","0%","100","sale","","0.0","percent","","tax_group_0","base","invoice","+KZ 020||+KZ 000","","","UST_020 Übrige steuerfreie Umsätze 0%"
"","","","","","","","","","","","tax","invoice","","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 020||-KZ 000","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_3505","",""
"account_tax_template_sales_20_code022","20% Ust","UST_022 Normal tax rate 20%","20%","50","sale","consu","20.0","percent","","tax_group_20","base","invoice","+KZ 000||+KZ 022 Bemessungsgrundlage","","","UST_022 Normalsteuersatz 20%"
"","","","","","","","","","","","tax","invoice","-KZ 022 Umsatzsteuer","chart_at_template_3500","",""
"","","","","","","","","","","","base","refund","-KZ 000||-KZ 022 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","+KZ 022 Umsatzsteuer","chart_at_template_3500","",""
"account_tax_template_sales_20_katalog022","20% Ust O S","UST_022 Normal tax rate 20% (Other services)","20%","100","sale","service","20.0","percent","","tax_group_20","base","invoice","+KZ 000||+KZ 022 Bemessungsgrundlage","","","UST_022 Normalsteuersatz 20% (Sonstige Leistungen)"
"","","","","","","","","","","","tax","invoice","-KZ 022 Umsatzsteuer","chart_at_template_3500","",""
"","","","","","","","","","","","base","refund","-KZ 000||-KZ 022 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","+KZ 022 Umsatzsteuer","chart_at_template_3500","",""
"account_tax_template_sales_10_code029","10% Ust R","UST_029 reduced tax rate 10%","10%","100","sale","consu","10.0","percent","","tax_group_10","base","invoice","+KZ 000||+KZ 029 Bemessungsgrundlage","","","UST_029 ermäßigter Steuersatz 10%"
"","","","","","","","","","","","tax","invoice","-KZ 029 Umsatzsteuer","chart_at_template_3501","",""
"","","","","","","","","","","","base","refund","-KZ 000||-KZ 029 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","+KZ 029 Umsatzsteuer","chart_at_template_3501","",""
"account_tax_template_sales_13_code006","13% Ust R","UST_006 reduced tax rate 13%","13%","100","sale","","13.0","percent","","tax_group_13","base","invoice","+KZ 000||+KZ 006 Bemessungsgrundlage","","","UST_006 ermäßigter Steuersatz 13%"
"","","","","","","","","","","","tax","invoice","-KZ 006 Umsatzsteuer","chart_at_template_3502","",""
"","","","","","","","","","","","base","refund","-KZ 000||-KZ 006 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","+KZ 006 Umsatzsteuer","chart_at_template_3502","",""
"account_tax_template_sales_19_code037","19% Ust","UST_037 Tax rate 19%","19%","100","sale","","19.0","percent","","tax_group_19","base","invoice","+KZ 000||+KZ 037 Bemessungsgrundlage","","","UST_037 Steuersatz 19%"
"","","","","","","","","","","","tax","invoice","-KZ 037 Umsatzsteuer","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 000||-KZ 037 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","+KZ 037 Umsatzsteuer","chart_at_template_3505","",""
"account_tax_template_sales_add10_code052","10% Ust Add","UST_052 Additional tax rate 10% (LWB/FWB)","10%","100","sale","","10.0","percent","","tax_group_10","base","invoice","+KZ 000||+KZ 052 Bemessungsgrundlage","","","UST_052 Zusatzsteuersatz 10% (LWB/FWB)"
"","","","","","","","","","","","tax","invoice","-KZ 052 Umsatzsteuer","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 000||-KZ 052 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","+KZ 052 Umsatzsteuer","chart_at_template_3505","",""
"account_tax_template_sales_add7_code007","7% Ust Add","UST_007 Additional tax rate 7% (LWB/FWB)","7%","100","sale","","7.0","percent","False","","base","invoice","+KZ 000||+KZ 007 Bemessungsgrundlage","","","UST_007 Zusatzsteuersatz 7% (LWB/FWB)"
"","","","","","","","","","","","tax","invoice","-KZ 007 Umsatzsteuer","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 000||-KZ 007 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","+KZ 007 Umsatzsteuer","chart_at_template_3505","",""
"account_tax_template_sales_self_20_code022","20% Ust C","UST_022 Normal tax rate 20% (own consumption)","20%","100","sale","","20.0","percent","","tax_group_20","base","invoice","+KZ 001||+KZ 022 Bemessungsgrundlage","","","UST_022 Normalsteuersatz 20% (Eigenverbrauch)"
"","","","","","","","","","","","tax","invoice","-KZ 022 Umsatzsteuer","chart_at_template_3500","",""
"","","","","","","","","","","","base","refund","-KZ 001||-KZ 022 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","+KZ 022 Umsatzsteuer","chart_at_template_3500","",""
"account_tax_template_sales_self_10_code029","10% Ust R O C","UST_029 reduced tax rate 10% (own consumption)","10%","100","sale","","10.0","percent","","tax_group_10","base","invoice","+KZ 001||+KZ 029 Bemessungsgrundlage","","","UST_029 ermäßigter Steuersatz 10% (Eigenverbrauch)"
"","","","","","","","","","","","tax","invoice","-KZ 029 Umsatzsteuer","chart_at_template_3501","",""
"","","","","","","","","","","","base","refund","-KZ 001||-KZ 029 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","+KZ 029 Umsatzsteuer","chart_at_template_3501","",""
"account_tax_template_sales_self_19_code037","19% Ust O C","UST_037 Tax rate 19% (own consumption)","19%","100","sale","","19.0","percent","","tax_group_19","base","invoice","+KZ 001||+KZ 037 Bemessungsgrundlage","","","UST_037 Steuersatz 19% (Eigenverbrauch)"
"","","","","","","","","","","","tax","invoice","-KZ 037 Umsatzsteuer","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 001||-KZ 037 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","+KZ 037 Umsatzsteuer","chart_at_template_3505","",""
"account_tax_template_sales_self_add10_code052","10% Ust Add C","UST_052 Additional tax rate 10% (LWB/FWB - own consumption)","10%","100","sale","","10.0","percent","","tax_group_10","base","invoice","+KZ 001||+KZ 052 Bemessungsgrundlage","","","UST_052 Zusatzsteuersatz 10% (LWB/FWB - Eigenverbrauch)"
"","","","","","","","","","","","tax","invoice","-KZ 052 Umsatzsteuer","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 001||-KZ 052 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","+KZ 052 Umsatzsteuer","chart_at_template_3505","",""
"account_tax_template_sales_self_add7_code007","7% Ust Add O C","UST_007 Additional tax rate 7% (LWB/FWB - own consumption)","7%","100","sale","","7.0","percent","","","base","invoice","+KZ 001||+KZ 007 Bemessungsgrundlage","","","UST_007 Zusatzsteuersatz 7% (LWB/FWB - Eigenverbrauch)"
"","","","","","","","","","","","tax","invoice","-KZ 007 Umsatzsteuer","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 001||-KZ 007 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","+KZ 007 Umsatzsteuer","chart_at_template_3505","",""
"account_tax_template_purchase_tax_invoiced_accepted_code056","20% Ust T I","UST_056 Tax invoiced accepted (§ 11 par. 12 and 14 § 16 par. 2 and according to Art. 7 par. 4)","20%","100","purchase","","20.0","percent","","tax_group_20","base","invoice","","","","UST_056 Tax invoiced accepted (§ 11 Abs. 12 und 14, § 16 Abs. 2 sowie gemäß Art. 7 Abs. 4)"
"","","","","","","","","","","","tax","invoice","+KZ 056","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 056","chart_at_template_3505","",""
"account_tax_template_sales_eu_0_services","0% Ust EU S","UST_EU Service (other services) 0%","0%","200","sale","service","0.0","percent","","tax_group_0","base","invoice","+AT_ZM_DL","","","UST_EU Dienstleistung (Sonstige Leistungen) 0%"
"","","","","","","","","","","","tax","invoice","","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-AT_ZM_DL","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_3505","",""
"account_tax_template_sales_non_eu_0_services","0% Ust EX S","UST_NON_EU Service (third countries) 0%","0%","300","sale","service","0.0","percent","","tax_group_0","base","invoice","","","","UST_NON_EU Dienstleistung (Drittstaaten) 0%"
"","","","","","","","","","","","tax","invoice","","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_3505","",""
"account_tax_template_purchase_eu_0_code071","0% Ust","UST_071 IGE 0% (Art. 6 para. 2)","0%","200","purchase","","0.0","percent","","tax_group_0","base","invoice","-KZ 070||-KZ 071","","","UST_071 IGE 0% (Art. 6 Abs. 2)"
"","","","","","","","","","","","tax","invoice","","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","+KZ 070||+KZ 071","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_3505","",""
"account_tax_template_purchase_eu_20","20%","IGE 20%","IGE 20%","500","purchase","","20.0","percent","","tax_group_0","base","invoice","-KZ 070||-KZ 072 Bemessungsgrundlage","","","IGE 20%"
"","","","","","","","","","","","tax","invoice","+KZ 072 Umsatzsteuer","chart_at_template_3511","-100",""
"","","","","","","","","","","","tax","invoice","+KZ 065","chart_at_template_2511","",""
"","","","","","","","","","","","base","refund","+KZ 070||+KZ 072 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","-KZ 072 Umsatzsteuer","chart_at_template_3511","-100",""
"","","","","","","","","","","","tax","refund","-KZ 065","chart_at_template_2511","",""
"account_tax_template_purchase_eu_10","10%","IGE 10%","IGE 10%","500","purchase","","10.0","percent","","tax_group_0","base","invoice","-KZ 070||-KZ 073 Bemessungsgrundlage","","","IGE 10%"
"","","","","","","","","","","","tax","invoice","+KZ 073 Umsatzsteuer","chart_at_template_3512","-100",""
"","","","","","","","","","","","tax","invoice","+KZ 065","chart_at_template_2512","",""
"","","","","","","","","","","","base","refund","+KZ 070||+KZ 073 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","-KZ 073 Umsatzsteuer","chart_at_template_3512","-100",""
"","","","","","","","","","","","tax","refund","-KZ 065","chart_at_template_2512","",""
"account_tax_template_purchase_eu_13","13%","IGE 13%","IGE 13%","500","purchase","","13.0","percent","","tax_group_0","base","invoice","-KZ 070||-KZ 008 Bemessungsgrundlage","","","IGE 13%"
"","","","","","","","","","","","tax","invoice","+KZ 008 Umsatzsteuer","chart_at_template_3513","-100",""
"","","","","","","","","","","","tax","invoice","+KZ 065","chart_at_template_2513","",""
"","","","","","","","","","","","base","refund","+KZ 070||+KZ 008 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","-KZ 008 Umsatzsteuer","chart_at_template_3513","-100",""
"","","","","","","","","","","","tax","refund","-KZ 065","chart_at_template_2513","",""
"account_tax_template_purchase_eu_19","19%","IGE 19%","IGE 19%","500","purchase","","19.0","percent","False","tax_group_0","base","invoice","-KZ 070||-KZ 088 Bemessungsgrundlage","","","IGE 19%"
"","","","","","","","","","","","tax","invoice","+KZ 088 Umsatzsteuer","chart_at_template_3511","-100",""
"","","","","","","","","","","","tax","invoice","+KZ 065","","",""
"","","","","","","","","","","","base","refund","+KZ 070||+KZ 088 Bemessungsgrundlage","","",""
"","","","","","","","","","","","tax","refund","-KZ 088 Umsatzsteuer","chart_at_template_3511","-100",""
"","","","","","","","","","","","tax","refund","-KZ 065","","",""
"account_tax_template_purchase_rev_charge_1a","20% RC C S","Reverse charge 20% (§ 19 para 1a - construction services)","RC 20% § 19 Abs. 1a","550","purchase","","20.0","percent","","tax_group_0","base","invoice","","","","Reverse Charge 20% (§ 19 Abs. 1a - Bauleistungen)"
"","","","","","","","","","","","tax","invoice","+KZ 048","chart_at_template_3510","-100",""
"","","","","","","","","","","","tax","invoice","+KZ 082","chart_at_template_2510","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 048","chart_at_template_3510","-100",""
"","","","","","","","","","","","tax","refund","-KZ 082","chart_at_template_2510","",""
"account_tax_template_purchase_rev_charge_1b","20% RC Secu","Reverse charge 20% (§ 19 para. 1b - security ownership, reserved property and properties in compulsory auction proceedings)","RC 20% § 19 Abs. 1b","550","purchase","","20.0","percent","","tax_group_0","base","invoice","","","","Reverse Charge 20% (§ 19 Abs. 1b - Sicherungseigentum, Vorbehaltseigentum und Grundstücke im Zwangsversteigerungsverfahren)"
"","","","","","","","","","","","tax","invoice","+KZ 044","chart_at_template_3510","-100",""
"","","","","","","","","","","","tax","invoice","+KZ 087","chart_at_template_2510","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 044","chart_at_template_3510","-100",""
"","","","","","","","","","","","tax","refund","-KZ 087","chart_at_template_2510","",""
"account_tax_template_purchase_rev_charge_19_2_25_5","20% RC O S T T","Reverse Charge 20% (§ 19 Abs. 1 second sentence - other services, Art. 25 Abs. 5 - triangular trade","RC 20% §19 Sonstige Leistungen, Dreiecksgeschäft","550","purchase","","20.0","percent","","tax_group_0","base","invoice","","","","Reverse Charge 20% (§ 19 Abs. 1 zweiter Satz - Sonstige Leistungen, Art. 25 Abs. 5 - Dreiecksgeschäft"
"","","","","","","","","","","","tax","invoice","+KZ 057","chart_at_template_3510","-100",""
"","","","","","","","","","","","tax","invoice","+KZ 066","chart_at_template_2510","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 057","chart_at_template_3510","-100",""
"","","","","","","","","","","","tax","refund","-KZ 066","chart_at_template_2510","",""
"account_tax_template_purchase_rev_charge_1c","20% RC 19(1c)","Reverse charge 20% (§ 19 para. 1c - gas, electricity, heat, cold)","RC 20% § 19 Abs. 1c","550","purchase","","20.0","percent","","tax_group_0","base","invoice","","","","Reverse Charge 20% (§ 19 Abs. 1c - Gas, Strom, Wärme, Kälte)"
"","","","","","","","","","","","tax","invoice","+KZ 057","chart_at_template_3510","-100",""
"","","","","","","","","","","","tax","invoice","+KZ 066","chart_at_template_2510","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 057","chart_at_template_3510","-100",""
"","","","","","","","","","","","tax","refund","-KZ 066","chart_at_template_2510","",""
"account_tax_template_purchase_rev_charge_1d","20% RC 19(1d)","Reverse charge 20% (§ 19 par. 1d - scrap and waste materials, game consoles, laptops, tablet computers >= EUR 5,000, gas and electricity, gas and electricity certificates, metals, investment gold)","RC 20% § 19 Abs. 1d","550","purchase","","20.0","percent","","tax_group_0","base","invoice","","","","Reverse Charge 20% (§ 19 Abs. 1d - Schrott und Abfallstoffe, Spielekonsolen, Laptops, Tablet-Computer >= EUR 5.000,-, Gas und Elektrizität, Gas- und Elektrizitätszertifikate, Metalle, Anlagegold)"
"","","","","","","","","","","","tax","invoice","+KZ 032","chart_at_template_3510","-100",""
"","","","","","","","","","","","tax","invoice","+KZ 089","chart_at_template_2510","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 032","chart_at_template_3510","-100",""
"","","","","","","","","","","","tax","refund","-KZ 089","chart_at_template_2510","",""
"account_tax_template_purchase_rev_charge_1e","20% RC 19(1e)","Reverse Charge 20% (§ 19 Abs. 1e - Greenhouse gas emission certificates, mobile devices >= EUR 5.000,-)","RC 20% § 19 Abs. 1e","550","purchase","","20.0","percent","","tax_group_0","base","invoice","","","","Reverse Charge 20% (§ 19 Abs. 1e - Treibhausgasemissionszertifikaten, Mobilfunkgeräte >= EUR 5.000,-)"
"","","","","","","","","","","","tax","invoice","+KZ 032","chart_at_template_3510","-100",""
"","","","","","","","","","","","tax","invoice","+KZ 089","chart_at_template_2510","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 032","chart_at_template_3510","-100",""
"","","","","","","","","","","","tax","refund","-KZ 089","chart_at_template_2510","",""
"account_tax_template_purchase_eu_xx_code076","0% EU A","Acquisitions pursuant to Art. 3(8), second sentence, which have been taxed in the Member State of destination (IGE-UST)","UST_076 IGE (im Bestimmungsland besteuert)","200","purchase","","0.0","percent","","tax_group_0","base","invoice","-KZ 076||-KZ 077","","","Erwerbe gemäß Art. 3 Abs. 8 zweiter Satz, die im Mitgliedstaat des Bestimmungslandes besteuert worden sind (IGE-UST)"
"","","","","","","","","","","","tax","invoice","","chart_at_template_2505","",""
"","","","","","","","","","","","base","refund","+KZ 076||+KZ 077","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_2505","",""
"account_tax_template_purchase_eu_xx_code077","0% A","Acquisitions under the second sentence of Art. 3(8) that are deemed to be taxed domestically under Art. 25(2) (IGE-UST)","UST_077 IGE (im Inland besteuert)","200","purchase","","0.0","percent","False","","base","invoice","+KZ 077||+KZ 037 Bemessungsgrundlage","","","Erwerbe gemäß Art. 3 Abs. 8 zweiter Satz, die gemäß Art. 25 Abs. 2 im Inland als besteuert gelten (IGE-UST)"
"","","","","","","","","","","","tax","invoice","-KZ 077","chart_at_template_3505","",""
"","","","","","","","","","","","base","refund","-KZ 077||-KZ 037 Bemessungsgrundlage","","100.0",""
"","","","","","","","","","","","tax","refund","+KZ 077","chart_at_template_3505","100.0",""
"account_tax_template_purchase_20_code060","20% Vst","VST_060 Normal tax rate 20%","20%","50","purchase","","20.0","percent","","tax_group_20","base","invoice","","","","VST_060 Normalsteuersatz 20%"
"","","","","","","","","","","","tax","invoice","+KZ 060","chart_at_template_2500","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 060","chart_at_template_2500","",""
"account_tax_template_purchase_20_misc_code060","20% Vst O S","VST_060 other services 20%","20%","400","purchase","","20.0","percent","","tax_group_20","base","invoice","","","","VST_060 sonstige Leistungen 20%"
"","","","","","","","","","","","tax","invoice","+KZ 060","chart_at_template_2500","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 060","chart_at_template_2500","",""
"account_tax_template_purchase_10_code060","10% Vst","VST_060 reduced tax rate 10%","10%","400","purchase","","10.0","percent","","tax_group_10","base","invoice","","","","VST_060 ermäßigter Steuersatz 10%"
"","","","","","","","","","","","tax","invoice","+KZ 060","chart_at_template_2501","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 060","chart_at_template_2501","",""
"account_tax_template_purchase_13_code060","13% Vst","VST_060 reduced tax rate 13%","13%","400","purchase","","13.0","percent","","tax_group_13","base","invoice","","","","VST_060 ermäßigter Steuersatz 13%"
"","","","","","","","","","","","tax","invoice","+KZ 060","chart_at_template_2502","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 060","chart_at_template_2502","",""
"account_tax_template_purchase_19_code060","19% Vst J M","VST_060 Jungholz and Mittelberg 19%","19%","400","purchase","","19.0","percent","","tax_group_19","base","invoice","","","","VST_060 Jungholz und Mittelberg 19%"
"","","","","","","","","","","","tax","invoice","+KZ 060","chart_at_template_2505","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 060","chart_at_template_2505","",""
"account_tax_template_purchase_12_code060","12% Vst W","VST_060 Wine purchase 12% (LWB)","12%","400","purchase","","12.0","percent","","tax_group_12","base","invoice","","","","VST_060 Weineinkauf 12% (LWB)"
"","","","","","","","","","","","tax","invoice","+KZ 060","chart_at_template_2505","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 060","chart_at_template_2505","",""
"account_tax_template_purchase_xx_code061","20% Vst EU T","VST_061 EU tax paid (§ 12 par. 1 no. 2 lit. a)","","400","purchase","","","percent","","","","","","","","VST_061 entrichtete EUst (§ 12 Abs. 1 Z 2 lit. a)"
"account_tax_template_purchase_xx_code083","20% Vst P","VST_083 Posted EUst. (§ 12 par. 1 line 2 lit. b)","","400","none","","","percent","","","","","","","","VST_083 verbuchte EUst. (§ 12 Abs. 1 Z 2 lit. b)"
"account_tax_template_purchase_correct_code063","0% Vst C-12","VST_063 (§12 par. 10 and 11 - correction)","0%","400","purchase","","0.0","percent","","tax_group_0","base","invoice","","","","VST_063 (§12 Abs. 10 und 11 - Berichtigung)"
"","","","","","","","","","","","tax","invoice","","chart_at_template_2505","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_2505","",""
"account_tax_template_purchase_correct_code067","0% Vst C-16","VST_067 (§ 16 - correction)","0%","400","purchase","","0.0","percent","","tax_group_0","base","invoice","","","","VST_067 (§ 16 - Berichtigung)"
"","","","","","","","","","","","tax","invoice","+KZ 067","chart_at_template_2505","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","+KZ 067","chart_at_template_2505","",""
"account_tax_template_purchase_correct_code090","0% Vst O C","VST_090 (Other corrections)","0%","600","purchase","","0.0","percent","False","","base","invoice","","","","VST_090 (Sonstige Berichtigungen)"
"","","","","","","","","","","","tax","invoice","+KZ 090","chart_at_template_2505","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 090","chart_at_template_2505","",""
"account_tax_template_purchase_cars_buildings_code027","20% Vst Car","VST_027 concerning motor vehicles according to EKR 063, 064, 732-733 und 744-747","20%","400","purchase","","20.0","percent","","tax_group_20","base","invoice","","","","VST_027 betreffend KFZ nach EKR 063, 064, 732-733 und 744-747"
"","","","","","","","","","","","tax","invoice","","chart_at_template_2505","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_2505","",""
"account_tax_template_purchase_cars_buildings_code028","20% Vst B","VST_028 concerning buildings according to EKR 030-037 and 070","20%","400","purchase","","20.0","percent","","tax_group_20","base","invoice","","","","VST_028 betreffend Gebäude nach EKR 030-037 und 070, 071"
"","","","","","","","","","","","tax","invoice","","chart_at_template_2505","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_2505","",""
"account_tax_template_purchase_eu_0_vst_071","0% Vst","VST_071 IGE 0%","VST_071 IGE 0% (Art. 6 Abs. 2)","500","purchase","","0.0","percent","","tax_group_0","base","invoice","","","","VST_071 IGE 0%"
"","","","","","","","","","","","tax","invoice","+KZ 065","chart_at_template_2505","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","-KZ 065","chart_at_template_2505","",""
"account_tax_template_purchase_eu_xx_vst_076","20% Vst EU","VST_076 IGE (taxed in the country of destination)","20%","500","purchase","","20.0","percent","","tax_group_20","base","invoice","-KZ 076||-KZ 077","","","VST_076 IGE (im Bestimmungsland besteuert)"
"","","","","","","","","","","","tax","invoice","","chart_at_template_2505","",""
"","","","","","","","","","","","base","refund","+KZ 076||+KZ 077","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_2505","",""
"account_tax_template_purchase_eu_xx_vst_077","20% Vst D","VST_077 IGE (taxed domestically)","20%","500","purchase","","20.0","percent","","tax_group_20","base","invoice","","","","VST_077 IGE (im Inland besteuert)"
"","","","","","","","","","","","tax","invoice","","chart_at_template_2505","",""
"","","","","","","","","","","","base","refund","","","",""
"","","","","","","","","","","","tax","refund","","chart_at_template_2505","",""

```

## File: data\template\account.tax.group-at.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id"
"tax_group_0","0%","base.at","chart_at_template_3530","chart_at_template_3530"
"tax_group_10","10%","base.at","chart_at_template_3530","chart_at_template_3530"
"tax_group_12","12%","base.at","chart_at_template_3530","chart_at_template_3530"
"tax_group_13","13%","base.at","chart_at_template_3530","chart_at_template_3530"
"tax_group_19","19%","base.at","chart_at_template_3530","chart_at_template_3530"
"tax_group_20","20%","base.at","chart_at_template_3530","chart_at_template_3530"

```

## File: migrations\3.2\post-migrate.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, SUPERUSER_ID, Command


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})
    # Tag PCVIII is no longer referenced in the Balance Sheet: users should use PCVIII3 instead.
    if (
        (tag_pcviii := env.ref('l10n_at.account_tag_l10n_at_PCVIII', raise_if_not_found=False))
        and (tag_pcviii3 := env.ref('l10n_at.account_tag_l10n_at_PCVIII3', raise_if_not_found=False))
    ):
        env['account.account'].search([('tag_ids', '=', tag_pcviii.id)]).write({
            'tag_ids': [Command.unlink(tag_pcviii.id), Command.link(tag_pcviii3.id)],
        })

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-

from odoo import api, models, Command


class AccountJournal(models.Model):
    _inherit = 'account.journal'

    @api.model
    def _prepare_liquidity_account_vals(self, company, code, vals):
        ''' Set Balance Sheet and SAF-T tags on new bank and cash accounts.'''
        # OVERRIDE
        account_vals = super()._prepare_liquidity_account_vals(company, code, vals)

        if company.account_fiscal_country_id.code == 'AT':
            account_vals.setdefault('tag_ids', [])
            account_vals['tag_ids'] += [
                Command.link(self.env.ref('l10n_at.account_tag_l10n_at_ABIV').id),
                Command.link(self.env.ref('l10n_at.account_tag_external_code_2300').id),
            ]

        return account_vals

```

## File: models\template_at.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('at')
    def _get_at_template_data(self):
        return {
            'visible': True,
            'property_account_receivable_id': 'chart_at_template_2000',
            'property_account_payable_id': 'chart_at_template_3300',
            'property_account_income_categ_id': 'chart_at_template_4000',
            'property_account_expense_categ_id': 'chart_at_template_5010',
            'property_stock_account_input_categ_id': 'chart_at_template_3740',
            'property_stock_account_output_categ_id': 'chart_at_template_5000',
            'property_stock_valuation_account_id': 'chart_at_template_1600',
            'code_digits': '4',
        }

    @template('at', 'res.company')
    def _get_at_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.at',
                'bank_account_code_prefix': '280',
                'cash_account_code_prefix': '270',
                'transfer_account_code_prefix': '288',
                'account_default_pos_receivable_account_id': 'chart_at_template_2099',
                'income_currency_exchange_account_id': 'chart_at_template_4860',
                'expense_currency_exchange_account_id': 'chart_at_template_7860',
                'account_journal_early_pay_discount_loss_account_id': 'chart_at_template_5800',
                'account_journal_early_pay_discount_gain_account_id': 'chart_at_template_8350',
                'external_report_layout_id': 'l10n_din5008.external_layout_din5008',
                'paperformat_id': 'l10n_din5008.paperformat_euro_din',
                'account_sale_tax_id': 'account_tax_template_sales_20_code022',
                'account_purchase_tax_id': 'account_tax_template_purchase_20_code060',
            },
        }

    def _setup_utility_bank_accounts(self, template_code, company, template_data):
        super()._setup_utility_bank_accounts(template_code, company, template_data)
        if template_code == "at":
            bank_tags = self.env.ref('l10n_at.account_tag_external_code_2300') | self.env.ref('l10n_at.account_tag_l10n_at_ABIV')
            company.account_journal_suspense_account_id.tag_ids = bank_tags
            company.transfer_account_id.tag_ids = self.env.ref('l10n_at.account_tag_external_code_2885') | self.env.ref('l10n_at.account_tag_l10n_at_ABIV')

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import account_journal
from . import template_at

```

