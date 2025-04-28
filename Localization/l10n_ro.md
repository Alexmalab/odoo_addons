# Odoo Module: l10n_ro

Category: Localization

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
# Copyright (C) 2015 Tatár Attila
# Copyright (C) 2015 Forest and Biomass Services Romania (http://www.forbiom.eu).
# Copyright (C) 2011 TOTAL PC SYSTEMS (http://www.erpsystems.ro).
# Copyright (C) 2009 (<http://www.filsystem.ro>)

{
    "name" : "Romania - Accounting",
    "author" : "Fekete Mihai (Forest and Biomass Services Romania)",
    "website": "http://www.forbiom.eu",
    'category': 'Localization',
    "depends" : [
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
             'data/account.account.template.csv',
             'data/l10n_ro_chart_post_data.xml',
             'data/account_data.xml',
             'data/account_tax_report_data.xml',
             'data/account_tax_data.xml',
             'data/account_fiscal_position_data.xml',
             'data/account_chart_template_data.xml',
             'data/res.bank.csv',
             ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","reconcile"
"pcg_1011","Capital subscris nevărsat","1011","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1012","Capital subscris vărsat","1012","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1015","Patrimoniul regiei","1015","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1016","Patrimoniul public","1016","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1017","Patrimoniul privat","1017","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1018","Patrimoniul institutelor naţionale de cercetare-dezvoltare","1018","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1031","Beneficii acordate angajaţilor sub forma instrumentelor de capitaluri proprii","1031","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1033","Diferenţe de curs valutar în relaţie cu investiţia netă într-o entitate străină","1033","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1038","Diferenţe din modificarea valorii juste a activelor financiare disponibile în vederea vânzării şi alte elemente de capitaluri proprii","1038","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1041","Prime de emisiune","1041","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1042","Prime de fuziune/divizare","1042","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1043","Prime de aport","1043","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1044","Prime de conversie a obligaţiunilor în acţiuni","1044","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_105","Rezerve din reevaluare","105","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1061","Rezerve legale","1061","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1063","Rezerve statutare sau contractuale","1063","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1068","Alte rezerve","1068","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_107","Diferenţe de curs valutar din conversie","107","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1081","Interese care nu controlează - rezultatul exerciţiului financiar","1081","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1082","Interese care nu controlează - alte capitaluri proprii","1082","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1091","Acţiuni proprii deţinute pe termen scurt","1091","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1092","Acţiuni proprii deţinute pe termen lung","1092","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1095","Acţiuni proprii reprezentând titluri deţinute de societatea absorbită la societatea absorbantă","1095","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1171","Rezultatul reportat reprezentând profitul nerepartizat sau pierderea neacoperită","1171","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1172","Rezultatul reportat provenit din adoptarea pentru prima dată a IAS, mai puţin IAS 29","1172","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1173","Rezultatul reportat provenit din modificările politicilor contabile","1173","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1174","Rezultatul reportat provenit din corectarea erorilor contabile","1174","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1175","Rezultatul reportat reprezentând surplusul realizat din rezerve din reevaluare","1175","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1176","Rezultatul reportat provenit din trecerea la aplicarea reglementărilor contabile conforme cu directivele europene","1176","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_121","Profit sau pierdere","121","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_129","Repartizarea profitului","129","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1411","Câştiguri legate de vânzarea instrumentelor de capitaluri proprii","1411","account.data_unaffected_earnings","l10n_ro.ro_chart_template","False"
"pcg_1412","Câştiguri legate de anularea instrumentelor de capitaluri proprii","1412","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1491","Pierderi rezultate din vânzarea instrumentelor de capitaluri proprii","1491","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1495","Pierderi rezultate din reorganizări, care sunt determinate de anularea titlurilor deţinute","1495","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1498","Alte pierderi legate de instrumentele de capitaluri proprii","1498","account.data_account_type_equity","l10n_ro.ro_chart_template","False"
"pcg_1511","Provizioane pentru litigii","1511","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_1512","Provizioane pentru garanţii acordate clienţilor","1512","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_1513","Provizioane pentru dezafectare imobilizări corporale și alte acțiuni similare legate de acestea","1513","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1514","Provizioane pentru restructurare","1514","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1515","Provizioane pentru pensii și obligații similare","1515","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1516","Provizioane pentru impozite","1516","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1517","Provizioane pentru terminarea contractului de muncă","1517","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1518","Alte provizioane","1518","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1614","Împrumuturi externe din emisiuni de obligaţiuni garantate de stat","1614","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1615","Împrumuturi externe din emisiuni de obligaţiuni garantate de bănci","1615","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1617","Împrumuturi interne din emisiuni de obligaţiuni garantate de stat","1617","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1618","Alte împrumuturi din emisiuni de obligaţiuni","1618","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1621","Credite bancare pe termen lung","1621","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1622","Credite bancare pe termen lung nerambursate la scadenţă","1622","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1623","Credite externe guvernamentale","1623","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1624","Credite bancare externe garantate de stat","1624","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1625","Credite bancare externe garantate de bănci","1625","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1626","Credite de la trezoreria statului","1626","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1627","Credite bancare interne garantate de stat","1627","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1661","Datorii faţă de entităţile afiliate","1661","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1663","Datorii faţă de entităţile asociate şi entităţile controlate în comun","1663","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_167","Alte împrumuturi şi datorii asimilate","167","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1681","Dobânzi aferente împrumuturilor din emisiuni de obligaţiuni","1681","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1682","Dobânzi aferente creditelor bancare pe termen lung","1682","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1685","Dobânzi aferente datoriilor faţă de entităţile afiliate","1685","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1686","Dobânzi aferente datoriilor fată de entitătile asociate şi entităţile controlate în comun","1686","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1687","Dobânzi aferente altor împrumuturi şi datorii asimilate","1687","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1691","Prime privind rambursarea obligaţiunilor","1691","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_1692","Prime privind rambursarea altor datorii","1692","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_201","Cheltuieli de constituire","201","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_203","Cheltuieli de dezvoltare","203","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_205","Concesiuni, brevete, licenţe, mărci comerciale, drepturi şi active similare","205","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_206","Active necorporale de explorare şi evaluare a resurselor minerale","206","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_2071","Fond comercial pozitiv","2071","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_2075","Fond comercial negativ","2075","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_208","Alte imobilizări necorporale","208","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_2111","Terenuri","2111","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_2112","Amenajări de terenuri","2112","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_212","Construcţii","212","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_2131","Echipamente tehnologice (maşini, utilaje şi instalaţii de lucru)","2131","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_2132","Aparate şi instalaţii de măsurare, control şi reglare","2132","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_2133","Mijloace de transport","2133","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_214","Mobilier, aparatură birotică, echipamente de protecţie a valorilor umane şi materiale şi alte active corporale","214","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_215","Investiţii imobiliare","215","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_216","Active corporale de explorare şi evaluare a resurselor minerale","216","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_217","Active biologice productive","217","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_223","Instalaţii tehnice şi mijloace de transport în curs de aprovizionare","223","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_224","Mobilier, aparatură birotică, echipamente de protecţie a valorilor umane şi materiale şi alte active corporale în curs de aprovizionare","224","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_227","Active biologice productive în curs de aprovizionare","227","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_231","Imobilizări corporale în curs de execuţie","231","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_235","Investiţii imobiliare în curs de execuţie","235","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_261","Acţiuni deţinute la entităţile afiliate","261","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_262","Acţiuni deţinute la entităţi asociate","262","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_263","Acţiuni deţinute la entităţi controlate în comun","263","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_264","Titluri puse în echivalenţă","264","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_265","Alte titluri imobilizate","265","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_266","Certificate verzi amânate","266","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_2671","Sume de încasat de la entităţile afiliate","2671","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_2672","Dobânda aferentă sumelor de încasat de la entităţile afiliate","2672","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_2673","Creanţe faţă de entităţile asociate şi entităţile controlate în comun","2673","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_2674","Dobânda aferentă creanţelor faţă de entităţile asociate şi entităţile controlate în comun","2674","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_2675","Împrumuturi acordate pe termen lung","2675","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_2676","Dobânda aferentă împrumuturilor acordate pe termen lung","2676","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_2677","Obligaţiuni achiziţionate cu ocazia emisiunilor efectuate de terţi","2677","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_2678","Alte creanţe imobilizate","2678","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_2679","Dobânzi aferente altor creanţe imobilizate","2679","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_2691","Vărsăminte de efectuat privind acţiunile deţinute la entităţile afiliate","2691","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_2692","Vărsăminte de efectuat privind acţiunile deţinute la entităţi asociate","2692","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_2693","Vărsăminte de efectuat privind acţiunile deţinute la entităţi controlate în comun","2693","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_2695","Vărsăminte de efectuat pentru alte imobilizări financiare","2695","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_2801","Amortizarea cheltuielilor de constituire","2801","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2803","Amortizarea cheltuielilor de dezvoltare","2803","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2805","Amortizarea concesiunilor, brevetelor, licenţelor, mărcilor comerciale, drepturilor şi activelor similare","2805","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2806","Amortizarea activelor necorporale de explorare şi evaluare a resurselor minerale","2806","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2807","Amortizarea fondului comercial","2807","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2808","Amortizarea altor imobilizări necorporale","2808","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2811","Amortizarea amenajărilor de terenuri","2811","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2812","Amortizarea construcţiilor","2812","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2813","Amortizarea instalaţiilor şi mijloacelor de transport","2813","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2814","Amortizarea altor imobilizări corporale","2814","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2815","Amortizarea investiţiilor imobiliare","2815","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2816","Amortizarea activelor corporale de explorare şi evaluare a resurselor minerale","2816","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2817","Amortizarea activelor biologice productive","2817","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2903","Ajustări pentru deprecierea cheltuielilor de dezvoltare","2903","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2905","Ajustări pentru deprecierea concesiunilor, brevetelor, licenţelor, mărcilor comerciale, drepturilor şi activelor similare","2905","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2906","Ajustări pentru deprecierea activelor necorporale de explorare şi evaluare a resurselor minerale","2906","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2908","Ajustări pentru deprecierea altor imobilizări necorporale","2908","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2911","Ajustări pentru deprecierea terenurilor şi amenajărilor de terenuri","2911","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2912","Ajustări pentru deprecierea construcţiilor","2912","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2913","Ajustări pentru deprecierea instalaţiilor şi mijloacelor de transport","2913","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2914","Ajustări pentru deprecierea altor imobilizări corporale","2914","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2915","Ajustări pentru deprecierea investiţiilor imobiliare","2915","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2916","Ajustări pentru deprecierea activelor corporale de explorare şi evaluare a resurselor minerale","2916","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2917","Ajustări pentru deprecierea activelor biologice productive","2917","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2931","Ajustări pentru deprecierea imobilizărilor corporale în curs de execuţie","2931","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2935","Ajustări pentru deprecierea investiţii lor imobiliare în curs de execuţie","2935","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2961","Ajustări pentru pierderea de valoare a acţiunilor deţinute la entităţile afiliate","2961","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2962","Ajustări pentru pierderea de valoare a acţiunilor deţinute la entităţi asociate şi entităţi controlate în comun","2962","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2963","Ajustări pentru pierderea de valoare a altor titluri imobilizate","2963","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2964","Ajustări pentru pierderea de valoare a sumelor de încasat de la entităţile afiliate","2964","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2965","Ajustări pentru pierderea de valoare a creanţelor faţă de entităţile asociate şi entităţile controlate în comun","2965","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2966","Ajustări pentru pierderea de valoare a împrumuturilor acordate pe termen lung","2966","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_2968","Ajustări pentru pierderea de valoare a altor creanţe","2968","account.data_account_type_depreciation","l10n_ro.ro_chart_template","False"
"pcg_301","Materii prime","301","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3021","Materiale auxiliare","3021","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3022","Combustibili","3022","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3023","Materiale pentru ambalat","3023","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3024","Piese de schimb","3024","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3025","Seminţe şi materiale de plantat","3025","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3026","Furaje","3026","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3028","Alte materiale consumabile","3028","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_303","Materiale de natura obiectelor de inventar","303","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_308","Diferenţe de preţ la materii prime şi materiale","308","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_321","Materii prime în curs de aprovizionare","321","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_322","Materiale consumabile în curs de aprovizionare","322","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_323","Materiale de natura obiectelor de inventar în curs de aprovizionare","323","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_326","Active biologice de natura stocurilor în curs de aprovizionare","326","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_327","Mărfuri în curs de aprovizionare","327","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_328","Ambalaje în curs de aprovizionare","328","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_331","Produse în curs de execuţie","331","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_332","Servicii în curs de execuţie","332","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_341","Semifabricate","341","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_345","Produse finite","345","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_346","Produse reziduale","346","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_347","Produse agricole","347","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_348","Diferenţe de preţ la produse","348","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_351","Materii şi materiale aflate la terţi","351","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_354","Produse aflate la terţi","354","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_356","Active biologice de natura stocurilor aflate la terţi","356","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_357","Mărfuri aflate la terţi","357","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_358","Ambalaje aflate la terţi","358","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_361","Active biologice de natura stocurilor","361","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_368","Diferenţe de preţ la active biologice de natura stocurilor","368","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_371","Mărfuri","371","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_378","Diferenţe de preţ la mărfuri","378","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_381","Ambalaje","381","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_388","Diferenţe de preţ la ambalaje","388","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_391","Ajustări pentru deprecierea materiilor prime","391","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3921","Ajustări pentru deprecierea materialelor consumabile","3921","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3922","Ajustări pentru deprecierea materialelor de natura obiectelor de inventar","3922","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_393","Ajustări pentru deprecierea producţiei în curs de execuţie","393","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3941","Ajustări pentru deprecierea semifabricatelor","3941","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3945","Ajustări pentru deprecierea produselor finite","3945","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3946","Ajustări pentru deprecierea produselor reziduale","3946","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3947","Ajustări pentru deprecierea produselor agricole","3947","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3951","Ajustări pentru deprecierea materiilor şi materialelor aflate la terţi","3951","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3952","Ajustări pentru deprecierea semifabricatelor aflate la terţi","3952","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3953","Ajustări pentru deprecierea produselor finite aflate la terţi","3953","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3954","Ajustări pentru deprecierea produselor reziduale aflate la terţi","3954","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3955","Ajustări pentru deprecierea produselor agricole aflate la terţi","3955","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3956","Ajustări pentru deprecierea activelor biologice de natura stocurilor aflate la terţi","3956","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3957","Ajustări pentru deprecierea mărfurilor aflate la terţi","3957","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_3958","Ajustări pentru deprecierea ambalajelor aflate la terţi","3958","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_396","Ajustări pentru deprecierea activelor biologice de natura stocurilor","396","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_397","Ajustări pentru deprecierea mărfurilor","397","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_398","Ajustări pentru deprecierea ambalajelor","398","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"ro_pcg_pay","Furnizori","401","account.data_account_type_payable","l10n_ro.ro_chart_template","True"
"pcg_403","Efecte de plătit","403","account.data_account_type_payable","l10n_ro.ro_chart_template","True"
"pcg_404","Furnizori de imobilizări","404","account.data_account_type_payable","l10n_ro.ro_chart_template","True"
"pcg_405","Efecte de plătit pentru imobilizări","405","account.data_account_type_payable","l10n_ro.ro_chart_template","True"
"pcg_408","Furnizori - facturi nesosite","408","account.data_account_type_payable","l10n_ro.ro_chart_template","True"
"pcg_4091","Furnizori - debitori pentru cumpărări de bunuri de natura stocurilor","4091","account.data_account_type_payable","l10n_ro.ro_chart_template","True"
"pcg_4092","Furnizori - debitori pentru prestări de servicii","4092","account.data_account_type_payable","l10n_ro.ro_chart_template","True"
"pcg_4093","Avansuri acordate pentru imobilizări corporale","4093","account.data_account_type_payable","l10n_ro.ro_chart_template","True"
"pcg_4094","Avansuri acordate pentru imobilizări necorporale","4094","account.data_account_type_payable","l10n_ro.ro_chart_template","True"
"ro_pcg_recv","Clienţi","4111","account.data_account_type_receivable","l10n_ro.ro_chart_template","True"
"ro_pcg_recv_pos","Clienţi (PoS)","4112","account.data_account_type_receivable","l10n_ro.ro_chart_template","True"
"pcg_4118","Clienţi incerţi sau în litigiu","4118","account.data_account_type_receivable","l10n_ro.ro_chart_template","True"
"pcg_413","Efecte de primit de la clienţi","413","account.data_account_type_receivable","l10n_ro.ro_chart_template","True"
"pcg_418","Clienţi - facturi de întocmit","418","account.data_account_type_receivable","l10n_ro.ro_chart_template","True"
"pcg_419","Clienţi - creditori","419","account.data_account_type_receivable","l10n_ro.ro_chart_template","True"
"pcg_421","Personal - salarii datorate","421","account.data_account_type_payable","l10n_ro.ro_chart_template","True"
"pcg_423","Personal - ajutoare materiale datorate","423","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_424","Prime reprezentând participarea personalului la profit","424","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_425","Avansuri acordate personalului","425","account.data_account_type_receivable","l10n_ro.ro_chart_template","True"
"pcg_426","Drepturi de personal neridicate","426","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_427","Reţineri din salarii datorate terţilor","427","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","True"
"pcg_4281","Alte datorii în legătură cu personalul","4281","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4282","Alte creanţe în legătură cu personalul","4282","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_4311","Contribuţia unităţii la asigurările sociale","4311","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4312","Contribuţia personalului la asigurările sociale","4312","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4313","Contribuţia angajatorului pentru asigurările sociale de sănătate","4313","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4314","Contribuţia angajaţilor pentru asigurările sociale de sănătate","4314","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4371","Contribuţia unităţii la fondul de şomaj","4371","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4372","Contribuţia personalului la fondul de şomaj","4372","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4381","Alte datorii sociale","4381","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4382","Alte creanţe sociale","4382","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_4411","Impozitul pe profit","4411","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4418","Impozitul pe venit","4418","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4423","TVA de plată","4423","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4424","TVA de recuperat","4424","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_4426","TVA deductibilă","4426","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4427","TVA colectată","4427","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_44281","TVA neexigibilă - Colectată","44281","account.data_account_type_non_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_44282","TVA neexigibilă - Deductibilă","44282","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_444","Impozitul pe venituri de natura salariilor","444","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","True"
"pcg_4451","Subvenţii guvernamentale","4451","account.data_account_type_non_current_assets","l10n_ro.ro_chart_template","False"
"pcg_4452","Împrumuturi nerambursabile cu caracter de subvenţii","4452","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_4458","Alte sume primite cu caracter de subvenţii","4458","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_446","Alte impozite, taxe şi vărsăminte asimilate","446","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","True"
"pcg_447","Fonduri speciale - taxe şi vărsăminte asimilate","447","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4481","Alte datorii faţă de bugetul statului","4481","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4482","Alte creanţe privind bugetul statului","4482","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_4511","Decontări între entităţile afiliate","4511","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4518","Dobânzi aferente decontărilor între entităţile afiliate","4518","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4531","Decontări cu entităţile asociate şi entităţile controlate în comun","4531","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4538","Dobânzi aferente decontărilor cu entităţile asociate şi entităţile controlate în comun","4538","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4551","Acţionari/asociaţi - conturi curente","4551","account.data_account_type_payable","l10n_ro.ro_chart_template","True"
"pcg_4558","Acţionari/asociaţi - dobânzi la conturi curente","4558","account.data_account_type_payable","l10n_ro.ro_chart_template","True"
"pcg_456","Decontări cu acţionarii/asociaţii privind capitalul","456","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_457","Dividende de plată","457","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4581","Decontări din operaţiuni în participaţie - activ","4581","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_4582","Decontări din operaţiuni în participaţie - pasiv","4582","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_461","Debitori diverşi","461","account.data_account_type_receivable","l10n_ro.ro_chart_template","True"
"pcg_462","Creditori diverşi","462","account.data_account_type_payable","l10n_ro.ro_chart_template","True"
"pcg_4661","Datorii din operaţiuni de fiducie","4661","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4662","Creanţe din operaţiuni de fiducie","4662","account.data_account_type_current_assets","l10n_ro.ro_chart_template","False"
"pcg_471","Cheltuieli înregistrate în avans","471","account.data_account_type_prepayments","l10n_ro.ro_chart_template","False"
"pcg_472","Venituri înregistrate în avans","472","account.data_account_type_prepayments","l10n_ro.ro_chart_template","False"
"pcg_473","Decontări din operaţiuni în curs de clarificare","473","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_4751","Subvenţii guvernamentale pentru investiţii","4751","account.data_account_type_other_income","l10n_ro.ro_chart_template","False"
"pcg_4752","Împrumuturi nerambursabile cu caracter de subvenţii pentru investiţii","4752","account.data_account_type_other_income","l10n_ro.ro_chart_template","False"
"pcg_4753","Donaţii pentru investiţii","4753","account.data_account_type_other_income","l10n_ro.ro_chart_template","False"
"pcg_4754","Plusuri de inventar de natura imobilizărilor","4754","account.data_account_type_other_income","l10n_ro.ro_chart_template","False"
"pcg_4758","Alte sume primite cu caracter de subvenţii pentru investiţii","4758","account.data_account_type_other_income","l10n_ro.ro_chart_template","False"
"pcg_478","Venituri în avans aferente activelor primite prin transfer de la clienţi","478","account.data_account_type_other_income","l10n_ro.ro_chart_template","False"
"pcg_481","Decontări între unitate şi subunităţi","481","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_482","Decontări între subunităţi","482","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_491","Ajustări pentru deprecierea creanţelor - clienţi","491","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_495","Ajustări pentru deprecierea creanţelor - decontări în cadrul grupului şi cu acţionarii/asociaţii","495","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_496","Ajustări pentru deprecierea creanţelor - debitori diverşi","496","account.data_account_type_current_liabilities","l10n_ro.ro_chart_template","False"
"pcg_501","Acţiuni deţinute la entităţile afiliate","501","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_505","Obligaţiuni emise şi răscumpărate","505","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_506","Obligaţiuni","506","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_507","Certificate verzi primite","507","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5081","Alte titluri de plasament","5081","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5088","Dobânzi la obligaţiuni şi titluri de plasament","5088","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5091","Vărsăminte de efectuat pentru acţiunile deţinute la entităţile afiliate","5091","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5092","Vărsăminte de efectuat pentru alte investiţii pe termen scurt","5092","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5112","Cecuri de încasat","5112","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5113","Efecte de încasat","5113","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5114","Efecte remise spre scontare","5114","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5121","Conturi la bănci în lei","5121","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5124","Conturi la bănci în valută","5124","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5125","Sume în curs de decontare","5125","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5186","Dobânzi de plătit","5186","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5187","Dobânzi de încasat","5187","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5191","Credite bancare pe termen scurt","5191","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5192","Credite bancare pe termen scurt nerambursate la scadenţă","5192","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5193","Credite externe guvernamentale","5193","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5194","Credite externe garantate de stat","5194","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5195","Credite externe garantate de bănci","5195","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5196","Credite de la trezoreria statului","5196","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5197","Credite interne garantate de stat","5197","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5198","Dobânzi aferente creditelor bancare pe termen scurt","5198","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5311","Casa în lei","5311","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5314","Casa în valută","5314","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5321","Timbre fiscale şi poştale","5321","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5322","Bilete de tratament şi odihnă","5322","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5323","Tichete şi bilete de călătorie","5323","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5328","Alte valori","5328","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5411","Acreditive în lei","5411","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_5412","Acreditive în valută","5412","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_542","Avansuri de trezorerie","542","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_581","Viramente interne","581","account.data_account_type_liquidity","l10n_ro.ro_chart_template","True"
"pcg_591","Ajustări pentru pierderea de valoare a acţiunilor deţinute la entităţile afiliate","591","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_595","Ajustări pentru pierderea de valoare a obligaţiunilor emise şi răscumpărate","595","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_596","Ajustări pentru pierderea de valoare a obligaţiunilor","596","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_598","Ajustări pentru pierderea de valoare a altor investiţii pe termen scurt şi creanţe asimilate","598","account.data_account_type_liquidity","l10n_ro.ro_chart_template","False"
"pcg_601","Cheltuieli cu materiile prime","601","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6021","Cheltuieli cu materiale auxiliare","6021","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6022","Cheltuieli privind combustibilul","6022","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6023","Cheltuieli privind materialele pentru ambalat","6023","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6024","Cheltuieli privind piesele de schimb","6024","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6025","Cheltuieli privind seminţele şi materialele de plantat","6025","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6026","Cheltuieli privind furajele","6026","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6028","Cheltuieli privind alte materiale consumabile","6028","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_603","Cheltuieli privind materialele de natura obiectelor de inventar","603","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_604","Cheltuieli privind materialele nestocate","604","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_605","Cheltuieli privind energia şi apa","605","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_606","Cheltuieli privind activele biologice de natura stocurilor","606","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"ro_pcg_expense","Cheltuieli privind mărfurile","607","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_608","Cheltuieli privind ambalajele","608","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg609","Reduceri comerciale primite","609","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_611","Cheltuieli cu întreţinerea şi reparaţiile","611","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_612","Cheltuieli cu redevenţele, locaţiile de gestiune şi chiriile","612","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_613","Cheltuieli cu primele de asigurare","613","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_614","Cheltuieli cu studiile şi cercetările","614","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_615","Cheltuieli cu pregătirea personalului","615","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_621","Cheltuieli cu colaboratorii","621","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_622","Cheltuieli privind comisioanele şi onorariile","622","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_623","Cheltuieli de protocol, reclamă şi publicitate","623","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_624","Cheltuieli cu transportul de bunuri şi personal","624","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_625","Cheltuieli cu deplasări, detaşări şi transferări","625","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_626","Cheltuieli poştale şi taxe de telecomunicaţii","626","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_627","Cheltuieli cu serviciile bancare şi asimilate","627","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_628","Alte cheltuieli cu serviciile executate de terţi","628","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6351","Cheltuieli cu alte impozite, taxe şi vărsăminte asimilate","6351","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6352","Cheltuieli cu alte impozite, taxe şi vărsăminte asimilate nedeductibile","6352","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_641","Cheltuieli cu salariile personalului","641","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6421","Cheltuieli cu avantajele în natură acordate salariaţilor","6421","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6422","Cheltuieli cu tichetele acordate salariaţilor","6422","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_643","Cheltuieli cu remunerarea în instrumente de capitaluri proprii","643","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_644","Cheltuieli cu primele reprezentând participarea personalului la profit","644","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6451","Cheltuieli privind contribuţia unităţii la asigurările sociale","6451","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6452","Cheltuieli privind contribuţia unităţii pentru ajutorul de şomaj","6452","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6453","Cheltuieli privind contribuţia angajatorului pentru asigurările sociale de sănătate","6453","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6455","Cheltuieli privind contribuţia unităţii la asigurările de viaţă","6455","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6456","Cheltuieli privind contribuţia unităţii la fondurile de pensii facultative","6456","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6457","Cheltuieli privind contribuţia unităţii la primele de asigurare voluntară de sănătate","6457","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6458","Alte cheltuieli privind asigurările şi protecţia socială","6458","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6511","Cheltuieli ocazionate de constituirea fiduciei","6511","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6512","Cheltuieli din derularea operaţiunilor de fiducie","6512","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6513","Cheltuieli din lichidarea operaţiunilor de fiducie","6513","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_652","Cheltuieli cu protecţia mediului înconjurător","652","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_654","Pierderi din creanţe şi debitori diverşi","654","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_655","Cheltuieli din reevaluarea imobilizărilor corporale","655","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6581","Despăgubiri, amenzi şi penalităţi","6581","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6582","Donaţii acordate","6582","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6583","Cheltuieli privind activele cedate şi alte operaţiuni de capital","6583","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6586","Cheltuieli reprezentând transferuri şi contribuţii datorate în baza unor acte normative speciale","6586","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6587","Cheltuieli privind calamităţile şi alte evenimente similare","6587","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_65881","Alte cheltuieli de exploatare","65881","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_65882","Alte cheltuieli de exploatare nedeductibile","65882","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_663","Pierderi din creanţe legate de participaţii","663","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6641","Cheltuieli privind imobilizările financiare cedate","6641","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6642","Pierderi din investiţiile pe termen scurt cedate","6642","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6651","Diferenţe nefavorabile de curs valutar legate de elementele monetare exprimate în valută","6651","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6652","Diferenţe nefavorabile de curs valutar din evaluarea elementelor monetare care fac parte din investiţia netă într-o entitate străină","6652","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_666","Cheltuieli privind dobânzile","666","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_667","Cheltuieli privind sconturile acordate","667","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_668","Alte cheltuieli financiare","668","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6811","Cheltuieli de exploatare privind amortizarea imobilizărilor","6811","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6812","Cheltuieli de exploatare privind provizioanele","6812","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6813","Cheltuieli de exploatare privind ajustările pentru deprecierea imobilizărilor","6813","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6814","Cheltuieli de exploatare privind ajustările pentru deprecierea activelor circulante","6814","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6817","Cheltuieli de exploatare privind ajustările pentru deprecierea fondului comercial","6817","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6861","Cheltuieli privind actualizarea provizioanelor","6861","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6863","Cheltuieli financiare privind ajustările pentru pierderea de valoare a imobilizărilor financiare","6863","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6864","Cheltuieli financiare privind ajustările pentru pierderea de valoare a activelor circulante","6864","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6865","Cheltuieli financiare privind amortizarea diferențelor aferente titlurilor de stat","6865","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_6868","Cheltuieli financiare privind amortizarea primelor de rambursare a obligaţiunilor şi a altor datorii","6868","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_691","Cheltuieli cu impozitul pe profit","691","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_698","Cheltuieli cu impozitul pe venit şi cu alte impozite care nu apar in elementele de mai sus","698","account.data_account_type_expenses","l10n_ro.ro_chart_template","False"
"pcg_7015","Venituri din vânzarea produselor finite","7015","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7017","Venituri din vânzarea produselor agricole","7017","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7018","Venituri din vânzarea activelor biologice de natura stocurilor","7018","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_702","Venituri din vânzarea semifabricatelor","702","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_703","Venituri din vânzarea produselor reziduale","703","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_704","Venituri din servicii prestate","704","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_705","Venituri din studii şi cercetări","705","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_706","Venituri din redevenţe, locaţii de gestiune şi chirii","706","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"ro_pcg_sale","Venituri din vânzarea mărfurilor","707","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_708","Venituri din activităţi diverse","708","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_709","Reduceri comerciale acordate","709","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_711","Venituri aferente costurilor stocurilor de produse","711","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_712","Venituri aferente costurilor serviciilor în curs de execuţie","712","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_721","Venituri din producţia de imobilizări necorporale","721","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_722","Venituri din producţia de imobilizări corporale","722","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_725","Venituri din producţia de investiţii imobiliare","725","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7411","Venituri din subvenţii de exploatare aferente cifrei de afaceri","7411","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7412","Venituri din subvenţii de exploatare pentru materii prime şi materiale","7412","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7413","Venituri din subvenţii de exploatare pentru alte cheltuieli externe","7413","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7414","Venituri din subvenţii de exploatare pentru plata personalului","7414","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7415","Venituri din subvenţii de exploatare pentru asigurări şi protecţie socială","7415","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7416","Venituri din subvenţii de exploatare pentru alte cheltuieli de exploatare","7416","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7417","Venituri din subvenţii de exploatare în caz de calamităţi şi alte evenimente similare","7417","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7418","Venituri din subvenţii de exploatare pentru dobânda datorată","7418","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7419","Venituri din subvenţii de exploatare aferente altor venituri","7419","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7511","Venituri ocazionate de constituirea fiduciei","7511","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7512","Venituri din derularea operaţiunilor de fiducie","7512","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7513","Venituri din lichidarea operaţiunilor de fiducie","7513","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_754","Venituri din creanţe reactivate şi debitori diverşi","754","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_755","Venituri din reevaluarea imobilizărilor corporale","755","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7581","Venituri din despăgubiri, amenzi şi penalităţi","7581","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7582","Venituri din donaţii primite","7582","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7583","Venituri din vânzarea activelor şi alte operaţiuni de capital","7583","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7584","Venituri din subvenţii pentru investiţii","7584","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7588","Alte venituri din exploatare","7588","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7611","Venituri din acţiuni deţinute la entităţile afiliate","7611","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7612","Venituri din acţiuni deţinute la entităţi asociate","7612","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7613","Venituri din acţiuni deţinute la entităţi controlate în comun","7613","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7615","Venituri din alte imobilizări financiare","7615","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_762","Venituri din investiţii financiare pe termen scurt","762","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7641","Venituri din imobilizări financiare cedate","7641","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7642","Câştiguri din investiţii pe termen scurt cedate","7642","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7651","Diferenţe favorabile de curs valutar legate de elementele monetare exprimate în valută","7651","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7652","Diferenţe favorabile de curs valutar din evaluarea elementelor monetare care fac parte din investiţia netă într-o entitate străină","7652","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_766","Venituri din dobânzi","766","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_767","Venituri din sconturi obţinute","767","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_768","Alte venituri financiare","768","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7812","Venituri din provizioane","7812","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7813","Venituri din ajustări pentru deprecierea imobilizărilor","7813","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7814","Venituri din ajustări pentru deprecierea activelor circulante","7814","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7815","Venituri din fondul comercial negativ","7815","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7863","Venituri financiare din ajustări pentru pierderea de valoare a imobilizărilor financiare","7863","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_7864","Venituri financiare din ajustări pentru pierderea de valoare a activelor circulante","7864","account.data_account_type_revenue","l10n_ro.ro_chart_template","False"
"pcg_8011","Giruri şi garanţii acordate","8011","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_8018","Alte angajamente acordate","8018","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_8021","Giruri şi garanţii primite","8021","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_8028","Alte angajamente primite","8028","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_8031","Imobilizări corporale luate cu chirie","8031","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_8032","Valori materiale primite spre prelucrare sau reparare","8032","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_8033","Valori materiale primite în păstrare sau custodie","8033","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_8034","Debitori scoşi din activ, urmăriţi în continuare","8034","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_8035","Stocuri de natura obiectelor de inventar date în folosinţă","8035","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_8036","Redevenţe, locaţii de gestiune, chirii şi alte datorii asimilate","8036","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_8037","Efecte scontate neajunse la scadenţă","8037","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_8038","Bunuri primite în administrare, concesiune şi cu chirie","8038","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_8039","Alte valori în afara bilanţului","8039","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_8051","Dobânzi de plătit","8051","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_8052","Dobânzi de încasat","8052","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_806","Certificate de emisii de gaze cu efect de seră","806","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_807","Active contingente","807","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_808","Datorii contingente","808","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_809","Creanţe preluate prin cesionare","809","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_891","Bilanţ de deschidere","891","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_892","Bilanţ de închidere","892","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_901","Decontări interne privind cheltuielile","901","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_902","Decontări interne privind producţia obţinută","902","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_903","Decontări interne privind diferenţele de preţ","903","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_921","Cheltuielile activităţii de bază","921","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_922","Cheltuielile activităţilor auxiliare","922","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_923","Cheltuieli indirecte de producţie","923","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_924","Cheltuieli generale de administraţie","924","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_925","Cheltuieli de desfacere","925","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_931","Costul producţiei obţinute","931","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"
"pcg_933","Costul producţiei în curs de execuţie","933","account.data_account_off_sheet","l10n_ro.ro_chart_template","False"

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

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Account Tax Group -->
        <record id="tax_group_tva_0" model="account.tax.group">
            <field name="name">TVA 0%</field>
        </record>
        <record id="tax_group_tva_5" model="account.tax.group">
            <field name="name">TVA 5%</field>
        </record>
        <record id="tax_group_tva_9" model="account.tax.group">
            <field name="name">TVA 9%</field>
        </record>
        <record id="tax_group_tva_19" model="account.tax.group">
            <field name="name">TVA 19%</field>
        </record>
        <record id="tax_group_tva_20" model="account.tax.group">
            <field name="name">TVA 20%</field>
        </record>
        <record id="tax_group_tva_24" model="account.tax.group">
            <field name="name">TVA 24%</field>
        </record>
    </data>
</odoo>

```

## File: data\account_fiscal_position_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Fiscal Position Templates -->

    <record id="fiscal_position_template_1" model="account.fiscal.position.template">
        <field name="name">Regim National</field>
        <field name="chart_template_id" ref="ro_chart_template"/>
    </record>

    <record id="fiscal_position_template_2" model="account.fiscal.position.template">
        <field name="name">Regim Taxare Inversa</field>
        <field name="chart_template_id" ref="ro_chart_template"/>
    </record>

    <record id="fiscal_position_template_3" model="account.fiscal.position.template">
        <field name="name">Regim Intra-Comunitar Bunuri</field>
        <field name="chart_template_id" ref="ro_chart_template"/>
    </record>

    <record id="fiscal_position_template_4" model="account.fiscal.position.template">
        <field name="name">Regim Intra-Comunitar Servicii</field>
        <field name="chart_template_id" ref="ro_chart_template"/>
    </record>

    <record id="fiscal_position_template_5" model="account.fiscal.position.template">
        <field name="name">Regim Scutite</field>
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="note">Pozitia se refera atat la livrari / achizitii intracomunitare cat si extracomunitare care beneficiaza de scutire</field>
    </record>

    <record id="fiscal_position_template_6" model="account.fiscal.position.template">
        <field name="name">Regim Intra-Comunitar Neimpozabile</field>
        <field name="chart_template_id" ref="ro_chart_template"/>
    </record>

    <record id="fiscal_position_template_7" model="account.fiscal.position.template">
        <field name="name">Regim Extra-Comunitar</field>
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="note">Pozitia se refera la livrari extracomunitare care sunt taxabile.</field>
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
    <record id="afptt_inverse_4_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvac_20"/>
        <field name="tax_dest_id" ref="tvati"/>
    </record>
    <record id="afptt_inverse_5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvac_24"/>
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
    <record id="afptt_inverse_9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvad_20"/>
        <field name="tax_dest_id" ref="tvatip20"/>
    </record>
    <record id="afptt_inverse_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_2"/>
        <field name="tax_src_id" ref="tvad_24"/>
        <field name="tax_dest_id" ref="tvatip24"/>
    </record>

    <!-- Intracomunitar Bunuri -->
    <!-- Sales -->
    <record id="afptt_intracom_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvac_00"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracom_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvac_05"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracom_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvac_09"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracom_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvac_19"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracom_4_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvac_20"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracom_5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvac_24"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <!-- Purchases -->
    <record id="afptt_intracom_6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvad_00"/>
        <field name="tax_dest_id" ref="tvati_intrap0"/>
    </record>
    <record id="afptt_intracom_8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvad_05"/>
        <field name="tax_dest_id" ref="tvati_intrap5"/>
    </record>
    <record id="afptt_intracom_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvad_09"/>
        <field name="tax_dest_id" ref="tvati_intrap9"/>
    </record>
    <record id="afptt_intracom_11" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvad_19"/>
        <field name="tax_dest_id" ref="tvati_intrap19"/>
    </record>
    <record id="afptt_intracom_13" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvad_20"/>
        <field name="tax_dest_id" ref="tvati_intrap20"/>
    </record>
    <record id="afptt_intracom_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_3"/>
        <field name="tax_src_id" ref="tvad_24"/>
        <field name="tax_dest_id" ref="tvati_intrap24"/>
    </record>


    <!-- Intracomunitar Servicii -->
    <!-- Sales -->
    <record id="afptt_intracoms_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvac_00"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracoms_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvac_05"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracoms_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvac_09"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracoms_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvac_19"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracoms_4_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvac_20"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <record id="afptt_intracoms_5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvac_24"/>
        <field name="tax_dest_id" ref="tvati_intras"/>
    </record>
    <!-- Purchases -->
    <record id="afptt_intracoms_6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvad_00"/>
        <field name="tax_dest_id" ref="tvati_intrap0"/>
    </record>
    <record id="afptt_intracoms_8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvad_05"/>
        <field name="tax_dest_id" ref="tvati_intrap5"/>
    </record>
    <record id="afptt_intracoms_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvad_09"/>
        <field name="tax_dest_id" ref="tvati_intrap9"/>
    </record>
    <record id="afptt_intracoms_11" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvad_19"/>
        <field name="tax_dest_id" ref="tvati_intrap19"/>
    </record>
    <record id="afptt_intracoms_13" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvad_20"/>
        <field name="tax_dest_id" ref="tvati_intrap20"/>
    </record>
    <record id="afptt_intracoms_14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_4"/>
        <field name="tax_src_id" ref="tvad_24"/>
        <field name="tax_dest_id" ref="tvati_intrap24"/>
    </record>

    <!-- Scutite -->
    <!-- Sales -->
    <record id="afptt_intracomsc_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_5"/>
        <field name="tax_src_id" ref="tvac_00"/>
        <field name="tax_dest_id" ref="tvatisc"/>
    </record>
    <record id="afptt_intracomsc_2" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_5"/>
        <field name="tax_src_id" ref="tvac_05"/>
        <field name="tax_dest_id" ref="tvatisc"/>
    </record>
    <record id="afptt_intracomsc_3" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_5"/>
        <field name="tax_src_id" ref="tvac_09"/>
        <field name="tax_dest_id" ref="tvatisc"/>
    </record>
    <record id="afptt_intracomsc_4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_5"/>
        <field name="tax_src_id" ref="tvac_19"/>
        <field name="tax_dest_id" ref="tvatisc"/>
    </record>
    <record id="afptt_intracomsc_4_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_5"/>
        <field name="tax_src_id" ref="tvac_20"/>
        <field name="tax_dest_id" ref="tvatisc"/>
    </record>
    <record id="afptt_intracomsc_5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_5"/>
        <field name="tax_src_id" ref="tvac_24"/>
        <field name="tax_dest_id" ref="tvatisc"/>
    </record>
    <!-- Purchases -->
    <record id="afptt_intracomsc_6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_5"/>
        <field name="tax_src_id" ref="tvad_00"/>
        <field name="tax_dest_id" ref="tvatisca"/>
    </record>
    <record id="afptt_intracomsc_7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_5"/>
        <field name="tax_src_id" ref="tvad_05"/>
        <field name="tax_dest_id" ref="tvatisca"/>
    </record>
    <record id="afptt_intracomsc_8" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_5"/>
        <field name="tax_src_id" ref="tvad_09"/>
        <field name="tax_dest_id" ref="tvatisca"/>
    </record>
    <record id="afptt_intracomsc_9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_5"/>
        <field name="tax_src_id" ref="tvad_19"/>
        <field name="tax_dest_id" ref="tvatisca"/>
    </record>
    <record id="afptt_intracomsc_9_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_5"/>
        <field name="tax_src_id" ref="tvad_20"/>
        <field name="tax_dest_id" ref="tvatisca"/>
    </record>
    <record id="afptt_intracomsc_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_5"/>
        <field name="tax_src_id" ref="tvad_24"/>
        <field name="tax_dest_id" ref="tvatisca"/>
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
    <record id="afptt_intracomne_4_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvac_20"/>
        <field name="tax_dest_id" ref="tvatine"/>
    </record>
    <record id="afptt_intracomne_5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvac_24"/>
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
    <record id="afptt_intracomne_9_1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvad_20"/>
        <field name="tax_dest_id" ref="tvatinea"/>
    </record>
    <record id="afptt_intracomne_10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_6"/>
        <field name="tax_src_id" ref="tvad_24"/>
        <field name="tax_dest_id" ref="tvatinea"/>
    </record>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tvac_00" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">13</field>
        <field name="description">TVA colectat 0%</field>
        <field name="name">TVA colectat 0%</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_0')],
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
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_0')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="tvac_05" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">12</field>
        <field name="name">TVA colectat 5%</field>
        <field name="description">TVA colectat 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_5')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_line_ids': [ref('account_tax_report_tva_5')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_5')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_line_ids': [ref('account_tax_report_tva_5')],
            }),
        ]"/>
    </record>

    <record id="tvac_09" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">14</field>
        <field name="name">TVA colectat 9%</field>
        <field name="description">TVA colectat 9%</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_9')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_line_ids': [ref('account_tax_report_tva_9')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_9')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_line_ids': [ref('account_tax_report_tva_9')],
            }),
        ]"/>
    </record>

    <record id="tvac_19" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">9</field>
        <field name="name">TVA colectat 19%</field>
        <field name="description">TVA colectat 19%</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_19')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_line_ids': [ref('account_tax_report_tva_19')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_19')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_line_ids': [ref('account_tax_report_tva_19')],
            }),
        ]"/>
    </record>

    <record id="tvac_20" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">10</field>
        <field name="name">TVA colectat 20%</field>
        <field name="description">TVA colectat 20%</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
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
                'account_id': ref('pcg_4427'),
            }),
        ]"/>
    </record>

    <record id="tvac_24" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">11</field>
        <field name="name">TVA colectat 24%</field>
        <field name="description">TVA colectat 24%</field>
        <field name="amount">24</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_24"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_24')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_line_ids': [ref('account_tax_report_tva_24')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_24')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_line_ids': [ref('account_tax_report_tva_24')],
            }),
        ]"/>
    </record>

    <record id="tvad_00" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">23</field>
        <field name="name">TVA deductibil 0%</field>
        <field name="description">TVA deductibil 0%</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_deducbl_0')],
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
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_deducbl_0')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="tvad_05" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">22</field>
        <field name="name">TVA deductibil 5%</field>
        <field name="description">TVA deductibil 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_deducbl_5')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_line_ids': [ref('account_tax_report_ro_tva_decucible_5')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_deducbl_5')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_line_ids': [ref('account_tax_report_ro_tva_decucible_5')],
            }),
        ]"/>
    </record>

    <record id="tvad_09" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">21</field>
        <field name="name">TVA deductibil 9%</field>
        <field name="description">TVA deductibil 9%</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_deducbl_9')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_line_ids': [ref('account_tax_report_ro_tva_decucible_9')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_deducbl_9')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_line_ids': [ref('account_tax_report_ro_tva_decucible_9')],
            }),
        ]"/>
    </record>

    <record id="tvad_19" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">19</field>
        <field name="name">TVA deductibil 19%</field>
        <field name="description">TVA deductibil 19%</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_deducbl_19')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_line_ids': [ref('account_tax_report_ro_tva_decucible_19')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_deducbl_19')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_line_ids': [ref('account_tax_report_ro_tva_decucible_19')],
            }),
        ]"/>
    </record>

    <record id="tvad_20" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">20</field>
        <field name="name">TVA deductibil 20%</field>
        <field name="description">TVA deductibil 20%</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
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
                'account_id': ref('pcg_4426'),
            }),
        ]"/>
    </record>

    <record id="tvad_24" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">23</field>
        <field name="name">TVA deductibil 24%</field>
        <field name="description">TVA deductibil 24%</field>
        <field name="amount">24</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_deducbl_24')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_line_ids': [ref('account_tax_report_ro_tva_decucible_24')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_deducbl_24')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_line_ids': [ref('account_tax_report_ro_tva_decucible_24')],
            }),
        ]"/>
    </record>

    <record id="tvati" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">30</field>
        <field name="name">TVA Taxare Inversa</field>
        <field name="description">TVA Taxare Inversa</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_tx_invrsa')],
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
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_tx_invrsa')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="tvatip00" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">34</field>
        <field name="name">TVA Taxare Inversa 0%</field>
        <field name="description">TVA Taxare Inversa 0%</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_tx_invrsa')],
                'minus_report_line_ids': [ref('account_tax_report_ro_baza_tva_tx_invrsa')],
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
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_tx_invrsa')],
                'plus_report_line_ids': [ref('account_tax_report_ro_baza_tva_tx_invrsa')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="tvatip05" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">33</field>
        <field name="name">TVA Taxare Inversa 5%</field>
        <field name="description">TVA Taxare Inversa 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_tx_invrsa'),
                ref('account_tax_report_ro_baza_tva_tx_invrsa')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_line_ids': [ref('account_tax_report_tva_tx_invrsa')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_line_ids': [ref('account_tax_report_ro_tva_tx_invrsa')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_tx_invrsa'),
                ref('account_tax_report_ro_baza_tva_tx_invrsa')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_line_ids': [ref('account_tax_report_ro_tva_tx_invrsa')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_line_ids': [ref('account_tax_report_tva_tx_invrsa')],
            }),
        ]"/>
    </record>

    <record id="tvatip09" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">32</field>
        <field name="name">TVA Taxare Inversa 9%</field>
        <field name="description">TVA Taxare Inversa 9%</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_ro_baza_tva_tx_invrsa'),
                ref('account_tax_report_baza_tva_tx_invrsa')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_line_ids': [ref('account_tax_report_tva_tx_invrsa')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_line_ids': [ref('account_tax_report_ro_tva_tx_invrsa')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_ro_baza_tva_tx_invrsa'),
                ref('account_tax_report_baza_tva_tx_invrsa')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_line_ids': [ref('account_tax_report_ro_tva_tx_invrsa')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_line_ids': [ref('account_tax_report_tva_tx_invrsa')],
            }),
        ]"/>
    </record>

    <record id="tvatip19" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">32</field>
        <field name="name">TVA Taxare Inversa 19%</field>
        <field name="description">TVA Taxare Inversa 19%</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
            }),
        ]"/>
    </record>

    <record id="tvatip20" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">31</field>
        <field name="name">TVA Taxare Inversa 20%</field>
        <field name="description">TVA Taxare Inversa 20%</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
            }),
        ]"/>
    </record>

    <record id="tvatip24" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">30</field>
        <field name="name">TVA Taxare Inversa 24%</field>
        <field name="description">TVA Taxare Inversa 24%</field>
        <field name="amount">24</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_tx_invrsa'), ref('account_tax_report_ro_baza_tva_tx_invrsa')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_line_ids': [ref('account_tax_report_tva_tx_invrsa')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_line_ids': [ref('account_tax_report_ro_tva_tx_invrsa')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_tx_invrsa'),
                ref('account_tax_report_ro_baza_tva_tx_invrsa')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_line_ids': [ref('account_tax_report_ro_tva_tx_invrsa')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_line_ids': [ref('account_tax_report_tva_tx_invrsa')],
            }),
        ]"/>
    </record>

    <record id="tvatisc" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">40</field>
        <field name="name">TVA Taxare Scutita Livrari</field>
        <field name="description">Scutit-L</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_ro_baza_tva_tx_scutita_vnzari')],
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
                'minus_report_line_ids': [ref('account_tax_report_ro_baza_tva_tx_scutita_vnzari')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="tvatisca" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">41</field>
        <field name="name">TVA Taxare Scutita Achizitii</field>
        <field name="description">Scutit-A</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_ro_baza_tva_tx_scutita_achizt')],
                'minus_report_line_ids': [ref('account_tax_report_ro_baza_tva_tx_scutita_vnzari')],
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
                'minus_report_line_ids': [ref('account_tax_report_ro_baza_tva_tx_scutita_achizt')],
                'plus_report_line_ids': [ref('account_tax_report_ro_baza_tva_tx_scutita_vnzari')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="tvatine" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">50</field>
        <field name="name">TVA Taxare Neimpozabila Livrari</field>
        <field name="description">Neimpozabil-L</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_ro_baza_tva_tx_intracmutr_nempzbl_vnzari')],
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
                'plus_report_line_ids': [ref('account_tax_report_ro_baza_tva_tx_intracmutr_nempzbl_vnzari')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="tvatinea" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">51</field>
        <field name="name">TVA Taxare Neimpozabila Achizitii</field>
        <field name="description">Neimpozabil-A</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_ro_baza_tva_tx_intracmutr_nempzbl_achzti')],
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
                'plus_report_line_ids': [ref('account_tax_report_ro_baza_tva_tx_intracmutr_nempzbl_achzti')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="tvati_intras" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">60</field>
        <field name="name">TVA Intracomunitara Livrari</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
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

    <record id="tvati_intrap0" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">64</field>
        <field name="name">TVA Intracomunitara Achizitii 0%</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_intrcmtr_bnr')],
                'plus_report_line_ids': [ref('account_tax_report_ro_baza_tva_intracmunitr_bnuri')],
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
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_intrcmtr_bnr')],
                'minus_report_line_ids': [ref('account_tax_report_ro_baza_tva_intracmunitr_bnuri')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="tvati_intrap5" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">63</field>
        <field name="name">TVA Intracomunitara Achizitii 5%</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_ro_baza_tva_intracmunitr_bnuri'),
                ref('account_tax_report_baza_tva_intrcmtr_bnr')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_line_ids': [ref('account_tax_report_tva_intracmunitr_bunri')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_line_ids': [ref('account_tax_report_ro_tva_intracmunitr_bunri')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_ro_baza_tva_intracmunitr_bnuri'),
                ref('account_tax_report_baza_tva_intrcmtr_bnr')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_line_ids': [ref('account_tax_report_ro_tva_intracmunitr_bunri')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_line_ids': [ref('account_tax_report_tva_intracmunitr_bunri')],
            }),

        ]"/>
    </record>

    <record id="tvati_intrap9" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">62</field>
        <field name="name">TVA Intracomunitara Achizitii 9%</field>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_baza_tva_intrcmtr_bnr'),
                ref('account_tax_report_ro_baza_tva_intracmunitr_bnuri')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_line_ids': [ref('account_tax_report_tva_intracmunitr_bunri')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_line_ids': [ref('account_tax_report_ro_tva_intracmunitr_bunri')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_baza_tva_intrcmtr_bnr'),
                ref('account_tax_report_ro_baza_tva_intracmunitr_bnuri')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_line_ids': [ref('account_tax_report_ro_tva_intracmunitr_bunri')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_line_ids': [ref('account_tax_report_tva_intracmunitr_bunri')],
            }),
        ]"/>
    </record>

    <record id="tvati_intrap19" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">62</field>
        <field name="name">TVA Intracomunitara Achizitii 19%</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
            }),
        ]"/>
    </record>

    <record id="tvati_intrap20" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">61</field>
        <field name="name">TVA Intracomunitara Achizitii 20%</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
            }),
        ]"/>
    </record>

    <record id="tvati_intrap24" model="account.tax.template">
        <field name="chart_template_id" ref="ro_chart_template"/>
        <field name="sequence">60</field>
        <field name="name">TVA Intracomunitara Achizitii 24%</field>
        <field name="amount">24</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_tax_report_ro_baza_tva_intracmunitr_bnuri'),
                ref('account_tax_report_baza_tva_intrcmtr_bnr')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'plus_report_line_ids': [ref('account_tax_report_tva_intracmunitr_bunri')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'plus_report_line_ids': [ref('account_tax_report_ro_tva_intracmunitr_bunri')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_tax_report_ro_baza_tva_intracmunitr_bnuri'),
                ref('account_tax_report_baza_tva_intrcmtr_bnr')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4426'),
                'minus_report_line_ids': [ref('account_tax_report_ro_tva_intracmunitr_bunri')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_4427'),
                'minus_report_line_ids': [ref('account_tax_report_tva_intracmunitr_bunri')],
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="account_tax_report_baza_tva_clt" model="account.tax.report.line">
        <field name="name">BAZA TVA COLECTAT</field>
        <field name="sequence" eval="1"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_baza_tva_0" model="account.tax.report.line">
        <field name="name">Baza TVA 0%</field>
        <field name="tag_name">Baza TVA 0%</field>
        <field name="code">ROTAX_TVA_colectata_0_Baza</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_clt"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_baza_tva_19" model="account.tax.report.line">
        <field name="name">Baza TVA 19%</field>
        <field name="tag_name">Baza TVA 19%</field>
        <field name="code">ROTAX_TVA_colectata_19_Baza</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_clt"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_baza_tva_24" model="account.tax.report.line">
        <field name="name">Baza TVA 24%</field>
        <field name="tag_name">Baza TVA 24%</field>
        <field name="code">ROTAX_TVA_colectata_24_Baza</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_clt"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_baza_tva_5" model="account.tax.report.line">
        <field name="name">Baza TVA 5%</field>
        <field name="tag_name">Baza TVA 5%</field>
        <field name="code">ROTAX_TVA_colectata_5_Baza</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_clt"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_baza_tva_9" model="account.tax.report.line">
        <field name="name">Baza TVA 9%</field>
        <field name="tag_name">Baza TVA 9%</field>
        <field name="code">ROTAX_TVA_colectata_9_Baza</field>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_clt"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_baza_tva_intrcmtr_bnr" model="account.tax.report.line">
        <field name="name">Baza TVA Intracomunitar Bunuri</field>
        <field name="tag_name">Baza TVA Intracomunitar Bunuri</field>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_clt"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_baza_tva_intrcmtr_srvci" model="account.tax.report.line">
        <field name="name">Baza TVA Intracomunitar Servicii</field>
        <field name="tag_name">Baza TVA Intracomunitar Servicii</field>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_clt"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_baza_tva_tx_invrsa" model="account.tax.report.line">
        <field name="name">Baza TVA Taxare Inversa</field>
        <field name="tag_name">Baza TVA Taxare Inversa</field>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_clt"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_tva_colcta" model="account.tax.report.line">
        <field name="name">TVA COLECTATA</field>
        <field name="sequence" eval="2"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_tva_0" model="account.tax.report.line">
        <field name="name">TVA 0%</field>
        <field name="tag_name">TVA 0% (TVA colectata)</field>
        <field name="code">ROTAX_TVA_colectata_0</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_tva_colcta"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_tva_19" model="account.tax.report.line">
        <field name="name">TVA 19%</field>
        <field name="tag_name">TVA 19% (TVA colectata)</field>
        <field name="code">ROTAX_TVA_colectata_19</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_tva_colcta"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_tva_24" model="account.tax.report.line">
        <field name="name">TVA 24%</field>
        <field name="tag_name">TVA 24% (TVA colectata)</field>
        <field name="code">ROTAX_TVA_colectata_24</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_tva_colcta"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_tva_5" model="account.tax.report.line">
        <field name="name">TVA 5%</field>
        <field name="tag_name">TVA 5% (TVA colectata)</field>
        <field name="code">ROTAX_TVA_colectata_5</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_tva_colcta"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_tva_9" model="account.tax.report.line">
        <field name="name">TVA 9%</field>
        <field name="tag_name">TVA 9% (TVA colectata)</field>
        <field name="code">ROTAX_TVA_colectata_9</field>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="account_tax_report_tva_colcta"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_tva_intracmunitr_bunri" model="account.tax.report.line">
        <field name="name">TVA Intracomunitar Bunuri (TVA colectata)</field>
        <field name="tag_name">TVA Intracomunitar Bunuri (TVA colectata)</field>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="account_tax_report_tva_colcta"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_tva_intra_srvci" model="account.tax.report.line">
        <field name="name">TVA Intracomunitar Servicii (TVA colectata)</field>
        <field name="tag_name">TVA Intracomunitar Servicii (TVA colectata)</field>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="account_tax_report_tva_colcta"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_tva_tx_invrsa" model="account.tax.report.line">
        <field name="name">TVA Taxare Inversa (TVA colectata)</field>
        <field name="tag_name">TVA Taxare Inversa (TVA colectata)</field>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="account_tax_report_tva_colcta"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_baza_tva_deducbl" model="account.tax.report.line">
        <field name="name">BAZA TVA DEDUCTIBILA</field>
        <field name="sequence" eval="3"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_baza_tva_deducbl_0" model="account.tax.report.line">
        <field name="name">Baza TVA 0%</field>
        <field name="tag_name">Baza TVA 0% (deductibila)</field>
        <field name="code">ROTAX_TVA_deductibila_0_Baza</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_deducbl"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_baza_tva_deducbl_19" model="account.tax.report.line">
        <field name="name">Baza TVA 19%</field>
        <field name="tag_name">Baza TVA 19% (deductibila)</field>
        <field name="code">ROTAX_TVA_deductibila_19_Baza</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_deducbl"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_baza_tva_deducbl_24" model="account.tax.report.line">
        <field name="name">Baza TVA 24%</field>
        <field name="tag_name">Baza TVA 24% (deductibila)</field>
        <field name="code">ROTAX_TVA_deductibila_24_Baza</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_deducbl"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_baza_tva_deducbl_5" model="account.tax.report.line">
        <field name="name">Baza TVA 5%</field>
        <field name="tag_name">Baza TVA 5% (deductibila)</field>
        <field name="code">ROTAX_TVA_deductibila_5_Baza</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_deducbl"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_baza_tva_deducbl_9" model="account.tax.report.line">
        <field name="name">Baza TVA 9%</field>
        <field name="tag_name">Baza TVA 9% (deductibila)</field>
        <field name="code">ROTAX_TVA_deductibila_9_Baza</field>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_deducbl"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_baza_tva_intracmunitr_bnuri" model="account.tax.report.line">
        <field name="name">Baza TVA Intracomunitar Bunuri (deductibila)</field>
        <field name="tag_name">Baza TVA Intracomunitar Bunuri (deductibila)</field>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_deducbl"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_baza_tva_intracmunitr_srvci" model="account.tax.report.line">
        <field name="name">Baza TVA Intracomunitar Servicii (deductibila)</field>
        <field name="tag_name">Baza TVA Intracomunitar Servicii (deductibila)</field>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_deducbl"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_baza_tva_tx_invrsa" model="account.tax.report.line">
        <field name="name">Baza TVA Taxare Inversa (deductibila)</field>
        <field name="tag_name">Baza TVA Taxare Inversa (deductibila)</field>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="account_tax_report_baza_tva_deducbl"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_tva_decucible" model="account.tax.report.line">
        <field name="name">TVA DEDUCTIBILA</field>
        <field name="sequence" eval="4"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_tva_decucible_0" model="account.tax.report.line">
        <field name="name">TVA 0%</field>
        <field name="tag_name">TVA 0%</field>
        <field name="code">ROTAX_TVA_deductibila_0</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_ro_tva_decucible"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_tva_decucible_19" model="account.tax.report.line">
        <field name="name">TVA 19% (deductibila)</field>
        <field name="tag_name">TVA 19% (deductibila)</field>
        <field name="code">ROTAX_TVA_deductibila_19</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_ro_tva_decucible"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_tva_decucible_24" model="account.tax.report.line">
        <field name="name">TVA 24% (deductibila)</field>
        <field name="tag_name">TVA 24% (deductibila)</field>
        <field name="code">ROTAX_TVA_deductibila_24</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_ro_tva_decucible"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_tva_decucible_5" model="account.tax.report.line">
        <field name="name">TVA 5% (deductibila)</field>
        <field name="tag_name">TVA 5% (deductibila)</field>
        <field name="code">ROTAX_TVA_deductibila_5</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_ro_tva_decucible"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_tva_decucible_9" model="account.tax.report.line">
        <field name="name">TVA 9% (deductibila)</field>
        <field name="tag_name">TVA 9% (deductibila)</field>
        <field name="code">ROTAX_TVA_deductibila_9</field>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="account_tax_report_ro_tva_decucible"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_tva_intracmunitr_bunri" model="account.tax.report.line">
        <field name="name">TVA Intracomunitar Bunuri (deductibila)</field>
        <field name="tag_name">TVA Intracomunitar Bunuri (deductibila)</field>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="account_tax_report_ro_tva_decucible"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_tva_intracmunitr_srvci" model="account.tax.report.line">
        <field name="name">TVA Intracomunitar Servicii (deductibila)</field>
        <field name="tag_name">TVA Intracomunitar Servicii (deductibila)</field>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="account_tax_report_ro_tva_decucible"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_tva_tx_invrsa" model="account.tax.report.line">
        <field name="name">TVA Taxare Inversa (deductibila)</field>
        <field name="tag_name">TVA Taxare Inversa (deductibila)</field>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="account_tax_report_ro_tva_decucible"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_baza_tva_nxgibl" model="account.tax.report.line">
        <field name="name">BAZA TVA NEEXIGIBILA</field>
        <field name="sequence" eval="5"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_baza_tva_nxgibl_colcta" model="account.tax.report.line">
        <field name="name">Baza TVA Neexigibil Colectat</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_ro_baza_tva_nxgibl"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_baza_tva_nxgibl_deducble" model="account.tax.report.line">
        <field name="name">Baza TVA Neexigibil Deductibil</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_ro_baza_tva_nxgibl"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_tva_nxgibl" model="account.tax.report.line">
        <field name="name">TVA NEEXIGIBILA</field>
        <field name="sequence" eval="6"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_tva_nxgibl_colcta" model="account.tax.report.line">
        <field name="name">TVA Neexigibil Colectat</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_ro_tva_nxgibl"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_tva_nxgibl_deducble" model="account.tax.report.line">
        <field name="name">TVA Neexigibil Deductibil</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_ro_tva_nxgibl"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_baza_tva_tx_invrs" model="account.tax.report.line">
        <field name="name">BAZA TVA TAXARE INVERSA</field>
        <field name="sequence" eval="7"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_baza_tva_tx1_invrs" model="account.tax.report.line">
        <field name="name">Baza TVA Taxare inversa</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_ro_baza_tva_tx_invrs"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_baza_tva_tx_scutita" model="account.tax.report.line">
        <field name="name">BAZA TVA TAXARE SCUTITA</field>
        <field name="sequence" eval="8"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_baza_tva_tx_scutita_achizt" model="account.tax.report.line">
        <field name="name">Baza TVA Taxare Scutita - Achizitii</field>
        <field name="tag_name">Baza TVA Taxare Scutita - Achizitii</field>
        <field name="code">ROTAX_Baza_TVA_Taxare_Scutita_Achizitii</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_ro_baza_tva_tx_scutita"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_baza_tva_tx_scutita_vnzari" model="account.tax.report.line">
        <field name="name">Baza TVA Taxare Scutita - Vanzari</field>
        <field name="tag_name">Baza TVA Taxare Scutita - Vanzari</field>
        <field name="code">ROTAX_Baza_TVA_Taxare_Scutita_Vanzari</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_ro_baza_tva_tx_scutita"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_baza_tva_tx_intracmutr_nempzbl" model="account.tax.report.line">
        <field name="name">BAZA TVA TAXARE INTRACOMUNITARA NEIMPOZABILA</field>
        <field name="sequence" eval="9"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_baza_tva_tx_intracmutr_nempzbl_achzti" model="account.tax.report.line">
        <field name="name">Baza TVA Taxare intracomunitara neimpozabila - Achizitii</field>
        <field name="tag_name">Baza TVA Taxare intracomunitara neimpozabila - Achizitii</field>
        <field name="code">ROTAX_Baza_TVA_Taxare_intracomunitara_neimpozabila_Achizitii</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_ro_baza_tva_tx_intracmutr_nempzbl"/>
        <field name="country_id" ref="base.ro"/>
    </record>

    <record id="account_tax_report_ro_baza_tva_tx_intracmutr_nempzbl_vnzari" model="account.tax.report.line">
        <field name="name">Baza TVA Taxare intracomunitara neimpozabila - Vanzari</field>
        <field name="tag_name">Baza TVA Taxare intracomunitara neimpozabila - Vanzari</field>
        <field name="code">ROTAX_Baza_TVA_Taxare_intracomunitara_neimpozabila_Vanzari</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_ro_baza_tva_tx_intracmutr_nempzbl"/>
        <field name="country_id" ref="base.ro"/>
    </record>

</odoo>
```

## File: data\l10n_ro_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_ro_statements_menu" name="Romania" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_user"/>

    <!-- Chart template -->
    <record id="ro_chart_template" model="account.chart.template">
        <field name="name">Romania - Chart of Accounts</field>
        <field name="bank_account_code_prefix">512</field>
        <field name="cash_account_code_prefix">531</field>
        <field name="transfer_account_code_prefix">581</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.RON"/>
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
        <field name="default_pos_receivable_account_id" ref="ro_pcg_recv_pos" />
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
# Copyright (C) 2015 Forest and Biomass Services Romania (http://www.forbiom.eu).
# Copyright (C) 2011 TOTAL PC SYSTEMS (http://www.erpsystems.ro).
# Copyright (C) 2009 (<http://www.filsystem.ro>)

from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = "res.partner"

    @api.model
    def _auto_init(self):
        res = super(ResPartner, self)._auto_init()
        # Remove constrains for vat, nrc on "commercial entities" because is not mandatory by legislation
        # Even that VAT numbers are unique, the NRC field is not unique, and there are certain entities that
        # doesn't have a NRC number plus the formatting was changed few times, so we cannot have a base rule for
        # checking if available and emmited by the Ministry of Finance, only online on their website.

        self.env.cr.execute("""
            DROP INDEX IF EXISTS res_partner_vat_uniq_for_companies;
            DROP INDEX IF EXISTS res_partner_nrc_uniq_for_companies;
        """)
        return res

    @api.model
    def _commercial_fields(self):
        return super(ResPartner, self)._commercial_fields() + ['nrc']

    nrc = fields.Char(string='NRC', help='Registration number at the Registry of Commerce')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner

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
				<field name="property_account_position_id" position="after">
					<field name="nrc" placeholder="e.g. J12/1234/2012" class="oe_inline"/>
                </field>
			</field>
		</record>
</odoo>

```

