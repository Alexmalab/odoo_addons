# Odoo Module: l10n_ro

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# @author -  Fekete Mihai <feketemihai@gmail.com>, Tatár Attila <atta@nvm.ro>
# Copyright (C) 2015 Tatár Attila
# Copyright (C) 2015 Forest and Biomass Services Romania (http://www.forbiom.eu).
# Copyright (C) 2011 TOTAL PC SYSTEMS (http://www.erpsystems.ro).
# Copyright (C) 2009 (<http://www.filsystem.ro>)

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# @author -  Fekete Mihai <feketemihai@gmail.com>, Tatár Attila <atta@nvm.ro>
# Copyright (C) 2019 NextERP Romania (https://nexterp.ro)
# Copyright (C) 2015 Tatár Attila
# Copyright (C) 2015 Forest and Biomass Services Romania (http://www.forbiom.eu).
# Copyright (C) 2011 TOTAL PC SYSTEMS (http://www.erpsystems.ro).
# Copyright (C) 2009 (<http://www.filsystem.ro>)

{
    "name": "Romania - Accounting",
    "author": "Fekete Mihai (NextERP Romania SRL)",
    "website": "https://www.nexterp.ro",
    'category': 'Accounting/Localizations/Account Charts',
    'version': '1.0',
    "depends": [
        'account',
        'base_vat',
    ],
    "description": """
This is the module to manage the Accounting Chart, VAT structure, Fiscal Position and Tax Mapping.
It also adds the Registration Number for Romania in Odoo.
================================================================================================================

Romanian accounting chart and localization.
    """,
    "data": ['views/res_partner_view.xml',
             'data/l10n_ro_chart_data.xml',
             'data/account.group.template.csv',
             'data/account.account.template.csv',
             'data/l10n_ro_chart_post_data.xml',
             'data/account_tax_group_data.xml',
             'data/account_tax_report_data.xml',
             'data/account_tax_data.xml',
             'data/account_fiscal_position_data.xml',
             'data/account_reconcile_model_template_data.xml',
             'data/account_chart_template_data.xml',
             'data/res.bank.csv',
             ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","account_type","chart_template_id/id","reconcile"
"pcg_1011","Capital subscris nevărsat","1011","equity","l10n_ro.ro_chart_template","False"
"pcg_1012","Capital subscris vărsat","1012","equity","l10n_ro.ro_chart_template","False"
"pcg_1015","Patrimoniul regiei","1015","equity","l10n_ro.ro_chart_template","False"
"pcg_1016","Patrimoniul public","1016","equity","l10n_ro.ro_chart_template","False"
"pcg_1017","Patrimoniul privat","1017","equity","l10n_ro.ro_chart_template","False"
"pcg_1018","Patrimoniul institutelor naționale de cercetare-dezvoltare","1018","equity","l10n_ro.ro_chart_template","False"
"pcg_1031","Beneficii acordate angajaților sub forma instrumentelor de capitaluri proprii","1031","equity","l10n_ro.ro_chart_template","False"
"pcg_1033","Diferențe de curs valutar în relație cu investiția netă într-o entitate străină","1033","equity","l10n_ro.ro_chart_template","False"
"pcg_1038","Diferențe din modificarea valorii juste a activelor financiare disponibile în vederea vânzării și alte elemente de capitaluri proprii","1038","equity","l10n_ro.ro_chart_template","False"
"pcg_1041","Prime de emisiune","1041","equity","l10n_ro.ro_chart_template","False"
"pcg_1042","Prime de fuziune/divizare","1042","equity","l10n_ro.ro_chart_template","False"
"pcg_1043","Prime de aport","1043","equity","l10n_ro.ro_chart_template","False"
"pcg_1044","Prime de conversie a obligațiunilor în acțiuni","1044","equity","l10n_ro.ro_chart_template","False"
"pcg_105","Rezerve din reevaluare","105","equity","l10n_ro.ro_chart_template","False"
"pcg_1061","Rezerve legale","1061","equity","l10n_ro.ro_chart_template","False"
"pcg_1063","Rezerve statutare sau contractuale","1063","equity","l10n_ro.ro_chart_template","False"
"pcg_1068","Alte rezerve","1068","equity","l10n_ro.ro_chart_template","False"
"pcg_107","Diferențe de curs valutar din conversie","107","equity","l10n_ro.ro_chart_template","False"
"pcg_1081","Interese care nu controlează - rezultatul exercițiului financiar","1081","equity","l10n_ro.ro_chart_template","False"
"pcg_1082","Interese care nu controlează - alte capitaluri proprii","1082","equity","l10n_ro.ro_chart_template","False"
"pcg_1091","Acțiuni proprii deținute pe termen scurt","1091","equity","l10n_ro.ro_chart_template","False"
"pcg_1092","Acțiuni proprii deținute pe termen lung","1092","equity","l10n_ro.ro_chart_template","False"
"pcg_1095","Acțiuni proprii reprezentând titluri deținute de societatea absorbită la societatea absorbantă","1095","equity","l10n_ro.ro_chart_template","False"
"pcg_1171","Rezultatul reportat reprezentând profitul nerepartizat sau pierderea neacoperită","1171","equity","l10n_ro.ro_chart_template","False"
"pcg_1172","Rezultatul reportat provenit din adoptarea pentru prima dată a IAS, mai puțin IAS 29","1172","equity","l10n_ro.ro_chart_template","False"
"pcg_1173","Rezultatul reportat provenit din modificările politicilor contabile","1173","equity","l10n_ro.ro_chart_template","False"
"pcg_1174","Rezultatul reportat provenit din corectarea erorilor contabile","1174","equity","l10n_ro.ro_chart_template","False"
"pcg_1175","Rezultatul reportat reprezentând surplusul realizat din rezerve din reevaluare","1175","equity","l10n_ro.ro_chart_template","False"
"pcg_1176","Rezultatul reportat provenit din trecerea la aplicarea reglementărilor contabile conforme cu directivele europene","1176","equity","l10n_ro.ro_chart_template","False"
"pcg_121","Profit sau pierdere","121","equity_unaffected","l10n_ro.ro_chart_template","False"
"pcg_129","Repartizarea profitului","129","equity","l10n_ro.ro_chart_template","False"
"pcg_1411","Câștiguri legate de vânzarea instrumentelor de capitaluri proprii","1411","equity","l10n_ro.ro_chart_template","False"
"pcg_1412","Câștiguri legate de anularea instrumentelor de capitaluri proprii","1412","equity","l10n_ro.ro_chart_template","False"
"pcg_1491","Pierderi rezultate din vânzarea instrumentelor de capitaluri proprii","1491","equity","l10n_ro.ro_chart_template","False"
"pcg_1495","Pierderi rezultate din reorganizări, care sunt determinate de anularea titlurilor deținute","1495","equity","l10n_ro.ro_chart_template","False"
"pcg_1498","Alte pierderi legate de instrumentele de capitaluri proprii","1498","equity","l10n_ro.ro_chart_template","False"
"pcg_1511","Provizioane pentru litigii","1511","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_1512","Provizioane pentru garanții acordate clienților","1512","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_1513","Provizioane pentru dezafectare imobilizări corporale și alte acțiuni similare legate de acestea","1513","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_1514","Provizioane pentru restructurare","1514","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_1515","Provizioane pentru pensii și obligații similare","1515","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_1516","Provizioane pentru impozite","1516","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_1517","Provizioane pentru terminarea contractului de muncă","1517","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_1518","Alte provizioane","1518","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_1614","Împrumuturi externe din emisiuni de obligațiuni garantate de stat","1614","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1615","Împrumuturi externe din emisiuni de obligațiuni garantate de bănci","1615","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1617","Împrumuturi interne din emisiuni de obligațiuni garantate de stat","1617","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1618","Alte împrumuturi din emisiuni de obligațiuni","1618","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1621","Credite bancare pe termen lung","1621","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1622","Credite bancare pe termen lung nerambursate la scadență","1622","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1623","Credite externe guvernamentale","1623","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1624","Credite bancare externe garantate de stat","1624","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1625","Credite bancare externe garantate de bănci","1625","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1626","Credite de la trezoreria statului","1626","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1627","Credite bancare interne garantate de stat","1627","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1661","Datorii față de entitățile afiliate","1661","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1663","Datorii față de entitățile asociate și entitățile controlate în comun","1663","liability_current","l10n_ro.ro_chart_template","False"
"pcg_167","Alte împrumuturi și datorii asimilate","167","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1681","Dobânzi aferente împrumuturilor din emisiuni de obligațiuni","1681","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1682","Dobânzi aferente creditelor bancare pe termen lung","1682","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1685","Dobânzi aferente datoriilor față de entitățile afiliate","1685","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1686","Dobânzi aferente datoriilor fată de entitătile asociate și entitățile controlate în comun","1686","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1687","Dobânzi aferente altor împrumuturi și datorii asimilate","1687","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1691","Prime privind rambursarea obligațiunilor","1691","liability_current","l10n_ro.ro_chart_template","False"
"pcg_1692","Prime privind rambursarea altor datorii","1692","liability_current","l10n_ro.ro_chart_template","False"
"pcg_201","Cheltuieli de constituire","201","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_203","Cheltuieli de dezvoltare","203","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_205","Concesiuni, brevete, licențe, mărci comerciale, drepturi și active similare","205","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_206","Active necorporale de explorare și evaluare a resurselor minerale","206","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_2071","Fond comercial pozitiv","2071","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_2075","Fond comercial negativ","2075","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_208","Alte imobilizări necorporale","208","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_2111","Terenuri","2111","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_2112","Amenajări de terenuri","2112","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_212","Construcții","212","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_2131","Echipamente tehnologice (mașini, utilaje și instalații de lucru)","2131","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_2132","Aparate și instalații de măsurare, control și reglare","2132","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_2133","Mijloace de transport","2133","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_214","Mobilier, aparatură birotică, echipamente de protecție a valorilor umane și materiale și alte active corporale","214","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_215","Investiții imobiliare","215","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_216","Active corporale de explorare și evaluare a resurselor minerale","216","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_217","Active biologice productive","217","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_223","Instalații tehnice și mijloace de transport în curs de aprovizionare","223","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_224","Mobilier, aparatură birotică, echipamente de protecție a valorilor umane și materiale și alte active corporale în curs de aprovizionare","224","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_227","Active biologice productive în curs de aprovizionare","227","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_231","Imobilizări corporale în curs de execuție","231","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_235","Investiții imobiliare în curs de execuție","235","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_261","Acțiuni deținute la entitățile afiliate","261","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_262","Acțiuni deținute la entități asociate","262","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_263","Acțiuni deținute la entități controlate în comun","263","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_264","Titluri puse în echivalență","264","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_265","Alte titluri imobilizate","265","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_266","Certificate verzi amânate","266","asset_fixed","l10n_ro.ro_chart_template","False"
"pcg_2671","Sume de încasat de la entitățile afiliate","2671","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_2672","Dobânda aferentă sumelor de încasat de la entitățile afiliate","2672","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_2673","Creanțe față de entitățile asociate și entitățile controlate în comun","2673","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_2674","Dobânda aferentă creanțelor față de entitățile asociate și entitățile controlate în comun","2674","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_2675","Împrumuturi acordate pe termen lung","2675","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_2676","Dobânda aferentă împrumuturilor acordate pe termen lung","2676","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_2677","Obligațiuni achiziționate cu ocazia emisiunilor efectuate de terți","2677","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_2678","Alte creanțe imobilizate","2678","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_2679","Dobânzi aferente altor creanțe imobilizate","2679","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_2691","Vărsăminte de efectuat privind acțiunile deținute la entitățile afiliate","2691","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_2692","Vărsăminte de efectuat privind acțiunile deținute la entități asociate","2692","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_2693","Vărsăminte de efectuat privind acțiunile deținute la entități controlate în comun","2693","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_2695","Vărsăminte de efectuat pentru alte imobilizări financiare","2695","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_2801","Amortizarea cheltuielilor de constituire","2801","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2803","Amortizarea cheltuielilor de dezvoltare","2803","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2805","Amortizarea concesiunilor, brevetelor, licențelor, mărcilor comerciale, drepturilor și activelor similare","2805","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2806","Amortizarea activelor necorporale de explorare și evaluare a resurselor minerale","2806","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2807","Amortizarea fondului comercial","2807","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2808","Amortizarea altor imobilizări necorporale","2808","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2811","Amortizarea amenajărilor de terenuri","2811","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2812","Amortizarea construcțiilor","2812","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2813","Amortizarea instalațiilor și mijloacelor de transport","2813","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2814","Amortizarea altor imobilizări corporale","2814","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2815","Amortizarea investițiilor imobiliare","2815","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2816","Amortizarea activelor corporale de explorare și evaluare a resurselor minerale","2816","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2817","Amortizarea activelor biologice productive","2817","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2903","Ajustări pentru deprecierea cheltuielilor de dezvoltare","2903","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2905","Ajustări pentru deprecierea concesiunilor, brevetelor, licențelor, mărcilor comerciale, drepturilor și activelor similare","2905","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2906","Ajustări pentru deprecierea activelor necorporale de explorare și evaluare a resurselor minerale","2906","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2908","Ajustări pentru deprecierea altor imobilizări necorporale","2908","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2911","Ajustări pentru deprecierea terenurilor și amenajărilor de terenuri","2911","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2912","Ajustări pentru deprecierea construcțiilor","2912","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2913","Ajustări pentru deprecierea instalațiilor și mijloacelor de transport","2913","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2914","Ajustări pentru deprecierea altor imobilizări corporale","2914","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2915","Ajustări pentru deprecierea investițiilor imobiliare","2915","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2916","Ajustări pentru deprecierea activelor corporale de explorare și evaluare a resurselor minerale","2916","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2917","Ajustări pentru deprecierea activelor biologice productive","2917","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2931","Ajustări pentru deprecierea imobilizărilor corporale în curs de execuție","2931","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2935","Ajustări pentru deprecierea investiții lor imobiliare în curs de execuție","2935","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2961","Ajustări pentru pierderea de valoare a acțiunilor deținute la entitățile afiliate","2961","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2962","Ajustări pentru pierderea de valoare a acțiunilor deținute la entități asociate și entități controlate în comun","2962","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2963","Ajustări pentru pierderea de valoare a altor titluri imobilizate","2963","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2964","Ajustări pentru pierderea de valoare a sumelor de încasat de la entitățile afiliate","2964","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2965","Ajustări pentru pierderea de valoare a creanțelor față de entitățile asociate și entitățile controlate în comun","2965","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2966","Ajustări pentru pierderea de valoare a împrumuturilor acordate pe termen lung","2966","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2968","Ajustări pentru pierderea de valoare a altor creanțe","2968","expense_depreciation","l10n_ro.ro_chart_template","False"
"pcg_301","Materii prime","301","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3021","Materiale auxiliare","3021","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3022","Combustibili","3022","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3023","Materiale pentru ambalat","3023","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3024","Piese de schimb","3024","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3025","Semințe și materiale de plantat","3025","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3026","Furaje","3026","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3028","Alte materiale consumabile","3028","asset_current","l10n_ro.ro_chart_template","False"
"pcg_303","Materiale de natura obiectelor de inventar","303","asset_current","l10n_ro.ro_chart_template","False"
"pcg_308","Diferențe de preț la materii prime și materiale","308","asset_current","l10n_ro.ro_chart_template","False"
"pcg_321","Materii prime în curs de aprovizionare","321","asset_current","l10n_ro.ro_chart_template","False"
"pcg_322","Materiale consumabile în curs de aprovizionare","322","asset_current","l10n_ro.ro_chart_template","False"
"pcg_323","Materiale de natura obiectelor de inventar în curs de aprovizionare","323","asset_current","l10n_ro.ro_chart_template","False"
"pcg_326","Active biologice de natura stocurilor în curs de aprovizionare","326","asset_current","l10n_ro.ro_chart_template","False"
"pcg_327","Mărfuri în curs de aprovizionare","327","asset_current","l10n_ro.ro_chart_template","False"
"pcg_328","Ambalaje în curs de aprovizionare","328","asset_current","l10n_ro.ro_chart_template","False"
"pcg_331","Produse în curs de execuție","331","asset_current","l10n_ro.ro_chart_template","False"
"pcg_332","Servicii în curs de execuție","332","asset_current","l10n_ro.ro_chart_template","False"
"pcg_341","Semifabricate","341","asset_current","l10n_ro.ro_chart_template","False"
"pcg_345","Produse finite","345","asset_current","l10n_ro.ro_chart_template","False"
"pcg_346","Produse reziduale","346","asset_current","l10n_ro.ro_chart_template","False"
"pcg_347","Produse agricole","347","asset_current","l10n_ro.ro_chart_template","False"
"pcg_348","Diferențe de preț la produse","348","asset_current","l10n_ro.ro_chart_template","False"
"pcg_351","Materii și materiale aflate la terți","351","asset_current","l10n_ro.ro_chart_template","False"
"pcg_354","Produse aflate la terți","354","asset_current","l10n_ro.ro_chart_template","False"
"pcg_356","Active biologice de natura stocurilor aflate la terți","356","asset_current","l10n_ro.ro_chart_template","False"
"pcg_357","Mărfuri aflate la terți","357","asset_current","l10n_ro.ro_chart_template","False"
"pcg_358","Ambalaje aflate la terți","358","asset_current","l10n_ro.ro_chart_template","False"
"pcg_361","Active biologice de natura stocurilor","361","asset_current","l10n_ro.ro_chart_template","False"
"pcg_368","Diferențe de preț la active biologice de natura stocurilor","368","asset_current","l10n_ro.ro_chart_template","False"
"pcg_371","Mărfuri","371","asset_current","l10n_ro.ro_chart_template","False"
"pcg_378","Diferențe de preț la mărfuri","378","asset_current","l10n_ro.ro_chart_template","False"
"pcg_381","Ambalaje","381","asset_current","l10n_ro.ro_chart_template","False"
"pcg_388","Diferențe de preț la ambalaje","388","asset_current","l10n_ro.ro_chart_template","False"
"pcg_391","Ajustări pentru deprecierea materiilor prime","391","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3921","Ajustări pentru deprecierea materialelor consumabile","3921","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3922","Ajustări pentru deprecierea materialelor de natura obiectelor de inventar","3922","asset_current","l10n_ro.ro_chart_template","False"
"pcg_393","Ajustări pentru deprecierea producției în curs de execuție","393","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3941","Ajustări pentru deprecierea semifabricatelor","3941","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3945","Ajustări pentru deprecierea produselor finite","3945","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3946","Ajustări pentru deprecierea produselor reziduale","3946","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3947","Ajustări pentru deprecierea produselor agricole","3947","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3951","Ajustări pentru deprecierea materiilor și materialelor aflate la terți","3951","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3952","Ajustări pentru deprecierea semifabricatelor aflate la terți","3952","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3953","Ajustări pentru deprecierea produselor finite aflate la terți","3953","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3954","Ajustări pentru deprecierea produselor reziduale aflate la terți","3954","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3955","Ajustări pentru deprecierea produselor agricole aflate la terți","3955","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3956","Ajustări pentru deprecierea activelor biologice de natura stocurilor aflate la terți","3956","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3957","Ajustări pentru deprecierea mărfurilor aflate la terți","3957","asset_current","l10n_ro.ro_chart_template","False"
"pcg_3958","Ajustări pentru deprecierea ambalajelor aflate la terți","3958","asset_current","l10n_ro.ro_chart_template","False"
"pcg_396","Ajustări pentru deprecierea activelor biologice de natura stocurilor","396","asset_current","l10n_ro.ro_chart_template","False"
"pcg_397","Ajustări pentru deprecierea mărfurilor","397","asset_current","l10n_ro.ro_chart_template","False"
"pcg_398","Ajustări pentru deprecierea ambalajelor","398","asset_current","l10n_ro.ro_chart_template","False"
"ro_pcg_pay","Furnizori","401","liability_payable","l10n_ro.ro_chart_template","True"
"pcg_403","Efecte de plătit","403","liability_payable","l10n_ro.ro_chart_template","True"
"pcg_404","Furnizori de imobilizări","404","liability_payable","l10n_ro.ro_chart_template","True"
"pcg_405","Efecte de plătit pentru imobilizări","405","liability_current","l10n_ro.ro_chart_template","True"
"pcg_408","Furnizori - facturi nesosite","408","liability_current","l10n_ro.ro_chart_template","True"
"pcg_4091","Furnizori - debitori pentru cumpărări de bunuri de natura stocurilor","4091","liability_current","l10n_ro.ro_chart_template","True"
"pcg_4092","Furnizori - debitori pentru prestări de servicii","4092","liability_current","l10n_ro.ro_chart_template","True"
"pcg_4093","Avansuri acordate pentru imobilizări corporale","4093","liability_current","l10n_ro.ro_chart_template","True"
"pcg_4094","Avansuri acordate pentru imobilizări necorporale","4094","liability_current","l10n_ro.ro_chart_template","True"
"ro_pcg_recv","Clienți","4111","asset_receivable","l10n_ro.ro_chart_template","True"
"pcg_4118","Clienți incerți sau în litigiu","4118","asset_current","l10n_ro.ro_chart_template","True"
"pcg_413","Efecte de primit de la clienți","413","asset_current","l10n_ro.ro_chart_template","True"
"pcg_418","Clienți - facturi de întocmit","418","asset_current","l10n_ro.ro_chart_template","True"
"pcg_419","Clienți - creditori","419","asset_current","l10n_ro.ro_chart_template","True"
"pcg_421","Personal - salarii datorate","421","liability_payable","l10n_ro.ro_chart_template","True"
"pcg_423","Personal - ajutoare materiale datorate","423","liability_current","l10n_ro.ro_chart_template","False"
"pcg_424","Prime reprezentând participarea personalului la profit","424","liability_current","l10n_ro.ro_chart_template","False"
"pcg_425","Avansuri acordate personalului","425","asset_receivable","l10n_ro.ro_chart_template","True"
"pcg_426","Drepturi de personal neridicate","426","liability_current","l10n_ro.ro_chart_template","False"
"pcg_427","Rețineri din salarii datorate terților","427","liability_current","l10n_ro.ro_chart_template","True"
"pcg_4281","Alte datorii în legătură cu personalul","4281","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4282","Alte creanțe în legătură cu personalul","4282","asset_current","l10n_ro.ro_chart_template","False"
"pcg_4311","Contribuția unității la asigurările sociale","4311","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4312","Contribuția personalului la asigurările sociale","4312","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4313","Contribuția angajatorului pentru asigurările sociale de sănătate","4313","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4314","Contribuția angajaților pentru asigurările sociale de sănătate","4314","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4315","Contribuția de asigurări sociale","4315","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4316","Contribuția de asigurări sociale de sănătate","4316","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4318","Alte contribuții pentru asigurările sociale de sănătate","4318","liability_current","l10n_ro.ro_chart_template","False"
"pcg_436","Contributia asiguratorie pentru munca","436","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4371","Contribuția unității la fondul de șomaj","4371","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4372","Contribuția personalului la fondul de șomaj","4372","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4381","Alte datorii sociale","4381","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4382","Alte creanțe sociale","4382","asset_current","l10n_ro.ro_chart_template","False"
"pcg_4411","Impozitul pe profit","4411","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4415","Impozitul specific unor activități","4415","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4418","Impozitul pe venit","4418","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4423","TVA de plată","4423","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4424","TVA de recuperat","4424","asset_current","l10n_ro.ro_chart_template","False"
"pcg_4426","TVA deductibilă","4426","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_4427","TVA colectată","4427","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_44281","TVA neexigibilă - Colectată","44281","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_44282","TVA neexigibilă - Deductibilă","44282","asset_current","l10n_ro.ro_chart_template","False"
"pcg_444","Impozitul pe venituri de natura salariilor","444","liability_current","l10n_ro.ro_chart_template","True"
"pcg_4451","Subvenții guvernamentale","4451","asset_non_current","l10n_ro.ro_chart_template","False"
"pcg_4452","Împrumuturi nerambursabile cu caracter de subvenții","4452","asset_current","l10n_ro.ro_chart_template","False"
"pcg_4458","Alte sume primite cu caracter de subvenții","4458","asset_current","l10n_ro.ro_chart_template","False"
"pcg_446","Alte impozite, taxe și vărsăminte asimilate","446","liability_current","l10n_ro.ro_chart_template","True"
"pcg_447","Fonduri speciale - taxe și vărsăminte asimilate","447","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4481","Alte datorii față de bugetul statului","4481","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4482","Alte creanțe privind bugetul statului","4482","asset_current","l10n_ro.ro_chart_template","False"
"pcg_4511","Decontări între entitățile afiliate","4511","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4518","Dobânzi aferente decontărilor între entitățile afiliate","4518","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4531","Decontări cu entitățile asociate și entitățile controlate în comun","4531","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4538","Dobânzi aferente decontărilor cu entitățile asociate și entitățile controlate în comun","4538","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4551","Acționari/asociați - conturi curente","4551","liability_payable","l10n_ro.ro_chart_template","True"
"pcg_4558","Acționari/asociați - dobânzi la conturi curente","4558","liability_current","l10n_ro.ro_chart_template","True"
"pcg_456","Decontări cu acționarii/asociații privind capitalul","456","liability_current","l10n_ro.ro_chart_template","False"
"pcg_457","Dividende de plată","457","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4581","Decontări din operațiuni în participație - activ","4581","asset_current","l10n_ro.ro_chart_template","False"
"pcg_4582","Decontări din operațiuni în participație - pasiv","4582","asset_current","l10n_ro.ro_chart_template","False"
"pcg_461","Debitori diverși","461","asset_receivable","l10n_ro.ro_chart_template","True"
"pcg_462","Creditori diverși","462","liability_payable","l10n_ro.ro_chart_template","True"
"pcg_463","Creante reprezentand dividende repartizate in cursul exercitiului financiar","463","liability_payable","l10n_ro.ro_chart_template","True"
"pcg_4661","Datorii din operațiuni de fiducie","4661","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4662","Creanțe din operațiuni de fiducie","4662","asset_current","l10n_ro.ro_chart_template","False"
"pcg_471","Cheltuieli înregistrate în avans","471","asset_prepayments","l10n_ro.ro_chart_template","False"
"pcg_472","Venituri înregistrate în avans","472","liability_non_current","l10n_ro.ro_chart_template","False"
"pcg_473","Decontări din operațiuni în curs de clarificare","473","liability_current","l10n_ro.ro_chart_template","False"
"pcg_4751","Subvenții guvernamentale pentru investiții","4751","income_other","l10n_ro.ro_chart_template","False"
"pcg_4752","Împrumuturi nerambursabile cu caracter de subvenții pentru investiții","4752","income_other","l10n_ro.ro_chart_template","False"
"pcg_4753","Donații pentru investiții","4753","income_other","l10n_ro.ro_chart_template","False"
"pcg_4754","Plusuri de inventar de natura imobilizărilor","4754","income_other","l10n_ro.ro_chart_template","False"
"pcg_4758","Alte sume primite cu caracter de subvenții pentru investiții","4758","income_other","l10n_ro.ro_chart_template","False"
"pcg_478","Venituri în avans aferente activelor primite prin transfer de la clienți","478","income_other","l10n_ro.ro_chart_template","False"
"pcg_481","Decontări între unitate și subunități","481","liability_current","l10n_ro.ro_chart_template","False"
"pcg_482","Decontări între subunități","482","liability_current","l10n_ro.ro_chart_template","False"
"pcg_491","Ajustări pentru deprecierea creanțelor - clienți","491","liability_current","l10n_ro.ro_chart_template","False"
"pcg_495","Ajustări pentru deprecierea creanțelor - decontări în cadrul grupului și cu acționarii/asociații","495","liability_current","l10n_ro.ro_chart_template","False"
"pcg_496","Ajustări pentru deprecierea creanțelor - debitori diverși","496","liability_current","l10n_ro.ro_chart_template","False"
"pcg_501","Acțiuni deținute la entitățile afiliate","501","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_505","Obligațiuni emise și răscumpărate","505","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_506","Obligațiuni","506","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_507","Certificate verzi primite","507","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5081","Alte titluri de plasament","5081","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5088","Dobânzi la obligațiuni și titluri de plasament","5088","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5091","Vărsăminte de efectuat pentru acțiunile deținute la entitățile afiliate","5091","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5092","Vărsăminte de efectuat pentru alte investiții pe termen scurt","5092","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5112","Cecuri de încasat","5112","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5113","Efecte de încasat","5113","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5114","Efecte remise spre scontare","5114","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5121","Conturi la bănci în lei","5121","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5124","Conturi la bănci în valută","5124","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5125","Sume în curs de decontare","5125","asset_current","l10n_ro.ro_chart_template","False"
"pcg_5186","Dobânzi de plătit","5186","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5187","Dobânzi de încasat","5187","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5191","Credite bancare pe termen scurt","5191","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5192","Credite bancare pe termen scurt nerambursate la scadență","5192","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5193","Credite externe guvernamentale","5193","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5194","Credite externe garantate de stat","5194","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5195","Credite externe garantate de bănci","5195","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5196","Credite de la trezoreria statului","5196","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5197","Credite interne garantate de stat","5197","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5198","Dobânzi aferente creditelor bancare pe termen scurt","5198","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5311","Casa în lei","5311","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5314","Casa în valută","5314","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5321","Timbre fiscale și poștale","5321","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5322","Bilete de tratament și odihnă","5322","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5323","Tichete și bilete de călătorie","5323","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5328","Alte valori","5328","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5411","Acreditive în lei","5411","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_5414","Acreditive în valută","5414","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_542","Avansuri de trezorerie","542","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_581","Viramente interne","581","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_591","Ajustări pentru pierderea de valoare a acțiunilor deținute la entitățile afiliate","591","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_595","Ajustări pentru pierderea de valoare a obligațiunilor emise și răscumpărate","595","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_596","Ajustări pentru pierderea de valoare a obligațiunilor","596","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_598","Ajustări pentru pierderea de valoare a altor investiții pe termen scurt și creanțe asimilate","598","asset_cash","l10n_ro.ro_chart_template","False"
"pcg_601","Cheltuieli cu materiile prime","601","expense","l10n_ro.ro_chart_template","False"
"pcg_6021","Cheltuieli cu materiale auxiliare","6021","expense","l10n_ro.ro_chart_template","False"
"pcg_6022","Cheltuieli privind combustibilul","6022","expense","l10n_ro.ro_chart_template","False"
"pcg_6023","Cheltuieli privind materialele pentru ambalat","6023","expense","l10n_ro.ro_chart_template","False"
"pcg_6024","Cheltuieli privind piesele de schimb","6024","expense","l10n_ro.ro_chart_template","False"
"pcg_6025","Cheltuieli privind semințele și materialele de plantat","6025","expense","l10n_ro.ro_chart_template","False"
"pcg_6026","Cheltuieli privind furajele","6026","expense","l10n_ro.ro_chart_template","False"
"pcg_6028","Cheltuieli privind alte materiale consumabile","6028","expense","l10n_ro.ro_chart_template","False"
"pcg_603","Cheltuieli privind materialele de natura obiectelor de inventar","603","expense","l10n_ro.ro_chart_template","False"
"pcg_604","Cheltuieli privind materialele nestocate","604","expense","l10n_ro.ro_chart_template","False"
"pcg_605","Cheltuieli privind energia și apa","605","expense","l10n_ro.ro_chart_template","False"
"pcg_606","Cheltuieli privind activele biologice de natura stocurilor","606","expense","l10n_ro.ro_chart_template","False"
"ro_pcg_expense","Cheltuieli privind mărfurile","607","expense","l10n_ro.ro_chart_template","False"
"pcg_608","Cheltuieli privind ambalajele","608","expense","l10n_ro.ro_chart_template","False"
"pcg_609","Reduceri comerciale primite","609","expense","l10n_ro.ro_chart_template","False"
"pcg_611","Cheltuieli cu întreținerea și reparațiile","611","expense","l10n_ro.ro_chart_template","False"
"pcg_612","Cheltuieli cu redevențele, locațiile de gestiune și chiriile","612","expense","l10n_ro.ro_chart_template","False"
"pcg_613","Cheltuieli cu primele de asigurare","613","expense","l10n_ro.ro_chart_template","False"
"pcg_614","Cheltuieli cu studiile și cercetările","614","expense","l10n_ro.ro_chart_template","False"
"pcg_615","Cheltuieli cu pregătirea personalului","615","expense","l10n_ro.ro_chart_template","False"
"pcg_621","Cheltuieli cu colaboratorii","621","expense","l10n_ro.ro_chart_template","False"
"pcg_622","Cheltuieli privind comisioanele și onorariile","622","expense","l10n_ro.ro_chart_template","False"
"pcg_6231","Cheltuieli de protocol","6231","expense","l10n_ro.ro_chart_template","False"
"pcg_6232","Cheltuieli de reclamă și publicitate","6232","expense","l10n_ro.ro_chart_template","False"
"pcg_624","Cheltuieli cu transportul de bunuri și personal","624","expense","l10n_ro.ro_chart_template","False"
"pcg_625","Cheltuieli cu deplasări, detașări și transferări","625","expense","l10n_ro.ro_chart_template","False"
"pcg_626","Cheltuieli poștale și taxe de telecomunicații","626","expense","l10n_ro.ro_chart_template","False"
"pcg_627","Cheltuieli cu serviciile bancare și asimilate","627","expense","l10n_ro.ro_chart_template","False"
"pcg_628","Alte cheltuieli cu serviciile executate de terți","628","expense","l10n_ro.ro_chart_template","False"
"pcg_6351","Cheltuieli cu alte impozite, taxe și vărsăminte asimilate","6351","expense","l10n_ro.ro_chart_template","False"
"pcg_6352","Cheltuieli cu alte impozite, taxe și vărsăminte asimilate nedeductibile","6352","expense","l10n_ro.ro_chart_template","False"
"pcg_641","Cheltuieli cu salariile personalului","641","expense","l10n_ro.ro_chart_template","False"
"pcg_6421","Cheltuieli cu avantajele în natură acordate salariaților","6421","expense","l10n_ro.ro_chart_template","False"
"pcg_6422","Cheltuieli cu tichetele acordate salariaților","6422","expense","l10n_ro.ro_chart_template","False"
"pcg_643","Cheltuieli cu remunerarea în instrumente de capitaluri proprii","643","expense","l10n_ro.ro_chart_template","False"
"pcg_644","Cheltuieli cu primele reprezentând participarea personalului la profit","644","expense","l10n_ro.ro_chart_template","False"
"pcg_6451","Cheltuieli privind contribuția unității la asigurările sociale","6451","expense","l10n_ro.ro_chart_template","False"
"pcg_6452","Cheltuieli privind contribuția unității pentru ajutorul de șomaj","6452","expense","l10n_ro.ro_chart_template","False"
"pcg_6453","Cheltuieli privind contribuția angajatorului pentru asigurările sociale de sănătate","6453","expense","l10n_ro.ro_chart_template","False"
"pcg_6455","Cheltuieli privind contribuția unității la asigurările de viață","6455","expense","l10n_ro.ro_chart_template","False"
"pcg_6456","Cheltuieli privind contribuția unității la fondurile de pensii facultative","6456","expense","l10n_ro.ro_chart_template","False"
"pcg_6457","Cheltuieli privind contribuția unității la primele de asigurare voluntară de sănătate","6457","expense","l10n_ro.ro_chart_template","False"
"pcg_6458","Alte cheltuieli privind asigurările și protecția socială","6458","expense","l10n_ro.ro_chart_template","False"
"pcg_646","Cheltuieli privind contribuția asiguratorie pentru muncă","646","expense","l10n_ro.ro_chart_template","False"
"pcg_6511","Cheltuieli ocazionate de constituirea fiduciei","6511","expense","l10n_ro.ro_chart_template","False"
"pcg_6512","Cheltuieli din derularea operațiunilor de fiducie","6512","expense","l10n_ro.ro_chart_template","False"
"pcg_6513","Cheltuieli din lichidarea operațiunilor de fiducie","6513","expense","l10n_ro.ro_chart_template","False"
"pcg_652","Cheltuieli cu protecția mediului înconjurător","652","expense","l10n_ro.ro_chart_template","False"
"pcg_654","Pierderi din creanțe și debitori diverși","654","expense","l10n_ro.ro_chart_template","False"
"pcg_655","Cheltuieli din reevaluarea imobilizărilor corporale","655","expense","l10n_ro.ro_chart_template","False"
"pcg_6581","Despăgubiri, amenzi și penalități","6581","expense","l10n_ro.ro_chart_template","False"
"pcg_6582","Donații acordate","6582","expense","l10n_ro.ro_chart_template","False"
"pcg_6583","Cheltuieli privind activele cedate și alte operațiuni de capital","6583","expense","l10n_ro.ro_chart_template","False"
"pcg_6584","Cheltuieli cu sumele sau bunurile acordate ca sponsorizări","6584","expense","l10n_ro.ro_chart_template","False"
"pcg_6586","Cheltuieli reprezentând transferuri și contribuții datorate în baza unor acte normative speciale","6586","expense","l10n_ro.ro_chart_template","False"
"pcg_6587","Cheltuieli privind calamitățile și alte evenimente similare","6587","expense","l10n_ro.ro_chart_template","False"
"pcg_65881","Alte cheltuieli de exploatare","65881","expense","l10n_ro.ro_chart_template","False"
"pcg_65882","Alte cheltuieli de exploatare nedeductibile","65882","expense","l10n_ro.ro_chart_template","False"
"pcg_663","Pierderi din creanțe legate de participații","663","expense","l10n_ro.ro_chart_template","False"
"pcg_6641","Cheltuieli privind imobilizările financiare cedate","6641","expense","l10n_ro.ro_chart_template","False"
"pcg_6642","Pierderi din investițiile pe termen scurt cedate","6642","expense","l10n_ro.ro_chart_template","False"
"pcg_6651","Diferențe nefavorabile de curs valutar legate de elementele monetare exprimate în valută","6651","expense","l10n_ro.ro_chart_template","False"
"pcg_6652","Diferențe nefavorabile de curs valutar din evaluarea elementelor monetare care fac parte din investiția netă într-o entitate străină","6652","expense","l10n_ro.ro_chart_template","False"
"pcg_666","Cheltuieli privind dobânzile","666","expense","l10n_ro.ro_chart_template","False"
"pcg_667","Cheltuieli privind sconturile acordate","667","expense","l10n_ro.ro_chart_template","False"
"pcg_668","Alte cheltuieli financiare","668","expense","l10n_ro.ro_chart_template","False"
"pcg_6811","Cheltuieli de exploatare privind amortizarea imobilizărilor","6811","expense","l10n_ro.ro_chart_template","False"
"pcg_6812","Cheltuieli de exploatare privind provizioanele","6812","expense","l10n_ro.ro_chart_template","False"
"pcg_6813","Cheltuieli de exploatare privind ajustările pentru deprecierea imobilizărilor","6813","expense","l10n_ro.ro_chart_template","False"
"pcg_6814","Cheltuieli de exploatare privind ajustările pentru deprecierea activelor circulante","6814","expense","l10n_ro.ro_chart_template","False"
"pcg_6817","Cheltuieli de exploatare privind ajustările pentru deprecierea fondului comercial","6817","expense","l10n_ro.ro_chart_template","False"
"pcg_6861","Cheltuieli privind actualizarea provizioanelor","6861","expense","l10n_ro.ro_chart_template","False"
"pcg_6863","Cheltuieli financiare privind ajustările pentru pierderea de valoare a imobilizărilor financiare","6863","expense","l10n_ro.ro_chart_template","False"
"pcg_6864","Cheltuieli financiare privind ajustările pentru pierderea de valoare a activelor circulante","6864","expense","l10n_ro.ro_chart_template","False"
"pcg_6865","Cheltuieli financiare privind amortizarea diferențelor aferente titlurilor de stat","6865","expense","l10n_ro.ro_chart_template","False"
"pcg_6868","Cheltuieli financiare privind amortizarea primelor de rambursare a obligațiunilor și a altor datorii","6868","expense","l10n_ro.ro_chart_template","False"
"pcg_691","Cheltuieli cu impozitul pe profit","691","expense","l10n_ro.ro_chart_template","False"
"pcg_695","Cheltuieli cu impozitul specific unor activitati","695","expense","l10n_ro.ro_chart_template","False"
"pcg_698","Cheltuieli cu impozitul pe venit și cu alte impozite care nu apar in elementele de mai sus","698","expense","l10n_ro.ro_chart_template","False"
"pcg_7015","Venituri din vânzarea produselor finite","7015","income","l10n_ro.ro_chart_template","False"
"pcg_7017","Venituri din vânzarea produselor agricole","7017","income","l10n_ro.ro_chart_template","False"
"pcg_7018","Venituri din vânzarea activelor biologice de natura stocurilor","7018","income","l10n_ro.ro_chart_template","False"
"pcg_702","Venituri din vânzarea semifabricatelor","702","income","l10n_ro.ro_chart_template","False"
"pcg_703","Venituri din vânzarea produselor reziduale","703","income","l10n_ro.ro_chart_template","False"
"pcg_704","Venituri din servicii prestate","704","income","l10n_ro.ro_chart_template","False"
"pcg_705","Venituri din studii și cercetări","705","income","l10n_ro.ro_chart_template","False"
"pcg_706","Venituri din redevențe, locații de gestiune și chirii","706","income","l10n_ro.ro_chart_template","False"
"ro_pcg_sale","Venituri din vânzarea mărfurilor","707","income","l10n_ro.ro_chart_template","False"
"pcg_708","Venituri din activități diverse","708","income","l10n_ro.ro_chart_template","False"
"pcg_709","Reduceri comerciale acordate","709","income","l10n_ro.ro_chart_template","False"
"pcg_711","Venituri aferente costurilor stocurilor de produse","711","income","l10n_ro.ro_chart_template","False"
"pcg_712","Venituri aferente costurilor serviciilor în curs de execuție","712","income","l10n_ro.ro_chart_template","False"
"pcg_721","Venituri din producția de imobilizări necorporale","721","income","l10n_ro.ro_chart_template","False"
"pcg_722","Venituri din producția de imobilizări corporale","722","income","l10n_ro.ro_chart_template","False"
"pcg_725","Venituri din producția de investiții imobiliare","725","income","l10n_ro.ro_chart_template","False"
"pcg_7411","Venituri din subvenții de exploatare aferente cifrei de afaceri","7411","income","l10n_ro.ro_chart_template","False"
"pcg_7412","Venituri din subvenții de exploatare pentru materii prime și materiale","7412","income","l10n_ro.ro_chart_template","False"
"pcg_7413","Venituri din subvenții de exploatare pentru alte cheltuieli externe","7413","income","l10n_ro.ro_chart_template","False"
"pcg_7414","Venituri din subvenții de exploatare pentru plata personalului","7414","income","l10n_ro.ro_chart_template","False"
"pcg_7415","Venituri din subvenții de exploatare pentru asigurări și protecție socială","7415","income","l10n_ro.ro_chart_template","False"
"pcg_7416","Venituri din subvenții de exploatare pentru alte cheltuieli de exploatare","7416","income","l10n_ro.ro_chart_template","False"
"pcg_7417","Venituri din subvenții de exploatare în caz de calamități și alte evenimente similare","7417","income","l10n_ro.ro_chart_template","False"
"pcg_7418","Venituri din subvenții de exploatare pentru dobânda datorată","7418","income","l10n_ro.ro_chart_template","False"
"pcg_7419","Venituri din subvenții de exploatare aferente altor venituri","7419","income","l10n_ro.ro_chart_template","False"
"pcg_7511","Venituri ocazionate de constituirea fiduciei","7511","income","l10n_ro.ro_chart_template","False"
"pcg_7512","Venituri din derularea operațiunilor de fiducie","7512","income","l10n_ro.ro_chart_template","False"
"pcg_7513","Venituri din lichidarea operațiunilor de fiducie","7513","income","l10n_ro.ro_chart_template","False"
"pcg_754","Venituri din creanțe reactivate și debitori diverși","754","income","l10n_ro.ro_chart_template","False"
"pcg_755","Venituri din reevaluarea imobilizărilor corporale","755","income","l10n_ro.ro_chart_template","False"
"pcg_7581","Venituri din despăgubiri, amenzi și penalități","7581","income","l10n_ro.ro_chart_template","False"
"pcg_7582","Venituri din donații primite","7582","income","l10n_ro.ro_chart_template","False"
"pcg_7583","Venituri din vânzarea activelor și alte operațiuni de capital","7583","income","l10n_ro.ro_chart_template","False"
"pcg_7584","Venituri din subvenții pentru investiții","7584","income","l10n_ro.ro_chart_template","False"
"pcg_7586","Venituri reprezentând transferuri cuvenite în baza unor acte normative speciale","7586","income","l10n_ro.ro_chart_template","False"
"pcg_7588","Alte venituri din exploatare","7588","income","l10n_ro.ro_chart_template","False"
"pcg_7611","Venituri din acțiuni deținute la entitățile afiliate","7611","income","l10n_ro.ro_chart_template","False"
"pcg_7612","Venituri din acțiuni deținute la entități asociate","7612","income","l10n_ro.ro_chart_template","False"
"pcg_7613","Venituri din acțiuni deținute la entități controlate în comun","7613","income","l10n_ro.ro_chart_template","False"
"pcg_7615","Venituri din alte imobilizări financiare","7615","income","l10n_ro.ro_chart_template","False"
"pcg_762","Venituri din investiții financiare pe termen scurt","762","income","l10n_ro.ro_chart_template","False"
"pcg_7641","Venituri din imobilizări financiare cedate","7641","income","l10n_ro.ro_chart_template","False"
"pcg_7642","Câștiguri din investiții pe termen scurt cedate","7642","income","l10n_ro.ro_chart_template","False"
"pcg_7651","Diferențe favorabile de curs valutar legate de elementele monetare exprimate în valută","7651","income","l10n_ro.ro_chart_template","False"
"pcg_7652","Diferențe favorabile de curs valutar din evaluarea elementelor monetare care fac parte din investiția netă într-o entitate străină","7652","income","l10n_ro.ro_chart_template","False"
"pcg_766","Venituri din dobânzi","766","income","l10n_ro.ro_chart_template","False"
"pcg_767","Venituri din sconturi obținute","767","income","l10n_ro.ro_chart_template","False"
"pcg_768","Alte venituri financiare","768","income","l10n_ro.ro_chart_template","False"
"pcg_7812","Venituri din provizioane","7812","income","l10n_ro.ro_chart_template","False"
"pcg_7813","Venituri din ajustări pentru deprecierea imobilizărilor","7813","income","l10n_ro.ro_chart_template","False"
"pcg_7814","Venituri din ajustări pentru deprecierea activelor circulante","7814","income","l10n_ro.ro_chart_template","False"
"pcg_7815","Venituri din fondul comercial negativ","7815","income","l10n_ro.ro_chart_template","False"
"pcg_7863","Venituri financiare din ajustări pentru pierderea de valoare a imobilizărilor financiare","7863","income","l10n_ro.ro_chart_template","False"
"pcg_7864","Venituri financiare din ajustări pentru pierderea de valoare a activelor circulante","7864","income","l10n_ro.ro_chart_template","False"
"pcg_7865","Venituri financiare din amortizarea diferențelor aferente titlurilor de stat","7865","income","l10n_ro.ro_chart_template","False"
"pcg_8011","Giruri și garanții acordate","8011","off_balance","l10n_ro.ro_chart_template","False"
"pcg_8018","Alte angajamente acordate","8018","off_balance","l10n_ro.ro_chart_template","False"
"pcg_8021","Giruri și garanții primite","8021","off_balance","l10n_ro.ro_chart_template","False"
"pcg_8028","Alte angajamente primite","8028","off_balance","l10n_ro.ro_chart_template","False"
"pcg_8031","Imobilizări corporale luate cu chirie","8031","off_balance","l10n_ro.ro_chart_template","False"
"pcg_8032","Valori materiale primite spre prelucrare sau reparare","8032","off_balance","l10n_ro.ro_chart_template","False"
"pcg_8033","Valori materiale primite în păstrare sau custodie","8033","off_balance","l10n_ro.ro_chart_template","False"
"pcg_8034","Debitori scoși din activ, urmăriți în continuare","8034","off_balance","l10n_ro.ro_chart_template","False"
"pcg_8035","Stocuri de natura obiectelor de inventar date în folosință","8035","off_balance","l10n_ro.ro_chart_template","False"
"pcg_8036","Redevențe, locații de gestiune, chirii și alte datorii asimilate","8036","off_balance","l10n_ro.ro_chart_template","False"
"pcg_8037","Efecte scontate neajunse la scadență","8037","off_balance","l10n_ro.ro_chart_template","False"
"pcg_8038","Bunuri primite în administrare, concesiune și cu chirie","8038","off_balance","l10n_ro.ro_chart_template","False"
"pcg_8039","Alte valori în afara bilanțului","8039","off_balance","l10n_ro.ro_chart_template","False"
"pcg_804","Certificate verzi","804","off_balance","l10n_ro.ro_chart_template","False"
"pcg_8051","Dobânzi de plătit","8051","off_balance","l10n_ro.ro_chart_template","False"
"pcg_8052","Dobânzi de încasat","8052","off_balance","l10n_ro.ro_chart_template","False"
"pcg_806","Certificate de emisii de gaze cu efect de seră","806","off_balance","l10n_ro.ro_chart_template","False"
"pcg_807","Active contingente","807","off_balance","l10n_ro.ro_chart_template","False"
"pcg_808","Datorii contingente","808","off_balance","l10n_ro.ro_chart_template","False"
"pcg_809","Creanțe preluate prin cesionare","809","off_balance","l10n_ro.ro_chart_template","False"
"pcg_891","Bilanț de deschidere","891","off_balance","l10n_ro.ro_chart_template","False"
"pcg_892","Bilanț de închidere","892","off_balance","l10n_ro.ro_chart_template","False"
"pcg_901","Decontări interne privind cheltuielile","901","off_balance","l10n_ro.ro_chart_template","False"
"pcg_902","Decontări interne privind producția obținută","902","off_balance","l10n_ro.ro_chart_template","False"
"pcg_903","Decontări interne privind diferențele de preț","903","off_balance","l10n_ro.ro_chart_template","False"
"pcg_921","Cheltuielile activității de bază","921","off_balance","l10n_ro.ro_chart_template","False"
"pcg_922","Cheltuielile activităților auxiliare","922","off_balance","l10n_ro.ro_chart_template","False"
"pcg_923","Cheltuieli indirecte de producție","923","off_balance","l10n_ro.ro_chart_template","False"
"pcg_924","Cheltuieli generale de administrație","924","off_balance","l10n_ro.ro_chart_template","False"
"pcg_925","Cheltuieli de desfacere","925","off_balance","l10n_ro.ro_chart_template","False"
"pcg_931","Costul producției obținute","931","off_balance","l10n_ro.ro_chart_template","False"
"pcg_933","Costul producției în curs de execuție","933","off_balance","l10n_ro.ro_chart_template","False"

```

## File: data\account.group.template.csv

```csv
"id","name","code_prefix_start","code_prefix_end","chart_template_id/id","parent_id/id"
"account_group_all","CLASELE 1-7","","","ro_chart_template",""
"account_group_cls_15","CLASELE 1-5","","","ro_chart_template","account_group_all"
"account_group_cls_67","CLASELE 6-7","","","ro_chart_template","account_group_all"
"account_group_1","CONTURI DE CAPITALURI, PROVIZIOANE, ÎMPRUMUTURI ȘI DATORII ASIMILATE","1","","ro_chart_template","account_group_cls_15"
"account_group_10","Capital și rezerve","10","","ro_chart_template","account_group_1"
"account_group_101","Capital","101","","ro_chart_template","account_group_10"
"account_group_103","Alte elemente de capitaluri proprii","103","","ro_chart_template","account_group_10"
"account_group_104","Prime de capital","104","","ro_chart_template","account_group_10"
"account_group_106","Rezerve","106","","ro_chart_template","account_group_10"
"account_group_108","Interese care nu controlează","108","","ro_chart_template","account_group_10"
"account_group_109","Acțiuni proprii","109","","ro_chart_template","account_group_10"
"account_group_11","Rezultatul reportat","11","","ro_chart_template","account_group_1"
"account_group_117","Acțiuni proprii","117","","ro_chart_template","account_group_11"
"account_group_12","Rezultatul exercițiului financiar","12","","ro_chart_template","account_group_1"
"account_group_14","Câștiguri sau pierderi legate de emiterea, răscumpărarea, vânzarea, cedarea cu titlu gratuit sau anularea instrumentelor de capitaluri proprii","14","","ro_chart_template","account_group_1"
"account_group_141","Câștiguri legate de vânzarea sau anularea instrumentelor de capitaluri proprii","141","","ro_chart_template","account_group_14"
"account_group_149","Pierderi legate de emiterea, răscumpărarea, vânzarea, cedarea cu titlu gratuit sau anularea instrumentelor de capitaluri proprii","149","","ro_chart_template","account_group_14"
"account_group_15","Provizioane","15","","ro_chart_template","account_group_1"
"account_group_151","Provizioane","151","","ro_chart_template","account_group_15"
"account_group_16","Împrumuturi și datorii asimilate","16","","ro_chart_template","account_group_1"
"account_group_161","Împrumuturi din emisiuni de obligațiuni","161","","ro_chart_template","account_group_16"
"account_group_162","Credite bancare pe termen lung","162","","ro_chart_template","account_group_16"
"account_group_166","Datorii care privesc imobilizările financiare","166","","ro_chart_template","account_group_16"
"account_group_168","Dobânzi aferente împrumuturilor și datoriilor asimilate","168","","ro_chart_template","account_group_16"
"account_group_169","Prime privind rambursarea obligațiunilor și a altor datorii","169","","ro_chart_template","account_group_16"
"account_group_2","CONTURI DE IMOBILIZĂRI","2","","ro_chart_template","account_group_cls_15"
"account_group_20","Imobilizări necorporale","20","","ro_chart_template","account_group_2"
"account_group_207","Fond comercial","207","","ro_chart_template","account_group_20"
"account_group_21","Imobilizări corporale","21","","ro_chart_template","account_group_2"
"account_group_211","Terenuri și amenajări de terenuri","211","","ro_chart_template","account_group_21"
"account_group_213","Instalații tehnice și mijloace de transport","213","","ro_chart_template","account_group_21"
"account_group_22","Imobilizări corporale în curs de aprovizionare","22","","ro_chart_template","account_group_2"
"account_group_23","Imobilizări în curs","23","","ro_chart_template","account_group_2"
"account_group_26","Imobilizări financiare","26","","ro_chart_template","account_group_2"
"account_group_267","Creanțe imobilizate","267","","ro_chart_template","account_group_26"
"account_group_269","Vărsăminte de efectuat pentru imobilizări financiare","269","","ro_chart_template","account_group_26"
"account_group_28","Amortizări privind imobilizările","28","","ro_chart_template","account_group_2"
"account_group_280","Amortizări privind imobilizările necorporale","280","","ro_chart_template","account_group_28"
"account_group_281","Amortizări privind imobilizările corporale","281","","ro_chart_template","account_group_28"
"account_group_29","Ajustări pentru deprecierea sau pierderea de valoare a imobilizărilor","29","","ro_chart_template","account_group_2"
"account_group_290","Ajustări pentru deprecierea imobilizărilor necorporale","290","","ro_chart_template","account_group_29"
"account_group_291","Ajustări pentru deprecierea imobilizărilor corporale","291","","ro_chart_template","account_group_29"
"account_group_293","Ajustări pentru deprecierea imobilizărilor în curs de execuție","293","","ro_chart_template","account_group_29"
"account_group_296","Ajustări pentru pierderea de valoare a imobilizărilor financiare","296","","ro_chart_template","account_group_29"
"account_group_3","CONTURI DE STOCURI șI PRODUCțIE ÎN CURS DE EXECUțIE","3","","ro_chart_template","account_group_cls_15"
"account_group_30","Stocuri de materii prime și materiale","30","","ro_chart_template","account_group_3"
"account_group_302","Materiale consumabile","302","","ro_chart_template","account_group_30"
"account_group_32","Stocuri în curs de aprovizionare","32","","ro_chart_template","account_group_3"
"account_group_33","Producție în curs de execuție","33","","ro_chart_template","account_group_3"
"account_group_34","Produse","34","","ro_chart_template","account_group_3"
"account_group_35","Stocuri aflate la terți","35","","ro_chart_template","account_group_3"
"account_group_36","Active biologice de natura stocurilor","36","","ro_chart_template","account_group_3"
"account_group_37","Mărfuri","37","","ro_chart_template","account_group_3"
"account_group_38","Ambalaje","38","","ro_chart_template","account_group_3"
"account_group_39","Ajustări pentru deprecierea stocurilor și producției în curs de execuție","39","","ro_chart_template","account_group_3"
"account_group_392","Ajustări pentru deprecierea materialelor","392","","ro_chart_template","account_group_39"
"account_group_393","Ajustări pentru deprecierea producției în curs de execuție","393","","ro_chart_template","account_group_39"
"account_group_394","Ajustări pentru deprecierea produselor","394","","ro_chart_template","account_group_39"
"account_group_395","Ajustări pentru deprecierea stocurilor aflate la terți","395","","ro_chart_template","account_group_39"
"account_group_4","CONTURI DE TERȚI","4","","ro_chart_template","account_group_cls_15"
"account_group_40","Furnizori și conturi asimilate","40","","ro_chart_template","account_group_4"
"account_group_409","Furnizori - debitori","409","","ro_chart_template","account_group_40"
"account_group_41","Clienți și conturi asimilate","41","","ro_chart_template","account_group_4"
"account_group_411","Clienți","411","","ro_chart_template","account_group_41"
"account_group_42","Personal și conturi asimilate","42","","ro_chart_template","account_group_4"
"account_group_428","Alte datorii și creanțe în legătură cu personalul","428","","ro_chart_template","account_group_42"
"account_group_43","Asigurări sociale, protecția socială și conturi asimilate","43","","ro_chart_template","account_group_4"
"account_group_431","Asigurări sociale","431","","ro_chart_template","account_group_43"
"account_group_437","Ajutor de șomaj","437","","ro_chart_template","account_group_43"
"account_group_438","Alte datorii și creanțe sociale","438","","ro_chart_template","account_group_43"
"account_group_44","Bugetul statului, fonduri speciale și conturi asimilate","44","","ro_chart_template","account_group_4"
"account_group_441","Impozitul pe profit/venit","441","","ro_chart_template","account_group_44"
"account_group_442","Taxa pe valoarea adăugată","442","","ro_chart_template","account_group_44"
"account_group_4428","TVA neexigibilă","4428","","ro_chart_template","account_group_442"
"account_group_445","Subvenții","445","","ro_chart_template","account_group_44"
"account_group_448","Alte datorii și creanțe cu bugetul statului","448","","ro_chart_template","account_group_44"
"account_group_45","Grup și acționari/asociați","45","","ro_chart_template","account_group_4"
"account_group_451","Decontări între entitățile afiliate","451","","ro_chart_template","account_group_45"
"account_group_453","Decontări cu entitățile asociate și entitățile controlate în comun","453","","ro_chart_template","account_group_45"
"account_group_455","Sume datorate acționarilor/asociaților","455","","ro_chart_template","account_group_45"
"account_group_458","Decontări din operațiuni în participație","458","","ro_chart_template","account_group_45"
"account_group_46","Debitori și creditori diverși","46","","ro_chart_template","account_group_4"
"account_group_466","Decontări din operațiuni de fiducie","466","","ro_chart_template","account_group_46"
"account_group_47","Conturi de subvenții, regularizare și asimilate","47","","ro_chart_template","account_group_4"
"account_group_475","Subvenții pentru investiții","475","","ro_chart_template","account_group_47"
"account_group_48","Decontări în cadrul unității","48","","ro_chart_template","account_group_4"
"account_group_49","Ajustări pentru deprecierea creanțelor","49","","ro_chart_template","account_group_4"
"account_group_5","CONTURI DE TREZORERIE","5","","ro_chart_template","account_group_cls_15"
"account_group_50","Investiții pe termen scurt","50","","ro_chart_template","account_group_5"
"account_group_508","Alte investiții pe termen scurt și creanțe asimilate","508","","ro_chart_template","account_group_50"
"account_group_509","Vărsăminte de efectuat pentru investițiile pe termen scurt","509","","ro_chart_template","account_group_50"
"account_group_51","Conturi la bănci","51","","ro_chart_template","account_group_5"
"account_group_511","Valori de încasat","511","","ro_chart_template","account_group_51"
"account_group_512","Valori de încasat","512","","ro_chart_template","account_group_51"
"account_group_518","Valori de încasat","518","","ro_chart_template","account_group_51"
"account_group_519","Valori de încasat","519","","ro_chart_template","account_group_51"
"account_group_53","Casa","53","","ro_chart_template","account_group_5"
"account_group_531","Casa","531","","ro_chart_template","account_group_53"
"account_group_532","Alte valori","532","","ro_chart_template","account_group_53"
"account_group_54","Acreditive","54","","ro_chart_template","account_group_5"
"account_group_541","Acreditive","541","","ro_chart_template","account_group_54"
"account_group_58","Viramente interne","58","","ro_chart_template","account_group_5"
"account_group_59","Ajustări pentru pierderea de valoare a conturilor de trezorerie","59","","ro_chart_template","account_group_5"
"account_group_6","CONTURI DE CHELTUIELI","6","","ro_chart_template","account_group_cls_67"
"account_group_60","Cheltuieli privind stocurile","60","","ro_chart_template","account_group_6"
"account_group_602","Cheltuieli cu materialele consumabile","602","","ro_chart_template","account_group_60"
"account_group_61","Cheltuieli cu serviciile executate de terți","61","","ro_chart_template","account_group_6"
"account_group_62","Cheltuieli cu alte servicii executate de terți","62","","ro_chart_template","account_group_6"
"account_group_623","Cheltuieli de protocol, reclama si publicitate","623","","ro_chart_template","account_group_62"
"account_group_63","Cheltuieli cu alte impozite, taxe și vărsăminte asimilate","63","","ro_chart_template","account_group_6"
"account_group_635","Cheltuieli cu alte impozite, taxe și vărsăminte asimilate","635","","ro_chart_template","account_group_63"
"account_group_64","Cheltuieli cu personalul","64","","ro_chart_template","account_group_6"
"account_group_642","Cheltuieli cu avantajele în natură și tichetele acordate salariaților","642","","ro_chart_template","account_group_64"
"account_group_645","Cheltuieli privind asigurările și protecția socială","645","","ro_chart_template","account_group_64"
"account_group_65","Alte cheltuieli de exploatare","65","","ro_chart_template","account_group_6"
"account_group_651","Cheltuieli din operațiuni de fiducie","651","","ro_chart_template","account_group_65"
"account_group_658","Alte cheltuieli de exploatare","658","","ro_chart_template","account_group_65"
"account_group_6588","Alte cheltuieli de exploatare","6588","","ro_chart_template","account_group_658"
"account_group_66","Cheltuieli financiare","66","","ro_chart_template","account_group_6"
"account_group_664","Cheltuieli privind investițiile financiare cedate","664","","ro_chart_template","account_group_66"
"account_group_665","Cheltuieli din diferențe de curs valutar","665","","ro_chart_template","account_group_66"
"account_group_68","Cheltuieli cu amortizările, provizioanele și ajustările pentru depreciere sau pierdere de valoare","68","","ro_chart_template","account_group_6"
"account_group_681","Cheltuieli de exploatare privind amortizările, provizioanele și ajustările pentru depreciere","681","","ro_chart_template","account_group_68"
"account_group_686","Cheltuieli financiare privind amortizările, provizioanele și ajustările pentru pierdere de valoare","686","","ro_chart_template","account_group_68"
"account_group_69","Cheltuieli cu impozitul pe profit și alte impozite","69","","ro_chart_template","account_group_6"
"account_group_7","CONTURI DE VENITURI","7","","ro_chart_template","account_group_cls_67"
"account_group_70","Cifra de afaceri netă","70","","ro_chart_template","account_group_7"
"account_group_701","Venituri din vânzarea produselor finite, produselor agricole și a activelor biologice de natura stocurilor","701","","ro_chart_template","account_group_70"
"account_group_71","Venituri aferente costului producției în curs de execuție","71","","ro_chart_template","account_group_7"
"account_group_72","Venituri din producția de imobilizări","72","","ro_chart_template","account_group_7"
"account_group_74","Venituri din subvenții de exploatare","74","","ro_chart_template","account_group_7"
"account_group_741","Venituri din subvenții de exploatare","741","","ro_chart_template","account_group_74"
"account_group_75","Alte venituri din exploatare","75","","ro_chart_template","account_group_7"
"account_group_751","Venituri din operațiuni de fiducie","751","","ro_chart_template","account_group_75"
"account_group_758","Alte venituri din exploatare","758","","ro_chart_template","account_group_75"
"account_group_76","Venituri financiare","76","","ro_chart_template","account_group_7"
"account_group_761","Venituri din imobilizări financiare","761","","ro_chart_template","account_group_76"
"account_group_764","Venituri din investiții financiare cedate","764","","ro_chart_template","account_group_76"
"account_group_765","Venituri din diferențe de curs valutar","765","","ro_chart_template","account_group_76"
"account_group_78","Venituri din provizioane, amortizări și ajustări pentru depreciere sau pierdere de valoare","78","","ro_chart_template","account_group_7"
"account_group_781","Venituri din provizioane și ajustări pentru depreciere privind activitatea de exploatare","781","","ro_chart_template","account_group_78"
"account_group_786","Venituri financiare din amortizări și ajustări pentru pierdere de valoare","786","","ro_chart_template","account_group_78"

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_ro.ro_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_fiscal_position_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Fiscal Position Templates -->

    <record id="fiscal_position_template_1" model="account.fiscal.position.template">
        <field name="name">Regim National (TVA)</field>
        <field name="country_id" ref="base.ro"/>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="sequence">10</field>
        <field name="chart_template_id" ref="ro_chart_template"/>
    </record>

    <record id="fiscal_position_template_11" model="account.fiscal.position.template">
        <field name="name">Regim National</field>
        <field name="country_id" ref="base.ro"/>
        <field name="auto_apply" eval="True"/>
        <field name="sequence">11</field>
        <field name="chart_template_id" ref="ro_chart_template"/>
    </record>

    <record id="fiscal_position_template_2" model="account.fiscal.position.template">
        <field name="name">Regim Taxare Inversa</field>
        <field name="country_id" ref="base.ro"/>
        <field name="vat_required" eval="True"/>
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">12</field>
    </record>

    <record id="fiscal_position_template_8" model="account.fiscal.position.template">
        <field name="name">Regim TVA la Incasare</field>
        <field name="vat_required" eval="True"/>
        <field name="country_id" ref="base.ro"/>
        <field name="sequence">13</field>
        <field name="chart_template_id" ref="ro_chart_template"/>
    </record>

    <record id="fiscal_position_template_3" model="account.fiscal.position.template">
        <field name="name">Regim Intra-Comunitar (TVA)</field>
        <field name="country_group_id" ref="base.europe"/>
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="auto_apply" eval="True"/>
        <field name="sequence">14</field>
        <field name="vat_required" eval="True"/>
    </record>

    <record id="fiscal_position_template_4" model="account.fiscal.position.template">
        <field name="name">Regim Intra-Comunitar Scutit</field>
        <field name="country_group_id" ref="base.europe"/>
        <field name="auto_apply" eval="True"/>
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">15</field>
    </record>

    <record id="fiscal_position_template_51" model="account.fiscal.position.template">
        <field name="name">Regim Scutite - cu drept de deducere</field>
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="note">Pozitia se refera atat la livrari / achizitii intracomunitare cat si extracomunitare cu drept de deducere</field>
        <field name="sequence">16</field>
    </record>

    <record id="fiscal_position_template_52" model="account.fiscal.position.template">
        <field name="name">Regim Scutite - fara drept de deducere</field>
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="note">Pozitia se refera atat la livrari / achizitii intracomunitare cat si extracomunitare fara drept de deducere</field>
        <field name="sequence">17</field>
    </record>

    <record id="fiscal_position_template_6" model="account.fiscal.position.template">
        <field name="name">Regim Intra-Comunitar Neimpozabile</field>
        <field name="country_group_id" ref="base.europe"/>
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">18</field>
    </record>

    <record id="fiscal_position_template_7" model="account.fiscal.position.template">
        <field name="name">Regim Extra-Comunitar</field>
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="note">Pozitia se refera la livrari extracomunitare care sunt taxabile.</field>
        <field name="sequence">19</field>
        <field name="auto_apply" eval="True"/>
    </record>

    <!-- account.fiscal.position.tax.template -->
    <!-- Inverse taxation -->
    <!-- Sales -->
    <record id="afptt_inverse_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvac_00"/>
        <field name="tax_dest_id" ref="tvati"/>
    </record>
    <record id="afptt_inverse_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvac_05"/>
        <field name="tax_dest_id" ref="tvati"/>
    </record>
    <record id="afptt_inverse_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvac_09"/>
        <field name="tax_dest_id" ref="tvati"/>
    </record>
    <record id="afptt_inverse_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvac_19"/>
        <field name="tax_dest_id" ref="tvati"/>
    </record>
    <record id="afptt_inverse_1s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvac_00_s"/>
        <field name="tax_dest_id" ref="tvati"/>
    </record>
    <record id="afptt_inverse_2s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvac_05_s"/>
        <field name="tax_dest_id" ref="tvati"/>
    </record>
    <record id="afptt_inverse_3s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvac_09_s"/>
        <field name="tax_dest_id" ref="tvati"/>
    </record>
    <record id="afptt_inverse_4s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvac_19_s"/>
        <field name="tax_dest_id" ref="tvati"/>
    </record>
    <!-- Purchases -->
    <record id="afptt_inverse_6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvad_00"/>
        <field name="tax_dest_id" ref="tvatip00"/>
    </record>
    <record id="afptt_inverse_7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvad_05"/>
        <field name="tax_dest_id" ref="tvatip05"/>
    </record>
    <record id="afptt_inverse_8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvad_09"/>
        <field name="tax_dest_id" ref="tvatip09"/>
    </record>
    <record id="afptt_inverse_11" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvad_19"/>
        <field name="tax_dest_id" ref="tvatip19"/>
    </record>
    <record id="afptt_inverse_6s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvad_00_s"/>
        <field name="tax_dest_id" ref="tvatip00"/>
    </record>
    <record id="afptt_inverse_7s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvad_05_s"/>
        <field name="tax_dest_id" ref="tvatip05"/>
    </record>
    <record id="afptt_inverse_8s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvad_09_s"/>
        <field name="tax_dest_id" ref="tvatip09"/>
    </record>
    <record id="afptt_inverse_11s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvad_19_s"/>
        <field name="tax_dest_id" ref="tvatip19"/>
    </record>

    <!-- Intracomunitar -->
    <!-- Sales -->
    <record id="afptt_intracom_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvac_00"/>
        <field name="tax_dest_id" ref="tvati_intrab"/>
    </record>
    <record id="afptt_intracom_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvac_05"/>
        <field name="tax_dest_id" ref="tvati_intrab"/>
    </record>
    <record id="afptt_intracom_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvac_09"/>
        <field name="tax_dest_id" ref="tvati_intrab"/>
    </record>
    <record id="afptt_intracom_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvac_19"/>
        <field name="tax_dest_id" ref="tvati_intrab"/>
    </record>
    <record id="afptt_intracoms_1s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvac_00_s"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracoms_2s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvac_05_s"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracoms_3s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvac_09_s"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracoms_4s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvac_19_s"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <!-- Purchases -->
    <record id="afptt_intracom_6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvad_00"/>
        <field name="tax_dest_id" ref="tvati_intrap0b"/>
    </record>
    <record id="afptt_intracom_7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvad_05"/>
        <field name="tax_dest_id" ref="tvati_intrap5b"/>
    </record>
    <record id="afptt_intracom_8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvad_09"/>
        <field name="tax_dest_id" ref="tvati_intrap9b"/>
    </record>
    <record id="afptt_intracom_9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvad_19"/>
        <field name="tax_dest_id" ref="tvati_intrap19b"/>
    </record>
    <record id="afptt_intracoms_6s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvad_00_s"/>
        <field name="tax_dest_id" ref="tvati_intrap0s"/>
    </record>
    <record id="afptt_intracoms_87s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvad_05_s"/>
        <field name="tax_dest_id" ref="tvati_intrap5s"/>
    </record>
    <record id="afptt_intracoms_8s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvad_09_s"/>
        <field name="tax_dest_id" ref="tvati_intrap9s"/>
    </record>
    <record id="afptt_intracoms_9s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvad_19_s"/>
        <field name="tax_dest_id" ref="tvati_intrap19s"/>
    </record>
    
    <!-- Intracomunitar Scutite-->
    <!-- Sales -->
    <record id="afptt_intracoms_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvac_00"/>
        <field name="tax_dest_id" ref="tvati_intrab"/>
    </record>
    <record id="afptt_intracoms_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvac_05"/>
        <field name="tax_dest_id" ref="tvati_intrab"/>
    </record>
    <record id="afptt_intracoms_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvac_09"/>
        <field name="tax_dest_id" ref="tvati_intrab"/>
    </record>
    <record id="afptt_intracoms_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvac_19"/>
        <field name="tax_dest_id" ref="tvati_intrab"/>
    </record>
    <record id="afptt_intracomss_1s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvac_00_s"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracomss_2s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvac_05_s"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracomss_3s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvac_09_s"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracomss_4s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvac_19_s"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <!-- Purchases -->
    <record id="afptt_intracoms_6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvad_00"/>
        <field name="tax_dest_id" ref="tvatiscdeda_intr"/>
    </record>
    <record id="afptt_intracoms_7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvad_05"/>
        <field name="tax_dest_id" ref="tvatiscdeda_intr"/>
    </record>
    <record id="afptt_intracoms_8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvad_09"/>
        <field name="tax_dest_id" ref="tvatiscdeda_intr"/>
    </record>
    <record id="afptt_intracoms_9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvad_19"/>
        <field name="tax_dest_id" ref="tvatiscdeda_intr"/>
    </record>
    <record id="afptt_intracomss_6s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvad_00_s"/>
        <field name="tax_dest_id" ref="tvatiscdeda_intr"/>
    </record>
    <record id="afptt_intracomss_7s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvad_05_s"/>
        <field name="tax_dest_id" ref="tvatiscdeda_intr"/>
    </record>
    <record id="afptt_intracomss_8s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvad_09_s"/>
        <field name="tax_dest_id" ref="tvatiscdeda_intr"/>
    </record>
    <record id="afptt_intracomss_9s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvad_19_s"/>
        <field name="tax_dest_id" ref="tvatiscdeda_intr"/>
    </record>

    <!-- Scutite - cu drept de deducere -->
    <!-- Sales -->
    <record id="afptt_intracomscded_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvac_00"/>
        <field name="tax_dest_id" ref="tvatiscded"/>
    </record>
    <record id="afptt_intracomscded_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvac_05"/>
        <field name="tax_dest_id" ref="tvatiscded"/>
    </record>
    <record id="afptt_intracomscded_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvac_09"/>
        <field name="tax_dest_id" ref="tvatiscded"/>
    </record>
    <record id="afptt_intracomscded_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvac_19"/>
        <field name="tax_dest_id" ref="tvatiscded"/>
    </record>
    <record id="afptt_intracomscded_1s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvac_00_s"/>
        <field name="tax_dest_id" ref="tvatiscded"/>
    </record>
    <record id="afptt_intracomscded_2s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvac_05_s"/>
        <field name="tax_dest_id" ref="tvatiscded"/>
    </record>
    <record id="afptt_intracomscded_3s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvac_09_s"/>
        <field name="tax_dest_id" ref="tvatiscded"/>
    </record>
    <record id="afptt_intracomscded_4s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvac_19_s"/>
        <field name="tax_dest_id" ref="tvatiscded"/>
    </record>
    <!-- Purchases -->
    <record id="afptt_intracomscded_6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvad_00"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <record id="afptt_intracomscded_7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvad_05"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <record id="afptt_intracomscded_8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvad_09"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <record id="afptt_intracomscded_9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvad_19"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <record id="afptt_intracomscded_6s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvad_00_s"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <record id="afptt_intracomscded_7s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvad_05_s"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <record id="afptt_intracomscded_8s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvad_09_s"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <record id="afptt_intracomscded_9s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_51"/>
        <field name="tax_src_id" ref="tvad_19_s"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <!-- Scutite - fara drept de deducere -->
    <!-- Sales -->
    <record id="afptt_intracomscnoded_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvac_00"/>
        <field name="tax_dest_id" ref="tvatiscnoded"/>
    </record>
    <record id="afptt_intracomscnoded_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvac_05"/>
        <field name="tax_dest_id" ref="tvatiscnoded"/>
    </record>
    <record id="afptt_intracomscnoded_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvac_09"/>
        <field name="tax_dest_id" ref="tvatiscnoded"/>
    </record>
    <record id="afptt_intracomscnoded_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvac_19"/>
        <field name="tax_dest_id" ref="tvatiscnoded"/>
    </record>
    <record id="afptt_intracomscnoded_1s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvac_00_s"/>
        <field name="tax_dest_id" ref="tvatiscnoded"/>
    </record>
    <record id="afptt_intracomscnoded_2s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvac_05_s"/>
        <field name="tax_dest_id" ref="tvatiscnoded"/>
    </record>
    <record id="afptt_intracomscnoded_3s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvac_09_s"/>
        <field name="tax_dest_id" ref="tvatiscnoded"/>
    </record>
    <record id="afptt_intracomscnoded_4s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvac_19_s"/>
        <field name="tax_dest_id" ref="tvatiscnoded"/>
    </record>
    <!-- Purchases -->
    <record id="afptt_intracomscnoded_6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvad_00"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <record id="afptt_intracomscnoded_7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvad_05"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <record id="afptt_intracomscnoded_8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvad_09"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <record id="afptt_intracomscnoded_9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvad_19"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <record id="afptt_intracomscnoded_6s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvad_00_s"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <record id="afptt_intracomscnoded_7s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvad_05_s"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <record id="afptt_intracomscnoded_8s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvad_09_s"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>
    <record id="afptt_intracomscnoded_9s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_52"/>
        <field name="tax_src_id" ref="tvad_19_s"/>
        <field name="tax_dest_id" ref="tvatiscdeda"/>
    </record>

    <!-- Taxare Inversa - Neimpozabile -->
    <!-- Sales -->
    <record id="afptt_intracomne_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvac_00"/>
        <field name="tax_dest_id" ref="tvatine"/>
    </record>
    <record id="afptt_intracomne_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvac_05"/>
        <field name="tax_dest_id" ref="tvatine"/>
    </record>
    <record id="afptt_intracomne_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvac_09"/>
        <field name="tax_dest_id" ref="tvatine"/>
    </record>
    <record id="afptt_intracomne_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvac_19"/>
        <field name="tax_dest_id" ref="tvatine"/>
    </record>
    <record id="afptt_intracomne_1s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvac_00_s"/>
        <field name="tax_dest_id" ref="tvatine"/>
    </record>
    <record id="afptt_intracomne_2s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvac_05_s"/>
        <field name="tax_dest_id" ref="tvatine"/>
    </record>
    <record id="afptt_intracomne_3s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvac_09_s"/>
        <field name="tax_dest_id" ref="tvatine"/>
    </record>
    <record id="afptt_intracomne_4s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvac_19_s"/>
        <field name="tax_dest_id" ref="tvatine"/>
    </record>
    <!-- Purchases -->
    <record id="afptt_intracomne_6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvad_00"/>
        <field name="tax_dest_id" ref="tvatinea"/>
    </record>
    <record id="afptt_intracomne_7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvad_05"/>
        <field name="tax_dest_id" ref="tvatinea"/>
    </record>
    <record id="afptt_intracomne_8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvad_09"/>
        <field name="tax_dest_id" ref="tvatinea"/>
    </record>
    <record id="afptt_intracomne_9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvad_19"/>
        <field name="tax_dest_id" ref="tvatinea"/>
    </record>
    <record id="afptt_intracomne_6s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvad_00_s"/>
        <field name="tax_dest_id" ref="tvatinea"/>
    </record>
    <record id="afptt_intracomne_7s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvad_05_s"/>
        <field name="tax_dest_id" ref="tvatinea"/>
    </record>
    <record id="afptt_intracomne_8s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvad_09_s"/>
        <field name="tax_dest_id" ref="tvatinea"/>
    </record>
    <record id="afptt_intracomne_9s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvad_19_s"/>
        <field name="tax_dest_id" ref="tvatinea"/>
    </record>

    <!-- Regim Extra-Comunitar -->
    <!-- Sales -->
    <record id="afptt_extracom_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvac_00"/>
        <field name="tax_dest_id" ref="tvati_extra"/>
    </record>
    <record id="afptt_extracom_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvac_05"/>
        <field name="tax_dest_id" ref="tvati_extra"/>
    </record>
    <record id="afptt_extracom_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvac_09"/>
        <field name="tax_dest_id" ref="tvati_extra"/>
    </record>
    <record id="afptt_extracom_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvac_19"/>
        <field name="tax_dest_id" ref="tvati_extra"/>
    </record>
    <record id="afptt_extracom_1s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvac_00_s"/>
        <field name="tax_dest_id" ref="tvati_extras"/>
    </record>
    <record id="afptt_extracom_2s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvac_05_s"/>
        <field name="tax_dest_id" ref="tvati_extras"/>
    </record>
    <record id="afptt_extracom_3s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvac_09_s"/>
        <field name="tax_dest_id" ref="tvati_extras"/>
    </record>
    <record id="afptt_extracom_4s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvac_19_s"/>
        <field name="tax_dest_id" ref="tvati_extras"/>
    </record>
    <!-- Purchases -->
    <record id="afptt_extracom_5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvad_00"/>
        <field name="tax_dest_id" ref="tvati_extrap0b"/>
    </record>
    <record id="afptt_extracom_6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvad_05"/>
        <field name="tax_dest_id" ref="tvati_extrap0b"/>
    </record>
    <record id="afptt_extracom_7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvad_09"/>
        <field name="tax_dest_id" ref="tvati_extrap0b"/>
    </record>
    <record id="afptt_extracom_8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvad_19"/>
        <field name="tax_dest_id" ref="tvati_extrap0b"/>
    </record>
    <record id="afptt_extracom_5s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvad_00_s"/>
        <field name="tax_dest_id" ref="tvati_extrap0s"/>
    </record>
    <record id="afptt_extracom_6s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvad_05_s"/>
        <field name="tax_dest_id" ref="tvati_extrap0s"/>
    </record>
    <record id="afptt_extracom_7s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvad_09_s"/>
        <field name="tax_dest_id" ref="tvati_extrap0s"/>
    </record>
    <record id="afptt_extracom_8s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_7"/>
        <field name="tax_src_id" ref="tvad_19_s"/>
        <field name="tax_dest_id" ref="tvati_extrap0s"/>
    </record>

    <!-- Tva la incasare -->
    <!-- Sales -->
    <record id="afptt_tvainc_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvac_00"/>
        <field name="tax_dest_id" ref="tvaic_00"/>
    </record>
    <record id="afptt_tvainc_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvac_05"/>
        <field name="tax_dest_id" ref="tvaic_05"/>
    </record>
    <record id="afptt_tvainc_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvac_09"/>
        <field name="tax_dest_id" ref="tvaic_09"/>
    </record>
    <record id="afptt_tvainc_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvac_19"/>
        <field name="tax_dest_id" ref="tvaic_19"/>
    </record>
    <record id="afptt_tvainc_1s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvac_00_s"/>
        <field name="tax_dest_id" ref="tvaic_00"/>
    </record>
    <record id="afptt_tvainc_2s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvac_05_s"/>
        <field name="tax_dest_id" ref="tvaic_05"/>
    </record>
    <record id="afptt_tvainc_3s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvac_09_s"/>
        <field name="tax_dest_id" ref="tvaic_09"/>
    </record>
    <record id="afptt_tvainc_4s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvac_19_s"/>
        <field name="tax_dest_id" ref="tvaic_19"/>
    </record>
    <!-- Purchases -->
    <record id="afptt_tvainc_6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvad_00"/>
        <field name="tax_dest_id" ref="tvaid_00"/>
    </record>
    <record id="afptt_tvainc_8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvad_05"/>
        <field name="tax_dest_id" ref="tvaid_05"/>
    </record>
    <record id="afptt_tvainc_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvad_09"/>
        <field name="tax_dest_id" ref="tvaid_09"/>
    </record>
    <record id="afptt_tvainc_11" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvad_19"/>
        <field name="tax_dest_id" ref="tvaid_19"/>
    </record>
    <record id="afptt_tvainc_6s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvad_00_s"/>
        <field name="tax_dest_id" ref="tvaid_00"/>
    </record>
    <record id="afptt_tvainc_8s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvad_05_s"/>
        <field name="tax_dest_id" ref="tvaid_05"/>
    </record>
    <record id="afptt_tvainc_10s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvad_09_s"/>
        <field name="tax_dest_id" ref="tvaid_09"/>
    </record>
    <record id="afptt_tvainc_11s" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_8"/>
        <field name="tax_src_id" ref="tvad_19_s"/>
        <field name="tax_dest_id" ref="tvaid_19"/>
    </record>
</odoo>

```

## File: data\account_reconcile_model_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="suppadvance_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="name">Avans Furnizor - Stocuri</field>
    </record>
    <record id="suppadvance_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="suppadvance_template"/>
        <field name="account_id" ref="pcg_4091"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Avans Furnizor - Stocuri</field>
    </record>
    <record id="suppadvance_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="name">Avans Furnizor - Servicii</field>
    </record>
    <record id="suppadvance_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="suppadvance_template"/>
        <field name="account_id" ref="pcg_4092"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Avans Furnizor - Servicii</field>
    </record>
    <record id="suppadvance_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="name">Avans Furnizor - Imobilizări Corporale</field>
    </record>
    <record id="suppadvance_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="suppadvance_template"/>
        <field name="account_id" ref="pcg_4093"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Avans Furnizor - Imobilizări Corporale</field>
    </record>
    <record id="suppadvance_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="name">Avans Furnizor - Imobilizări Necorporale</field>
    </record>
    <record id="suppadvance_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="suppadvance_template"/>
        <field name="account_id" ref="pcg_4094"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Avans Furnizor - Imobilizări Necorporale</field>
    </record>
    <record id="custadvance_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="name">Avans Client</field>
    </record>
    <record id="custadvance_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="custadvance_template"/>
        <field name="account_id" ref="pcg_419"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Avans Client</field>
    </record>
    <record id="bankcomm_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="name">Comision Bancar</field>
    </record>
    <record id="bankcomm_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="bankcomm_template"/>
        <field name="account_id" ref="pcg_627" />
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Comision Bancar</field>
    </record>
    <record id="interest_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="name">Dobândă</field>
    </record>
    <record id="interest_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="interest_template"/>
        <field name="account_id" ref="pcg_766"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Dobândă</field>
    </record>
    <record id="inttransfer_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="name">Virament Intern</field>
    </record>
    <record id="inttransfer_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="inttransfer_template"/>
        <field name="account_id" ref="pcg_581"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Virament Intern</field>
    </record>
    <record id="payroll_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="name">Salarii</field>
    </record>
    <record id="payroll_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="payroll_template"/>
        <field name="account_id" ref="pcg_421"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Salarii</field>
    </record>
    <record id="pendsettl_template" model="account.reconcile.model.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="name">Operațiuni în curs de clarificare</field>
    </record>
    <record id="pendsettl_line_template" model="account.reconcile.model.line.template">
        <field name="model_id" ref="pendsettl_template"/>
        <field name="account_id" ref="pcg_473"/>
        <field name="amount_type">percentage</field>
        <field name="amount_string">100</field>
        <field name="label">Operațiuni în curs de clarificare</field>
    </record>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tvac_00" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">4</field>
        <field name="description">TVA colectat 0% Bunuri</field>
        <field name="name">TVA colectat 0% Bunuri</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd14_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'plus_report_expression_ids': [],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd14_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'minus_report_expression_ids': [],
            }),
        ]"/>
    </record>
    <record id="tvac_05" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">3</field>
        <field name="name">TVA colectat 5% Bunuri</field>
        <field name="description">TVA colectat 5% Bunuri</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd11_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd11_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd11_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd11_tag')],
            }),
        ]"/>
    </record>
    <record id="tvac_09" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">2</field>
        <field name="name">TVA colectat 9% Bunuri</field>
        <field name="description">TVA colectat 9% Bunuri</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd10_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd10_tag')],
            }),
        ]"/>
    </record>
    <record id="tvac_19" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">1</field>
        <field name="name">TVA colectat 19% Bunuri</field>
        <field name="description">TVA colectat 19% Bunuri</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd9_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd9_tag')],
            }),
        ]"/>
    </record>
    <record id="tvac_00_s" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">4</field>
        <field name="description">TVA colectat 0% Servicii</field>
        <field name="name">TVA colectat 0% Servicii</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd14_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'plus_report_expression_ids': [],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd14_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'minus_report_expression_ids': [],
            }),
        ]"/>
    </record>
    <record id="tvac_05_s" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">3</field>
        <field name="name">TVA colectat 5% Servicii</field>
        <field name="description">TVA colectat 5% Servicii</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd11_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd11_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd11_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd11_tag')],
            }),
        ]"/>
    </record>
    <record id="tvac_09_s" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">2</field>
        <field name="name">TVA colectat 9% Servicii</field>
        <field name="description">TVA colectat 9% Servicii</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd10_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd10_tag')],
            }),
        ]"/>
    </record>
    <record id="tvac_19_s" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">1</field>
        <field name="name">TVA colectat 19% Servicii</field>
        <field name="description">TVA colectat 19% Servicii</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd9_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd9_tag')],
            }),
        ]"/>
    </record>
    <record id="tvad_00" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">43</field>
        <field name="name">TVA deductibil 0% Bunuri</field>
        <field name="description">TVA deductibil 0% Bunuri</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'plus_report_expression_ids': [],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'minus_report_expression_ids': [],
            }),
        ]"/>
    </record>
    <record id="tvad_05" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">42</field>
        <field name="name">TVA deductibil 5% Bunuri</field>
        <field name="description">TVA deductibil 5% Bunuri</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd261_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd261_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd261_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd261_tag')],
            }),
        ]"/>
    </record>
    <record id="tvad_09" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">41</field>
        <field name="name">TVA deductibil 9% Bunuri</field>
        <field name="description">TVA deductibil 9% Bunuri</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd251_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd251_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd251_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd251_tag')],
            }),
        ]"/>
    </record>
    <record id="tvad_19" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">40</field>
        <field name="name">TVA deductibil 19% Bunuri</field>
        <field name="description">TVA deductibil 19% Bunuri</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd241_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd241_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd241_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd241_tag')],
            }),
        ]"/>
    </record>
    <record id="tvad_00_s" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">43</field>
        <field name="name">TVA deductibil 0% Servicii</field>
        <field name="description">TVA deductibil 0% Servicii</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'plus_report_expression_ids': [],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'minus_report_expression_ids': [],
            }),
        ]"/>
    </record>
    <record id="tvad_05_s" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">42</field>
        <field name="name">TVA deductibil 5% Servicii</field>
        <field name="description">TVA deductibil 5% Servicii</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd261_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd261_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd261_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd261_tag')],
            }),
        ]"/>
    </record>
    <record id="tvad_09_s" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">41</field>
        <field name="name">TVA deductibil 9% Servicii</field>
        <field name="description">TVA deductibil 9% Servicii</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd251_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd251_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd251_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd251_tag')],
            }),
        ]"/>
    </record>
    <record id="tvad_19_s" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">40</field>
        <field name="name">TVA deductibil 19% Servicii</field>
        <field name="description">TVA deductibil 19% Servicii</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd241_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd241_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd241_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd241_tag')],
            }),
        ]"/>
    </record>
    <record id="tvaic_00" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">13</field>
        <field name="name">TVA la Incasare - colectat 0%</field>
        <field name="description">TVA 0%</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd14_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'plus_report_expression_ids': [],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd14_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'minus_report_expression_ids': [],
            }),
        ]"/>
    </record>
    <record id="tvaic_05" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">12</field>
        <field name="name">TVA la Incasare - colectat 5%</field>
        <field name="description">TVA colectat 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44281"/>
        <field name="tax_group_id" ref="tax_group_tva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd11_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd11_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd11_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd11_tag')],
            }),
        ]"/>
    </record>
    <record id="tvaic_09" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">11</field>
        <field name="name">TVA la Incasare - colectat 9%</field>
        <field name="description">TVA colectat 9%</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44281"/>
        <field name="tax_group_id" ref="tax_group_tva_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd10_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd10_tag')],
            }),
        ]"/>
    </record>
    <record id="tvaic_19" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">10</field>
        <field name="name">TVA la Incasare - colectat 19%</field>
        <field name="description">TVA colectat 19%</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44281"/>
        <field name="tax_group_id" ref="tax_group_tva_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd9_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd9_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd9_tag')],
            }),
        ]"/>
    </record>
    <record id="tvaid_00" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">53</field>
        <field name="name">TVA la Incasare - deductibil 0%</field>
        <field name="description">TVA deductibil 0%</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'plus_report_expression_ids': [],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'minus_report_expression_ids': [],
            }),
        ]"/>
    </record>
    <record id="tvaid_05" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">52</field>
        <field name="name">TVA la Incasare - deductibil 5%</field>
        <field name="description">TVA deductibil 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44282"/>
        <field name="tax_group_id" ref="tax_group_tva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd261_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd261_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd261_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd261_tag')],
            }),
        ]"/>
    </record>
    <record id="tvaid_09" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">51</field>
        <field name="name">TVA la Incasare - deductibil 9%</field>
        <field name="description">TVA deductibil 9%</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44282"/>
        <field name="tax_group_id" ref="tax_group_tva_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd251_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd251_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd251_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd251_tag')],
            }),
        ]"/>
    </record>
    <record id="tvaid_19" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">50</field>
        <field name="name">TVA la Incasare - deductibil 19%</field>
        <field name="description">TVA deductibil 19%</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_exigibility">on_payment</field>
        <field name="cash_basis_transition_account_id" ref="pcg_44282"/>
        <field name="tax_group_id" ref="tax_group_tva_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd241_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd241_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd241_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd241_tag')],
            }),
        ]"/>
    </record>
    <record id="tvati" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">20</field>
        <field name="name">TVA Taxare Inversa</field>
        <field name="description">TVA Taxare Inversa</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_ti"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd13_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd13_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tvatip00" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">63</field>
        <field name="name">TVA Taxare Inversa 0%</field>
        <field name="description">TVA Taxare Inversa 0%</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_ti"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'plus_report_expression_ids': [],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'minus_report_expression_ids': [],
            }),
        ]"/>
    </record>
    <record id="tvatip05" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">62</field>
        <field name="name">TVA Taxare Inversa 5%</field>
        <field name="description">TVA Taxare Inversa 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_ti"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd123_tag'), ref('account_tax_report_ro_baza_rd273_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd123_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd273_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd123_tag'), ref('account_tax_report_ro_baza_rd273_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd123_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd273_tag')],
            }),
        ]"/>
    </record>
    <record id="tvatip09" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">61</field>
        <field name="name">TVA Taxare Inversa 9%</field>
        <field name="description">TVA Taxare Inversa 9%</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_ti"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd122_tag'), ref('account_tax_report_ro_baza_rd272_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd122_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd272_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd122_tag'), ref('account_tax_report_ro_baza_rd272_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd122_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd272_tag')],
            }),
        ]"/>
    </record>
    <record id="tvatip19" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">60</field>
        <field name="name">TVA Taxare Inversa 19%</field>
        <field name="description">TVA Taxare Inversa 19%</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_ti"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd121_tag'), ref('account_tax_report_ro_baza_rd271_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd121_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd271_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd121_tag'), ref('account_tax_report_ro_baza_rd271_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd121_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd271_tag')],
            }),
        ]"/>
    </record>
    <record id="tvatiscded" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">31</field>
        <field name="name">TVA Taxare Scutita Livrari cu drept de deducere</field>
        <field name="description">Scutit-L1</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_scutit"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd14_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd14_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tvatiscnoded" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">32</field>
        <field name="name">TVA Taxare Scutita Livrari fara drept de deducere</field>
        <field name="description">Scutit-L2</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_scutit"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd15_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd15_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tvatiscdeda" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">61</field>
        <field name="name">TVA Taxare Scutita Achizitii</field>
        <field name="description">Scutit-A</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_scutit"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tvatiscdeda_intr" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">61</field>
        <field name="name">TVA Taxare Scutita Achizitii Intracomunitare</field>
        <field name="description">Scutit-AI</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_scutit"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag'), ref('account_tax_report_ro_baza_rd301_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag'), ref('account_tax_report_ro_baza_rd301_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tvatine" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">33</field>
        <field name="name">TVA Taxare Neimpozabila Livrari</field>
        <field name="description">Neimpozabil-L</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_neimp"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd15_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd15_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tvatinea" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">62</field>
        <field name="name">TVA Taxare Neimpozabila Achizitii</field>
        <field name="description">Neimpozabil-A</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_neimp"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tvati_intrab" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">34</field>
        <field name="name">TVA Intracomunitara Livrari Bunuri</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd1_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd1_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tvati_intras" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">34</field>
        <field name="name">TVA Intracomunitara Livrari Servicii</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd3_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd3_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tvati_extra" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">35</field>
        <field name="name">TVA Export Bunuri</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_scutit"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd14_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd14_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tvati_extras" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">36</field>
        <field name="name">TVA Export Servicii</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_scutit"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd14_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd14_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="tvati_intrap0b" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">66</field>
        <field name="name">TVA Intracomunitara Achizitii 0% - Bunuri</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0_eu"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'plus_report_expression_ids': [],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'minus_report_expression_ids': [],
            }),
        ]"/>
    </record>
    <record id="tvati_intrap5b" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">65</field>
        <field name="name">TVA Intracomunitara Achizitii 5% - Bunuri</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_5_eu"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd5_tag'), ref('account_tax_report_ro_baza_rd51_tag'), ref('account_tax_report_ro_baza_rd20_tag'), ref('account_tax_report_ro_baza_rd201_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd5_tag'), ref('account_tax_report_ro_tva_rd51_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd20_tag'), ref('account_tax_report_ro_tva_rd201_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd5_tag'), ref('account_tax_report_ro_baza_rd51_tag'), ref('account_tax_report_ro_baza_rd20_tag'), ref('account_tax_report_ro_baza_rd201_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd5_tag'), ref('account_tax_report_ro_tva_rd51_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd20_tag'), ref('account_tax_report_ro_tva_rd201_tag')],
            }),
        ]"/>
    </record>
    <record id="tvati_intrap9b" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">64</field>
        <field name="name">TVA Intracomunitara Achizitii 9% - Bunuri</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_9_eu"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd5_tag'), ref('account_tax_report_ro_baza_rd51_tag'), ref('account_tax_report_ro_baza_rd20_tag'), ref('account_tax_report_ro_baza_rd201_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd5_tag'), ref('account_tax_report_ro_tva_rd51_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd20_tag'), ref('account_tax_report_ro_tva_rd201_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd5_tag'), ref('account_tax_report_ro_baza_rd51_tag'), ref('account_tax_report_ro_baza_rd20_tag'), ref('account_tax_report_ro_baza_rd201_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd5_tag'), ref('account_tax_report_ro_tva_rd51_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd20_tag'), ref('account_tax_report_ro_tva_rd201_tag')],
            }),
        ]"/>
    </record>
    <record id="tvati_intrap19b" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">63</field>
        <field name="name">TVA Intracomunitara Achizitii 19% - Bunuri</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_19_eu"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd5_tag'), ref('account_tax_report_ro_baza_rd51_tag'), ref('account_tax_report_ro_baza_rd20_tag'), ref('account_tax_report_ro_baza_rd201_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd5_tag'), ref('account_tax_report_ro_tva_rd51_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd20_tag'), ref('account_tax_report_ro_tva_rd201_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd5_tag'), ref('account_tax_report_ro_baza_rd51_tag'), ref('account_tax_report_ro_baza_rd20_tag'), ref('account_tax_report_ro_baza_rd201_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd5_tag'), ref('account_tax_report_ro_tva_rd51_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd20_tag'), ref('account_tax_report_ro_tva_rd201_tag')],
            }),
        ]"/>
    </record>
    <record id="tvati_intrap0s" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">70</field>
        <field name="name">TVA Intracomunitara Achizitii 0% - Servicii</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0_eu"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'plus_report_expression_ids': [],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd30_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'minus_report_expression_ids': [],
            }),
        ]"/>
    </record>
    <record id="tvati_intrap5s" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">69</field>
        <field name="name">TVA Intracomunitara Achizitii 5% - Servicii</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_5_eu"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd7_tag'), ref('account_tax_report_ro_baza_rd71_tag'), ref('account_tax_report_ro_baza_rd22_tag'), ref('account_tax_report_ro_baza_rd221_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd7_tag'), ref('account_tax_report_ro_tva_rd71_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd22_tag'), ref('account_tax_report_ro_tva_rd221_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd7_tag'), ref('account_tax_report_ro_baza_rd71_tag'), ref('account_tax_report_ro_baza_rd22_tag'), ref('account_tax_report_ro_baza_rd221_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd7_tag'), ref('account_tax_report_ro_tva_rd71_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd22_tag'), ref('account_tax_report_ro_tva_rd221_tag')],
            }),
        ]"/>
    </record>
    <record id="tvati_intrap9s" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">68</field>
        <field name="name">TVA Intracomunitara Achizitii 9% - Servicii</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_9_eu"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd7_tag'), ref('account_tax_report_ro_baza_rd71_tag'), ref('account_tax_report_ro_baza_rd22_tag'), ref('account_tax_report_ro_baza_rd221_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd7_tag'), ref('account_tax_report_ro_tva_rd71_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd22_tag'), ref('account_tax_report_ro_tva_rd221_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd7_tag'), ref('account_tax_report_ro_baza_rd71_tag'), ref('account_tax_report_ro_baza_rd22_tag'), ref('account_tax_report_ro_baza_rd221_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd7_tag'), ref('account_tax_report_ro_tva_rd71_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd22_tag'), ref('account_tax_report_ro_tva_rd221_tag')],
            }),
        ]"/>
    </record>
    <record id="tvati_intrap19s" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">67</field>
        <field name="name">TVA Intracomunitara Achizitii 19% - Servicii</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_19_eu"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd7_tag'), ref('account_tax_report_ro_baza_rd71_tag'), ref('account_tax_report_ro_baza_rd22_tag'), ref('account_tax_report_ro_baza_rd221_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd7_tag'), ref('account_tax_report_ro_tva_rd71_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd22_tag'), ref('account_tax_report_ro_tva_rd221_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd7_tag'), ref('account_tax_report_ro_baza_rd71_tag'), ref('account_tax_report_ro_baza_rd22_tag'), ref('account_tax_report_ro_baza_rd221_tag')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd7_tag'), ref('account_tax_report_ro_tva_rd71_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd22_tag'), ref('account_tax_report_ro_tva_rd221_tag')],
            }),
        ]"/>
    </record>
    <!-- Import -->
    <record id="tvati_extrap0b" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">66</field>
        <field name="name">TVA Import Bunuri</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_scutit"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd7_tag'), ref('account_tax_report_ro_baza_rd71_tag'), ref('account_tax_report_ro_baza_rd22_tag'), ref('account_tax_report_ro_baza_rd221_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'plus_report_expression_ids': [],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd7_tag'), ref('account_tax_report_ro_baza_rd71_tag'), ref('account_tax_report_ro_baza_rd22_tag'), ref('account_tax_report_ro_baza_rd221_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'minus_report_expression_ids': [],
            }),
        ]"/>
    </record>
    <record id="tvati_extrap0s" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">70</field>
        <field name="name">TVA Import Servicii</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_scutit"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd7_tag'), ref('account_tax_report_ro_baza_rd71_tag'), ref('account_tax_report_ro_baza_rd22_tag'), ref('account_tax_report_ro_baza_rd221_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'plus_report_expression_ids': [],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd7_tag'), ref('account_tax_report_ro_baza_rd71_tag'), ref('account_tax_report_ro_baza_rd22_tag'), ref('account_tax_report_ro_baza_rd221_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'minus_report_expression_ids': [],
            }),
        ]"/>
    </record>
    <record id="tva_ned_5_50" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">72</field>
        <field name="name">TVA 5% Nedeductibil 50%</field>
        <field name="description">TVA 5% Nedeductibil 50%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_ned"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd262_tag')],
            }),
            (0,0, {
                'factor_percent': -50,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd262_tag')],
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('pcg_6352'),
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd262_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd262_tag')],
            }),
            (0,0, {
                'factor_percent': -50,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd262_tag')],
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('pcg_6352'),
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd262_tag')],
            }),
        ]"/>
    </record>
    <record id="tva_ned_9_50" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">71</field>
        <field name="name">TVA 9% Nedeductibil 50%</field>
        <field name="description">TVA 9% Nedeductibil 50%</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_ned"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd252_tag')],
            }),
            (0,0, {
                'factor_percent': -50,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd252_tag')],
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('pcg_6352'),
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd252_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd252_tag')],
            }),
            (0,0, {
                'factor_percent': -50,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd252_tag')],
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('pcg_6352'),
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd252_tag')],
            }),
        ]"/>
    </record>
    <record id="tva_ned_19_50" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">70</field>
        <field name="name">TVA 19% Nedeductibil 50%</field>
        <field name="description">TVA 19% Nedeductibil 50%</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_ned"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_tax_report_ro_baza_rd242_tag')],
            }),
            (0,0, {
                'factor_percent': -50,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd242_tag')],
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('pcg_6352'),
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd242_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_tax_report_ro_baza_rd242_tag')],
            }),
            (0,0, {
                'factor_percent': -50,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_expression_ids': [ref('account_tax_report_ro_tva_rd242_tag')],
            }),
            (0,0, {
                'factor_percent': 50,
                'repartition_type': 'tax',
                'account_id': ref('pcg_6352'),
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_expression_ids': [ref('account_tax_report_ro_tva_rd242_tag')],
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
            <field name="country_id" ref="base.ro"/>
        </record>
        <record id="tax_group_tva_5" model="account.tax.group">
            <field name="name">TVA 5%</field>
            <field name="country_id" ref="base.ro"/>
        </record>
        <record id="tax_group_tva_9" model="account.tax.group">
            <field name="name">TVA 9%</field>
            <field name="country_id" ref="base.ro"/>
        </record>
        <record id="tax_group_tva_19" model="account.tax.group">
            <field name="name">TVA 19%</field>
            <field name="country_id" ref="base.ro"/>
        </record>
        <record id="tax_group_tva_20" model="account.tax.group">
            <field name="name">TVA 20%</field>
            <field name="country_id" ref="base.ro"/>
        </record>
        <record id="tax_group_tva_24" model="account.tax.group">
            <field name="name">TVA 24%</field>
            <field name="country_id" ref="base.ro"/>
        </record>
        <record id="tax_group_tva_0_eu" model="account.tax.group">
            <field name="name">TVA 0% EU</field>
            <field name="country_id" ref="base.ro"/>
        </record>
        <record id="tax_group_tva_5_eu" model="account.tax.group">
            <field name="name">TVA 5% EU</field>
            <field name="country_id" ref="base.ro"/>
        </record>
        <record id="tax_group_tva_9_eu" model="account.tax.group">
            <field name="name">TVA 9% EU</field>
            <field name="country_id" ref="base.ro"/>
        </record>
        <record id="tax_group_tva_19_eu" model="account.tax.group">
            <field name="name">TVA 19% EU</field>
            <field name="country_id" ref="base.ro"/>
        </record>
        <record id="tax_group_tva_ned" model="account.tax.group">
            <field name="name">TVA Nedeductibil</field>
            <field name="country_id" ref="base.ro"/>
        </record>
        <record id="tax_group_tva_ti" model="account.tax.group">
            <field name="name">TVA Taxare Inversa</field>
            <field name="country_id" ref="base.ro"/>
        </record>
        <record id="tax_group_tva_scutit" model="account.tax.group">
            <field name="name">TVA Scutit</field>
            <field name="country_id" ref="base.ro"/>
        </record>
        <record id="tax_group_tva_neimp" model="account.tax.group">
            <field name="name">TVA Neimpozabil</field>
            <field name="country_id" ref="base.ro"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report" model="account.report">
        <field name="name">Romanian Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.ro"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_ro_baza_intracom_eu" model="account.report.line">
                <field name="name">Baza COMERŢ INTRACOMUNITAR ŞI ÎN AFARA UE</field>
                <field name="code">tax_ro_baza_intracom_eu</field>
                <field name="aggregation_formula">tax_ro_baza_rd1.balance + tax_ro_baza_rd2.balance + tax_ro_baza_rd3.balance + tax_ro_baza_rd4.balance + tax_ro_baza_rd5.balance + tax_ro_baza_rd6.balance + tax_ro_baza_rd7.balance + tax_ro_baza_rd8.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_baza_rd1" model="account.report.line">
                        <field name="name">1 - BAZA - Livrări intracomunitare de bunuri, scutite conform art. 294 alin.(2) lit.a) şid) din Codul fiscal</field>
                        <field name="code">tax_ro_baza_rd1</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">01 - BAZA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd2" model="account.report.line">
                        <field name="name">2 - BAZA - Regularizări livrări intracomunitare scutite conform art. 294 alin.(2) lit.a) şi d) din Codul fiscal</field>
                        <field name="code">tax_ro_baza_rd2</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">02 - BAZA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd3" model="account.report.line">
                        <field name="name">3 - BAZA - Livrări de bunuri/prestări de servicii pentru care locul livrării/prestării este în afara României, precum şi livrări intracom. de bunuri, scut. conf. art. 294 alin.(2) lit.b) şi c) din CF, din care: </field>
                        <field name="code">tax_ro_baza_rd3</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">03 - BAZA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd31" model="account.report.line">
                                <field name="name">3.1 - BAZA - Prestări de servicii intracomunitare care nu beneficiază de scutire in statul membru in care taxa este datorată</field>
                                <field name="code">tax_ro_baza_rd31</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd31_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">03_1 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd4" model="account.report.line">
                        <field name="name">4 - BAZA - Regularizări privind prestările de servicii intracomunitare care nu beneficiază de scutire in statul membru in care taxa este datorată</field>
                        <field name="code">tax_ro_baza_rd4</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd4_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">04 - BAZA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd5" model="account.report.line">
                        <field name="name">5 - BAZA - Achizitii intracomunitare de bunuri pentru care cumpărătorul este obligat la plata TVA (taxare inversă), din care:</field>
                        <field name="code">tax_ro_baza_rd5</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">05 - BAZA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd51" model="account.report.line">
                                <field name="name">5.1 - BAZA - Achiziţii intracom. pentru care cumpărătorul este obligat la plata TVA (TI), iar furnizorul este înregistrat în scopuri de TVA în statul membru din care a avut loc livrarea intracom.</field>
                                <field name="code">tax_ro_baza_rd51</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd51_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">05_1 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd6" model="account.report.line">
                        <field name="name">6 - BAZA - Regularizări privind achiziţiile intracomunitare de bunuri pentru care cumpărătorul este obligat la plata TVA(taxare inversă)</field>
                        <field name="code">tax_ro_baza_rd6</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd6_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">06 - BAZA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd7" model="account.report.line">
                        <field name="name">7 - BAZA - Achizitii de bunuri, altele decat cele de la rd.5 şi 6 si achizitii de servicii pentru care beneficiarul din Romania este obligat la plata TVA (taxare inversa) din care:</field>
                        <field name="code">tax_ro_baza_rd7</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd7_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">07 - BAZA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd71" model="account.report.line">
                                <field name="name">7.1 - BAZA - Achizitii de servicii intracomunitare pentru care beneficiarul este obligat la plata TVA (taxare inversa)</field>
                                <field name="code">tax_ro_baza_rd71</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd71_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">07_1 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd8" model="account.report.line">
                        <field name="name">8 - BAZA - Regularizari privind achizitii de servicii intracomunitare pentru care beneficiarul este obligat la plata TVA (taxare inversa)</field>
                        <field name="code">tax_ro_baza_rd8</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd8_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">08 - BAZA</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_intracom_eu" model="account.report.line">
                <field name="name">TVA COMERŢ INTRACOMUNITAR ŞI ÎN AFARA UE</field>
                <field name="code">tax_ro_tva_intracom_eu</field>
                <field name="aggregation_formula">tax_ro_tva_rd5.balance + tax_ro_tva_rd6.balance + tax_ro_tva_rd7.balance + tax_ro_tva_rd8.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_tva_rd5" model="account.report.line">
                        <field name="name">5 - TVA - Achizitii intracomunitare de bunuri pentru care cumpărătorul este obligat la plata TVA (taxare inversă), din care:</field>
                        <field name="code">tax_ro_tva_rd5</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">05 - TVA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd51" model="account.report.line">
                                <field name="name">5.1 - TVA - Achiziţii intracom. pentru care cumpărătorul este obligat la plata TVA (TI), iar furnizorul este înregistrat în scopuri de TVA în statul membru din care a avut loc livrarea intracom.</field>
                                <field name="code">tax_ro_tva_rd51</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd51_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">05_1 - TVA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd6" model="account.report.line">
                        <field name="name">6 - TVA - Regularizări privind achiziţiile intracomunitare de bunuri pentru care cumpărătorul este obligat la plata TVA(taxare inversă)</field>
                        <field name="code">tax_ro_tva_rd6</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd6_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">06 - TVA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd7" model="account.report.line">
                        <field name="name">7 - TVA - Achizitii de bunuri, altele decat cele de la rd.5 şi 6 si achizitii de servicii pentru care beneficiarul din Romania este obligat la plata TVA (taxare inversa) din care:</field>
                        <field name="code">tax_ro_tva_rd7</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd7_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">07 - TVA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd71" model="account.report.line">
                                <field name="name">7.1 - TVA - Achizitii de servicii intracomunitare pentru care beneficiarul este obligat la plata TVA (taxare inversa)</field>
                                <field name="code">tax_ro_tva_rd71</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd71_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">07_1 - TVA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd8" model="account.report.line">
                        <field name="name">8 - TVA - Regularizari privind achizitii de servicii intracomunitare pentru care beneficiarul este obligat la plata TVA (taxare inversa)</field>
                        <field name="code">tax_ro_tva_rd8</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd8_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">08 - TVA</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_baza_livrari" model="account.report.line">
                <field name="name">BAZA LIVRĂRI DE BUNURI/ PRESTĂRI DE SERVICII ÎN INTERIORUL ŢĂRII ŞI EXPORTURI</field>
                <field name="code">tax_ro_baza_livrari</field>
                <field name="aggregation_formula">tax_ro_baza_rd9.balance + tax_ro_baza_rd10.balance + tax_ro_baza_rd11.balance + tax_ro_baza_rd12.balance + tax_ro_baza_rd13.balance + tax_ro_baza_rd14.balance + tax_ro_baza_rd15.balance + tax_ro_baza_rd16.balance + tax_ro_baza_rd17.balance + tax_ro_baza_rd18.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_baza_rd9" model="account.report.line">
                        <field name="name">9 - BAZA - Livrări de bunuri şi prestări de servicii taxabile cu cota 19%</field>
                        <field name="code">tax_ro_baza_rd9</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd9_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">09 - BAZA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd91" model="account.report.line">
                                <field name="name">9.1 - BAZA - Livrări de bunuri şi prestări de servicii taxabile cu cota 19%</field>
                                <field name="code">tax_ro_baza_rd91</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd91_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">09_1 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd92" model="account.report.line">
                                <field name="name">9.2 - BAZA - Achizitii de bunuri şi prestări de servicii nedeductibile 50% taxabile cu cota 19%</field>
                                <field name="code">tax_ro_baza_rd92</field>
                                <field name="aggregation_formula">0.5 * tax_ro_baza_rd242.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd10" model="account.report.line">
                        <field name="name">10 - BAZA - Livrări de bunuri şi prestări de servicii taxabile cu cota 9%</field>
                        <field name="code">tax_ro_baza_rd10</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10 - BAZA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd101" model="account.report.line">
                                <field name="name">10.1 - BAZA - Livrări de bunuri şi prestări de servicii taxabile cu cota 9%</field>
                                <field name="code">tax_ro_baza_rd101</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd101_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">10_1 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd102" model="account.report.line">
                                <field name="name">10_2 - BAZA - Achizitii de bunuri şi prestări de servicii nedeductibile 50% taxabile cu cota 9%</field>
                                <field name="code">tax_ro_baza_rd102</field>
                                <field name="aggregation_formula">0.5 * tax_ro_baza_rd252.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd11" model="account.report.line">
                        <field name="name">11 - BAZA - Livrări de bunuri taxabile cu cota 5%</field>
                        <field name="code">tax_ro_baza_rd11</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11 - BAZA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd111" model="account.report.line">
                                <field name="name">11.1 - BAZA - Livrări de bunuri şi prestări de servicii taxabile cu cota 5%</field>
                                <field name="code">tax_ro_baza_rd111</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd111_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">11_1 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd112" model="account.report.line">
                                <field name="name">11.2 - BAZA - Achizitii de bunuri şi prestări de servicii nedeductibile 50% taxabile cu cota 5%</field>
                                <field name="code">tax_ro_baza_rd112</field>
                                <field name="aggregation_formula">0.5 * tax_ro_baza_rd262.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd12" model="account.report.line">
                        <field name="name">12 - BAZA - Achiziţii de bunuri şi servicii supuse măsurilor de simplificare pentru care beneficiarul este obligat la plata TVA (taxare inversă) , din care:</field>
                        <field name="code">tax_ro_baza_rd12</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">12 - BAZA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd121" model="account.report.line">
                                <field name="name">12.1 - BAZA - Achizitii de bunuri si servicii, taxabile cu cota 19%</field>
                                <field name="code">tax_ro_baza_rd121</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd121_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">12_1 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd122" model="account.report.line">
                                <field name="name">12.2 - BAZA - Achizitii de bunuri si servicii, taxabile cu cota 9%</field>
                                <field name="code">tax_ro_baza_rd122</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd122_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">12_2 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd123" model="account.report.line">
                                <field name="name">12.3 - BAZA - Achizitii de bunuri si servicii, taxabile cu cota 5%</field>
                                <field name="code">tax_ro_baza_rd123</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd123_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">12_3 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd13" model="account.report.line">
                        <field name="name">13 - BAZA - Livrări de bunuri şi prestări de servicii supuse masurilor de simplificare (taxare inversa)</field>
                        <field name="code">tax_ro_baza_rd13</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd13_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">13 - BAZA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd14" model="account.report.line">
                        <field name="name">14 - BAZA - Livrări de bunuri şi prestări de servicii scutite cu drept de deducere, altele decat cele de la rd. 1-3</field>
                        <field name="code">tax_ro_baza_rd14</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd14_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">14 - BAZA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd15" model="account.report.line">
                        <field name="name">15 - BAZA - Livrări de bunuri şi prestări de servicii scutite fără drept de deducere</field>
                        <field name="code">tax_ro_baza_rd15</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd15_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">15 - BAZA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd16" model="account.report.line">
                        <field name="name">16 - BAZA - Regularizări taxă colectată</field>
                        <field name="code">tax_ro_baza_rd16</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd16_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">16 - BAZA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd17" model="account.report.line">
                        <field name="name">17 - BAZA - Prestări de servicii intracomunitare conform art.278 alin.(8) din Codul fiscal pentru care locul prestării este în România</field>
                        <field name="code">tax_ro_baza_rd17</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd17_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">17 - BAZA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd18" model="account.report.line">
                        <field name="name">18 - BAZA - Regularizări privind prestări de servicii intracomunitare conform art.278 alin.(8) din Codul fiscal pentru care locul prestării este în România</field>
                        <field name="code">tax_ro_baza_rd18</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd18_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">18 - BAZA</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_livrari" model="account.report.line">
                <field name="name">TVA LIVRĂRI DE BUNURI/ PRESTĂRI DE SERVICII ÎN INTERIORUL ŢĂRII ŞI EXPORTURI</field>
                <field name="code">tax_ro_tva_livrari</field>
                <field name="aggregation_formula">tax_ro_tva_rd9.balance + tax_ro_tva_rd10.balance + tax_ro_tva_rd11.balance + tax_ro_tva_rd12.balance + tax_ro_tva_rd16.balance + tax_ro_tva_rd17.balance + tax_ro_tva_rd18.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_tva_rd9" model="account.report.line">
                        <field name="name">9 - TVA - Livrări de bunuri şi prestări de servicii taxabile cu cota 19%</field>
                        <field name="code">tax_ro_tva_rd9</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd9_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">09 - TVA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd91" model="account.report.line">
                                <field name="name">9.1 - TVA - Livrări de bunuri şi prestări de servicii taxabile cu cota 19%</field>
                                <field name="code">tax_ro_tva_rd91</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd91_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">09_1 - TVA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd92" model="account.report.line">
                                <field name="name">9.2 - TVA - Achizitii de bunuri şi prestări de servicii nedeductibile 50% taxabile cu cota 19%</field>
                                <field name="code">tax_ro_tva_rd92</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd92_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">09_2 - TVA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd10" model="account.report.line">
                        <field name="name">10 - TVA - Livrări de bunuri şi prestări de servicii taxabile cu cota 9%</field>
                        <field name="code">tax_ro_tva_rd10</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">10 - TVA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd101" model="account.report.line">
                                <field name="name">10.1 - TVA - Livrări de bunuri şi prestări de servicii taxabile cu cota 9%</field>
                                <field name="code">tax_ro_tva_rd101</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd101_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">10_1 - TVA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd102" model="account.report.line">
                                <field name="name">10.2 - TVA - Achizitii de bunuri şi prestări de servicii nedeductibile 50% taxabile cu cota 9%</field>
                                <field name="code">tax_ro_tva_rd102</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd102_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">10_2 - TVA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd11" model="account.report.line">
                        <field name="name">11 - TVA - Livrări de bunuri taxabile cu cota 5%</field>
                        <field name="code">tax_ro_tva_rd11</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">11 - TVA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd111" model="account.report.line">
                                <field name="name">11.1 - TVA - Livrări de bunuri şi prestări de servicii taxabile cu cota 5%</field>
                                <field name="code">tax_ro_tva_rd111</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd111_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">11_1 - TVA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd112" model="account.report.line">
                                <field name="name">11.2 - TVA - Achizitii de bunuri şi prestări de servicii nedeductibile 50% taxabile cu cota 5%</field>
                                <field name="code">tax_ro_tva_rd112</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd112_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">11_2 - TVA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd12" model="account.report.line">
                        <field name="name">12 - TVA - Achiziţii de bunuri şi servicii supuse măsurilor de simplificare pentru care beneficiarul este obligat la plata TVA (taxare inversă) , din care:</field>
                        <field name="code">tax_ro_tva_rd12</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">12 - TVA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd121" model="account.report.line">
                                <field name="name">12.1 - TVA - Achizitii de bunuri si servicii, taxabile cu cota 19%</field>
                                <field name="code">tax_ro_tva_rd121</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd121_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">12_1 - TVA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd122" model="account.report.line">
                                <field name="name">12.2 - TVA - Achizitii de bunuri si servicii, taxabile cu cota 9%</field>
                                <field name="code">tax_ro_tva_rd122</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd122_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">12_2 - TVA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd123" model="account.report.line">
                                <field name="name">12.3 - TVA - Achizitii de bunuri si servicii, taxabile cu cota 5%</field>
                                <field name="code">tax_ro_tva_rd123</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd123_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">12_3 - TVA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd16" model="account.report.line">
                        <field name="name">16 - TVA - Regularizări taxă colectată</field>
                        <field name="code">tax_ro_tva_rd16</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd16_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">16 - TVA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd17" model="account.report.line">
                        <field name="name">17 - TVA - Prestări de servicii intracomunitare conform art.278 alin.(8) din Codul fiscal pentru care locul prestării este în România</field>
                        <field name="code">tax_ro_tva_rd17</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd17_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">17 - TVA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd18" model="account.report.line">
                        <field name="name">18 - TVA - Regularizări privind prestări de servicii intracomunitare conform art.278 alin.(8) din Codul fiscal pentru care locul prestării este în România</field>
                        <field name="code">tax_ro_tva_rd18</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd18_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">18 - TVA</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_baza_col" model="account.report.line">
                <field name="name">Baza Total Taxa COLECTATĂ</field>
                <field name="code">total_tax_ro_baza_col</field>
                <field name="aggregation_formula">tax_ro_baza_intracom_eu.balance + tax_ro_baza_livrari.balance</field>
            </record>
            <record id="account_tax_report_ro_tva_col" model="account.report.line">
                <field name="name">TVA Total Taxa COLECTATĂ</field>
                <field name="code">total_tax_ro_tva_col</field>
                <field name="aggregation_formula">tax_ro_tva_intracom_eu.balance + tax_ro_tva_livrari.balance</field>
            </record>
            <record id="account_tax_report_ro_baza_intracom_eu_achiz" model="account.report.line">
                <field name="name">BAZA ACHIZIŢII INTRACOMUNITARE DE BUNURI ŞI ALTE ACHIZIŢII DE BUNURI ŞI SERVICII IMPOZABILE ÎN ROMÂNIA</field>
                <field name="code">tax_ro_baza_intracom_eu_a</field>
                <field name="aggregation_formula">tax_ro_baza_rd20.balance + tax_ro_baza_rd21.balance + tax_ro_baza_rd22.balance + tax_ro_baza_rd23.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_baza_rd20" model="account.report.line">
                        <field name="name">20 - BAZA - Achiziţii intracomunitare de bunuri pentru care cumpărătorul este obligat la plata TVA (taxare inversă) (rd.18=rd.5), din care:</field>
                        <field name="code">tax_ro_baza_rd20</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd20_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">20 - BAZA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd201" model="account.report.line">
                                <field name="name">20.1 - BAZA - Achiziţii intracom. pentru care cumpărătorul este obligat la plata TVA (TI), iar furnizorul este înregistrat în scopuri de TVA în statul membru din care a avut loc livrarea (rd.18.1=rd.5.1)</field>
                                <field name="code">tax_ro_baza_rd201</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd201_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">20_1 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd21" model="account.report.line">
                        <field name="name">21 - BAZA - Regularizări privind achiziţiile intracomunitare de bunuri pentru care cumparatorul este obligat la plata TVA (taxare inversa) (rd.19=rd.6)</field>
                        <field name="code">tax_ro_baza_rd21</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd21_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">21 - BAZA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd22" model="account.report.line">
                        <field name="name">22 - BAZA - Achiziţii de bunuri, altele decat cele de la rd. 18 şi 19, si achizitii de servicii pentru care beneficiarul din Romania este obligat la plata TVA (taxare inversa) (rd.20=rd.7), din care:</field>
                        <field name="code">tax_ro_baza_rd22</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd22_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">22 - BAZA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd221" model="account.report.line">
                                <field name="name">22.1 - BAZA - Achizitii de servicii intracomunitare pentru care beneficiarul este obligat la plata TVA (taxare inversa) (rd.20.1=rd.7.1)</field>
                                <field name="code">tax_ro_baza_rd221</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd221_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">22_1 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd23" model="account.report.line">
                        <field name="name">23 - BAZA - Regularizari privind achizitii de servicii intracomunitare pentru care beneficiarul din Romania este obligat la plata TVA (taxare inversa) (rd.21=rd.8)</field>
                        <field name="code">tax_ro_baza_rd23</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd23_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">23 - BAZA</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_intracom_eu_achiz" model="account.report.line">
                <field name="name">TVA ACHIZIŢII INTRACOMUNITARE DE BUNURI ŞI ALTE ACHIZIŢII DE BUNURI ŞI SERVICII IMPOZABILE ÎN ROMÂNIA</field>
                <field name="code">tax_ro_tva_intracom_eu_a</field>
                <field name="aggregation_formula">tax_ro_tva_rd20.balance + tax_ro_tva_rd21.balance + tax_ro_tva_rd22.balance + tax_ro_tva_rd23.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_tva_rd20" model="account.report.line">
                        <field name="name">20 - TVA - Achiziţii intracomunitare de bunuri pentru care cumpărătorul este obligat la plata TVA (taxare inversă) (rd.18=rd.5), din care:</field>
                        <field name="code">tax_ro_tva_rd20</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd20_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">20 - TVA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd201" model="account.report.line">
                                <field name="name">20.1 - TVA - Achiziţii intracom. pentru care cumpărătorul este obligat la plata TVA (TI), iar furnizorul este înregistrat în scopuri de TVA în statul membru din care a avut loc livrarea (rd.18.1=rd.5.1)</field>
                                <field name="code">tax_ro_tva_rd201</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd201_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">20_1 - TVA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd21" model="account.report.line">
                        <field name="name">21 - TVA - Regularizări privind achiziţiile intracomunitare de bunuri pentru care cumparatorul este obligat la plata TVA (taxare inversa) (rd.19=rd.6)</field>
                        <field name="code">tax_ro_tva_rd21</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd21_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">21 - TVA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd22" model="account.report.line">
                        <field name="name">22 - TVA - Achiziţii de bunuri, altele decat cele de la rd. 18 şi 19, si achizitii de servicii pentru care beneficiarul din Romania este obligat la plata TVA (taxare inversa) (rd.20=rd.7), din care:</field>
                        <field name="code">tax_ro_tva_rd22</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd22_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">22 - TVA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd221" model="account.report.line">
                                <field name="name">22.1 - TVA - Achizitii de servicii intracomunitare pentru care beneficiarul este obligat la plata TVA (taxare inversa) (rd.20.1=rd.7.1)</field>
                                <field name="code">tax_ro_tva_rd221</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd221_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">22_1 - TVA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd23" model="account.report.line">
                        <field name="name">23 - TVA - Regularizari privind achizitii de servicii intracomunitare pentru care beneficiarul din Romania este obligat la plata TVA (taxare inversa) (rd.21=rd.8)</field>
                        <field name="code">tax_ro_tva_rd23</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd23_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">23 - TVA</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_baza_achiz" model="account.report.line">
                <field name="name">BAZA ACHIZIŢII DE BUNURI/ SERVICII ÎN INTERIORUL ŢĂRII ŞI IMPORTURI, ACHIZIŢII INTRACOMUNITARE, SCUTITE SAU NEIMPOZABILE</field>
                <field name="code">tax_ro_baza_achiz</field>
                <field name="aggregation_formula">tax_ro_baza_rd24.balance + tax_ro_baza_rd25.balance + tax_ro_baza_rd26.balance + tax_ro_baza_rd27.balance + tax_ro_baza_rd30.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_baza_rd24" model="account.report.line">
                        <field name="name">24 - BAZA - Achiziţii de bunuri şi servicii taxabile cu cota de 19%, altele decat cele de la rd.27</field>
                        <field name="code">tax_ro_baza_rd24</field>
                        <field name="aggregation_formula">tax_ro_baza_rd241.balance + 0.5 * tax_ro_baza_rd242.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd241" model="account.report.line">
                                <field name="name">24.1 - BAZA - Achiziţii de bunuri şi servicii taxabile cu cota de 19%, altele decat cele de la rd.27</field>
                                <field name="code">tax_ro_baza_rd241</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd241_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">24_1 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd242" model="account.report.line">
                                <field name="name">24.2 - BAZA - Achiziţii de bunuri şi servicii taxabile cu cota de 19%, nedeductibile 50%</field>
                                <field name="code">tax_ro_baza_rd242</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd242_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">24_2 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd25" model="account.report.line">
                        <field name="name">25 - BAZA - Achiziţii de bunuri şi servicii taxabile cu cota de 9%</field>
                        <field name="code">tax_ro_baza_rd25</field>
                        <field name="aggregation_formula">tax_ro_baza_rd251.balance + 0.5 * tax_ro_baza_rd252.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd251" model="account.report.line">
                                <field name="name">25.1 - BAZA - Achiziţii de bunuri şi servicii taxabile cu cota de 9%</field>
                                <field name="code">tax_ro_baza_rd251</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd251_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">25_1 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd252" model="account.report.line">
                                <field name="name">25.2 - BAZA - Achiziţii de bunuri şi servicii taxabile cu cota de 9%, nedeductibile 50%</field>
                                <field name="code">tax_ro_baza_rd252</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd252_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">25_2 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd26" model="account.report.line">
                        <field name="name">26 - BAZA - Achiziţii de bunuri taxabile cu cota de 5%</field>
                        <field name="code">tax_ro_baza_rd26</field>
                        <field name="aggregation_formula">tax_ro_baza_rd261.balance + 0.5 * tax_ro_baza_rd262.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd261" model="account.report.line">
                                <field name="name">26.1 - BAZA - Achiziţii de bunuri taxabile cu cota de 5%</field>
                                <field name="code">tax_ro_baza_rd261</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd261_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">26_1 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd262" model="account.report.line">
                                <field name="name">26.2 - BAZA - Achiziţii de bunuri taxabile cu cota de 5%, nedeductibile 50%</field>
                                <field name="code">tax_ro_baza_rd262</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd262_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">26_2 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd27" model="account.report.line">
                        <field name="name">27 - BAZA - Achiziţii de bunuri şi servicii supuse masurilor de simplificare pentru care beneficiarul este obligat la plata TVA (taxare inversa),din care (rd.25=rd.12)</field>
                        <field name="code">tax_ro_baza_rd27</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd27_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">27 - BAZA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd271" model="account.report.line">
                                <field name="name">27.1 - BAZA - Achizitii de bunuri si servicii, taxabile cu cota 19% (rd.25.1=rd.12.1)</field>
                                <field name="code">tax_ro_baza_rd271</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd271_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">27_1 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd272" model="account.report.line">
                                <field name="name">27.2 - BAZA - Achizitii de bunuri, taxabile cu cota 9% (rd.25.2=rd.12.2)</field>
                                <field name="code">tax_ro_baza_rd272</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd272_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">27_2 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_baza_rd273" model="account.report.line">
                                <field name="name">27.3 - BAZA - Achizitii de bunuri, taxabile cu cota 5% (rd.25.3=rd.12.3)</field>
                                <field name="code">tax_ro_baza_rd273</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd273_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">27_3 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_baza_rd30" model="account.report.line">
                        <field name="name">30 - BAZA - Achiziţii de bunuri şi servicii scutite de taxă sau neimpozabile, din care:</field>
                        <field name="code">tax_ro_baza_rd30</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_baza_rd30_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">30 - BAZA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_baza_rd301" model="account.report.line">
                                <field name="name">30.1 - BAZA - Achizitii de servicii intracomunitare scutite de taxa (nu se completeaza la metoda simplificata)</field>
                                <field name="code">tax_ro_baza_rd301</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_baza_rd301_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">30_1 - BAZA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_achiz" model="account.report.line">
                <field name="name">TVA ACHIZIŢII DE BUNURI/ SERVICII ÎN INTERIORUL ŢĂRII ŞI IMPORTURI, ACHIZIŢII INTRACOMUNITARE, SCUTITE SAU NEIMPOZABILE</field>
                <field name="code">tax_ro_tva_achiz</field>
                <field name="aggregation_formula">tax_ro_tva_rd24.balance + tax_ro_tva_rd25.balance + tax_ro_tva_rd26.balance + tax_ro_tva_rd27.balance + tax_ro_tva_rd28.balance + tax_ro_tva_rd29.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_ro_tva_rd24" model="account.report.line">
                        <field name="name">24 - TVA - Achiziţii de bunuri şi servicii taxabile cu cota de 19%, altele decat cele de la rd.27</field>
                        <field name="code">tax_ro_tva_rd24</field>
                        <field name="aggregation_formula">tax_ro_tva_rd241.balance + tax_ro_tva_rd242.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd241" model="account.report.line">
                                <field name="name">24.1 - TVA - Achiziţii de bunuri şi servicii taxabile cu cota de 19%, altele decat cele de la rd.27</field>
                                <field name="code">tax_ro_tva_rd241</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd241_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">24_1 - TVA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd242" model="account.report.line">
                                <field name="name">24.2 - TVA - Achiziţii de bunuri şi servicii taxabile cu cota de 19%, nedeductibile 50%</field>
                                <field name="code">tax_ro_tva_rd242</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd242_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">24_2 - TVA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd25" model="account.report.line">
                        <field name="name">25 - TVA - Achiziţii de bunuri şi servicii taxabile cu cota de 9%</field>
                        <field name="code">tax_ro_tva_rd25</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd25_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">25 - TVA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd251" model="account.report.line">
                                <field name="name">25.1 - TVA - Achiziţii de bunuri şi servicii taxabile cu cota de 9%</field>
                                <field name="code">tax_ro_tva_rd251</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd251_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">25_1 - TVA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd252" model="account.report.line">
                                <field name="name">25.2 - TVA - Achiziţii de bunuri şi servicii taxabile cu cota de 9%, nedeductibile 50%</field>
                                <field name="code">tax_ro_tva_rd252</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd252_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">25_2 - TVA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd26" model="account.report.line">
                        <field name="name">26 - TVA - Achiziţii de bunuri taxabile cu cota de 5%</field>
                        <field name="code">tax_ro_tva_rd26</field>
                        <field name="aggregation_formula">tax_ro_tva_rd261.balance + tax_ro_tva_rd262.balance</field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd261" model="account.report.line">
                                <field name="name">26.1 - TVA - Achiziţii de bunuri taxabile cu cota de 5%</field>
                                <field name="code">tax_ro_tva_rd261</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd261_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">26_1 - TVA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd262" model="account.report.line">
                                <field name="name">26.2 - TVA - Achiziţii de bunuri taxabile cu cota de 5%, nedeductibile 50%</field>
                                <field name="code">tax_ro_tva_rd262</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd262_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">26_2 - TVA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd27" model="account.report.line">
                        <field name="name">27 - TVA - Achiziţii de bunuri şi servicii supuse masurilor de simplificare pentru care beneficiarul este obligat la plata TVA (taxare inversa),din care (rd.25=rd.12)</field>
                        <field name="code">tax_ro_tva_rd27</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd27_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">27 - TVA</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_ro_tva_rd271" model="account.report.line">
                                <field name="name">27.1 - TVA - Achizitii de bunuri si servicii, taxabile cu cota 19% (rd.25.1=rd.12.1)</field>
                                <field name="code">tax_ro_tva_rd271</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd271_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">27_1 - TVA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd272" model="account.report.line">
                                <field name="name">27.2 - TVA - Achizitii de bunuri, taxabile cu cota 9% (rd.25.2=rd.12.2)</field>
                                <field name="code">tax_ro_tva_rd272</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd272_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">27_2 - TVA</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_ro_tva_rd273" model="account.report.line">
                                <field name="name">27.3 - TVA - Achizitii de bunuri, taxabile cu cota 5% (rd.25.3=rd.12.3)</field>
                                <field name="code">tax_ro_tva_rd273</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_ro_tva_rd273_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">27_3 - TVA</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd28" model="account.report.line">
                        <field name="name">28 - TVA - Compensația în cotă forfetară pentru achiziții de produse și servicii agricole de la furnizori care aplică regimul special pentru agricultori</field>
                        <field name="code">tax_ro_tva_rd28</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd28_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">28 - TVA</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_ro_tva_rd29" model="account.report.line">
                        <field name="name">29 - TVA - Regularizări privind compensația în cotă forfetară</field>
                        <field name="code">tax_ro_tva_rd29</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_ro_tva_rd29_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">29 - TVA</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_baza_total_rd31" model="account.report.line">
                <field name="name">31 - BAZA - TOTAL TAXĂ DEDUCTIBILĂ ( sumă de la rd.20 până la rd.29, cu excepţia celor de la rd.20.1,22.1, 27.1, 27.2, 27.3)</field>
                <field name="code">tax_ro_baza_total_rd31</field>
                <field name="aggregation_formula">tax_ro_baza_intracom_eu_a.balance + tax_ro_baza_achiz.balance</field>
            </record>
            <record id="account_tax_report_ro_tva_total_rd31" model="account.report.line">
                <field name="name">31 - TVA - TOTAL TAXĂ DEDUCTIBILĂ ( sumă de la rd.20 până la rd.29, cu excepţia celor de la rd.20.1,22.1, 27.1, 27.2, 27.3)</field>
                <field name="code">tax_ro_tva_total_rd31</field>
                <field name="aggregation_formula">tax_ro_tva_intracom_eu_a.balance + tax_ro_tva_achiz.balance</field>
            </record>
            <record id="account_tax_report_ro_tva_rd32" model="account.report.line">
                <field name="name">32 - TVA - SUB-TOTAL TAXĂ DEDUSĂ CONFORM ART. 297 ŞI ART. 298 SAU ART. 300 ŞI ART. 298 (rd.30&lt;=rd.29)</field>
                <field name="code">tax_ro_tva_rd32</field>
                <field name="aggregation_formula">tax_ro_tva_intracom_eu_a.balance + tax_ro_tva_achiz.balance</field>
            </record>
            <record id="account_tax_report_ro_tva_rd33" model="account.report.line">
                <field name="name">33 - TVA - TVA efectiv restituită cumpărătorilor straini, inclusiv comisionul unităţilor autorizate</field>
                <field name="code">tax_ro_tva_rd33</field>
                <field name="expression_ids">
                    <record id="account_tax_report_ro_tva_rd33_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">33 - TVA</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_baza_rd34" model="account.report.line">
                <field name="name">34 - BAZA - Regularizări taxă dedusă</field>
                <field name="code">tax_ro_baza_rd34</field>
                <field name="expression_ids">
                    <record id="account_tax_report_ro_baza_rd34_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">34 - BAZA</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_rd34" model="account.report.line">
                <field name="name">34 - TVA - Regularizări taxă dedusă</field>
                <field name="code">tax_ro_tva_rd34</field>
                <field name="expression_ids">
                    <record id="account_tax_report_ro_tva_rd34_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">34 - TVA</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_rd35" model="account.report.line">
                <field name="name">35 - TVA - Ajustări conform pro-rata / ajustări pentru bunurile de capital</field>
                <field name="code">tax_ro_tva_rd35</field>
                <field name="expression_ids">
                    <record id="account_tax_report_ro_tva_rd35_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">35 - TVA</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_rd36" model="account.report.line">
                <field name="name">36 - TVA - TOTAL TAXA DEDUSA (rd.32+rd.33+rd.34+rd.35)</field>
                <field name="code">tax_ro_tva_rd36</field>
                <field name="aggregation_formula">tax_ro_tva_intracom_eu_a.balance + tax_ro_tva_achiz.balance + tax_ro_tva_rd33.balance + tax_ro_tva_rd34.balance + tax_ro_tva_rd35.balance</field>
            </record>
            <record id="account_tax_report_ro_tva_rd40" model="account.report.line">
                <field name="name">40 - TVA - Diferenţe de TVA de plată stabilite de organele de inspecţie fiscală prin decizie comunicată şi neachitate până la data depunerii decontului de TVA </field>
                <field name="code">tax_ro_tva_rd40</field>
                <field name="expression_ids">
                    <record id="account_tax_report_ro_tva_rd40_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">40 - TVA</field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_ro_tva_rd43" model="account.report.line">
                <field name="name">43 - TVA - Diferenţe negative de TVA stabilite de organele de inspecţie fiscală prin decizie comunicată până la data depunerii decontului de TVA </field>
                <field name="code">tax_ro_tva_rd43</field>
                <field name="expression_ids">
                    <record id="account_tax_report_ro_tva_rd43_tag" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">43 - TVA</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_ro_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Chart template -->
    <record id="ro_chart_template" model="account.chart.template">
        <field name="name">Romania - Chart of Accounts</field>
        <field name="bank_account_code_prefix">5121</field>
        <field name="cash_account_code_prefix">5311</field>
        <field name="transfer_account_code_prefix">581</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.RON"/>
        <field name="country_id" ref="base.ro"/>
        <field name="use_storno_accounting" eval="True"/>
    </record>
</odoo>

```

## File: data\l10n_ro_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ro_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="ro_pcg_recv"/> <!-- 4111 -->
        <field name="property_account_payable_id" ref="ro_pcg_pay"/> <!-- 4011 -->
        <field name="property_account_expense_categ_id" ref="ro_pcg_expense"/> <!-- 607 -->
        <field name="property_account_income_categ_id" ref="ro_pcg_sale"/> <!-- 707 -->
        <field name="income_currency_exchange_account_id" ref="pcg_7651"/>
        <field name="expense_currency_exchange_account_id" ref="pcg_6651"/>
        <field name="property_tax_payable_account_id" ref="pcg_4423"/>
        <field name="property_tax_receivable_account_id" ref="pcg_4424"/>
        <field name="default_pos_receivable_account_id" ref="ro_pcg_recv"/>
        <field name="account_journal_suspense_account_id" ref="pcg_5125"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="pcg_609"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="pcg_709"/>
    </record>
</odoo>

```

## File: data\res.bank.csv

```csv
"id","active","name","bic","street","city","state/id","country/id","phone"
"res_bank_1","TRUE","Alpha Bank","BUCUROBU","Calea Dorobantilor 237 B, sector1","BUCURESTI","base.RO_B","base.ro","(021) 209.99.99"
"res_bank_2","TRUE","Anglo-Romanian Bank Limited, Anglia Londra","ARBLROBU","Bd. Carol I nr.34-36, sector 2","BUCURESTI","base.RO_B","base.ro","+44(0)207 826 4200"
"res_bank_3","TRUE","Ate Bank","MINDROBU","Bd.Grivitei, nr. 24, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 30.30.732"
"res_bank_4","TRUE","Banca Comercială Carpatica","CARPRO22","str. Autogarii nr.1","SIBIU","base.RO_SB","base.ro","(0269) 23.39.85"
"res_bank_5","TRUE","Banca Comercială Română","RNCBROBU","Bd.Regina Elisabeta nr.5, sector 3","BUCURESTI","base.RO_B","base.ro","0801 0801 227"
"res_bank_6","TRUE","Banca C.R. Firenze","DAROROBU","Bd. Unirii nr.55, bl.E4a, Tronson 1, sector 3","BUCURESTI","base.RO_B","base.ro","(021) 201.19.30"
"res_bank_7","TRUE","Banca de Export-Import a României Eximbank","EXIMROBU","Spl. Independentei nr.15, sector 5","BUCURESTI","base.RO_B","base.ro","(021) 336.61.62"
"res_bank_8","TRUE","Banca di Roma","BROMROBU","Intrarea Murmurului nr. 2-4, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 232.08.18"
"res_bank_9","TRUE","Banca Italo Romena","BITRROBU","Bd. Dimitrie Cantemir nr.1, bl.B2, sc.2, parter si mezanin, sector 4","BUCURESTI","base.RO_B","base.ro","(021) 330.78.76"
"res_bank_10","TRUE","Banca Românească","BRMAROBU","bd.Unirii nr.35, bl.A3, sector 3","BUCURESTI","base.RO_B","base.ro","(021) 321.16.01"
"res_bank_11","TRUE","Banca Transilvania","BTRLRO22","George Baritiu nr.8","CLUJ-NAPOCA","base.RO_CJ","base.ro","(0264) 407 150"
"res_bank_12","TRUE","Banc Post","BPOSROBU","Calea Vitan nr.6, 6A, Tronson B si C, et.3-7, sector 3","BUCURESTI","base.RO_B","base.ro","(021) 308.09.01"
"res_bank_13","TRUE","Blom Bank Egypt","MIRBROBU","Bd. Unirii nr.66 bl. K 3 sector 3","BUCURESTI","base.RO_B","base.ro","(021) 302.72.00"
"res_bank_14","TRUE","BRD - Groupe Société Générale","BRDEROBU","Bd. Ion Mihalache nr.1-7, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 301.61.00"
"res_bank_15","TRUE","C.E.C.","CECEROBU","Calea Victoriei nr.13, sector 3","BUCURESTI","base.RO_B","base.ro","(021) 311.11.19."
"res_bank_16","TRUE","Citibank România","CITIROBU","bd. Iancu de Hunedoara nr. 8, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 210.18.50"
"res_bank_17","TRUE","Credit Coop Casa Centrală","CRCOROBU","Calea Plevnei nr.200","BUCURESTI","base.RO_B","base.ro","(021) 317.74.05"
"res_bank_18","TRUE","Credit Europe Bank","FNNBROBU","Bd. Timisoara Nr. 26Z, Sector 6","BUCURESTI","base.RO_B","base.ro","(021) 406.40.00"
"res_bank_19","TRUE","Egnatia Bank","EGNAROBX","str. General Constantin Budisteanu nr.28C, P+1, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 303.21.00"
"res_bank_20","TRUE","Emporiki Bank - România","BSEAROBU","str.Berzei nr.19, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 310.39.55"
"res_bank_21","TRUE","GarantiBank International NV","UGBIROBU","str.Paris nr.30, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 230.84.30"
"res_bank_22","TRUE","HVB Banca pentru Locuinţe","HVBLROBU","str.Dr.Grigore Mora nr.37, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 300 11 22"
"res_bank_23","TRUE","ING Bank N.V., Amsterdam","INGBROBU","sos.Kiseleff nr.11-13, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 222.16.00"
"res_bank_24","TRUE","Leumi Bank România","DAFBRO22","B-dul Aviatorilor nr.45, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 206.70.75"
"res_bank_25","TRUE","Libra Bank","BRELROBU","str. dr. Grigore Mora nr.11, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 20.88.000"
"res_bank_26","TRUE","Millenium Bank","MILBROBU","Piaţa Presei Libere nr. 3-5, Clădirea City Gate, Turnul Sudic, parter si et. 13-17","BUCURESTI","base.RO_B","base.ro","(021) 308 13 00"
"res_bank_27","TRUE","OTP Bank România S.A.","OTPVROBU","str.Buzesti nr.66-68, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 307.57.00"
"res_bank_28","TRUE","Piraeus Bank România","PIRBROBU","bd.Carol I nr.34-36, et. VI, sector 2","BUCURESTI","base.RO_B","base.ro","(021) 250.67.98"
"res_bank_29","TRUE","Porsche Bank România","PORLROBU","sos.Pipera-Tunari nr.2, cladirea PORSCHE, parter, etaj 1 si 2","VOLUNTARI","base.RO_B","base.ro","(021) 208.26.00"
"res_bank_30","TRUE","ProCredit Bank","MIROROBU","str.Buzesti nr.62-64, et.1 si et.2, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 201.60.00"
"res_bank_31","TRUE","Raiffeisen Banca pentru Locuinţe","RZBLROBU","str. Nicolae Caramfil nr.79, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 233.30.00"
"res_bank_32","TRUE","Raiffeisen Bank","RZBRROBU","Piata Charles de Gaulle nr.15, et.4,5,6,7 si 8, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 323.00.31"
"res_bank_33","TRUE","RBS Bank România","ABNAROBU","Piata Montreal nr.10, WTCB - E etajul 2, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 20.20.400"
"res_bank_34","TRUE","Romanian International Bank","ROINROBU","bd.Unirii nr.68, bl. K2, sector 3","BUCURESTI","base.RO_B","base.ro","(021) 323.10.35."
"res_bank_35","TRUE","Romexterra Bank","CRDZROBU","Bdul 1 Decembrie 1918 nr.93","TARGU MURES","base.RO_MS","base.ro","(0265) 16.66.41"
"res_bank_36","TRUE","Sanpaolo Imi Bank România","WBANRO22","str.Revolutiei nr.88","ARAD","base.RO_AR","base.ro","(0257) 30.82.00"
"res_bank_37","TRUE","Trezoreria Statului","TREZROBU","Splaiul Unirii 6-8","BUCURESTI","base.RO_B","base.ro","(021)  317.27.70"
"res_bank_38","TRUE","UniCredit Tiriac Bank","BACXROBU","Str. Ghetarilor nr.23-25, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 200.20.20"
"res_bank_39","TRUE","Volksbank Romania","VBBUROBU","sos. Mihai Bravu nr.171, sector 2","BUCURESTI","base.RO_B","base.ro","(021) 303.93.00"
"res_bank_40","TRUE","Banca Comercială Feroviară","BFERROBU","Strada Popa Tatu nr. 62A, Tronson A, Sector 1,","BUCURESTI","base.RO_B","base.ro","(021) 303.40.00"
"res_bank_41","TRUE","BCR Banca pentru Locuinţe","BCRLROBU","B-dul Lascar Catargiu, nr.47-53, sector 1","BUCURESTI","base.RO_B","base.ro","0801 080.275"
"res_bank_42","TRUE","Bank of Cyprus plc Nicosia","BCYPROBU","Calea Dorobanti nr.187B, sector 1","BUCURESTI","base.RO_B","base.ro","08000 877.777"
"res_bank_43","TRUE","BNP Paribas Fortis SA/NV","FTSBROBU","str. Banul Antonache nr.40-44, etaj 5, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 310.09.70"
"res_bank_44","TRUE","TBI Bank EAD Sofia","TBIBROBU","Str. Putul lui Zamfir, nr. 8-12, etaj 4, sector 1","BUCURESTI","base.RO_B","base.ro","(021) 529.86.00"

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# @author -  Fekete Mihai <feketemihai@gmail.com>
# Copyright (C) 2020 NextERP Romania (https://www.nexterp.ro) <contact@nexterp.ro>
# Copyright (C) 2015 Forest and Biomass Services Romania (http://www.forbiom.eu).
# Copyright (C) 2011 TOTAL PC SYSTEMS (http://www.erpsystems.ro).
# Copyright (C) 2009 (<http://www.filsystem.ro>)

from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = "res.partner"

    @api.model
    def _commercial_fields(self):
        return super(ResPartner, self)._commercial_fields() + ['nrc']

    nrc = fields.Char(string='NRC', help='Registration number at the Registry of Commerce')

    @api.depends('vat', 'country_id')
    def _compute_company_registry(self):
        # OVERRIDE
        # In Romania, if you have a VAT number, it's also your company registry (CUI) number
        super()._compute_company_registry()
        for partner in self.filtered(lambda p: p.country_id.code == 'RO' and p.vat):
            vat_country, vat_number = self._split_vat(partner.vat)
            if vat_country.isnumeric():
                vat_country = 'ro'
                vat_number = partner.vat
            if vat_country == 'ro' and self.simple_vat_check(vat_country, vat_number):
                partner.company_registry = vat_number

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="5.8" width="50.4" height="36.42" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.37" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="1000" height="667" transform="translate(4.8 5.8) scale(0.05 0.05)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAA+kAAALTCAYAAABqq0MAAAAACXBIWXMAANvDAADbwwG11nNdAAAR4UlEQVR4Xu3XLW4UUBhG4XeG8GMqwYFjR8huA4lkK6yERaARKILCNE1nEDNVCBw5SZ8nueb+yS85h73/fB7AU3PYdjxt5233x22n7dnj4cvtdNoOd9tx28Pz7cXv7e7bPt2+2scPX3Zz3OWtCQo8RYfrevSwyzw8Xtf5uvfj3b7eft+bX9vNXu/+Mnx32P1fXwKwvT3/PBz/dQkAAAD4P0Q6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARIh0AAAAiBDpAAAAECHSAQAAIEKkAwAAQIRIBwAAgAiRDgAAABEiHQAAACJEOgAAAESIdAAAAIgQ6QAAABAh0gEAACBCpAMAAECESAcAAIAIkQ4AAAARIh0AAAAiRDoAAABEiHQAAACIEOkAAAAQIdIBAAAgQqQDAABAhEgHAACACJEOAAAAESIdAAAAIkQ6AAAARPwBHDcio77lqvMAAAAASUVORK5CYII="/>
    </g>
  </g>
</svg>

```

## File: views\res_partner_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
		<record model="ir.ui.view" id="res_partner_form_ro">
			<field name="name">res.partner.form.ro</field>
			<field name="inherit_id" ref="account.view_partner_property_form"/>
			<field name="model">res.partner</field>
			<field name="arch" type="xml">
				<field name="vat" position="after">
					<field name="nrc" placeholder="e.g. J12/1234/2012" class="oe_inline"/>
                </field>
			</field>
		</record>
</odoo>

```

