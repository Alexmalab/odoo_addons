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
    'version': '1.0',
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
"chart_sk_012000","Aktivované náklady na vývoj","012000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_013000","Softvér","013000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_014000","Oceniteľné práva","014000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_015000","Goodwill","015000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_019000","Ostatný dlhodobý nehmotný majetok","019000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_021000","Stavby","021000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_022000","Samostatné hnuteľné veci a súbory hnuteľných vecí","022000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_025000","Pestovateľské celky trvalých porastov","025000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_026000","Základné stádo a ťažné zvieratá","026000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_029000","Ostatný dlhodobý hmotný majetok","029000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_031000","Pozemky","031000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_032000","Umelecké diela a zbierky","032000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_041000","Obstaranie dlhodobého nehmotného majetku","041000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_042000","Obstaranie dlhodobého hmotného majetku","042000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_043000","Obstaranie dlhodobého finančného majetku","043000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_051000","Poskytnuté preddavky na dlhodobý nehmotný majetok","051000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_052000","Poskytnuté preddavky na dlhodobý hmotný majetok","052000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_055000","Poskytnuté preddavky na dlhodobý finančný majetok","055000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_061000","Podielové cenné papiere a podiely v dcérskej účtovnej jednotke","061000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_062000","Podielové cenné papiere a podiely v spoločnosti alebo družstve s podielovou účasťou","062000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_063000","Realizovateľné cenné papiere a podiely","063000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_065000","Dlhové cenné papiere držané do splatnosti","065000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_066000","Pôžičky prepojeným účtovným jednotkám a účtovným jednotkám v rámci podielovej účasti","066000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_067000","Ostatné pôžičky","067000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_069000","Ostatný dlhodobý finančný majetok","069000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_072000","Oprávky k aktivovaným nákladom na vývoj","072000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_073000","Oprávky k softvéru","073000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_074000","Oprávky k oceniteľným právam","074000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_075000","Oprávky ku goodwilu","075000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_079000","Opravky k ostatnému dlhodobému nehmotnému majetku","079000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_081000","Oprávky k stavbám","081000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_082000","Oprávky k samostatným hnuteľným veciam a k súboru hnuteľných vecí","082000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_085000","Oprávky k pestovateľským celkom trvalých porastov","085000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_086000","Oprávky k základnému stádu a ťažným zvieratám","086000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_089000","Oprávky k ostatnému dlhodobému hmotnému majetku","089000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_091000","Opravné položky k dlhodobému nehmotnému majetku","091000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_092000","Opravné položky k dlhodobému hmotnému majetku","092000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_093000","Opravné položky k nedokončenému dlhodobému nehmotnému majetku","093000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_094000","Opravné položky k nedokončenému dlhodobému hmotnému majetku","094000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_095000","Opravné položky k poskytnutým preddavkom na dlhodobý majetok","095000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_096000","Opravné položky k dlhodobému finančnému majetku","096000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_097000","Opravné položky k nadobudnutému majetku","097000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_098000","Oprávky k opravnej položke k nadobudnutému majetku","098000","l10n_sk.sk_chart_template","account.data_account_type_non_current_assets","False"
"chart_sk_111000","Obstaranie materiálu","111000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_112000","Materiál na sklade","112000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_119000","Materiál na ceste","119000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_121000","Nedokončená výroba","121000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_122000","Polotovary vlastnej výroby","122000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_123000","Výrobky","123000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_124000","Zvieratá","124000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_131000","Obstaranie tovaru","131000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_132000","Tovar na sklade a v predajniach","132000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_133000","Nehnuteľnosť na predaj","133000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_139000","Tovar na ceste","139000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_191000","Opravné položky k materiálu","191000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_192000","Opravné položky k nedokončenej výrobe","192000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_193000","Opravné položky k polotovarom vlastnej výroby","193000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_194000","Opravné položky k výrobkom","194000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_195000","Opravné položky k zvieratám","195000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_196000","Opravné položky k tovaru","196000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_211000","Pokladnica","211000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_213000","Ceniny","213000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_221000","Bankové účty","221000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_231000","Krátkodobé bankové úvery","231000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_232000","Eskontné úvery","232000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_241000","Vydané krátkodobé dlhopisy","241000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_249000","Ostatné krátkodobé finančné výpomoci","249000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_251000","Majetkové cenné papiere na obchodovanie","251000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_252000","Vlastné akcie a vlastné obchodné podiely","252000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_253000","Dlhové cenné papiere na obchodovanie","253000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_255000","Vlastné dlhopisy","255000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_256000","Dlhové cenné papiere so splatnosťou do jedného roka držané do splatnosti","256000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_257000","Ostatné realizovateľné cenné papiere","257000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_259000","Obstaranie krátkodobého finančného majetku","259000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_261000","Peniaze na ceste","261000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_291000","Opravné položky ku krátkodobému finančnému majetku","291000","l10n_sk.sk_chart_template","account.data_account_type_liquidity","False"
"chart_sk_311000","Odberatelia","311000","l10n_sk.sk_chart_template","account.data_account_type_receivable","True"
"chart_sk_312000","Zmenky na inkaso","312000","l10n_sk.sk_chart_template","account.data_account_type_receivable","True"
"chart_sk_313000","Pohľadávky za eskontované cenné papiere","313000","l10n_sk.sk_chart_template","account.data_account_type_receivable","True"
"chart_sk_314000","Poskytnuté preddavky","314000","l10n_sk.sk_chart_template","account.data_account_type_receivable","True"
"chart_sk_315000","Ostatné pohľadávky","315000","l10n_sk.sk_chart_template","account.data_account_type_receivable","True"
"chart_sk_316000","Čistá hodnota zákazky","316000","l10n_sk.sk_chart_template","account.data_account_type_receivable","True"
"chart_sk_321000","Dodávatelia","321000","l10n_sk.sk_chart_template","account.data_account_type_payable","True"
"chart_sk_322000","Zmenky na úhradu","322000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_323000","Krátkodobé rezervy","323000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_324000","Prijaté preddavky","324000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_325000","Ostatné záväzky","325000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_326000","Nevyfakturované dodávky","326000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_331000","Zamestnanci","331000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_333000","Ostatné záväzky voči zamestnancom","333000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_335000","Pohľadávky voči zamestnancom","335000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_336000","Zúčtovanie s orgánmi sociálneho zabezpečenia a zdravotného poistenia","336000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_341000","Daň z príjmov","341000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_342000","Ostatné priame dane","342000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_343000","Daň z pridanej hodnoty","343000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_343110","DPH nižšia sadzba vstup","343110","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_343120","DPH základná sadzba vstup","343120","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_343210","DPH nižšia sadzba výstup","343210","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_343220","DPH základná sadzba výstup","343220","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_345000","Ostatné dane a poplatky","345000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_346000","Dotácie zo štatneho rozpočtu","346000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_347000","Ostatné dodácie","347000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_351000","Pohľadávky voči prepojeným účtovným jednotkám a účtovným jednotkám v rámci podielovej účasti","351000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_353000","Pohľadávky za upísané vlastné imanie","353000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_354000","Pohľadávky voči spoločníkom a členom pri úhrade straty","354000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_355000","Ostatné pohľadávky voči spoločníkom a členom","355000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_358000","Pohľadávky voči účastníkom združenia","358000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_361000","Záväzky v rámci konsolidovaného celku","361000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_364000","Záväzky voči spoločníkom a členom pri rozdeľovaní zisku","364000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_365000","Ostatné záväzky voči spoločníkom a členom","365000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_366000","Záväzky voči spoločníkom a členom zo závislej činnosti","366000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_367000","Záväzky z upísaných nesplatených cenných papierov a vkladov","367000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_368000","Záväzky voči účastníkom združenia","368000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_371000","Pohľadávky z predaja podniku","371000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_372000","Záväzky z kúpy podniku","372000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_373000","Pohľadávky a záväzky z pevných termínových operácií","373000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_374000","Pohľadávky z nájmu","374000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_375000","Pohľadávky z vydaných dlhopisov","375000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_376000","Nakúpené opcie","376000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_377000","Predané opcie","377000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_378000","Iné pohľadávky","378000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_379000","Iné záväzky","379000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_381000","Náklady budúcich období","381000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_382000","Komplexné náklady budúcich období","382000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_383000","Výdaje budúcich období","383000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_384000","Výnosy budúcich období","384000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_385000","Príjmy budúcich období","385000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_391000","Opravná položka k pohľadávkam","391000","l10n_sk.sk_chart_template","account.data_account_type_current_assets","False"
"chart_sk_395000","Vnútorné zúčtovanie","395000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_398000","Spojovací účet pri združení","398000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_411000","Základné imanie","411000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_412000","Emisné ážio","412000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_413000","Ostatné kapitalové fondy","413000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_414000","Oceňovacie rozdiely z precenenia majetku a záväzkov","414000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_415000","Oceňovacie rozdiely z kapitálových účastín","415000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_416000","Oceňovacie rozdiely z precenenia pri zlúčení, splynutí a rozdelení","416000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_417000","Zákonný rezervný fond z kapitálových vkladov","417000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_418000","Nedeliteľný fond z kapitálových vkladov","418000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_419000","Zmeny základného imania","419000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_421000","Zakonný rezervný fond","421000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_422000","Nedeliteľný fond","422000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_423000","Štatutárne fondy","423000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_427000","Ostatné fondy","427000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_428000","Nerozdelený zisk minulých rokov","428000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_429000","Neuhradená strata minulých rokov","429000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_431000","Výsledok hospodárenia v schvaľovaní","431000","l10n_sk.sk_chart_template","account.data_unaffected_earnings","False"
"chart_sk_451000","Rezervy zákonné","451000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_459000","Ostatné rezervy","459000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_461000","Bankové úvery","461000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_471000","Dlhodobé záväzky voči prepojeným účtovným jednotkám a účtovným jednotkám v rámci podielovej účasti ","471000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_472000","Záväzky zo sociálneho fondu","472000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_473000","Vydané dlhopisy","473000","l10n_sk.sk_chart_template","account.data_account_type_non_current_liabilities","False"
"chart_sk_474000","Záväzky z nájmu","474000","l10n_sk.sk_chart_template","account.data_account_type_current_liabilities","False"
"chart_sk_475000","Dlhodobé prijaté preddavky","475000","l10n_sk.sk_chart_template","account.data_account_type_non_current_liabilities","False"
"chart_sk_476000","Dlhodobé nevyfakturované dodávky","476000","l10n_sk.sk_chart_template","account.data_account_type_non_current_liabilities","False"
"chart_sk_478000","Dlhodobé zmenky na úhradu","478000","l10n_sk.sk_chart_template","account.data_account_type_non_current_liabilities","False"
"chart_sk_479000","Ostatné dlhodobé záväzky","479000","l10n_sk.sk_chart_template","account.data_account_type_non_current_liabilities","False"
"chart_sk_481000","Odložený daňový záväzok a odložená daňová pohľadávka","481000","l10n_sk.sk_chart_template","account.data_account_type_non_current_liabilities","False"
"chart_sk_491000","Vlastné imanie fyzickej osoby - podnikateľa","491000","l10n_sk.sk_chart_template","account.data_account_type_equity","False"
"chart_sk_501000","Spotreba materiálu","501000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_502000","Spotreba energie","502000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_503000","Spotreba ostatných neskladovateľných dodávok","503000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_504000","Predaný tovar","504000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_505000","Tvorba a zúčtovanie opravných položiek k zásobám","505000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_507000","Predaná nehnuteľnosť","507000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_511000","Opravy a udržiavanie","511000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_512000","Cestovné","512000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_513000","Náklady na reprezentáciu","513000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_518000","Ostatné služby","518000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_521000","Mzdové náklady","521000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_522000","Príjmy spoločníkov a členov zo závislej činnosti","522000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_523000","Odmeny členom orgánov spoločnosti a družstva","523000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_524000","Zákonné sociálne poistenie","524000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_525000","Ostatné sociálne zabezpečenie","525000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_526000","Sociálne náklady fyzickej osoby - podnikateľa","526000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_527000","Zákonné sociálne náklady","527000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_528000","Ostatné sociálne náklady","528000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_531000","Daň z motorových vozidiel","531000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_532000","Daň z nehnuteľností","532000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_538000","Ostatné dane a poplatky","538000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_541000","Zostatková cena predaného dlhodobého nehmotného majetku a dlhodobého hmotného majetku","541000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_542000","Predaný materiál","542000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_543000","Dary","543000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_544000","Zmluvné pokuty, penále a úroky z omeškania","544000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_545000","Ostatné pokuty, penále a úroky z omeškania","545000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_546000","Odpis pohľadávky","546000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_547000","Tvorba a zúčtovanie opravných položiek k pohľadávkam","547000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_548000","Ostatné náklady na hospodársku činnosť","548000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_549000","Manká a škody","549000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_551000","Odpisy dlhodobého mehmotného majetku a dlhodobého hmotného majetku","551000","l10n_sk.sk_chart_template","account.data_account_type_depreciation","False"
"chart_sk_553000","Tvorba a zúčtovanie opravných položiek k dlhodobému majetku","553000","l10n_sk.sk_chart_template","account.data_account_type_depreciation","False"
"chart_sk_555000","Zúčtovanie komplexných nákladov budúcich období","555000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_557000","Zúčtovanie oprávky k opravnej položke k nadobudnutému majetku","557000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_561000","Predané cenné papiere a podiely","561000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_562000","Úroky","562000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_563000","Kurzové straty","563000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_564000","Náklady na precenenie cenných papierov","564000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_565000","Tvorba a zúčtovanie opravných položiek k finančnému majetku","565000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_566000","Náklady na krátkodobý finančný majetok","566000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_567000","Náklady na derivátové operácie","567000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_568000","Ostatné finančné náklady","568000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_569000","Manká a škody na finančnom majetku","569000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_591000","Splatná daň z príjmov","591000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_592000","Odložená daň z príjmov","592000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_595000","Dodatočné odvody dane z príjmov","595000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_596000","Prevod podielov na výsledku hospodárenia spoločníkom","596000","l10n_sk.sk_chart_template","account.data_account_type_expenses","False"
"chart_sk_601000","Tržby za vlastné výrobky","601000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_602000","Tržby z predaja služieb","602000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_604000","Tržby za tovar","604000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_606000","Výnosy za zákazky","606000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_607000","Výnosy z nehnuťelnosti na predaj","607000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_611000","Zmena stavu nedokončenej výroby","611000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_612000","Zmena stavu polotovarov","612000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_613000","Zmena stavu výrobkov","613000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_614000","Zmena stavu zvierat","614000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_621000","Aktivácia materiálu a tovaru","621000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_622000","Aktivácia vnútroorganizačných služieb","622000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_623000","Aktivácia dlhodobého nehmotného majetku","623000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_624000","Aktivácia dlhodobého hmotného majetku","624000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_641000","Tržby z predaja dlhodobého nehmotného majetku a dlhodobého hmotného majetku","641000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_642000","Tržby z predaja materiálu","642000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_644000","Zmluvné pokuty, penále a úroky z omeškania","644000","l10n_sk.sk_chart_template","account.data_account_type_other_income","False"
"chart_sk_645000","Ostatné pokuty, penále a úroky z omeškania","645000","l10n_sk.sk_chart_template","account.data_account_type_other_income","False"
"chart_sk_646000","Výnosy z odpísaných pohľadávok","646000","l10n_sk.sk_chart_template","account.data_account_type_other_income","False"
"chart_sk_648000","Ostatné výnosy z hospodárskej činnosti","648000","l10n_sk.sk_chart_template","account.data_account_type_other_income","False"
"chart_sk_655000","Zúčtovanie komplexných nákladov budúcich období","655000","l10n_sk.sk_chart_template","account.data_account_type_other_income","False"
"chart_sk_657000","Zúčtovanie oprávky opravnej položke k nadobudnutému majetku","657000","l10n_sk.sk_chart_template","account.data_account_type_other_income","False"
"chart_sk_661000","Tržby z predaja cenných papierov a podielov","661000","l10n_sk.sk_chart_template","account.data_account_type_revenue","False"
"chart_sk_662000","Úroky","662000","l10n_sk.sk_chart_template","account.data_account_type_other_income","False"
"chart_sk_663000","Kurzové zisky","663000","l10n_sk.sk_chart_template","account.data_account_type_other_income","False"
"chart_sk_664000","Výnosy z precenenia cenných papierov","664000","l10n_sk.sk_chart_template","account.data_account_type_other_income","False"
"chart_sk_665000","Výnosy z dlhodobého finančného majetku","665000","l10n_sk.sk_chart_template","account.data_account_type_other_income","False"
"chart_sk_666000","Výnosy z krátkodobého finančného majetku","666000","l10n_sk.sk_chart_template","account.data_account_type_other_income","False"
"chart_sk_667000","Výnosy z derivátových operácií","667000","l10n_sk.sk_chart_template","account.data_account_type_other_income","False"
"chart_sk_668000","Ostatné finančné výnosy","668000","l10n_sk.sk_chart_template","account.data_account_type_other_income","False"
"chart_sk_701000","Začiatočný účet súvahový","701000","l10n_sk.sk_chart_template","account.data_account_off_sheet","False"
"chart_sk_702000","Konečný účet súvahový","702000","l10n_sk.sk_chart_template","account.data_account_off_sheet","False"
"chart_sk_710000","Účet ziskov a strát","710000","l10n_sk.sk_chart_template","account.data_account_off_sheet","False"
"chart_sk_711000","Začiatočný účet nákladov a výnosov","711000","l10n_sk.sk_chart_template","account.data_account_off_sheet","False"

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

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Account Tax Group -->
        <record id="tax_group_vat_20" model="account.tax.group">
            <field name="name">DPH 20%</field>
        </record>
        <record id="tax_group_vat_10" model="account.tax.group">
            <field name="name">DPH 10%</field>
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

    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <!-- VAT domestic sale-->

    <record id="vy_tuz_20" model="account.tax.template">
        <field name="chart_template_id" ref="sk_chart_template"/>
        <field name="name">DPH na výstupe 20%</field>
        <field name="description">20%</field>
        <field name="amount">20</field>
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
                'account_id': ref('chart_sk_343220'),
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
                'account_id': ref('chart_sk_343220'),
            }),
        ]"/>
        <field name="tax_group_id" ref="tax_group_vat_20"/>
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343210'),
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343120'),
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
                'account_id': ref('chart_sk_343120'),
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
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343110'),
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

    <record id="vs_nad_eu" model="account.tax.template">
        <field name="chart_template_id" ref="sk_chart_template"/>
        <field name="name">Nadobudnutie z EU</field>
        <field name="description">20%</field>
        <field name="amount">20.0</field>
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
                'account_id': ref('chart_sk_343120'),
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('chart_sk_343220'),
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
    </record>
</odoo>

```

