# Odoo Module: l10n_sk

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
    'name': 'Slovak - Accounting',
    'version': '1.1',
    'author': '26HOUSE',
    'website': 'http://www.26house.com',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Slovakia accounting chart and localization: Chart of Accounts 2020, basic VAT rates + 
fiscal positions.

Tento modul definuje:
• Slovenskú účtovú osnovu za rok 2020

• Základné sadzby pre DPH z predaja a nákupu

• Základné fiškálne pozície pre slovenskú legislatívu

 
Pre viac informácií kontaktujte info@26house.com alebo navštívte https://www.26house.com.
    
    """,
    'depends': [
        'account',
        'base_iban',
        'base_vat',
    ],
    'data': [
          'data/l10n_sk_coa_data.xml',
          'data/account.account.template.csv',
          'data/account.group.template.csv',
          'data/l10n_sk_coa_post_data.xml',
          'data/account_tax_group_data.xml',
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
"id","name","code","chart_template_id/id","account_type","reconcile"
"chart_sk_012000","Aktivované náklady na vývoj","012000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_013000","Softvér","013000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_014000","Oceniteľné práva","014000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_015000","Goodwill","015000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_019000","Ostatný dlhodobý nehmotný majetok","019000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_021000","Stavby","021000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_022000","Samostatné hnuteľné veci a súbory hnuteľných vecí","022000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_025000","Pestovateľské celky trvalých porastov","025000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_026000","Základné stádo a ťažné zvieratá","026000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_029000","Ostatný dlhodobý hmotný majetok","029000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_031000","Pozemky","031000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_032000","Umelecké diela a zbierky","032000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_041000","Obstaranie dlhodobého nehmotného majetku","041000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_042000","Obstaranie dlhodobého hmotného majetku","042000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_043000","Obstaranie dlhodobého finančného majetku","043000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_051000","Poskytnuté preddavky na dlhodobý nehmotný majetok","051000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_052000","Poskytnuté preddavky na dlhodobý hmotný majetok","052000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_055000","Poskytnuté preddavky na dlhodobý finančný majetok","055000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_061000","Podielové cenné papiere a podiely v dcérskej účtovnej jednotke","061000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_062000","Podielové cenné papiere a podiely v spoločnosti alebo družstve s podielovou účasťou","062000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_063000","Realizovateľné cenné papiere a podiely","063000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_065000","Dlhové cenné papiere držané do splatnosti","065000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_066000","Pôžičky prepojeným účtovným jednotkám a účtovným jednotkám v rámci podielovej účasti","066000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_067000","Ostatné pôžičky","067000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_069000","Ostatný dlhodobý finančný majetok","069000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_072000","Oprávky k aktivovaným nákladom na vývoj","072000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_073000","Oprávky k softvéru","073000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_074000","Oprávky k oceniteľným právam","074000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_075000","Oprávky ku goodwilu","075000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_079000","Opravky k ostatnému dlhodobému nehmotnému majetku","079000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_081000","Oprávky k stavbám","081000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_082000","Oprávky k samostatným hnuteľným veciam a k súboru hnuteľných vecí","082000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_085000","Oprávky k pestovateľským celkom trvalých porastov","085000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_086000","Oprávky k základnému stádu a ťažným zvieratám","086000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_089000","Oprávky k ostatnému dlhodobému hmotnému majetku","089000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_091000","Opravné položky k dlhodobému nehmotnému majetku","091000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_092000","Opravné položky k dlhodobému hmotnému majetku","092000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_093000","Opravné položky k nedokončenému dlhodobému nehmotnému majetku","093000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_094000","Opravné položky k nedokončenému dlhodobému hmotnému majetku","094000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_095000","Opravné položky k poskytnutým preddavkom na dlhodobý majetok","095000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_096000","Opravné položky k dlhodobému finančnému majetku","096000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_097000","Opravné položky k nadobudnutému majetku","097000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_098000","Oprávky k opravnej položke k nadobudnutému majetku","098000","l10n_sk.sk_chart_template","asset_non_current","False"
"chart_sk_111000","Obstaranie materiálu","111000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_112000","Materiál na sklade","112000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_119000","Materiál na ceste","119000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_121000","Nedokončená výroba","121000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_122000","Polotovary vlastnej výroby","122000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_123000","Výrobky","123000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_124000","Zvieratá","124000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_131000","Obstaranie tovaru","131000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_132000","Tovar na sklade a v predajniach","132000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_133000","Nehnuteľnosť na predaj","133000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_139000","Tovar na ceste","139000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_191000","Opravné položky k materiálu","191000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_192000","Opravné položky k nedokončenej výrobe","192000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_193000","Opravné položky k polotovarom vlastnej výroby","193000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_194000","Opravné položky k výrobkom","194000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_195000","Opravné položky k zvieratám","195000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_196000","Opravné položky k tovaru","196000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_211000","Pokladnica","211000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_213000","Ceniny","213000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_221000","Bankové účty","221000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_231000","Krátkodobé bankové úvery","231000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_232000","Eskontné úvery","232000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_241000","Vydané krátkodobé dlhopisy","241000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_249000","Ostatné krátkodobé finančné výpomoci","249000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_251000","Majetkové cenné papiere na obchodovanie","251000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_252000","Vlastné akcie a vlastné obchodné podiely","252000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_253000","Dlhové cenné papiere na obchodovanie","253000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_255000","Vlastné dlhopisy","255000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_256000","Dlhové cenné papiere so splatnosťou do jedného roka držané do splatnosti","256000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_257000","Ostatné realizovateľné cenné papiere","257000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_259000","Obstaranie krátkodobého finančného majetku","259000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_261000","Peniaze na ceste","261000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_291000","Opravné položky ku krátkodobému finančnému majetku","291000","l10n_sk.sk_chart_template","asset_cash","False"
"chart_sk_311000","Odberatelia","311000","l10n_sk.sk_chart_template","asset_receivable","True"
"chart_sk_312000","Zmenky na inkaso","312000","l10n_sk.sk_chart_template","asset_receivable","True"
"chart_sk_313000","Pohľadávky za eskontované cenné papiere","313000","l10n_sk.sk_chart_template","asset_receivable","True"
"chart_sk_314000","Poskytnuté preddavky","314000","l10n_sk.sk_chart_template","asset_receivable","True"
"chart_sk_315000","Ostatné pohľadávky","315000","l10n_sk.sk_chart_template","asset_receivable","True"
"chart_sk_316000","Čistá hodnota zákazky","316000","l10n_sk.sk_chart_template","asset_receivable","True"
"chart_sk_321000","Dodávatelia","321000","l10n_sk.sk_chart_template","liability_payable","True"
"chart_sk_322000","Zmenky na úhradu","322000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_323000","Krátkodobé rezervy","323000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_324000","Prijaté preddavky","324000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_325000","Ostatné záväzky","325000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_326000","Nevyfakturované dodávky","326000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_331000","Zamestnanci","331000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_333000","Ostatné záväzky voči zamestnancom","333000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_335000","Pohľadávky voči zamestnancom","335000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_336000","Zúčtovanie s orgánmi sociálneho zabezpečenia a zdravotného poistenia","336000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_341000","Daň z príjmov","341000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_342000","Ostatné priame dane","342000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_343000","Daň z pridanej hodnoty","343000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_343110","DPH nižšia sadzba vstup","343110","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_343120","DPH základná sadzba vstup","343120","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_343210","DPH nižšia sadzba výstup","343210","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_343220","DPH základná sadzba výstup","343220","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_345000","Ostatné dane a poplatky","345000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_346000","Dotácie zo štatneho rozpočtu","346000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_347000","Ostatné dodácie","347000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_351000","Pohľadávky voči prepojeným účtovným jednotkám a účtovným jednotkám v rámci podielovej účasti","351000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_353000","Pohľadávky za upísané vlastné imanie","353000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_354000","Pohľadávky voči spoločníkom a členom pri úhrade straty","354000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_355000","Ostatné pohľadávky voči spoločníkom a členom","355000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_358000","Pohľadávky voči účastníkom združenia","358000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_361000","Záväzky v rámci konsolidovaného celku","361000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_364000","Záväzky voči spoločníkom a členom pri rozdeľovaní zisku","364000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_365000","Ostatné záväzky voči spoločníkom a členom","365000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_366000","Záväzky voči spoločníkom a členom zo závislej činnosti","366000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_367000","Záväzky z upísaných nesplatených cenných papierov a vkladov","367000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_368000","Záväzky voči účastníkom združenia","368000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_371000","Pohľadávky z predaja podniku","371000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_372000","Záväzky z kúpy podniku","372000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_373000","Pohľadávky a záväzky z pevných termínových operácií","373000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_374000","Pohľadávky z nájmu","374000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_375000","Pohľadávky z vydaných dlhopisov","375000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_376000","Nakúpené opcie","376000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_377000","Predané opcie","377000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_378000","Iné pohľadávky","378000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_379000","Iné záväzky","379000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_381000","Náklady budúcich období","381000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_382000","Komplexné náklady budúcich období","382000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_383000","Výdaje budúcich období","383000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_384000","Výnosy budúcich období","384000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_385000","Príjmy budúcich období","385000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_391000","Opravná položka k pohľadávkam","391000","l10n_sk.sk_chart_template","asset_current","False"
"chart_sk_395000","Vnútorné zúčtovanie","395000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_398000","Spojovací účet pri združení","398000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_411000","Základné imanie","411000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_412000","Emisné ážio","412000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_413000","Ostatné kapitalové fondy","413000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_414000","Oceňovacie rozdiely z precenenia majetku a záväzkov","414000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_415000","Oceňovacie rozdiely z kapitálových účastín","415000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_416000","Oceňovacie rozdiely z precenenia pri zlúčení, splynutí a rozdelení","416000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_417000","Zákonný rezervný fond z kapitálových vkladov","417000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_418000","Nedeliteľný fond z kapitálových vkladov","418000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_419000","Zmeny základného imania","419000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_421000","Zakonný rezervný fond","421000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_422000","Nedeliteľný fond","422000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_423000","Štatutárne fondy","423000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_427000","Ostatné fondy","427000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_428000","Nerozdelený zisk minulých rokov","428000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_429000","Neuhradená strata minulých rokov","429000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_431000","Výsledok hospodárenia v schvaľovaní","431000","l10n_sk.sk_chart_template","equity_unaffected","False"
"chart_sk_451000","Rezervy zákonné","451000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_459000","Ostatné rezervy","459000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_461000","Bankové úvery","461000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_471000","Dlhodobé záväzky voči prepojeným účtovným jednotkám a účtovným jednotkám v rámci podielovej účasti ","471000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_472000","Záväzky zo sociálneho fondu","472000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_473000","Vydané dlhopisy","473000","l10n_sk.sk_chart_template","liability_non_current","False"
"chart_sk_474000","Záväzky z nájmu","474000","l10n_sk.sk_chart_template","liability_current","False"
"chart_sk_475000","Dlhodobé prijaté preddavky","475000","l10n_sk.sk_chart_template","liability_non_current","False"
"chart_sk_476000","Dlhodobé nevyfakturované dodávky","476000","l10n_sk.sk_chart_template","liability_non_current","False"
"chart_sk_478000","Dlhodobé zmenky na úhradu","478000","l10n_sk.sk_chart_template","liability_non_current","False"
"chart_sk_479000","Ostatné dlhodobé záväzky","479000","l10n_sk.sk_chart_template","liability_non_current","False"
"chart_sk_481000","Odložený daňový záväzok a odložená daňová pohľadávka","481000","l10n_sk.sk_chart_template","liability_non_current","False"
"chart_sk_491000","Vlastné imanie fyzickej osoby - podnikateľa","491000","l10n_sk.sk_chart_template","equity","False"
"chart_sk_501000","Spotreba materiálu","501000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_502000","Spotreba energie","502000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_503000","Spotreba ostatných neskladovateľných dodávok","503000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_504000","Predaný tovar","504000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_505000","Tvorba a zúčtovanie opravných položiek k zásobám","505000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_507000","Predaná nehnuteľnosť","507000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_511000","Opravy a udržiavanie","511000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_512000","Cestovné","512000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_513000","Náklady na reprezentáciu","513000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_518000","Ostatné služby","518000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_521000","Mzdové náklady","521000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_522000","Príjmy spoločníkov a členov zo závislej činnosti","522000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_523000","Odmeny členom orgánov spoločnosti a družstva","523000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_524000","Zákonné sociálne poistenie","524000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_525000","Ostatné sociálne zabezpečenie","525000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_526000","Sociálne náklady fyzickej osoby - podnikateľa","526000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_527000","Zákonné sociálne náklady","527000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_528000","Ostatné sociálne náklady","528000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_531000","Daň z motorových vozidiel","531000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_532000","Daň z nehnuteľností","532000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_538000","Ostatné dane a poplatky","538000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_541000","Zostatková cena predaného dlhodobého nehmotného majetku a dlhodobého hmotného majetku","541000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_542000","Predaný materiál","542000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_543000","Dary","543000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_544000","Zmluvné pokuty, penále a úroky z omeškania","544000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_545000","Ostatné pokuty, penále a úroky z omeškania","545000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_546000","Odpis pohľadávky","546000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_547000","Tvorba a zúčtovanie opravných položiek k pohľadávkam","547000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_548000","Ostatné náklady na hospodársku činnosť","548000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_549000","Manká a škody","549000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_551000","Odpisy dlhodobého mehmotného majetku a dlhodobého hmotného majetku","551000","l10n_sk.sk_chart_template","expense_depreciation","False"
"chart_sk_553000","Tvorba a zúčtovanie opravných položiek k dlhodobému majetku","553000","l10n_sk.sk_chart_template","expense_depreciation","False"
"chart_sk_555000","Zúčtovanie komplexných nákladov budúcich období","555000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_557000","Zúčtovanie oprávky k opravnej položke k nadobudnutému majetku","557000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_561000","Predané cenné papiere a podiely","561000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_562000","Úroky","562000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_563000","Kurzové straty","563000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_564000","Náklady na precenenie cenných papierov","564000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_565000","Tvorba a zúčtovanie opravných položiek k finančnému majetku","565000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_566000","Náklady na krátkodobý finančný majetok","566000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_567000","Náklady na derivátové operácie","567000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_568000","Ostatné finančné náklady","568000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_569000","Manká a škody na finančnom majetku","569000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_591000","Splatná daň z príjmov","591000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_592000","Odložená daň z príjmov","592000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_595000","Dodatočné odvody dane z príjmov","595000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_596000","Prevod podielov na výsledku hospodárenia spoločníkom","596000","l10n_sk.sk_chart_template","expense","False"
"chart_sk_601000","Tržby za vlastné výrobky","601000","l10n_sk.sk_chart_template","income","False"
"chart_sk_602000","Tržby z predaja služieb","602000","l10n_sk.sk_chart_template","income","False"
"chart_sk_604000","Tržby za tovar","604000","l10n_sk.sk_chart_template","income","False"
"chart_sk_606000","Výnosy za zákazky","606000","l10n_sk.sk_chart_template","income","False"
"chart_sk_607000","Výnosy z nehnuťelnosti na predaj","607000","l10n_sk.sk_chart_template","income","False"
"chart_sk_611000","Zmena stavu nedokončenej výroby","611000","l10n_sk.sk_chart_template","income","False"
"chart_sk_612000","Zmena stavu polotovarov","612000","l10n_sk.sk_chart_template","income","False"
"chart_sk_613000","Zmena stavu výrobkov","613000","l10n_sk.sk_chart_template","income","False"
"chart_sk_614000","Zmena stavu zvierat","614000","l10n_sk.sk_chart_template","income","False"
"chart_sk_621000","Aktivácia materiálu a tovaru","621000","l10n_sk.sk_chart_template","income","False"
"chart_sk_622000","Aktivácia vnútroorganizačných služieb","622000","l10n_sk.sk_chart_template","income","False"
"chart_sk_623000","Aktivácia dlhodobého nehmotného majetku","623000","l10n_sk.sk_chart_template","income","False"
"chart_sk_624000","Aktivácia dlhodobého hmotného majetku","624000","l10n_sk.sk_chart_template","income","False"
"chart_sk_641000","Tržby z predaja dlhodobého nehmotného majetku a dlhodobého hmotného majetku","641000","l10n_sk.sk_chart_template","income","False"
"chart_sk_642000","Tržby z predaja materiálu","642000","l10n_sk.sk_chart_template","income","False"
"chart_sk_644000","Zmluvné pokuty, penále a úroky z omeškania","644000","l10n_sk.sk_chart_template","income_other","False"
"chart_sk_645000","Ostatné pokuty, penále a úroky z omeškania","645000","l10n_sk.sk_chart_template","income_other","False"
"chart_sk_646000","Výnosy z odpísaných pohľadávok","646000","l10n_sk.sk_chart_template","income_other","False"
"chart_sk_648000","Ostatné výnosy z hospodárskej činnosti","648000","l10n_sk.sk_chart_template","income_other","False"
"chart_sk_655000","Zúčtovanie komplexných nákladov budúcich období","655000","l10n_sk.sk_chart_template","income_other","False"
"chart_sk_657000","Zúčtovanie oprávky opravnej položke k nadobudnutému majetku","657000","l10n_sk.sk_chart_template","income_other","False"
"chart_sk_661000","Tržby z predaja cenných papierov a podielov","661000","l10n_sk.sk_chart_template","income","False"
"chart_sk_662000","Úroky","662000","l10n_sk.sk_chart_template","income_other","False"
"chart_sk_663000","Kurzové zisky","663000","l10n_sk.sk_chart_template","income_other","False"
"chart_sk_664000","Výnosy z precenenia cenných papierov","664000","l10n_sk.sk_chart_template","income_other","False"
"chart_sk_665000","Výnosy z dlhodobého finančného majetku","665000","l10n_sk.sk_chart_template","income_other","False"
"chart_sk_666000","Výnosy z krátkodobého finančného majetku","666000","l10n_sk.sk_chart_template","income_other","False"
"chart_sk_667000","Výnosy z derivátových operácií","667000","l10n_sk.sk_chart_template","income_other","False"
"chart_sk_668000","Ostatné finančné výnosy","668000","l10n_sk.sk_chart_template","income_other","False"
"chart_sk_701000","Začiatočný účet súvahový","701000","l10n_sk.sk_chart_template","off_balance","False"
"chart_sk_702000","Konečný účet súvahový","702000","l10n_sk.sk_chart_template","off_balance","False"
"chart_sk_710000","Účet ziskov a strát","710000","l10n_sk.sk_chart_template","off_balance","False"
"chart_sk_711000","Začiatočný účet nákladov a výnosov","711000","l10n_sk.sk_chart_template","off_balance","False"

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_sk.sk_chart_template')]"/>
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
            <field name="name">Obchody v SK</field>
            <field name="chart_template_id" ref="sk_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_id" ref="base.sk"/>
        </record>

        <record id="fp_intra_private" model="account.fiscal.position.template">
            <field name="sequence">2</field>
            <field name="name">Obchody s EU konzument</field>
            <field name="chart_template_id" ref="sk_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>

        <record id="fiscal_position_template_2" model="account.fiscal.position.template">
            <field name="sequence">3</field>
            <field name="name">Obchody s EU</field>
            <field name="chart_template_id" ref="sk_chart_template"/>
            <field name="auto_apply" eval="True"/>
            <field name="vat_required" eval="True"/>
            <field name="country_group_id" ref="base.europe"/>
        </record>

    <!-- Fiscal Position Tax Templates -->

        <!-- European Union -->
        <!-- Sales -->

        <record id="fiscal_position_tax_template_2a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vy_tuz_20" />
            <field name="tax_dest_id" ref="vy_dod_eu" />
        </record>

        <record id="fiscal_position_tax_template_2b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vy_tuz_10" />
            <field name="tax_dest_id" ref="vy_dod_eu" />
        </record>

        <record id="fiscal_position_tax_template_2c" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vy_tuz_0" />
            <field name="tax_dest_id" ref="vy_dod_eu" />
        </record>

        <record id="fiscal_position_tax_template_2d" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vy_tuz_23" />
            <field name="tax_dest_id" ref="vy_dod_eu" />
        </record>

        <record id="fiscal_position_tax_template_2e" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vy_tuz_19" />
            <field name="tax_dest_id" ref="vy_dod_eu" />
        </record>

        <!-- European Union -->
        <!-- Purchase -->

        <record id="fiscal_position_tax_template_21a" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vs_tuz_20" />
            <field name="tax_dest_id" ref="vs_nad_eu" />
        </record>

        <record id="fiscal_position_tax_template_21b" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vs_tuz_10" />
            <field name="tax_dest_id" ref="vs_nad_eu" />
        </record>

        <record id="fiscal_position_tax_template_21c" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vs_tuz_23" />
            <field name="tax_dest_id" ref="vs_nad_eu" />
        </record>

        <record id="fiscal_position_tax_template_21d" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_2"  />
            <field name="tax_src_id" ref="vs_tuz_19" />
            <field name="tax_dest_id" ref="vs_nad_eu" />
        </record>

    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- VAT domestic sale-->
    <record id="vy_tuz_23" model="account.tax.template">
        <field name="chart_template_id" ref="sk_chart_template"/>
        <field name="name">DPH na výstupe 23%</field>
        <field name="description">23%</field>
        <field name="amount">23</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343220'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343220'),
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_vat_23"/>
    </record>
    <record id="vy_tuz_20" model="account.tax.template">
        <field name="chart_template_id" ref="sk_chart_template"/>
        <field name="name">DPH na výstupe 20%</field>
        <field name="description">20%</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343220'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343220'),
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
    </record>
    <record id="vy_tuz_19" model="account.tax.template">
        <field name="chart_template_id" ref="sk_chart_template"/>
        <field name="name">DPH na výstupe 19%</field>
        <field name="description">19%</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343210'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343210'),
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_vat_19"/>
    </record>
    <record id="vy_tuz_10" model="account.tax.template">
        <field name="chart_template_id" ref="sk_chart_template"/>
        <field name="name">DPH na výstupe 10%</field>
        <field name="description">10%</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343210'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343210'),
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_vat_10"/>
    </record>
    <record id="vy_tuz_0" model="account.tax.template">
        <field name="chart_template_id" ref="sk_chart_template"/>
        <field name="name">DPH na výstupe 0%</field>
        <field name="description">0%</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
    </record>
    <!-- VAT domestic purchase -->
    <record id="vs_tuz_23" model="account.tax.template">
        <field name="chart_template_id" ref="sk_chart_template"/>
        <field name="name">DPH na vstupe 23%</field>
        <field name="description">23%</field>
        <field name="amount">23</field>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="0"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_23"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343120'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343120'),
            }),
        ]"/>
    </record>
    <record id="vs_tuz_20" model="account.tax.template">
        <field name="chart_template_id" ref="sk_chart_template"/>
        <field name="name">DPH na vstupe 20%</field>
        <field name="description">20%</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="0"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343120'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343120'),
            }),
        ]"/>
    </record>
    <record id="vs_tuz_19" model="account.tax.template">
        <field name="chart_template_id" ref="sk_chart_template"/>
        <field name="name">DPH na vstupe 19%</field>
        <field name="description">19%</field>
        <field name="amount">19</field>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="0"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_19"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343110'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343110'),
            }),
        ]"/>
    </record>
    <record id="vs_tuz_10" model="account.tax.template">
        <field name="chart_template_id" ref="sk_chart_template"/>
        <field name="name">DPH na vstupe 10%</field>
        <field name="description">10%</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="sequence" eval="0"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343110'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343110'),
            }),
        ]"/>
    </record>
    <!-- Eurpean Union -->
    <!-- =========================================================== -->
    <record id="vy_dod_eu" model="account.tax.template">
        <field name="chart_template_id" ref="sk_chart_template"/>
        <field name="name">Dodanie do EU</field>
        <field name="description">0%</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
                  ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="tax_group_id" ref="tax_group_vat_0"/>
    </record>
    <record id="vs_nad_eu" model="account.tax.template">
        <field name="chart_template_id" ref="sk_chart_template"/>
        <field name="name">Nadobudnutie z EU</field>
        <field name="description">20%</field>
        <field name="amount">20.0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343120'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343220'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343120'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343220'),
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
    </record>
    <record id="vs_nad_eu_23" model="account.tax.template">
        <field name="chart_template_id" ref="sk_chart_template"/>
        <field name="name">Nadobudnutie z EU</field>
        <field name="description">23%</field>
        <field name="amount">23.0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343120'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343220'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343120'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343220'),
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_vat_23"/>
    </record>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_vat_23" model="account.tax.group">
            <field name="name">DPH 23%</field>
            <field name="country_id" ref="base.sk"/>
        </record>
        <record id="tax_group_vat_20" model="account.tax.group">
            <field name="name">DPH 20%</field>
            <field name="country_id" ref="base.sk"/>
        </record>
        <record id="tax_group_vat_19" model="account.tax.group">
            <field name="name">DPH 19%</field>
            <field name="country_id" ref="base.sk"/>
        </record>
        <record id="tax_group_vat_10" model="account.tax.group">
            <field name="name">DPH 10%</field>
            <field name="country_id" ref="base.sk"/>
        </record>
        <record id="tax_group_vat_0" model="account.tax.group">
            <field name="name">DPH 0%</field>
            <field name="country_id" ref="base.sk"/>
        </record>
    </data>
</odoo>

```

## File: data\demo_company.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="partner_demo_company_sk" model="res.partner">
        <field name="name">SK Company</field>
        <field name="vat">SK2022749619</field>
        <field name="street">Pařížská Street 25/31</field>
        <field name="city">Bratislava</field>
        <field name="country_id" ref="base.sk"/>
        <field name="zip"></field>
        <field name="phone">+421 5 12 34 56 78</field>
        <field name="email">info@company.skexample.com</field>
        <field name="website">www.skexample.com</field>
    </record>

    <record id="demo_company_sk" model="res.company">
        <field name="name">SK Company</field>
        <field name="partner_id" ref="partner_demo_company_sk"/>
    </record>

    <function model="res.company" name="_onchange_country_id">
        <value eval="[ref('demo_company_sk')]"/>
    </function>

    <function model="res.users" name="write">
        <value eval="[ref('base.user_root'), ref('base.user_admin'), ref('base.user_demo')]"/>
        <value eval="{'company_ids': [(4, ref('l10n_sk.demo_company_sk'))]}"/>
    </function>

    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_sk.sk_chart_template')]"/>
        <value model="res.company" eval="obj().env.ref('l10n_sk.demo_company_sk')"/>
    </function>
</odoo>

```

## File: data\l10n_sk_coa_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Account Tax Group -->
        <record id="sk_chart_template" model="account.chart.template">
            <field name="name">Slovenská účtová osnova</field>
            <field name="code_digits">6</field>
            <field name="bank_account_code_prefix">221</field>
            <field name="cash_account_code_prefix">211</field>
            <field name="transfer_account_code_prefix">261</field>
            <field name="currency_id" ref="base.EUR"/>
            <field name="country_id" ref="base.sk"/>
            <field name="use_storno_accounting" eval="True"/>
        </record>
    </data>
</odoo>

```

## File: data\l10n_sk_coa_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="sk_chart_template" model="account.chart.template">
        <field name="account_journal_suspense_account_id" ref="chart_sk_261000"/>
        <field name="property_account_receivable_id" ref="chart_sk_311000"/>
        <field name="property_account_payable_id" ref="chart_sk_321000"/>
        <field name="property_account_expense_categ_id" ref="chart_sk_504000"/>
        <field name="property_account_income_categ_id" ref="chart_sk_604000"/>
        <field name="property_account_expense_id" ref="chart_sk_504000"/>
        <field name="property_account_income_id" ref="chart_sk_604000"/>
        <field name="income_currency_exchange_account_id" ref="chart_sk_663000"/>
        <field name="expense_currency_exchange_account_id" ref="chart_sk_563000"/>
        <field name="default_cash_difference_income_account_id" ref="chart_sk_668000"/>
        <field name="default_cash_difference_expense_account_id" ref="chart_sk_568000"/>
        <field name="property_stock_account_input_categ_id" ref="chart_sk_131000"/>
        <field name="property_stock_account_output_categ_id" ref="chart_sk_504000"/>
        <field name="property_stock_valuation_account_id" ref="chart_sk_132000"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="chart_sk_546000"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="chart_sk_646000"/>
    </record>
</odoo>

```

## File: migrations\1.1\post-migrate_update_taxes.py

```python
from odoo.addons.account.models.chart_template import update_taxes_from_templates


def migrate(cr, version):
    update_taxes_from_templates(cr, 'l10n_sk.sk_chart_template')

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106"><defs><mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse"><path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill:#fff;fill-rule:evenodd"/></mask><mask id="b" x="6.24" y="7.21" width="48.52" height="32.34" maskUnits="userSpaceOnUse"><rect x="6.29" y="7.65" width="48.45" height="31.57" rx="1" style="fill:#fff"/></mask><symbol id="c" viewBox="0 0 106 106"><g style="mask:url(#a)"><path d="M0,0H106V106H0Z" style="fill:#5a5a64;fill-rule:evenodd"/><path d="M6.06,1.51H98.43q6.06,0,7.57,3V0H0V4.54Q1.52,1.51,6.06,1.51Z" style="fill:#fff;fill-opacity:0.382999986410141;fill-rule:evenodd"/><path d="M6.06,104.49H98.43q6.06,0,7.57-4.55V106H0V99.94Q1.52,104.49,6.06,104.49Z" style="fill-opacity:0.382999986410141;fill-rule:evenodd"/><path d="M70.38,104.49H6.06C3,104.49,0,103,0,98.43V61.28L28.77,19.69H59.06a77.33,77.33,0,0,0,21.2,13.87c.07,11.31.07,4.86,0,16.17h3.12l.21,36.82Z" style="fill:#393939;fill-rule:evenodd;opacity:0.324000000953674;isolation:isolate"/><g style="opacity:0.30000000000000004"><path d="M68.77,58.54H76c.76,0,1,.12,1,.46v2.45c0,.31-.24.43-.93.43H61.44c-.66,0-.92-.12-.92-.42,0-.83,0-1.67,0-2.51,0-.29.26-.4.92-.41Z"/><path d="M64.33,77.42c.42.39.76.66,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,4.25,4.25,0,0,1-.48-.47c-.14-.15-.26-.31-.49-.6-.32.37-.54.66-.79.91-.53.53-1.08.58-1.5.15s-.36-.94.15-1.45c.26-.26.54-.5.91-.83-.38-.34-.72-.61-1-.91a.9.9,0,0,1,0-1.36.91.91,0,0,1,1.36,0c.29.28.54.6.93,1A12.1,12.1,0,0,1,64,75.18a.91.91,0,0,1,1.36,0,.87.87,0,0,1,0,1.31C65.07,76.79,64.73,77.06,64.33,77.42Z"/><path d="M62.13,66.9c0-.47,0-.88,0-1.28a.92.92,0,0,1,.92-1,.91.91,0,0,1,1,1c0,.41,0,.81,0,1.3h1.14a1.16,1.16,0,0,1,1.22,1c0,.55-.42.85-1.18.86H64.12c0,.49,0,.91,0,1.34a.94.94,0,1,1-1.88,0c0-.41,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88C61.3,66.89,61.68,66.9,62.13,66.9Z"/><path d="M74.31,76H72.23c-.67,0-1-.34-1-.93a.89.89,0,0,1,1-1q2.18,0,4.35,0a1,1,0,1,1,0,1.91c-.74,0-1.47,0-2.21,0Z"/><path d="M74.28,68.61c-.71,0-1.43,0-2.14,0a.86.86,0,0,1-1-.9.85.85,0,0,1,.92-1c1.5,0,3,0,4.48,0a.93.93,0,0,1,1,1,.91.91,0,0,1-1,.91c-.75,0-1.51,0-2.27,0Z"/><path d="M74.36,78.09c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.57-.38.93-1,.94H72.28c-.75,0-1.09-.32-1.09-.94s.37-1,1.09-1,1.39,0,2.08,0Z"/><path d="M81.29,90.55H56.14a4,4,0,0,1-4-4V53.73a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V86.55A4,4,0,0,1,81.29,90.55ZM56.14,53.73V86.55H81.29V53.73Z"/><path d="M43.49,83.26H31.8V25.71H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V34.8c-4.55-3-16.66-12.11-19.69-13.63H30.29a2.68,2.68,0,0,0-3,3V84.77a2.68,2.68,0,0,0,3,3H48.45V83.26ZM60.57,25.71l15.14,10.6H60.57Z"/></g><path d="M60.57,18.68H30.29a2.68,2.68,0,0,0-3,3V82.28a2.68,2.68,0,0,0,3,3H48.45V80.77H31.8V23.22H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V32.31C75.71,29.28,63.6,20.2,60.57,18.68Zm0,15.14V23.22l15.14,10.6Z" style="fill:#a8a9ab"/><path d="M68.77,55.78H76c.76,0,1,.13,1,.53v2.85c0,.37-.24.5-.93.5q-7.3,0-14.61,0c-.66,0-.92-.14-.92-.48,0-1,0-2,0-2.93,0-.34.26-.47.92-.47Z" style="fill:#a8a9ab"/><path d="M64.33,76.53c.42.38.76.65,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,5.44,5.44,0,0,1-.48-.48c-.14-.14-.26-.31-.49-.59-.32.36-.54.65-.79.91-.53.53-1.08.57-1.5.14s-.36-.94.15-1.45c.26-.26.54-.49.91-.82-.38-.35-.72-.61-1-.92a.9.9,0,0,1,0-1.36.92.92,0,0,1,1.36,0c.29.28.54.61.93,1A13.78,13.78,0,0,1,64,74.28a.91.91,0,0,1,1.36,0,.88.88,0,0,1,0,1.32C65.07,75.89,64.73,76.16,64.33,76.53Z" style="fill:#a8a9ab"/><path d="M62.13,65.88c0-.48,0-.88,0-1.29a1,1,0,1,1,1.91,0c0,.4,0,.81,0,1.3h1.14a1.15,1.15,0,0,1,1.22,1c0,.54-.42.85-1.18.85H64.12c0,.49,0,.92,0,1.34a.94.94,0,1,1-1.88,0c0-.4,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88Z" style="fill:#a8a9ab"/><path d="M74.31,75.11c-.69,0-1.38,0-2.08,0s-1-.35-1-.94a.89.89,0,0,1,1-1q2.18,0,4.35,0a.91.91,0,0,1,1,1,.93.93,0,0,1-1,1c-.74,0-1.47,0-2.21,0Z" style="fill:#a8a9ab"/><path d="M74.28,67.76H72.14a.87.87,0,0,1-1-.9.84.84,0,0,1,.92-1c1.5,0,3,0,4.48,0a.94.94,0,0,1,1,1,.91.91,0,0,1-1,.91H74.28Z" style="fill:#a8a9ab"/><path d="M74.36,77.2c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.56-.38.93-1,.93q-2.12,0-4.23,0c-.75,0-1.09-.32-1.09-.94s.37-.94,1.09-1,1.39,0,2.08,0Z" style="fill:#a8a9ab"/><path d="M81.29,88.06H56.14a4,4,0,0,1-4-4V51.24a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V84.06A4,4,0,0,1,81.29,88.06ZM56.14,51.24V84.06H81.29V51.24Z" style="fill:#a8a9ab"/></g></symbol></defs><use width="106" height="106" transform="translate(-0.07 0)" xlink:href="#c"/><rect x="6.2" y="10.57" width="48.45" height="31.57" rx="1" style="fill:#393939;opacity:0.44;isolation:isolate"/><g style="mask:url(#b)"><rect x="6.24" y="7.21" width="48.52" height="32.34" style="fill:#ee1c25"/><rect x="6.24" y="7.21" width="48.52" height="21.56" style="fill:#0b4ea2"/><rect x="6.24" y="7.21" width="48.52" height="10.78" style="fill:#fff"/><path d="M27.45,14.81H14.13l0,.43c0,.1-.25,2.37-.25,7.37a9,9,0,0,0,2.36,6.16,13.8,13.8,0,0,0,4.38,3.13l.21.1.21-.1a13.8,13.8,0,0,0,4.38-3.13,8.93,8.93,0,0,0,2.36-6.16c0-5-.23-7.27-.24-7.37l-.05-.43Z" style="fill:#fff"/><path d="M20.79,31.47c-2.66-1.29-6.46-3.83-6.46-8.86s.24-7.32.24-7.32H27s.24,2.29.24,7.32-3.81,7.57-6.47,8.86Z" style="fill:#ee1c25"/><path d="M21.36,21.29a10.38,10.38,0,0,0,3.38-.38s0,.45,0,1,0,1,0,1a11,11,0,0,0-3.38-.38v2.78H20.22V22.47a10.93,10.93,0,0,0-3.37.38s0-.44,0-1,0-1,0-1a10.32,10.32,0,0,0,3.37.38V19.55a7.82,7.82,0,0,0-2.67.38s0-.45,0-1,0-1,0-1a8.1,8.1,0,0,0,2.67.39,15.29,15.29,0,0,0-.35-2.5s.66,0,.92,0,.93,0,.93,0a16,16,0,0,0-.36,2.5A8.06,8.06,0,0,0,24,18s0,.45,0,1,0,1,0,1a7.79,7.79,0,0,0-2.67-.38v1.74Z" style="fill:#fff"/><path d="M20.79,25c-1.34,0-2.06,1.86-2.06,1.86a1.63,1.63,0,0,0-1.49-.88,2.1,2.1,0,0,0-1.63,1.26,12.38,12.38,0,0,0,5.18,4.28A12.32,12.32,0,0,0,26,27.19a2.08,2.08,0,0,0-1.63-1.26,1.64,1.64,0,0,0-1.49.88S22.13,25,20.79,25Z" style="fill:#0b4ea2"/></g></svg>
```

