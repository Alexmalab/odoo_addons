# Odoo Module: l10n_hu

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2014 InnOpen Group Kft (<http://www.innopen.eu>).

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2014 InnOpen Group Kft (<http://www.innopen.eu>).
# Copyright (C) 2021 Odoo S.A.

{
    'name': 'Hungarian - Accounting',
    'version': '3.0',
    'category': 'Accounting/Localizations/Account Charts',
    'author': 'Odoo S.A.',
    'description': """

Base module for Hungarian localization
==========================================

This module consists of:

 - Generic Hungarian chart of accounts
 - Hungarian taxes
 - Hungarian Bank information
 """,
    'depends': [
        'account'
    ],
    'data': [
        'data/l10n_hu_chart_data.xml',
        'data/account.account.template.csv',
        'data/account.group.template.csv',
        'data/account.tax.group.csv',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/account.fiscal.position.template.csv',
        'data/account.fiscal.position.tax.template.csv',
        'data/res.bank.csv',
        'data/account_chart_template_data.xml',
        'data/account_chart_template_configure_data.xml',
        'data/menuitem_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","code","name","user_type_id/id","reconcile",chart_template_id/id
"l10n_hu_111",111,"Alapítás-átszervezés aktívált értéke","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_112",112,"Kísérleti fejlesztés aktívált értéke","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_113",113,"Vagyoni értékű jogok","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_114",114,"Szellemi termékek","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_115",115,"Üzleti vagy cégérték","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_117",117,"Immateriális javak értékhelyesbítése","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_118",118,"Immateriális javak terven felüli értékcsökkenése és annak visszaírása","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_119",119,"Immateriális javak terv szerinti értékcsökkenése","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_121",121,"Földterület","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_122",122,"Telek, telkesítés","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_123",123,"Épületek,tulajdoni hányadok","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_124",124,"Egyéb építmények","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_125",125,"Üzemkörön kivüli ingatlanok, épületek","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_126",126,"Ingatlanhoz kapcs. vagyoni ért. jogok","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_127",127,"Ingatlanok értékhelyesbítése","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_128",128,"Ingatlanok terven felüli értékcsökkenése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_129",129,"Ingatlanok terv szerinti értékcsökkenése","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_131",131,"Termelő gépek, berendezések, szerszámok, gyártóeszközök","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_132",132,"Termelésben közvetlenül résztvevő járművek","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_137",137,"Műszaki berendezések, gépek, járművek értékhelyesbítése","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_138",138,"Műszaki berendezések, gépek, járművek terven felüli értékcsökkenése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_139",139,"Műszaki berendezések, gépek, járművek terv szerinti értékcsökkenése","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_141",141,"Üzemi (üzleti) gépek, berendezések, felszerelések","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_142",142,"Egyéb járművek","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_143",143,"Irodai, igazgatási berendezések és felszerelések","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_144",144,"Üzemkörön kívüli berendezések, felszerelések, járművek","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_145",145,"Jóléti berendezések, felszerelési tárgyak és képzőművészeti alkotások","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_147",147,"Egyéb berendezések, felszerelések, járművek értékhelyesbítése","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_148",148,"Egyéb berendezések, felszerelések, járművek terven felüli értékcsökkenése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_149",149,"Egyéb berendezések, felszerelések, járművek terv szerinti értékcsökkenése","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_151",151,"Tenyészállatok","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_152",152,"Igásállatok","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_153",153,"Egyéb állatok","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_157",157,"Tenyészállatok értékhelyesbítése","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_158",158,"Tenyészállatok terven felüli értékcsökkenése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_159",159,"Tenyészállatok terv szerinti értékcsökkenése","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_161",161,"Befejezetlen beruházások","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_162",162,"Felújítások","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_168",168,"Beruházások terven felüli értékcsökk","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_171",171,"Tartós részesedés kapcsolt vállalkozásban","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_172",172,"Tartós jelentős tulajdonosi részesedés","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_173",173,"Egyéb tartós részesedés","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_177",177,"Részesedések értékhelyesbítése","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_178",178,"Tartós részesedések értékelési különbözete","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_179",179,"Részesedések értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_181",181,"Államkötvények","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_182",182,"Kapcsolt vállalkozások értékpapírjai","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_183",183,"Tartós jelentős tulajdonosi részesedésű vállalkozások értékpapírjai","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_184",184,"Egyéb vállalkozások értékpapírjai","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_185",185,"Tartós diszkont értékpapírok","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_188",188,"Hitelviszonyt megtestesítő értékpapírok értékelési különbözete","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_189",189,"Értékpapírok értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_191",191,"TTartósan adott kölcsönök kapcsolt vállalkozásban","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_192",192,"Tartósan adott kölcsönök, egyéb részesedési viszonyban álló vállalkozásban","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_193",193,"Egyéb tartósan adott kölcsönök","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_194",194,"Tartós bankbetétek kapcsolt vállalkozásban","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_195",195,"Tartós bankbetétek egyéb részesedési viszonyban álló vállalkozásban","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_196",196,"Egyéb tartós bankbetétek","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_197",197,"Pénzügyi lízing miatti tartós követelés","account.data_account_type_fixed_assets","FALSE",hungarian_chart_template
"l10n_hu_199",199,"Tartósan adott kölcsönök (és bankbetétek) értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_211",211,"Nyers- és alapanyagok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_221",221,"Segédanyagok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_222",222,"Üzem- és fűtőanyagok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_223",223,"Fenntartási anyagok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_224",224,"Építési anyagok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_225",225,"Egy éven belül elhasználódó anyagi eszközök","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_226",226,"Tárgyi eszközök közül átsorolt anyagok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_227",227,"Egyéb anyagok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_228",228,"Anyagok árkülönbözete","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_229",229,"Anyagok értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_231",231,"Befejezetlen termelés","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_235",235,"Félkész termékek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_238",238,"Félkész termékek készletérték-különbözete","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_239",239,"Befejezetlen termelés és félkész termékek értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_241",241,"Növendékállatok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_242",242,"Hízóállatok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_243",243,"Egyéb állatok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_246",246,"Bérbevett állatok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_248",248,"Állatok készletérték-különbözete","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_249",249,"Állatok értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_251",251,"Késztermékek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_258",258,"Késztermékek készletérték-különbözete","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_259",259,"Késztermékek értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_261",261,"Áruk beszerzési áron","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_262",262,"Áruk elszámoló áron","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_263",263,"Áruk árkülönbözete","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_264",264,"Áruk eladási áron","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_265",265,"Áruk árrése","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_266",266,"Idegen helyen tárolt, bizományba átadott áruk","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_267",267,"Tárgyi eszközök közül átsorolt áruk","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_269",269,"Kereskedelmi áruk értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_271",271,"Közvetített szolgáltatások","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_279",279,"Közvetített szolgáltatások értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_281",281,"Saját göngyölegek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_282",282,"Idegen göngyölegek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_288",288,"Betétdíjas göngyölegek árkülönbözete","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_289",289,"Betétdíjas göngyölegek értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_311",311,"Belföldi követelések (forintban)","account.data_account_type_receivable","TRUE",hungarian_chart_template
"l10n_hu_312",312,"Belföldi követelések (devizában)","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_315",315,"Belföldi követelések értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_316",316,"Külföldi követelések (forintban)","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_317",317,"Külföldi követelések (devizában)","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_318",318,"Követelések értékelési különbözete","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_319",319,"Külföldi követelések értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_321",321,"Követelések kapcsolt vállalkozással szemben","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_322",322,"Követelések jelentős tulajdoni részesedési viszonyban lévő vállalkozássalszemben","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_323",323,"Jegyzett, de még be nem fizetett tőke kapcsolat vállalkozástól","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_324",324,"Jegyzett, de még be nem fizetett tőke jelentős tulajdoni részesedési viszonyban lévő vállalkozástól","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_325",325,"Kapcsolt vállalkozással szembeni követelések értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_326",326,"Jelentős tulajdoni részesedési viszonyban lévő vállalkozással szembeni követelések értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_331",331,"Követelések egyéb részesedési viszonyban lévő vállalkozással szemben","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_332",332,"Jegyzett, de még be nem fizetett tőke az egyéb részesedési viszonyban lévő vállalkozástól","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_333",333,"Jegyzett, de még be nem fizetett tőke részesedési viszonyban nem lévő vállalkozástól","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_339",339,"Egyéb részesedési viszonyban lévő vállalkozással szembeni követelések értékvesztése és annak visszaírás","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_341",341,"Belföldi váltókövetelések","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_345",345,"Belföldi váltókövetelések értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_346",346,"Külföldi váltókövetelések","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_349",349,"Külföldi váltókövetelések értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_351",351,"Immateriális javakra adott előlegek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_352",352,"Beruházásokra adott előlegek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_353",353,"Készletekre adott előlegek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_354",354,"Szolgáltatásokra adott előlegek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_355",355,"Egyéb adott előlegek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_359",359,"Adott előlegek értékvesztése és visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_361",361,"Munkavállalókkal szembeni követelés","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3611",3611,"Munkavállalóknak folyósított előlegek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3612",3612,"Előírt tartozások","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3613",3613,"Egyéb elszámolások a munkavállalókkal","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_362",362,"Költségvetési kiutalási igények","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_363",363,"Költségvetési kiutalási igények teljesítése","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_364",364,"Rövid lejáratú kölcsönadott pénzeszközök","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_365",365,"Vásárolt és kapott követelések","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_366",366,"Részesedésekkel, értékpapírokkal kapcsolatos követelések","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_367",367,"Határidős, opciós és swapügyletekkel kapcsolatos követelések","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_368",368,"Különféle egyéb követelések","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3681",3681,"Bizományosi ügylettel kapcsolatos elszámolások","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3683",3683,"KImport beszerzések áfája","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3684",3684,"Adósok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3685",3685,"Biztosítóintézettel szembeni követelések","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3686",3686,"Barter ügylet elszámolási számla","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3687",3687,"Árfolyamkülönbözetek elszámolási számla","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3688",3688,"Immateriális javak és tárgyi eszközök elszámolási számla","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3689",3689,"Ki nem emelt egyéb követelések","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_369",369,"Egyéb követelések értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_371",371,"Részesedés kapcsolt vállalkozásban","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3711",3711,"Eladásra vásárolt részesedések kapcsolt vállalkozásban","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3719",3719,"Kapcsolt vállalkozásban lévő részesedések értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_372",372,"Jelentős tulajdonosi részesedés","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3721",3721,"Eladásra vásárolt jelentős tulajdonosi részesedések","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3729",3729,"Jelentős tulajdonosi részesedések értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_373",373,"Egyéb részesedés","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3731",3731,"Eladásra vásárolt egyéb részesedések","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3739",3739,"Egyéb részesedések értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_374",374,"Saját részvények, saját üzletrészek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3741",3741,"Visszavásárolt saját részvények, üzletrészek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3749",3749,"Saját részvények, saját üzletrészek értékvesztése","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_375",375,"Forgatási célú hitelviszonyt megtestesítő értékpapírok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3751",3751,"Eladási célra vásárolt hitelviszonyt megtestesítő értékpapírok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_3759",3759,"Forgatási célú hitelviszonyt megtestesítő értékpapírok értékvesztése és annak visszaírása","account.data_account_type_depreciation","FALSE",hungarian_chart_template
"l10n_hu_378",378,"Értékpapírok értékelési különbözete","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_379",379,"Értékpapír elszámolási számla","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_381",381,"Pénztár","account.data_account_type_liquidity","FALSE",hungarian_chart_template
"l10n_hu_382",382,"Valutapénztár","account.data_account_type_liquidity","FALSE",hungarian_chart_template
"l10n_hu_383",383,"Csekkek","account.data_account_type_liquidity","FALSE",hungarian_chart_template
"l10n_hu_384",384,"Elszámolási betétszámla","account.data_account_type_liquidity","FALSE",hungarian_chart_template
"l10n_hu_385",385,"Elkülönített betétszámlák","account.data_account_type_liquidity","FALSE",hungarian_chart_template
"l10n_hu_386",386,"Deviza betétszámla","account.data_account_type_liquidity","FALSE",hungarian_chart_template
"l10n_hu_389",389,"Belső áthelyezés","account.data_account_type_liquidity","FALSE",hungarian_chart_template
"l10n_hu_391",391,"Bevételek aktív időbeli elhatárolása","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_392",392,"Költségek, ráfordítások aktív időbeli elhatárolása","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_393",393,"Halasztott ráfordítások","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"l10n_hu_411",411,"Jegyzettőke","account.data_account_type_equity","FALSE",hungarian_chart_template
"l10n_hu_412",412,"Tőketartalék","account.data_account_type_equity","FALSE",hungarian_chart_template
"l10n_hu_413",413,"Eredménytartalék","account.data_account_type_equity","FALSE",hungarian_chart_template
"l10n_hu_414",414,"Lekötött tartalék","account.data_account_type_equity","FALSE",hungarian_chart_template
"l10n_hu_417",417,"Értékelési tartalék","account.data_account_type_equity","FALSE",hungarian_chart_template
"l10n_hu_4171",4171,"Értékhelyesbítés értékelési tartaléka","account.data_account_type_equity","FALSE",hungarian_chart_template
"l10n_hu_4172",4172,"Valós értékelés értékelési tartaléka","account.data_account_type_equity","FALSE",hungarian_chart_template
"l10n_hu_418",418,"Előző évek helyesbítéséből származó mérleg szerinti eredmény","account.data_account_type_equity","FALSE",hungarian_chart_template
"l10n_hu_419",419,"Adózott eredmény","account.data_account_type_equity","FALSE",hungarian_chart_template
"l10n_hu_421",421,"Céltartalék a várható kötelezettségekre","account.data_account_type_equity","FALSE",hungarian_chart_template
"l10n_hu_422",422,"Céltartalék a jövőbeni költségekre","account.data_account_type_equity","FALSE",hungarian_chart_template
"l10n_hu_429",429,"Egyéb céltartalék","account.data_account_type_equity","FALSE",hungarian_chart_template
"l10n_hu_431",431,"Hátrasorolt kötelezettségek kapcsolt vállalkozással szemben","account.data_account_type_non_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_432",432,"Hátrasorolt kötelezettségek jelentős tulajdoni részesedési viszonyban lévő vállalkozással szemben","account.data_account_type_non_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_433",433,"Hátrasorolt kötelezettségek egyéb részesedési viszonyban lévő vállalkozással szemben","account.data_account_type_non_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_441",441,"Hosszú lejáratra kapott kölcsönök","account.data_account_type_non_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_442",442,"Átváltoztatható kötvények","account.data_account_type_non_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_443",443,"Tartozások kötvénykibocsátásból","account.data_account_type_non_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_444",444,"Beruházási és fejlesztési hitelek","account.data_account_type_non_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_445",445,"Egyéb hosszú lejáratú hitelek","account.data_account_type_non_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_446",446,"Tartós kötelezettségek kapcsolt vállalkozással szemben","account.data_account_type_non_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_447",447,"Tartós kötelezettségek jelentős tulajdonosi részesedési viszonyban lévő vállalkozással szemben","account.data_account_type_non_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_448",448,"Tartós kötelezettségek részesedési viszonyban lévő vállalkozással szemben","account.data_account_type_non_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_449",449,"Egyéb hosszú lejáratú kötelezettségek","account.data_account_type_non_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_4491",4491,"Pénzügyi lízing miatti kötelezettségek","account.data_account_type_non_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_451",451,"Rövid lejáratú kölcsönök","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_4511",4511,"Rövid lejáratú hitelek átváltoztatható és átváltoztatható kötvényekből","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_452",452,"Rövid lejáratú hitelek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_453",453,"Vevőktől kapott előlegek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_454",454,"Szállítók","account.data_account_type_payable","TRUE",hungarian_chart_template
"l10n_hu_455",455,"Beruházási szállítók","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_456",456,"Váltótartozások","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_457",457,"Rövid lejáratú kötelezettségek kapcsolt vállalkozással szemben","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_458",458,"Rövid lejáratú kötelezettségek jelentős tulajdoni részesedési viszonyban lévő vállalkozásokkal szemben","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_459",459,"Rövid lejáratú kötelezettségek egyéb részesedési viszonyban lévő vállalkozással szemben","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_461",461,"Társasági adó","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_462",462,"Személyi jövedelemadó elszámolása","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_463",463,"Költségvetési befizetési kötelezettségek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_464",464,"Költségvetési befizetési kötelezettségek teljesítése","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_465",465,"Vám és import áfa tartozások","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_466",466,"Előzetesen felszámított általános forgalmi adó","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_467",467,"Fizetendő általános forgalmi adó","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_468",468,"Általános forgalmi adó pénzügyi elszámolása","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_469",469,"Helyi adók elszámolási számla","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_471",471,"Jövedelemelszámolási számla","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_472",472,"Fel nem vett járandóságok","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_473",473,"Társadalombiztosítási járulék kötelezettség","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_474",474,"Elkülönített alapokkal kapcsolatos fizetési kötelezettségek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_475",475,"Vagyonkezelő szervezetekkel szembeni kötelezettségek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_476",476,"Rövid lejáratú egyéb kötelezettségek munkavállalókkal és tagokkal szemben","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_4761",4761,"Kártérítés","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_4762",4762,"Bírói letiltás","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_4764",4764,"Levont szakszervezeti díj","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_4765",4765,"Magán nyugdíjpénztári befizetési kötelezettségek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_478",478,"Részesedésekkel, értékpapírokkal kapcsolatos kötelezettségek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_479",479,"Különféle rövid lejáratú egyéb kötelezettségek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_4791",4791,"Biztosító intézetekkel szembeni kötelezettségek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_4792",4792,"Hitelezők","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_4796",4796,"Kötelezettségek értékelési különbözete","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_481",481,"Bevételek passzív időbeli elhatárolása","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_482",482,"Költségek, ráfordítások passzív időbeli elhatárolása","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_483",483,"Halasztott bevételek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_491",491,"Nyitómérleg számla","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_492",492,"Zárómérleg számla","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_493",493,"Tárgyévi eredmény elszámolása","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_494",494,"Adózott eredmény elszámolása","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"l10n_hu_511",511,"Vásárolt anyagok költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5111",5111,"Alapanyag költségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5112",5112,"Segédanyag költségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5113",5113,"Üzemanyag költségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5114",5114,"Egy éven belül elhasználódó gyártóeszközök, berendezések, felszerelések és egyéb eszközök költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5115",5115,"Egy éven belül elhasználódó munkaruha, védőruha felhasználás költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5116",5116,"Nyomtatványok, irodaszerek költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5117",5117,"Fűtőanyag költségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5118",5118,"Villamosenergia felhasználás és vízfelhasználás költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5119",5119,"Egyéb anyagfelhasználás költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_512",512,"Egy éven belül elhasználódó anyagi eszközök költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_522",522,"Bérleti díjak","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_523",523,"Fuvarozási, szállítási, rakodási és raktározási költségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_524",524,"Javítás, karbantartás költsége","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_525",525,"Elektronikus adathordozón megjelent kiadványok költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_526",526,"Újságok, könyvek költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_527",527,"Posta, telefon, Internet és egyéb telekommunikációs költségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_528",528,"Mosoda, vegytisztítás, takarítás költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_529",529,"Egyéb igénybevett szolgáltatások költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5291",5291,"Fénymásolás, sokszorosítás költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5292",5292,"Távfűtés költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5293",5293,"Más vállalkozással végeztetett garanciális javítások költsége","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5294",5294,"Kiállítások, bemutatók, vásárok rendezési díja","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5295",5295,"Hirdetés, reklám, propaganda költségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5296",5296,"Oktatás és továbbképzés költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_531",531,"Hatósági igazgatási, szolgáltatási díjak, illetékek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_532",532,"Pénzügyi, befektetési szolgáltatási díjak","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5321",5321,"Bankköltség","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_533",533,"Biztosítási díj","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_534",534,"Költségként elszámolandó adók, járulékok, termékdíj","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_539",539,"Különféle egyéb költségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_541",541,"Bérköltség","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_542",542,"Tulajdonos személyes közreműködésének ellenértéke","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_551",551,"Munkavállalóknak, tagoknak fizetett személyi jellegű kifizetések","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5511",5511,"Betegszabadság díja, munkáltatót terhelő táppénz, táppénz kiegészítés","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5512",5512,"Végkielégítés","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5513",5513,"Munkábajárással kapcsolatos egyéb költségek térítése","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5514",5514,"Kiküldetés napidíja","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5515",5515,"Megváltozott munkaképességű munkavállalók keresetkiegészítése, fizetett segélyek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5516",5516,"Üdülési hozzájárulás","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5517",5517,"Lakásépítésre nyújtott támogatás, albérleti hozzájárulás","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5518",5518,"Jubileumi jutalom, tárgyjutalom","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_552",552,"Jóléti és kulturális költségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_559",559,"Egyéb személyi jellegű kifizetések","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5591",5591,"Munkáltató által fizetett baleset-, élet- és nyugdíjbiztosítás díja","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5592",5592,"Munkáltató által önkéntes pénztárba befizetett munkáltatói tagdíj hozzájárulás","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5593",5593,"Munkáltatót terhelő személyi jövedelemadó","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5594",5594,"Munkáltatói hozzájárulás a korengedményes nyugdíj igénybevételéhez","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5595",5595,"Találmányi díj, szabadalom vételára és hasznosítási díja","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5596",5596,"Fizetett szerzői, írói és más jogvédelmet élvező munkák díjai és ezekkel kapcsolatos közreműködői díjak","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5597",5597,"Fizetett ösztöndíjak","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5598",5598,"Reprezentációs költségek, étkezési hozzájárulás","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_5599",5599,"EMunkavállalókkal kapcsolatos biztosítási díjak","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_561",561,"Szociális hozzájárulási adó","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_564",564,"Szakképzési hozzájárulás","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_565",565,"Rehabilitációs hozzájárulás","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_571",571,"Terv szerinti értékcsökkenési leírás","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_572",572,"Kisértékű (200 eFt egyedi beszerzési érték alatti) eszközök terv szerinti értékcsökkenési leírása","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_581",581,"Saját termelésű készletek állományváltozása","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_582",582,"Saját előállítású eszközök aktivált értéke","account.data_account_type_expenses","FALSE",hungarian_chart_template
"l10n_hu_811",811,"Anyagköltség","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_812",812,"Igénybevett szolgáltatások értéke","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_813",813,"Egyéb szolgáltatások értéke","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_814",814,"Eladott áruk beszerzési értéke","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_815",815,"Eladott (közvetített) szolgáltatások értéke","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_821",821,"Bérköltség","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_822",822,"Személyi jellegü egyéb kifizetések","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_823",823,"Bérjárulékok","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_851",851,"Értékesítési, forgalmazási költségek","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_852",852,"Igazgatási költségek","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_853",853,"Egyéb általános költségek","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_861",861,"Értékesített immateriális javak, tárgyi eszközök értékesítésének realizált vesztesége","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_862",862,"Értékesített, átruházott (engedményezett) követelések könyv szerinti értéke","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_863",863,"Mérleg fordulónap előtt bekövetkezett eseményeknek az üzleti évhez kapcsolódó ráfordításai","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8631",8631,"Káreseménnyel kapcsolatos fizetések, fizetendő összegek","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8632",8632,"Bírságok, kötbérek, fekbérek, késedelmi kamatok, kártérítések","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_864",864,"Utólag adott - közvetve kapcsolódó - pénzügyileg rendezett engedmény","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_865",865,"Céltartalék képzése","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_866",866,"Értékvesztés, terven felüli értékcsökkenés","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8661",8661,"Készletek elszámolt értékvesztése","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8662",8662,"Követelések elszámolt értékvesztése","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8663",8663,"Immateriális javak elszámolt terven felüli értékcsökkenése","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8664",8664,"Tárgyi eszközök elszámolt terven felüli értékcsökkenése","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_867",867,"Adók, illetékek, hozzájárulások","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_868",868,"Kivételes nagyságú vagy előfordulású ráfordítások","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8681",8681,"Társaságba bevitt vagyontárgyak nyilvántartás szerinti értéke","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8682",8682,"Behajthatatlannak nem minősülő elengedett követelés könyv szerinti értéke","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8683",8683,"Tartozásátvállalás szerződés szerinti összege","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8684",8684,"Visszafizetési kötelezettség nélkül átadott, pénzügyileg rendezett támogatás","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8685",8685,"Fejlesztési célra kapott támogatás visszafizetendő összege","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8686",8686,"Térítés nélkül átadott eszközök nyilvántartás szerinti értéke","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8687",8687,"Térítés nélkül nyújtott szolgáltatások bekerülési értéke","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_869",869,"Különféle egyéb ráfordítások","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8691",8691,"Behajthatatlan követelés leírt összege","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8692",8692,"Hiányzó, megsemmisült, állományból kivezetett immateriális javak, tárgyi eszközök könyv szerinti értéke","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8693",8693,"Hiányzó, megsemmisült, állományból kivezetett készletek könyv szerinti értéke","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8694",8694,"Kereskedelmi áruk veszteségjellegű leltárértékelési különbözete","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_871",871,"Részesedésekből származó ráfordítások és árfolyamveszteségek","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8711",8711,"Részvényekből származó költségek, árfolyamveszteségek kapcsolt vállalkozásoknak","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_872",872,"Befektetett pénzügyi eszközökből (értékpapírból, kölcsönökből) származó ráfordítások, árfolyamveszteségek","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8721",8721,"Befektetett pénzügyi eszközök (értékpapírok, kölcsönök) ráfordításaiból, kapcsolt vállalkozások árfolyamveszteségéből származik","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_873",873,"Fizetendő (fizetett) kamatok és kamatjellegű ráfordítások","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8731",8731,"Fizetendő kamat (fizetett kamat) és kamatszerű kiadások kapcsolt vállalkozásnak","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_874",874,"Részesedések, értékpapírok, bankbetétek értékvesztése","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_875",875,"Forgóeszközök között kimutatott befektetés, értékpapír értékesítésének, beváltásának árfolyamvesztesége","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_876",876,"Átváltáskori, értékeléskori árfolyamveszteség","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8761",8761,"Deviza- és valutakészletek forintra átváltásának árfolyam-vesztesége","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8762",8762,"Külföldi pénzértékre szóló eszközök és kötelezettségek pénzügyileg rendezett árfolyamvesztesége","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8763",8763,"Külföldi pénzértékre szóló eszközök és kötelezettségek mérlegfordulónapi értékelésének összevont árfolyamvesztesége","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_877",877,"Egyéb árfolyamveszteségek, opciós díjak","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_878",878,"Vásárolt követelésekkel kapcsolatos ráfordítások","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_879",879,"Egyéb pénzügyi ráfordítások","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_8791",8791,"Értékelési különbözet egyéb pénzügyi ráfordítások","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_891",891,"Társasági adó","account.data_account_type_direct_costs","FALSE",hungarian_chart_template
"l10n_hu_911",911,"Belföldi értékesítés árbevétele","account.data_account_type_revenue","FALSE",hungarian_chart_template
"l10n_hu_931",931,"Exportértékesítés árbevétele","account.data_account_type_revenue","FALSE",hungarian_chart_template
"l10n_hu_961",961,"Értékesített immateriális javak, tárgyi eszközök értékesítésének realizált nyeresége","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_962",962,"Értékesített, átruházott (engedményezett) követelések elismert értéke","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_963",963,"A mérlegkészítés időpontjáig pénzügyileg rendezett, az üzleti évhez kapcsolódó egyéb bevételek","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9631",9631,"Káreseményekkel kapcsolatosan kapott bevételek","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9632",9632,"Kapott bírságok, kötbérek, fekbérek, késedelmi kamatok, kártérítések","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9633",9633,"Behajthatatlannak minősített és leírt követelésekre kapott összegek","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_964",964,"Utólag kapott - közvetve kapcsolódó - pénzügyileg rendezett engedmény","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_965",965,"Céltartalék felhasználása (csökkenése, megszűnése)","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_966",966,"Visszaírt értékvesztés, terven felüli értékcsökkenés","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9661",9661,"Készletek visszaírt értékvesztése","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9662",9662,"Követelések visszaírt értékvesztése","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9663",9663,"Immateriális javak visszaírt terven felüli értékcsökkenése","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9664",9664,"Tárgyi eszközök visszaírt terven felüli értékcsökkenése","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_967",967,"Visszafizetési kötelezettség nélkül kapott támogatás, juttatás","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_968",968,"Kivételes nagyságú vagy előfordulású bevételek","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9681",9681,"Társaságba bevitt vagyontárgyak létesítő okiratban meghatározott értéke","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9682",9682,"Elengedett kötelezettségek értéke","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9683",9683,"Tartozásátvállalás során harmadik személy által átvállalt kötelezettség","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9684",9684,"Visszafizetési kötelezettség nélkül kapott, véglegesen átvett pénzeszköz","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9685",9685,"Fejlesztési célra visszafizetési kötelezettség nélkül kapott véglegesen átvett pénzeszköz","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9686",9686,"Térítés nélkül átvett, ajándékként, hagyatékként kapott, fellelt eszközök piaci értéke","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9687",9687,"Térítés nélkül kapott szolgáltatások piaci értéke","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_969",969,"Különféle egyéb bevételek","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9691",9691,"Biztosító által visszaigazolt kártérítés összege","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_971",971,"Kapott (járó) osztalék és részesedés","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9711",9711,"Kapcsolt vállalkozástól kapott (járó) osztalék és részesedés","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_972",972,"Részesedések értékesítésének árfolyamnyeresége","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9721",9721,"Kapcsolt vállalkozásnak értékesített részesedés árfolyamnyeresége","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_973",973,"Befektetett pénzügyi eszközök kamatai, árfolyamnyeresége","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9731",9731,"Befektetett pénzügyi eszközök kamatai, devizaárfolyam nyereség kapcsolt vállalkozástól","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_974",974,"Egyéb kapott (járó) kamatok és kamatjellegű bevételek","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9741",9741,"Egyéb kamatkövetelések és hasonló bevételek kapcsolt vállalkozástól","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_975",975,"Forgóeszközök között kimutatott befektetés, értékpapír értékesítésének, beváltásának árfolyamnyeresége","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_976",976,"Átváltási, értékeléskori árfolyamnyereség","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9761",9761,"Deviza- és valutakészletek forintra átváltásának árfolyamnyeresége","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9762",9762,"Külföldi pénzértékre szóló eszközök és kötelezettségek pénzügyileg rendezett árfolyamnyeresége","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9763",9763,"Külföldi pénzértékre szóló eszközök és kötelezettségek mérlegfordulónapi értékelésének összevont árfolyamnyeresége","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_977",977,"Egyéb árfolyamnyereségek, opciós díjbevételek","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_978",978,"Vásárolt követelésekkel kapcsolatos bevételek","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_979",979,"Egyéb pénzügyi bevételek","account.data_account_type_other_income","FALSE",hungarian_chart_template
"l10n_hu_9791",9791,"Egyéb pénzügyi bevételek értékelési különbözete","account.data_account_type_other_income","FALSE",hungarian_chart_template

```

## File: data\account.fiscal.position.tax.template.csv

```csv
"id","tax_src_id/id","tax_dest_id/id","position_id/id"
"fiscal_position_hu_exempt_tax_F27","F27","FA","fiscal_position_hu_exempt"
"fiscal_position_hu_exempt_tax_F18","F18","FA","fiscal_position_hu_exempt"
"fiscal_position_hu_exempt_tax_F5","F5","FA","fiscal_position_hu_exempt"
"fiscal_position_hu_exempt_tax_V27","V27","VA","fiscal_position_hu_exempt"
"fiscal_position_hu_exempt_tax_V27TE","V27TE","VA","fiscal_position_hu_exempt"
"fiscal_position_hu_exempt_tax_V18","V18","VA","fiscal_position_hu_exempt"
"fiscal_position_hu_exempt_tax_V5","V5","VA","fiscal_position_hu_exempt"
"fiscal_position_hu_eu_F27","F27","FEUT","fiscal_position_hu_eu"
"fiscal_position_hu_eu_F18","F18","FEUT","fiscal_position_hu_eu"
"fiscal_position_hu_eu_F5","F5","FEUT","fiscal_position_hu_eu"
"fiscal_position_hu_eu_V27","V27","VEU27T","fiscal_position_hu_eu"
"fiscal_position_hu_eu_V27TE","V27TE","VEU27TE","fiscal_position_hu_eu"
"fiscal_position_hu_eu_out_F27","F27","FEXT","fiscal_position_hu_eu_out"
"fiscal_position_hu_eu_out_F18","F18","FEXT","fiscal_position_hu_eu_out"
"fiscal_position_hu_eu_out_F5","F5","FEXT","fiscal_position_hu_eu_out"
"fiscal_position_hu_eu_out_V27","V27","VIMK","fiscal_position_hu_eu_out"
"fiscal_position_hu_eu_out_V27TE","V27TE","VIMK","fiscal_position_hu_eu_out"

```

## File: data\account.fiscal.position.template.csv

```csv
"id","chart_template_id/id","name","sequence","auto_apply","vat_required","country_id:id","country_group_id:id"
"fiscal_position_hu_exempt","hungarian_chart_template","Alanyi adómentes",,,,,
"fiscal_position_hu_national","hungarian_chart_template","Magyar",1,1,1,base.hu,
"fiscal_position_hu_eu_private","hungarian_chart_template","EU partner private",2,1,,,base.europe
"fiscal_position_hu_eu","hungarian_chart_template","EU partner",3,1,1,,base.europe
"fiscal_position_hu_eu_out","hungarian_chart_template","EU-n kívüli partner",4,1,,,

```

## File: data\account.group.template.csv

```csv
id,code_prefix_start,code_prefix_end,name,chart_template_id/id
l10n_hu_group_1,1,,"BEFEKTETETT ESZKÖZÖK",l10n_hu.hungarian_chart_template
l10n_hu_group_11,11,,"IMMATERIÁLIS JAVAK",l10n_hu.hungarian_chart_template
l10n_hu_group_12,12,,"INGATLANOK ÉS KAPCSOLÓDÓ VAGYONI ÉRTÉKŰ JOGOK",l10n_hu.hungarian_chart_template
l10n_hu_group_13,13,,"MŰSZAKI BERENDEZÉSEK, GÉPEK, JÁRMŰVEK",l10n_hu.hungarian_chart_template
l10n_hu_group_14,14,,"EGYÉB BERENDEZÉSEK, FELSZERELÉSEK, JÁRMŰVEK",l10n_hu.hungarian_chart_template
l10n_hu_group_15,15,,"TENYÉSZÁLLATOK",l10n_hu.hungarian_chart_template
l10n_hu_group_16,16,,"BERUHÁZÁSOK, FELÚJÍTÁSOK",l10n_hu.hungarian_chart_template
l10n_hu_group_17,17,,"TULAJDONI RÉSZESEDÉST JELENTŐ BEFEKTETÉSEK (RÉSZESEDÉSEK)",l10n_hu.hungarian_chart_template
l10n_hu_group_18,18,,"HITELVISZONYT MEGTESTESÍTŐ ÉRTÉKPAPÍROK",l10n_hu.hungarian_chart_template
l10n_hu_group_19,19,,"TARTÓSAN ADOTT KÖLCSÖNÖK",l10n_hu.hungarian_chart_template
l10n_hu_group_2,2,,"KÉSZLETEK",l10n_hu.hungarian_chart_template
l10n_hu_group_21-22,21,22,"ANYAGOK",l10n_hu.hungarian_chart_template
l10n_hu_group_23,23,,"BEFEJEZETLEN TERMELÉS ÉS FÉLKÉSZ TERMÉKEK",l10n_hu.hungarian_chart_template
l10n_hu_group_24,24,,"NÖVENDÉK-, HÍZÓ- ÉS EGYÉB ÁLLATOK",l10n_hu.hungarian_chart_template
l10n_hu_group_25,25,,"KÉSZTERMÉKEK",l10n_hu.hungarian_chart_template
l10n_hu_group_26,26,,"NKERESKEDELMI ÁRUK",l10n_hu.hungarian_chart_template
l10n_hu_group_27,27,,"KÖZVETÍTETT SZOLGÁLTATÁSOK",l10n_hu.hungarian_chart_template
l10n_hu_group_28,28,,"BETÉTDÍJAS GÖNGYÖLEGEK",l10n_hu.hungarian_chart_template
l10n_hu_group_3,3,,"KÖVETELÉSEK, ÉRTÉKPAPÍROK, PÉNZESZKÖZÖK ÉS AKTÍV IDŐBELIELHATÁROLÁSOK",l10n_hu.hungarian_chart_template
l10n_hu_group_31,31,,"KÖVETELÉSEK ÁRUSZÁLLÍTÁSBÓL ÉS SZOLGÁLTATÁSBÓL (VEVŐK)",l10n_hu.hungarian_chart_template
l10n_hu_group_32,32,,"KÖVETELÉSEK KAPCSOLT VÁLLALKOZÁSSAL SZEMBEN",l10n_hu.hungarian_chart_template
l10n_hu_group_33,33,,"KÖVETELÉSEK EGYÉB RÉSZESEDÉSI VISZONYBAN LÉVŐ VÁLLALKOZÁSSAL SZEMBEN",l10n_hu.hungarian_chart_template
l10n_hu_group_34,34,,"VÁLTÓKÖVETELÉSEK",l10n_hu.hungarian_chart_template
l10n_hu_group_35,35,,"ADOTT ELŐLEGEK",l10n_hu.hungarian_chart_template
l10n_hu_group_36,36,,"EGYÉB KÖVETELÉSEK",l10n_hu.hungarian_chart_template
l10n_hu_group_37,37,,"ÉRTÉKPAPÍROK",l10n_hu.hungarian_chart_template
l10n_hu_group_38,38,,"PÉNZESZKÖZÖK",l10n_hu.hungarian_chart_template
l10n_hu_group_39,39,,"EAKTÍV IDŐBELI ELHATÁROLÁSOK",l10n_hu.hungarian_chart_template
l10n_hu_group_4,4,,"FORRÁSOK",l10n_hu.hungarian_chart_template
l10n_hu_group_41,41,,"SAJÁT TŐKE",l10n_hu.hungarian_chart_template
l10n_hu_group_42,42,,"CÉLTARTALÉKOK",l10n_hu.hungarian_chart_template
l10n_hu_group_43,43,,"HÁTRASOROLT KÖTELEZETTSÉGEK",l10n_hu.hungarian_chart_template
l10n_hu_group_44,44,,"HOSSZÚ LEJÁRATÚ KÖTELEZETTSÉGEK",l10n_hu.hungarian_chart_template
l10n_hu_group_45-47,45,47,"RÖVID LEJÁRATÚ KÖTELEZETTSÉGEK",l10n_hu.hungarian_chart_template
l10n_hu_group_48,48,,"PASSZÍV IDŐBELI ELHATÁROLÁSOK",l10n_hu.hungarian_chart_template
l10n_hu_group_49,49,,"ÉVI MÉRLEGSZÁMLÁK",l10n_hu.hungarian_chart_template
l10n_hu_group_5,5,,"KÖLTSÉGNEMEK",l10n_hu.hungarian_chart_template
l10n_hu_group_51,51,,"ANYAGKÖLTSÉG",l10n_hu.hungarian_chart_template
l10n_hu_group_52,52,,"IGÉNYBE VETT SZOLGÁLTATÁSOK KÖLTSÉGEI",l10n_hu.hungarian_chart_template
l10n_hu_group_53,53,,"EGYÉB SZOLGÁLTATÁSOK KÖLTSÉGEI",l10n_hu.hungarian_chart_template
l10n_hu_group_54,54,,"BÉRKÖLTSÉG",l10n_hu.hungarian_chart_template
l10n_hu_group_55,55,,"SZEMÉLYI JELLEGŰ EGYÉB KIFIZETÉSEK",l10n_hu.hungarian_chart_template
l10n_hu_group_56,56,,"BÉRJÁRULÉKOK",l10n_hu.hungarian_chart_template
l10n_hu_group_57,57,,"ÉRTÉKCSÖKKENÉSI LEÍRÁS",l10n_hu.hungarian_chart_template
l10n_hu_group_58,58,,"AKTIVÁLT SAJÁT TELJESÍTMÉNYEK ÉRTÉKE",l10n_hu.hungarian_chart_template
l10n_hu_group_59,59,,"KÖLTSÉGNEM ÁTVEZETÉSI SZÁMLA",l10n_hu.hungarian_chart_template
l10n_hu_group_6,6,,"KÖLTSÉGHELYEK, ÁLTALÁNOS KÖLTSÉGEK",l10n_hu.hungarian_chart_template
l10n_hu_group_61,61,,"JAVÍTÓ-KARBANTARTÓ ÜZEMEK KÖLTSÉGEI",l10n_hu.hungarian_chart_template
l10n_hu_group_62,62,,"SZOLGÁLTATÁST VÉGZŐ ÜZEMEK (EGYSÉGEK) KÖLTSÉGEI",l10n_hu.hungarian_chart_template
l10n_hu_group_63,63,,"GÉPKÖLTSÉG",l10n_hu.hungarian_chart_template
l10n_hu_group_64-65,64,65,"ÜZEMI IRÁNYÍTÁS ÁLTALÁNOS KÖLTSÉGEI",l10n_hu.hungarian_chart_template
l10n_hu_group_66,66,,"KÖZPONTI IRÁNYÍTÁS ÁLTLÁNOS KÖLTSÉGEI",l10n_hu.hungarian_chart_template
l10n_hu_group_67,67,,"ÉRTÉKESÍTÉSI, FORGALMAZÁSI KÖLTSÉGEK",l10n_hu.hungarian_chart_template
l10n_hu_group_68,68,,"ELKÜLÖNÍTETT EGYÉB ÁLTLÁNOS KÖLTSÉGEK",l10n_hu.hungarian_chart_template
l10n_hu_group_69,69,,"KÖLTSÉGHELYEK KÖLTSÉGNEMEK ÁTVEZETÉSE",l10n_hu.hungarian_chart_template
l10n_hu_group_7,7,,"TEVÉKENYSÉGEKKÖLTSÉGEI",l10n_hu.hungarian_chart_template
l10n_hu_group_71-74,71,74,"TERMELÉS KÖLTSÉGEI",l10n_hu.hungarian_chart_template
l10n_hu_group_75,75,,"SZOLGÁLTATÁS KÖLTSÉGEI",l10n_hu.hungarian_chart_template
l10n_hu_group_76,76,,"KÖLTSÉGHELYEK TERMELÉSI KÖLTSÉGEI",l10n_hu.hungarian_chart_template
l10n_hu_group_77-78,77,78,"FORGALOMBAHOZATAL KÖLTSÉGEI",l10n_hu.hungarian_chart_template
l10n_hu_group_79,79,,"TEVÉKENYSÉGEK KÖLTSÉGEINEK ÁTVEZETÉSE",l10n_hu.hungarian_chart_template
l10n_hu_group_8,8,,"ÉRTÉKESÍTÉS ELSZÁMOLT ÖNKÖLTSÉGE ÉS RÁFORDÍTÁSOK",l10n_hu.hungarian_chart_template
l10n_hu_group_81,81,,"ANYAGJELLEGŰ RÁFORDÍTÁSOK",l10n_hu.hungarian_chart_template
l10n_hu_group_82,82,,"SZEMÉLYI JELLEGŰ RÁFORDÍTÁSOK",l10n_hu.hungarian_chart_template
l10n_hu_group_83,83,,"ÉRTÉKCSÖKKENÉSI LEÍRÁS",l10n_hu.hungarian_chart_template
l10n_hu_group_86,86,,"EGYÉB RÁFORDÍTÁSOK",l10n_hu.hungarian_chart_template
l10n_hu_group_87,87,,"PÉNZÜGYI MŰVELETEK RÁFORDÍTÁSAI",l10n_hu.hungarian_chart_template
l10n_hu_group_89,89,,"NYERESÉGET TERHELŐ ADÓK",l10n_hu.hungarian_chart_template
l10n_hu_group_9,9,,"ÉRTÉKESÍTÉS ÁRBEVÉTELE ÉS BEVÉTELEK",l10n_hu.hungarian_chart_template
l10n_hu_group_91-92,91,92,"BELFÖLDI ÉRTÉKESÍTÉS ÁRBEVÉTELE",l10n_hu.hungarian_chart_template
l10n_hu_group_93-94,93,94,"EXPORTÉRTÉKESÍTÉS ÁRBEVÉTELE",l10n_hu.hungarian_chart_template
l10n_hu_group_96,96,,"EGYÉB BEVÉTELEK",l10n_hu.hungarian_chart_template
l10n_hu_group_97,97,,"PÉNZÜGYI MŰVELETEK BEVÉTELEI",l10n_hu.hungarian_chart_template
l10n_hu_group_0,0,,"NYILVÁNTARTÁSI SZÁMLÁK",l10n_hu.hungarian_chart_template

```

## File: data\account.tax.group.csv

```csv
id,name,country_id/id
tax_group_afa_0,Áfa 0%,base.hu
tax_group_afa_5,Áfa 5%,base.hu
tax_group_afa_18,Áfa 18%,base.hu
tax_group_afa_27,Áfa 27%,base.hu
tax_group_afa_komp,Komp. felár,base.hu

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_hu.hungarian_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hungarian_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="l10n_hu_311"/>
        <field name="property_account_payable_id" ref="l10n_hu_454"/>
        <field name="property_account_expense_id" ref="l10n_hu_811"/>
        <field name="property_account_income_id" ref="l10n_hu_911"/>
        <field name="property_account_expense_categ_id" ref="l10n_hu_811"/>
        <field name="property_account_income_categ_id" ref="l10n_hu_911"/>
        <field name="income_currency_exchange_account_id" ref="l10n_hu_976"/>
        <field name="expense_currency_exchange_account_id" ref="l10n_hu_876"/>
        <field name="default_pos_receivable_account_id" ref="l10n_hu_312"/>
    </record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.hu"/>
    </record>

    <!-- VAT base -->
    <record id="tax_report_alap" model="account.tax.report.line">
        <field name="name">ÁFA alap</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>
        <!-- Tax base - VAT payable on exports -->
        <record id="tax_report_alap_fiz_export" model="account.tax.report.line">
            <field name="name">Adóalap - Fizetendő ÁFA Export</field>
            <field name="tag_name">Adóalap - Fizetendő ÁFA Export</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="1"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - VAT payable EU -->
        <record id="tax_report_alap_fiz_eu" model="account.tax.report.line">
            <field name="name">Adóalap - Fizetendő ÁFA EU</field>
            <field name="tag_name">Adóalap - Fizetendő ÁFA EU</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="2"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - VAT payable is exempt from property tax -->
        <record id="tax_report_alap_fiz_targyi" model="account.tax.report.line">
            <field name="name">Adóalap - Fizetendő ÁFA tárgyi adómentes</field>
            <field name="tag_name">Adóalap - Fizetendő ÁFA tárgyi adómentes</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="3"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - VAT payable is exempt from tax -->
        <record id="tax_report_alap_fiz_alanyi" model="account.tax.report.line">
            <field name="name">Adóalap - Fizetendő ÁFA alanyi adómentes</field>
            <field name="tag_name">Adóalap - Fizetendő ÁFA alanyi adómentes</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="4"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - VAT payable outside the scope -->
        <record id="tax_report_alap_fiz_koron_kivuli" model="account.tax.report.line">
            <field name="name">Adóalap - Fizetendő ÁFA körön kívüli</field>
            <field name="tag_name">Adóalap - Fizetendő ÁFA körön kívüli</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="5"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - VAT payable 5% -->
        <record id="tax_report_alap_fiz_afa_5" model="account.tax.report.line">
            <field name="name">Adóalap - Fizetendő ÁFA 5%</field>
            <field name="tag_name">Adóalap - Fizetendő ÁFA 5%</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="6"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - VAT payable 18% -->
        <record id="tax_report_alap_fiz_afa_18" model="account.tax.report.line">
            <field name="name">Adóalap - Fizetendő ÁFA 18%</field>
            <field name="tag_name">Adóalap - Fizetendő ÁFA 18%</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="7"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - VAT payable 27% -->
        <record id="tax_report_alap_fiz_afa_27" model="account.tax.report.line">
            <field name="name">Adóalap - Fizetendő ÁFA 27%</field>
            <field name="tag_name">Adóalap - Fizetendő ÁFA 27%</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="8"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - Recoverable VAT EU -->
        <record id="tax_report_alap_viss" model="account.tax.report.line">
            <field name="name">Adóalap - Visszaigényelhető ÁFA EU</field>
            <field name="tag_name">Adóalap - Visszaigényelhető ÁFA EU</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="9"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - Import VAT -->
        <record id="tax_report_alap_import" model="account.tax.report.line">
            <field name="name">Adóalap – Import ÁFA</field>
            <field name="tag_name">Adóalap – Import ÁFA</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="10"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - Reverse charge -->
        <record id="tax_report_alap_forditott" model="account.tax.report.line">
            <field name="name">Adóalap – Fordított ÁFA</field>
            <field name="tag_name">Adóalap – Fordított ÁFA</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="11"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - Reclaimable VAT is exempt from tax -->
        <record id="tax_report_alap_viss_alanyi" model="account.tax.report.line">
            <field name="name">Adóalap - Visszaigényelhető ÁFA alanyi adómentes</field>
            <field name="tag_name">Adóalap - Visszaigényelhető ÁFA alanyi adómentes</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="12"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - Recoverable VAT is exempt from material tax -->
        <record id="tax_report_alap_viss_targyi" model="account.tax.report.line">
            <field name="name">Adóalap - Visszaigényelhető ÁFA tárgyi adómentes</field>
            <field name="tag_name">Adóalap - Visszaigényelhető ÁFA tárgyi adómentes</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="13"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - Recoverable VAT outside the scope -->
        <record id="tax_report_alap_viss_koron_kivuli" model="account.tax.report.line">
            <field name="name">Adóalap - Visszaigényelhető ÁFA körön kívüli</field>
            <field name="tag_name">Adóalap - Visszaigényelhető ÁFA körön kívüli</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="14"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - Recoverable VAT 5% -->
        <record id="tax_report_alap_viss_5" model="account.tax.report.line">
            <field name="name">Adóalap - Visszaigényelhető ÁFA 5%</field>
            <field name="tag_name">Adóalap - Visszaigényelhető ÁFA 5%</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="15"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - Recoverable VAT 18% -->
        <record id="tax_report_alap_viss_18" model="account.tax.report.line">
            <field name="name">Adóalap - Visszaigényelhető ÁFA 18%</field>
            <field name="tag_name">Adóalap - Visszaigényelhető ÁFA 18%</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="16"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax base - Recoverable VAT 27% -->
        <record id="tax_report_alap_viss_27" model="account.tax.report.line">
            <field name="name">Adóalap - Visszaigényelhető ÁFA 27%</field>
            <field name="tag_name">Adóalap - Visszaigényelhető ÁFA 27%</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="17"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>

        <!-- Tax Base - Reimbursable Compensation Surcharge -->
        <record id="tax_report_alap_komp" model="account.tax.report.line">
            <field name="name">Adóalap - Visszaigényelhető kompenzációs felár</field>
            <field name="tag_name">Adóalap - Visszaigényelhető kompenzációs felár</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="18"/>
            <field name="parent_id" ref="tax_report_alap"/>
        </record>
    <!-- VAT base [END] -->

    <!-- VAT is payable / reclaimable -->
    <record id="tax_report_fizetndo" model="account.tax.report.line">
        <field name="name">ÁFA fizetndő / visszaigényelhető</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>
        <!-- VAT payable 5% -->
        <record id="tax_report_fizetndo_5" model="account.tax.report.line">
            <field name="name">Fizetendő ÁFA 5%</field>
            <field name="tag_name">Fizetendő ÁFA 5%</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="1"/>
            <field name="parent_id" ref="tax_report_fizetndo"/>
        </record>

        <!-- VAT payable 18% -->
        <record id="tax_report_fizetndo_18" model="account.tax.report.line">
            <field name="name">Fizetendő ÁFA 18%</field>
            <field name="tag_name">Fizetendő ÁFA 18%</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="2"/>
            <field name="parent_id" ref="tax_report_fizetndo"/>
        </record>

        <!-- VAT payable 27% -->
        <record id="tax_report_fizetndo_27" model="account.tax.report.line">
            <field name="name">Fizetendő ÁFA 27%</field>
            <field name="tag_name">Fizetendő ÁFA 27%</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="3"/>
            <field name="parent_id" ref="tax_report_fizetndo"/>
        </record>

        <!-- Recoverable VAT 5% -->
        <record id="tax_report_fizetndo_viss_5" model="account.tax.report.line">
            <field name="name">Visszaigényelhető ÁFA 5%</field>
            <field name="tag_name">Visszaigényelhető ÁFA 5%</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="4"/>
            <field name="parent_id" ref="tax_report_fizetndo"/>
        </record>

        <!-- Recoverable VAT 18% -->
        <record id="tax_report_fizetndo_viss_18" model="account.tax.report.line">
            <field name="name">Visszaigényelhető ÁFA 18%</field>
            <field name="tag_name">Visszaigényelhető ÁFA 18%</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="5"/>
            <field name="parent_id" ref="tax_report_fizetndo"/>
        </record>

        <!-- Recoverable VAT 27% -->
        <record id="tax_report_fizetndo_viss_27" model="account.tax.report.line">
            <field name="name">Visszaigényelhető ÁFA 27%</field>
            <field name="tag_name">Visszaigényelhető ÁFA 27%</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="6"/>
            <field name="parent_id" ref="tax_report_fizetndo"/>
        </record>

        <!-- Reimbursable compensation surcharge -->
        <record id="tax_report_fizetndo_viss_komp" model="account.tax.report.line">
            <field name="name">Visszaigényelhető kompenzációs felár</field>
            <field name="tag_name">Visszaigényelhető kompenzációs felár</field>
            <field name="report_id" ref="tax_report"/>
            <field name="sequence" eval="7"/>
            <field name="parent_id" ref="tax_report_fizetndo"/>
        </record>
    <!-- VAT is payable / reclaimable [END] -->
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Sales -->
        <!-- Payable 27% -->
        <record id="F27" model="account.tax.template">
            <field name="description">27%</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="name">Fizetendő 27%</field>
            <field name="amount_type">percent</field>
            <field name="amount">27</field>
            <field name="sequence">1010</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_27"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_fiz_afa_27')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_467'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_27')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_alap_fiz_afa_27')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_467'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_27')],
                }),
            ]"/>
        </record>

        <!-- Payable 18% -->
        <record id="F18" model="account.tax.template">
            <field name="description">18%</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="name">Fizetendő 18%</field>
            <field name="amount_type">percent</field>
            <field name="amount">18</field>
            <field name="sequence">1020</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_18"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_fiz_afa_18')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_467'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_18')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_alap_fiz_afa_18')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_467'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_18')],
                }),
            ]"/>
        </record>

        <!-- Payable 5% -->
        <record id="F5" model="account.tax.template">
            <field name="description">5%</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="name">Fizetendő 5%</field>
            <field name="amount_type">percent</field>
            <field name="amount">5</field>
            <field name="sequence">1030</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_5"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_fiz_afa_5')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_467'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_5')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_alap_fiz_afa_5')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_467'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_5')],
                }),
            ]"/>
        </record>

        <!-- Payable - Subject Tax Free -->
        <record id="FA" model="account.tax.template">
            <field name="description">AAM</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="name">Fizetendő – Alanyi Adómentes</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">1040</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_fiz_alanyi')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_fiz_alanyi')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <!-- Payable - Material Tax Free -->
        <record id="FT" model="account.tax.template">
            <field name="description">TAM</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="name">Fizetendő – Tárgyi Adómentes</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">1050</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_fiz_targyi')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_fiz_targyi')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <!-- Excluding VAT service -->
        <record id="FKKS" model="account.tax.template">
            <field name="description">ÁKK</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="name">Áfa körön kívüli szolgáltatás</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">1060</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_fiz_koron_kivuli')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_fiz_koron_kivuli')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <!-- Sales of products outside the scope of VAT -->
        <record id="FKKT" model="account.tax.template">
            <field name="description">ÁKK</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="name">Áfa körön kívüli termék értékesítés</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">1070</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_fiz_koron_kivuli')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_fiz_koron_kivuli')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <!-- Payable - Reverse VAT -->
        <record id="FF" model="account.tax.template">
            <field name="description">FORD</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="name">Fizetendő – Fordított ÁFA</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">1080</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_forditott')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_forditott')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <!-- EU service provision, tax-free -->
        <record id="FEUSZ" model="account.tax.template">
            <field name="description">EU</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="name">EU szolgáltatás nyújtás, adómentes</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">1090</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_fiz_eu')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_fiz_eu')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <!-- EU supply of goods, tax-free -->
        <record id="FEUT" model="account.tax.template">
            <field name="description">EU</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="name">EU termékértékesítés, adómentes</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">1100</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_fiz_eu')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_fiz_eu')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <!-- Provision of export services -->
        <record id="FEXS" model="account.tax.template">
            <field name="description">Export</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="name">Export szolgáltatás nyújtás</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">1110</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_fiz_export')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_fiz_export')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <!-- Export supply of goods -->
        <record id="FEXT" model="account.tax.template">
            <field name="description">Export</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">sale</field>
            <field name="name">Export termékértékesítés</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">1120</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_fiz_export')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_fiz_export')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>
    <!-- Sale [END] -->

    <!-- Purchase -->
        <!-- Recoverable 27% -->
        <record id="V27" model="account.tax.template">
            <field name="description">27%</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">Visszaigényelhető 27%</field>
            <field name="amount_type">percent</field>
            <field name="amount">27</field>
            <field name="sequence">2010</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_27"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_viss_27')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_viss_27')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_alap_viss_27')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_viss_27')]
                }),
            ]"/>
        </record>

        <!-- Property, plant and equipment 27% -->
        <record id="V27TE" model="account.tax.template">
            <field name="description">27%</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">Tárgyi eszköz 27%</field>
            <field name="amount_type">percent</field>
            <field name="amount">27</field>
            <field name="sequence">2020</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_27"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_viss_27')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_viss_27')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_alap_viss_27')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_viss_27')]
                }),
            ]"/>
        </record>

        <!-- Recoverable 18% -->
        <record id="V18" model="account.tax.template">
            <field name="description">18%</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">Visszaigényelhető 18%</field>
            <field name="amount_type">percent</field>
            <field name="amount">18</field>
            <field name="sequence">2030</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_18"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_viss_18')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_viss_18')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_alap_viss_18')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_viss_18')]
                }),
            ]"/>
        </record>

        <!-- Recoverable 5% -->
        <record id="V5" model="account.tax.template">
            <field name="description">5%</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">Visszaigényelhető 5%</field>
            <field name="amount_type">percent</field>
            <field name="amount">5</field>
            <field name="sequence">2040</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_5"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_viss_5')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_viss_5')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_alap_viss_5')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_viss_5')]
                }),
            ]"/>
        </record>

        <!-- Compensation surcharge 7% -->
        <record id="VKOMP7" model="account.tax.template">
            <field name="description">Komp. 7%</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">Kompenzációs felár 7%</field>
            <field name="amount_type">percent</field>
            <field name="amount">7</field>
            <field name="sequence">2050</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_komp"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_komp')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_viss_komp')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_alap_komp')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_viss_komp')]
                }),
            ]"/>
        </record>

        <!-- Compensation surcharge 12% -->
        <record id="VKOMP12" model="account.tax.template">
            <field name="description">Komp. 12%</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">Kompenzációs felár 12%</field>
            <field name="amount_type">percent</field>
            <field name="amount">12</field>
            <field name="sequence">2060</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_komp"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_komp')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_viss_komp')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_alap_komp')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_viss_komp')]
                }),
            ]"/>
        </record>

        <!-- Reclaimable - Subject Tax Free -->
        <record id="VA" model="account.tax.template">
            <field name="description">AAM</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">Visszaigényelhető – Alanyi Adómentes</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">2070</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_viss_alanyi')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_viss_alanyi')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <!-- Reclaimable - Material Tax Free -->
        <record id="VT" model="account.tax.template">
            <field name="description">TAM</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">Visszaigényelhető – Tárgyi Adómentes</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">2080</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_viss_targyi')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_viss_targyi')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <!-- Excluding VAT service -->
        <record id="VKKS" model="account.tax.template">
            <field name="description">ÁKK</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">Áfa körön kívüli szolgáltatás</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">2090</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_viss_koron_kivuli')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_viss_koron_kivuli')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <!-- Purchase of products outside the scope of VAT -->
        <record id="VKKT" model="account.tax.template">
            <field name="description">ÁKK</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">Áfa körön kívüli termék beszerzés</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">2100</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_viss_koron_kivuli')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_viss_koron_kivuli')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <!-- Reverse VAT -->
        <record id="VF" model="account.tax.template">
            <field name="description">FORD</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">Fordított ÁFA</field>
            <field name="amount_type">percent</field>
            <field name="amount">27</field>
            <field name="sequence">2110</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_forditott')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_forditott')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <!-- EU product procurement 27% -->
        <record id="VEU27T" model="account.tax.template">
            <field name="description">EU</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">EU termék beszerzés 27%</field>
            <field name="amount_type">percent</field>
            <field name="amount">27</field>
            <field name="sequence">2120</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_27"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_viss')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_467'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_27')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_viss_27')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_alap_viss')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_467'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_27')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_viss_27')],
                }),
            ]"/>
        </record>

        <!-- EU service 27% -->
        <record id="VEU27S" model="account.tax.template">
            <field name="description">EU</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">EU szolgáltatás 27%</field>
            <field name="amount_type">percent</field>
            <field name="amount">27</field>
            <field name="sequence">2130</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_27"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_viss')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_467'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_27')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_viss_27')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_alap_viss')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_467'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_27')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_viss_27')],
                }),
            ]"/>
        </record>

        <!-- EU Property 27% -->
        <record id="VEU27TE" model="account.tax.template">
            <field name="description">EU</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">EU Tárgyi eszköz 27%</field>
            <field name="amount_type">percent</field>
            <field name="amount">27</field>
            <field name="sequence">2140</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_27"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_viss')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_467'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_27')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_viss_27')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_alap_viss')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_467'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_27')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_viss_27')],
                }),
            ]"/>
        </record>

        <!-- EU Tax Free Service -->
        <record id="VEUM" model="account.tax.template">
            <field name="description">EU</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">EU Adómentes szolgáltatás</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">2150</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_viss')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_viss')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>

        <!-- Import service 27% -->
        <record id="VIMS" model="account.tax.template">
            <field name="description">Import</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">Import szolgáltatás 27%</field>
            <field name="amount_type">percent</field>
            <field name="amount">27</field>
            <field name="sequence">2160</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_27"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_import')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_467'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_27')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_viss_27')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_alap_import')],
                }),
                (0,0, {
                    'factor_percent': -100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_467'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_27')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_viss_27')],
                }),
            ]"/>
        </record>

        <!-- Import sourcing 27% -->
        <record id="VIMK" model="account.tax.template">
            <field name="description">Import</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">Import beszerzés 27%</field>
            <field name="amount_type">percent</field>
            <field name="amount">27</field>
            <field name="sequence">2170</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_27"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_import')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'plus_report_line_ids': [ref('tax_report_fizetndo_viss_27')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('tax_report_alap_import')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('l10n_hu_466'),
                    'minus_report_line_ids': [ref('tax_report_fizetndo_viss_27')],
                }),
            ]"/>
        </record>

        <!-- Import duty free service -->
        <record id="VIM" model="account.tax.template">
            <field name="description">Import</field>
            <field name="chart_template_id" ref="hungarian_chart_template"/>
            <field name="type_tax_use">purchase</field>
            <field name="name">Import adómentes szolgáltatás</field>
            <field name="amount_type">percent</field>
            <field name="amount">0</field>
            <field name="sequence">2180</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_afa_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5,0,0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('tax_report_alap_import')],
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
                    'minus_report_line_ids': [ref('tax_report_alap_import')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                }),
            ]"/>
        </record>
    <!-- Purchase [END] -->
</odoo>

```

## File: data\l10n_hu_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="hungarian_chart_template" model="account.chart.template">
        <field name="name">Magyar főkönyvi kivonat</field>
        <field name="code_digits">4</field>
        <field name="cash_account_code_prefix">381</field>
        <field name="bank_account_code_prefix">384</field>
        <field name="transfer_account_code_prefix">389</field>
        <field name="currency_id" ref="base.HUF"/>
        <field name="country_id" ref="base.hu"/>
    </record>
</odoo>

```

## File: data\menuitem_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <menuitem id="hu_reports_menu" name="Hungary" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>
</odoo>

```

## File: data\res.bank.csv

```csv
"id","name","bic","country/id","zip","city","street","email","phone"
"BKCHHUHBXXX","Bank of China (Hungária) Hitelintézet Rt.","BKCHHUHBXXX","base.hu",1051,"Budapest","József Nádor tér 7.","service_hu@bank-of-china.com","+3614299200"
"BNPAHUHX","BNP Paribas Hungária Bank Rt.","BNPAHUHX","base.hu",1051,"Budapest","Széchenyi István  tér 7-8.",
"BUDAHUHB","Budapest Hitel- és Fejlesztési Bank Rt.","BUDAHUHB","base.hu",1138,"Budapest","Váci út 188."," info@budapestbank.hu","+3614506060"
"CIBHHUHB","CIB Közép-Európai Nemzetközi Bank Zrt.","CIBHHUHB","base.hu",1027,"Budapest","Medve utca 4-14.","cib@cib.hu","+3614231000"
"CITIHUHX","Citibank Rt.","CITIHUHX","base.hu",1051,"Budapest","Szabadság tér 7.",,"+3613745000"
"COBAHUHX","Commerzbank Zártkörűen Működő Rt.","COBAHUHX","base.hu",1054,"Budapest","Széchenyi  rakpart 8.","info.budapest@commerzbank.com","+3613748100"
"DEUTHU2B","Deutsche Bank Zártkörűen Működő Rt.","DEUTHU2B","base.hu",1054,"Budapest","Hold utca 27.","db.hungary@db.com","+3613013700"
"GIBAHUHB","ERSTE Bank Hungary Zrt.","GIBAHUHB","base.hu",1138,"Budapest","Népfürdő u. 24-26.","erste@erstebank.hu","+3640222221",
"FHKBHUHB","FHB Kereskedelmi Bank Zrt.","FHKBHUHB","base.hu",1082,"Budapest","Üllői út 48."," info@fhb.hu","+3614529100"
"GNBAHUHB","Gránit Bank Zrt.","GNBAHUHB","base.hu",1095,"Budapest","Lechner Ödön fasor 8.","info@granitbank.hu","+3640100777"
"INCNHUHB","IC Bank Rt.","INCNHUHB","base.hu",1088,"Budapest","Rákóczi  út 1-3."," level@bancopopolare.hu ","+3640200515",
"INGBHUHB","ING Bank N.V. Magyarországi Fióktelepe","INGBHUHB","base.hu",1068,"Budapest","Dózsa György  út 84.","communications.hu@ingbank.com","+362680140"
"OKHBHUHB","K&H Bank Zrt.","OKHBHUHB","base.hu",1095,"Budapest","Lechner Ödön fasor 9."," bank@kh.hu ","+3613289000"
"HBWEHUHB","MagNet Magyar Közösségi Bank Zrt.","HBWEHUHB","base.hu",1062,"Budapest","Andrássy utca 98.","info@magnetbank.hu","+3614288888"
"MANEHUHB","Magyar Nemzeti Bank","MANEHUHB","base.hu",1054,"Budapest","Szabadság tér 8-9.","info@mnb.hu","+3614282600"
"MKKBHUHB","MKB Bank Zrt.","MKKBHUHB","base.hu",1056,"Budapest","Váci utca 38.","telebankar@mkb.hu","+3613278600"
"OTPVHUHB","OTP Bank Nyrt.","OTPVHUHB","base.hu",1051,"Budapest","Nádor utca 16.",,"+3614735000"
"UBRTHUHB","RAIFFEISEN Bank Zrt.","UBRTHUHB","base.hu",1054,"Budapest","Akadémia utca 6.","+3640484848",
"MAVOHUHB","Sberbank Magyarország Zrt.","MAVOHUHB","base.hu",1088,"Budapest","Rákóczi  út 7.","info@sberbank.hu","+3614114200"
"TAKBHUHB","TakarékBank Zrt.","TAKBHUHB","base.hu",1122,"Budapest","Pethényi  köz 10.","info@tbank.hu"
"BACXHUHB","UniCredit Bank Hungary Zrt.","BACXHUHB","base.hu",1054,"Budapest","Szabadság tér 5-6.","info@unicreditgroup.hu","+3613011271"

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
      <image width="255" height="128" transform="translate(4.8 7.25) scale(0.2 0.25)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAP8AAAClCAYAAACTHStbAAAACXBIWXMAADf6AAA3+gH8300lAAACJElEQVR4Xu3WMW0DQRRF0Wy0krvACQYDMQ33xmEOBhAGgWAm3xCmn3tO/dorveN9+Z0vIOd7NQD2JH6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R58/tutoAGzpmZlYjYD9uP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ9R5fz5WG2BD5+v/b7UBNuT2Q5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RB0zs9oAG/oAROcPClSjRlIAAAAASUVORK5CYII="/>
    </g>
  </g>
</svg>

```

