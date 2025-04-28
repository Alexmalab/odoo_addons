# Odoo Module: l10n_pl

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


def _preserve_tag_on_taxes(cr, registry):
    from odoo.addons.account.models.chart_template import preserve_existing_tags_on_taxes
    preserve_existing_tags_on_taxes(cr, registry, 'l10n_pl')

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2009 - now Grzegorz Grzelak grzegorz.grzelak@openglobe.pl

{
    'name' : 'Poland - Accounting',
    'version' : '2.0',
    'author': 'Odoo S.A., Grzegorz Grzelak (OpenGLOBE)',
    'website': 'http://www.openglobe.pl',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the module to manage the accounting chart and taxes for Poland in Odoo.
==================================================================================

To jest moduł do tworzenia wzorcowego planu kont, podatków, obszarów podatkowych i
rejestrów podatkowych. Moduł ustawia też konta do kupna i sprzedaży towarów
zakładając, że wszystkie towary są w obrocie hurtowym.

Niniejszy moduł jest przeznaczony dla odoo 8.0.
Wewnętrzny numer wersji OpenGLOBE 1.02
    """,
    'depends' : [
        'account',
        'base_iban',
        'base_vat',
    ],
    'data': [
              'data/l10n_pl_chart_data.xml',
              'data/account.account.template.csv',
              'data/account.group.template.csv',
              'data/res.country.state.csv',
              'data/l10n_pl_chart_post_data.xml',
              'data/account_tax_group_data.xml',
              'data/account_tax_report_data.xml',
              'data/account_tax_data.xml',
              'data/account_fiscal_position_data.xml',
              'data/account_chart_template_data.xml'
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
"id","name","code","chart_template_id/id","account_type","tag_ids/id","reconcile"
"chart01010000","Grunty własne i prawa wieczystego użytkowania gruntów","01.010.000","l10n_pl.pl_chart_template","asset_fixed",,"False"
"chart01020000","Budynki, lokale i obiekty inżynierii lądowej i wodnej","01.020.000","l10n_pl.pl_chart_template","asset_fixed",,"False"
"chart01030000","Urządzenia techniczne i maszyny","01.030.000","l10n_pl.pl_chart_template","asset_fixed",,"False"
"chart01040000","Środki transportu","01.040.000","l10n_pl.pl_chart_template","asset_fixed",,"False"
"chart01050000","Inne środki trwałe","01.050.000","l10n_pl.pl_chart_template","asset_fixed",,"False"
"chart02010000","Koszty zakończonych prac rozwojowych","02.010.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart02020000","Nabyta wartość firmy","02.020.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart02080000","Inne wartości niematerialne i prawne","02.080.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart02090000","Zaliczki na wartości niematerialne i prawne","02.090.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart03010100","Udziały lub akcje","03.010.100","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart03010200","Inne papiery wartościowe","03.010.200","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart03010300","Udzielone pożyczki","03.010.300","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart03010400","Inne długoterminowe aktywa finansowe","03.010.400","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart03020100","Udziały lub akcje","03.020.100","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart03020200","Inne papiery wartościowe","03.020.200","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart03020300","Udzielone pożyczki","03.020.300","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart03020400","Inne długoterminowe aktywa finansowe","03.020.400","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart03050100","Udziały lub akcje","03.050.100","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart03050200","Inne papiery wartościowe","03.050.200","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart03050300","Udzielone pożyczki","03.050.300","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart03050400","Inne długoterminowe aktywa finansowe","03.050.400","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart03090100","Inne rodzaje długoterminowych aktywów finansowych","03.090.100","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart04010000","Nieruchomości","04.010.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart04020000","Wartości niematerialne i prawne","04.020.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart05010000","Aktywa z tytułu odroczonego podatku dochodowego","05.010.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart05020000","Inne rozliczenia międzyokresowe","05.020.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart06010000","Od jednostek powiązanych","06.010.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart06020000","Od pozostałych jednostek","06.020.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart07010100","Odpisy umorzeniowe wartości gruntów i prawa wieczystego użytkowania gruntów","07.010.100","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart07010200","Odpisy umorzeniowe budynków, lokali i obiektów inżynierii lądowej i wodnej","07.010.200","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart07010300","Odpisy umorzeniowe urządzeń technicznych i maszyn","07.010.300","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart07010400","Odpisy umorzeniowe środków transportu","07.010.400","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart07010500","Odpisy umorzeniowe innych środków trwałych","07.010.500","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart07010900","Odpisy umorzeniowe ulepszeń obcych środków trwałych","07.010.900","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart07020100","Odpisy umorzeniowe zakończonych prac rozwojowych","07.020.100","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart07020200","Odpisy umorzeniowe wartości firmy","07.020.200","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart07020300","Odpisy umorzeniowe innych wartości niematerialnych i prawnych","07.020.300","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart07030100","Odpisy umorzeniowe inwestycji w nieruchomości","07.030.100","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart07030200","Odpisy umorzeniowe inwestycji w wartości niematerialne i prawne","07.030.200","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart08010000","Inwestycje budowy środka trwałego","08.010.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart08020000","Ulepszenia środka trwałego","08.020.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart08030000","Nakłady na budowę środka trwałego","08.030.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart08040000","Zaliczki na środki trwałe w budowie","08.040.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart08050000","Ulepszenia obcych środków trwałych","08.050.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart09010000","Odpisy aktualizujące udziały i akcje w obcych jednostkach","09.010.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart09020000","Odpisy aktualizujące lokaty","09.020.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart09030000","Odpisy aktualizujące udzielone pożyczki długoterminowe","09.030.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart09090000","Odpisy aktualizujące inne rodzaje długoterminowych aktywów finansowych","09.090.000","l10n_pl.pl_chart_template","asset_non_current",,"False"
"chart10010000","Kasa krajowych środków pieniężnych","10.010.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart10020000","Kasa zagranicznych środków pieniężnych","10.020.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart10090000","Pozostałe inne środki pieniężne","10.090.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart14000000","Inne aktywa pieniężne","14.000.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart15010100","Udziały lub akcje","15.010.100","l10n_pl.pl_chart_template","asset_current",,"False"
"chart15010200","Inne papiery wartościowe","15.010.200","l10n_pl.pl_chart_template","asset_current",,"False"
"chart15010300","Udzielone pożyczki","15.010.300","l10n_pl.pl_chart_template","asset_current",,"False"
"chart15010400","Inne krótkoterminowe aktywa finansowe","15.010.400","l10n_pl.pl_chart_template","asset_current",,"False"
"chart15020100","Udziały lub akcje","15.020.100","l10n_pl.pl_chart_template","asset_current",,"False"
"chart15020200","Inne papiery wartościowe","15.020.200","l10n_pl.pl_chart_template","asset_current",,"False"
"chart15020300","Udzielone pożyczki","15.020.300","l10n_pl.pl_chart_template","asset_current",,"False"
"chart15020400","Inne krótkoterminowe aktywa finansowe","15.020.400","l10n_pl.pl_chart_template","asset_current",,"False"
"chart15050000","Odpisy aktualizujące krótkoterminowe aktywa finansowe","15.050.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart18000000","Krótkoterminowe rozliczenia międzyokresowe","18.000.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart20000000","Rozrachunki z odbiorcami","20.000.000","l10n_pl.pl_chart_template","asset_receivable",,"True"
"chart20000100","Rozrachunki z odbiorcami (PoS)","20.000.100","l10n_pl.pl_chart_template","asset_receivable",,"True"
"chart20010000","Zaliczki od klientów","20.010.000","l10n_pl.pl_chart_template","liability_current",,"False"
"chart21000000","Rozrachunki z dostawcami","21.000.000","l10n_pl.pl_chart_template","liability_payable",,"True"
"chart22010000","Rozrachunki z urzędem skarbowym z tytułu VAT","22.010.000","l10n_pl.pl_chart_template","liability_current",,"False"
"chart22020100","Rozliczenie naliczonego VAT 22%","22.020.100","l10n_pl.pl_chart_template","asset_current",,"False"
"chart22020200","Rozliczenie naliczonego VAT 7%","22.020.200","l10n_pl.pl_chart_template","asset_current",,"False"
"chart22020300","Rozliczenie naliczonego VAT 3%","22.020.300","l10n_pl.pl_chart_template","asset_current",,"False"
"chart22020400","Rozliczenie naliczonego VAT 23%","22.020.400","l10n_pl.pl_chart_template","asset_current",,"False"
"chart22020500","Rozliczenie naliczonego VAT 8%","22.020.500","l10n_pl.pl_chart_template","asset_current",,"False"
"chart22020600","Rozliczenie naliczonego VAT 5%","22.020.600","l10n_pl.pl_chart_template","asset_current",,"False"
"chart22030100","Rozliczenie należnego VAT 22%","22.030.100","l10n_pl.pl_chart_template","liability_current",,"False"
"chart22030200","Rozliczenie należnego VAT 7%","22.030.200","l10n_pl.pl_chart_template","liability_current",,"False"
"chart22030300","Rozliczenie należnego VAT 3%","22.030.300","l10n_pl.pl_chart_template","liability_current",,"False"
"chart22030400","Rozliczenie należnego VAT 23%","22.030.400","l10n_pl.pl_chart_template","liability_current",,"False"
"chart22030500","Rozliczenie należnego VAT 8%","22.030.500","l10n_pl.pl_chart_template","liability_current",,"False"
"chart22030600","Rozliczenie należnego VAT 5%","22.030.600","l10n_pl.pl_chart_template","liability_current",,"False"
"chart22040000","Pozostałe rozrachunki publicznoprawne","22.040.000","l10n_pl.pl_chart_template","liability_current",,"False"
"chart22040100","Pozostałe rozrachunki z urzędem skarbowym","22.040.100","l10n_pl.pl_chart_template","liability_current",,"False"
"chart22040200","Rozrachunki publicznoprawne z urzędem miasta/gminy","22.040.200","l10n_pl.pl_chart_template","liability_current",,"False"
"chart22040300","Rozrachunki publicznoprawne z urzędem celnym","22.040.300","l10n_pl.pl_chart_template","liability_current",,"False"
"chart22040400","Rozrachunki publicznoprawne z ZUS","22.040.400","l10n_pl.pl_chart_template","liability_current",,"False"
"chart22040500","Rozrachunki publicznoprawne z PFRON","22.040.500","l10n_pl.pl_chart_template","liability_current",,"False"
"chart23010000","Rozrachunki z tytułu wynagrodzeń","23.010.000","l10n_pl.pl_chart_template","liability_current",,"False"
"chart23020000","Rozrachunki z tytułu pożyczek udzielonych pracownikom","23.020.000","l10n_pl.pl_chart_template","liability_current",,"False"
"chart23090000","Inne rozrachunki z pracownikami","23.090.000","l10n_pl.pl_chart_template","liability_current",,"False"
"chart24010100","Pożyczki otrzymane","24.010.100","l10n_pl.pl_chart_template","liability_non_current",,"False"
"chart24010200","Pożyczki udzielone","24.010.200","l10n_pl.pl_chart_template","asset_current",,"False"
"chart24020100","Rozliczenie niedoborów","24.020.100","l10n_pl.pl_chart_template","asset_current",,"False"
"chart24020200","Rozliczenie nadwyżek","24.020.200","l10n_pl.pl_chart_template","liability_current",,"False"
"chart24030100","Rozrachunki z tytułu wpłat na kapitał zakładowy","24.030.100","l10n_pl.pl_chart_template","liability_current",,"False"
"chart24030200","Rozrachunki z tytułu wkładów niepieniężnych na kapitał zakładowy","24.030.200","l10n_pl.pl_chart_template","liability_current",,"False"
"chart24030300","Rozrachunki z tytułu podwyższenia kapitału ze środków własnych spółki","24.030.300","l10n_pl.pl_chart_template","liability_current",,"False"
"chart24030400","Rozrachunki z tytułu umorzenia udziałów własnych","24.030.400","l10n_pl.pl_chart_template","liability_current",,"False"
"chart24050000","Rozrachunki wewnątrzzakładowe","24.050.000","l10n_pl.pl_chart_template","liability_current",,"False"
"chart24060000","Należności dochodzone na drodze sądowej","24.060.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart24070000","Rozrachunki z tytułu dywidend","24.070.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart24080000","Rozrachunki z tytułu dopłat i zwrotu dopłat","24.080.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart24090000","Pozostałe rozrachunki","24.090.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart28000000","Odpisy aktualizujące rozrachunki","28.000.000","l10n_pl.pl_chart_template","liability_current",,"False"
"chart29010000","Należności warunkowe","29.010.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart29020000","Zobowiązania warunkowe","29.020.000","l10n_pl.pl_chart_template","liability_current",,"False"
"chart29030000","Weksle obce dyskontowane lub indosowane","29.030.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart30010000","Rozliczenie zakupu materiałów","30.010.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart30020000","Rozliczenie zakupu towarów","30.020.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart30030000","Rozliczenie zakupu usług obcych","30.030.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart30040000","Rozliczenie zakupu składników aktywów trwałych","30.040.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart30050000","Rozliczenie wartości materiałów i towarów w drodze","30.050.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart30060000","Wartości dostaw niefakturowanych","30.060.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart30070000","Opłaty manipulacyjne policzone przez Urząd Celny","30.070.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart30080000","Niedobory, szkody i nadwyżki w transporcie","30.080.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart30090000","Reklamacje faktur dostawców","30.090.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart31010000","Materiały","31.010.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart31060000","Opakowania","31.060.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart31090000","Materiały w przerobie","31.090.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart33010000","Towary w hurcie","33.010.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart33020000","Towary w detalu","33.020.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart33030000","Towary w zakładach gastronomicznych","33.030.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart33040000","Towary skupu","33.040.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart33080000","Towary poza jednostką","33.080.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart33090000","Nieruchomości i prawa majątkowe przeznaczone do obrotu","33.090.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart34010000","Odchylenia od cen ewidencyjnych materiałów","34.010.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart34020100","Odchylenia od cen ewidencyjnych towarów w hurcie","34.020.100","l10n_pl.pl_chart_template","asset_current",,"False"
"chart34020200","Odchylenia od cen ewidencyjnych towarów w detalu","34.020.200","l10n_pl.pl_chart_template","asset_current",,"False"
"chart34020300","Odchylenia od cen ewidencyjnych towarów w zakładach gastronomicznych","34.020.300","l10n_pl.pl_chart_template","asset_current",,"False"
"chart34020400","Odchylenia od cen ewidencyjnych towarów skupu","34.020.400","l10n_pl.pl_chart_template","asset_current",,"False"
"chart34060000","Odchylenia od cen ewidencyjnych opakowań","34.060.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart34070000","Odchylenia z tytułu aktualizacji wartości zapasów materiałów i towarów","34.070.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart39000000","Zapasy obce","39.000.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart40010100","Zużycie surowców do wytwarzania produktów","40.010.100","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart40010200","Zużycie energii","40.010.200","l10n_pl.pl_chart_template","expense",,"False"
"chart40010300","Zużycie paliwa do środków transportu","40.010.300","l10n_pl.pl_chart_template","expense",,"False"
"chart40010400","Zużycie materiałów biurowych","40.010.400","l10n_pl.pl_chart_template","expense",,"False"
"chart40010900","Zużycie innych materiałów","40.010.900","l10n_pl.pl_chart_template","expense",,"False"
"chart40020100","Usługi celne","40.020.100","l10n_pl.pl_chart_template","expense",,"False"
"chart40020200","Usługi telekomunikacyjne","40.020.200","l10n_pl.pl_chart_template","expense",,"False"
"chart40020300","Usługi pocztowe","40.020.300","l10n_pl.pl_chart_template","expense",,"False"
"chart40020400","Usługi kurierskie i transportowe","40.020.400","l10n_pl.pl_chart_template","expense",,"False"
"chart40020500","Analizy sanitarne","40.020.500","l10n_pl.pl_chart_template","expense",,"False"
"chart40020600","Usługi graficzne i drukarskie","40.020.600","l10n_pl.pl_chart_template","expense",,"False"
"chart40020700","Usługi remontowe","40.020.700","l10n_pl.pl_chart_template","expense",,"False"
"chart40020900","Pozostałe usługi","40.020.900","l10n_pl.pl_chart_template","expense",,"False"
"chart40030100","Podatek od nieruchomości","40.030.100","l10n_pl.pl_chart_template","expense",,"False"
"chart40030200","Podatek od środków transportowych","40.030.200","l10n_pl.pl_chart_template","expense",,"False"
"chart40030300","Podatek akcyzowy","40.030.300","l10n_pl.pl_chart_template","expense",,"False"
"chart40030400","VAT niepodlegający odliczeniu","40.030.400","l10n_pl.pl_chart_template","expense",,"False"
"chart40030500","Opłaty i prowizje bankowe","40.030.500","l10n_pl.pl_chart_template","expense",,"False"
"chart40030600","Opłaty sądowe, prawnicze i notarialne","40.030.600","l10n_pl.pl_chart_template","expense",,"False"
"chart40030700","Opłaty skarbowe","40.030.700","l10n_pl.pl_chart_template","expense",,"False"
"chart40030800","Koncesje","40.030.800","l10n_pl.pl_chart_template","expense",,"False"
"chart40030900","Pozostałe podatki i opłaty","40.030.900","l10n_pl.pl_chart_template","expense",,"False"
"chart40040100","Wynagrodzenia pracowników","40.040.100","l10n_pl.pl_chart_template","expense",,"False"
"chart40040200","Wynagrodzenia osób doraźnie zatrudnionych","40.040.200","l10n_pl.pl_chart_template","expense",,"False"
"chart40050100","Składki na ubezpieczenia społeczne, FP, FGŚP","40.050.100","l10n_pl.pl_chart_template","expense",,"False"
"chart40050200","Odpisy na zakładowy fundusz świadczeń socjalnych lub świadczenia urlopowe","40.050.200","l10n_pl.pl_chart_template","expense",,"False"
"chart40050900","Pozostałe świadczenia","40.050.900","l10n_pl.pl_chart_template","expense",,"False"
"chart41000000","Świadczenia na rzecz pracowników","41.000.000","l10n_pl.pl_chart_template","expense",,"False"
"chart42000000","Amortyzacja","42.000.000","l10n_pl.pl_chart_template","expense_depreciation",,"False"
"chart43000000","Pozostałe koszty rodzajowe","43.000.000","l10n_pl.pl_chart_template","expense",,"False"
"chart49010000","Nie podlegające rozliczeniu w czasie","49.010.000","l10n_pl.pl_chart_template","expense",,"False"
"chart49020000","Przypadające na przyszłe okresy","49.020.000","l10n_pl.pl_chart_template","expense",,"False"
"chart49030000","Koszty zgromadzone","49.030.000","l10n_pl.pl_chart_template","expense",,"False"
"chart49040000","Koszty nie wliczane do wartości sprzedaży","49.040.000","l10n_pl.pl_chart_template","expense",,"False"
"chart50010000","Rozliczone koszty działalności","50.010.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart50020000","Koszty nie zakończonych długotrwałych usług","50.020.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart50030000","Straty związane z wykonaniem długotrwałych usług","50.030.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart50040000","Koszty utrzymania hurtowni","50.040.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart52010000","Koszty utrzymania punktów sprzedaży detalicznej","52.010.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart52030000","Koszty sprzedaży wyrobów","52.030.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart53010000","Świadczenia usług transportowych","53.010.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart53020000","Pozostałe koszty","53.020.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart55010000","Koszty zarządzania jednostką","55.010.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart55020000","Świadczenia usług na potrzeby reprezentacji i reklamy","55.020.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart58000000","Rozliczenie kosztów działalności","58.000.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart60010100","Produkty gotowe w magazynie","60.010.100","l10n_pl.pl_chart_template","asset_current",,"False"
"chart60010200","Produkty gotowe poza jednostką","60.010.200","l10n_pl.pl_chart_template","asset_current",,"False"
"chart60020000","Półprodukty","60.020.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart62010000","Odchylenia od cen ewidencyjnych produktów","62.010.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart62020000","Odchylenia z tytułu aktualizacji wartości zapasów produktów","62.020.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart64010000","Czynne rozliczenia międzyokresowe kosztów","64.010.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart64020000","Bierne rozliczenia międzyokresowe kosztów","64.020.000","l10n_pl.pl_chart_template","liability_current",,"False"
"chart65010000","Aktywa z tytułu odroczonego podatku dochodowego","65.010.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart65020000","Inne rozliczenia międzyokresowe","65.020.000","l10n_pl.pl_chart_template","liability_current",,"False"
"chart70010000","Sprzedaż produktów na kraj","70.010.000","l10n_pl.pl_chart_template","income","account.account_tag_operating","False"
"chart70020000","Sprzedaż produktów na eksport","70.020.000","l10n_pl.pl_chart_template","income","account.account_tag_operating","False"
"chart70030000","Sprzedaż usług na kraj","70.030.000","l10n_pl.pl_chart_template","income","account.account_tag_operating","False"
"chart70040000","Sprzedaż usług na eksport","70.040.000","l10n_pl.pl_chart_template","income","account.account_tag_operating","False"
"chart70110000","Koszt własny sprzedaży produktów na kraj","70.110.000","l10n_pl.pl_chart_template","expense_direct_cost","account.account_tag_operating","False"
"chart70120000","Koszt własny sprzedaży produktów na eksport","70.120.000","l10n_pl.pl_chart_template","expense_direct_cost","account.account_tag_operating","False"
"chart70130000","Koszt własny sprzedaży usług na kraj","70.130.000","l10n_pl.pl_chart_template","expense_direct_cost","account.account_tag_operating","False"
"chart70140000","Koszt własny sprzedaży usług na eksport","70.140.000","l10n_pl.pl_chart_template","expense_direct_cost","account.account_tag_operating","False"
"chart73010000","Sprzedaż hurtowa towarów","73.010.000","l10n_pl.pl_chart_template","income","account.account_tag_operating","False"
"chart73020000","Sprzedaż detaliczna towarów","73.020.000","l10n_pl.pl_chart_template","income","account.account_tag_operating","False"
"chart73030000","Sprzedaż wysyłkowa towarów","73.030.000","l10n_pl.pl_chart_template","income","account.account_tag_operating","False"
"chart73040000","Prowizja komisowa","73.040.000","l10n_pl.pl_chart_template","income_other","account.account_tag_operating","False"
"chart73110000","Wartość sprzedanych towarów w sprzedaży hurtowej","73.110.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart73120000","Wartość sprzedanych towarów w sprzedaży detalicznej","73.120.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart73130000","Wartość sprzedanych towarów w sprzedaży wysyłkowej","73.130.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart73140000","Prowizja komisowa","73.140.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart74010000","Sprzedaż materiałów","74.010.000","l10n_pl.pl_chart_template","income","account.account_tag_operating","False"
"chart74020000","Sprzedaż opakowań","74.020.000","l10n_pl.pl_chart_template","income","account.account_tag_operating","False"
"chart74030000","Sprzedaż odpadów","74.030.000","l10n_pl.pl_chart_template","income","account.account_tag_operating","False"
"chart74110000","Wartość w cenach zakupu sprzedanych materiałów","74.110.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart74120000","Wartość w cenach zakupu sprzedanych opakowań","74.120.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart74130000","Wartość w cenach zakupu sprzedanych odpadów","74.130.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart75010000","Kwoty należne ze sprzedaży aktywów finansowych","75.010.000","l10n_pl.pl_chart_template","income_other","account.account_tag_investing","False"
"chart75020000","Kwoty należne z tytułu dywidend","75.020.000","l10n_pl.pl_chart_template","income_other","account.account_tag_financing","False"
"chart75030000","Otrzymane odsetki","75.030.000","l10n_pl.pl_chart_template","income_other","account.account_tag_financing","False"
"chart75040000","Przychody ze zbycia inwestycji","75.040.000","l10n_pl.pl_chart_template","income","account.account_tag_investing","False"
"chart75050000","Aktualizacja wartości inwestycji.przychody","75.050.000","l10n_pl.pl_chart_template","income_other","account.account_tag_investing","False"
"chart75060000","Dodatnie różnice kursu walut","75.060.000","l10n_pl.pl_chart_template","income_other","account.account_tag_financing","False"
"chart75070000","Pozostałe przychody finansowe","75.070.000","l10n_pl.pl_chart_template","income_other","account.account_tag_financing","False"
"chart75110000","Wartość sprzedanych inwestycji","75.110.000","l10n_pl.pl_chart_template","expense","account.account_tag_investing","False"
"chart75120000","Odpisy z tytułu utraty wartości inwestycji.koszty","75.120.000","l10n_pl.pl_chart_template","expense","account.account_tag_investing","False"
"chart75130000","Odsetki zapłacone","75.130.000","l10n_pl.pl_chart_template","expense","account.account_tag_financing","False"
"chart75140000","Ujemne różnice kursu walut","75.140.000","l10n_pl.pl_chart_template","expense","account.account_tag_financing","False"
"chart75190000","Pozostałe koszty finansowe","75.190.000","l10n_pl.pl_chart_template","expense","account.account_tag_financing","False"
"chart76010000","Przychody ze zbycia niefinansowych aktywów trwałych","76.010.000","l10n_pl.pl_chart_template","income","account.account_tag_investing","False"
"chart76020000","Otrzymane dotacje","76.020.000","l10n_pl.pl_chart_template","income","account.account_tag_financing","False"
"chart76030000","Przychody z usług socjalnych","76.030.000","l10n_pl.pl_chart_template","income","account.account_tag_financing","False"
"chart76040000","Przychody ze wzrostu wartości niefinansowych aktywów trwałych","76.040.000","l10n_pl.pl_chart_template","income","account.account_tag_investing","False"
"chart76090000","Inne pozostałe przychody operacyjne","76.090.000","l10n_pl.pl_chart_template","income","account.account_tag_operating","False"
"chart76110000","Wartość sprzedanych niefinansowych aktywów trwałych","76.110.000","l10n_pl.pl_chart_template","expense","account.account_tag_investing","False"
"chart76120000","Dotacje przekazane","76.120.000","l10n_pl.pl_chart_template","expense","account.account_tag_financing","False"
"chart76130000","Odpisy z tytułu utraty wartości aktywów niefinansowych","76.130.000","l10n_pl.pl_chart_template","expense","account.account_tag_investing","False"
"chart76140000","Inne pozostałe koszty operacyjne","76.140.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart77000000","Zyski nadzwyczajne","77.000.000","l10n_pl.pl_chart_template","income",,"False"
"chart77100000","Straty nadzwyczajne","77.100.000","l10n_pl.pl_chart_template","expense",,"False"
"chart79110000","Koszt wytworzenia wyrobów gotowych wydanych do własnych sklepów","79.110.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart79120000","Koszt wytworzenia świadczeń na rzecz środków trwałych w budowie","79.120.000","l10n_pl.pl_chart_template","expense","account.account_tag_investing","False"
"chart79130000","Koszt wytworzenia zakończonych prac rozwojowych","79.130.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart79140000","Koszt wytworzenia produktów uznanych za niedobory","79.140.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart79150000","Koszt zaniechania określonego rodzaju działalności","79.150.000","l10n_pl.pl_chart_template","expense","account.account_tag_operating","False"
"chart80000000","Kapitał podstawowy","80.000.000","l10n_pl.pl_chart_template","equity",,"False"
"chart80120000","Należne wpłaty na kapitał podstawowy (wielkość ujemna)","80.120.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart80130000","Udziały (akcje) własne (wielkość ujemna)","80.130.000","l10n_pl.pl_chart_template","asset_current",,"False"
"chart80190000","Odpisy z zysku netto w ciągu roku obrotowego (wielkość ujemna)","80.190.000","l10n_pl.pl_chart_template","equity",,"False"
"chart81100000","Kapitał zapasowy","81.100.000","l10n_pl.pl_chart_template","equity",,"False"
"chart81200000","Kapitał rezerwowy","81.200.000","l10n_pl.pl_chart_template","equity",,"False"
"chart81300000","Kapitał z aktualizacji wyceny","81.300.000","l10n_pl.pl_chart_template","equity",,"False"
"chart81400000","Kapitały wydzielone w jednostce statutowej i zakładach (oddziałach) samodzielnie sporządzających bilans","81.400.000","l10n_pl.pl_chart_template","equity",,"False"
"chart82000000","Rozliczenia wyniku finansowego","82.000.000","l10n_pl.pl_chart_template","equity",,"False"
"chart83010000","Rezerwa z tytułu odroczonego podatku dochodowego","83.010.000","l10n_pl.pl_chart_template","liability_current",,"False"
"chart83020100","Rezerwa długoterminowa na świadczenia emerytalne i podobne","83.020.100","l10n_pl.pl_chart_template","liability_non_current",,"False"
"chart83020200","Rezerwa krótkoterminowa na świadczenia emerytalne i podobne","83.020.200","l10n_pl.pl_chart_template","liability_current",,"False"
"chart83090100","Pozostałe rezerwy długoterminowe","83.090.100","l10n_pl.pl_chart_template","liability_non_current",,"False"
"chart83090200","Pozostałe rezerwy krótkoterminowe","83.090.200","l10n_pl.pl_chart_template","liability_current",,"False"
"chart84010000","Ujemna wartość firmy","84.010.000","l10n_pl.pl_chart_template","liability_non_current",,"False"
"chart84020100","Inne rozliczenia międzyokresowe długoterminowe","84.020.100","l10n_pl.pl_chart_template","liability_non_current",,"False"
"chart84020200","Inne rozliczenia międzyokresowe krótkoterminowe","84.020.200","l10n_pl.pl_chart_template","liability_current",,"False"
"chart85010000","Zakładowy fundusz świadczeń socjalnych","85.010.000","l10n_pl.pl_chart_template","liability_current",,"False"
"chart85020100","Zakładowy fundusz rehabilitacji osób niepełnosprawnych","85.020.100","l10n_pl.pl_chart_template","liability_current",,"False"
"chart85020200","Fundusz nagród","85.020.200","l10n_pl.pl_chart_template","liability_current",,"False"
"chart85020300","Fundusz na remont zasobów mieszkaniowych","85.020.300","l10n_pl.pl_chart_template","liability_current",,"False"
"chart86000000","Wynik finansowy","86.000.000","l10n_pl.pl_chart_template","equity_unaffected",,"False"
"chart87010000","Podatek dochodowy od osób prawnych","87.010.000","l10n_pl.pl_chart_template","expense","account.account_tag_financing","False"
"chart87020000","Inne obowiązkowe obciążenia wyniku finansowego","87.020.000","l10n_pl.pl_chart_template","expense","account.account_tag_financing","False"

```

## File: data\account.group.template.csv

```csv
id,code_prefix_start,code_prefix_end,name,chart_template_id/id
pl_group_0,0,,"Aktywa trwałe",l10n_pl.pl_chart_template
pl_group_01,01,,"Środki trwałe",l10n_pl.pl_chart_template
pl_group_02,02,,"Wartości niematerialne i prawne",l10n_pl.pl_chart_template
pl_group_03,03,,"Długoterminowe aktywa finansowe",l10n_pl.pl_chart_template
pl_group_04,04,,"Inwestycje w nieruchomości i prawa",l10n_pl.pl_chart_template
pl_group_05,05,,"Długoterminowe rozliczenia międzyokresowe",l10n_pl.pl_chart_template
pl_group_06,06,,"Należności długoterminowe",l10n_pl.pl_chart_template
pl_group_07,07,,"Odpisy umorzeniowe środków trwałych, wartości niematerialnych i prawnych oraz inwestycji w nieruchomości i prawa",l10n_pl.pl_chart_template
pl_group_08,08,,"Środki trwałe w budowie",l10n_pl.pl_chart_template
pl_group_09,09,,"Odpisy aktualizujące długoterminowych aktywów finansowych",l10n_pl.pl_chart_template
pl_group_1,1,,"Środki pieniężne, rachunki bankowe oraz inne krótkoterminowe aktywa finansowe",l10n_pl.pl_chart_template
pl_group_10,10,,"Kasa",l10n_pl.pl_chart_template
pl_group_15,15,,"Krótkoterminowe aktywa finansowe",l10n_pl.pl_chart_template
pl_group_18,18,,"Krótkoterminowe rozliczenia międzyokresowe",l10n_pl.pl_chart_template
pl_group_2,2,,"Rozrachunki i roszczenia",l10n_pl.pl_chart_template
pl_group_20,20,,"Rozrachunki z odbiorcami",l10n_pl.pl_chart_template
pl_group_21,21,,"Rozrachunki z dostawcami",l10n_pl.pl_chart_template
pl_group_22,22,,"Rozrachunki publicznoprawne",l10n_pl.pl_chart_template
pl_group_23,23,,"Rozrachunki z tytułu wynagrodzeń i innych świadczeń na rzecz pracowników",l10n_pl.pl_chart_template
pl_group_24,24,,"Pozostałe rozrachunki",l10n_pl.pl_chart_template
pl_group_28,28,,"Odpisy aktualizujące rozrachunki",l10n_pl.pl_chart_template
pl_group_29,29,,"Rozrachunki pozabilansowe",l10n_pl.pl_chart_template
pl_group_3,3,,"Materiały i towary",l10n_pl.pl_chart_template
pl_group_30,30,,"Rozliczenie zakupu",l10n_pl.pl_chart_template
pl_group_31,31,,"Materiały i opakowania",l10n_pl.pl_chart_template
pl_group_33,33,,"Towary",l10n_pl.pl_chart_template
pl_group_34,34,,"Odchylenia od cen ewidencyjnych materiałów i towarów",l10n_pl.pl_chart_template
pl_group_39,39,,"Zapasy obce",l10n_pl.pl_chart_template
pl_group_4,4,,"Koszty według rodzajów i ich rozliczenie",l10n_pl.pl_chart_template
pl_group_40,40,,"Koszty według rodzajów",l10n_pl.pl_chart_template
pl_group_401,401,,"Zużycie materiałów i energii",l10n_pl.pl_chart_template
pl_group_402,402,,"Usługi obce",l10n_pl.pl_chart_template
pl_group_403,403,,"Podatki i opłaty",l10n_pl.pl_chart_template
pl_group_404,404,,"Wynagrodzenia",l10n_pl.pl_chart_template
pl_group_405,405,,"Ubezpieczenia społeczne i inne świadczenia",l10n_pl.pl_chart_template
pl_group_41,41,,"Świadczenia na rzecz pracowników",l10n_pl.pl_chart_template
pl_group_42,42,,"Amortyzacja",l10n_pl.pl_chart_template
pl_group_43,43,,"Pozostałe koszty rodzajowe",l10n_pl.pl_chart_template
pl_group_49,49,,"Rozliczenie kosztów",l10n_pl.pl_chart_template
pl_group_5,5,,"Koszty według typów działalności i ich rozliczenie",l10n_pl.pl_chart_template
pl_group_6,6,,"Produkty i rozliczenia międzyokresowe",l10n_pl.pl_chart_template
pl_group_60,60,,"Produkty gotowe i półprodukty",l10n_pl.pl_chart_template
pl_group_62,62,,"Odchylenia od cen ewidencyjnych produktów",l10n_pl.pl_chart_template
pl_group_64,64,,"Rozliczenia międzyokresowe kosztów",l10n_pl.pl_chart_template
pl_group_65,65,,"Pozostałe rozliczenia międzyokresowe",l10n_pl.pl_chart_template
pl_group_7,7,,"Przychody i koszty związane z ich osiągnięciem",l10n_pl.pl_chart_template
pl_group_70,70,,"Przychody i koszty związane ze sprzedażą produktów",l10n_pl.pl_chart_template
pl_group_700,700,,"Sprzedaż produktów",l10n_pl.pl_chart_template
pl_group_701,701,,"Koszt sprzedanych produktów",l10n_pl.pl_chart_template
pl_group_73,73,,"Przychody i koszty związane ze sprzedażą towarów",l10n_pl.pl_chart_template
pl_group_730,730,,"Sprzedaż towarów",l10n_pl.pl_chart_template
pl_group_731,731,,"Wartość sprzedanych towarów w cenach zakupu (nabycia)",l10n_pl.pl_chart_template
pl_group_74,74,,"Przychody i koszty związane ze sprzedażą materiałów i opakowań",l10n_pl.pl_chart_template
pl_group_740,740,,"Sprzedaż materiałów i opakowań",l10n_pl.pl_chart_template
pl_group_741,741,,"Wartość sprzedanych materiałów i opakowań",l10n_pl.pl_chart_template
pl_group_75,75,,"Przychody i koszty finansowe",l10n_pl.pl_chart_template
pl_group_750,750,,"Przychody finansowe",l10n_pl.pl_chart_template
pl_group_751,751,,"Koszty finansowe",l10n_pl.pl_chart_template
pl_group_76,76,,"Pozostałe przychody i koszty operacyjne",l10n_pl.pl_chart_template
pl_group_760,760,,"Pozostałe przychody operacyjne",l10n_pl.pl_chart_template
pl_group_761,761,,"Pozostałe koszty operacyjne",l10n_pl.pl_chart_template
pl_group_77,77,,"Zyski i straty nadzwyczajne",l10n_pl.pl_chart_template
pl_group_79,79,,"Obroty wewnętrzne",l10n_pl.pl_chart_template
pl_group_8,8,,"Kapitały (fundusze) własne, fundusze specjalne i wynik finansowy",l10n_pl.pl_chart_template
pl_group_80,80,,"Kapitał (fundusz) podstawowy",l10n_pl.pl_chart_template
pl_group_811,811,,"Kapitał (fundusz) zapasowy",l10n_pl.pl_chart_template
pl_group_812,812,,"Kapitał (fundusz) rezerwowy",l10n_pl.pl_chart_template
pl_group_813,813,,"Kapitał (fundusz) z aktualizacji wyceny",l10n_pl.pl_chart_template
pl_group_814,814,,"Kapitały (fundusze) wydzielone w jednostce statutowej i zakładach (oddziałach) samodzielnie sporządzających bilans",l10n_pl.pl_chart_template
pl_group_82,82,,"Rozliczenie wyniku finansowego",l10n_pl.pl_chart_template
pl_group_83,83,,"Rezerwy",l10n_pl.pl_chart_template
pl_group_84,84,,"Rozliczenia międzyokresowe przychodów",l10n_pl.pl_chart_template
pl_group_85,85,,"Fundusze specjalne",l10n_pl.pl_chart_template
pl_group_86,86,,"Wynik finansowy",l10n_pl.pl_chart_template
pl_group_87,87,,"Podatek dochodowy i inne obowiązkowe obciążenia wyniku finansowego",l10n_pl.pl_chart_template

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_pl.pl_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_fiscal_position_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
    
        <!-- Fiscal Position Templates -->
        
        <record id="fiscal_position_template_1" model="account.fiscal.position.template">
            <field name="sequence">1</field>
            <field name="name">Kraj</field>
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_id" ref="base.pl"/>
        </record>

        <record id="fiscal_position_template_4" model="account.fiscal.position.template">
            <field name="sequence">2</field>
            <field name="name">Wspólnota Prywatny</field>
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>

        <record id="fiscal_position_template_2" model="account.fiscal.position.template">
            <field name="sequence">3</field>
            <field name="name">Wspólnota</field>
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>

        <record id="fiscal_position_template_3" model="account.fiscal.position.template">
            <field name="sequence">4</field>
            <field name="name">Import/Eksport</field>
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="auto_apply" eval="True"/>
        </record>
    
    <!-- Fiscal Position Tax Templates -->

        <!-- European Union -->
        <!-- Sales -->

        <record id="fiscal_position_tax_template_1a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vs_kraj_23" />
            <field name="tax_dest_id" ref="vs_unia" />
        </record>

        <record id="fiscal_position_tax_template_1" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vs_kraj_22" />
            <field name="tax_dest_id" ref="vs_unia" />
        </record>

        <record id="fiscal_position_tax_template_2a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vs_kraj_8" />
            <field name="tax_dest_id" ref="vs_unia" />
        </record>

        <record id="fiscal_position_tax_template_2" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vs_kraj_7" />
            <field name="tax_dest_id" ref="vs_unia" />
        </record>

        <record id="fiscal_position_tax_template_3a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vs_kraj_5" />
            <field name="tax_dest_id" ref="vs_unia" />
        </record>

        <record id="fiscal_position_tax_template_3" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vs_kraj_3" />
            <field name="tax_dest_id" ref="vs_unia" />
        </record>

        <record id="fiscal_position_tax_template_4" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vs_kraj_0" />
            <field name="tax_dest_id" ref="vs_unia" />
        </record>

        <record id="fiscal_position_tax_template_5" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vs_kraj_zw" />
            <field name="tax_dest_id" ref="vs_unia" />
        </record>

        <record id="fiscal_position_tax_template_6a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vs_kraj_usl_23" />
            <field name="tax_dest_id" ref="vs_dostu" />
        </record>

        <record id="fiscal_position_tax_template_6" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vs_kraj_usl_22" />
            <field name="tax_dest_id" ref="vs_dostu" />
        </record>

        <!-- European Union -->
        <!-- Purchase -->

        <record id="fiscal_position_tax_template_11a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vz_kraj_23" />
            <field name="tax_dest_id" ref="vz_unia" />
        </record>

        <record id="fiscal_position_tax_template_11" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vz_kraj_22" />
            <field name="tax_dest_id" ref="vz_unia" />
        </record>

        <record id="fiscal_position_tax_template_12a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vz_kraj_8" />
            <field name="tax_dest_id" ref="vz_unia" />
        </record>

        <record id="fiscal_position_tax_template_12" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vz_kraj_7" />
            <field name="tax_dest_id" ref="vz_unia" />
        </record>

        <record id="fiscal_position_tax_template_13a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vz_kraj_5" />
            <field name="tax_dest_id" ref="vz_unia" />
        </record>

        <record id="fiscal_position_tax_template_13" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vz_kraj_3" />
            <field name="tax_dest_id" ref="vz_unia" />
        </record>

        <record id="fiscal_position_tax_template_14" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vz_kraj_0" />
            <field name="tax_dest_id" ref="vz_unia" />
        </record>

        <record id="fiscal_position_tax_template_15" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vz_kraj_zw" />
            <field name="tax_dest_id" ref="vz_unia" />
        </record>

        <record id="fiscal_position_tax_template_16a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vz_kraj_usl_23" />
            <field name="tax_dest_id" ref="vz_nabu" />
        </record>

        <record id="fiscal_position_tax_template_16" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vz_kraj_usl_22" />
            <field name="tax_dest_id" ref="vz_nabu" />
        </record>


        <!-- World Export/Import -->
        <!-- Sales -->

        <record id="fiscal_position_tax_template_21a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vs_kraj_23" />
            <field name="tax_dest_id" ref="vs_eksp_tow" />
        </record>

        <record id="fiscal_position_tax_template_21" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vs_kraj_22" />
            <field name="tax_dest_id" ref="vs_eksp_tow" />
        </record>

        <record id="fiscal_position_tax_template_22a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vs_kraj_8" />
            <field name="tax_dest_id" ref="vs_eksp_tow" />
        </record>

        <record id="fiscal_position_tax_template_22" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vs_kraj_7" />
            <field name="tax_dest_id" ref="vs_eksp_tow" />
        </record>

        <record id="fiscal_position_tax_template_23a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vs_kraj_5" />
            <field name="tax_dest_id" ref="vs_eksp_tow" />
        </record>

        <record id="fiscal_position_tax_template_23" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vs_kraj_3" />
            <field name="tax_dest_id" ref="vs_eksp_tow" />
        </record>

        <record id="fiscal_position_tax_template_24" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vs_kraj_0" />
            <field name="tax_dest_id" ref="vs_eksp_tow" />
        </record>

        <record id="fiscal_position_tax_template_25" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vs_kraj_zw" />
            <field name="tax_dest_id" ref="vs_eksp_tow" />
        </record>

        <record id="fiscal_position_tax_template_26a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vs_kraj_usl_23" />
            <field name="tax_dest_id" ref="vs_ekspu" />
        </record>

        <record id="fiscal_position_tax_template_26" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vs_kraj_usl_22" />
            <field name="tax_dest_id" ref="vs_ekspu" />
        </record>

        <!-- World Export/Import -->
        <!-- Purchase -->

        <record id="fiscal_position_tax_template_31a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vz_kraj_23" />
            <field name="tax_dest_id" ref="vz_imp_tow" />
        </record>

        <record id="fiscal_position_tax_template_31" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vz_kraj_22" />
            <field name="tax_dest_id" ref="vz_imp_tow" />
        </record>

        <record id="fiscal_position_tax_template_32a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vz_kraj_8" />
            <field name="tax_dest_id" ref="vz_imp_tow" />
        </record>

        <record id="fiscal_position_tax_template_32" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vz_kraj_7" />
            <field name="tax_dest_id" ref="vz_imp_tow" />
        </record>

        <record id="fiscal_position_tax_template_33a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vz_kraj_5" />
            <field name="tax_dest_id" ref="vz_imp_tow" />
        </record>

        <record id="fiscal_position_tax_template_33" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vz_kraj_3" />
            <field name="tax_dest_id" ref="vz_imp_tow" />
        </record>

        <record id="fiscal_position_tax_template_34" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vz_kraj_0" />
            <field name="tax_dest_id" ref="vz_imp_tow" />
        </record>

        <record id="fiscal_position_tax_template_35" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vz_kraj_zw" />
            <field name="tax_dest_id" ref="vz_imp_tow" />
        </record>

        <record id="fiscal_position_tax_template_36a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vz_kraj_usl_23" />
            <field name="tax_dest_id" ref="vz_impu" />
        </record>

        <record id="fiscal_position_tax_template_36" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_3"  />
            <field name="tax_src_id" ref="vz_kraj_usl_22" />
            <field name="tax_dest_id" ref="vz_impu" />
        </record>

    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="gold_tag" model="account.account.tag">
            <field name="name">Gold</field>
            <field name="applicability">taxes</field>
            <field name="country_id" ref="base.pl"/>
        </record>


        <!-- VAT domestic sale-->
        <record id="vs_kraj_23" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT-23%</field>
            <field name="description">V23</field>
            <field name="amount">23</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="sequence" eval="0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_kraj_22_lub_23_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_kraj_22_lub_23_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_23"/>
        </record>
        <record id="vs_kraj_22" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT-22%</field>
            <field name="description">V22</field>
            <field name="amount">22</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_kraj_22_lub_23_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030100'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_kraj_22_lub_23_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030100'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_22"/>
        </record>
        <record id="vs_kraj_8" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT-8%</field>
            <field name="description">V8</field>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_kraj_7_lub_8_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030500'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_7_lub_8_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_kraj_7_lub_8_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030500'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_7_lub_8_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_8"/>
        </record>
        <record id="vs_kraj_7" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT-7%</field>
            <field name="description">V7</field>
            <field name="amount">7</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_kraj_7_lub_8_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030200'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_7_lub_8_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_kraj_7_lub_8_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030200'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_7_lub_8_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_7"/>
        </record>
        <record id="vs_kraj_5" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT-5%</field>
            <field name="description">V5</field>
            <field name="amount">5</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_kraj_3_lub_5_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030600'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_3_lub_5_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_kraj_3_lub_5_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030600'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_3_lub_5_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_5"/>
        </record>
        <record id="vs_kraj_3" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT-3%</field>
            <field name="description">V3</field>
            <field name="amount">3</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_kraj_3_lub_5_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030300'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_3_lub_5_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_kraj_3_lub_5_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030300'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_3_lub_5_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_3"/>
        </record>
        <record id="vs_kraj_0" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT-0%</field>
            <field name="description">V0</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslugi_kraj_0_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslugi_kraj_0_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vs_zwrot_podróżnych_0" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">0% Zwrot podróżnych</field>
            <field name="description">0%</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                  (0,0, {
                      'repartition_type': 'base',
                      'plus_report_expression_ids': [ref('account_tax_report_line_towary_art_129_tag')],
                  }),
                  (0,0, {
                      'repartition_type': 'tax',
                  }),
                        ]"/>
                    <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                  (0,0, {
                      'repartition_type': 'base',
                      'minus_report_expression_ids': [ref('account_tax_report_line_towary_art_129_tag')],
                  }),
                  (0,0, {
                      'repartition_type': 'tax',
                  }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vs_kraj_zw" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT-ZW</field>
            <field name="description">VZW</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_kraj_zwolnione_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_kraj_zwolnione_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vs_kraj_zw_gold" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">0% Exempt Złota</field>
            <field name="description">0%</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_kraj_zwolnione_tag')],
              'tag_ids': [(4, ref('gold_tag'))],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_kraj_zwolnione_tag')],
              'tag_ids': [(4, ref('gold_tag'))],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vs_kraj_usl_23" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT usł-23%</field>
            <field name="description">VU23</field>
            <field name="amount">23</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">service</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_kraj_22_lub_23_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_kraj_22_lub_23_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_23"/>
        </record>
        <record id="vs_kraj_usl_22" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT usł-22%</field>
            <field name="description">VU22</field>
            <field name="amount">22</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">service</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_kraj_22_lub_23_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030100'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_kraj_22_lub_23_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22030100'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_22"/>
        </record>
        <!-- VAT domestic purchase -->
        <record id="vz_kraj_23" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT naliczony-23%</field>
            <field name="description">Z23</field>
            <field name="amount">23</field>
            <field name="amount_type">percent</field>
            <field name="sequence" eval="0"/>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_vat_23"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
      ]"/>
        </record>
        <record id="vz_kraj_22" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT naliczony-22%</field>
            <field name="description">Z22</field>
            <field name="amount">22</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_vat_22"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020100'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020100'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
      ]"/>
        </record>
        <record id="vz_kraj_8" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT naliczony-8%</field>
            <field name="description">Z8</field>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020500'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020500'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_8"/>
        </record>
        <record id="vz_kraj_7" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT naliczony-7%</field>
            <field name="description">Z7</field>
            <field name="amount">7</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020200'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020200'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_7"/>
        </record>
        <record id="vz_kraj_5" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT naliczony-5%</field>
            <field name="description">Z5</field>
            <field name="amount">5</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020600'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020600'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_5"/>
        </record>
        <record id="vz_kraj_3" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT naliczony-3%</field>
            <field name="description">Z3</field>
            <field name="amount">3</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020300'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020300'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_3"/>
        </record>
        <record id="vz_kraj_0" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT naliczony-0%</field>
            <field name="description">Z0</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vz_kraj_zw" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT naliczony-ZW</field>
            <field name="description">ZZW</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vz_kraj_usl_23" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT usł nalicz-23%</field>
            <field name="description">ZU23</field>
            <field name="amount">23</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_scope">service</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_23"/>
        </record>
        <record id="vz_kraj_usl_22" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT usł nalicz-22%</field>
            <field name="description">ZU22</field>
            <field name="amount">22</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_scope">service</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020100'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020100'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_22"/>
        </record>
        <!-- Vehicle leasing -->
        <!-- ================================================ -->
        <record id="vp_leas_sale" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT - leasing pojazdu(sale)</field>
            <field name="description">VLP</field>
            <field name="amount">23.00</field>
            <field name="sequence" eval="1"/>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': 60,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': 40,
              'repartition_type': 'tax',
              'account_id': ref('chart76140000'),
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': 60,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': 40,
              'repartition_type': 'tax',
              'account_id': ref('chart76140000'),
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_23"/>
        </record>
        <record id="vp_leas_purchase" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">VAT - leasing pojazdu(purchase)</field>
            <field name="description">VLP</field>
            <field name="amount">23.00</field>
            <field name="sequence" eval="1"/>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': 60,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': 40,
              'repartition_type': 'tax',
              'account_id': ref('chart76140000'),
          }),
      ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': 60,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': 40,
              'repartition_type': 'tax',
              'account_id': ref('chart76140000'),
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_23"/>
        </record>
        <!-- Steel trade -->
        <record id="vs_stal" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">Sprzedaż stali</field>
            <field name="description">VST</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">consu</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_podatnik_nabywca_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_podatnik_nabywca_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vz_stal" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">Zakup stali</field>
            <field name="description">ZST</field>
            <field name="amount">23.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_scope">consu</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag'), ref('account_tax_report_line_podatnik_nabywca_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_podatnik_nabywca_tag')],
          }),
      ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag'), ref('account_tax_report_line_podatnik_nabywca_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_podatnik_nabywca_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <!-- European Union -->
        <!-- =========================================================== -->
        <record id="vs_unia" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">Dost tow. unia</field>
            <field name="description">UDT</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">consu</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_dostawa_towarow_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_dostawa_towarow_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vs_unia_i42" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">0% EU I_42</field>
            <field name="description">0% I_42</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">consu</field>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('account_tax_report_line_dostawa_towarow_tag'), ref('account_tax_report_line_intracom_procedure_i_42_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('account_tax_report_line_dostawa_towarow_tag'), ref('account_tax_report_line_intracom_procedure_i_42_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vs_unia_triangular" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">0% EU T 2nd Payer</field>
            <field name="description">0% EU T 2nd Payer</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">consu</field>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('account_tax_report_line_nabycie_towarow_tag'), ref('account_tax_report_line_triangular_2nd_payer_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('account_tax_report_line_nabycie_towarow_tag'), ref('account_tax_report_line_triangular_2nd_payer_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vs_unia_i63" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">0% EU I_63</field>
            <field name="description">0% I_63</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">consu</field>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('account_tax_report_line_dostawa_towarow_tag'), ref('account_tax_report_line_intracom_procedure_i_63_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('account_tax_report_line_dostawa_towarow_tag'), ref('account_tax_report_line_intracom_procedure_i_63_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vz_unia_triangular" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">0% EU T 2nd Payer</field>
            <field name="description">0% EU T 2nd Payer</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_scope">consu</field>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('account_tax_report_line_poza_kraj_tag'), ref('account_tax_report_line_triangular_buyer_2nd_payer_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'minus_report_expression_ids': [ref('account_tax_report_line_poza_kraj_tag'), ref('account_tax_report_line_triangular_buyer_2nd_payer_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vz_unia" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">Nab tow unia</field>
            <field name="description">UNT</field>
            <field name="amount">23.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_scope">consu</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag'), ref('account_tax_report_line_nabycie_towarow_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_nabycie_towarow_tag')],
          }),
      ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag'), ref('account_tax_report_line_nabycie_towarow_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_nabycie_towarow_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vs_dostu" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">Świad Usł</field>
            <field name="description">UDU</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">service</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslugi_art_100_1_4_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslugi_art_100_1_4_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vz_nabu" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">Nab Usł</field>
            <field name="description">UNU</field>
            <field name="amount">23.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_scope">service</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag'), ref('account_tax_report_line_art_28b_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_art_28b_tag')],
          }),
      ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag'), ref('account_tax_report_line_art_28b_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_art_28b_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <!-- Export / Import -->
        <!-- ================================================ -->
        <record id="vs_eksp_tow" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">Eksp Tow</field>
            <field name="description">EXT</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">consu</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_eksport_towarow_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
      ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_eksport_towarow_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vz_imp_tow" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">Imp Tow</field>
            <field name="description">IMT</field>
            <field name="amount">23.0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_scope">consu</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag'), ref('account_tax_report_line_art_33a_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_art_33a_tag')],
          }),
      ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag'), ref('account_tax_report_line_art_33a_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_art_33a_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vs_ekspu" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">Eksp Usł</field>
            <field name="description">EXU</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">service</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_poza_kraj_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_poza_kraj_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
        <record id="vz_impu" model="account.tax.template">
            <field name="chart_template_id" ref="pl_chart_template"/>
            <field name="name">Imp Usł</field>
            <field name="description">IMU</field>
            <field name="amount">23.00</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_scope">service</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'plus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag'), ref('account_tax_report_line_import_uslug_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_import_uslug_tag')],
          }),
      ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'repartition_type': 'base',
              'minus_report_expression_ids': [ref('account_tax_report_line_uslug_pozostalych_tag'), ref('account_tax_report_line_import_uslug_tag')],
          }),
          (0,0, {
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_expression_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych_tag')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'plus_report_expression_ids': [ref('account_tax_report_line_podatek_import_uslug_tag')],
          }),
      ]"/>
            <field name="tax_group_id" ref="tax_group_vat_0"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_vat_23" model="account.tax.group">
            <field name="name">VAT 23%</field>
            <field name="country_id" ref="base.pl"/>
        </record>
        <record id="tax_group_vat_22" model="account.tax.group">
            <field name="name">VAT 22%</field>
            <field name="country_id" ref="base.pl"/>
        </record>
        <record id="tax_group_vat_8" model="account.tax.group">
            <field name="name">VAT 8%</field>
            <field name="country_id" ref="base.pl"/>
        </record>
        <record id="tax_group_vat_7" model="account.tax.group">
            <field name="name">VAT 7%</field>
            <field name="country_id" ref="base.pl"/>
        </record>
        <record id="tax_group_vat_5" model="account.tax.group">
            <field name="name">VAT 5%</field>
            <field name="country_id" ref="base.pl"/>
        </record>
        <record id="tax_group_vat_3" model="account.tax.group">
            <field name="name">VAT 3%</field>
            <field name="country_id" ref="base.pl"/>
        </record>
        <record id="tax_group_vat_0" model="account.tax.group">
            <field name="name">VAT 0%</field>
            <field name="country_id" ref="base.pl"/>
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
        <field name="country_id" ref="base.pl"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_razem_c" model="account.report.line">
                <field name="name">Podstawa - Razem C</field>
                <field name="aggregation_formula">PLTAXC_01_10.balance + PLTAXC_02_11.balance + PLTAXC_03_13.balance + PLTAXC_04_15.balance + PLTAXC_05_17.balance + PLTAXC_06_19.balance + PLTAXC_07_21.balance + PLTAXC_08_22.balance + PLTAXC_09_23.balance + PLTAXC_10_25.balance + PLTAXC_11_27.balance + PLTAXC_12_31.balance</field>
                <field name="sequence">10</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_kraj_zwolnione" model="account.report.line">
                        <field name="name">Podstawa - Dostawa towarów/usług, kraj, zwolnione</field>
                        <field name="sequence">20</field>
                        <field name="code">PLTAXC_01_10</field>
                        <field name="sequence">20</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_kraj_zwolnione_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_10</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_poza_kraj" model="account.report.line">
                        <field name="name">Podstawa - Dostawa towarów/usług, poza kraj</field>
                        <field name="sequence">30</field>
                        <field name="code">PLTAXC_02_11</field>
                        <field name="sequence">30</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_poza_kraj_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_11</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_uslugi_art_100_1_4" model="account.report.line">
                                <field name="name">Podstawa - W tym usługi art 100.1.4</field>
                                <field name="sequence">40</field>
                                <field name="code">PLTAXC_02a_12</field>
                                <field name="sequence">40</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_uslugi_art_100_1_4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_12</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_triangular_buyer_2nd_payer" model="account.report.line">
                                <field name="name">- Transakcja trójstronna 2. płatnik VAT</field>
                                <field name="code">PLTAXC_02a_12_Triangular</field>
                                <field name="sequence">50</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_triangular_buyer_2nd_payer_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Triangular Purchase</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_uslugi_kraj_0" model="account.report.line">
                        <field name="name">Podstawa - Dostawa towarów/usług, kraj, 0%</field>
                        <field name="code">PLTAXC_03_13</field>
                        <field name="sequence">60</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_uslugi_kraj_0_tag" model="account.report.expression">
                                <field name="label">tag</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_13</field>
                            </record>
                            <record id="account_tax_report_line_uslugi_kraj_0_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">aggregation</field>
                                <field name="formula">PLTAXC_03_13.tag + PLTAXC_03_14.balance</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_towary_art_129" model="account.report.line">
                                <field name="name">Podstawa - W tym towary art 129</field>
                                <field name="code">PLTAXC_03_14</field>
                                <field name="sequence">70</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_towary_art_129_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_14</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_kraj_3_lub_5" model="account.report.line">
                        <field name="name">Podstawa - Dostawa towarów/usług, kraj, 3% lub 5%</field>
                        <field name="code">PLTAXC_04_15</field>
                        <field name="sequence">80</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_kraj_3_lub_5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_15</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_kraj_7_lub_8" model="account.report.line">
                        <field name="name">Podstawa - Dostawa towarów/usług, kraj, 7% lub 8%</field>
                        <field name="code">PLTAXC_05_17</field>
                        <field name="sequence">90</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_kraj_7_lub_8_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_17</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_kraj_22_lub_23" model="account.report.line">
                        <field name="name">Podstawa - Dostawa towarów/usług, kraj, 22% lub 23%</field>
                        <field name="code">PLTAXC_06_19</field>
                        <field name="sequence">100</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_kraj_22_lub_23_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_19</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_dostawa_towarow" model="account.report.line">
                        <field name="name">Podstawa - Wew-wspól dostawa towarów</field>
                        <field name="code">PLTAXC_07_21</field>
                        <field name="sequence">110</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_dostawa_towarow_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_21</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_intracom_procedure_i_42" model="account.report.line">
                                <field name="name">- w ramach procedury celnej 42</field>
                                <field name="code">PLTAXC_07_21_I42</field>
                                <field name="sequence">120</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_intracom_procedure_i_42_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I_42</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_intracom_procedure_i_63" model="account.report.line">
                                <field name="name">- w ramach procedury celnej 63</field>
                                <field name="code">PLTAXC_07_21_I63</field>
                                <field name="sequence">130</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_intracom_procedure_i_63_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">I_63</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_eksport_towarow" model="account.report.line">
                        <field name="name">Podstawa - Eksport towarów</field>
                        <field name="code">PLTAXC_08_22</field>
                        <field name="sequence">140</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_eksport_towarow_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_22</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_nabycie_towarow" model="account.report.line">
                        <field name="name">Podstawa - Wewn-wspól. nabycie towarów</field>
                        <field name="code">PLTAXC_09_23</field>
                        <field name="sequence">150</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_nabycie_towarow_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_23</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_triangular_2nd_payer" model="account.report.line">
                                <field name="name">- Transakcja trójstronna 2. płatnik VAT</field>
                                <field name="code">PLTAXC_09_23_Triangular</field>
                                <field name="sequence">160</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_triangular_2nd_payer_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">Triangular Sale</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_art_33a" model="account.report.line">
                        <field name="name">Podstawa - Import towarów art. 33a</field>
                        <field name="code">PLTAXC_10_25</field>
                        <field name="sequence">170</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_art_33a_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_25</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_import_uslug" model="account.report.line">
                        <field name="name">Podstawa - Import usług</field>
                        <field name="code">PLTAXC_11_27</field>
                        <field name="sequence">180</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_import_uslug_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_27</field>
                            </record>
                        </field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_art_28b" model="account.report.line">
                                <field name="name">Podstawa - W tym nabycie wg art 28b</field>
                                <field name="code">PLTAXC_11a_29</field>
                                <field name="sequence">190</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_art_28b_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_29</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_podatnik_nabywca" model="account.report.line">
                        <field name="name">Podstawa - Dostawa towarów, podatnik nabywca</field>
                        <field name="code">PLTAXC_12_31</field>
                        <field name="sequence">200</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_podatnik_nabywca_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_31</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_razem_d" model="account.report.line">
                <field name="name">Podstawa - Razem D</field>
                <field name="aggregation_formula">PODSTAWA__NABYCIE_TOWAROW_I_USLUG_STRWALE.balance + PLTAXD_02_41a.balance</field>
                <field name="sequence">210</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_uslug_s_trwale" model="account.report.line">
                        <field name="name">Podstawa - Nabycie towarów i usług ś.trwałe</field>
                        <field name="code">PODSTAWA__NABYCIE_TOWAROW_I_USLUG_STRWALE</field>
                        <field name="sequence">220</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_uslug_s_trwale_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_40</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_uslug_pozostalych" model="account.report.line">
                        <field name="name">Podstawa - Nabycie towarów i usług pozostałych</field>
                        <field name="code">PLTAXD_02_41a</field>
                        <field name="sequence">230</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_uslug_pozostalych_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">K_42</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_do_przeniesienia" model="account.report.line">
                <field name="name">Podatek - Do przeniesienia</field>
                <field name="code">PLTAX</field>
                <field name="sequence">240</field>
                <field name="expression_ids">
                    <record id="account_tax_report_line_do_przeniesienia_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">PLTAXD.balance + PLTAX_49.balance + PLTAX_50.balance - PLTAXC.balance</field>
                    </record>
                    <record id="tax_report_line_do_przeniesienia_carryover" model="account.report.expression">
                        <field name="label">_carryover_balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">PLTAX.balance</field>
                        <field name="subformula">if_above(EUR(0))</field>
                        <field name="carryover_target">PODATEK__NADWYZKA_Z_POPRZEDNIEJ_DEKLARACJI._applied_carryover_balance</field>
                    </record>
                </field>
                <field name="children_ids">
                    <record id="account_tax_report_line_podatek_razem_c" model="account.report.line">
                        <field name="name">Podatek - Razem C</field>
                        <field name="code">PLTAXC</field>
                        <field name="aggregation_formula">PLTAXC_04_16.balance + PLTAXC_05_18.balance + PLTAXC_06_20.balance + PLTAXC_09_24.formula + PLTAXC_10_26.balance + PLTAXC_11_28.balance + PLTAXC_12_32.balance + PLTAXC_12_33.balance + PLTAXC_01_34.balance</field>
                        <field name="sequence">250</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_podatek_kraj_3_lub_5" model="account.report.line">
                                <field name="name">Podatek - Dostawa towarów/usług, kraj, 3% lub 5%</field>
                                <field name="code">PLTAXC_04_16</field>
                                <field name="sequence">260</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_kraj_3_lub_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_16</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_kraj_7_lub_8" model="account.report.line">
                                <field name="name">Podatek - Dostawa towarów/usług, kraj, 7% lub 8%</field>
                                <field name="code">PLTAXC_05_18</field>
                                <field name="sequence">270</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_kraj_7_lub_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_18</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_kraj_22_lub_23" model="account.report.line">
                                <field name="name">Podatek - Dostawa towarów/usług, kraj, 22% lub 23%</field>
                                <field name="code">PLTAXC_06_20</field>
                                <field name="sequence">280</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_kraj_22_lub_23_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_20</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_nabycie_towarow" model="account.report.line">
                                <field name="name">Podatek - Wewn-wspól. nabycie towarów</field>
                                <field name="code">PLTAXC_09_24</field>
                                <field name="sequence">290</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_nabycie_towarow_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_24</field>
                                    </record>
                                    <record id="account_tax_report_line_podatek_nabycie_towarow_formula" model="account.report.expression">
                                        <field name="label">formula</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">PLTAXC_09_24.balance + PLTAXC_10_35.balance</field>
                                    </record>
                                </field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_podatek_transp_termin" model="account.report.line">
                                        <field name="name">Podatek - Wew.wspól. nabycie środk. transp. termin</field>
                                        <field name="code">PLTAXC_10_35</field>
                                        <field name="sequence">300</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_podatek_transp_termin_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">K_35</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_art_33a" model="account.report.line">
                                <field name="name">Podatek - Import towarów art. 33a</field>
                                <field name="code">PLTAXC_10_26</field>
                                <field name="sequence">310</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_art_33a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_26</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_import_uslug" model="account.report.line">
                                <field name="name">Podatek - Import usług</field>
                                <field name="code">PLTAXC_11_28</field>
                                <field name="sequence">320</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_import_uslug_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_28</field>
                                    </record>
                                </field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_podatek_art_28b" model="account.report.line">
                                        <field name="name">Podatek - W tym nabycie wg art 28b</field>
                                        <field name="code">PLTAXC_11a_30</field>
                                        <field name="sequence">330</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_podatek_art_28b_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">K_30</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_podatnik_nabywca" model="account.report.line">
                                <field name="name">Podatek - Dostawa towarów, podatnik nabywca</field>
                                <field name="code">PLTAXC_12_32</field>
                                <field name="sequence">340</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_podatnik_nabywca_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_32</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_art_14_5" model="account.report.line">
                                <field name="code">PLTAXC_12_33</field>
                                <field name="name">Podatek - Ze spisu z natury art 14.5</field>
                                <field name="sequence">350</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_art_14_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_33</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_kasy_rejestrujace" model="account.report.line">
                                <field name="name">Podatek - Wydatek na kasy rejestrujące</field>
                                <field name="code">PLTAXC_01_34</field>
                                <field name="sequence">360</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_kasy_rejestrujace_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_34</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_wewnątrzwspólnotowe_103_5a" model="account.report.line">
                                <field name="name">Podatek - Wewn-wspól. nabycie towarów art. 103.5a</field>
                                <field name="code">PLTAXC_01_36</field>
                                <field name="sequence">370</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_wewnątrzwspólnotowe_103_5a_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_36</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_podatek_razem_d" model="account.report.line">
                        <field name="name">Podatek - Razem D</field>
                        <field name="code">PLTAXD</field>
                        <field name="aggregation_formula">PODATEK__NADWYZKA_Z_POPRZEDNIEJ_DEKLARACJI.balance + PODATEK__NABYCIE_TOWAROW_I_USLUG_STRWALE.balance + PLTAXD_02_42.balance + PODATEK__KOREKTA_NALICZONEGO_OD_NABYCIA_STRWALYCH.balance + PODATEK__KOREKTA_NALICZONEGO_OD_POZOSTALYCH_NABYC.balance + PLTAXD_02_46.balance + PLTAXD_02_47.balance</field>
                        <field name="sequence">380</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_podatek_deklaracji" model="account.report.line">
                                <field name="name">Podatek - Nadwyżka z poprzedniej deklaracji</field>
                                <field name="code">PODATEK__NADWYZKA_Z_POPRZEDNIEJ_DEKLARACJI</field>
                                <field name="sequence">390</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_deklaracji_applied_carryover" model="account.report.expression">
                                        <field name="label">_applied_carryover_balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">most_recent</field>
                                        <field name="date_scope">previous_tax_period</field>
                                    </record>
                                    <record id="account_tax_report_line_podatek_deklaracji_tag" model="account.report.expression">
                                        <field name="label">tag</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_39</field>
                                    </record>
                                    <record id="account_tax_report_line_podatek_deklaracji_balance" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">aggregation</field>
                                        <field name="formula">PODATEK__NADWYZKA_Z_POPRZEDNIEJ_DEKLARACJI.tag + PODATEK__NADWYZKA_Z_POPRZEDNIEJ_DEKLARACJI._applied_carryover_balance</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_s_trwale" model="account.report.line">
                                <field name="name">Podatek - Nabycie towarów i usług ś.trwałe</field>
                                <field name="code">PODATEK__NABYCIE_TOWAROW_I_USLUG_STRWALE</field>
                                <field name="sequence">400</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_s_trwale_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_41</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_uslug_pozostalych" model="account.report.line">
                                <field name="name">Podatek - Nabycie towarów i usług pozostałych</field>
                                <field name="code">PLTAXD_02_42</field>
                                <field name="sequence">410</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_uslug_pozostalych_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_43</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_s_trwalych" model="account.report.line">
                                <field name="name">Podatek - Korekta naliczonego od nabycia ś.trwałych</field>
                                <field name="code">PODATEK__KOREKTA_NALICZONEGO_OD_NABYCIA_STRWALYCH</field>
                                <field name="sequence">420</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_s_trwalych_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_44</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_podatek_pozostalych_nabyc" model="account.report.line">
                                <field name="name">Podatek - Korekta naliczonego od pozostałych nabyć</field>
                                <field name="code">PODATEK__KOREKTA_NALICZONEGO_OD_POZOSTALYCH_NABYC</field>
                                <field name="sequence">430</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_podatek_pozostalych_nabyc_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_45</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_korekta_art89b1" model="account.report.line">
                                <field name="name">Tax - Korekta naliczonego art. 89b.1</field>
                                <field name="code">PLTAXD_02_46</field>
                                <field name="sequence">440</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_korekta_art89b1_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_46</field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_korekta_art89b4" model="account.report.line">
                                <field name="name">Tax - Korekta naliczonego art. 89b.4</field>
                                <field name="code">PLTAXD_02_47</field>
                                <field name="sequence">450</field>
                                <field name="expression_ids">
                                    <record id="account_tax_report_line_korekta_art89b4_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">K_47</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_podatek_okresie" model="account.report.line">
                        <field name="name">Podatek - Wydatek na kasy do zwrotu w tym okresie</field>
                        <field name="code">PLTAX_49</field>
                        <field name="sequence">460</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_podatek_okresie_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_tax_report_line_zaniechaniem_poboru" model="account.report.line">
                        <field name="name">Podatek - Objęty zaniechaniem poboru</field>
                        <field name="code">PLTAX_50</field>
                        <field name="sequence">470</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_zaniechaniem_poboru_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_pl_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<data>
    <!-- Chart template -->

        <record id="pl_chart_template" model="account.chart.template">
            <field name="name">Polska - Plan kont</field>
            <field name="code_digits">8</field>
            <field name="currency_id" ref="base.PLN"/>
            <field name="bank_account_code_prefix">11.000.00</field>
            <field name="cash_account_code_prefix">12.000.00</field>
            <field name="transfer_account_code_prefix">11.090.00</field>
            <field name="country_id" ref="base.pl"/>
            <field name="use_storno_accounting" eval="True"/>
        </record>

</data>
</odoo>

```

## File: data\l10n_pl_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="pl_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="chart20000000"/>
        <field name="property_account_payable_id" ref="chart21000000"/>
        <field name="property_account_expense_categ_id" ref="chart70110000"/>
        <field name="property_account_income_categ_id" ref="chart73010000"/>
        <field name="property_tax_payable_account_id" ref="chart22010000"/>
        <field name="property_tax_receivable_account_id" ref="chart22010000"/>
        <field name="income_currency_exchange_account_id" ref="chart75060000"/>
        <field name="expense_currency_exchange_account_id" ref="chart75140000"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="chart75190000"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="chart75070000"/>
        <field name="default_pos_receivable_account_id" ref="chart20000100" />
        <field name="default_cash_difference_income_account_id" ref="chart77000000"/>
        <field name="default_cash_difference_expense_account_id" ref="chart77100000"/>
    </record>
</odoo>

```

## File: data\res.country.state.csv

```csv
"id","country_id:id","name","code"
state_pl_ds,base.pl,"dolnośląskie","DŚ"
state_pl_kp,base.pl,"kujawsko-pomorskie","KP"
state_pl_lb,base.pl,"lubelskie","LB"
state_pl_ls,base.pl,"lubuskie","LS"
state_pl_ld,base.pl,"łódzkie","ŁD"
state_pl_mp,base.pl,"małopolskie","MP"
state_pl_mz,base.pl,"mazowieckie","MZ"
state_pl_op,base.pl,"opolskie","OP"
state_pl_pk,base.pl,"podkarpackie","PK"
state_pl_pl,base.pl,"podlaskie","PL"
state_pl_pm,base.pl,"pomorskie","PM"
state_pl_sl,base.pl,"śląskie","ŚL"
state_pl_sk,base.pl,"świętokrzyskie","ŚK"
state_pl_wm,base.pl,"warmińsko-mazurskie","WM"
state_pl_wp,base.pl,"wielkopolskie","WP"
state_pl_zp,base.pl,"zachodniopomorskie","ZP"

```

## File: migrations\9.0.2.0\pre-set_tags_and_taxes_updatable.py

```python
from openerp.modules.registry import RegistryManager

def migrate(cr, version):
    registry = RegistryManager.get(cr.dbname)
    from openerp.addons.account.models.chart_template import migrate_set_tags_and_taxes_updatable
    migrate_set_tags_and_taxes_updatable(cr, registry, 'l10n_pl')

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="7.06" width="50.4" height="34.13" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.34" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="1280" height="800" transform="translate(4.8 7.06) scale(0.04 0.04)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABQEAAANjCAYAAAAaltwHAAAACXBIWXMAARlHAAEZRwHSp9OrAAAYw0lEQVR4XuzaMQ2AUBQEQT6hRg3+FaAGAw8J1Gxm6lOwuTUzswEAAAAAVWv/WgAAAAAA/yYCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEHc+6vjYAAAAAwE+dc3sCAgAAAECdCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAgAAAECcCAgAAAAAcSIgAAAAAMSJgAAAAAAQJwICAAAAQJwICAAAAABxIiAAAAAAxImAAAAAABAnAgIAAABAnAgIAAAAAHEiIAAAAADEiYAAAAAAECcCAvC2YwcyAAAAAIP8re/xFUYAAADMSUAAAAAAmJOAAAAAADAnAQEAAABgTgICAAAAwJwEBAAAAIA5CQgAAAAAcxIQAAAAAOYkIAAAAADMSUAAAAAAmJOAAAAAADAnAQEAAABgTgICAAAAwJwEBAAAAIA5CQgAAAAAcxIQAAAAAOYkIAAAAADMSUAAAAAAmJOAAAAAADAnAQEAAABgTgICAAAAwJwEBAAAAIA5CQgAAAAAcxIQAAAAAOYkIAAAAADMSUAAAAAAmJOAAAAAADAnAQEAAABgTgICAAAAwJwEBAAAAIA5CQgAAAAAcxIQAAAAAOYkIAAAAADMSUAAAAAAmJOAAAAAADAnAQEAAABgTgICAAAAwJwEBAAAAIA5CQgAAAAAcxIQAAAAAOYkIAAAAADMSUAAAAAAmJOAAAAAADAnAQEAAABgTgICAAAAwJwEBAAAAIA5CQgAAAAAcxIQAAAAAOYkIAAAAADMBbILDcXxuikxAAAAAElFTkSuQmCC"/>
    </g>
  </g>
</svg>

```

