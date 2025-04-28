# Odoo Module: l10n_hu

Category: Localization

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

{
    'name': 'Hungarian - Accounting',
    'version': '2.0',
    'category': 'Localization',
    'description': """

Base module for Hungarian localization
==========================================

This module consists of:

 - Generic Hungarian chart of accounts
 - Hungarian taxes
 - Hungarian Bank information
 """,
    'author': 'InnOpen Group Kft',
    'website': 'http://www.innopen.eu',
    'depends': ['account'],
    'data': [
        'data/l10n_hu_chart_data.xml',
        'data/account.account.template.csv',
        'data/account.tax.group.csv',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/account.fiscal.position.template.csv',
        'data/account.fiscal.position.tax.template.csv',
        'data/res.bank.csv',
        'data/account_chart_template_data.xml',
        'data/account_chart_template_configure_data.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","code","name","user_type_id/id","reconcile",chart_template_id/id
"chart_hu_111",111,"Alapítás-átszervezés aktívált értéke","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_112",112,"Kísérleti fejlesztés aktívált értéke","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_113",113,"Vagyoni értékû jogok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_114",114,"Szellemi termékek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_115",115,"Üzleti vagy cégérték","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_117",117,"Immateriális javak értékhelyesbítése","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_121",121,"Földterület","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_122",122,"Telek, telkesítés","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_123",123,"Épületek,tulajdoni hányadok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_124",124,"Egyéb építmények","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_125",125,"Üzemkörön kivüli ingatlanok, épületek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_126",126,"Ingatlanhoz kapcs. vagyoni ért. jogok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_127",127,"Ingatlanok értékhelyesbítése","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_131",131,"Termelõ gépek, gyártóeszk.","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_132",132,"Termelésben résztvevõ jármûvek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_137",137,"Müszaki gépek,járm. értékhelyesb.","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_141",141,"Üzemi berendezések,felszerelések","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_142",142,"Egyéb jármûvek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_143",143,"Irodai, igazgatási berendezések","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_144",144,"Üzemkörön kivüli berendezések, felsz.","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_147",147,"Egyéb gépek,járm. értékhelyesbítés","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_151",151,"Tenyészállatok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_161",161,"Befejezetlen beruházások","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_162",162,"Felújítások","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_168",168,"Beruházások terven felüli értékcsökk.","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_171",171,"Tartós részesedés kapcs. vállalkozásban","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_172",172,"Egyéb tartós részesedés","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_177",177,"Részesedések értékhelyesbítése","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_179",179,"Részesedések értékvesztése, visszaírása","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_181",181,"Államkötvények","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_182",182,"Kapcsolt vállalkozások értékpapírjai","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_183",183,"Egyéb vállalkozások értékpapírjai","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_184",184,"Tartós diszkont értékpapírok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_189",189,"Értékpapírok értékvesztése, visszaírása","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_191",191,"Tartósan adott kölcsönök kapcs. váll.","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_192",192,"Tartósan adott kölcsön egyéb rész.váll.","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_193",193,"Egyéb tartósan adott kölcsönök","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_195",195,"Tartós bankbetétek kapcs. váll.-ban","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_196",196,"Tartós bankbetétek egyéb rész. váll.-ban","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_197",197,"Egyéb tartós bankbetétek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_198",198,"Pénzügyi lízing miatti tartós követelés","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_199",199,"Tartósan adott kölcsönök értékvesztése","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_211",211,"Nyers- és alapanyagok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_221",221,"Segédanyagok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_231",231,"Befejezetlen termelés","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_251",251,"Késztermékek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_261",261,"Áruk beszerzési áron","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_271",271,"Közvetített szolgáltatások","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_281",281,"Betétdíjas göngyölegek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_311",311,"Belföldi követelések","account.data_account_type_receivable","TRUE",hungarian_chart_template
"chart_hu_312",312,"Belföldi követelések (PoS)","account.data_account_type_receivable","TRUE",hungarian_chart_template
"chart_hu_316",316,"Külföldi követelések","account.data_account_type_receivable","TRUE",hungarian_chart_template
"chart_hu_351",351,"Adott elõlegek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_361",361,"Munkavállalókkal szembeni követelés","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_368",368,"Különféle egyéb követelések","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_371",371,"Részesedés kapcsolt vállalkozásban","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_372",372,"Egyéb részesedés","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_373",373,"Saját részvények, saját üzletrészek","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_374",374,"Forgatási célú hitelv. m. értékpapírok","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_382",382,"Valuta pénztár","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_385",385,"Elkülönített betétszámlák","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_386",386,"Deviza betétszámla","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_387",387,"Pénzhelyettesítõ eszk. (utalvány, jegy)","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_391",391,"Aktív idõbeli elhatárolása","account.data_account_type_current_assets","FALSE",hungarian_chart_template
"chart_hu_411",411,"Jegyzett tõke","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_412",412,"Tõketartalék","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_413",413,"Eredménytartalék","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_414",414,"Lekötött tartalék","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_417",417,"Értékelési tartalék","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_419",419,"Mérleg szerinti eredmény","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_421",421,"Céltartalék várható kötelezettségre","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_431",431,"Hátrasorolt kötelezettség","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_441",441,"Hosszú lejáratra kapott kölcsönök","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_442",442,"Átváltoztatható kötvények","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_443",443,"Tartozások kötvénykibocsátásból","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_444",444,"Beruházási és fejlesztési hitelek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_445",445,"Egyéb hosszú lejáratú hitelek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_446",446,"Tartós köt. kapcs. vállalkozással sz.","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_447",447,"Tartós köt. egyéb rész. váll. szemben","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_448",448,"Pénzügyi lízinggel kapcsolatos kötelez.","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_449",449,"Egyéb hosszú lej. kötelezettségek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_451",451,"Rövid lejáratú kölcsönök","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_452",452,"Rövid lejáratú hitelek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_453",453,"Vevõktõl kapott elõlegek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_4541",4541,"Belföldi szállítók","account.data_account_type_payable","TRUE",hungarian_chart_template
"chart_hu_4542",4542,"Külföldi szállítók","account.data_account_type_payable","TRUE",hungarian_chart_template
"chart_hu_461",461,"Társasági adó és osztalékadó elszámolás","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_462",462,"Személyi jövedelemadó elszámolása","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_463",463,"Költségvetési befizetési kötelezettségek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_464",464,"Költségvetési befizetési köt.teljesítése","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_465",465,"Vám- és Pénzügyõrség elszámolási számla","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_466",466,"Elõzetesen felszámított ált.forgalmi adó","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_467",467,"Fizetendõ általános forgalmi adó","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_468",468,"Áfa pénzügyi elszámolási számla","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_469",469,"Önkormányzati adók elszámolási számla","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_471",471,"Jövedelem elszámolási számla","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_472",472,"Fel nem vett járandóságok","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_473",473,"Társadalombiztosítási kötelezettség","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_474",474,"Elkülönített alapokkal kapcs. fiz. köt","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_475",475,"Magánnyugdíjpénztárak befiz. kötelezetts","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_476",476,"Egyéb rövid lej.kötelezettség munkaváll.","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_477",477,"Egyéb rövid lejáratú kötelezettség","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_478",478,"Magánszemélytõl levont 4% különadó","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_479",479,"Egyéb befizetési kötelezettségek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_481",481,"Bevételek passzív idõbeli elhatárolása","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_482",482,"Költségek,ráford. passzív idõbeli elhat.","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_483",483,"Halasztott bevételek","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_491",491,"Nyitómérleg számla","account.data_account_type_current_liabilities","FALSE",hungarian_chart_template
"chart_hu_511",511,"Vásárolt anyagok költségei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_512",512,"Egy éven belül elhaszn. anyagi eszközök","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_513",513,"Egyéb anyagköltség","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_519",519,"Anyagköltség megtérülés","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_521",521,"Szállítási, rakodási költség","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_522",522,"Bérleti díjak","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_523",523,"Javítási, karbantartási költségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_524",524,"Hirdetés, reklám-propaganda költség","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_525",525,"Oktatási, továbbképzési költségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_526",526,"Utazási- és kiküldetési költségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_527",527,"Postai, távközlési költségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_528",528,"Szakkönyv, napilap beszerzés","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_529",529,"Egyéb igénybevett szolgáltatások ktg-ei","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_531",531,"Hatósági igazgatási díjak (illetékek)","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_532",532,"Pénzügyi szolg-i díjak, bankköltségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_533",533,"Biztosítási díjak","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_541",541,"Munkavállalók munkabér költsége","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_542",542,"Megbízási díjak bérköltség terhére","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_543",543,"Tagok személyes közr. ellenértéke","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_544",544,"Egyszerûsített fogl. bérköltsége","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_551",551,"Személyi jellegû kifizetések","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_552",552,"Jóléti és kulturális költségek","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_553",553,"Természetbeni juttatások","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_554",554,"Egyéb személyi jellegû kifizetések","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_555",555,"Magánnyugdíjpénztári tagdíjak, hozzájár.","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_556",556,"Foglalkoztatót terhelõ táppénz hjárulás","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_557",557,"Kifizetõt terhelõ személyi jövedelemadó","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_561",561,"Társadalombiztosítási járulék","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_562",562,"Egészségügyi hozzájárulás","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_563",563,"Munkaadói járulék","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_564",564,"Szakképzési hozzájárulás","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_565",565,"Rehabilitációs hozzájárulás","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_566",566,"Egyszerûsített közteherviselési hjár","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_567",567,"Egyszerûsített fogl. közteher","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_568",568,"Közteherjegy","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_571",571,"Terv szerinti értékcsökkenés lineáris","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_572",572,"Terv szerinti egyösszegû (kisértékûek)","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_581",581,"Saját term. készletek állományváltozása","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_582",582,"Saját elõállítási eszközök aktivált ért.","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_591",591,"Anyagköltség átvezetési szla","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_592",592,"Igénybevett szolg. átvezetési szla","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_593",593,"Egyéb szolgáltatások átvezetési szla","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_594",594,"Bérköltség átvezetési szla","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_595",595,"Személyi jell. kif. átvezetési szla","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_596",596,"Bérjárulékok átvezetési szla","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_597",597,"Értékcsökkenési leírás átvez. szla","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_81",81,"ANYAGJELLEGÛ RÁFORDÍTÁSOK","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_811",811,"Anyagköltség","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_812",812,"Igénybevett szolgáltatások értéke","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_813",813,"Egyéb szolgáltatások értéke","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_814",814,"Eladott áruk beszerzési értéke","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_815",815,"Eladott (közvetített) szolg. értéke","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_821",821,"Bérköltség","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_822",822,"Személyi jellegü egyéb kifizetések","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_823",823,"Bérjárulékok","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_861",861,"Értékesített eszk.imm.javak nytsz értéke","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_862",862,"Ért.átruházott követelések könyvsz. ért.","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_863",863,"Az üzleti évhez kapcs. ráfordítások","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_864",864,"Utólag adott pü. rendezett engedmény","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_865",865,"Céltartalék képzése","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_866",866,"Elszámolt értékvesztés, tervenf. értékcs","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_867",867,"Adók, hozzájárulások","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_869",869,"Különféle egyéb ráfordítások","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_871",871,"Befektetett püi. eszk. árf.vesztesége","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_872",872,"Fizetendõ kamatok, kamatjell. ráford.","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_874",874,"Részesedések,bankb. értékveszt","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_875",875,"Forgóeszk. értékpapír árf.vesztesége","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_876",876,"Átváltási, értékelési árfolyamveszteség","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_877",877,"Egyéb árfolyamveszteségek, opciós díjak","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_878",878,"Vásárolt köv. kapcs. ráfordítások","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_879",879,"Egyéb pénzügyi ráfordítások","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_891",891,"Társasági adó","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_892",892,"Társas vállalkozás különadója","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_895",895,"Egyszerüsített vállalkozói adó","account.data_account_type_expenses","FALSE",hungarian_chart_template
"chart_hu_911",911,"Belföldi értékesítés árbevétele","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_921",921,"Belföldi értékesítés árbevétele","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_931",931,"Export értékesítés árbev. EU tagországba","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_932",932,"Export értékesítés árbev.nem EU tagorsz.","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_961",961,"Ért.immat. javak, tárgyi eszk.bevétele","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_962",962,"Ért,átruházott követelések elism.mértéke","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_963",963,"Az üzleti évhez kapcs. egyéb bevételek","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_964",964,"Utólag kapott pü. rendezett engedmény","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_965",965,"Céltartalék felhasználása","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_966",966,"Értékvesztések visszaírása, tervenf.écs.","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_967",967,"Visszafiz. köt. nélkül kapott támogatás","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_968",968,"Biztosító által visszaig. kártérítés ö.","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_969",969,"Különféle egyéb bevételek","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_971",971,"Kapott (járó) osztalék, részesedés","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_972",972,"Részesedések ért. árfolyamnyeresége","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_973",973,"Befekt. püi.eszk. kamatai, árf.nyeres.","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_974",974,"Egyéb kapott kamatok,kamatjell.bevételek","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_975",975,"Forgóeszk. értékpapír árfolyamnyeresége","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_976",976,"Átváltási, átértékeléskori árf.nyereség","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_977",977,"Egyéb árfolyamnyereségek, opciós bev.","account.data_account_type_other_income","FALSE",hungarian_chart_template
"chart_hu_978",978,"Vás. követelésekkel kapcs. bevételek","account.data_account_type_revenue","FALSE",hungarian_chart_template
"chart_hu_979",979,"Egyéb pénzügyi mûveletek bevételei","account.data_account_type_revenue","FALSE",hungarian_chart_template

```

## File: data\account.financial.report.csv

```csv
"id","display_detail","style_overwrite","account_type_ids/id","account_ids/id","parent_id/id","sign","name","account_report_id/id","sequence","type"
"l10n_hu.account_financial_report_pl_hu","Display children flat","Main Title 1 (bold, underlined)",,,,"Reverse balance sign","Eredménykimutatás – HU",,30,"View"
"l10n_hu.account_financial_report_pl_hu_G","Display children flat","Main Title 1 (bold, underlined)",,,"l10n_hu.account_financial_report_pl_hu","Reverse balance sign","Mérleg szerinti eredmény",,10,"View"
"l10n_hu.account_financial_report_pl_hu_F","Display children flat","Main Title 1 (bold, underlined)",,,"l10n_hu.account_financial_report_pl_hu_G","Reverse balance sign","Adózott eredmény",,12,"View"
"l10n_hu.account_financial_report_pl_hu_XIII","Display children with hierarchy","Normal Text","account_type_pl_hu_XIII",,"l10n_hu.account_financial_report_pl_hu_G","Reverse balance sign","Osztalék",,11,"Account Type"
"l10n_hu.account_financial_report_pl_hu_E","Display children flat","Main Title 1 (bold, underlined)",,,"l10n_hu.account_financial_report_pl_hu_F","Reverse balance sign","Adózás előtti eredmény",,14,"View"
"l10n_hu.account_financial_report_pl_hu_XII","Display children with hierarchy","Normal Text","account_type_pl_hu_XII",,"l10n_hu.account_financial_report_pl_hu_F","Reverse balance sign","Adófizetési kötelezettség",,13,"Account Type"
"l10n_hu.account_financial_report_pl_hu_D","Display children flat","Title 2 (bold)",,,"l10n_hu.account_financial_report_pl_hu_E","Reverse balance sign","Rendkívüli eredmény",,15,"View"
"l10n_hu.account_financial_report_pl_hu_X","Display children with hierarchy","Normal Text","account_type_pl_hu_X",,"l10n_hu.account_financial_report_pl_hu_D","Reverse balance sign","Rendkívüli bevételek",,17,"Account Type"
"l10n_hu.account_financial_report_pl_hu_XI","Display children with hierarchy","Normal Text","account_type_pl_hu_XI",,"l10n_hu.account_financial_report_pl_hu_D","Reverse balance sign","Rendkivüli ráfordítások",,16,"Account Type"
"l10n_hu.account_financial_report_pl_hu_C","Display children flat","Main Title 1 (bold, underlined)",,,"l10n_hu.account_financial_report_pl_hu_E","Reverse balance sign","Szokásos vállalkozási eredmény",,18,"View"
"l10n_hu.account_financial_report_pl_hu_A","Display children flat","Main Title 1 (bold, underlined)",,,"l10n_hu.account_financial_report_pl_hu_C","Reverse balance sign","Üzemi (üzleti) tevékenység eredménye",,22,"View"
"l10n_hu.account_financial_report_pl_hu_B","Display children flat","Title 2 (bold)",,,"l10n_hu.account_financial_report_pl_hu_C","Reverse balance sign","Pénzügyi műveletek eredménye",,19,"View"
"l10n_hu.account_financial_report_pl_hu_VIII","Display children with hierarchy","Normal Text","account_type_pl_hu_VIII",,"l10n_hu.account_financial_report_pl_hu_B","Reverse balance sign","Pénzügyi műveletek bevételei",,21,"Account Type"
"l10n_hu.account_financial_report_pl_hu_IX","Display children with hierarchy","Normal Text","account_type_pl_hu_IX",,"l10n_hu.account_financial_report_pl_hu_B","Reverse balance sign","Pénzügyi műveletek ráfordításai ",,20,"Account Type"
"l10n_hu.account_financial_report_pl_hu_I","Display children with hierarchy","Normal Text","account_type_pl_hu_I",,"l10n_hu.account_financial_report_pl_hu_A","Reverse balance sign","Értékesítés nettó árbevétele ",,1,"Account Type"
"l10n_hu.account_financial_report_pl_hu_II","Display children with hierarchy","Normal Text","account_type_pl_hu_II",,"l10n_hu.account_financial_report_pl_hu_A","Reverse balance sign","Aktivált saját teljesítmények értéke ",,2,"Account Type"
"l10n_hu.account_financial_report_pl_hu_III","Display children with hierarchy","Normal Text","account_type_pl_hu_III",,"l10n_hu.account_financial_report_pl_hu_A","Reverse balance sign","Egyéb bevételek",,3,"Account Type"
"l10n_hu.account_financial_report_pl_hu_IV","Display children with hierarchy","Normal Text","account_type_pl_hu_IV",,"l10n_hu.account_financial_report_pl_hu_A","Reverse balance sign","Anyagjellegű ráfordítások ",,4,"Account Type"
"l10n_hu.account_financial_report_pl_hu_V","Display children with hierarchy","Normal Text","account_type_pl_hu_V",,"l10n_hu.account_financial_report_pl_hu_A","Reverse balance sign","Személyi jellegű ráfordítások ",,5,"Account Type"
"l10n_hu.account_financial_report_pl_hu_VI","Display children with hierarchy","Normal Text","account_type_pl_hu_VI",,"l10n_hu.account_financial_report_pl_hu_A","Reverse balance sign","Értékcsökkenési leírás",,6,"Account Type"
"l10n_hu.account_financial_report_pl_hu_VII","Display children with hierarchy","Normal Text","account_type_pl_hu_VII",,"l10n_hu.account_financial_report_pl_hu_A","Reverse balance sign","Egyéb ráfordítások",,7,"Account Type"
"l10n_hu.account_financial_report_bs_hu","Display children flat",,,,,"Preserve balance sign","Mérleg – HU",,30,"View"
"l10n_hu.account_financial_report_bs_hu_AC","Display children with hierarchy","Main Title 1 (bold, underlined)",,,"l10n_hu.account_financial_report_bs_hu","Preserve balance sign","Eszközök Összesen",,100,"View"
"l10n_hu.account_financial_report_bs_hu_DG","Display children with hierarchy","Main Title 1 (bold, underlined)",,,"l10n_hu.account_financial_report_bs_hu","Reverse balance sign","Források Összesen",,200,"View"
"l10n_hu.account_financial_report_bs_hu_A","Display children flat","Title 2 (bold)","account_type_bs_hu_A",,"l10n_hu.account_financial_report_bs_hu_AC","Preserve balance sign","Befektetett Eszközök",,110,"View"
"l10n_hu.account_financial_report_bs_hu_AI","Display children flat","Normal Text","account_type_bs_hu_AI",,"l10n_hu.account_financial_report_bs_hu_A","Preserve balance sign","Immateriális Javak",,10,"Account Type"
"l10n_hu.account_financial_report_bs_hu_AII","Display children flat","Normal Text","account_type_bs_hu_AII",,"l10n_hu.account_financial_report_bs_hu_A","Preserve balance sign","Tárgyi Eszközök",,10,"Account Type"
"l10n_hu.account_financial_report_bs_hu_AIII","Display children flat","Normal Text","account_type_bs_hu_AIII",,"l10n_hu.account_financial_report_bs_hu_A","Preserve balance sign","Befektetett Pénzügyi Eszközök",,10,"Account Type"
"l10n_hu.account_financial_report_bs_hu_B","Display children with hierarchy","Title 2 (bold)","account_type_bs_hu_B",,"l10n_hu.account_financial_report_bs_hu_AC","Preserve balance sign","Forgóeszközök",,120,"View"
"l10n_hu.account_financial_report_bs_hu_BI","Display children flat","Normal Text","account_type_bs_hu_BI",,"l10n_hu.account_financial_report_bs_hu_B","Preserve balance sign","Készletek",,10,"Account Type"
"l10n_hu.account_financial_report_bs_hu_BII","Display children flat","Normal Text","account_type_bs_hu_BII",,"l10n_hu.account_financial_report_bs_hu_B","Preserve balance sign","Követelések",,10,"Account Type"
"l10n_hu.account_financial_report_bs_hu_BIII","Display children flat","Normal Text","account_type_bs_hu_BIII",,"l10n_hu.account_financial_report_bs_hu_B","Preserve balance sign","Értékpapírok",,10,"Account Type"
"l10n_hu.account_financial_report_bs_hu_BIV","Display children flat","Normal Text","account_type_bs_hu_BIV",,"l10n_hu.account_financial_report_bs_hu_B","Preserve balance sign","Pénzeszközök",,10,"Account Type"
"l10n_hu.account_financial_report_bs_hu_C","Display children with hierarchy","Title 2 (bold)","account_type_bs_hu_C",,"l10n_hu.account_financial_report_bs_hu_AC","Preserve balance sign","Aktív Időbeli Elhatárolások",,130,"Account Type"
"l10n_hu.account_financial_report_bs_hu_pl","Display children flat","Title 2 (bold)",,,"l10n_hu.account_financial_report_bs_hu_DG","Reverse balance sign","Számított eredmény","l10n_hu.account_financial_report_pl_hu",201,"Report Value"
"l10n_hu.account_financial_report_bs_hu_D","Display children with hierarchy","Title 2 (bold)","account_type_bs_hu_D",,"l10n_hu.account_financial_report_bs_hu_DG","Reverse balance sign","Saját Tőke",,210,"Account Type"
"l10n_hu.account_financial_report_bs_hu_E","Display children with hierarchy","Title 2 (bold)","account_type_bs_hu_E",,"l10n_hu.account_financial_report_bs_hu_DG","Reverse balance sign","Céltartalékok",,220,"Account Type"
"l10n_hu.account_financial_report_bs_hu_F","Display children with hierarchy","Title 2 (bold)","account_type_bs_hu_F",,"l10n_hu.account_financial_report_bs_hu_DG","Reverse balance sign","Kötelezettségek",,230,"View"
"l10n_hu.account_financial_report_bs_hu_FI","Display children with hierarchy","Normal Text","account_type_bs_hu_FI",,"l10n_hu.account_financial_report_bs_hu_F","Reverse balance sign","Hátrasorolt Kötelezettségek",,10,"Account Type"
"l10n_hu.account_financial_report_bs_hu_FII","Display children with hierarchy","Normal Text","account_type_bs_hu_FII",,"l10n_hu.account_financial_report_bs_hu_F","Reverse balance sign","Hosszú Lejáratú Kötelezettségek",,10,"Account Type"
"l10n_hu.account_financial_report_bs_hu_FIII","Display children with hierarchy","Normal Text","account_type_bs_hu_FIII",,"l10n_hu.account_financial_report_bs_hu_F","Reverse balance sign","Rövid Lejáratú Kötelezettségek",,10,"Account Type"
"l10n_hu.account_financial_report_bs_hu_G","Display children with hierarchy","Title 2 (bold)","account_type_bs_hu_G",,"l10n_hu.account_financial_report_bs_hu_DG","Reverse balance sign","Passzív Időbeli Elhatárolások",,240,"Account Type"

```

## File: data\account.fiscal.position.tax.template.csv

```csv
"id","tax_src_id/id","tax_dest_id/id","position_id/id"
"fiscal_position_hu_exempt_tax_F27","F27","FA","fiscal_position_hu_exempt"
"fiscal_position_hu_exempt_tax_F18","F18","FA","fiscal_position_hu_exempt"
"fiscal_position_hu_exempt_tax_F5","F5","FA","fiscal_position_hu_exempt"
"fiscal_position_hu_exempt_tax_V27","V27","VA","fiscal_position_hu_exempt"
"fiscal_position_hu_exempt_tax_V18","V18","VA","fiscal_position_hu_exempt"
"fiscal_position_hu_exempt_tax_V5","V5","VA","fiscal_position_hu_exempt"
"fiscal_position_hu_eu_tax_F27","F27","FEU","fiscal_position_hu_eu"
"fiscal_position_hu_eu_tax_F18","F18","FEU","fiscal_position_hu_eu"
"fiscal_position_hu_eu_tax_F5","F5","FEU","fiscal_position_hu_eu"
"fiscal_position_hu_eu_tax_FA","FA","FEU","fiscal_position_hu_eu"
"fiscal_position_hu_eu_tax_V27","V27","VEU","fiscal_position_hu_eu"
"fiscal_position_hu_eu_tax_V18","V18","VEU","fiscal_position_hu_eu"
"fiscal_position_hu_eu_tax_V5","V5","VEU","fiscal_position_hu_eu"
"fiscal_position_hu_eu_tax_VA","VA","VEU","fiscal_position_hu_eu"
"fiscal_position_hu_eu_out_tax_F27","F27","FEUO","fiscal_position_hu_eu_out"
"fiscal_position_hu_eu_out_tax_F18","F18","FEUO","fiscal_position_hu_eu_out"
"fiscal_position_hu_eu_out_tax_F5","F5","FEUO","fiscal_position_hu_eu_out"
"fiscal_position_hu_eu_out_tax_FA","FA","FEUO","fiscal_position_hu_eu_out"
"fiscal_position_hu_eu_out_tax_V27","V27","VEUO","fiscal_position_hu_eu_out"
"fiscal_position_hu_eu_out_tax_V18","V18","VEUO","fiscal_position_hu_eu_out"
"fiscal_position_hu_eu_out_tax_V5","V5","VEUO","fiscal_position_hu_eu_out"
"fiscal_position_hu_eu_out_tax_VA","VA","VEUO","fiscal_position_hu_eu_out"

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

## File: data\account.tax.group.csv

```csv
id,name
tax_group_afa_0,áfa 0%
tax_group_afa_5,áfa 5%
tax_group_afa_18,áfa 18%
tax_group_afa_27,áfa 27%

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
    <!-- Chart Template -->

    <record id="hungarian_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="chart_hu_311"/>
        <field name="property_account_payable_id" ref="chart_hu_4541"/>
        <field name="property_account_expense_id" ref="chart_hu_81"/>
        <field name="property_account_income_id" ref="chart_hu_911"/>
        <field name="property_account_expense_categ_id" ref="chart_hu_81"/>
        <field name="property_account_income_categ_id" ref="chart_hu_911"/>
        <field name="income_currency_exchange_account_id" ref="chart_hu_977"/>
        <field name="expense_currency_exchange_account_id" ref="chart_hu_876"/>
        <field name="default_pos_receivable_account_id" ref="chart_hu_312" />
    </record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report_alap" model="account.tax.report.line">
        <field name="name">ÁFA alap</field>
        <field name="sequence" eval="1"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_fiz_export" model="account.tax.report.line">
        <field name="name">Adóalap - Fizetendő ÁFA Export</field>
        <field name="tag_name">Adóalap - Fizetendő ÁFA Export</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_fiz_eu" model="account.tax.report.line">
        <field name="name">Adóalap - Fizetendő ÁFA EU</field>
        <field name="tag_name">Adóalap - Fizetendő ÁFA EU</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_fiz_targyi" model="account.tax.report.line">
        <field name="name">Adóalap - Fizetendő ÁFA tárgyi adómentes</field>
        <field name="tag_name">Adóalap - Fizetendő ÁFA tárgyi adómentes</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_fiz_alanyi" model="account.tax.report.line">
        <field name="name">Adóalap - Fizetendő ÁFA alanyi adómentes</field>
        <field name="tag_name">Adóalap - Fizetendő ÁFA alanyi adómentes</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_fiz_afa_5" model="account.tax.report.line">
        <field name="name">Adóalap - Fizetendő ÁFA 5%</field>
        <field name="tag_name">Adóalap - Fizetendő ÁFA 5%</field>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_fiz_afa_18" model="account.tax.report.line">
        <field name="name">Adóalap - Fizetendő ÁFA 18%</field>
        <field name="tag_name">Adóalap - Fizetendő ÁFA 18%</field>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_fiz_afa_27" model="account.tax.report.line">
        <field name="name">Adóalap - Fizetendő ÁFA 27%</field>
        <field name="tag_name">Adóalap - Fizetendő ÁFA 27%</field>
        <field name="sequence" eval="7"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_viss" model="account.tax.report.line">
        <field name="name">Adóalap - Visszaigényelhető ÁFA EU</field>
        <field name="tag_name">Adóalap - Visszaigényelhető ÁFA EU</field>
        <field name="sequence" eval="8"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_import" model="account.tax.report.line">
        <field name="name">Adóalap – Import ÁFA</field>
        <field name="tag_name">Adóalap – Import ÁFA</field>
        <field name="sequence" eval="9"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_forditott" model="account.tax.report.line">
        <field name="name">Adóalap – Fordított ÁFA</field>
        <field name="tag_name">Adóalap – Fordított ÁFA</field>
        <field name="sequence" eval="10"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_viss_alanyi" model="account.tax.report.line">
        <field name="name">Adóalap - Visszaigényelhető ÁFA alanyi adómentes</field>
        <field name="tag_name">Adóalap - Visszaigényelhető ÁFA alanyi adómentes</field>
        <field name="sequence" eval="11"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_viss_targyi" model="account.tax.report.line">
        <field name="name">Adóalap - Visszaigényelhető ÁFA tárgyi adómentes</field>
        <field name="tag_name">Adóalap - Visszaigényelhető ÁFA tárgyi adómentes</field>
        <field name="sequence" eval="12"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_viss_5" model="account.tax.report.line">
        <field name="name">Adóalap - Visszaigényelhető ÁFA 5%</field>
        <field name="tag_name">Adóalap - Visszaigényelhető ÁFA 5%</field>
        <field name="sequence" eval="13"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_viss_18" model="account.tax.report.line">
        <field name="name">Adóalap - Visszaigényelhető ÁFA 18%</field>
        <field name="tag_name">Adóalap - Visszaigényelhető ÁFA 18%</field>
        <field name="sequence" eval="14"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_alap_viss_27" model="account.tax.report.line">
        <field name="name">Adóalap - Visszaigényelhető ÁFA 27%</field>
        <field name="tag_name">Adóalap - Visszaigényelhető ÁFA 27%</field>
        <field name="sequence" eval="15"/>
        <field name="parent_id" ref="tax_report_alap"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_fizetndo" model="account.tax.report.line">
        <field name="name">ÁFA fizetndő / visszaigényelhető</field>
        <field name="sequence" eval="2"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_fizetndo_5" model="account.tax.report.line">
        <field name="name">Fizetendő ÁFA 5%</field>
        <field name="tag_name">Fizetendő ÁFA 5%</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_fizetndo"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_fizetndo_18" model="account.tax.report.line">
        <field name="name">Fizetendő ÁFA 18%</field>
        <field name="tag_name">Fizetendő ÁFA 18%</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_fizetndo"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_fizetndo_27" model="account.tax.report.line">
        <field name="name">Fizetendő ÁFA 27%</field>
        <field name="tag_name">Fizetendő ÁFA 27%</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_fizetndo"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_fizetndo_viss_5" model="account.tax.report.line">
        <field name="name">Visszaigényelhető ÁFA 5%</field>
        <field name="tag_name">Visszaigényelhető ÁFA 5%</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="tax_report_fizetndo"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_fizetndo_viss_18" model="account.tax.report.line">
        <field name="name">Visszaigényelhető ÁFA 18%</field>
        <field name="tag_name">Visszaigényelhető ÁFA 18%</field>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="tax_report_fizetndo"/>
        <field name="country_id" ref="base.hu"/>
    </record>

    <record id="tax_report_fizetndo_viss_27" model="account.tax.report.line">
        <field name="name">Visszaigényelhető ÁFA 27%</field>
        <field name="tag_name">Visszaigényelhető ÁFA 27%</field>
        <field name="sequence" eval="6"/>
        <field name="parent_id" ref="tax_report_fizetndo"/>
        <field name="country_id" ref="base.hu"/>
    </record>

</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="F27" model="account.tax.template">
        <field name="description">27%</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Fizetendő - 27%</field>
        <field name="amount_type">percent</field>
        <field name="amount">27</field>
        <field name="sequence">1</field>
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
                'account_id': ref('chart_hu_467'),
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
                'account_id': ref('chart_hu_467'),
                'minus_report_line_ids': [ref('tax_report_fizetndo_27')],
            }),
        ]"/>
    </record>

    <record id="F18" model="account.tax.template">
        <field name="description">18%</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Fizetendő – 18%</field>
        <field name="amount_type">percent</field>
        <field name="amount">18</field>
        <field name="sequence">2</field>
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
                'account_id': ref('chart_hu_467'),
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
                'account_id': ref('chart_hu_467'),
                'minus_report_line_ids': [ref('tax_report_fizetndo_18')],
            }),
        ]"/>
    </record>

    <record id="F5" model="account.tax.template">
        <field name="description">5%</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Fizetendő – 5%</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="sequence">2</field>
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
                'account_id': ref('chart_hu_467'),
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
                'account_id': ref('chart_hu_467'),
                'minus_report_line_ids': [ref('tax_report_fizetndo_5')],
            }),
        ]"/>
    </record>

    <record id="FA" model="account.tax.template">
        <field name="description">AAM</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Fizetendő – Alanyi Adómentes</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="sequence">2</field>
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

    <record id="FT" model="account.tax.template">
        <field name="description">TAM</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Fizetendő – Tárgyi Adómentes</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="sequence">2</field>
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

    <record id="FF" model="account.tax.template">
        <field name="description">FORD</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Fizetendő – Fordított ÁFA</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="sequence">2</field>
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

    <record id="FEUO" model="account.tax.template">
        <field name="description">Export</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Fizetendő – Export</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="sequence">2</field>
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

    <record id="FEU" model="account.tax.template">
        <field name="description">EU</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Fizetendő – EU</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="sequence">2</field>
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

    <record id="V18" model="account.tax.template">
        <field name="description">18%</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Visszaigényelhető – 18%</field>
        <field name="amount_type">percent</field>
        <field name="amount">18</field>
        <field name="sequence">2</field>
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
                'account_id': ref('chart_hu_466'),
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
                'account_id': ref('chart_hu_466'),
                'minus_report_line_ids': [ref('tax_report_fizetndo_viss_18')]
            }),
        ]"/>
    </record>

    <record id="V27" model="account.tax.template">
        <field name="description">27%</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Visszaigényelhető – 27%</field>
        <field name="amount_type">percent</field>
        <field name="amount">27</field>
        <field name="sequence">1</field>
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
                'account_id': ref('chart_hu_466'),
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
                'account_id': ref('chart_hu_466'),
                'minus_report_line_ids': [ref('tax_report_fizetndo_viss_27')]
            }),
        ]"/>
    </record>

    <record id="V5" model="account.tax.template">
        <field name="description">5%</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Visszaigényelhető – 5%</field>
        <field name="amount_type">percent</field>
        <field name="amount">5</field>
        <field name="sequence">2</field>
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
                'account_id': ref('chart_hu_466'),
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
                'account_id': ref('chart_hu_466'),
                'minus_report_line_ids': [ref('tax_report_fizetndo_viss_5')]
            }),
        ]"/>
    </record>

    <record id="VA" model="account.tax.template">
        <field name="description">AAM</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Visszaigényelhető – Alanyi Adómentes</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="sequence">2</field>
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

    <record id="VT" model="account.tax.template">
        <field name="description">TAM</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Visszaigényelhető – Tárgyi Adómentes</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="sequence">2</field>
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

    <record id="VAHT" model="account.tax.template">
        <field name="description">ÁHT</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Visszaigényelhető – ÁFA hatályán kívüli</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="sequence">2</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
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

    <record id="VEU" model="account.tax.template">
        <field name="description">EU</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Visszaigényelhető – EU</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="sequence">2</field>
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

    <record id="VEUO" model="account.tax.template">
        <field name="description">Import</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Visszaigényelhető – Import</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="sequence">2</field>
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

    <record id="VF" model="account.tax.template">
        <field name="description">FORD</field>
        <field name="chart_template_id" ref="hungarian_chart_template"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Visszaigényelhető – Fordított ÁFA</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="sequence">2</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_afa_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
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
        <field name="refund_repartition_line_ids" eval="[(5,0,0),
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
    </record>

</odoo>

```

## File: data\l10n_hu_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_hu_statements_menu" name="Hungary" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_user"/>

    <record id="hungarian_chart_template" model="account.chart.template">
        <field name="name">Magyar főkönyvi kivonat</field>
        <field name="code_digits">4</field>
        <field name="cash_account_code_prefix">381</field>
        <field name="bank_account_code_prefix">384</field>
        <field name="transfer_account_code_prefix">389</field>
        <field name="currency_id" ref="base.HUF"/>
    </record>
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

