# Odoo Module: l10n_cz

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-

{
    'name': 'Czech - Accounting',
    'version': '1.0',
    'author': '26HOUSE',
    'website': 'http://www.26house.com',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Czech accounting chart and localization.  With Chart of Accounts with taxes and basic fiscal positions.

Tento modul definuje:

- Českou účetní osnovu za rok 2020

- Základní sazby pro DPH z prodeje a nákupu

- Základní fiskální pozice pro českou legislativu
    """,
    'depends': [
        'account',
        'base_iban',
        'base_vat',
    ],
    'data': [
          'data/l10n_cz_coa_data.xml',
          'data/account.account.template.csv',
          'data/account.group.template.csv',
          'data/l10n_cz_coa_post_data.xml',
          'data/account_data.xml',
          'data/account_tax_data.xml',
          'data/account_fiscal_position_data.xml',
          'data/account_chart_template_data.xml'
    ],
    'demo': ['data/demo_company.xml'],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","chart_template_id/id","user_type_id/id","reconcile"
"chart_cz_012000","Nehmotné výsledky výzkumu a vývoje","012000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_013000","Software","013000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_014000","Ostatní ocenitelná práva","014000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_015000","Goodwill","015000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_016000","Povolenky na emise","016000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_017000","Preferenční limity","017000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_019000","Ostatní dlouhodobý nehmotný majetek","019000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_021000","Stavby","021000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_022000","Hmotné movité věci a jejich soubory","022000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_025000","Pěstitelské celky trvalých porostů","025000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_026000","Dospělá zvířata a jejich skupiny","26000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_027000","Oceňovací rozdíl k nabytému majetku","027000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_029000","Jiný dlouhodobý hmotný majetek","029000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_031000","Pozemky","031000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_032000","Umělecká díla a sbírky","032000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_041000","Pořízení dlouhodobého nehmotného majetku","041000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_042000","Pořízení dlouhodobého hmotného majetku","042000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_043000","Pořízení dlouhodobého finančního majetku","043000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_051000","Poskytnuté zálohy na dlouhodobý nehmotný majetek","051000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_052000","Poskytnuté zálohy na dlouhodobý hmotný majetek","052000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_055000","Poskytnuté zálohy na dlouhodobý finanční majetek","055000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_061000","Podíly - ovládaná a ovládajicí osoba","061000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_062000","Podíly v účetních jednotkách pod podstatným vplyvem","062000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_063000","Ostatní dlouhodobé cenné papíry a podíly","063000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_065000","Dluhové cenné papíry držené do splatnosti","065000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_066000","Zápůjčky a úvéry - ovládaná neno ovládajicí osoba","066000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_067000","Zápůjčky a úvéry - podstatní vliv","067000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_068000","Ostatní zápůjčky a úvéry","068000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_069000","Jiný dlouhodobý finanční majetek","069000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_072000","Oprávky k nehmotným výsledkům výzkumu a vývoje","072000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_073000","Oprávky k softwaru","073000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_074000","Oprávky k ocenitelným právům","074000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_075000","Oprávky ke goodwillu","075000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_079000","Oprávky k ostatnímu dlouhodobému nehmotnému majetku","079000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_081000","Oprávky k stavbám","081000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_082000","Oprávky k hmotným movitým věcem a jejich souborům","082000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_085000","Oprávky k pěstitelským celkům trvalých porostů","085000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_086000","Oprávky k dospělým zvířatům a jejich skupinám","086000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_089000","Oprávky k jinému dlouhodobému hmotnému majetku","089000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_091000","Opravná položka k dlouhodobému nehmotnému majetku","091000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_092000","Opravná položka k dlouhodobému hmotnému majetku","092000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_093000","Opravná položka k dlouhodobému nedokončenému nehmotnému majetku","093000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_094000","Opravná položka k dlouhodobému nedokončenému hmotnému majetku","094000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_095000","Opravná položka k poskytnutým zálohám na dlouhodobý majetek","095000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_096000","Opravná položka k dlouhodobému finančnímu majetku","096000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_097000","Oceňovací rozdíl k nabytému majetku","097000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_098000","Oprávky k oceňovacímu rozdílu k nabytému majetku","098000","l10n_cz.cz_chart_template","account.data_account_type_non_current_assets","False"
"chart_cz_111000","Pořízení materiálu","111000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_112000","Materiál na skladě","112000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_119000","Materiál na cestě","119000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_121000","Nedokončená výroba","121000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_122000","Polotovary","122000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_123000","Výrobky","123000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_124000","Mladá a ostatní zvířata ajejich skupiny","124000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_131000","Pořízení zboží","131000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_132000","Zboží na skladě a v prodejnách","132000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_139000","Zboží na cestě","139000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_151000","Poskytnuté zálohy na materiál","151000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_152000","Poskytnuté zálohy na zvířata","152000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_153000","Poskytnuté zálohy na zboží","153000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_191000","Opravná položka k materiálu","191000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_192000","Opravná položka k nedokončené výrobě","192000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_193000","Opravná položka k polotovarům","193000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_194000","Opravná položka k výrobkům","194000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_195000","Opravná položka k mladým a ostatním zvířatům a jejich skupinám","195000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_196000","Opravná položka ke zboží","196000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_197000","Opravná položka k zálohám na materiál","197000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_198000","Opravná položka k zálohám na zvířata","198000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_199000","Opravná položka k zálohám na zboží","199000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_211000","Pokladna","211000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_213000","Ceniny","213000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_221000","Bankovní účty","221000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_231000","Krátkodobé úvěry","231000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_232000","Eskontní úvěry","232000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_241000","Emitované krátkodobé dluhopisy","241000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_249000","Ostatní krátkodobé finanční výpomoci","249000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_251000","Majetkové cenné papíry k obchodování","251000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_252000","Vlastní akcie a vlastní obchodní podíly","252000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_253000","Dluhové cenné papíry k obchodování","253000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_255000","Vlastní dluhopisy","255000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_256000","Dluhové cenné papíry se splatností do jednoho roku držené do splatnosti","256000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_257000","Ostatní cenné papíry","257000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_259000","Pořízování krátkodobého finančního majetku","259000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_261000","Peníze na cestě","261000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_291000","Opravná položka ke krátkodobému finančnímu majetku","291000","l10n_cz.cz_chart_template","account.data_account_type_liquidity","False"
"chart_cz_311000","Odběratelé","311000","l10n_cz.cz_chart_template","account.data_account_type_receivable","True"
"chart_cz_313000","Pohledávky za eskontované cenné papíry","313000","l10n_cz.cz_chart_template","account.data_account_type_receivable","True"
"chart_cz_314000","Poskytnuté provozní zálohy","314000","l10n_cz.cz_chart_template","account.data_account_type_receivable","True"
"chart_cz_315000","Ostatní pohledávky","315000","l10n_cz.cz_chart_template","account.data_account_type_receivable","True"
"chart_cz_321000","Dodavatelé","321000","l10n_cz.cz_chart_template","account.data_account_type_payable","True"
"chart_cz_322000","Směnky k úhradě","322000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_324000","Přijaté provozní zálohy","324000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_325000","Ostatní závazky","325000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_331000","Zaměstnanci","331000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_333000","Ostatní závazky vůči zaměstnancům","333000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_335000","Pohledávky za zaměstnanci","335000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_336000","Zúčtování s institucemi sociálního zabezpečení a zdravotního pojištění","336000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_341000","Daň z příjmů","341000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_342000","Ostatní přímé daně","342000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_343000","Daň z přidané hodnoty","343000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_343115","DPH snížená sazba vstup","343115","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_343121","DPH základní sazba vstup","343121","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_343215","DPH snížená sazba výstup","343215","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_343221","DPH základní sazba výstup","343221","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_345000","Ostatní daně a poplatky","345000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_346000","Dotace ze státního rozpočtu","346000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_347000","Ostatní dodace","347000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_349000","Spojovací účet k DPH","349000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_351000","Pohledávky – ovládaná nebo ovládající osoba","351000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_352000","Pohledávky – podstatný vliv","352000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_353000","Pohledávky za upsaný základní kapitál","353000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_354000","Pohledávky za společníky při úhradě ztráty","354000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_355000","Ostatní pohledávky za společníky obchodní korporace","355000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_358000","Pohledávky za společníky združenými ve společnosti","358000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_361000","Závazky – ovládaná nebo ovládající osoba","361000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_362000","Závazky - podstatný vliv","362000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_364000","Závazky ke společníkům při rozdělování zisku","364000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_365000","Ostatní závazky ke společníkům obchodní korporace","365000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_366000","Závazky ke společníkům a členům družstva ze závislé činnosti","366000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_367000","Závazky z upsaných nesplacených cenných papírů a vkladů","367000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_368000","Závazky ke společníkům sdruženým ve společnosti","368000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_371000","Pohledávky z prodeje závodu","371000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_372000","Závazky z koupě závodu","372000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_373000","Pohledávky a závazky z pevných termínových operací","373000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_374000","Pohledávky z nájmu a pachtu","374000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_375000","Pohledávky z emitovaných dluhopisů","375000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_376000","Nakoupené opce","376000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_377000","Prodané opce","377000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_378000","Jiné pohledávky","378000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_379000","Jiné závazky","379000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_381000","Náklady příštích období","381000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_382000","Komplexní náklady příštích období","382000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_383000","Výdaje příštích období","383000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_384000","Výnosy příštích období","384000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_385000","Příjmy příštích období","385000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_388000","Dohadné účty aktivní","388000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_389000","Dohadné účty pasivní","389000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_391000","Opravná položka k pohledávkám","391000","l10n_cz.cz_chart_template","account.data_account_type_current_assets","False"
"chart_cz_395000","Vnitřní zúčtování","395000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_398000","Spojovací účet při společnosti (sdružení)","398000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_411000","Základní kapitál","411000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_412000","Ážio","412000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_413000","Ostatní kapitálové fondy","413000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_414000","Oceňovací rozdíly z přecenění majetku a závazků","414000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_416000","Rozdíly z přecenění při přemenách obchodních korporací","416000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_417000","Rozdíly z přeměn obchodních korporací","417000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_418000","Oceňovací rozdíly z přecenění při přeměnách obchodních korporací","418000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_419000","Změny základního kapitálu","419000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_421000","Rezervní fond","421000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_422000","Nedělitelný fond","422000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_423000","Statutární fondy","423000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_424000","Ostatní fondy ze zisku","424000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_426000","Jiný výsledek hospodáření minulých let","426000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_428000","Nerozdělený zisk minulých let","428000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_429000","Neuhrazená ztráta minulých let","429000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_431000","Výsledek hospodaření v schvalovacím řízení","431000","l10n_cz.cz_chart_template","account.data_unaffected_earnings","False"
"chart_cz_432000","Zálohy na podíly na zisku","432000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_451000","Rezervy podle zvláštních právních předpisů","451000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_453000","Rezerva na daň z příjmů","453000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_459000","Ostatní rezervy","459000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_461000","Dlouhodobé úvěry","461000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_471000","Dlouhodobé závazky – ovládaná nebo ovládající osoba","471000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_472000","Dlouhodobé závazky – podstatný vliv","472000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_473000","Emitované dluhopisy","473000","l10n_cz.cz_chart_template","account.data_account_type_non_current_liabilities","False"
"chart_cz_474000","Závazky z nájmu a pachtu","474000","l10n_cz.cz_chart_template","account.data_account_type_current_liabilities","False"
"chart_cz_475000","Dlouhodobé přijaté zálohy","475000","l10n_cz.cz_chart_template","account.data_account_type_non_current_liabilities","False"
"chart_cz_478000","Dlouhodobé směnky k úhradě","478000","l10n_cz.cz_chart_template","account.data_account_type_non_current_liabilities","False"
"chart_cz_479000","Ostatní dlouhodobé závazky","479000","l10n_cz.cz_chart_template","account.data_account_type_non_current_liabilities","False"
"chart_cz_481000","Odložený daňový závazek a pohledávka","481000","l10n_cz.cz_chart_template","account.data_account_type_non_current_liabilities","False"
"chart_cz_491000","Účet individuálního podnikatele","491000","l10n_cz.cz_chart_template","account.data_account_type_equity","False"
"chart_cz_501000","Spotřeba materiálu","501000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_502000","Spotřeba energie","502000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_503000","Spotřeba ostatních neskladovatelných dodávek","503000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_504000","Prodané zboží","504000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_511000","Opravy a udržování","511000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_512000","Cestovné","512000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_513000","Náklady na reprezentaci","513000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_518000","Ostatní služby","518000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_521000","Mzdové náklady","521000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_522000","Příjmy společníků obchodní korporace ze závislé činnosti","522000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_523000","Odměny členům orgánů obchodní korporace","523000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_524000","Zákonné sociální a zdravotní pojištění","524000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_525000","Ostatní sociální pojištění","525000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_526000","Zdravotní a sociální pojištění individuálního podnikatele","526000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_527000","Zákonné sociální náklady","527000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_528000","Ostatní sociální náklady","528000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_531000","Daň silniční","531000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_532000","Daň z nemovitých věcí","532000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_538000","Ostatní daně a poplatky","538000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_541000","Zůstatková cena prodaného dlouhodobého nehmotného a hmotného majetku","541000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_542000","Prodaný materiál","542000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_543000","Dary","543000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_544000","Smluvní pokuty a úroky z prodlení","544000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_545000","Ostatní pokuty a penále","545000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_546000","Odpis pohledávky","546000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_547000","Mimořádné provozní náklady","547000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_548000","Ostatní provozní náklady","548000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_549000","Manka a škody z provozní činnosti","549000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_551000","Odpisy dlouhodobého nehmotného a hmotného majetku","551000","l10n_cz.cz_chart_template","account.data_account_type_depreciation","False"
"chart_cz_552000","Tvorba a zúčtování zákonných rezerv podle zvláštních právních předpisů","552000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_554000","Tvorba a zúčtování ostatních rezerv","554000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_555000","Tvorba a zúčtování komplexních nákladů příštích období","555000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_557000","Zúčtování oprávky k oceňovacímu rozdílu k nabytému majetku","557000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_558000","Tvorba a zúčtování zákonných opravných položek v provozní činnosti","558000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_559000","Tvorba a zúčtování opravných položek v provozní činnosti","559000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_561000","Prodané cenné papíry a podíly","561000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_562000","Úroky","562000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_563000","Kursové ztráty","563000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_564000","Náklady na přecenění cenných papírů","564000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_565000","Náklady z finančního majetku","565000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_566000","Náklady z derivátových operací","566000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_567000","Mimořádné finanční náklady","567000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_568000","Ostatní finanční náklady","568000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_569000","Manka a škody na finančním majetku","569000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_574000","Tvorba a zúčtování finančních rezerv","574000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_579000","Tvorba a zúčtování opravných položek ve finanční činnosti","579000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_581000","Změna stavu nedokončené výroby","581000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_582000","Změna stavu polotovarů","582000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_583000","Změna stavu výrobků","583000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_584000","Změna stavu mladých a ostatních zvířat","584000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_585000","Aktivace materiálu a zboží","585000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_586000","Aktivace vnitropodnikových služeb","586000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_587000","Aktivace dlouhodobého nehmotného majetku","587000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_588000","Aktivace dlouhodobého hmotného majetku","588000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_591000","Daň z příjmů splatná","591000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_592000","Daň z příjmů odložená","592000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_593000","Tvorba a zúčtování rezervy na daň z příjmů","593000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_595000","Dodatečné odvody daně z příjmů","595000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_596000","Převod podílu na výsledku hospodaření společníkům v.o.s. a komplementářům k.s.","596000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_597000","Převod provozních nákladů","597000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_598000","Převod finančních nákladů","598000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_599000","Náklady hospodářských středisek","599000","l10n_cz.cz_chart_template","account.data_account_type_expenses","False"
"chart_cz_601000","Tržby za vlastní výrobky","601000","l10n_cz.cz_chart_template","account.data_account_type_revenue","False"
"chart_cz_602000","Tržby z prodeje služeb","602000","l10n_cz.cz_chart_template","account.data_account_type_revenue","False"
"chart_cz_604000","Tržby za zboží","604000","l10n_cz.cz_chart_template","account.data_account_type_revenue","False"
"chart_cz_641000","Tržby z prodeje dlouhodobého nehmotného a hmotného majetku","641000","l10n_cz.cz_chart_template","account.data_account_type_revenue","False"
"chart_cz_642000","Tržby z prodeje materiálu","642000","l10n_cz.cz_chart_template","account.data_account_type_revenue","False"
"chart_cz_644000","Smluvní pokuty a úroky z prodlení","644000","l10n_cz.cz_chart_template","account.data_account_type_other_income","False"
"chart_cz_646000","Výnosy z odepsaných pohledávek","646000","l10n_cz.cz_chart_template","account.data_account_type_other_income","False"
"chart_cz_647000","Mimořádné provozní výnosy","647000","l10n_cz.cz_chart_template","account.data_account_type_other_income","False"
"chart_cz_648000","Ostatní provizní výnosy","648000","l10n_cz.cz_chart_template","account.data_account_type_other_income","False"
"chart_cz_661000","Tržby z prodeje cenných papírů a podílů","661000","l10n_cz.cz_chart_template","account.data_account_type_revenue","False"
"chart_cz_662000","Úroky","662000","l10n_cz.cz_chart_template","account.data_account_type_other_income","False"
"chart_cz_663000","Kurzové zisky","663000","l10n_cz.cz_chart_template","account.data_account_type_other_income","False"
"chart_cz_664000","Výnosy z přecenění cenných papírů","664000","l10n_cz.cz_chart_template","account.data_account_type_other_income","False"
"chart_cz_665000","Výnosy z finančního majetku","665000","l10n_cz.cz_chart_template","account.data_account_type_other_income","False"
"chart_cz_666000","Výnosy z derivátových operací","666000","l10n_cz.cz_chart_template","account.data_account_type_other_income","False"
"chart_cz_667000","Mimořádné finanční výnosy","667000","l10n_cz.cz_chart_template","account.data_account_type_other_income","False"
"chart_cz_668000","Ostatní finanční výnosy","668000","l10n_cz.cz_chart_template","account.data_account_type_other_income","False"
"chart_cz_697000","Převod provozních výnosů","697000","l10n_cz.cz_chart_template","account.data_account_type_other_income","False"
"chart_cz_698000","Převod finančních výnosů","698000","l10n_cz.cz_chart_template","account.data_account_type_other_income","False"
"chart_cz_699000","Výnosy hospodářských středisek","699000","l10n_cz.cz_chart_template","account.data_account_type_other_income","False"
"chart_cz_701000","Počáteční účet rozvažný","701000","l10n_cz.cz_chart_template","account.data_account_off_sheet","False"
"chart_cz_702000","Konečný účet rozvažný","702000","l10n_cz.cz_chart_template","account.data_account_off_sheet","False"
"chart_cz_710000","Účet zisků a ztrát","710000","l10n_cz.cz_chart_template","account.data_account_off_sheet","False"

```

## File: data\account.group.template.csv

```csv
"id","code_prefix_start","name","chart_template_id/id"
"chart_cz_0","0","Dlouhodobý majetek","l10n_cz.cz_chart_template"
"chart_cz_01","01","Dlouhodobý nehmotný majetek","l10n_cz.cz_chart_template"
"chart_cz_012","012","Nehmotné výsledky výzkumu a vývoje","l10n_cz.cz_chart_template"
"chart_cz_013","013","Software","l10n_cz.cz_chart_template"
"chart_cz_014","014","Ostatní ocenitelná práva","l10n_cz.cz_chart_template"
"chart_cz_015","015","Goodwill","l10n_cz.cz_chart_template"
"chart_cz_016","016","Povolenky na emise","l10n_cz.cz_chart_template"
"chart_cz_017","017","Preferenční limity","l10n_cz.cz_chart_template"
"chart_cz_019","019","Ostatní dlouhodobý nehmotný majetek","l10n_cz.cz_chart_template"
"chart_cz_02","02","Dlouhodobý hmotný majetek - odpisovaný","l10n_cz.cz_chart_template"
"chart_cz_021","021","Stavby","l10n_cz.cz_chart_template"
"chart_cz_022","022","Hmotné movité věci a jejich soubory","l10n_cz.cz_chart_template"
"chart_cz_025","025","Pěstitelské celky trvalých porostů","l10n_cz.cz_chart_template"
"chart_cz_026","026","Dospělá zvířata a jejich skupiny","l10n_cz.cz_chart_template"
"chart_cz_027","027","Oceňovací rozdíl k nabytému majetku","l10n_cz.cz_chart_template"
"chart_cz_029","029","Jiný dlouhodobý hmotný majetek","l10n_cz.cz_chart_template"
"chart_cz_03","03","Dlouhodobý hmotný majetek - neodpisovaný","l10n_cz.cz_chart_template"
"chart_cz_031","031","Pozemky","l10n_cz.cz_chart_template"
"chart_cz_032","032","Umělecká díla a sbírky","l10n_cz.cz_chart_template"
"chart_cz_04","04","Nedokončený dlouhodobý nehmotný a hmotný majetek","l10n_cz.cz_chart_template"
"chart_cz_041","041","Pořízení dlouhodobého nehmotného majetku","l10n_cz.cz_chart_template"
"chart_cz_042","042","Pořízení dlouhodobého hmotného majetku","l10n_cz.cz_chart_template"
"chart_cz_043","043","Pořízení dlouhodobého finančního majetku","l10n_cz.cz_chart_template"
"chart_cz_05","05","Poskytnuté zálohy na dlouhodobý majetek","l10n_cz.cz_chart_template"
"chart_cz_051","051","Poskytnuté zálohy na dlouhodobý nehmotný majetek","l10n_cz.cz_chart_template"
"chart_cz_052","052","Poskytnuté zálohy na dlouhodobý hmotný majetek","l10n_cz.cz_chart_template"
"chart_cz_055","055","Poskytnuté zálohy na dlouhodobý finanční majetek","l10n_cz.cz_chart_template"
"chart_cz_06","06","Dlouhodobý finanční majetek","l10n_cz.cz_chart_template"
"chart_cz_061","061","Podíly - ovládaná a ovládajicí osoba","l10n_cz.cz_chart_template"
"chart_cz_062","062","Podíly v účetních jednotkách pod podstatným vplyvem","l10n_cz.cz_chart_template"
"chart_cz_063","063","Ostatní dlouhodobé cenné papíry a podíly","l10n_cz.cz_chart_template"
"chart_cz_065","065","Dluhové cenné papíry držené do splatnosti","l10n_cz.cz_chart_template"
"chart_cz_066","066","Zápůjčky a úvéry - ovládaná neno ovládajicí osoba","l10n_cz.cz_chart_template"
"chart_cz_067","067","Zápůjčky a úvéry - podstatní vliv","l10n_cz.cz_chart_template"
"chart_cz_068","068","Ostatní zápůjčky a úvéry","l10n_cz.cz_chart_template"
"chart_cz_069","069","Jiný dlouhodobý finanční majetek","l10n_cz.cz_chart_template"
"chart_cz_07","07","Oprávky k dlouhodobému nehmotnému majetku","l10n_cz.cz_chart_template"
"chart_cz_072","072","Oprávky k nehmotným výsledkům výzkumu a vývoje","l10n_cz.cz_chart_template"
"chart_cz_073","073","Oprávky k softwaru","l10n_cz.cz_chart_template"
"chart_cz_074","074","Oprávky k ocenitelným právům","l10n_cz.cz_chart_template"
"chart_cz_075","075","Oprávky ke goodwillu","l10n_cz.cz_chart_template"
"chart_cz_079","079","Oprávky k ostatnímu dlouhodobému nehmotnému majetku","l10n_cz.cz_chart_template"
"chart_cz_08","08","Oprávky k dlouhodobému hmotnému majetku","l10n_cz.cz_chart_template"
"chart_cz_081","081","Oprávky k stavbám","l10n_cz.cz_chart_template"
"chart_cz_082","082","Oprávky k hmotným movitým věcem a jejich souborům","l10n_cz.cz_chart_template"
"chart_cz_085","085","Oprávky k pěstitelským celkům trvalých porostů","l10n_cz.cz_chart_template"
"chart_cz_086","086","Oprávky k dospělým zvířatům a jejich skupinám","l10n_cz.cz_chart_template"
"chart_cz_089","089","Oprávky k jinému dlouhodobému hmotnému majetku","l10n_cz.cz_chart_template"
"chart_cz_09","09","Opravné položky k dlouhodobému majetku","l10n_cz.cz_chart_template"
"chart_cz_091","091","Opravná položka k dlouhodobému nehmotnému majetku","l10n_cz.cz_chart_template"
"chart_cz_092","092","Opravná položka k dlouhodobému hmotnému majetku","l10n_cz.cz_chart_template"
"chart_cz_093","093","Opravná položka k dlouhodobému nedokončenému nehmotnému majetku","l10n_cz.cz_chart_template"
"chart_cz_094","094","Opravná položka k dlouhodobému nedokončenému hmotnému majetku","l10n_cz.cz_chart_template"
"chart_cz_095","095","Opravná položka k poskytnutým zálohám na dlouhodobý majetek","l10n_cz.cz_chart_template"
"chart_cz_096","096","Opravná položka k dlouhodobému finančnímu majetku","l10n_cz.cz_chart_template"
"chart_cz_097","097","Oceňovací rozdíl k nabytému majetku","l10n_cz.cz_chart_template"
"chart_cz_098","098","Oprávky k oceňovacímu rozdílu k nabytému majetku","l10n_cz.cz_chart_template"
"chart_cz_1","1","Zásoby","l10n_cz.cz_chart_template"
"chart_cz_11","11","Materiál","l10n_cz.cz_chart_template"
"chart_cz_111","111","Pořízení materiálu","l10n_cz.cz_chart_template"
"chart_cz_112","112","Materiál na skladě","l10n_cz.cz_chart_template"
"chart_cz_119","119","Materiál na cestě","l10n_cz.cz_chart_template"
"chart_cz_12","12","Zásoby vlastní činnosti","l10n_cz.cz_chart_template"
"chart_cz_121","121","Nedokončená výroba","l10n_cz.cz_chart_template"
"chart_cz_122","122","Polotovary","l10n_cz.cz_chart_template"
"chart_cz_123","123","Výrobky","l10n_cz.cz_chart_template"
"chart_cz_124","124","Mladá a ostatní zvířata ajejich skupiny","l10n_cz.cz_chart_template"
"chart_cz_13","13","Zboží","l10n_cz.cz_chart_template"
"chart_cz_131","131","Pořízení zboží","l10n_cz.cz_chart_template"
"chart_cz_132","132","Zboží na skladě a v prodejnách","l10n_cz.cz_chart_template"
"chart_cz_139","139","Zboží na cestě","l10n_cz.cz_chart_template"
"chart_cz_15","15","Poskytnuté zálohy na zásoby","l10n_cz.cz_chart_template"
"chart_cz_151","151","Poskytnuté zálohy na materiál","l10n_cz.cz_chart_template"
"chart_cz_152","152","Poskytnuté zálohy na zvířata","l10n_cz.cz_chart_template"
"chart_cz_153","153","Poskytnuté zálohy na zboží","l10n_cz.cz_chart_template"
"chart_cz_19","19","Opravné položky k zásobám","l10n_cz.cz_chart_template"
"chart_cz_191","191","Opravná položka k materiálu","l10n_cz.cz_chart_template"
"chart_cz_192","192","Opravná položka k nedokončené výrobě","l10n_cz.cz_chart_template"
"chart_cz_193","193","Opravná položka k polotovarům","l10n_cz.cz_chart_template"
"chart_cz_194","194","Opravná položka k výrobkům","l10n_cz.cz_chart_template"
"chart_cz_195","195","Opravná položka k mladým a ostatním zvířatům a jejich skupinám","l10n_cz.cz_chart_template"
"chart_cz_196","196","Opravná položka ke zboží","l10n_cz.cz_chart_template"
"chart_cz_197","197","Opravná položka k zálohám na materiál","l10n_cz.cz_chart_template"
"chart_cz_198","198","Opravná položka k zálohám na zvířata","l10n_cz.cz_chart_template"
"chart_cz_199","199","Opravná položka k zálohám na zboží","l10n_cz.cz_chart_template"
"chart_cz_2","2","Krátkodobý finanční majetek a peněžní prostředky","l10n_cz.cz_chart_template"
"chart_cz_21","21","Peněžní prostředky v pokladně","l10n_cz.cz_chart_template"
"chart_cz_211","211","Pokladna","l10n_cz.cz_chart_template"
"chart_cz_213","213","Ceniny","l10n_cz.cz_chart_template"
"chart_cz_22","22","Peněžní prostředky na účtech","l10n_cz.cz_chart_template"
"chart_cz_221","221","Bankovní účty","l10n_cz.cz_chart_template"
"chart_cz_23","23","Krátkodobé úvěry","l10n_cz.cz_chart_template"
"chart_cz_231","231","Krátkodobé úvěry","l10n_cz.cz_chart_template"
"chart_cz_232","232","Eskontní úvěry","l10n_cz.cz_chart_template"
"chart_cz_24","24","Krátkodobé finanční výpomoci","l10n_cz.cz_chart_template"
"chart_cz_241","241","Emitované krátkodobé dluhopisy","l10n_cz.cz_chart_template"
"chart_cz_249","249","Ostatní krátkodobé finanční výpomoci","l10n_cz.cz_chart_template"
"chart_cz_25","25","Krátkodobý finanční majetek","l10n_cz.cz_chart_template"
"chart_cz_251","251","Majetkové cenné papíry k obchodování","l10n_cz.cz_chart_template"
"chart_cz_252","252","Vlastní akcie a vlastní obchodní podíly","l10n_cz.cz_chart_template"
"chart_cz_253","253","Dluhové cenné papíry k obchodování","l10n_cz.cz_chart_template"
"chart_cz_255","255","Vlastní dluhopisy","l10n_cz.cz_chart_template"
"chart_cz_256","256","Dluhové cenné papíry se splatností do jednoho roku držené do splatnosti","l10n_cz.cz_chart_template"
"chart_cz_257","257","Ostatní cenné papíry","l10n_cz.cz_chart_template"
"chart_cz_259","259","Pořízování krátkodobého finančního majetku","l10n_cz.cz_chart_template"
"chart_cz_26","26","Převody mezi finančními účty","l10n_cz.cz_chart_template"
"chart_cz_261","261","Peníze na cestě","l10n_cz.cz_chart_template"
"chart_cz_29","29","Opravné položky ke krátkodobému finančnímu majetku","l10n_cz.cz_chart_template"
"chart_cz_291","291","Opravná položka ke krátkodobému finančnímu majetku","l10n_cz.cz_chart_template"
"chart_cz_3","3","Zúčtovací vztahy","l10n_cz.cz_chart_template"
"chart_cz_31","31","Pohledávky","l10n_cz.cz_chart_template"
"chart_cz_311","311","Odběratelé","l10n_cz.cz_chart_template"
"chart_cz_313","313","Pohledávky za eskontované cenné papíry","l10n_cz.cz_chart_template"
"chart_cz_314","314","Poskytnuté provozní zálohy","l10n_cz.cz_chart_template"
"chart_cz_315","315","Ostatní pohledávky","l10n_cz.cz_chart_template"
"chart_cz_32","32","Závazky (krátkodobé)","l10n_cz.cz_chart_template"
"chart_cz_321","321","Dodavatelé","l10n_cz.cz_chart_template"
"chart_cz_322","322","Směnky k úhradě","l10n_cz.cz_chart_template"
"chart_cz_324","324","Přijaté provozní zálohy","l10n_cz.cz_chart_template"
"chart_cz_325","325","Ostatní závazky","l10n_cz.cz_chart_template"
"chart_cz_33","33","Zúčtování se zaměstnanci a institucemi","l10n_cz.cz_chart_template"
"chart_cz_331","331","Zaměstnanci","l10n_cz.cz_chart_template"
"chart_cz_333","333","Ostatní závazky vůči zaměstnancům","l10n_cz.cz_chart_template"
"chart_cz_335","335","Pohledávky za zaměstnanci","l10n_cz.cz_chart_template"
"chart_cz_336","336","Zúčtování s institucemi sociálního zabezpečení a zdravotního pojištění","l10n_cz.cz_chart_template"
"chart_cz_34","34","Zúčtování daní a dotací","l10n_cz.cz_chart_template"
"chart_cz_341","341","Daň z příjmů","l10n_cz.cz_chart_template"
"chart_cz_342","342","Ostatní přímé daně","l10n_cz.cz_chart_template"
"chart_cz_343","343","Daň z přidané hodnoty","l10n_cz.cz_chart_template"
"chart_cz_345","345","Ostatní daně a poplatky","l10n_cz.cz_chart_template"
"chart_cz_346","346","Dotace ze státního rozpočtu","l10n_cz.cz_chart_template"
"chart_cz_347","347","Ostatní dodace","l10n_cz.cz_chart_template"
"chart_cz_349","349","Spojovací účet k DPH","l10n_cz.cz_chart_template"
"chart_cz_35","35","Pohledávky za společníky","l10n_cz.cz_chart_template"
"chart_cz_351","351","Pohledávky – ovládaná nebo ovládající osoba","l10n_cz.cz_chart_template"
"chart_cz_352","352","Pohledávky – podstatný vliv","l10n_cz.cz_chart_template"
"chart_cz_353","353","Pohledávky za upsaný základní kapitál","l10n_cz.cz_chart_template"
"chart_cz_354","354","Pohledávky za společníky při úhradě ztráty","l10n_cz.cz_chart_template"
"chart_cz_355","355","Ostatní pohledávky za společníky obchodní korporace","l10n_cz.cz_chart_template"
"chart_cz_358","358","Pohledávky za společníky združenými ve společnosti","l10n_cz.cz_chart_template"
"chart_cz_36","36","Závazky ke společníkům","l10n_cz.cz_chart_template"
"chart_cz_361","361","Závazky – ovládaná nebo ovládající osoba","l10n_cz.cz_chart_template"
"chart_cz_362","362","Závazky - podstatný vliv","l10n_cz.cz_chart_template"
"chart_cz_364","364","Závazky ke společníkům při rozdělování zisku","l10n_cz.cz_chart_template"
"chart_cz_365","365","Ostatní závazky ke společníkům obchodní korporace","l10n_cz.cz_chart_template"
"chart_cz_366","366","Závazky ke společníkům a členům družstva ze závislé činnosti","l10n_cz.cz_chart_template"
"chart_cz_367","367","Závazky z upsaných nesplacených cenných papírů a vkladů","l10n_cz.cz_chart_template"
"chart_cz_368","368","Závazky ke společníkům sdruženým ve společnosti","l10n_cz.cz_chart_template"
"chart_cz_37","37","Jiné pohledávky a závazky","l10n_cz.cz_chart_template"
"chart_cz_371","371","Pohledávky z prodeje závodu","l10n_cz.cz_chart_template"
"chart_cz_372","372","Závazky z koupě závodu","l10n_cz.cz_chart_template"
"chart_cz_373","373","Pohledávky a závazky z pevných termínových operací","l10n_cz.cz_chart_template"
"chart_cz_374","374","Pohledávky z nájmu a pachtu","l10n_cz.cz_chart_template"
"chart_cz_375","375","Pohledávky z emitovaných dluhopisů","l10n_cz.cz_chart_template"
"chart_cz_376","376","Nakoupené opce","l10n_cz.cz_chart_template"
"chart_cz_377","377","Prodané opce","l10n_cz.cz_chart_template"
"chart_cz_378","378","Jiné pohledávky","l10n_cz.cz_chart_template"
"chart_cz_379","379","Jiné závazky","l10n_cz.cz_chart_template"
"chart_cz_38","38","Přechodné účty aktiv a pasiv","l10n_cz.cz_chart_template"
"chart_cz_381","381","Náklady příštích období","l10n_cz.cz_chart_template"
"chart_cz_382","382","Komplexní náklady příštích období","l10n_cz.cz_chart_template"
"chart_cz_383","383","Výdaje příštích období","l10n_cz.cz_chart_template"
"chart_cz_384","384","Výnosy příštích období","l10n_cz.cz_chart_template"
"chart_cz_385","385","Příjmy příštích období","l10n_cz.cz_chart_template"
"chart_cz_388","388","Dohadné účty aktivní","l10n_cz.cz_chart_template"
"chart_cz_389","389","Dohadné účty pasivní","l10n_cz.cz_chart_template"
"chart_cz_39","39","Opravná položka k zúčtovacím vztahům a vnitřní zúčtování","l10n_cz.cz_chart_template"
"chart_cz_391","391","Opravná položka k pohledávkám","l10n_cz.cz_chart_template"
"chart_cz_395","395","Vnitřní zúčtování","l10n_cz.cz_chart_template"
"chart_cz_398","398","Spojovací účet při společnosti (sdružení)","l10n_cz.cz_chart_template"
"chart_cz_4","4","Kapitálové účty a dlouhodobé závazky","l10n_cz.cz_chart_template"
"chart_cz_41","41","Základní kapitál a kapitálové fondy","l10n_cz.cz_chart_template"
"chart_cz_411","411","Základní kapitál","l10n_cz.cz_chart_template"
"chart_cz_412","412","Ážio","l10n_cz.cz_chart_template"
"chart_cz_413","413","Ostatní kapitálové fondy","l10n_cz.cz_chart_template"
"chart_cz_414","414","Oceňovací rozdíly z přecenění majetku a závazků","l10n_cz.cz_chart_template"
"chart_cz_416","416","Rozdíly z přecenění při přemenách obchodních korporací","l10n_cz.cz_chart_template"
"chart_cz_417","417","Rozdíly z přeměn obchodních korporací","l10n_cz.cz_chart_template"
"chart_cz_418","418","Oceňovací rozdíly z přecenění při přeměnách obchodních korporací","l10n_cz.cz_chart_template"
"chart_cz_419","419","Změny základního kapitálu","l10n_cz.cz_chart_template"
"chart_cz_42","42","Fondy ze zisku a převedené výsledky hospodaření","l10n_cz.cz_chart_template"
"chart_cz_421","421","Rezervní fond","l10n_cz.cz_chart_template"
"chart_cz_422","422","Nedělitelný fond","l10n_cz.cz_chart_template"
"chart_cz_423","423","Statutární fondy","l10n_cz.cz_chart_template"
"chart_cz_424","424","Ostatní fondy ze zisku","l10n_cz.cz_chart_template"
"chart_cz_426","426","Jiný výsledek hospodáření minulých let","l10n_cz.cz_chart_template"
"chart_cz_428","428","Nerozdělený zisk minulých let","l10n_cz.cz_chart_template"
"chart_cz_429","429","Neuhrazená ztráta minulých let","l10n_cz.cz_chart_template"
"chart_cz_43","43","Výsledek hospodaření","l10n_cz.cz_chart_template"
"chart_cz_431","431","Výsledek hospodaření v schvalovacím řízení","l10n_cz.cz_chart_template"
"chart_cz_432","432","Zálohy na podíly na zisku","l10n_cz.cz_chart_template"
"chart_cz_45","45","Rezervy","l10n_cz.cz_chart_template"
"chart_cz_451","451","Rezervy podle zvláštních právních předpisů","l10n_cz.cz_chart_template"
"chart_cz_453","453","Rezerva na daň z příjmů","l10n_cz.cz_chart_template"
"chart_cz_459","459","Ostatní rezervy","l10n_cz.cz_chart_template"
"chart_cz_46","46","Dlouhodobé závazky k úvěrovým institucím","l10n_cz.cz_chart_template"
"chart_cz_461","461","Dlouhodobé úvěry","l10n_cz.cz_chart_template"
"chart_cz_47","47","dlouhodobé závazky","l10n_cz.cz_chart_template"
"chart_cz_471","471","Dlouhodobé závazky – ovládaná nebo ovládající osoba","l10n_cz.cz_chart_template"
"chart_cz_472","472","Dlouhodobé závazky – podstatný vliv","l10n_cz.cz_chart_template"
"chart_cz_473","473","Emitované dluhopisy","l10n_cz.cz_chart_template"
"chart_cz_474","474","Závazky z nájmu a pachtu","l10n_cz.cz_chart_template"
"chart_cz_475","475","Dlouhodobé přijaté zálohy","l10n_cz.cz_chart_template"
"chart_cz_478","478","Dlouhodobé směnky k úhradě","l10n_cz.cz_chart_template"
"chart_cz_479","479","Ostatní dlouhodobé závazky","l10n_cz.cz_chart_template"
"chart_cz_48","48","Odložený daňový závazek a pohledávka","l10n_cz.cz_chart_template"
"chart_cz_481","481","Odložený daňový závazek a pohledávka","l10n_cz.cz_chart_template"
"chart_cz_49","49","Individuální podnikatel","l10n_cz.cz_chart_template"
"chart_cz_491","491","Účet individuálního podnikatele","l10n_cz.cz_chart_template"
"chart_cz_5","5","Náklady","l10n_cz.cz_chart_template"
"chart_cz_50","50","Spotřebované nákupy","l10n_cz.cz_chart_template"
"chart_cz_501","501","Spotřeba materiálu","l10n_cz.cz_chart_template"
"chart_cz_502","502","Spotřeba energie","l10n_cz.cz_chart_template"
"chart_cz_503","503","Spotřeba ostatních neskladovatelných dodávek","l10n_cz.cz_chart_template"
"chart_cz_504","504","Prodané zboží","l10n_cz.cz_chart_template"
"chart_cz_51","51","Služby","l10n_cz.cz_chart_template"
"chart_cz_511","511","Opravy a udržování","l10n_cz.cz_chart_template"
"chart_cz_512","512","Cestovné","l10n_cz.cz_chart_template"
"chart_cz_513","513","Náklady na reprezentaci","l10n_cz.cz_chart_template"
"chart_cz_518","518","Ostatní služby","l10n_cz.cz_chart_template"
"chart_cz_52","52","Osobní náklady","l10n_cz.cz_chart_template"
"chart_cz_521","521","Mzdové náklady","l10n_cz.cz_chart_template"
"chart_cz_522","522","Příjmy společníků obchodní korporace ze závislé činnosti","l10n_cz.cz_chart_template"
"chart_cz_523","523","Odměny členům orgánů obchodní korporace","l10n_cz.cz_chart_template"
"chart_cz_524","524","Zákonné sociální a zdravotní pojištění","l10n_cz.cz_chart_template"
"chart_cz_525","525","Ostatní sociální pojištění","l10n_cz.cz_chart_template"
"chart_cz_526","526","Zdravotní a sociální pojištění individuálního podnikatele","l10n_cz.cz_chart_template"
"chart_cz_527","527","Zákonné sociální náklady","l10n_cz.cz_chart_template"
"chart_cz_528","528","Ostatní sociální náklady","l10n_cz.cz_chart_template"
"chart_cz_53","53","Daně a poplatky","l10n_cz.cz_chart_template"
"chart_cz_531","531","Daň silniční","l10n_cz.cz_chart_template"
"chart_cz_532","532","Daň z nemovitých věcí","l10n_cz.cz_chart_template"
"chart_cz_538","538","Ostatní daně a poplatky","l10n_cz.cz_chart_template"
"chart_cz_54","54","Jiné provozní náklady","l10n_cz.cz_chart_template"
"chart_cz_541","541","Zůstatková cena prodaného dlouhodobého nehmotného a hmotného majetku","l10n_cz.cz_chart_template"
"chart_cz_542","542","Prodaný materiál","l10n_cz.cz_chart_template"
"chart_cz_543","543","Dary","l10n_cz.cz_chart_template"
"chart_cz_544","544","Smluvní pokuty a úroky z prodlení","l10n_cz.cz_chart_template"
"chart_cz_545","545","Ostatní pokuty a penále","l10n_cz.cz_chart_template"
"chart_cz_546","546","Odpis pohledávky","l10n_cz.cz_chart_template"
"chart_cz_547","547","Mimořádné provozní náklady","l10n_cz.cz_chart_template"
"chart_cz_548","548","Ostatní provozní náklady","l10n_cz.cz_chart_template"
"chart_cz_549","549","Manka a škody z provozní činnosti","l10n_cz.cz_chart_template"
"chart_cz_55","55","Odpisy, rezervy, komplexní náklady příštích období a opravné položky v provozní oblasti","l10n_cz.cz_chart_template"
"chart_cz_551","551","Odpisy dlouhodobého nehmotného a hmotného majetku","l10n_cz.cz_chart_template"
"chart_cz_552","552","Tvorba a zúčtování zákonných rezerv podle zvláštních právních předpisů","l10n_cz.cz_chart_template"
"chart_cz_554","554","Tvorba a zúčtování ostatních rezerv","l10n_cz.cz_chart_template"
"chart_cz_555","555","Tvorba a zúčtování komplexních nákladů příštích období","l10n_cz.cz_chart_template"
"chart_cz_557","557","Zúčtování oprávky k oceňovacímu rozdílu k nabytému majetku","l10n_cz.cz_chart_template"
"chart_cz_558","558","Tvorba a zúčtování zákonných opravných položek v provozní činnosti","l10n_cz.cz_chart_template"
"chart_cz_559","559","Tvorba a zúčtování opravných položek v provozní činnosti","l10n_cz.cz_chart_template"
"chart_cz_56","56","Finanční náklady","l10n_cz.cz_chart_template"
"chart_cz_561","561","Prodané cenné papíry a podíly","l10n_cz.cz_chart_template"
"chart_cz_562","562","Úroky","l10n_cz.cz_chart_template"
"chart_cz_563","563","Kursové ztráty","l10n_cz.cz_chart_template"
"chart_cz_564","564","Náklady na přecenění cenných papírů","l10n_cz.cz_chart_template"
"chart_cz_565","565","Náklady z finančního majetku","l10n_cz.cz_chart_template"
"chart_cz_566","566","Náklady z derivátových operací","l10n_cz.cz_chart_template"
"chart_cz_567","567","Mimořádné finanční náklady","l10n_cz.cz_chart_template"
"chart_cz_568","568","Ostatní finanční náklady","l10n_cz.cz_chart_template"
"chart_cz_569","569","Manka a škody na finančním majetku","l10n_cz.cz_chart_template"
"chart_cz_57","57","Rezervy a opravné položky ve finanční oblasti","l10n_cz.cz_chart_template"
"chart_cz_574","574","Tvorba a zúčtování finančních rezerv","l10n_cz.cz_chart_template"
"chart_cz_579","579","Tvorba a zúčtování opravných položek ve finanční činnosti","l10n_cz.cz_chart_template"
"chart_cz_58","58","Změny stavu zásob vlastní činnosti a aktivace","l10n_cz.cz_chart_template"
"chart_cz_581","581","Změna stavu nedokončené výroby","l10n_cz.cz_chart_template"
"chart_cz_582","582","Změna stavu polotovarů","l10n_cz.cz_chart_template"
"chart_cz_583","583","Změna stavu výrobků","l10n_cz.cz_chart_template"
"chart_cz_584","584","Změna stavu mladých a ostatních zvířat","l10n_cz.cz_chart_template"
"chart_cz_585","585","Aktivace materiálu a zboží","l10n_cz.cz_chart_template"
"chart_cz_586","586","Aktivace vnitropodnikových služeb","l10n_cz.cz_chart_template"
"chart_cz_587","587","Aktivace dlouhodobého nehmotného majetku","l10n_cz.cz_chart_template"
"chart_cz_588","588","Aktivace dlouhodobého hmotného majetku","l10n_cz.cz_chart_template"
"chart_cz_59","59","Daně z příjmů, převodové účty a rezerva na daň z příjmů","l10n_cz.cz_chart_template"
"chart_cz_591","591","Daň z příjmů splatná","l10n_cz.cz_chart_template"
"chart_cz_592","592","Daň z příjmů odložená","l10n_cz.cz_chart_template"
"chart_cz_593","593","Tvorba a zúčtování rezervy na daň z příjmů","l10n_cz.cz_chart_template"
"chart_cz_595","595","Dodatečné odvody daně z příjmů","l10n_cz.cz_chart_template"
"chart_cz_596","596","Převod podílu na výsledku hospodaření společníkům v.o.s. a komplementářům k.s.","l10n_cz.cz_chart_template"
"chart_cz_597","597","Převod provozních nákladů","l10n_cz.cz_chart_template"
"chart_cz_598","598","Převod finančních nákladů","l10n_cz.cz_chart_template"
"chart_cz_599","599","Náklady hospodářských středisek","l10n_cz.cz_chart_template"
"chart_cz_6","6"," Výnosy","l10n_cz.cz_chart_template"
"chart_cz_60","60","Tržby za vlastní výkony a zboží","l10n_cz.cz_chart_template"
"chart_cz_601","601","Tržby za vlastní výrobky","l10n_cz.cz_chart_template"
"chart_cz_602","602","Tržby z prodeje služeb","l10n_cz.cz_chart_template"
"chart_cz_604","604","Tržby za zboží","l10n_cz.cz_chart_template"
"chart_cz_64","64","Jiné provozní výnosy","l10n_cz.cz_chart_template"
"chart_cz_641","641","Tržby z prodeje dlouhodobého nehmotného a hmotného majetku","l10n_cz.cz_chart_template"
"chart_cz_642","642","Tržby z prodeje materiálu","l10n_cz.cz_chart_template"
"chart_cz_644","644","Smluvní pokuty a úroky z prodlení","l10n_cz.cz_chart_template"
"chart_cz_646","646","Výnosy z odepsaných pohledávek","l10n_cz.cz_chart_template"
"chart_cz_647","647","Mimořádné provozní výnosy","l10n_cz.cz_chart_template"
"chart_cz_648","648","Ostatní provizní výnosy","l10n_cz.cz_chart_template"
"chart_cz_66","66","Finanční výnosy","l10n_cz.cz_chart_template"
"chart_cz_661","661","Tržby z prodeje cenných papírů a podílů","l10n_cz.cz_chart_template"
"chart_cz_662","662","Úroky","l10n_cz.cz_chart_template"
"chart_cz_663","663","Kurzové zisky","l10n_cz.cz_chart_template"
"chart_cz_664","664","Výnosy z přecenění cenných papírů","l10n_cz.cz_chart_template"
"chart_cz_665","665","Výnosy z finančního majetku","l10n_cz.cz_chart_template"
"chart_cz_666","666","Výnosy z derivátových operací","l10n_cz.cz_chart_template"
"chart_cz_667","667","Mimořádné finanční výnosy","l10n_cz.cz_chart_template"
"chart_cz_668","668","Ostatní finanční výnosy","l10n_cz.cz_chart_template"
"chart_cz_69","69","Převodové účty","l10n_cz.cz_chart_template"
"chart_cz_697","697","Převod provozních výnosů","l10n_cz.cz_chart_template"
"chart_cz_698","698","Převod finančních výnosů","l10n_cz.cz_chart_template"
"chart_cz_699","699","Výnosy hospodářských středisek","l10n_cz.cz_chart_template"
"chart_cz_7","7","Závěrkové účty a podrozvahové účty","l10n_cz.cz_chart_template"
"chart_cz_70","70","Rozvahové závěrkové účty","l10n_cz.cz_chart_template"
"chart_cz_701","701","Počáteční účet rozvažný","l10n_cz.cz_chart_template"
"chart_cz_702","702","Konečný účet rozvažný","l10n_cz.cz_chart_template"
"chart_cz_71","71","Výsledovkový zisků a účet","l10n_cz.cz_chart_template"
"chart_cz_710","710","Účet zisků a ztrát","l10n_cz.cz_chart_template"

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_cz.cz_chart_template')]"/>
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
        <record id="tax_group_vat_21" model="account.tax.group">
            <field name="name">DPH 21%</field>
        </record>
        <record id="tax_group_vat_15" model="account.tax.group">
            <field name="name">DPH 15%</field>
        </record>
        <record id="tax_group_vat_0" model="account.tax.group">
            <field name="name">DPH 0%</field>
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
            <field name="name">Obchody v CZ</field>
            <field name="chart_template_id" ref="cz_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_id" ref="base.cz"/>
        </record>

        <record id="fp_intra_private" model="account.fiscal.position.template">
            <field name="sequence">2</field>
            <field name="name">Obchody s EU konzument</field>
            <field name="chart_template_id" ref="cz_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>
      
        <record id="fiscal_position_template_2" model="account.fiscal.position.template">
            <field name="sequence">3</field>
            <field name="name">Obchody s EU</field>
            <field name="chart_template_id" ref="cz_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>

    <!-- Fiscal Position Tax Templates -->

        <!-- European Union -->
        <!-- Sales -->
        <record id="fiscal_position_tax_template_2a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vyc_tuz_21" />
            <field name="tax_dest_id" ref="vyc_dod_eu" />
        </record>

        <record id="fiscal_position_tax_template_2b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vyc_tuz_15" />
            <field name="tax_dest_id" ref="vyc_dod_eu" />
        </record>

        <record id="fiscal_position_tax_template_2c" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vyc_tuz_0" />
            <field name="tax_dest_id" ref="vyc_dod_eu" />
        </record>

        <!-- European Union -->
        <!-- Purchase -->

        <record id="fiscal_position_tax_template_21a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vsc_tuz_21" />
            <field name="tax_dest_id" ref="vsc_nad_eu" />
        </record>

        <record id="fiscal_position_tax_template_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vsc_tuz_15" />
            <field name="tax_dest_id" ref="vsc_nad_eu" />
        </record>
        
    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- VAT domestic sale-->
    <record id="vyc_tuz_21" model="account.tax.template">
        <field name="chart_template_id" ref="cz_chart_template"/>
        <field name="name">DPH na výstupu 21%</field>
        <field name="description">21%</field>
        <field name="amount">21</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'base',
            }),
            (0,0, {
              'factor_percent': 100,
              'repartition_type': 'tax',
              'account_id': ref('chart_cz_343221'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart_cz_343221'),
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_vat_21"/>
    </record>

    <record id="vyc_tuz_15" model="account.tax.template">
        <field name="chart_template_id" ref="cz_chart_template"/>
        <field name="name">DPH na výstupu 15%</field>
        <field name="description">15%</field>
        <field name="amount">15</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart_cz_343215'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart_cz_343215'),
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_vat_15"/>
    </record>

    <record id="vyc_tuz_0" model="account.tax.template">
        <field name="chart_template_id" ref="cz_chart_template"/>
        <field name="name">DPH na výstupu 0%</field>
        <field name="description">0%</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
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
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
    </record>

    <!-- VAT domestic purchase -->

    <record id="vsc_tuz_21" model="account.tax.template">
        <field name="chart_template_id" ref="cz_chart_template"/>
        <field name="name">DPH na vstupu 21%</field>
        <field name="description">21%</field>
        <field name="amount">21</field>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="0"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_21"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart_cz_343121'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart_cz_343121'),
            }),
        ]"/>
    </record>

    <record id="vsc_tuz_15" model="account.tax.template">
        <field name="chart_template_id" ref="cz_chart_template"/>
        <field name="name">DPH na vstupu 15%</field>
        <field name="description">15%</field>
        <field name="amount">15</field>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="0"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart_cz_343115'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart_cz_343115'),
            }),
        ]"/>
    </record>


    <!-- European Union -->
    <!-- =========================================================== -->

    <record id="vyc_dod_eu" model="account.tax.template">
        <field name="chart_template_id" ref="cz_chart_template"/>
        <field name="name">Dodání do EU</field>
        <field name="description">0%</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
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
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
    </record>

    <record id="vsc_nad_eu" model="account.tax.template">
        <field name="chart_template_id" ref="cz_chart_template"/>
        <field name="name">Pořízení z EU</field>
        <field name="description">21%</field>
        <field name="amount">21.0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart_cz_343121'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('chart_cz_343221'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart_cz_343121'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('chart_cz_343221'),
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_vat_21"/>
    </record>
</odoo>

```

## File: data\demo_company.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="partner_demo_company_cz" model="res.partner">
        <field name="name">CZ Company</field>
        <field name="vat">CZ12345679</field>
        <field name="street">Pařížská Street 25/31</field>
        <field name="city">Praha</field>
        <field name="country_id" ref="base.cz"/>
        <field name="state_id" ref="base.state_L"/>
        <field name="zip"></field>
        <field name="phone">+420 5 12 34 56 78</field>
        <field name="email">info@company.czexample.com</field>
        <field name="website">www.czexample.com</field>
    </record>

    <record id="demo_company_cz" model="res.company">
        <field name="name">CZ Company</field>
        <field name="partner_id" ref="partner_demo_company_cz"/>
    </record>

    <function model="res.company" name="_onchange_country_id">
        <value eval="[ref('demo_company_cz')]"/>
    </function>

    <function model="res.users" name="write">
        <value eval="[ref('base.user_root'), ref('base.user_admin'), ref('base.user_demo')]"/>
        <value eval="{'company_ids': [(4, ref('l10n_cz.demo_company_cz'))]}"/>
    </function>

    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_cz.cz_chart_template')]"/>
        <value model="res.company" eval="obj().env.ref('l10n_cz.demo_company_cz')"/>
    </function>
</odoo>

```

## File: data\l10n_cz_coa_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Chart template -->
        <record id="cz_chart_template" model="account.chart.template">
            <field name="name">Česká účetní osnova</field>
            <field name="code_digits">6</field>
            <field name="bank_account_code_prefix">221</field>
            <field name="cash_account_code_prefix">211</field>
            <field name="transfer_account_code_prefix">261</field>
            <field name="currency_id" ref="base.CZK"/>
        </record>
    </data>
</odoo>

```

## File: data\l10n_cz_coa_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="cz_chart_template" model="account.chart.template">
        <field name="account_journal_suspense_account_id" ref="chart_cz_261000"/>
        <field name="property_account_receivable_id" ref="chart_cz_311000"/>
        <field name="property_account_payable_id" ref="chart_cz_321000"/>
        <field name="property_account_expense_categ_id" ref="chart_cz_504000"/>
        <field name="property_account_income_categ_id" ref="chart_cz_604000"/>
        <field name="property_account_expense_id" ref="chart_cz_504000"/>
        <field name="property_account_income_id" ref="chart_cz_604000"/>
        <field name="income_currency_exchange_account_id" ref="chart_cz_663000"/>
        <field name="expense_currency_exchange_account_id" ref="chart_cz_563000"/>
        <field name="default_cash_difference_income_account_id" ref="chart_cz_668000"/>
        <field name="default_cash_difference_expense_account_id" ref="chart_cz_568000"/>
        <field name="property_stock_account_input_categ_id" ref="chart_cz_131000"/>
        <field name="property_stock_account_output_categ_id" ref="chart_cz_504000"/>
        <field name="property_stock_valuation_account_id" ref="chart_cz_132000"/>
    </record>
</odoo>

```

