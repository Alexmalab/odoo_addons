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
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2009 - now Grzegorz Grzelak grzegorz.grzelak@openglobe.pl

{
    'name' : 'Poland - Accounting',
    'version' : '2.0',
    'author' : 'Grzegorz Grzelak (OpenGLOBE)',
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
              'data/res.country.state.csv',
              'data/l10n_pl_chart_post_data.xml',
              'data/account_data.xml',
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
"id","name","code","chart_template_id/id","user_type_id/id","reconcile"
"chart01010000","Grunty własne i prawa wieczystego użytkowania gruntów","01-010-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart01020000","Budynki, lokale i obiekty inżynierii lądowej i wodnej","01-020-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart01030000","Urządzenia techniczne i maszyny","01-030-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart01040000","Środki transportu","01-040-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart01050000","Inne środki trwałe","01-050-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart02010000","Koszty zakończonych prac rozwojowych","02-010-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart02020000","Nabyta wartość firmy","02-020-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart02080000","Inne wartości niematerialne i prawne","02-080-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart02090000","Zaliczki na wartości niematerialne i prawne","02-090-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart03010100","Udziały lub akcje","03-010-100","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart03010200","Inne papiery wartościowe","03-010-200","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart03010300","Udzielone pożyczki","03-010-300","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart03010400","Inne długoterminowe aktywa finansowe","03-010-400","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart03020100","Udziały lub akcje","03-020-100","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart03020200","Inne papiery wartościowe","03-020-200","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart03020300","Udzielone pożyczki","03-020-300","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart03020400","Inne długoterminowe aktywa finansowe","03-020-400","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart03050100","Udziały lub akcje","03-050-100","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart03050200","Inne papiery wartościowe","03-050-200","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart03050300","Udzielone pożyczki","03-050-300","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart03050400","Inne długoterminowe aktywa finansowe","03-050-400","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart03090100","Inne rodzaje długoterminowych aktywów finansowych","03-090-100","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart04010000","Nieruchomości","04-010-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart04020000","Wartości niematerialne i prawne","04-020-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart05010000","Aktywa z tytułu odroczonego podatku dochodowego","05-010-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart05020000","Inne rozliczenia międzyokresowe","05-020-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart06010000","Od jednostek powiązanych","06-010-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart06020000","Od pozostałych jednostek","06-020-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart07010100","Odpisy umorzeniowe wartości gruntów i prawa wieczystego użytkowania gruntów","07-010-100","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart07010200","Odpisy umorzeniowe budynków, lokali i obiektów inżynierii lądowej i wodnej","07-010-200","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart07010300","Odpisy umorzeniowe urządzeń technicznych i maszyn","07-010-300","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart07010400","Odpisy umorzeniowe środków transportu","07-010-400","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart07010500","Odpisy umorzeniowe innych środków trwałych","07-010-500","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart07010900","Odpisy umorzeniowe ulepszeń obcych środków trwałych","07-010-900","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart07020100","Odpisy umorzeniowe zakończonych prac rozwojowych","07-020-100","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart07020200","Odpisy umorzeniowe wartości firmy","07-020-200","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart07020300","Odpisy umorzeniowe innych wartości niematerialnych i prawnych","07-020-300","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart07030100","Odpisy umorzeniowe inwestycji w nieruchomości","07-030-100","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart07030200","Odpisy umorzeniowe inwestycji w wartości niematerialne i prawne","07-030-200","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart08010000","Inwestycje budowy środka trwałego","08-010-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart08020000","Ulepszenia środka trwałego","08-020-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart08030000","Nakłady na budowę środka trwałego","08-030-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart08040000","Zaliczki na środki trwałe w budowie","08-040-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart08050000","Ulepszenia obcych środków trwałych","08-050-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart09010000","Odpisy aktualizujące udziały i akcje w obcych jednostkach","09-010-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart09020000","Odpisy aktualizujące lokaty","09-020-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart09030000","Odpisy aktualizujące udzielone pożyczki długoterminowe","09-030-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart09090000","Odpisy aktualizujące inne rodzaje długoterminowych aktywów finansowych","09-090-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart10010000","Kasa krajowych środków pieniężnych","10-010-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart10020000","Kasa zagranicznych środków pieniężnych","10-020-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart13090000","Pozostałe inne środki pieniężne","13-090-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart16000000","Inne aktywa pieniężne","14-000-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart15010100","Udziały lub akcje","15-010-100","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart15010200","Inne papiery wartościowe","15-010-200","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart15010300","Udzielone pożyczki","15-010-300","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart15010400","Inne krótkoterminowe aktywa finansowe","15-010-400","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart15020100","Udziały lub akcje","15-020-100","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart15020200","Inne papiery wartościowe","15-020-200","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart15020400","Inne krótkoterminowe aktywa finansowe","15-020-400","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart15050000","Odpisy aktualizujące krótkoterminowe aktywa finansowe","15-050-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart18000000","Krótkoterminowe rozliczenia międzyokresowe","18-000-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart20000000","Rozrachunki z odbiorcami","20-000-000","l10n_pl.pl_chart_template","account.data_account_type_receivable","True"
"chart20000100","Rozrachunki z odbiorcami (PoS)","20-000-100","l10n_pl.pl_chart_template","account.data_account_type_receivable","True"
"chart21000000","Rozrachunki z dostawcami","21-000-000","l10n_pl.pl_chart_template","account.data_account_type_payable","True"
"chart22010000","Rozrachunki z urzędem skarbowym z tytułu VAT","22-010-000","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart22020100","Rozliczenie naliczonego VAT 22%","22-020-100","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart22020200","Rozliczenie naliczonego VAT 7%","22-020-200","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart22020300","Rozliczenie naliczonego VAT 3%","22-020-300","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart22020400","Rozliczenie naliczonego VAT 23%","22-020-400","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart22020500","Rozliczenie naliczonego VAT 8%","22-020-500","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart22020600","Rozliczenie naliczonego VAT 5%","22-020-600","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart22030100","Rozliczenie należnego VAT 22%","22-030-100","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart22030200","Rozliczenie należnego VAT 7%","22-030-200","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart22030300","Rozliczenie należnego VAT 3%","22-030-300","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart22030400","Rozliczenie należnego VAT 23%","22-030-400","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart22030500","Rozliczenie należnego VAT 8%","22-030-500","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart22030600","Rozliczenie należnego VAT 5%","22-030-600","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart22040000","Pozostałe rozrachunki publicznoprawne","22-040-000","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart22040100","Pozostałe rozrachunki z urzędem skarbowym","22-040-100","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart22040200","Rozrachunki publicznoprawne z urzędem miasta/gminy","22-040-200","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart22040300","Rozrachunki publicznoprawne z urzędem celnym","22-040-300","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart22040400","Rozrachunki publicznoprawne z ZUS","22-040-400","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart22040500","Rozrachunki publicznoprawne z PFRON","22-040-500","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart23010000","Rozrachunki z tytułu wynagrodzeń","23-010-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart23020000","Rozrachunki z tytułu pożyczek udzielonych pracownikom","23-020-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart23090000","Inne rozrachunki z pracownikami","23-090-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart24010100","Pożyczki otrzymane","24-010-100","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart24010200","Pożyczki udzielone","24-010-200","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart24020100","Rozliczenie niedoborów","24-020-100","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart24020200","Rozliczenie nadwyżek","24-020-200","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart24030100","Rozrachunki z tytułu wpłat na kapitał zakładowy","24-030-100","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart24030200","Rozrachunki z tytułu wkładów niepieniężnych na kapitał zakładowy","24-030-200","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart24030300","Rozrachunki z tytułu podwyższenia kapitału ze środków własnych spółki","24-030-300","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart24030400","Rozrachunki z tytułu umorzenia udziałów własnych","24-030-400","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart24050000","Rozrachunki wewnątrzzakładowe","24-050-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart24060000","Należności dochodzone na drodze sądowej","24-060-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart24070000","Rozrachunki z tytułu dywidend","24-070-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart24080000","Rozrachunki z tytułu dopłat i zwrotu dopłat","24-080-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart24090000","Pozostałe rozrachunki","24-090-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart28000000","Odpisy aktualizujące rozrachunki","28-000-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart29010000","Należności warunkowe","29-010-000","l10n_pl.pl_chart_template","account.data_account_off_sheet","False"
"chart29020000","Zobowiązania warunkowe","29-020-000","l10n_pl.pl_chart_template","account.data_account_off_sheet","False"
"chart29030000","Weksle obce dyskontowane lub indosowane","29-030-000","l10n_pl.pl_chart_template","account.data_account_off_sheet","False"
"chart30010000","Rozliczenie zakupu materiałów","30-010-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart30020000","Rozliczenie zakupu towarów","30-020-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart30030000","Rozliczenie zakupu usług obcych","30-030-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart30040000","Rozliczenie zakupu składników aktywów trwałych","30-040-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart30050000","Rozliczenie wartości materiałów i towarów w drodze","30-050-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart30060000","Wartości dostaw niefakturowanych","30-060-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart30070000","Opłaty manipulacyjne policzone przez Urząd Celny","30-070-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart30080000","Niedobory, szkody i nadwyżki w transporcie","30-080-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart30090000","Reklamacje faktur dostawców","30-090-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart31010000","Materiały","31-010-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart31060000","Opakowania","31-060-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart31090000","Materiały w przerobie","31-090-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart33010000","Towary w hurcie","33-010-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart33020000","Towary w detalu","33-020-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart33030000","Towary w zakładach gastronomicznych","33-030-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart33040000","Towary skupu","33-040-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart33080000","Towary poza jednostką","33-080-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart33090000","Nieruchomości i prawa majątkowe przeznaczone do obrotu","33-090-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart34010000","Odchylenia od cen ewidencyjnych materiałów","34-010-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart34020100","Odchylenia od cen ewidencyjnych towarów w hurcie","34-020-100","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart34020200","Odchylenia od cen ewidencyjnych towarów w detalu","34-020-200","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart34020300","Odchylenia od cen ewidencyjnych towarów w zakładach gastronomicznych","34-020-300","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart34020400","Odchylenia od cen ewidencyjnych towarów skupu","34-020-400","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart34060000","Odchylenia od cen ewidencyjnych opakowań","34-060-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart34070000","Odchylenia z tytułu aktualizacji wartości zapasów materiałów i towarów","34-070-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart39000000","Zapasy obce","39-000-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart40010100","Zużycie surowców do wytwarzania produktów","40-010-100","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40010200","Zużycie energii","40-010-200","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40010300","Zużycie paliwa do środków transportu","40-010-300","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40010400","Zużycie materiałów biurowych","40-010-400","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40010900","Zużycie innych materiałów","40-010-900","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40020100","Usługi celne","40-020-100","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40020200","Usługi telekomunikacyjne","40-020-200","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40020300","Usługi pocztowe","40-020-300","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40020400","Usługi kurierskie i transportowe","40-020-400","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40020500","Analizy sanitarne","40-020-500","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40020600","Usługi graficzne i drukarskie","40-020-600","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40020700","Usługi remontowe","40-020-700","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40020900","Pozostałe usługi","40-020-900","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40030100","Podatek od nieruchomości","40-030-100","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40030200","Podatek od środków transportowych","40-030-200","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40030300","Podatek akcyzowy","40-030-300","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40030400","VAT niepodlegający odliczeniu","40-030-400","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40030500","Opłaty i prowizje bankowe","40-030-500","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40030600","Opłaty sądowe, prawnicze i notarialne","40-030-600","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40030700","Opłaty skarbowe","40-030-700","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40030800","Koncesje","40-030-800","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40030900","Pozostałe podatki i opłaty","40-030-900","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40040100","Wynagrodzenia pracowników","40-040-100","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40040200","Wynagrodzenia osób doraźnie zatrudnionych","40-040-200","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40050100","Składki na ubezpieczenia społeczne, FP, FGŚP","40-050-100","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40050200","Odpisy na zakładowy fundusz świadczeń socjalnych lub świadczenia urlopowe","40-050-200","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart40050900","Pozostałe świadczenia","40-050-900","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart41000000","Świadczenia na rzecz pracowników","41-000-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart42000000","Amortyzacja","42-000-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart43000000","Pozostałe koszty rodzajowe","43-000-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart49010000","Nie podlegające rozliczeniu w czasie","49-010-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart49020000","Przypadające na przyszłe okresy","49-020-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart49030000","Koszty zgromadzone","49-030-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart49040000","Koszty nie wliczane do wartości sprzedaży","49-040-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart50010000","Rozliczone koszty działalności","50-010-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart50020000","Koszty nie zakończonych długotrwałych usług","50-020-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart50030000","Straty związane z wykonaniem długotrwałych usług","50-030-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart50040000","Koszty utrzymania hurtowni","50-040-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart52010000","Koszty utrzymania punktów sprzedaży detalicznej","52-010-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart52030000","Koszty sprzedaży wyrobów","52-030-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart53010000","Świadczenia usług transportowych","53-010-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart53020000","Pozostałe koszty","53-020-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart55010000","Koszty zarządzania jednostką","55-010-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart55020000","Świadczenia usług na potrzeby reprezentacji i reklamy","55-020-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart58000000","Rozliczenie kosztów działalności","58-000-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart60010100","Produkty gotowe w magazynie","60-010-100","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart60010200","Produkty gotowe poza jednostką","60-010-200","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart60020000","Półprodukty","60-020-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart62010000","Odchylenia od cen ewidencyjnych produktów","62-010-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart62020000","Odchylenia z tytułu aktualizacji wartości zapasów produktów","62-020-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart64010000","Czynne rozliczenia międzyokresowe kosztów","64-010-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart64020000","Bierne rozliczenia międzyokresowe kosztów","64-020-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart65010000","Aktywa z tytułu odroczonego podatku dochodowego","65-010-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart65020000","Inne rozliczenia międzyokresowe","65-020-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart70010000","Sprzedaż produktów na kraj","70-010-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart70020000","Sprzedaż produktów na eksport","70-020-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart70030000","Sprzedaż usług na kraj","70-030-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart70040000","Sprzedaż usług na eksport","70-040-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart70110000","Koszt własny sprzedaży produktów na kraj","70-110-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart70120000","Koszt własny sprzedaży produktów na eksport","70-120-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart70130000","Koszt własny sprzedaży usług na kraj","70-130-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart70140000","Koszt własny sprzedaży usług na eksport","70-140-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart73010000","Sprzedaż hurtowa towarów","73-010-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart73020000","Sprzedaż detaliczna towarów","73-020-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart73030000","Sprzedaż wysyłkowa towarów","73-030-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart73040000","Prowizja komisowa","73-040-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"pl_a_expense","Wartość sprzedanych towarów w sprzedaży hurtowej","73-110-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart73120000","Wartość sprzedanych towarów w sprzedaży detalicznej","73-120-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart73130000","Wartość sprzedanych towarów w sprzedaży wysyłkowej","73-130-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart73140000","Prowizja komisowa","73-140-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart74010000","Sprzedaż materiałów","74-010-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart74020000","Sprzedaż opakowań","74-020-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart74030000","Sprzedaż odpadów","74-030-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart74110000","Wartość w cenach zakupu sprzedanych materiałów","74-110-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart74120000","Wartość w cenach zakupu sprzedanych opakowań","74-120-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart74130000","Wartość w cenach zakupu sprzedanych odpadów","74-130-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart75010000","Kwoty należne ze sprzedaży aktywów finansowych","75-010-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart75020000","Kwoty należne z tytułu dywidend","75-020-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart75030000","Otrzymane odsetki","75-030-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart75040000","Przychody ze zbycia inwestycji","75-040-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart75050000","Aktualizacja wartości inwestycji-przychody","75-050-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart75060000","Dodatnie różnice kursu walut","75-060-000","l10n_pl.pl_chart_template","account.data_account_type_other_income","False"
"chart75070000","Pozostałe przychody finansowe","75-070-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart75110000","Wartość sprzedanych inwestycji","75-110-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart75120000","Odpisy z tytułu utraty wartości inwestycji-koszty","75-120-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart75130000","Odsetki zapłacone","75-130-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"pl_a_pay","Ujemne różnice kursu walut","75-140-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart75190000","Pozostałe koszty finansowe","75-190-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart76010000","Przychody ze zbycia niefinansowych aktywów trwałych","76-010-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart76020000","Otrzymane dotacje","76-020-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart76030000","Przychody z usług socjalnych","76-030-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart76040000","Przychody ze wzrostu wartości niefinansowych aktywów trwałych","76-040-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart76090000","Inne pozostałe przychody operacyjne","76-090-000","l10n_pl.pl_chart_template","account.data_account_type_revenue","False"
"chart76110000","Wartość sprzedanych niefinansowych aktywów trwałych","76-110-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart76120000","Dotacje przekazane","76-120-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart76130000","Odpisy z tytułu utraty wartości aktywów niefinansowych","76-130-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart76140000","Inne pozostałe koszty operacyjne","76-140-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart77000000","Zyski nadzwyczajne","77-000-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart77100000","Straty nadzwyczajne","77-100-000","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart79010000","Koszt wyrobów własnej produkcji wydanych do własnych sklepów","79-010-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart79020000","Świadczenia na rzecz środków trwałych w budowie","79-020-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart79040000","Koszt niedoborów produktów","79-040-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart79050000","Koszt zaniechania określonego rodzaju działalności","79-050-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart79110000","Koszt wytworzenia wyrobów gotowych wydanych do własnych sklepów","79-110-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart79120000","Koszt wytworzenia świadczeń na rzecz środków trwałych w budowie","79-120-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart79130000","Koszt wytworzenia zakończonych prac rozwojowych","79-130-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart79140000","Koszt wytworzenia produktów uznanych za niedobory","79-140-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart79150000","Koszt zaniechania określonego rodzaju działalności","79-150-000","l10n_pl.pl_chart_template","account.data_account_type_expenses","False"
"chart80000000","Kapitał podstawowy","80-000-000","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart81100000","Kapitał zapasowy","81-100-000","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart81200000","Kapitał rezerwowy","81-200-000","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart81300000","Kapitał z aktualizacji wyceny","81-300-000","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart81400000","Kapitały wydzielone w jednostce statutowej i zakładach (oddziałach) samodzielnie sporządzających bilans","81-400-000","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart82000000","Rozliczenia wyniku finansowego","82-000-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart83010000","Rezerwa z tytułu odroczonego podatku dochodowego","83-010-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart83020100","Rezerwa długoterminowa na świadczenia emerytalne i podobne","83-020-100","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart83020200","Rezerwa krótkoterminowa na świadczenia emerytalne i podobne","83-020-200","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart83090100","Pozostałe rezerwy długoterminowe","83-090-100","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart83090200","Pozostałe rezerwy krótkoterminowe","83-090-200","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart84010000","Ujemna wartość firmy","84-010-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart84020100","Inne rozliczenia międzyokresowe długoterminowe","84-020-100","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart84020200","Inne rozliczenia międzyokresowe krótkoterminowe","84-020-200","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart85010000","Zakładowy fundusz świadczeń socjalnych","85-010-000","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart85020100","Zakładowy fundusz rehabilitacji osób niepełnosprawnych","85-020-100","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart85020200","Fundusz nagród","85-020-200","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart85020300","Fundusz na remont zasobów mieszkaniowych","85-020-300","l10n_pl.pl_chart_template","l10n_pl.account_type_settlement","False"
"chart86000000","Wynik finansowy","86-000-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"
"chart87010000","Podatek dochodowy od osób prawnych","87-010-000","l10n_pl.pl_chart_template","account.data_account_type_current_liabilities","False"
"chart87020000","Inne obowiązkowe obciążenia wyniku finansowego","87-020-000","l10n_pl.pl_chart_template","account.data_account_type_current_assets","False"

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

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Account Tax Group -->
        <record id="tax_group_vat_23" model="account.tax.group">
            <field name="name">VAT 23%</field>
        </record>
        <record id="tax_group_vat_22" model="account.tax.group">
            <field name="name">VAT 22%</field>
        </record>
        <record id="tax_group_vat_8" model="account.tax.group">
            <field name="name">VAT 8%</field>
        </record>
        <record id="tax_group_vat_7" model="account.tax.group">
            <field name="name">VAT 7%</field>
        </record>
        <record id="tax_group_vat_5" model="account.tax.group">
            <field name="name">VAT 5%</field>
        </record>
        <record id="tax_group_vat_3" model="account.tax.group">
            <field name="name">VAT 3%</field>
        </record>
        <record id="tax_group_vat_0" model="account.tax.group">
            <field name="name">VAT 0%</field>
        </record>
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_kraj_22_lub_23')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_kraj_22_lub_23')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_kraj_22_lub_23')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030100'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_kraj_22_lub_23')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030100'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_kraj_7_lub_8')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030500'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_7_lub_8')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_kraj_7_lub_8')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030500'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_7_lub_8')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_kraj_7_lub_8')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030200'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_7_lub_8')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_kraj_7_lub_8')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030200'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_7_lub_8')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_kraj_3_lub_5')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030600'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_3_lub_5')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_kraj_3_lub_5')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030600'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_3_lub_5')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_kraj_3_lub_5')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030300'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_3_lub_5')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_kraj_3_lub_5')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030300'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_3_lub_5')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslugi_kraj_0')],
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
              'minus_report_line_ids': [ref('account_tax_report_line_uslugi_kraj_0')],
          }),
          (0,0, {
              'factor_percent': 100,
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_kraj_zwolnione')],
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
              'minus_report_line_ids': [ref('account_tax_report_line_kraj_zwolnione')],
          }),
          (0,0, {
              'factor_percent': 100,
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
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_kraj_22_lub_23')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_kraj_22_lub_23')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23')],
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
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_kraj_22_lub_23')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030100'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_kraj_22_lub_23')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030100'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_kraj_22_lub_23')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020100'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020100'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020500'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020500'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020200'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020200'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020600'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020600'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020300'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020300'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
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
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
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
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
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
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
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
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020100'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020100'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 60,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 40,
              'repartition_type': 'tax',
              'account_id': ref('chart76140000'),
          }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 60,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
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
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 60,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 40,
              'repartition_type': 'tax',
              'account_id': ref('chart76140000'),
          }),
      ]"/>
      <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': 60,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
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
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_podatnik_nabywca')],
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
              'minus_report_line_ids': [ref('account_tax_report_line_podatnik_nabywca')],
          }),
          (0,0, {
              'factor_percent': 100,
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
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych'), ref('account_tax_report_line_podatnik_nabywca')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_podatnik_nabywca')],
          }),
      ]"/>
      <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych'), ref('account_tax_report_line_podatnik_nabywca')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_podatnik_nabywca')],
          }),
      ]"/>
      <field name="tax_group_id" ref="tax_group_vat_0"/>
    </record>

    <!-- Eurpean Union -->
    <!-- =========================================================== -->

    <record id="vs_unia" model="account.tax.template">
      <field name="chart_template_id" ref="pl_chart_template"/>
      <field name="name">Dost tow. unia</field>
      <field name="description">UDT</field>
      <field name="amount">0</field>
      <field name="amount_type">percent</field>
      <field name="type_tax_use">sale</field>
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_dostawa_towarow')],
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
              'minus_report_line_ids': [ref('account_tax_report_line_dostawa_towarow')],
          }),
          (0,0, {
              'factor_percent': 100,
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
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych'), ref('account_tax_report_line_nabycie_towarow')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_nabycie_towarow')],
          }),
      ]"/>
      <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych'), ref('account_tax_report_line_nabycie_towarow')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_nabycie_towarow')],
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
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslugi_art_100_1_4')],
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
              'minus_report_line_ids': [ref('account_tax_report_line_uslugi_art_100_1_4')],
          }),
          (0,0, {
              'factor_percent': 100,
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
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych'), ref('account_tax_report_line_art_28b')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_art_28b')],
          }),
      ]"/>
      <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych'), ref('account_tax_report_line_art_28b')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_art_28b')],
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
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_eksport_towarow')],
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
              'minus_report_line_ids': [ref('account_tax_report_line_eksport_towarow')],
          }),
          (0,0, {
              'factor_percent': 100,
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
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych'), ref('account_tax_report_line_art_33a')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_art_33a')],
          }),
      ]"/>
      <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych'), ref('account_tax_report_line_art_33a')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_art_33a')],
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
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_poza_kraj')],
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
              'minus_report_line_ids': [ref('account_tax_report_line_poza_kraj')],
          }),
          (0,0, {
              'factor_percent': 100,
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
      <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'plus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych'), ref('account_tax_report_line_import_uslug')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_import_uslug')],
          }),
      ]"/>
      <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
              'minus_report_line_ids': [ref('account_tax_report_line_uslug_pozostalych'), ref('account_tax_report_line_import_uslug')],
          }),
          (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart22020400'),
              'minus_report_line_ids': [ref('account_tax_report_line_podatek_uslug_pozostalych')],
          }),
          (0,0, {
              'factor_percent': -100,
              'repartition_type': 'tax',
              'account_id': ref('chart22030400'),
              'plus_report_line_ids': [ref('account_tax_report_line_podatek_import_uslug')],
          }),
      ]"/>
      <field name="tax_group_id" ref="tax_group_vat_0"/>
    </record>


  </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.pl"/>
    </record>

    <record id="account_tax_report_line_razem_c" model="account.tax.report.line">
        <field name="name">Podstawa - Razem C</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="formula">PLTAXC_01_10 + PLTAXC_02_11 + PLTAXC_03_13 + PLTAXC_04_15 + PLTAXC_05_17 + PLTAXC_06_19 + PLTAXC_07_21 + PLTAXC_08_22 + PLTAXC_09_23 + PLTAXC_10_25 + PLTAXC_11_27 + PLTAXC_12_31</field>
    </record>

    <record id="account_tax_report_line_kraj_zwolnione" model="account.tax.report.line">
        <field name="name">Podstawa - Dostawa towarów/usług, kraj, zwolnione</field>
        <field name="tag_name">Podstawa - Dostawa towarów/usług, kraj, zwolnione</field>
        <field name="code">PLTAXC_01_10</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_razem_c"/>
    </record>

    <record id="account_tax_report_line_poza_kraj" model="account.tax.report.line">
        <field name="name">Podstawa - Dostawa towarów/usług, poza kraj</field>
        <field name="tag_name">Podstawa - Dostawa towarów/usług, poza kraj</field>
        <field name="code">PLTAXC_02_11</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_razem_c"/>
    </record>

    <record id="account_tax_report_line_uslugi_art_100_1_4" model="account.tax.report.line">
        <field name="name">Podstawa - W tym usługi art 100.1.4</field>
        <field name="tag_name">Podstawa - W tym usługi art 100.1.4</field>
        <field name="code">PLTAXC_02a_12</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_poza_kraj"/>
    </record>

    <record id="account_tax_report_line_uslugi_kraj_0" model="account.tax.report.line">
        <field name="name">Podstawa - Dostawa towarów/usług, kraj, 0%</field>
        <field name="tag_name">Podstawa - Dostawa towarów/usług, kraj, 0%</field>
        <field name="code">PLTAXC_03_13</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_razem_c"/>
    </record>

    <record id="account_tax_report_line_towary_art_129" model="account.tax.report.line">
        <field name="name">Podstawa - W tym towary art 129</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_uslugi_kraj_0"/>
    </record>

    <record id="account_tax_report_line_kraj_3_lub_5" model="account.tax.report.line">
        <field name="name">Podstawa - Dostawa towarów/usług, kraj, 3% lub 5%</field>
        <field name="tag_name">Podstawa - Dostawa towarów/usług, kraj, 3% lub 5%</field>
        <field name="code">PLTAXC_04_15</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_razem_c"/>
    </record>

    <record id="account_tax_report_line_kraj_7_lub_8" model="account.tax.report.line">
        <field name="name">Podstawa - Dostawa towarów/usług, kraj, 7% lub 8%</field>
        <field name="tag_name">Podstawa - Dostawa towarów/usług, kraj, 7% lub 8%</field>
        <field name="code">PLTAXC_05_17</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="account_tax_report_line_razem_c"/>
    </record>

    <record id="account_tax_report_line_kraj_22_lub_23" model="account.tax.report.line">
        <field name="name">Podstawa - Dostawa towarów/usług, kraj, 22% lub 23%</field>
        <field name="tag_name">Podstawa - Dostawa towarów/usług, kraj, 22% lub 23%</field>
        <field name="code">PLTAXC_06_19</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="account_tax_report_line_razem_c"/>
    </record>

    <record id="account_tax_report_line_dostawa_towarow" model="account.tax.report.line">
        <field name="name">Podstawa - Wew-wspól dostawa towarów</field>
        <field name="tag_name">Podstawa - Wew-wspól dostawa towarów</field>
        <field name="code">PLTAXC_07_21</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="account_tax_report_line_razem_c"/>
    </record>

    <record id="account_tax_report_line_eksport_towarow" model="account.tax.report.line">
        <field name="name">Podstawa - Eksport towarów</field>
        <field name="tag_name">Podstawa - Eksport towarów</field>
        <field name="code">PLTAXC_08_22</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="account_tax_report_line_razem_c"/>
    </record>

    <record id="account_tax_report_line_nabycie_towarow" model="account.tax.report.line">
        <field name="name">Podstawa - Wewn-wspól. nabycie towarów</field>
        <field name="tag_name">Podstawa - Wewn-wspól. nabycie towarów</field>
        <field name="code">PLTAXC_09_23</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="9"/>
        <field name="parent_id" ref="account_tax_report_line_razem_c"/>
    </record>

    <record id="account_tax_report_line_art_33a" model="account.tax.report.line">
        <field name="name">Podstawa - Import towarów art. 33a</field>
        <field name="tag_name">Podstawa - Import towarów art. 33a</field>
        <field name="code">PLTAXC_10_25</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="10"/>
        <field name="parent_id" ref="account_tax_report_line_razem_c"/>
    </record>

    <record id="account_tax_report_line_import_uslug" model="account.tax.report.line">
        <field name="name">Podstawa - Import usług</field>
        <field name="tag_name">Podstawa - Import usług</field>
        <field name="code">PLTAXC_11_27</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="11"/>
        <field name="parent_id" ref="account_tax_report_line_razem_c"/>
    </record>

    <record id="account_tax_report_line_art_28b" model="account.tax.report.line">
        <field name="name">Podstawa - W tym nabycie wg art 28b</field>
        <field name="tag_name">Podstawa - W tym nabycie wg art 28b</field>
        <field name="code">PLTAXC_11a_29</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_import_uslug"/>
    </record>

    <record id="account_tax_report_line_podatnik_nabywca" model="account.tax.report.line">
        <field name="name">Podstawa - Dostawa towarów, podatnik nabywca</field>
        <field name="tag_name">Podstawa - Dostawa towarów, podatnik nabywca</field>
        <field name="code">PLTAXC_12_31</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="12"/>
        <field name="parent_id" ref="account_tax_report_line_razem_c"/>
    </record>

    <record id="account_tax_report_line_razem_d" model="account.tax.report.line">
        <field name="name">Podstawa - Razem D</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="account_tax_report_line_uslug_s_trwale" model="account.tax.report.line">
        <field name="name">Podstawa - Nabycie towarów i usług ś.trwałe</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_razem_d"/>
    </record>

    <record id="account_tax_report_line_uslug_pozostalych" model="account.tax.report.line">
        <field name="name">Podstawa - Nabycie towarów i usług pozostałych</field>
        <field name="tag_name">Podstawa - Nabycie towarów i usług pozostałych</field>
        <field name="code">PLTAXD_02_41a</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_razem_d"/>
    </record>

    <record id="account_tax_report_line_do_przeniesienia" model="account.tax.report.line">
        <field name="name">Podatek - Do przeniesienia</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="formula">PLTAXC - PLTAXD</field>
    </record>

    <record id="account_tax_report_line_nad_naleznym" model="account.tax.report.line">
        <field name="name">Podatek - Nadwyżka naliczonego nad należnym</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="formula">PLTAXC - PLTAXD</field>
        <field name="parent_id" ref="account_tax_report_line_do_przeniesienia"/>
    </record>

    <record id="account_tax_report_line_do_US" model="account.tax.report.line">
        <field name="name">Podatek - Do wpłaty do US</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="formula">PLTAXC - PLTAXD</field>
        <field name="parent_id" ref="account_tax_report_line_nad_naleznym"/>
    </record>

    <record id="account_tax_report_line_kasy_rejestrujace" model="account.tax.report.line">
        <field name="name">Podatek - Wydatek na kasy rejestrujące</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_do_US"/>
    </record>

    <record id="account_tax_report_line_zaniechaniem_poboru" model="account.tax.report.line">
        <field name="name">Podatek - Objęty zaniechaniem poboru</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_do_US"/>
    </record>

    <record id="account_tax_report_line_podatek_razem_c" model="account.tax.report.line">
        <field name="name">Podatek - Razem C</field>
        <field name="code">PLTAXC</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="formula">PLTAXC_04_16 + PLTAXC_05_18 + PLTAXC_06_20 + PLTAXC_09_24 + PLTAXC_10_26 + PLTAXC_11_28 + PLTAXC_12_32</field>
        <field name="parent_id" ref="account_tax_report_line_do_US"/>
    </record>

    <record id="account_tax_report_line_podatek_kraj_3_lub_5" model="account.tax.report.line">
        <field name="name">Podatek - Dostawa towarów/usług, kraj, 3% lub 5%</field>
        <field name="tag_name">Podatek - Dostawa towarów/usług, kraj, 3% lub 5%</field>
        <field name="code">PLTAXC_04_16</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_c"/>
    </record>

    <record id="account_tax_report_line_podatek_kraj_7_lub_8" model="account.tax.report.line">
        <field name="name">Podatek - Dostawa towarów/usług, kraj, 7% lub 8%</field>
        <field name="tag_name">Podatek - Dostawa towarów/usług, kraj, 7% lub 8%</field>
        <field name="code">PLTAXC_05_18</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_c"/>
    </record>

    <record id="account_tax_report_line_podatek_kraj_22_lub_23" model="account.tax.report.line">
        <field name="name">Podatek - Dostawa towarów/usług, kraj, 22% lub 23%</field>
        <field name="tag_name">Podatek - Dostawa towarów/usług, kraj, 22% lub 23%</field>
        <field name="code">PLTAXC_06_20</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_c"/>
    </record>

    <record id="account_tax_report_line_podatek_nabycie_towarow" model="account.tax.report.line">
        <field name="name">Podatek - Wewn-wspól. nabycie towarów</field>
        <field name="tag_name">Podatek - Wewn-wspól. nabycie towarów</field>
        <field name="code">PLTAXC_09_24</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_c"/>
    </record>

    <record id="account_tax_report_line_podatek_art_33a" model="account.tax.report.line">
        <field name="name">Podatek - Import towarów art. 33a</field>
        <field name="tag_name">Podatek - Import towarów art. 33a</field>
        <field name="code">PLTAXC_10_26</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_c"/>
    </record>

    <record id="account_tax_report_line_podatek_import_uslug" model="account.tax.report.line">
        <field name="name">Podatek - Import usług</field>
        <field name="tag_name">Podatek - Import usług</field>
        <field name="code">PLTAXC_11_28</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_c"/>
    </record>

    <record id="account_tax_report_line_podatek_art_28b" model="account.tax.report.line">
        <field name="name">Podatek - W tym nabycie wg art 28b</field>
        <field name="tag_name">Podatek - W tym nabycie wg art 28b</field>
        <field name="code">PLTAXC_11a_30</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_import_uslug"/>
    </record>

    <record id="account_tax_report_line_podatek_podatnik_nabywca" model="account.tax.report.line">
        <field name="name">Podatek - Dostawa towarów, podatnik nabywca</field>
        <field name="tag_name">Podatek - Dostawa towarów, podatnik nabywca</field>
        <field name="code">PLTAXC_12_32</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_c"/>
    </record>

    <record id="account_tax_report_line_podatek_art_14_5" model="account.tax.report.line">
        <field name="name">Podatek - Ze spisu z natury art 14.5</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_c"/>
    </record>

    <record id="account_tax_report_line_podatek_transp_termin" model="account.tax.report.line">
        <field name="name">Podatek - Wew.wspól. nabycie środk. transp. termin</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="9"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_c"/>
    </record>

    <record id="account_tax_report_line_podatek_razem_d" model="account.tax.report.line">
        <field name="name">Podatek - Razem D</field>
        <field name="code">PLTAXD</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_do_US"/>
    </record>

    <record id="account_tax_report_line_podatek_deklaracji" model="account.tax.report.line">
        <field name="name">Podatek - Nadwyżka z poprzedniej deklaracji</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_d"/>
    </record>

    <record id="account_tax_report_line_pl_03_01_01_04_02" model="account.tax.report.line">
        <field name="name">Podatek - Naliczony ze spisu z natury, art.113</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_d"/>
    </record>

    <record id="account_tax_report_line_podatek_s_trwale" model="account.tax.report.line">
        <field name="name">Podatek - Nabycie towarów i usług ś.trwałe</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_d"/>
    </record>

    <record id="account_tax_report_line_podatek_uslug_pozostalych" model="account.tax.report.line">
        <field name="name">Podatek - Nabycie towarów i usług pozostałych</field>
        <field name="tag_name">Podatek - Nabycie towarów i usług pozostałych</field>
        <field name="code">PLTAXD_02_42</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_d"/>
    </record>

    <record id="account_tax_report_line_podatek_s_trwalych" model="account.tax.report.line">
        <field name="name">Podatek - Korekta naliczonego od nabycia ś.trwałych</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_d"/>
    </record>

    <record id="account_tax_report_line_podatek_pozostalych_nabyc" model="account.tax.report.line">
        <field name="name">Podatek - Korekta naliczonego od pozostałych nabyć</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_razem_d"/>
    </record>

    <record id="account_tax_report_line_podatek_okresie" model="account.tax.report.line">
        <field name="name">Podatek - Wydatek na kasy do zwrotu w tym okresie</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_nad_naleznym"/>
    </record>

    <record id="account_tax_report_line_podatek_do_zwrotu" model="account.tax.report.line">
        <field name="name">Podatek - Do zwrotu</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_do_przeniesienia"/>
    </record>

    <record id="account_tax_report_line_do_zwrotu_25_dni" model="account.tax.report.line">
        <field name="name">Podatek - Do zwrotu 25 dni</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_do_zwrotu"/>
    </record>

    <record id="account_tax_report_line_do_zwrotu_60_dni" model="account.tax.report.line">
        <field name="name">Podatek - Do zwrotu 60 dni</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_do_zwrotu"/>
    </record>

    <record id="account_tax_report_line_do_zwrotu_180_dni" model="account.tax.report.line">
        <field name="name">Podatek - Do zwrotu 180 dni</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="account_tax_report_line_podatek_do_zwrotu"/>
    </record>

</odoo>

```

## File: data\l10n_pl_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
<menuitem id="account_reports_pl_statements_menu" name="Poland" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>

<data noupdate="1">

    <record model="account.account.type" id="account_type_settlement">
      <field name="name">Rozrachunki</field>
      <field name="internal_group">off_balance</field>
    </record>

</data>
<data>
    <!-- Chart template -->

        <record id="pl_chart_template" model="account.chart.template">
            <field name="name">Polska - Plan kont</field>
            <field name="code_digits">6</field>
            <field name="currency_id" ref="base.PLN"/>
            <field name="bank_account_code_prefix">11-000-000</field>
            <field name="cash_account_code_prefix">12-000-000</field>
            <field name="transfer_account_code_prefix">11-090-000</field>
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
        <field name="property_account_expense_categ_id" ref="chart30020000"/>
        <field name="property_account_income_categ_id" ref="chart73010000"/>
        <field name="income_currency_exchange_account_id" ref="chart75060000"/>
        <field name="expense_currency_exchange_account_id" ref="pl_a_pay"/>
        <field name="default_pos_receivable_account_id" ref="chart20000100" />
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

