# Odoo Module: l10n_hr

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Author: Goran Kliska
# mail:   goran.kliska(AT)slobodni-programi.hr
# Copyright (C) 2011- Slobodni programi d.o.o., Zagreb

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Author: Goran Kliska
# mail:   goran.kliska(AT)slobodni-programi.hr
# Copyright (C) 2011- Slobodni programi d.o.o., Zagreb
# Contributions:
#           Tomislav Bošnjaković, Storm Computers d.o.o. :
#              - account types

{
    "name": "Croatia - Accounting (RRIF 2012)",
    "description": """
Croatian localisation.
======================

Author: Goran Kliska, Slobodni programi d.o.o., Zagreb
        https://www.slobodni-programi.hr

Contributions:
  Tomislav Bošnjaković, Storm Computers: tipovi konta
  Ivan Vađić, Slobodni programi: tipovi konta

Description:

Croatian Chart of Accounts (RRIF ver.2012)

RRIF-ov računski plan za poduzetnike za 2012.
Vrste konta
Kontni plan prema RRIF-u, dorađen u smislu kraćenja naziva i dodavanja analitika
Porezne grupe prema poreznoj prijavi
Porezi PDV obrasca
Ostali porezi
Osnovne fiskalne pozicije

Izvori podataka:
 https://www.rrif.hr/dok/preuzimanje/rrif-rp2011.rar
 https://www.rrif.hr/dok/preuzimanje/rrif-rp2012.rar

""",
    "version": "13.0",
    "author": "OpenERP Croatian Community",
    'category': 'Accounting/Localizations/Account Charts',

    'depends': [
        'account',
    ],
    'data': [
        'data/l10n_hr_chart_data.xml',
        'data/account.account.template.csv',
        'data/account_chart_tag_data.xml',
        'data/account.tax.group.csv',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_tax_fiscal_position_data.xml',
        'data/account_chart_template_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"code","id","chart_template_id/id","account_type","reconcile","name","note"
"0000","kp_rrif0000","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za upisani a neuplaćeni dionički kapital (analitika po upisnicima)-a1","Potraživanja za upisani a neuplaćeni dionički kapital (analitika po upisnicima)-Analitika 1"
"0010","kp_rrif0010","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja iz ponovljene emisije dionica za upisane a neuplaćene svote kapitala po emisijama dionica-a1","Potraživanja iz ponovljene emisije dionica za upisane a neuplaćene svote kapitala (razrada po emisijama dionica)-Analitika 1"
"0020","kp_rrif0020","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za upisani a neuplaćeni kapital u d.o.o. (analitika po članovima društva)-a1","Potraživanja za upisani a neuplaćeni kapital u d.o.o. (analitika po članovima društva)-Analitika 1"
"0030","kp_rrif0030","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za temeljni ulog komanditora-a1","Potraživanja za temeljni ulog komanditora-Analitika 1"
"0040","kp_rrif0040","l10n_hr_chart_template_rrif","asset_current",,"Potraživanje za ostale uloge u kapital-a1","Potraživanje za ostale uloge u kapital-Analitika 1"
"0100","kp_rrif0100","l10n_hr_chart_template_rrif","asset_non_current",,"Izdatci za razvoj projekta (konstruiranje i test. prototipova i modela,alata,naprava i kalupa i sl.","Izdatci za razvoj projekta (konstruiranje i testiranje prototipova i modela, dizajn alata, naprava i kalupa i sl."
"0101","kp_rrif0101","l10n_hr_chart_template_rrif","asset_non_current",,"Izdatci za razvoj proizvoda (uzorci, recepture, troškovi pronalazaka i sl.).","Izdatci za razvoj proizvoda (uzorci, recepture, troškovi pronalazaka i sl.)."
"0102","kp_rrif0102","l10n_hr_chart_template_rrif","asset_non_current",,"Izdatci za istraživanje mineralnih blaga (MSFI 6)","Izdatci za istraživanje mineralnih blaga (MSFI 6)"
"0110","kp_rrif0110","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganje u koncesije idozvole(za resurse, ceste, ribarenje, linije, sirovine, itd.)","Ulaganje u koncesije i dozvole (za resurse, ceste, ribarenje, linije, sirovine, itd.)"
"0111","kp_rrif0111","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganje u patente i tehnologiju, inovacije, teh.dokumentaciju za proizv. proizvoda ili pružanje usluga","Ulaganje u patente i tehnologiju, inovacije, tehničku i tehnološku dokumentaciju za proizvodnju proizvoda ili pružanje usluga"
"0112","kp_rrif0112","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganje u licenciju i frenčajz (Franchising)","Ulaganje u licenciju i frenčajz (Franchising)"
"0113","kp_rrif0113","l10n_hr_chart_template_rrif","asset_non_current",,"Robne marke, trgovačko ime, lista kupaca, industrijska prava, marketinška prava, usl. marke i sl.prava","Robne marke, trgovačko ime, lista kupaca, industrijska prava, marketinška prava, uslužne marke i sl. prava"
"0114","kp_rrif0114","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganje u tržišni udio (otkup prava distribucije za neko područje)","Ulaganje u tržišni udio (otkup prava distribucije za neko područje)"
"01200","kp_rrif01200","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganje u računalni softwer","Ulaganje u računalni softwer"
"01201","kp_rrif01201","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganje u internetske stranice","Ulaganje u internetske stranice"
"01210","kp_rrif01210","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganje u autorska i dr. prava korištenja","Ulaganje u autorska i dr. prava korištenja"
"01211","kp_rrif01211","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganje u znanje (know how), dizajn","Ulaganje u znanje (know how), dizajn"
"01212","kp_rrif01212","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganje u pravo reproduciranja (npr. filmova), pravo objave u izdavaštvu i sl.","Ulaganje u pravo reproduciranja (npr. filmova), pravo objave u izdavaštvu i sl."
"01213","kp_rrif01213","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganja na tuđoj imovini radi uporabe ili poboljšanja (nekretnina, opreme i sl.)","Ulaganja na tuđoj imovini radi uporabe ili poboljšanja (nekretnina, opreme i sl.)"
"01214","kp_rrif01214","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganje u dugogodišnje pravo uporabe (prema ugovoru)","Ulaganje u dugogodišnje pravo uporabe (prema ugovoru)"
"01215","kp_rrif01215","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganje u pravo suvlasništva opreme","Ulaganje u pravo suvlasništva opreme"
"01220","kp_rrif01220","l10n_hr_chart_template_rrif","asset_non_current",,"Založno pravo i hipoteke (realizirane)","Založno pravo i hipoteke (realizirane)"
"01221","kp_rrif01221","l10n_hr_chart_template_rrif","asset_non_current",,"Ostala dugogodišnja prava","Ostala dugogodišnja prava"
"0130","kp_rrif0130","l10n_hr_chart_template_rrif","asset_non_current",,"Goodwill","Goodwill"
"0140","kp_rrif0140","l10n_hr_chart_template_rrif","asset_non_current",,"Dugogodišnje naknade plaćene za pravo građenja, pravo prolaza i sl.","Dugogodišnje naknade plaćene za pravo građenja, pravo prolaza i sl."
"0141","kp_rrif0141","l10n_hr_chart_template_rrif","asset_non_current",,"Filmovi, glazbeni zapisi","Filmovi, glazbeni zapisi"
"0142","kp_rrif0142","l10n_hr_chart_template_rrif","asset_non_current",,"Ostala nematerijalna imovina","Ostala nematerijalna imovina"
"0150","kp_rrif0150","l10n_hr_chart_template_rrif","asset_non_current",,"Predujmovi za razvoj","Predujmovi za razvoj"
"0151","kp_rrif0151","l10n_hr_chart_template_rrif","asset_non_current",,"Predujmovi za koncesije","Predujmovi za koncesije"
"0152","kp_rrif0152","l10n_hr_chart_template_rrif","asset_non_current",,"Predujmovi za patente, licencije i dr.","Predujmovi za patente, licencije i dr."
"0153","kp_rrif0153","l10n_hr_chart_template_rrif","asset_non_current",,"Predujmovi za robne ili uslužne marke","Predujmovi za robne ili uslužne marke"
"0154","kp_rrif0154","l10n_hr_chart_template_rrif","asset_non_current",,"Predujmovi za nabavu softwera","Predujmovi za nabavu softwera"
"0155","kp_rrif0155","l10n_hr_chart_template_rrif","asset_non_current",,"Predujmovi za nabavu ostalih prava uporabe","Predujmovi za nabavu ostalih prava uporabe"
"0156","kp_rrif0156","l10n_hr_chart_template_rrif","asset_non_current",,"Predujmovi za ostalu nematerijalnu imovinu","Predujmovi za ostalu nematerijalnu imovinu"
"0160","kp_rrif0160","l10n_hr_chart_template_rrif","asset_non_current",,"Nematerijalna imovina u pripremi (analitika prema vrsti računa skupine 01)-a1","Nematerijalna imovina u pripremi (analitika prema vrsti računa skupine 01)-Analitika 1"
"0180","kp_rrif0180","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosno usklađenje nematerijalne imovine (analitika prema vrsti računa skupine 01)-a1","Vrijednosno usklađenje nematerijalne imovine (analitika prema vrsti računa skupine 01)-Analitika 1"
"0190","kp_rrif0190","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortizacija izdataka za razvoj","Akumulirana amortizacija izdataka za razvoj"
"0191","kp_rrif0191","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortizacija koncesija, patenata, licencija i sl.","Akumulirana amortizacija koncesija, patenata, licencija i sl."
"0192","kp_rrif0192","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amort. robne i uslužne marke","Akumulirana amort. robne i uslužne marke"
"0193","kp_rrif0193","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortizacija softwera","Akumulirana amortizacija softwera"
"0194","kp_rrif0194","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortizacija godwilla (v. napom. 3.)","Akumulirana amortizacija godwilla (v. napom. 3.)"
"0195","kp_rrif0195","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amort. ostale nematerijalne imovine","Akumulirana amort. ostale nematerijalne imovine"
"0200","kp_rrif0200","l10n_hr_chart_template_rrif","asset_non_current",,"Građevinsko zemljište (bez zgrada)","Građevinsko zemljište (bez zgrada)"
"0201","kp_rrif0201","l10n_hr_chart_template_rrif","asset_non_current",,"Poljoprivredno zemljište","Poljoprivredno zemljište"
"0202","kp_rrif0202","l10n_hr_chart_template_rrif","asset_non_current",,"Zemljište za deponije smeća i otpada","Zemljište za deponije smeća i otpada"
"0203","kp_rrif0203","l10n_hr_chart_template_rrif","asset_non_current",,"Zemljište za eksploataciju kamena, gline, šljunka i pijeska,","Zemljište za eksploataciju kamena, gline, šljunka i pijeska,"
"0204","kp_rrif0204","l10n_hr_chart_template_rrif","asset_non_current",,"Zemljište sa supstancijalnom potrošnjom ili odlagališta","Zemljište sa supstancijalnom potrošnjom ili odlagališta"
"0205","kp_rrif0205","l10n_hr_chart_template_rrif","asset_non_current",,"Zemljište pod prometnicama, dvorištima, parkiralištima i sl.","Zemljište pod prometnicama, dvorištima, parkiralištima i sl."
"0206","kp_rrif0206","l10n_hr_chart_template_rrif","asset_non_current",,"Zemljišta pod dugogodišnjim nasadama, parkovima, vrtovima i sl.","Zemljišta pod dugogodišnjim nasadama, parkovima, vrtovima i sl."
"0207","kp_rrif0207","l10n_hr_chart_template_rrif","asset_non_current",,"Čista neobrađena i nezasađena zemljišta i kamenjari","Čista neobrađena i nezasađena zemljišta i kamenjari"
"0208","kp_rrif0208","l10n_hr_chart_template_rrif","asset_non_current",,"Zemljišta ispod građevina","Zemljišta ispod građevina"
"0209","kp_rrif0209","l10n_hr_chart_template_rrif","asset_non_current",,"Poboljšanja na zemljištu (ulaganja u odvodnjavanje, uređivanje prilaza i sl.)","Poboljšanja na zemljištu (ulaganja u odvodnjavanje, uređivanje prilaza i sl.)"
"0210","kp_rrif0210","l10n_hr_chart_template_rrif","asset_non_current",,"Zemljište s upisanim pravom građenja (unaprijed plaćena)","Zemljište s upisanim pravom građenja (unaprijed plaćena)"
"0211","kp_rrif0211","l10n_hr_chart_template_rrif","asset_non_current",,"Pravo služnosti na zemljištu (unaprijed plaćena)","Pravo služnosti na zemljištu (unaprijed plaćena)"
"0212","kp_rrif0212","l10n_hr_chart_template_rrif","asset_non_current",,"Zemljišta u zakupu (unaprijed plaćena)","Zemljišta u zakupu (unaprijed plaćena)"
"0230","kp_rrif0230","l10n_hr_chart_template_rrif","asset_non_current",,"Poslovne zgrade","Poslovne zgrade"
"0231","kp_rrif0231","l10n_hr_chart_template_rrif","asset_non_current",,"Tvorničke zgrade, hale i radionice","Tvorničke zgrade, hale i radionice"
"0232","kp_rrif0232","l10n_hr_chart_template_rrif","asset_non_current",,"Zgrade trgovine, hotela, motela, restorana","Zgrade trgovine, hotela, motela, restorana"
"0233","kp_rrif0233","l10n_hr_chart_template_rrif","asset_non_current",,"Skladišta, silosi, nadstrešnice i garaže, staklenici, sušionice, hladnjače","Skladišta, silosi, nadstrešnice i garaže, staklenici, sušionice, hladnjače"
"0234","kp_rrif0234","l10n_hr_chart_template_rrif","asset_non_current",,"Zgrade montažne, barake, mostovi, drvene konstrukcije i sl.","Zgrade montažne, barake, mostovi, drvene konstrukcije i sl."
"0235","kp_rrif0235","l10n_hr_chart_template_rrif","asset_non_current",,"Ograde, izlozi, potporni zidovi -brane, športski tereni,šatori,žičare i sl.","Ograde (betonske, kamene, metalne i sl.), izlozi, potporni zidovi - brane, športski tereni, šatori, žičare i sl."
"0236","kp_rrif0236","l10n_hr_chart_template_rrif","asset_non_current",,"Putovi, parkirališta, staze i dr. građevine (rampe i sl.), nadvožnjaci i dr. bet. ili met. konstruk.","Putovi, parkirališta, staze i dr. građevine (rampe i sl.), nadvožnjaci i dr. betonske ili metalne konstrukcije"
"0237","kp_rrif0237","l10n_hr_chart_template_rrif","asset_non_current",,"Cjevovodi, vodospremnici, utvrđene obale, kanali, kanalizacija, dalekovodi","Cjevovodi, vodospremnici, utvrđene obale, kanali, kanalizacija, dalekovodi"
"0238","kp_rrif0238","l10n_hr_chart_template_rrif","asset_non_current",,"Objekti poljoprivrede i ribarstva","Objekti poljoprivrede i ribarstva"
"0239","kp_rrif0239","l10n_hr_chart_template_rrif","asset_non_current",,"Ostali nespomenuti građevinski objekti (rudnici, brane) i objekti izvan uporabe","Ostali nespomenuti građevinski objekti (rudnici, brane) i objekti izvan uporabe"
"0240","kp_rrif0240","l10n_hr_chart_template_rrif","asset_non_current",,"Stanovi za vlastite zaposlenike-a1","Stanovi za vlastite zaposlenike-Analitika 1"
"0260","kp_rrif0260","l10n_hr_chart_template_rrif","asset_non_current",,"Predujmovi za nabavu zemljišta","Predujmovi za nabavu zemljišta"
"0261","kp_rrif0261","l10n_hr_chart_template_rrif","asset_non_current",,"Predujmovi za građevine u nabavi","Predujmovi za građevine u nabavi"
"0270","kp_rrif0270","l10n_hr_chart_template_rrif","asset_non_current",,"Zemljišta u pripremi","Zemljišta u pripremi"
"0271","kp_rrif0271","l10n_hr_chart_template_rrif","asset_non_current",,"Građevine u pripremi","Građevine u pripremi"
"0280","kp_rrif0280","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosno usklađenje zemljišta","Vrijednosno usklađenje zemljišta"
"0281","kp_rrif0281","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosno usklađenje građevina","Vrijednosno usklađenje građevina"
"0290","kp_rrif0290","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortizacija građevina (analitika prema pojedinim građevinama)","Akumulirana amortizacija građevina (analitika prema pojedinim građevinama)"
"0291","kp_rrif0291","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortizacija odlagališta otpada, kamenoloma i sl. (MRS 16. t. 58.)","Akumulirana amortizacija odlagališta otpada, kamenoloma i sl. (MRS 16. t. 58.)"
"0300","kp_rrif0300","l10n_hr_chart_template_rrif","asset_non_current",,"Tehnička postrojenja, uređaji, spremnici, pogonski motori, platforme i dr.","Tehnička postrojenja, uređaji, spremnici, pogonski motori, platforme i dr."
"0301","kp_rrif0301","l10n_hr_chart_template_rrif","asset_non_current",,"Strojevi i alati u svezi sa strojevima u pogonima i radionicama za obradu i preradu","Strojevi i alati u svezi sa strojevima u pogonima i radionicama za obradu i preradu"
"0302","kp_rrif0302","l10n_hr_chart_template_rrif","asset_non_current",,"Energetska postrojenja (kotlovnice, generatori,solarne ćelije, vjetro-elektrane, toplinske crpke i dr.)","Energetska postrojenja (kotlovnice, generatori, naponske i solarne ćelije, vjetro-elektrane, toplinske crpke i dr.)"
"0303","kp_rrif0303","l10n_hr_chart_template_rrif","asset_non_current",,"Rashladna postrojenja","Rashladna postrojenja"
"0304","kp_rrif0304","l10n_hr_chart_template_rrif","asset_non_current",,"Prijenosna postrojenja (dizala, pokretne stepenice, elevatori, pokretne trake i sl.)","Prijenosna postrojenja (dizala, pokretne stepenice, elevatori, pokretne trake i sl.)"
"0305","kp_rrif0305","l10n_hr_chart_template_rrif","asset_non_current",,"Poboljšanje na postrojenjima","Poboljšanje na postrojenjima"
"0306","kp_rrif0306","l10n_hr_chart_template_rrif","asset_non_current",,"Mlinska postrojenja","Mlinska postrojenja"
"0307","kp_rrif0307","l10n_hr_chart_template_rrif","asset_non_current",,"Postrojenje za pakiranje, ambalažu i sl.","Postrojenje za pakiranje, ambalažu i sl."
"0309","kp_rrif0309","l10n_hr_chart_template_rrif","asset_non_current",,"Ostala postrojenja i postrojenja izvan uporabe","Ostala postrojenja i postrojenja izvan uporabe"
"0310","kp_rrif0310","l10n_hr_chart_template_rrif","asset_non_current",,"Uredska oprema (fotokopirni, telefoni, telefaxi, blagajne, alarmi, klima, hladnjaci, televizori, i dr.)","Uredska oprema (fotokopirni aparati, telefoni, telefaxi, blagajne, alarmi, klimatizacijski uređaji, hladnjaci, televizori, i dr.)"
"0311","kp_rrif0311","l10n_hr_chart_template_rrif","asset_non_current",,"Računalna oprema","Računalna oprema"
"0312","kp_rrif0312","l10n_hr_chart_template_rrif","asset_non_current",,"Telekomunikacijska oprema (mobiteli, tel. centrale, antene i sl.)","Telekomunikacijska oprema (mobiteli, tel. centrale, antene i sl.)"
"0313","kp_rrif0313","l10n_hr_chart_template_rrif","asset_non_current",,"Oprema trgovine (police, blagajne, hladnjaci, i dr.)","Oprema trgovine (police, blagajne, hladnjaci, i dr.)"
"0314","kp_rrif0314","l10n_hr_chart_template_rrif","asset_non_current",,"Oprema ugostiteljstva, hotela i sl. (aparati, štednjaci, hladnjaci, pokućstvo i sl.)","Oprema ugostiteljstva, hotela i sl. (aparati, štednjaci, hladnjaci, pokućstvo i sl.)"
"0315","kp_rrif0315","l10n_hr_chart_template_rrif","asset_non_current",,"Oprema servisa (dizalice, ispitni uređaji, aparati i dr.)","Oprema servisa (dizalice, ispitni uređaji, aparati i dr.)"
"0316","kp_rrif0316","l10n_hr_chart_template_rrif","asset_non_current",,"Oprema za graditeljstvo i montažu(kranovi, bageri, skele, oplate, mješalice, dizalice, valjci i sl.)","Oprema za graditeljstvo (kranovi, bageri, skele, oplate, mješalice, dizalice, valjci i sl.)"
"0317","kp_rrif0317","l10n_hr_chart_template_rrif","asset_non_current",,"Oprema grijanja i hlađenja","Oprema grijanja i hlađenja"
"0318","kp_rrif0318","l10n_hr_chart_template_rrif","asset_non_current",,"Oprema zaštite na radu i protupožarne zaštite","Oprema zaštite na radu i protupožarne zaštite"
"0319","kp_rrif0319","l10n_hr_chart_template_rrif","asset_non_current",,"Ostala oprema i oprema izvan uporabe","Ostala oprema i oprema izvan uporabe"
"03200","kp_rrif03200","l10n_hr_chart_template_rrif","asset_non_current",,"Putnička vozila (osobna i putnički kombi) i motor kotači","Putnička vozila (osobna i putnički kombi) i motor kotači"
"03201","kp_rrif03201","l10n_hr_chart_template_rrif","asset_non_current",,"Teretna i vučna vozila, tegljači i kamioni","Teretna i vučna vozila, tegljači i kamioni"
"03202","kp_rrif03202","l10n_hr_chart_template_rrif","asset_non_current",,"Priključna transportna sredstva (prikolice)","Priključna transportna sredstva (prikolice)"
"03203","kp_rrif03203","l10n_hr_chart_template_rrif","asset_non_current",,"Teretna vozila (dostavna i kombi) i hladnjače, cisterne","Teretna vozila (dostavna i kombi) i hladnjače, cisterne"
"03204","kp_rrif03204","l10n_hr_chart_template_rrif","asset_non_current",,"Auto mješalice, auto crpke za beton, auto dizalice i sl.","Auto mješalice, auto crpke za beton, auto dizalice i sl."
"03205","kp_rrif03205","l10n_hr_chart_template_rrif","asset_non_current",,"Autobusi","Autobusi"
"03206","kp_rrif03206","l10n_hr_chart_template_rrif","asset_non_current",,"Zrakoplovi","Zrakoplovi"
"03207","kp_rrif03207","l10n_hr_chart_template_rrif","asset_non_current",,"Brodovi (veći od 1000 BRT)","Brodovi (veći od 1000 BRT)"
"03208","kp_rrif03208","l10n_hr_chart_template_rrif","asset_non_current",,"Brodice, jahte i ost. plovila","Brodice, jahte i ost. plovila"
"03209","kp_rrif03209","l10n_hr_chart_template_rrif","asset_non_current",,"Ostala transportna sredstva i uređaji (gusjeničari, el. vozila, viljuškari, vagoni bicikli i dr.)","Ostala transportna sredstva i uređaji (gusjeničari, el. vozila, viljuškari, vagoni, elevatori, traktori, bicikli i dr.)"
"03210","kp_rrif03210","l10n_hr_chart_template_rrif","asset_non_current",,"Uredsko pokućstvo, sagovi, zavjese i sl.","Uredsko pokućstvo, sagovi, zavjese i sl."
"03211","kp_rrif03211","l10n_hr_chart_template_rrif","asset_non_current",,"Inventar trgovine (police, pregrade, pultovi)","Inventar trgovine (police, pregrade, pultovi)"
"03212","kp_rrif03212","l10n_hr_chart_template_rrif","asset_non_current",,"Ugostiteljsko i hotelsko pokućstvo i inventar","Ugostiteljsko i hotelsko pokućstvo i inventar"
"03213","kp_rrif03213","l10n_hr_chart_template_rrif","asset_non_current",,"Ostalo pokućstvo i inventar","Ostalo pokućstvo i inventar"
"0322","kp_rrif0322","l10n_hr_chart_template_rrif","asset_non_current",,"Pogonski i skladišni inventar (stalaže, zatvoreni ormari, skele, oplate, protupožarni aparati i sl.)","Pogonski i skladišni inventar (stalaže, zatvoreni ormari, skele, oplate, protupožarni aparati, zaštitna sredstva i sl.)"
"0323","kp_rrif0323","l10n_hr_chart_template_rrif","asset_non_current",,"Alati, mjerni i kontrolni instrumenti i pomoćna oprema","Alati, mjerni i kontrolni instrumenti i pomoćna oprema"
"0324","kp_rrif0324","l10n_hr_chart_template_rrif","asset_non_current",,"Audio i video aparati, kamere, parkir. rampe i sl.","Audio i video aparati, kamere, parkir. rampe i sl."
"0325","kp_rrif0325","l10n_hr_chart_template_rrif","asset_non_current",,"Reklame (svjetleće), stupovi i sl.","Reklame (svjetleće), stupovi i sl."
"0326","kp_rrif0326","l10n_hr_chart_template_rrif","asset_non_current",,"Inventar ustanova (aparati, kreveti i sl.)","Inventar ustanova (aparati, kreveti i sl.)"
"0327","kp_rrif0327","l10n_hr_chart_template_rrif","asset_non_current",,"Ostali pogonski inventar,","Ostali pogonski inventar,"
"0328","kp_rrif0328","l10n_hr_chart_template_rrif","asset_non_current",,"Višegodišnja ambalaža","Višegodišnja ambalaža"
"0329","kp_rrif0329","l10n_hr_chart_template_rrif","asset_non_current",,"Alati, inventar i vozila izvan uporabe","Alati, inventar i vozila izvan uporabe"
"0330","kp_rrif0330","l10n_hr_chart_template_rrif","asset_non_current",,"30% pretporeza od osobnih automobila (n. v. do 400.000,00 kn)","30% pretporeza od osobnih automobila (n. v. do 400.000,00 kn)"
"0331","kp_rrif0331","l10n_hr_chart_template_rrif","asset_non_current",,"30% i 100% pretporeza od osobnih automobila (n. v. veće od 400.000,00 kn)","30% i 100% pretporeza od osobnih automobila (n. v. veće od 400.000,00 kn)"
"0332","kp_rrif0332","l10n_hr_chart_template_rrif","asset_non_current",,"30% pretporeza od brodova, jahti i dr. (n.v. do 400.000,00 kn)","30% pretporeza od brodova, jahti i dr. (n.v. do 400.000,00 kn)"
"0333","kp_rrif0333","l10n_hr_chart_template_rrif","asset_non_current",,"30% i 100% pretporeza od brodova, jahti i dr. (n.v. veće od 400.000,00 kn)","30% i 100% pretporeza od brodova, jahti i dr. (n.v. veće od 400.000,00 kn)"
"0340","kp_rrif0340","l10n_hr_chart_template_rrif","asset_non_current",,"Traktori, kombajni, prikolice, kosilice i sl.","Traktori, kombajni, prikolice kosilice i sl."
"0341","kp_rrif0341","l10n_hr_chart_template_rrif","asset_non_current",,"Radni priključci (plugovi, beračice, freze, prskalice, sabirače i sl.)","Radni priključci (plugovi, beračice, freze, prskalice, sabirače i sl.)"
"0342","kp_rrif0342","l10n_hr_chart_template_rrif","asset_non_current",,"Oprema za mljekarstvo (muzilice, separatori, police, spremnici i sl.)","Oprema za mljekarstvo (muzilice, separatori, police, spremnici i sl.)"
"0343","kp_rrif0343","l10n_hr_chart_template_rrif","asset_non_current",,"Oprema ribarstva (kavezi, mreže, čamci i brodovi, pakirnice)","Oprema ribarstva (kavezi, mreže, čamci i brodovi, pakirnice)"
"0344","kp_rrif0344","l10n_hr_chart_template_rrif","asset_non_current",,"Oprema vinogradarstva (bačve, filteri, preše, punionica)","Oprema vinogradarstva (bačve, filteri, preše, punionica)"
"0345","kp_rrif0345","l10n_hr_chart_template_rrif","asset_non_current",,"Oprema vočarstva i maslinarstva","Oprema vočarstva i maslinarstva"
"0346","kp_rrif0346","l10n_hr_chart_template_rrif","asset_non_current",,"Oprema stočarstva i pćelarstva","Oprema stočarstva i pćelarstva"
"0347","kp_rrif0347","l10n_hr_chart_template_rrif","asset_non_current",,"Oprema mlinova","Oprema mlinova"
"0349","kp_rrif0349","l10n_hr_chart_template_rrif","asset_non_current",,"Ostala oprema poljoprivrede, stočarstva i ribarstva","Ostala oprema poljoprivrede, stočarstva i ribarstva"
"0350","kp_rrif0350","l10n_hr_chart_template_rrif","asset_non_current",,"Umjetnine, slike i sl.","Umjetnine, slike i sl."
"0351","kp_rrif0351","l10n_hr_chart_template_rrif","asset_non_current",,"Arhivski predmeti, makete i sl.","Arhivski predmeti, makete i sl."
"0352","kp_rrif0352","l10n_hr_chart_template_rrif","asset_non_current",,"Knjige, karte, fotografije i sl.","Knjige, karte, fotografije i sl."
"0353","kp_rrif0353","l10n_hr_chart_template_rrif","asset_non_current",,"Oldtimeri (automobili, brodovi i dr.)","Oldtimeri (automobili, brodovi i dr.)"
"0354","kp_rrif0354","l10n_hr_chart_template_rrif","asset_non_current",,"Ostala materijalna imovina","Ostala materijalna imovina"
"0360","kp_rrif0360","l10n_hr_chart_template_rrif","asset_non_current",,"Predujmovi za postrojenja i opremu","Predujmovi za postrojenja i opremu"
"0361","kp_rrif0361","l10n_hr_chart_template_rrif","asset_non_current",,"Predujmovi za alate, pogonski inventar i transportnu imovinu","Predujmovi za alate, pogonski inventar i transportnu imovinu"
"0367","kp_rrif0367","l10n_hr_chart_template_rrif","asset_non_current",,"Predujam za ostalu imovinu","Predujam za ostalu imovinu"
"0370","kp_rrif0370","l10n_hr_chart_template_rrif","asset_non_current",,"Postrojenja u pripremi","Postrojenja u pripremi"
"0371","kp_rrif0371","l10n_hr_chart_template_rrif","asset_non_current",,"Oprema u pripremi","Oprema u pripremi"
"0372","kp_rrif0372","l10n_hr_chart_template_rrif","asset_non_current",,"Alati, pogonski inventar u pripremi","Alati, pogonski inventar u pripremi"
"0373","kp_rrif0373","l10n_hr_chart_template_rrif","asset_non_current",,"Osobni automobili i transportna sredstva u pripremi","Osobni automobili i transportna sredstva u pripremi"
"0374","kp_rrif0374","l10n_hr_chart_template_rrif","asset_non_current",,"Poljoprivredna oprema u pripremi","Poljoprivredna oprema u pripremi"
"0375","kp_rrif0375","l10n_hr_chart_template_rrif","asset_non_current",,"Ostala imovina u pripremi","Ostala imovina u pripremi"
"0380","kp_rrif0380","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosno usklađenje postrojenja","Vrijednosno usklađenje postrojenja"
"0381","kp_rrif0381","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosno usklađenje opreme","Vrijednosno usklađenje opreme"
"0382","kp_rrif0382","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosno usklađenje alata, pogonskog inventara i transportne imovine","Vrijednosno usklađenje alata, pogonskog inventara i transportne imovine"
"0383","kp_rrif0383","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosno usklađenje ostale mat. imovine","Vrijednosno usklađenje ostale mat. imovine"
"0384","kp_rrif0384","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosno usklađenje poljoprivredne opreme","Vrijednosno usklađenje poljoprivredne opreme"
"0390","kp_rrif0390","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortiz. postrojenja","Akumulirana amortiz. postrojenja"
"0391","kp_rrif0391","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortiz. opreme","Akumulirana amortiz. opreme"
"0392","kp_rrif0392","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortiz. alata, pogonskog inventara i transportne imovine","Akumulirana amortiz. alata, pogonskog inventara i transportne imovine"
"0393","kp_rrif0393","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortiz. ostale mat. imovine","Akumulirana amortiz. ostale mat. imovine"
"0394","kp_rrif0394","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortiz. poljoprivredne opreme","Akumulirana amortiz. poljoprivredne opreme"
"0395","kp_rrif0395","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortiz. 30% pretporeza od osob. automobila","Akumulirana amortiz. 30% pretporeza od osob. automobila"
"0396","kp_rrif0396","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortiz. 30% i 100% pretporeza od osob. automobila (n. v. veće od 400.000,00 kn)","Akumulirana amortiz. 30% i 100% pretporeza od osob. automobila (n. v. veće od 400.000,00 kn)"
"0397","kp_rrif0397","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortiz. 30% od pretporeza od brodova, jahti i dr. sred. za osobni prijevoz","Akumulirana amortiz. 30% od pretporeza od brodova, jahti i dr. sred. za osobni prijevoz"
"0398","kp_rrif0398","l10n_hr_chart_template_rrif","asset_non_current",,"Akum. amortiz.30% i 100% pretporeza od brodova, jahti i dr. sred. za os. prijevoz(NV preko 400000,00kn)","Akumulirana amortiz. 30% i 100% pretporeza od brodova, jahti i dr. sred. za osobni prijevoz (n. v. veće od 400.000,00 kn)"
"0399","kp_rrif0399","l10n_hr_chart_template_rrif","asset_non_current",,"Ostala akumulirana amortizacija","Ostala akumulirana amortizacija"
"0400","kp_rrif0400","l10n_hr_chart_template_rrif","asset_non_current",,"Voćnjaci","Voćnjaci"
"0401","kp_rrif0401","l10n_hr_chart_template_rrif","asset_non_current",,"Vinogradi","Vinogradi"
"0402","kp_rrif0402","l10n_hr_chart_template_rrif","asset_non_current",,"Maslinici","Maslinici"
"0403","kp_rrif0403","l10n_hr_chart_template_rrif","asset_non_current",,"Plantaže drveća i bilja (šume)","Plantaže drveća i bilja (šume)"
"0404","kp_rrif0404","l10n_hr_chart_template_rrif","asset_non_current",,"Parkovi, zelenila, nasadi i cvijeće","Parkovi, zelenila, nasadi i cvijeće"
"0405","kp_rrif0405","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganja u ostale višegodišnje nasade","Ulaganja u ostale višegodišnje nasade"
"0410","kp_rrif0410","l10n_hr_chart_template_rrif","asset_non_current",,"Goveda","Goveda"
"0411","kp_rrif0411","l10n_hr_chart_template_rrif","asset_non_current",,"Konji","Konji"
"0412","kp_rrif0412","l10n_hr_chart_template_rrif","asset_non_current",,"Mazge, magarci i mule","Mazge, magarci i mule"
"0413","kp_rrif0413","l10n_hr_chart_template_rrif","asset_non_current",,"Svinje","Svinje"
"0414","kp_rrif0414","l10n_hr_chart_template_rrif","asset_non_current",,"Ovce i koze","Ovce i koze"
"0415","kp_rrif0415","l10n_hr_chart_template_rrif","asset_non_current",,"Perad","Perad"
"0416","kp_rrif0416","l10n_hr_chart_template_rrif","asset_non_current",,"Ribe","Ribe"
"0417","kp_rrif0417","l10n_hr_chart_template_rrif","asset_non_current",,"Pčelinja društva","Pčelinja društva"
"0418","kp_rrif0418","l10n_hr_chart_template_rrif","asset_non_current",,"Stado divljači","Stado divljači, kunića, nutrija (za rasplod)"
"0419","kp_rrif0419","l10n_hr_chart_template_rrif","asset_non_current",,"Ostale nespomenute životinje (psi, ptice i dr.)","Ostale nespomenute životinje (psi, ptice i dr.)"
"0460","kp_rrif0460","l10n_hr_chart_template_rrif","asset_non_current",,"Predujmovi za višegodišnje nasade","Predujmovi za višegodišnje nasade"
"0461","kp_rrif0461","l10n_hr_chart_template_rrif","asset_non_current",,"Predujmovi na nabavu životinja","Predujmovi na nabavu životinja"
"0470","kp_rrif0470","l10n_hr_chart_template_rrif","asset_non_current",,"Višegodišnji nasadi u pripremi","Višegodišnji nasadi u pripremi"
"0471","kp_rrif0471","l10n_hr_chart_template_rrif","asset_non_current",,"Životinje (osnovno stado) u nabavi","Životinje (osnovno stado) u nabavi"
"0480","kp_rrif0480","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosno usklađenje višegodišnjih nasada","Vrijednosno usklađenje višegodišnjih nasada"
"0481","kp_rrif0481","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosno usklađenje životinja (osnovnog stada)","Vrijednosno usklađenje životinja (osnovnog stada)"
"0490","kp_rrif0490","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortiz. višegodišnjih nasada","Akumulirana amortiz. višegodišnjih nasada"
"0491","kp_rrif0491","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortiz. životinja (osnovnog stada)","Akumulirana amortiz. životinja (osnovnog stada)"
"0500","kp_rrif0500","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganja u nekretnine - zemljišta-a1","Ulaganja u nekretnine - zemljišta-Analitika 1"
"0510","kp_rrif0510","l10n_hr_chart_template_rrif","asset_non_current",,"Građevine u najmovima (poslovne zgrade, stanovi, apartmani, kuće)","Građevine u najmovima (poslovne zgrade, stanovi, apartmani, kuće)"
"0511","kp_rrif0511","l10n_hr_chart_template_rrif","asset_non_current",,"Građevine izvan uporabe (zgrade, stanovi, apartmani, kuće)","Građevine izvan uporabe (zgrade, stanovi, apartmani, kuće)"
"0560","kp_rrif0560","l10n_hr_chart_template_rrif","asset_non_current",,"Predujam za ulaganja u zemljište","Predujam za ulaganja u zemljište"
"0561","kp_rrif0561","l10n_hr_chart_template_rrif","asset_non_current",,"Predujam za ulaganje u građevine","Predujam za ulaganje u građevine"
"0570","kp_rrif0570","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganja u zemljišta u nabavi","Ulaganja u zemljišta u nabavi"
"0571","kp_rrif0571","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganja u građevine u nabavi","Ulaganja u građevine u nabavi"
"0572","kp_rrif0572","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganja u građevine u izgradnji","Ulaganja u građevine u izgradnji"
"0580","kp_rrif0580","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosno usklađenje ulaganja u zemljište","Vrijednosno usklađenje ulaganja u zemljište"
"0581","kp_rrif0581","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosno usklađenje ulaganja u građevine","Vrijednosno usklađenje ulaganja u građevine"
"0590","kp_rrif0590","l10n_hr_chart_template_rrif","asset_non_current",,"Akumulirana amortizacija ulaganja u građevine-a1","Akumulirana amortizacija ulaganja u građevine-Analitika 1"
"0600","kp_rrif0600","l10n_hr_chart_template_rrif","asset_non_current",,"Udjel u dionicama (s više od 20%)","Udjel u dionicama (s više od 20%)"
"0601","kp_rrif0601","l10n_hr_chart_template_rrif","asset_non_current",,"Udjel u kapitalu d.o.o.-a (s više od 20%)","Udjel u kapitalu d.o.o.-a (s više od 20%)"
"0602","kp_rrif0602","l10n_hr_chart_template_rrif","asset_non_current",,"Udjeli u komanditnom društvu","Udjeli u komanditnom društvu"
"0603","kp_rrif0603","l10n_hr_chart_template_rrif","asset_non_current",,"Udjeli u društvima u inozemstvu (s više od 20%)","Udjeli u društvima u inozemstvu (s više od 20%)"
"0604","kp_rrif0604","l10n_hr_chart_template_rrif","asset_non_current",,"Osnivački udjeli u ustanovama","Osnivački udjeli u ustanovama"
"0605","kp_rrif0605","l10n_hr_chart_template_rrif","asset_non_current",,"Udjeli u zadrugama","Udjeli u zadrugama"
"0606","kp_rrif0606","l10n_hr_chart_template_rrif","asset_non_current",,"Udio u društvu s uzajamnim udjelima","Udio u društvu s uzajamnim udjelima"
"0607","kp_rrif0607","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganje u kapitalne pričuve (neupisani kapital)","Ulaganje u kapitalne pričuve (neupisani kapital)"
"0610","kp_rrif0610","l10n_hr_chart_template_rrif","asset_non_current",,"Dani zajmovi povezanim poduzetnicima (analitika po društvima u kojima se ima više od 20% udjela)-a1","Dani zajmovi povezanim poduzetnicima (analitika po društvima u kojima se ima više od 20% udjela)-Analitika 1"
"0620","kp_rrif0620","l10n_hr_chart_template_rrif","asset_non_current",,"Udjel u dioničkom kapitalu (do 20% udjela - analitika po društvima)","Udjel u dioničkom kapitalu (do 20% udjela - analitika po društvima)"
"0621","kp_rrif0621","l10n_hr_chart_template_rrif","asset_non_current",,"Udjel u kapitalu d.o.o. (do 20% udjela)","Udjel u kapitalu d.o.o. (do 20% udjela)"
"0622","kp_rrif0622","l10n_hr_chart_template_rrif","asset_non_current",,"Udjeli u društvima u inozemstvu (do 20% udjela)","Udjeli u društvima u inozemstvu (do 20% udjela)"
"0623","kp_rrif0623","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganje u dionice i udjele radi preprodaje","Ulaganje u dionice i udjele radi preprodaje"
"0624","kp_rrif0624","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja za udio u dobitku (analitika po udjelima)","Potraživanja za udio u dobitku (analitika po udjelima)"
"0630","kp_rrif0630","l10n_hr_chart_template_rrif","asset_non_current",,"Zajmovi dani poduzetnicima u kojima postoje sudjelujući interesi (do 20% udjela u T.K.)-a1","Zajmovi dani poduzetnicima u kojima postoje sudjelujući interesi (do 20% udjela u T.K.)-Analitika 1"
"0640","kp_rrif0640","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganja u mjenice, zadužnice (kupljene)","Ulaganja u mjenice, zadužnice (kupljene)"
"0641","kp_rrif0641","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganja u državne obveznice","Ulaganja u državne obveznice"
"0642","kp_rrif0642","l10n_hr_chart_template_rrif","asset_non_current",,"Dugotrajna ulaganja u obveznice društva","Dugotrajna ulaganja u obveznice društva"
"0643","kp_rrif0643","l10n_hr_chart_template_rrif","asset_non_current",,"Dugotrajna ulaganja u blagajničke zapise","Dugotrajna ulaganja u blagajničke zapise"
"0644","kp_rrif0644","l10n_hr_chart_template_rrif","asset_non_current",,"Dugotrajna ulaganja u opcije, certifikate i sl.","Dugotrajna ulaganja u opcije, certifikate i sl."
"0645","kp_rrif0645","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganja u robne ugovore","Ulaganja u robne ugovore"
"06460","kp_rrif06460","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganja u vrijednosne papire po fer vrijednosti","Ulaganja u vrijednosne papire po fer vrijednosti"
"06461","kp_rrif06461","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosni papiri namijenjeni prodaji (do dospijeća)","Vrijednosni papiri namijenjeni prodaji (do dospijeća)"
"0648","kp_rrif0648","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganje u vrijed. pap. raspoložive za prodaju","Ulaganje u vrijed. pap. raspoložive za prodaju"
"0649","kp_rrif0649","l10n_hr_chart_template_rrif","asset_non_current",,"Nezarađeni prihodi u financijskim instrumentima","Nezarađeni prihodi u financijskim instrumentima"
"06500","kp_rrif06500","l10n_hr_chart_template_rrif","asset_non_current",,"Dani zajmovi vanjskim pravnim osobama (nepovezanim)","Dani zajmovi vanjskim pravnim osobama (nepovezanim)"
"06501","kp_rrif06501","l10n_hr_chart_template_rrif","asset_non_current",,"Dani zajmovi vanjskim fizičkim osobama - obrtnicima","Dani zajmovi vanjskim fizičkim osobama - obrtnicima"
"06502","kp_rrif06502","l10n_hr_chart_template_rrif","asset_non_current",,"Dani zajmovi kooperantima","Dani zajmovi kooperantima"
"06503","kp_rrif06503","l10n_hr_chart_template_rrif","asset_non_current",,"Dani zajmovi direktoru, ortacima, menadžerima, zaposlenicima","Dani zajmovi direktoru, ortacima, menadžerima, zaposlenicima"
"06504","kp_rrif06504","l10n_hr_chart_template_rrif","asset_non_current",,"Oročenja u bankama","Oročenja u bankama"
"06505","kp_rrif06505","l10n_hr_chart_template_rrif","asset_non_current",,"Financijski zajmovi dani u inozemstvo","Financijski zajmovi dani u inozemstvo"
"06509","kp_rrif06509","l10n_hr_chart_template_rrif","asset_non_current",,"Ostali dugotrajni zajmovi","Ostali dugotrajni zajmovi"
"06510","kp_rrif06510","l10n_hr_chart_template_rrif","asset_non_current",,"Depoziti iz poslovnih aktivnosti","Depoziti iz poslovnih aktivnosti"
"06511","kp_rrif06511","l10n_hr_chart_template_rrif","asset_non_current",,"Depoziti kod osiguravajućih društava","Depoziti kod osiguravajućih društava"
"06512","kp_rrif06512","l10n_hr_chart_template_rrif","asset_non_current",,"Depoziti u poslovnim bankama","Depoziti u poslovnim bankama"
"06513","kp_rrif06513","l10n_hr_chart_template_rrif","asset_non_current",,"Depoziti na carini (garancija špeditera)","Depoziti na carini (garancija špeditera)"
"06514","kp_rrif06514","l10n_hr_chart_template_rrif","asset_non_current",,"Sudski depoziti","Sudski depoziti"
"06520","kp_rrif06520","l10n_hr_chart_template_rrif","asset_non_current",,"Kaucije za obveze","Kaucije za obveze"
"06521","kp_rrif06521","l10n_hr_chart_template_rrif","asset_non_current",,"Kaucije iz kupoprodajnih poslova","Kaucije iz kupoprodajnih poslova"
"06522","kp_rrif06522","l10n_hr_chart_template_rrif","asset_non_current",,"Kaucije za plaćanja","Kaucije za plaćanja"
"06523","kp_rrif06523","l10n_hr_chart_template_rrif","asset_non_current",,"Dane kapare i osiguranja","Dane kapare i osiguranja"
"0660","kp_rrif0660","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganja (udjeli) koji su raspoloživi za prodaju (dugotrajno ulaganje -MRS28.t.11. i 13. te MRS39.t.9.)","Ulaganja (udjeli) koji su raspoloživi za prodaju (dugotrajno ulaganje - MRS 28. t. 11. i 13. te MRS 39. t. 9.)"
"0670","kp_rrif0670","l10n_hr_chart_template_rrif","asset_non_current",,"Ulaganja u investicijske fondove (s rokom dužim od 1 god.)","Ulaganja u investicijske fondove (s rokom dužim od 1 god.)"
"0672","kp_rrif0672","l10n_hr_chart_template_rrif","asset_non_current",,"Ostala nespomenuta dugoročna ulaganja","Ostala nespomenuta dugoročna ulaganja"
"0680","kp_rrif0680","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosno usklađenje financijske imovine - dugotrajne (analitika prema oblicima imovine u usklađenju)-a1","Vrijednosno usklađenje financijske imovine - dugotrajne (analitika prema oblicima imovine u usklađenju)-Analitika 1"
"0690","kp_rrif0690","l10n_hr_chart_template_rrif","asset_non_current",,"Nezarađene kamate u kreditima i sl.-a1","Nezarađene kamate u kreditima i sl.-Analitika 1"
"0700","kp_rrif0700","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja od povezanih društava za isporuke dobara i usluga","Potraživanja od povezanih društava za isporuke dobara i usluga"
"0701","kp_rrif0701","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja od povezanih društava za dana sredstva na dugoročnu posudbu (osim novca)","Potraživanja od povezanih društava za dana sredstva na dugoročnu posudbu (osim novca)"
"0702","kp_rrif0702","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja od zadrugara, kooperanata i sl.","Potraživanja od zadrugara, kooperanata i sl."
"0710","kp_rrif0710","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja s osnove prodaje na robni kredit","Potraživanja s osnove prodaje na robni kredit"
"0711","kp_rrif0711","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja s osnove prodaje na robni kredit u inozemstvu","Potraživanja s osnove prodaje na robni kredit u inozemstvu"
"0712","kp_rrif0712","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja za prodane usluge na kredit","Potraživanja za prodane usluge na kredit"
"0713","kp_rrif0713","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanje za dugotr. imovinu prodanu na kredit","Potraživanje za dugotr. imovinu prodanu na kredit"
"0714","kp_rrif0714","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja za prodaju u financijskom lizingu","Potraživanja za prodaju u financijskom lizingu"
"0715","kp_rrif0715","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja za prodani udjel na kredit","Potraživanja za prodani udjel na kredit"
"0716","kp_rrif0716","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja za prodaju na potrošački kredit","Potraživanja za prodaju na potrošački kredit"
"0717","kp_rrif0717","l10n_hr_chart_template_rrif","asset_non_current",,"Ostala potraživanja iz prodaje na kredit","Ostala potraživanja iz prodaje na kredit"
"0720","kp_rrif0720","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja iz faktoringa-a1","Potraživanja iz faktoringa-Analitika 1"
"0730","kp_rrif0730","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja za jamčevine iz operativnog lizinga","Potraživanja za jamčevine iz operativnog lizinga"
"0731","kp_rrif0731","l10n_hr_chart_template_rrif","asset_non_current",,"Jamstvo za dobro izvedene radove","Jamstvo za dobro izvedene radove"
"0732","kp_rrif0732","l10n_hr_chart_template_rrif","asset_non_current",,"Jamčevine iz natječaja","Jamčevine iz natječaja"
"0733","kp_rrif0733","l10n_hr_chart_template_rrif","asset_non_current",,"Jamčevine za štete","Jamčevine za štete"
"0740","kp_rrif0740","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja u sporu i rizična potraživanja-a1","Potraživanja u sporu i rizična potraživanja-Analitika 1"
"0750","kp_rrif0750","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja za predujmove za usluge-a1","Potraživanja za predujmove za usluge-Analitika 1"
"0760","kp_rrif0760","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja od radnika","Potraživanja od radnika"
"0761","kp_rrif0761","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja od članova društva","Potraživanja od članova društva"
"0762","kp_rrif0762","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja iz ortaštva","Potraživanja iz ortaštva"
"0780","kp_rrif0780","l10n_hr_chart_template_rrif","asset_non_current",,"Vrijednosno usklađivanje dugotrajnih potraživanja-a1","Vrijednosno usklađivanje dugotrajnih potraživanja-Analitika 1"
"0790","kp_rrif0790","l10n_hr_chart_template_rrif","asset_non_current",,"Potraživanja za nezarađenu kamatu-a1","Potraživanja za nezarađenu kamatu-Analitika 1"
"08000","kp_rrif08000","l10n_hr_chart_template_rrif","asset_non_current",,"Odgođena porezna imovina s osnove poreznog gubitka-a1","Odgođena porezna imovina s osnove poreznog gubitka-Analitika 1"
"08010","kp_rrif08010","l10n_hr_chart_template_rrif","asset_non_current",,"Odgođena porezna imovina s osnove povećane amortizacije-a1","Odgođena porezna imovina s osnove povećane amortizacije -Analitika 1"
"0810","kp_rrif0810","l10n_hr_chart_template_rrif","asset_non_current",,"Ostala odgođena porezna imovina-a1","Ostala odgođena porezna imovina-Analitika 1"
"1007","kp_rrif1007","l10n_hr_chart_template_rrif","asset_current",,"Podračun društva","Podračun društva"
"1008","kp_rrif1008","l10n_hr_chart_template_rrif","asset_current",,"Račun društva u osnivanju","Račun društva u osnivanju"
"1030","kp_rrif1030","l10n_hr_chart_template_rrif","asset_current",,"Devizni račun u domaćoj banci (analitika po devizama)","Devizni račun u domaćoj banci (analitika po devizama)"
"1031","kp_rrif1031","l10n_hr_chart_template_rrif","asset_current",,"Devizni račun u inozemnoj banci (u EU)","Devizni račun u inozemnoj banci (u EU)"
"1032","kp_rrif1032","l10n_hr_chart_template_rrif","asset_current",,"Devizni račun reeksportnih poslova","Devizni račun reeksportnih poslova"
"1034","kp_rrif1034","l10n_hr_chart_template_rrif","asset_current",,"Devizni račun investicijskih radova","Devizni račun investicijskih radova"
"1035","kp_rrif1035","l10n_hr_chart_template_rrif","asset_current",,"Devizni račun poslovne jedinice u inozemstvu","Devizni račun poslovne jedinice u inozemstvu"
"1036","kp_rrif1036","l10n_hr_chart_template_rrif","asset_current",,"Devizni račun u slobodnoj zoni","Devizni račun u slobodnoj zoni"
"1037","kp_rrif1037","l10n_hr_chart_template_rrif","asset_current",,"Nerezidentski devizni račun","Nerezidentski devizni račun"
"1038","kp_rrif1038","l10n_hr_chart_template_rrif","asset_current",,"Devizni račun terminskih poslova","Devizni račun terminskih poslova"
"1039","kp_rrif1039","l10n_hr_chart_template_rrif","asset_current",,"Prijelazni devizni račun","Prijelazni devizni račun"
"1040","kp_rrif1040","l10n_hr_chart_template_rrif","asset_current",,"Otvoreni devizni akreditiv u domaćoj banci","Otvoreni devizni akreditiv u domaćoj banci"
"1041","kp_rrif1041","l10n_hr_chart_template_rrif","asset_current",,"Otvoreni akreditiv u inozemnoj banci","Otvoreni akreditiv u inozemnoj banci"
"1042","kp_rrif1042","l10n_hr_chart_template_rrif","asset_current",,"Ecsrow račun","Ecsrow račun"
"1050","kp_rrif1050","l10n_hr_chart_template_rrif","asset_current",,"Glavna devizna blagajna","Glavna devizna blagajna"
"1051","kp_rrif1051","l10n_hr_chart_template_rrif","asset_current",,"Devizna blagajna za službena putovanja u inozemstvo","Devizna blagajna za službena putovanja u inozemstvo"
"1052","kp_rrif1052","l10n_hr_chart_template_rrif","asset_current",,"Devizna blagajna za troškove prijevoza robe u inozemstvo","Devizna blagajna za troškove prijevoza robe u inozemstvo"
"1053","kp_rrif1053","l10n_hr_chart_template_rrif","asset_current",,"Devizna blagajna za mjenjačke poslove","Devizna blagajna za mjenjačke poslove"
"1055","kp_rrif1055","l10n_hr_chart_template_rrif","asset_current",,"Devizna blagajna za razne isplate","Devizna blagajna za razne isplate"
"1060","kp_rrif1060","l10n_hr_chart_template_rrif","asset_current",,"Novac za kupnju deviza-a1","Novac za kupnju deviza-Analitika 1"
"1080","kp_rrif1080","l10n_hr_chart_template_rrif","asset_current",,"Ostala novčana sredstva-a1","Ostala novčana sredstva-Analitika 1"
"1090","kp_rrif1090","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađenje depozita u bankama-a1","Vrijednosno usklađenje depozita u bankama-Analitika 1"
"1100","kp_rrif1100","l10n_hr_chart_template_rrif","asset_current",,"Udjeli u povezanim društvima (s više od 20%)","Udjeli u povezanim društvima (s više od 20%)"
"1101","kp_rrif1101","l10n_hr_chart_template_rrif","asset_current",,"Dionički udio u d.d. (s više od 20%)","Dionički udio u d.d. (s više od 20%)"
"1102","kp_rrif1102","l10n_hr_chart_template_rrif","asset_current",,"Ulaganje u dionice radi preprodaje (iz udjela više od 20%)","Ulaganje u dionice radi preprodaje (iz udjela više od 20%)"
"1110","kp_rrif1110","l10n_hr_chart_template_rrif","asset_current",,"Kratkoročni zajam povezanom društvu","Kratkoročni zajam povezanom društvu"
"1111","kp_rrif1111","l10n_hr_chart_template_rrif","asset_current",,"Kratkoročni zajam povezanom društvu u inozemstvu","Kratkoročni zajam povezanom društvu u inozemstvu"
"1112","kp_rrif1112","l10n_hr_chart_template_rrif","asset_current",,"Zajmovi dani vlastitim podružnicama","Zajmovi dani vlastitim podružnicama"
"1113","kp_rrif1113","l10n_hr_chart_template_rrif","asset_current",,"Zajmovi dani ustanovama i školama (kojih je društvo osnivač)","Zajmovi dani ustanovama i školama (kojih je društvo osnivač)"
"1120","kp_rrif1120","l10n_hr_chart_template_rrif","asset_current",,"Udjeli (do 20%) u društvima kapitala","Udjeli (do 20%) u društvima kapitala"
"1121","kp_rrif1121","l10n_hr_chart_template_rrif","asset_current",,"Udjeli - dionice u bankama","Udjeli - dionice u bankama"
"1122","kp_rrif1122","l10n_hr_chart_template_rrif","asset_current",,"Udjeli (do 20%) u ustanovama, zadrugama i dr.","Udjeli (do 20%) u ustanovama, zadrugama i dr."
"1130","kp_rrif1130","l10n_hr_chart_template_rrif","asset_current",,"Zajmovi u novcu","Zajmovi u novcu"
"1131","kp_rrif1131","l10n_hr_chart_template_rrif","asset_current",,"Zajmovi iz preuzetog duga","Zajmovi iz preuzetog duga"
"1132","kp_rrif1132","l10n_hr_chart_template_rrif","asset_current",,"Zajmovi iz dospjelih anuiteta u razdoblju do 12 mj. od dospijeća","Zajmovi iz dospjelih anuiteta u razdoblju do 12 mj. od dospijeća"
"1133","kp_rrif1133","l10n_hr_chart_template_rrif","asset_current",,"Ostali zajmovi društvima u kojima se drži udjel do 20%","Ostali zajmovi društvima u kojima se drži udjel do 20%"
"1140","kp_rrif1140","l10n_hr_chart_template_rrif","asset_current",,"Čekovi","Čekovi"
"11410","kp_rrif11410","l10n_hr_chart_template_rrif","asset_current",,"Mjenice u portfelju","Mjenice u portfelju"
"11411","kp_rrif11411","l10n_hr_chart_template_rrif","asset_current",,"Mjenice na naplati","Mjenice na naplati"
"11412","kp_rrif11412","l10n_hr_chart_template_rrif","asset_current",,"Mjenice u protestu","Mjenice u protestu"
"11413","kp_rrif11413","l10n_hr_chart_template_rrif","asset_current",,"Utužene mjenice","Utužene mjenice"
"1142","kp_rrif1142","l10n_hr_chart_template_rrif","asset_current",,"Komercijalni zapisi","Komercijalni zapisi"
"1143","kp_rrif1143","l10n_hr_chart_template_rrif","asset_current",,"Kratkotrajne obveze poduzetnika","Kratkotrajne obveze poduzetnika"
"1144","kp_rrif1144","l10n_hr_chart_template_rrif","asset_current",,"Blagajnički zapisi (izdani od banaka)","Blagajnički zapisi (izdani od banaka)"
"1145","kp_rrif1145","l10n_hr_chart_template_rrif","asset_current",,"Ulaganje u obveznice","Ulaganje u obveznice"
"1146","kp_rrif1146","l10n_hr_chart_template_rrif","asset_current",,"Zadužnice (iskupljene)-trađbina s temelja jamstva","Zadužnice (iskupljene)-trađbina s temelja jamstva"
"1147","kp_rrif1147","l10n_hr_chart_template_rrif","asset_current",,"Ulaganje u vrijednosne papire (namjenjene za trgovanje)","Ulaganje u vrijednosne papire (namjenjene za trgovanje)"
"1148","kp_rrif1148","l10n_hr_chart_template_rrif","asset_current",,"Ostali brzounovčivi vrijednosni papiri (robni papiri, ostale obveznice)","Ostali brzounovčivi vrijednosni papiri (robni papiri, ostale obveznice)"
"1149","kp_rrif1149","l10n_hr_chart_template_rrif","asset_current",,"Predani vrijednosni papiri na naplatu","Predani vrijednosni papiri na naplatu"
"11500","kp_rrif11500","l10n_hr_chart_template_rrif","asset_current",,"Zajmovi poduzetnicima","Zajmovi poduzetnicima"
"11501","kp_rrif11501","l10n_hr_chart_template_rrif","asset_current",,"Zajmovi ortacima","Zajmovi ortacima"
"11502","kp_rrif11502","l10n_hr_chart_template_rrif","asset_current",,"Zajmovi obrtnicima","Zajmovi obrtnicima"
"11503","kp_rrif11503","l10n_hr_chart_template_rrif","asset_current",,"Zajmovi podružnicama","Zajmovi podružnicama"
"11504","kp_rrif11504","l10n_hr_chart_template_rrif","asset_current",,"Zajmovi ustanovama","Zajmovi ustanovama"
"11505","kp_rrif11505","l10n_hr_chart_template_rrif","asset_current",,"Zajmovi dani u inozemstvo","Zajmovi dani u inozemstvo"
"11506","kp_rrif11506","l10n_hr_chart_template_rrif","asset_current",,"Zajmovi članovima uprave i zaposlenicima","Zajmovi članovima uprave i zaposlenicima"
"11507","kp_rrif11507","l10n_hr_chart_template_rrif","asset_current",,"Zajmovi dani poljoprivrednicima ili zadrugarima","Zajmovi dani poljoprivrednicima ili zadrugarima"
"11508","kp_rrif11508","l10n_hr_chart_template_rrif","asset_current",,"Dospjeli anuiteti dugotrajnih zajmova (naplaćuju se u roku od 12 mjeseci - analitika po korisnicima)","Dospjeli anuiteti dugotrajnih zajmova (koji se naplaćuju u roku od 12 mjeseci - analitika po korisnicima)"
"11509","kp_rrif11509","l10n_hr_chart_template_rrif","asset_current",,"Ostali kratkotrajni zajmovi","Ostali kratkotrajni zajmovi"
"11510","kp_rrif11510","l10n_hr_chart_template_rrif","asset_current",,"Depoziti u bankama","Depoziti u bankama"
"11511","kp_rrif11511","l10n_hr_chart_template_rrif","asset_current",,"Depoziti u osiguravajućim društvima","Depoziti u osiguravajućim društvima"
"11512","kp_rrif11512","l10n_hr_chart_template_rrif","asset_current",,"Depoziti u inozemnim financijskim institucijama","Depoziti u inozemnim financijskim institucijama"
"11513","kp_rrif11513","l10n_hr_chart_template_rrif","asset_current",,"Depoziti za ostale poslovne aktivnosti","Depoziti za ostale poslovne aktivnosti"
"11520","kp_rrif11520","l10n_hr_chart_template_rrif","asset_current",,"Kaucije na aukcijama (dražbama)","Kaucije na aukcijama (dražbama)"
"11521","kp_rrif11521","l10n_hr_chart_template_rrif","asset_current",,"Kaucije za robu","Kaucije za robu"
"11522","kp_rrif11522","l10n_hr_chart_template_rrif","asset_current",,"Kaucije za ambalažu","Kaucije za ambalažu"
"11523","kp_rrif11523","l10n_hr_chart_template_rrif","asset_current",,"Dane jamčevine za natječaje","Dane jamčevine za natječaje"
"11524","kp_rrif11524","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za kapare (čl. 303. ZOO-a)","Potraživanja za kapare (čl. 303. ZOO-a)"
"11525","kp_rrif11525","l10n_hr_chart_template_rrif","asset_current",,"Polozi gotovine za ostale poslovne aktivnosti","Polozi gotovine za ostale poslovne aktivnosti"
"1160","kp_rrif1160","l10n_hr_chart_template_rrif","asset_current",,"Kratkotrajno ulaganje u novčane fondove","Kratkotrajno ulaganje u novčane fondove"
"1161","kp_rrif1161","l10n_hr_chart_template_rrif","asset_current",,"Kratkotrajno ulaganje u ostale investicijske fondove","Kratkotrajno ulaganje u ostale investicijske fondove"
"1170","kp_rrif1170","l10n_hr_chart_template_rrif","asset_current",,"Otkup kratkoročnih potraživanja (faktoring)","Otkup kratkoročnih potraživanja (faktoring)"
"1171","kp_rrif1171","l10n_hr_chart_template_rrif","asset_current",,"Ulaganje u kratkotrajne eskontne poslove","Ulaganje u kratkotrajne eskontne poslove"
"1172","kp_rrif1172","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za više uplaćeno po kreditnoj kartici","Potraživanja za više uplaćeno po kreditnoj kartici"
"1173","kp_rrif1173","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za isplate avaliranih ili indosiranih mjenica","Potraživanja za isplate avaliranih ili indosiranih mjenica"
"1174","kp_rrif1174","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja po isplaćenim garancijama","Potraživanja po isplaćenim garancijama"
"1175","kp_rrif1175","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja iz preuzetog duga (čl. 96. ZOO)","Potraživanja iz preuzetog duga (čl. 96. ZOO)"
"1176","kp_rrif1176","l10n_hr_chart_template_rrif","asset_current",,"Potraživanje po asignacijama, novacijama i sl.","Potraživanje po asignacijama, novacijama i sl."
"1180","kp_rrif1180","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja u sporu (npr. utužena, u stečaju i sl. iz fin. imovine)-a1","Potraživanja u sporu (npr. utužena, u stečaju i sl. iz fin. imovine)-Analitika 1"
"1190","kp_rrif1190","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađivanje financijske imovine - kratkotrajne (analitika po otpisima iz ove skupine rn)-a1","Vrijednosno usklađivanje financijske imovine - kratkotrajne (analitika po otpisima iz ove skupine računa)-Analitika 1"
"1200","kp_rrif1200","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja od kupaca dobara","Potraživanja od kupaca dobara"
"1201","kp_rrif1201","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja od kupaca usluga (servisne, najmovi, ustupanje radne snage, kapaciteta i dr.)","Potraživanja od kupaca usluga (servisne, najmovi, ustupanje radne snage, kapaciteta i dr.)"
"1202","kp_rrif1202","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za prodaju prava","Potraživanja za prodaju prava"
"1203","kp_rrif1203","l10n_hr_chart_template_rrif","asset_receivable","True","Kupci građani i prodaja na potrošački kredit","Kupci građani i prodaja na potrošački kredit"
"1204","kp_rrif1204","l10n_hr_chart_template_rrif","asset_receivable","True","Kupci imovinskih sredstava, inventara, materijala, otpadaka i sl.","Kupci imovinskih sredstava, inventara, materijala, otpadaka i sl."
"1205","kp_rrif1205","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja od kupaca za prodanu robu iz komisije","Potraživanja od kupaca za prodanu robu iz komisije"
"1206","kp_rrif1206","l10n_hr_chart_template_rrif","asset_receivable","True","Kupci zastupničke i franšizne prodaje","Kupci zastupničke i franšizne prodaje"
"1207","kp_rrif1207","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za prodaju na kreditne kartice","Potraživanja za prodaju na kreditne kartice"
"1208","kp_rrif1208","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja od ostalih prodaja","Potraživanja od ostalih prodaja"
"1209","kp_rrif1209","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za nefakturiranu isporuku dobara ili usluga","Potraživanja za nefakturiranu isporuku dobara ili usluga"
"1210","kp_rrif1210","l10n_hr_chart_template_rrif","asset_receivable","True","Kupci dobara iz inozemstva","Kupci dobara iz inozemstva"
"1211","kp_rrif1211","l10n_hr_chart_template_rrif","asset_receivable","True","Kupci usluga iz inozemstva","Kupci usluga iz inozemstva"
"1212","kp_rrif1212","l10n_hr_chart_template_rrif","asset_receivable","True","Kupci prava iz inozemstva","Kupci prava iz inozemstva"
"1213","kp_rrif1213","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja od kupaca dobara (PoS)","Potraživanja od kupaca dobara"
"1220","kp_rrif1220","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za prodaju - isporuku povezanim društvima","Potraživanja za prodaju - isporuku povezanim društvima"
"1221","kp_rrif1221","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za dividende od povezanih društava","Potraživanja za dividende od povezanih društava"
"1222","kp_rrif1222","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za isporuke povezanim društvima u inozemstvu","Potraživanja za isporuke povezanim društvima u inozemstvu"
"1223","kp_rrif1223","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za nadoknadu gubitka od povezanih društava (čl. 489. ZTD)","Potraživanja za nadoknadu gubitka od povezanih društava (čl. 489. ZTD)"
"1224","kp_rrif1224","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za kamate od povezanih društava","Potraživanja za kamate od povezanih društava"
"1225","kp_rrif1225","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja od podružnica","Potraživanja od podružnica"
"1226","kp_rrif1226","l10n_hr_chart_template_rrif","asset_receivable","True","Ostala kratkoročna potraživanja od povezanih poduzetnika","Ostala kratkoročna potraživanja od povezanih poduzetnika"
"1227","kp_rrif1227","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za udio u dobitku d.o.o.-a","Potraživanja za udio u dobitku d.o.o.-a"
"1228","kp_rrif1228","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja iz ulaganja radi povećanja udjela (do upisa u t.k.)","Potraživanja iz ulaganja radi povećanja udjela (do upisa u t.k.)"
"1230","kp_rrif1230","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za isporuke dobara i usluga","Potraživanja za isporuke dobara i usluga"
"1231","kp_rrif1231","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za dividendu - dobitak","Potraživanja za dividendu - dobitak"
"1232","kp_rrif1232","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za kamate","Potraživanja za kamate"
"1233","kp_rrif1233","l10n_hr_chart_template_rrif","asset_receivable","True","Ostala potraživanja od sudjelujućih društava","Ostala potraživanja od sudjelujućih društava"
"1240","kp_rrif1240","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja od kupaca za ugovorene kamate (koje nisu pripisane glavnici)","Potraživanja od kupaca za ugovorene kamate (koje nisu pripisane glavnici)"
"1241","kp_rrif1241","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za zatezne kamate (koje nisu pripisane glavnici)","Potraživanja za zatezne kamate (koje nisu pripisane glavnici)"
"1242","kp_rrif1242","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za kamate po nagodbama","Potraživanja za kamate po nagodbama"
"1243","kp_rrif1243","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanje za kamatu iz danih zajmova","Potraživanje za kamatu iz danih zajmova"
"1245","kp_rrif1245","l10n_hr_chart_template_rrif","asset_receivable","True","Kamate iz ostalih trađbina","Kamate iz ostalih trađbina"
"1250","kp_rrif1250","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za predujmove za usluge (koje nisu u svezi sa zalihama i dugotr. imov.)-a1","Potraživanja za predujmove za usluge (koje nisu u svezi sa zalihama i dugotr. imov.)-Analitika 1"
"1260","kp_rrif1260","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja po poslovima uvoza za tuđi račun","Potraživanja po poslovima uvoza za tuđi račun"
"1261","kp_rrif1261","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja od izvoznika","Potraživanja od izvoznika"
"1270","kp_rrif1270","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja s osnove prodaje udjela i dionica","Potraživanja s osnove prodaje udjela i dionica"
"1271","kp_rrif1271","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za nakladno odobrene popuste (bonifikacije, casa-sconto, rabat i sl.)","Potraživanja za nakladno odobrene popuste (bonifikacije, casa-sconto, rabat i sl.)"
"1272","kp_rrif1272","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za predujam za kupnju dionica i udjela","Potraživanja za predujam za kupnju dionica i udjela"
"1273","kp_rrif1273","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja od komisionara","Potraživanja od komisionara"
"1280","kp_rrif1280","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja iz odštetnih zahtjeva (od osiguravajućih društava)","Potraživanja iz odštetnih zahtjeva (od osiguravajućih društava)"
"1281","kp_rrif1281","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za udjel u dobitku - prihodu u investicijskom fondu","Potraživanja za udjel u dobitku - prihodu u investicijskom fondu"
"1282","kp_rrif1282","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za tantijeme (nadoknade za korištenje patenta, znaka i autorskih prava)","Potraživanja za tantijeme (nadoknade za korištenje patenta, znaka i autorskih prava)"
"1283","kp_rrif1283","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja stečena cesijom, asignacijom i preuzimanjem duga od prodaje","Potraživanja stečena cesijom, asignacijom i preuzimanjem duga od prodaje"
"1284","kp_rrif1284","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za nadoknadu troškova iz jamstva","Potraživanja za nadoknadu troškova iz jamstva"
"1285","kp_rrif1285","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja od kooperanata, zadrugara, ustanove","Potraživanja od kooperanata, zadrugara, ustanove"
"1286","kp_rrif1286","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja od članova društva za pokriće gubitka","Potraživanja od članova društva za pokriće gubitka"
"1287","kp_rrif1287","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za dana jamstva i činidbe","Potraživanja za dana jamstva i činidbe"
"1288","kp_rrif1288","l10n_hr_chart_template_rrif","asset_receivable","True","Potraživanja za poreze iz poslovnih odnosa","Potraživanja za poreze iz poslovnih odnosa"
"1289","kp_rrif1289","l10n_hr_chart_template_rrif","asset_receivable","True","Ostala potraživanja","Ostala potraživanja"
"1290","kp_rrif1290","l10n_hr_chart_template_rrif","asset_receivable","True","Vrijednosno usklađenje potraživanja od kupaca","Vrijednosno usklađenje potraživanja od kupaca"
"1291","kp_rrif1291","l10n_hr_chart_template_rrif","asset_receivable","True","Vrijednosno usklađenje potraživanja od povezanih društava","Vrijednosno usklađenje potraživanja od povezanih društava"
"1292","kp_rrif1292","l10n_hr_chart_template_rrif","asset_receivable","True","Vrijednosno usklađenje od poduzetnika iz sudjelujućih interesa","Vrijednosno usklađenje od poduzetnika iz sudjelujućih interesa"
"1293","kp_rrif1293","l10n_hr_chart_template_rrif","asset_receivable","True","Vrijednosno usklađenje kamata","Vrijednosno usklađenje kamata"
"1294","kp_rrif1294","l10n_hr_chart_template_rrif","asset_receivable","True","Vrijednosno usklađenje za dane predujmove za usluge","Vrijednosno usklađenje za dane predujmove za usluge"
"1295","kp_rrif1295","l10n_hr_chart_template_rrif","asset_receivable","True","Vrijednosno usklađenje ostalih potraživanja (računi 126 do 128)","Vrijednosno usklađenje ostalih potraživanja (računi 126 do 128)"
"1300","kp_rrif1300","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja od zaposlenih za više isplaćenu plaću","Potraživanja od zaposlenih za više isplaćenu plaću"
"1301","kp_rrif1301","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za isplaćeni predujam za službeni put","Potraživanja za isplaćeni predujam za službeni put"
"1302","kp_rrif1302","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za dane novčane svote za nabave u gotovini (za tržišni nakup, za karnete i dr.)","Potraživanja za dane novčane svote za nabave u gotovini (za tržišni nakup, za karnete i dr.)"
"1303","kp_rrif1303","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za manjkove i učinjene štete (s PDV-om)","Potraživanja za manjkove i učinjene štete (s PDV-om)"
"1304","kp_rrif1304","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za prehranu, kazne, za korištenje odmarališta i sl.","Potraživanja za prehranu, kazne, za korištenje odmarališta i sl."
"1305","kp_rrif1305","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja od zaposlenih za manje plaćene poreze i doprinose","Potraživanja od zaposlenih za manje plaćene poreze i doprinose"
"1306","kp_rrif1306","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja od zaposlenika za plaćene privatne troškove (npr. po službenoj kred. kartici)","Potraživanja od zaposlenika za plaćene privatne troškove (npr. po službenoj kred. kartici)"
"1307","kp_rrif1307","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja od zaposlenika za primitak u naravi","Potraživanja od zaposlenika za primitak u naravi"
"1309","kp_rrif1309","l10n_hr_chart_template_rrif","asset_current",,"Ostala potraživanja od zaposlenih","Ostala potraživanja od zaposlenih"
"1310","kp_rrif1310","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za predujmljene honorare","Potraživanja za predujmljene honorare"
"1311","kp_rrif1311","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za isplaćene svote","Potraživanja za isplaćene svote"
"1319","kp_rrif1319","l10n_hr_chart_template_rrif","asset_current",,"Ostala potraživanja od vanjskih suradnika","Ostala potraživanja od vanjskih suradnika"
"1330","kp_rrif1330","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja od članova društva za predujmljeni dobitak - dividendu","Potraživanja od članova društva za predujmljeni dobitak - dividendu"
"1331","kp_rrif1331","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja od članova društva za privatne troškove","Potraživanja od članova društva za privatne troškove"
"1332","kp_rrif1332","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja od članova ustanove","Potraživanja od članova ustanove"
"1333","kp_rrif1333","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja od članova zadruge","Potraživanja od članova zadruge"
"1340","kp_rrif1340","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja u sporu i rizična potraživanja (iz skupine 12 i 13)-a1","Potraživanja u sporu i rizična potraživanja (iz skupine 12 i 13)-Analitika 1"
"1350","kp_rrif1350","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za sredstva u slobodnoj zoni-a1","Potraživanja za sredstva u slobodnoj zoni-Analitika 1"
"1360","kp_rrif1360","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja od banaka za prodaju na potrošački kredit-a1","Potraživanja od banaka za prodaju na potrošački kredit-Analitika 1"
"1370","kp_rrif1370","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za isporučena dobra i usluge","Potraživanja za isporučena dobra i usluge"
"1371","kp_rrif1371","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za doznake novca, za dobitak i dr.","Potraživanja za doznake novca, za dobitak i dr."
"1380","kp_rrif1380","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za naknadne popuste","Potraživanja za naknadne popuste"
"1381","kp_rrif1381","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja od ortaka","Potraživanja od ortaka"
"1382","kp_rrif1382","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja od zastupnika","Potraživanja od zastupnika"
"1389","kp_rrif1389","l10n_hr_chart_template_rrif","asset_current",,"Ostala poslovna potraživanja","Ostala poslovna potraživanja"
"1390","kp_rrif1390","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađenje potraživanja od zaposlenih članova društva i ostalih potraživanja-a1","Vrijednosno usklađenje potraživanja od zaposlenih članova društva i ostalih potraživanja-Analitika 1"
"14000","kp_rrif14000","l10n_hr_chart_template_rrif","asset_current",,"Pretporez - 10%","Pretporez - 10%"
"14001","kp_rrif14001","l10n_hr_chart_template_rrif","asset_current",,"Pretporez - 22%","Pretporez - 22%"
"14002","kp_rrif14002","l10n_hr_chart_template_rrif","asset_current",,"Pretporez - 23%","Pretporez - 23%"
"14003","kp_rrif14003","l10n_hr_chart_template_rrif","asset_current",,"Pretporez - 25%","Pretporez - 25%"
"14010","kp_rrif14010","l10n_hr_chart_template_rrif","asset_current",,"Pretporez iz predujmova -10%","Pretporez iz predujmova -10%"
"14011","kp_rrif14011","l10n_hr_chart_template_rrif","asset_current",,"Pretporez iz predujmova - 22%","Pretporez iz predujmova - 22%"
"14012","kp_rrif14012","l10n_hr_chart_template_rrif","asset_current",,"Pretporez iz predujmova - 23%","Pretporez iz predujmova - 23%"
"14013","kp_rrif14013","l10n_hr_chart_template_rrif","asset_current",,"Pretporez iz predujmova - 25%","Pretporez iz predujmova - 25%"
"14020","kp_rrif14020","l10n_hr_chart_template_rrif","asset_current",,"Plaćeni PDV pri uvozu dobara - 10%","Plaćeni PDV pri uvozu dobara - 10%"
"14021","kp_rrif14021","l10n_hr_chart_template_rrif","asset_current",,"Plaćeni PDV pri uvozu dobara - 22%","Plaćeni PDV pri uvozu dobara - 22%"
"14022","kp_rrif14022","l10n_hr_chart_template_rrif","asset_current",,"Plaćeni PDV pri uvozu dobara - 23%","Plaćeni PDV pri uvozu dobara - 23%"
"14023","kp_rrif14023","l10n_hr_chart_template_rrif","asset_current",,"Plaćeni PDV pri uvozu dobara - 25%","Plaćeni PDV pri uvozu dobara - 25%"
"14030","kp_rrif14030","l10n_hr_chart_template_rrif","asset_current",,"Plaćeni PDV na usluge inozemnih poduzetnika - 10%","Plaćeni PDV na usluge inozemnih poduzetnika - 10%"
"14031","kp_rrif14031","l10n_hr_chart_template_rrif","asset_current",,"Plaćeni PDV na usluge inozemnih poduzetnika - 22%","Plaćeni PDV na usluge inozemnih poduzetnika - 22%"
"14032","kp_rrif14032","l10n_hr_chart_template_rrif","asset_current",,"Plaćeni PDV na usluge inozemnih poduzetnika - 23%","Plaćeni PDV na usluge inozemnih poduzetnika - 23%"
"14033","kp_rrif14033","l10n_hr_chart_template_rrif","asset_current",,"Plaćeni PDV na usluge inozemnih poduzetnika - 25%","Plaćeni PDV na usluge inozemnih poduzetnika - 25%"
"14040","kp_rrif14040","l10n_hr_chart_template_rrif","asset_current",,"Ispravak pretporeza zbog promjene postotka priznavanja PDV-a","Ispravak pretporeza zbog promjene postotka priznavanja PDV-a"
"1406","kp_rrif1406","l10n_hr_chart_template_rrif","asset_current",,"Pretporez stečen po ugovoru o asignaciji ili iz preknjižavanja","Pretporez stečen po ugovoru o asignaciji ili iz preknjižavanja"
"1407","kp_rrif1407","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za razliku većeg pretporeza od obveze u obračunskom razdoblju","Potraživanja za razliku većeg pretporeza od obveze u obračunskom razdoblju"
"1408","kp_rrif1408","l10n_hr_chart_template_rrif","asset_current",,"Pretporez koji još nije priznan (uključivo i neplaćeni R-2)","Pretporez koji još nije priznan (uključivo i neplaćeni R-2)"
"1409","kp_rrif1409","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za više plaćeni PDV po konačnom obračunu","Potraživanja za više plaćeni PDV po konačnom obračunu"
"1410","kp_rrif1410","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za porez na dohodak iz plaća","Potraživanja za porez na dohodak iz plaća"
"1411","kp_rrif1411","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za prirez iz plaća","Potraživanja za prirez iz plaća"
"1412","kp_rrif1412","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za porez na dohodak iz autorskih prava, ugovora o djelu, članova nadz. odbora i dr.doh.","Potraživanja za porez na dohodak iz autorskih prava, ugovora o djelu, dohodaka članova nadzornog odbora i drugih dohodaka"
"1413","kp_rrif1413","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za porez i prirez na stipendije i nagrade učenika i studenta","Potraživanja za porez i prirez na stipendije i nagrade učenika i studenta"
"1414","kp_rrif1414","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za poreze iz drugih dohodaka","Potraživanja za poreze iz drugih dohodaka"
"1417","kp_rrif1417","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za više plaćeni porez na dohodak od kapitala","Potraživanja za više plaćeni porez na dohodak od kapitala"
"1420","kp_rrif1420","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za više plaćene doprinose za MO iz plaće","Potraživanja za više plaćene doprinose za MO iz plaće"
"1421","kp_rrif1421","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja od MO za više plaćeni doprinos za beneficirani staž","Potraživanja od MO za više plaćeni doprinos za beneficirani staž"
"1422","kp_rrif1422","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za više plaćeni doprinos za zdravstveno osiguranje na plaće","Potraživanja za više plaćeni doprinos za zdravstveno osiguranje na plaće"
"1423","kp_rrif1423","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za više plaćeni dopr. za zdrav. osigur. za slučaj ozljede na radu i prof. bolesti","Potraživanja za više plaćeni dopr. za zdrav. osigur. za slučaj ozljede na radu i prof. bolesti"
"1424","kp_rrif1424","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za više plaćeni doprinos za zapošljavanje na plaće","Potraživanja za više plaćeni doprinos za zapošljavanje na plaće"
"1429","kp_rrif1429","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za ostale nespomenute doprinose","Potraživanja za ostale nespomenute doprinose"
"1430","kp_rrif1430","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za plaćene predujmove poreza na dobitak","Potraživanja za plaćene predujmove poreza na dobitak"
"1432","kp_rrif1432","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za više plaćeni porez po odbitku (na inozemne usluge)","Potraživanja za više plaćeni porez po odbitku (na inozemne usluge)"
"1433","kp_rrif1433","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za porez na dobitak stečen po ugovoru o cesiji ili iz preknjižavanja","Potraživanja za porez na dobitak stečen po ugovoru o cesiji ili iz preknjižavanja"
"1434","kp_rrif1434","l10n_hr_chart_template_rrif","asset_current",,"Porez na dobitak plaćen u inozemstvu","Porez na dobitak plaćen u inozemstvu"
"1440","kp_rrif1440","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za poseban porez na kavu","Potraživanja za poseban porez na kavu"
"1441","kp_rrif1441","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za poseban porez na bezalkoholna pića","Potraživanja za poseban porez na bezalkoholna pića"
"1442","kp_rrif1442","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za poseban porez na alkoholna pića","Potraživanja za poseban porez na alkoholna pića"
"1443","kp_rrif1443","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za poseban porez na pivo","Potraživanja za poseban porez na pivo"
"1444","kp_rrif1444","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za poseban porez na duhanske proizvode","Potraživanja za poseban porez na duhanske proizvode"
"1445","kp_rrif1445","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za poseban porez na naftne derivate i naknade za ceste","Potraživanja za poseban porez na naftne derivate i naknade za ceste"
"1446","kp_rrif1446","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za poseban porez na osobne automobile, motocikle, ostala mot. vozila, plovila i zrakop.","Potraživanja za poseban porez na osobne automobile, motocikle, ostala motorna vozila, plovila i zrakoplove"
"1447","kp_rrif1447","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za poseban porez na luksuzne proizvode","Potraživanja za poseban porez na luksuzne proizvode"
"1448","kp_rrif1448","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za porez na promet nekretnina","Potraživanja za porez na promet nekretnina"
"1449","kp_rrif1449","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za 5% poreza na promet motornih vozila i plovila","Potraživanja za 5% poreza na promet motornih vozila i plovila"
"1450","kp_rrif1450","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za plaćenu članarinu turističkim zajed.-a1","Potraživanja za plaćenu članarinu turističkim zajed.-Analitika 1"
"1460","kp_rrif1460","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za članarine komori (HGK ili HOK)-a1","Potraživanja za članarine komori (HGK ili HOK)-Analitika 1"
"1470","kp_rrif1470","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za carinu i više plaćene carinske pristojbe-a1","Potraživanja za carinu i više plaćene carinske pristojbe-Analitika 1"
"1480","kp_rrif1480","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za više plaćeni porez na motorna vozila i plovne objekte","Potraživanja za više plaćeni porez na motorna vozila i plovne objekte"
"1481","kp_rrif1481","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za porez na kuće za odmor i korištenje javnih površina","Potraživanja za porez na kuće za odmor i korištenje javnih površina"
"1482","kp_rrif1482","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za porez na reklamu","Potraživanja za porez na reklamu"
"1483","kp_rrif1483","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za porez na tvrtku ili naziv","Potraživanja za porez na tvrtku ili naziv"
"1484","kp_rrif1484","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za porez na potrošnju u ugostiteljstvu","Potraživanja za porez na potrošnju u ugostiteljstvu"
"1485","kp_rrif1485","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za plaćeni porez na priređivanje zabavnih i športskih priredbi","Potraživanja za plaćeni porez na priređivanje zabavnih i športskih priredbi"
"1486","kp_rrif1486","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za plaćeni porez na nasljedstva i darove","Potraživanja za plaćeni porez na nasljedstva i darove"
"1487","kp_rrif1487","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za više plaćeni porez na neiskorištene nekretnine","Potraživanja za više plaćeni porez na neiskorištene nekretnine"
"1489","kp_rrif1489","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za ostale poreze županije (grada), općine","Potraživanja za ostale poreze županije (grada), općine"
"1490","kp_rrif1490","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za više plaćenu nadoknadu za šume","Potraživanja za više plaćenu nadoknadu za šume"
"1491","kp_rrif1491","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za više plaćene kazne i sl.","Potraživanja za više plaćene kazne i sl."
"1492","kp_rrif1492","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za više uplaćene naknade za iskorištavanje mineralnih sirovina","Potraživanja za više uplaćene naknade za iskorištavanje mineralnih sirovina"
"1493","kp_rrif1493","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za više plaćene naknade za koncesije","Potraživanja za više plaćene naknade za koncesije"
"1495","kp_rrif1495","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za spomeničku rentu","Potraživanja za spomeničku rentu"
"1497","kp_rrif1497","l10n_hr_chart_template_rrif","asset_current",,"Potraživanje od Fonda za razvoj i sl.","Potraživanje od Fonda za razvoj i sl."
"1499","kp_rrif1499","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja za ostala plaćena davanja državi i državnim institucijama","Potraživanja za ostala plaćena davanja državi i državnim institucijama"
"1500","kp_rrif1500","l10n_hr_chart_template_rrif","asset_current",,"Potraživanje za nadoknade bolovanja od HZZO-a1","Potraživanje za nadoknade bolovanja od HZZO-Analitika 1"
"1510","kp_rrif1510","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja od mirovinskog osiguranja-a1","Potraživanja od mirovinskog osiguranja-Analitika 1"
"1520","kp_rrif1520","l10n_hr_chart_template_rrif","asset_current",,"Potraživanja od lokalne samouprave-a1","Potraživanja od lokalne samouprave-Analitika 1"
"1530","kp_rrif1530","l10n_hr_chart_template_rrif","asset_current",,"Potraživ. za regrese, premije, stimulac. i držav. potpore-a1","Potraživ. za regrese, premije, stimulac. i držav. potpore-Analitika 1"
"1540","kp_rrif1540","l10n_hr_chart_template_rrif","asset_current",,"Potraživanje od Fonda za otkupljenu ambalažu-a1","Potraživanje od Fonda za otkupljenu ambalažu-Analitika 1"
"1550","kp_rrif1550","l10n_hr_chart_template_rrif","asset_current",,"Ostala potraživanja od državnih institucija-a1","Ostala potraživanja od državnih institucija-Analitika 1"
"1590","kp_rrif1590","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađenje potraživanja od države i drugih institucija-a1","Vrijednosno usklađenje potraživanja od države i drugih institucija-Analitika 1"
"1900","kp_rrif1900","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed plaćeni troškovi održavanja, opreme, postrojenja i građevina","Unaprijed plaćeni troškovi održavanja, opreme, postrojenja i građevina"
"19010","kp_rrif19010","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed plaćena zakupnina","Unaprijed plaćena zakupnina"
"19011","kp_rrif19011","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed plaćeni a nepriznati PDV na zakupninu (npr. 30% od leasinga osob. aut.)","Unaprijed plaćeni a nepriznati PDV na zakupninu (npr. 30% od leasinga osob. aut.)"
"1902","kp_rrif1902","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed plaćeni troškovi reklame, propagande i sajmova","Unaprijed plaćeni troškovi reklame, propagande i sajmova"
"1903","kp_rrif1903","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed plaćeni troškovi energije za sljedeće razdoblje","Unaprijed plaćeni troškovi energije za sljedeće razdoblje"
"1904","kp_rrif1904","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed plaćeni troškovi osig. imovine i osoba na opasnim poslovima ili putnika u prometu i sl.","Unaprijed plaćeni troškovi osiguranja imovine i osoba koje rade na opasnim poslovima ili putnika u prometu i sl."
"1905","kp_rrif1905","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed plaćene kamate na bankovna jamstava i sl.","Unaprijed plaćene kamate na bankovna jamstava i sl."
"1906","kp_rrif1906","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed plaćeni troškovi reprezentacije","Unaprijed plaćeni troškovi reprezentacije"
"1907","kp_rrif1907","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed plaćene pretplate na službena glasila i stručne časopise","Unaprijed plaćene pretplate na službena glasila i stručne časopise"
"1908","kp_rrif1908","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed isplaćene plaće za buduće razdoblje","Unaprijed isplaćene plaće za buduće razdoblje"
"1909","kp_rrif1909","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed plaćeni ostali troškovi posl.(prijevoza, bank. usluge, zdr. zaštita,autorski,rad po ug. i sl)","Unaprijed plaćeni ostali troškovi poslovanja (troškovi prijevoza, bankovne usluge, zdravstvena zaštita, autorski honorari, rad po ugovoru i sl.)"
"1910","kp_rrif1910","l10n_hr_chart_template_rrif","asset_current",,"Obračunani prihodi (budućeg razdoblja)-a1","Obračunani prihodi (budućeg razdoblja)-Analitika 1"
"1920","kp_rrif1920","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed plaćeni ovisni troškovi nabave (koji nisu na 651)-a1","Unaprijed plaćeni ovisni troškovi nabave (koji nisu na 651)-Analitika 1"
"1930","kp_rrif1930","l10n_hr_chart_template_rrif","asset_current",,"Troškovi kamata iz budućeg razdoblja-a1","Troškovi kamata iz budućeg razdoblja-Analitika 1"
"1940","kp_rrif1940","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed plaćeni troškovi koncesija (za razdoblje do 12 mj.)-a1","Unaprijed plaćeni troškovi koncesija (za razdoblje do 12 mj.)-Analitika 1"
"1950","kp_rrif1950","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed plaćene franšize, trgovačko znakovlje, prava i sl. (do 12 mj.)-a1","Unaprijed plaćene franšize, trgovačko znakovlje, prava i sl. (do 12 mj.)-Analitika 1"
"1960","kp_rrif1960","l10n_hr_chart_template_rrif","asset_current",,"Unaprijed plaćene licencije i patenti (do 12 mj.)-a1","Unaprijed plaćene licencije i patenti (do 12 mj.)-Analitika 1"
"1999","kp_rrif1999","l10n_hr_chart_template_rrif","asset_current",,"Ostala aktivna vremenska razgraničenja","Ostala aktivna vremenska razgraničenja"
"2000","kp_rrif2000","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema povezanim društvima za zalihe (poluproizvoda, robe i sl.)","Obveze prema povezanim društvima za zalihe (poluproizvoda, robe i sl.)"
"2001","kp_rrif2001","l10n_hr_chart_template_rrif","liability_current",,"Obveze za raspoređeni dobitak iz poslovanja prema povezanim društvima","Obveze za raspoređeni dobitak iz poslovanja prema povezanim društvima"
"2002","kp_rrif2002","l10n_hr_chart_template_rrif","liability_current",,"Obveze za zajmove prema povezanim društvima","Obveze za zajmove prema povezanim društvima"
"2003","kp_rrif2003","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema podružnicama","Obveze prema podružnicama"
"2004","kp_rrif2004","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema osnovanim ustanovama, zadrugama i sl.","Obveze prema osnovanim ustanovama, zadrugama i sl."
"2005","kp_rrif2005","l10n_hr_chart_template_rrif","liability_current",,"Obveze za predujmove od povezanih društvima","Obveze za predujmove od povezanih društvima"
"2008","kp_rrif2008","l10n_hr_chart_template_rrif","liability_current",,"Obveze za kamate prema povezanim društvima","Obveze za kamate prema povezanim društvima"
"2009","kp_rrif2009","l10n_hr_chart_template_rrif","liability_current",,"Ostale kratkoročne obveze prema povezanim društvima","Ostale kratkoročne obveze prema povezanim društvima"
"2010","kp_rrif2010","l10n_hr_chart_template_rrif","liability_current",,"Obveze s osnove udjela u dobitku (analitika prema članovima)","Obveze s osnove udjela u dobitku (analitika prema članovima)"
"2011","kp_rrif2011","l10n_hr_chart_template_rrif","liability_current",,"Obveze za dividende dioničarima","Obveze za dividende dioničarima"
"2012","kp_rrif2012","l10n_hr_chart_template_rrif","liability_current",,"Obveze za sudjelujuće dobitke u rezultatu iz zajedničkog pothvata","Obveze za sudjelujuće dobitke u rezultatu iz zajedničkog pothvata"
"2013","kp_rrif2013","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema zadrugarima iz rezultata","Obveze prema zadrugarima iz rezultata"
"2014","kp_rrif2014","l10n_hr_chart_template_rrif","liability_current",,"Obveze iz dobitka prema tajnim članovima","Obveze iz dobitka prema tajnim članovima"
"2015","kp_rrif2015","l10n_hr_chart_template_rrif","liability_current",,"Ostale obveze s osnove udjela u dobitku","Ostale obveze s osnove udjela u dobitku"
"2100","kp_rrif2100","l10n_hr_chart_template_rrif","liability_current",,"Obveze za izdane čekove-a1","Obveze za izdane čekove-Analitika 1"
"2110","kp_rrif2110","l10n_hr_chart_template_rrif","liability_current",,"Obveze za dane mjenice","Obveze za dane mjenice"
"2111","kp_rrif2111","l10n_hr_chart_template_rrif","liability_current",,"Obveze za dane mjenične akcepte","Obveze za dane mjenične akcepte"
"2120","kp_rrif2120","l10n_hr_chart_template_rrif","liability_current",,"Obveze po izdanim komercijalnim zapisima","Obveze po izdanim komercijalnim zapisima"
"2121","kp_rrif2121","l10n_hr_chart_template_rrif","liability_current",,"Obveze po izdanim obveznicama","Obveze po izdanim obveznicama"
"2122","kp_rrif2122","l10n_hr_chart_template_rrif","liability_current",,"Obveze po izdanim zadužnicama","Obveze po izdanim zadužnicama"
"2123","kp_rrif2123","l10n_hr_chart_template_rrif","liability_current",,"Obveze po ostalim kratkoročnim vrijednosnim papirima","Obveze po ostalim kratkoročnim vrijednosnim papirima"
"2128","kp_rrif2128","l10n_hr_chart_template_rrif","liability_current",,"Obveze za kamate sadržane u vrijed. papirima","Obveze za kamate sadržane u vrijed. papirima"
"2130","kp_rrif2130","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema poduzetnicima u kojima postoje sudjelujući interesi (do 20% udjela)-a1","Obveze prema poduzetnicima u kojima postoje sudjelujući interesi (do 20% udjela)-Analitika 1"
"2140","kp_rrif2140","l10n_hr_chart_template_rrif","liability_current",,"Obveze za financijske zajmove od društava","Obveze za financijske zajmove od društava"
"2141","kp_rrif2141","l10n_hr_chart_template_rrif","liability_current",,"Obveze za zajmove od ustanova, zadruga i sl.","Obveze za zajmove od ustanova, zadruga i sl."
"2142","kp_rrif2142","l10n_hr_chart_template_rrif","liability_current",,"Obveze za kratkoročne zajmove iz inozemstva","Obveze za kratkoročne zajmove iz inozemstva"
"2143","kp_rrif2143","l10n_hr_chart_template_rrif","liability_current",,"Obveze za dio dospjelih dugoročnih zajmova koji se trebaju platiti u roku 12 mj.","Obveze za dio dospjelih dugoročnih zajmova koji se trebaju platiti u roku 12 mj."
"2144","kp_rrif2144","l10n_hr_chart_template_rrif","liability_current",,"Obveze za zajmove prema građanima","Obveze za zajmove prema građanima"
"2145","kp_rrif2145","l10n_hr_chart_template_rrif","liability_current",,"Obveze za zajmove članova društva","Obveze za zajmove članova društva"
"2146","kp_rrif2146","l10n_hr_chart_template_rrif","liability_current",,"Obveze po kontokorentnom računu","Obveze po kontokorentnom računu"
"2147","kp_rrif2147","l10n_hr_chart_template_rrif","liability_current",,"Obveze za depozite, jamčevine i kapare","Obveze za depozite, jamčevine i kapare"
"2148","kp_rrif2148","l10n_hr_chart_template_rrif","liability_current",,"Obveze za kaucije","Obveze za kaucije"
"2149","kp_rrif2149","l10n_hr_chart_template_rrif","liability_current",,"Ostali kratkoročni zajmovi i sl.","Ostali kratkoročni zajmovi i sl."
"2150","kp_rrif2150","l10n_hr_chart_template_rrif","liability_current",,"Obveze za kratkoročne kredite u banci (analitika po bankama a unutar banke po ugovorima o kreditu)","Obveze za kratkoročne kredite u banci (analitika po bankama a unutar banke po ugovorima o kreditu)"
"2151","kp_rrif2151","l10n_hr_chart_template_rrif","liability_current",,"Obveze za kratkoročne kredite u osiguravajućim društvima (analitika po društvima i kreditima)","Obveze za kratkoročne kredite u osiguravajućim društvima (analitika po društvima i kreditima)"
"2152","kp_rrif2152","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema ostalim kreditnim institucijama (štedionicama, mirovinskom fondu i dr.)","Obveze prema ostalim kreditnim institucijama (štedionicama, mirovinskom fondu i dr.)"
"2153","kp_rrif2153","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema kreditnim institucijama i bankama u inozemstvu","Obveze prema kreditnim institucijama i bankama u inozemstvu"
"2154","kp_rrif2154","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema bankama po naplaćenim garancijama","Obveze prema bankama po naplaćenim garancijama"
"2155","kp_rrif2155","l10n_hr_chart_template_rrif","liability_current",,"Obveze po isplaćenom akreditivu","Obveze po isplaćenom akreditivu"
"2156","kp_rrif2156","l10n_hr_chart_template_rrif","liability_current",,"Obveze za prekoračenje na računu (okvirni kredit)","Obveze za prekoračenje na računu (okvirni kredit)"
"2157","kp_rrif2157","l10n_hr_chart_template_rrif","liability_current",,"Obveze za kamate prema bankama","Obveze za kamate prema bankama"
"2158","kp_rrif2158","l10n_hr_chart_template_rrif","liability_current",,"Obveze za kamate prema financ. institucijama","Obveze za kamate prema financ. institucijama"
"2159","kp_rrif2159","l10n_hr_chart_template_rrif","liability_current",,"Ostale obveze prema kreditnim institucijama","Ostale obveze prema kreditnim institucijama"
"2160","kp_rrif2160","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema izdavateljima kreditnih kartica (analitika prema kartičarima)-a1","Obveze prema izdavateljima kreditnih kartica (analitika prema kartičarima)-Analitika 1"
"2170","kp_rrif2170","l10n_hr_chart_template_rrif","liability_current",,"Obveze s temelja eskontiranih mjenica","Obveze s temelja eskontiranih mjenica"
"2171","kp_rrif2171","l10n_hr_chart_template_rrif","liability_current",,"Ostale obveze iz eskonta vrijednosnih papira","Ostale obveze iz eskonta vrijednosnih papira"
"2172","kp_rrif2172","l10n_hr_chart_template_rrif","liability_current",,"Obveze iz otkupa tražbina","Obveze iz otkupa tražbina"
"2180","kp_rrif2180","l10n_hr_chart_template_rrif","liability_current",,"Obveze s osnove dugotrajne imovine namijenjene prodaji (analitika prema izvorima)-a1","Obveze s osnove dugotrajne imovine namijenjene prodaji (analitika prema izvorima)-Analitika 1"
"2200","kp_rrif2200","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači dobara","Dobavljači dobara"
"2201","kp_rrif2201","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači usluga","Dobavljači usluga"
"2202","kp_rrif2202","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači opreme, postrojenja i nekretnina","Dobavljači opreme, postrojenja i nekretnina"
"2203","kp_rrif2203","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači nematerijalne imovine","Dobavljači nematerijalne imovine"
"2204","kp_rrif2204","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači iz operativnog lizinga (za najmove)","Dobavljači iz operativnog lizinga (za najmove)"
"2205","kp_rrif2205","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači iz ortačkog ugovora","Dobavljači iz ortačkog ugovora"
"2206","kp_rrif2206","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači zadruge, ustanove i dr.","Dobavljači zadruge, ustanove i dr."
"2210","kp_rrif2210","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači dobara iz inozemstva","Dobavljači dobara iz inozemstva"
"2211","kp_rrif2211","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači usluga iz inozemstva","Dobavljači usluga iz inozemstva"
"2212","kp_rrif2212","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači prava intelektualnog vlasništva, istraživanja tržišta i dr.","Dobavljači prava intelektualnog vlasništva, istraživanja tržišta i dr."
"2213","kp_rrif2213","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači prava na franšizu, uporabu imena i sl.","Dobavljači prava na franšizu, uporabu imena i sl."
"2220","kp_rrif2220","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači, obrtnici i slobodna zanimanja","Dobavljači, obrtnici i slobodna zanimanja"
"2221","kp_rrif2221","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači fizičke osobe (za isporučene osnovne proizvode poljodjelstva, ribarstva i šumarstva)","Dobavljači fizičke osobe (za isporučene osnovne proizvode poljodjelstva, ribarstva i šumarstva)"
"2222","kp_rrif2222","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači za isporučenu osobnu imovinu","Dobavljači za isporučenu osobnu imovinu"
"2223","kp_rrif2223","l10n_hr_chart_template_rrif","liability_payable","True","Obveze s osnove autorskih prava, inovacija, patenata i sl.","Obveze s osnove autorskih prava, inovacija, patenata i sl."
"2224","kp_rrif2224","l10n_hr_chart_template_rrif","liability_payable","True","Obveze s osnove ugovora o djelu i akviziterstva","Obveze s osnove ugovora o djelu i akviziterstva"
"2225","kp_rrif2225","l10n_hr_chart_template_rrif","liability_payable","True","Obveze prema studentima i učenicima za rad preko studentskog servisa","Obveze prema studentima i učenicima za rad preko studentskog servisa"
"2226","kp_rrif2226","l10n_hr_chart_template_rrif","liability_payable","True","Dobavljači kooperanti - fizičke osobe","Dobavljači kooperanti - fizičke osobe"
"2229","kp_rrif2229","l10n_hr_chart_template_rrif","liability_payable","True","Ostali dobavljači - fizičke osobe","Ostali dobavljači - fizičke osobe"
"2230","kp_rrif2230","l10n_hr_chart_template_rrif","liability_payable","True","Obveze za energetske isporuke","Obveze za energetske isporuke"
"2231","kp_rrif2231","l10n_hr_chart_template_rrif","liability_payable","True","Obveze za usluge odvodnje i odvoza smeća","Obveze za usluge odvodnje i odvoza smeća"
"2232","kp_rrif2232","l10n_hr_chart_template_rrif","liability_payable","True","Obveze za ostale komunalne usluge","Obveze za ostale komunalne usluge"
"2240","kp_rrif2240","l10n_hr_chart_template_rrif","liability_payable","True","Obveze za nefakturirane a preuzete isporuke robe i usluge-a1","Obveze za nefakturirane a preuzete isporuke robe i usluge-Analitika 1"
"2250","kp_rrif2250","l10n_hr_chart_template_rrif","liability_payable","True","Primljeni predujmovi za koje su izdani računi s PDV-om","Primljeni predujmovi za koje su izdani računi s PDV-om"
"2251","kp_rrif2251","l10n_hr_chart_template_rrif","liability_payable","True","Primljeni predujmovi koji nisu pod PDV-om","Primljeni predujmovi koji nisu pod PDV-om"
"2252","kp_rrif2252","l10n_hr_chart_template_rrif","liability_payable","True","Primljeni predujmovi iz inozemstva","Primljeni predujmovi iz inozemstva"
"2253","kp_rrif2253","l10n_hr_chart_template_rrif","liability_payable","True","Obveze za predujmove građana","Obveze za predujmove građana"
"2300","kp_rrif2300","l10n_hr_chart_template_rrif","liability_current",,"Obveze za neto-plaće","Obveze za neto-plaće"
"2301","kp_rrif2301","l10n_hr_chart_template_rrif","liability_current",,"Nadoknade plaća koje se refundiraju (od državnih institucija, od HZZO, od lokalne samoupr.)","Nadoknade plaća koje se refundiraju (od državnih institucija, od HZZO, od lokalne samoupr.)"
"2302","kp_rrif2302","l10n_hr_chart_template_rrif","liability_current",,"Obveze za nadoknade troškova (dnevnice, terenski dod.,troškovi sl. puta, km,dolazak na posao i dr.)","Obveze za nadoknade troškova (dnevnice, terenski dodatak, nadoknade troškova službenog puta, uključivo i kilometraža, nadoknade za dolazak na posao i dr.)"
"2303","kp_rrif2303","l10n_hr_chart_template_rrif","liability_current",,"Obveze za darove i potpore (božićnica, dar djeci, potpora zbog bolesti, regres za g. o. i sl.)","Obveze za darove i potpore (božićnica, dar djeci, potpora zbog bolesti, regres za g. o. i sl.)"
"2304","kp_rrif2304","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema zaposlenima za primitke koji se smatraju dohotkom (prehrana, davanja u naravi i sl.)","Obveze prema zaposlenima za primitke koji se smatraju dohotkom (prehrana, davanja u naravi i sl.)"
"2305","kp_rrif2305","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema zaposlenima zbog otpremnine, jubilarne, odvojeni život, pomoć obitelji umrlog posloprimca","Obveze prema zaposlenima s temelja otpremnine, jubilarne nagrade, nadoknade za odvojeni život, pomoći obitelji umrlog posloprimca i sl."
"23060","kp_rrif23060","l10n_hr_chart_template_rrif","liability_current",,"Obveze za obustave iz neto-plaće za isplate kredita i pozajmica","Obveze za obustave iz neto-plaće za isplate kredita i pozajmica"
"23061","kp_rrif23061","l10n_hr_chart_template_rrif","liability_current",,"Obveze za obustave iz neto-plaća i nadoknada za sudske zabrane, kazne i sl.","Obveze za obustave iz neto-plaća i nadoknada za sudske zabrane, kazne i sl."
"23062","kp_rrif23062","l10n_hr_chart_template_rrif","liability_current",,"Obveze iz ovrha na neto-plaći","Obveze iz ovrha na neto-plaći"
"23063","kp_rrif23063","l10n_hr_chart_template_rrif","liability_current",,"Obveze za obustave iz neto-plaća i nadoknada plaća za članarine, i dr.","Obveze za obustave iz neto-plaća i nadoknada plaća za članarine, i dr."
"2307","kp_rrif2307","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema zaposlenima za zatezne kamate i sl.","Obveze prema zaposlenima za zatezne kamate i sl."
"2308","kp_rrif2308","l10n_hr_chart_template_rrif","liability_current",,"Obveza prema zaposlenima za naknadu šteta: zbog ozljede na radu, neiskorištenog godišnjeg odmora i sl.","Obveza prema zaposlenima za naknadu šteta: zbog ozljede na radu, neiskorištenog godišnjeg odmora i sl."
"2309","kp_rrif2309","l10n_hr_chart_template_rrif","liability_current",,"Obveze za obračunanu bruto-plaću iz prošle poslovne godine (koje nisu isplaćene i XIII. plaća)","Obveze za obračunanu bruto-plaću iz prošle poslovne godine (koje nisu isplaćene i XIII. plaća)"
"2310","kp_rrif2310","l10n_hr_chart_template_rrif","liability_current",,"Obveze s temelja tekuće nabave u gotovini","Obveze s temelja tekuće nabave u gotovini"
"2311","kp_rrif2311","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema vanjskim članovima uprave, nadzornog odbora, prokuristima, stečajnim upraviteljima i sl.","Obveze prema vanjskim članovima uprave, nadzornog odbora, prokuristima, stečajnim upraviteljima i sl."
"2312","kp_rrif2312","l10n_hr_chart_template_rrif","liability_current",,"Obveze za preuzeta plaćanja temeljem ugovora o cesiji, asignaciji i preuzimanjem duga","Obveze za preuzeta plaćanja temeljem ugovora o cesiji, asignaciji i preuzimanjem duga"
"2313","kp_rrif2313","l10n_hr_chart_template_rrif","liability_current",,"Obveze za ugovorene penale, kazne i sl.","Obveze za ugovorene penale, kazne i sl."
"2314","kp_rrif2314","l10n_hr_chart_template_rrif","liability_current",,"Obveze za darovanja (do 2% od ukupnog prihoda)","Obveze za darovanja (do 2% od ukupnog prihoda)"
"2315","kp_rrif2315","l10n_hr_chart_template_rrif","liability_current",,"Obveze za nadoknadu troškova (refundacije)","Obveze za nadoknadu troškova (refundacije)"
"2316","kp_rrif2316","l10n_hr_chart_template_rrif","liability_current",,"Obveze za naknadno odobrene bonifikacije, casasconte i druge popuste","Obveze za naknadno odobrene bonifikacije, casasconte i druge popuste"
"2317","kp_rrif2317","l10n_hr_chart_template_rrif","liability_current",,"Obveze za stipendije","Obveze za stipendije"
"2318","kp_rrif2318","l10n_hr_chart_template_rrif","liability_current",,"Obveze za primljene potpore (nisu za prihod)","Obveze za primljene potpore (nisu za prihod)"
"2319","kp_rrif2319","l10n_hr_chart_template_rrif","liability_current",,"Ostale kratkoročne obveze (npr. prema vanjskim suradnicima)","Ostale kratkoročne obveze(npr. prema vanjskim suradnicima)"
"2320","kp_rrif2320","l10n_hr_chart_template_rrif","liability_current",,"Obveze za ugovorenu kamatu","Obveze za ugovorenu kamatu"
"2321","kp_rrif2321","l10n_hr_chart_template_rrif","liability_current",,"Obveze za zateznu kamatu","Obveze za zateznu kamatu"
"2322","kp_rrif2322","l10n_hr_chart_template_rrif","liability_current",,"Obveze za kamate na zajmove prema poduzetnicima","Obveze za kamate na zajmove prema poduzetnicima"
"2323","kp_rrif2323","l10n_hr_chart_template_rrif","liability_current",,"Obveze za kamate po sudskim sporovima","Obveze za kamate po sudskim sporovima"
"2330","kp_rrif2330","l10n_hr_chart_template_rrif","liability_current",,"Obveze po obračunu za prodana dobra","Obveze po obračunu za prodana dobra"
"2331","kp_rrif2331","l10n_hr_chart_template_rrif","liability_current",,"Obveze za isplatu komitentu","Obveze za isplatu komitentu"
"2332","kp_rrif2332","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema nalogodavatelju iz zastupničke prodaje","Obveze prema nalogodavatelju iz zastupničke prodaje"
"2340","kp_rrif2340","l10n_hr_chart_template_rrif","liability_current",,"Obveze iz osiguranja imovine i osoba","Obveze iz osiguranja imovine i osoba"
"2341","kp_rrif2341","l10n_hr_chart_template_rrif","liability_current",,"Obveze za životna osiguranja","Obveze za životna osiguranja"
"2342","kp_rrif2342","l10n_hr_chart_template_rrif","liability_current",,"Obveze za III. stupu mirovinskog osiguranja","Obveze za III. stupu mirovinskog osiguranja"
"2343","kp_rrif2343","l10n_hr_chart_template_rrif","liability_current",,"Obveze za premije zdravstvenog osiguranja","Obveze za premije zdravstvenog osiguranja"
"2344","kp_rrif2344","l10n_hr_chart_template_rrif","liability_current",,"Obveze za dopunsko zdravstveno osiguranje","Obveze za dopunsko zdravstveno osiguranje"
"2350","kp_rrif2350","l10n_hr_chart_template_rrif","liability_current",,"Obveze iz poslovanja u slobodnoj zoni-a1","Obveze iz poslovanja u slobodnoj zoni-Analitika 1"
"2360","kp_rrif2360","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema uvozniku (ili naručitelju)","Obveze prema uvozniku (ili naručitelju)"
"2361","kp_rrif2361","l10n_hr_chart_template_rrif","liability_current",,"Obveze po poslovima izvoza za tuđi račun","Obveze po poslovima izvoza za tuđi račun"
"2370","kp_rrif2370","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema P.J. u inozemstvu","Obveze prema P.J. u inozemstvu"
"2371","kp_rrif2371","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema podružnicama u inozemstvu","Obveze prema podružnicama u inozemstvu"
"2380","kp_rrif2380","l10n_hr_chart_template_rrif","liability_current",,"Obveze za uplaćeni a neupisani temeljni kapital","Obveze za uplaćeni a neupisani temeljni kapital"
"2381","kp_rrif2381","l10n_hr_chart_template_rrif","liability_current",,"Obveze za kupnju poslovnog udjela","Obveze za kupnju poslovnog udjela"
"2390","kp_rrif2390","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema odštetnim zahtjevima","Obveze prema odštetnim zahtjevima"
"2391","kp_rrif2391","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema brokerima za kupljene vrijednosne papire","Obveze prema brokerima za kupljene vrijednosne papire"
"2392","kp_rrif2392","l10n_hr_chart_template_rrif","liability_current",,"Obveze za doprinos za komunalnu infrastrukturu","Obveze za doprinos za komunalnu infrastrukturu"
"2393","kp_rrif2393","l10n_hr_chart_template_rrif","liability_current",,"Obveze za PDV iz poslovnih odnosa","Obveze za PDV iz poslovnih odnosa"
"2394","kp_rrif2394","l10n_hr_chart_template_rrif","liability_current",,"Obveze iz primjene valutne klauzule","Obveze iz primjene valutne klauzule"
"2395","kp_rrif2395","l10n_hr_chart_template_rrif","liability_current",,"Obveze s osnove sudskih presuda","Obveze s osnove sudskih presuda"
"2396","kp_rrif2396","l10n_hr_chart_template_rrif","liability_current",,"Obveze iz ortaštva","Obveze iz ortaštva"
"2397","kp_rrif2397","l10n_hr_chart_template_rrif","liability_current",,"Obveze za tuđa sredstva","Obveze za tuđa sredstva"
"2398","kp_rrif2398","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema inozemnom poduzet. za PDV","Obveze prema inozemnom poduzet. za PDV"
"2399","kp_rrif2399","l10n_hr_chart_template_rrif","liability_current",,"Ostale nespomenute obveze","Ostale nespomenute obveze"
"24000","kp_rrif24000","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV - 10%","Obveza za PDV - 10%"
"24001","kp_rrif24001","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV - 22%","Obveza za PDV - 22%"
"24002","kp_rrif24002","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV - 23%","Obveza za PDV - 23%"
"24003","kp_rrif24003","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV - 25%","Obveza za PDV - 25%"
"24010","kp_rrif24010","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV za predujam - 10%","Obveza za PDV za predujam - 10%"
"24011","kp_rrif24011","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV za predujam - 22%","Obveza za PDV za predujam - 22%"
"24012","kp_rrif24012","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV za predujam - 23%","Obveza za PDV za predujam - 23%"
"24013","kp_rrif24013","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV za predujam - 25%","Obveza za PDV za predujam - 25%"
"2402","kp_rrif2402","l10n_hr_chart_template_rrif","liability_current",,"Obveze za PDV s osnove vlastite potrošnje - 23% (za proizv. za reprez. te za o.auto. nab. do 2010.)","Obveze za PDV s osnove vlastite potrošnje - 23% (za uporabu proizv. za reprezentaciju te za osobne automobile nabavljene do 1.I. 2010.)"
"24030","kp_rrif24030","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV po nezaračunanim isporukama - 10%","Obveza za PDV po nezaračunanim isporukama - 10%"
"24031","kp_rrif24031","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV po nezaračunanim isporukama - 22%","Obveza za PDV po nezaračunanim isporukama - 22%"
"24032","kp_rrif24032","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV po nezaračunanim isporukama - 23%","Obveza za PDV po nezaračunanim isporukama - 23%"
"24033","kp_rrif24033","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV po nezaračunanim isporukama - 25%","Obveza za PDV po nezaračunanim isporukama - 25%"
"24041","kp_rrif24041","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV zbog promjene postotka priznavanja PDV-a","Obveza za PDV zbog promjene postotka priznavanja PDV-a"
"2405","kp_rrif2405","l10n_hr_chart_template_rrif","liability_current",,"Obračunana a nedospjela obveza za PDV","Obračunana a nedospjela obveza za PDV"
"24060","kp_rrif24060","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV po Rješenju PU","Obveza za PDV po Rješenju PU"
"24061","kp_rrif24061","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV na inozemne usluge","Obveza za PDV inozemne usluge"
"2407","kp_rrif2407","l10n_hr_chart_template_rrif","liability_current",,"Obveza za razliku poreza i pretporeza u obračunskom razdoblju","Obveza za razliku poreza i pretporeza u obračunskom razdoblju"
"2408","kp_rrif2408","l10n_hr_chart_template_rrif","liability_current",,"Povrat PDV-a iz putničkog prometa","Povrat PDV-a iz putničkog prometa"
"2409","kp_rrif2409","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PDV po konačnom obračunu","Obveza za PDV po konačnom obračunu"
"2410","kp_rrif2410","l10n_hr_chart_template_rrif","liability_current",,"Obveze za porez na dohodak iz plaća i primitaka izjednačenih s plaćom","Obveze za porez na dohodak iz plaća i primitaka izjednačenih s plaćom"
"2411","kp_rrif2411","l10n_hr_chart_template_rrif","liability_current",,"Obveze za prirez iz plaća i primitaka izjednačenih s plaćama","Obveze za prirez iz plaća i primitaka izjednačenih s plaćama"
"2412","kp_rrif2412","l10n_hr_chart_template_rrif","liability_current",,"Obveze za porez i prirez na drugi dohodak (autorski honorari, ug. o djelu, doh. članova nadz.i dr.)","Obveze za porez i prirez na drugi dohodak (iz autorskih honorara, ugovora o djelu, dohodaka članova nadzornog odbora i dr.)"
"2413","kp_rrif2413","l10n_hr_chart_template_rrif","liability_current",,"Obveze za porez i prirez iz stipendija i nagrada učenika na praksi","Obveze za porez i prirez iz stipendija i nagrada učenika na praksi"
"2414","kp_rrif2414","l10n_hr_chart_template_rrif","liability_current",,"Obveze za porez i prirez za ostale dohotke","Obveze za porez i prirez za ostale dohotke"
"2420","kp_rrif2420","l10n_hr_chart_template_rrif","liability_current",,"Doprinos za MO iz plaća (I. stup)","Doprinos za MO iz plaća (I. stup)"
"2421","kp_rrif2421","l10n_hr_chart_template_rrif","liability_current",,"Doprinos za MO iz plaća (II. stup - analitika prema fondovima)","Doprinos za MO iz plaća (II. stup - analitika prema fondovima)"
"2422","kp_rrif2422","l10n_hr_chart_template_rrif","liability_current",,"Doprinos za MO za beneficirani staž (I. i II. stup)","Doprinos za MO za beneficirani staž (I. i II. stup)"
"2423","kp_rrif2423","l10n_hr_chart_template_rrif","liability_current",,"Doprinos za zdravstveno osiguranje na plaće","Doprinos za zdravstveno osiguranje na plaće"
"2424","kp_rrif2424","l10n_hr_chart_template_rrif","liability_current",,"Poseban doprinos za zdravstveno osiguranje na plaće za ozljede na radu i prof. bolesti","Poseban doprinos za zdravstveno osiguranje na plaće za ozljede na radu i prof. bolesti"
"2426","kp_rrif2426","l10n_hr_chart_template_rrif","liability_current",,"Doprinos za zapošljavanje na plaću","Doprinos za zapošljavanje na plaću"
"24270","kp_rrif24270","l10n_hr_chart_template_rrif","liability_current",,"Doprinos za MO I. stup","Doprinos za MO I. stup"
"24271","kp_rrif24271","l10n_hr_chart_template_rrif","liability_current",,"Doprinos za MO II. stup","Doprinos za MO II. stup"
"24272","kp_rrif24272","l10n_hr_chart_template_rrif","liability_current",,"Doprinos za zdravstveno osiguranje na honorar","Doprinos za zdravstveno osiguranje na honorar"
"2428","kp_rrif2428","l10n_hr_chart_template_rrif","liability_current",,"Doprinos HZZO-u na službena putovanja u inozemstvo","Doprinos HZZO-u na službena putovanja u inozemstvo"
"2429","kp_rrif2429","l10n_hr_chart_template_rrif","liability_current",,"Obveze za ostale nespomenute doprinose koji se plaćaju na dohotke","Obveze za ostale nespomenute doprinose koji se plaćaju na dohotke"
"2430","kp_rrif2430","l10n_hr_chart_template_rrif","liability_current",,"Obveze za porez na dobitak","Obveze za porez na dobitak"
"24310","kp_rrif24310","l10n_hr_chart_template_rrif","liability_current",,"Porez na dohodak od kapitala na izuzimanja ili privatni život članova društva i dioničara (40%+prirez)","Porez na dohodak od kapitala na izuzimanja ili privatni život članova društva i dioničara (40% + prirez)"
"24311","kp_rrif24311","l10n_hr_chart_template_rrif","liability_current",,"Porez na dohodak od kapitala na isplate dobitka i dividendi (iz 2001. do 2004. = 12% + prirez)","Porez na dohodak od kapitala na isplate dobitka i dividendi (iz 2001. do 2004. = 12% + prirez)"
"24312","kp_rrif24312","l10n_hr_chart_template_rrif","liability_current",,"Porez na dohodak od kapitala na kamate (na zajmove fizičkih osoba = 40% + prirez)","Porez na dohodak od kapitala na kamate (na zajmove fizičkih osoba = 40% + prirez)"
"24313","kp_rrif24313","l10n_hr_chart_template_rrif","liability_current",,"Porez na dohodak od kapitala s osnove opcijskih dionica (25% + prirez)","Porez na dohodak od kapitala s osnove opcijskih dionica (25% + prirez)"
"2432","kp_rrif2432","l10n_hr_chart_template_rrif","liability_current",,"Obveze za porez po odbitku (-15% -čl.31.Zakona o porezu na dobit -na ino usluge intelek. vlas. i dr.)","Obveze za porez po odbitku (na inozemne usluge intelektualnog vlasništva, za istraživ. tržišta, poslovno i porezno savjetovanje, revizorske usluge i na kamate inozemnim nebankarskim pravnim osobama - 15% - čl. 31. Zakona o porezu na dobit)"
"2436","kp_rrif2436","l10n_hr_chart_template_rrif","liability_current",,"Obveza za PD po rješenju poreznog nadzora","Obveza za PD po rješenju poreznog nadzora"
"2440","kp_rrif2440","l10n_hr_chart_template_rrif","liability_current",,"Obveza za posebni porez pri uvozu automobila, plovila i zrakoplova","Obveza za posebni porez pri uvozu automobila, plovila i zrakoplova"
"2441","kp_rrif2441","l10n_hr_chart_template_rrif","liability_current",,"Obveza za posebni porez na bezalkoholna pića","Obveza za posebni porez na bezalkoholna pića"
"2442","kp_rrif2442","l10n_hr_chart_template_rrif","liability_current",,"Obveza za posebni porez na pivo","Obveza za posebni porez na pivo"
"2443","kp_rrif2443","l10n_hr_chart_template_rrif","liability_current",,"Obveza za posebni porez na alkohol","Obveza za posebni porez na alkohol"
"24440","kp_rrif24440","l10n_hr_chart_template_rrif","liability_current",,"Obveze za naknade za Hrvatske autoceste","Obveze za naknade za Hrvatske autoceste"
"24441","kp_rrif24441","l10n_hr_chart_template_rrif","liability_current",,"Obveze za naknade za Hrvatske ceste","Obveze za naknade za Hrvatske ceste"
"2445","kp_rrif2445","l10n_hr_chart_template_rrif","liability_current",,"Obveze za posebni porez na duhanske proizvode","Obveze za posebni porez na duhanske proizvode"
"2446","kp_rrif2446","l10n_hr_chart_template_rrif","liability_current",,"Obveza za posebni porez na kavu","Obveza za posebni porez na kavu"
"2447","kp_rrif2447","l10n_hr_chart_template_rrif","liability_current",,"Obveze za posebni porez na luksuzne proizvode","Obveze za posebni porez na luksuzne proizvode"
"2448","kp_rrif2448","l10n_hr_chart_template_rrif","liability_current",,"Obveza za 5% poreza na promet nekretnina","Obveza za 5% poreza na promet nekretnina"
"2449","kp_rrif2449","l10n_hr_chart_template_rrif","liability_current",,"Obveze za 5% poreza na promet mot. vozila i plovila","Obveze za 5% poreza na promet mot. vozila i plovila"
"2450","kp_rrif2450","l10n_hr_chart_template_rrif","liability_current",,"Obveze za članarinu turist. zajednicama-a1","Obveze za članarinu turist. zajednicama-Analitika 1"
"2460","kp_rrif2460","l10n_hr_chart_template_rrif","liability_current",,"Obveza za HGK za paušalnu naknadu","Obveza za HGK za paušalnu naknadu"
"2461","kp_rrif2461","l10n_hr_chart_template_rrif","liability_current",,"Obveza za HGK za javnu funkciju","Obveza za HGK za javnu funkciju"
"2462","kp_rrif2462","l10n_hr_chart_template_rrif","liability_current",,"Obveza za članarinu Obrtničkoj komori","Obveza za članarinu Obrtničkoj komori"
"2463","kp_rrif2463","l10n_hr_chart_template_rrif","liability_current",,"Obveze za paušal HOK-u","Obveze za paušal HOK-u"
"2464","kp_rrif2464","l10n_hr_chart_template_rrif","liability_current",,"Obveze za članarinu granskoj ili strukovnoj komori","Obveze za članarinu granskoj ili strukovnoj komori"
"2470","kp_rrif2470","l10n_hr_chart_template_rrif","liability_current",,"Obveze za carinu","Obveze za carinu"
"2471","kp_rrif2471","l10n_hr_chart_template_rrif","liability_current",,"Obveze za carinu prema mjernoj jedinici (prelevmani)","Obveze za carinu prema mjernoj jedinici (prelevmani)"
"2472","kp_rrif2472","l10n_hr_chart_template_rrif","liability_current",,"Obveze za carinske pristojbe i takse","Obveze za carinske pristojbe i takse"
"2473","kp_rrif2473","l10n_hr_chart_template_rrif","liability_current",,"Ostale obveze prema carini (za PDV i dr.)","Ostale obveze prema carini (za PDV i dr.)"
"2480","kp_rrif2480","l10n_hr_chart_template_rrif","liability_current",,"Obveze za imovinski porez na motorna vozila i plovne objekte","Obveze za imovinski porez na motorna vozila i plovne objekte"
"2481","kp_rrif2481","l10n_hr_chart_template_rrif","liability_current",,"Obveze za imovinski porez na kuće za odmor i porez na korištenje javnih površina","Obveze za imovinski porez na kuće za odmor i porez na korištenje javnih površina"
"2482","kp_rrif2482","l10n_hr_chart_template_rrif","liability_current",,"Obveze za porez na istaknutu reklamu","Obveze za porez na istaknutu reklamu"
"2483","kp_rrif2483","l10n_hr_chart_template_rrif","liability_current",,"Obveze za porez na tvrtku ili naziv","Obveze za porez na tvrtku ili naziv"
"2484","kp_rrif2484","l10n_hr_chart_template_rrif","liability_current",,"Obveza za porez na potrošnju alkoholnih i bezalkoholnih pića i piva u ugostiteljstvu","Obveza za porez na potrošnju alkoholnih i bezalkoholnih pića i piva u ugostiteljstvu"
"2485","kp_rrif2485","l10n_hr_chart_template_rrif","liability_current",,"Obveze za porez na zabavne i športske priredbe","Obveze za porez na zabavne i športske priredbe"
"2486","kp_rrif2486","l10n_hr_chart_template_rrif","liability_current",,"Obveze za porez na nasljedstva i darove","Obveze za porez na nasljedstva i darove"
"2489","kp_rrif2489","l10n_hr_chart_template_rrif","liability_current",,"Obveze za ostale poreze županiji, gradu ili općini","Obveze za ostale poreze županiji, gradu ili općini"
"2490","kp_rrif2490","l10n_hr_chart_template_rrif","liability_current",,"Obveze za nadoknadu za šume","Obveze za nadoknadu za šume"
"2491","kp_rrif2491","l10n_hr_chart_template_rrif","liability_current",,"Obveze prema lokalnoj samoupravi za financiranje komunalne izgradnje (Zakon o komunalnom gospodarstvu)","Obveze prema lokalnoj samoupravi za financiranje komunalne izgradnje (prema Zakonu o komunalnom gospodarstvu)"
"2492","kp_rrif2492","l10n_hr_chart_template_rrif","liability_current",,"Obveze za zakonske kazne","Obveze za zakonske kazne"
"2493","kp_rrif2493","l10n_hr_chart_template_rrif","liability_current",,"Obveze za boravišnu pristojbu","Obveze za boravišnu pristojbu"
"2494","kp_rrif2494","l10n_hr_chart_template_rrif","liability_current",,"Obveze za koncesije","Obveze za koncesije"
"2495","kp_rrif2495","l10n_hr_chart_template_rrif","liability_current",,"Obveze za spomeničku rentu","Obveze za spomeničku rentu"
"2496","kp_rrif2496","l10n_hr_chart_template_rrif","liability_current",,"Obveze za naknade za iskorištav. mineral. sirovina","Obveze za naknade za iskorištav. mineral. sirovina"
"2497","kp_rrif2497","l10n_hr_chart_template_rrif","liability_current",,"Obveze za naknade za ambalažu","Obveze za naknade za ambalažu"
"2499","kp_rrif2499","l10n_hr_chart_template_rrif","liability_current",,"Ostale obveze za ostala javna davanja","Ostale obveze za ostala javna davanja"
"2500","kp_rrif2500","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za primljena dobra i usluge","Obveze za primljena dobra i usluge"
"2501","kp_rrif2501","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za korištenje zajmova","Obveze za korištenje zajmova"
"2510","kp_rrif2510","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za dugoročne financijske zajmove","Obveze za dugoročne financijske zajmove"
"2511","kp_rrif2511","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za dugoročne hipotekarne zajmove","Obveze za dugoročne hipotekarne zajmove"
"2512","kp_rrif2512","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za dugoročne zajmove prema građanima","Obveze za dugoročne zajmove prema građanima"
"2513","kp_rrif2513","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za dugoročne zajmove članovima društva","Obveze za dugoročne zajmove članovima društva"
"2514","kp_rrif2514","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za dugoročne zajmove iz inozemstva","Obveze za dugoročne zajmove iz inozemstva"
"2515","kp_rrif2515","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za depozite","Obveze za depozite"
"2516","kp_rrif2516","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za kapare","Obveze za kapare"
"2517","kp_rrif2517","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze s osnove jamstva (realiziranih)","Obveze s osnove jamstva (realiziranih)"
"2519","kp_rrif2519","l10n_hr_chart_template_rrif","liability_non_current",,"Ostale obveze za dugoročne zajmove","Ostale obveze za dugoročne zajmove"
"2520","kp_rrif2520","l10n_hr_chart_template_rrif","liability_non_current",,"Dugoročni financijski krediti banaka (analitika po bankama, pa po sklopljenim ugovorima o kreditu)","Dugoročni financijski krediti banaka (analitika po bankama a unutar toga po sklopljenim ugovorima o kreditu)"
"2521","kp_rrif2521","l10n_hr_chart_template_rrif","liability_non_current",,"Dugoročni krediti od osiguravajućih društava (analitika po društvima, pa po sklopljenim ugovorima)","Dugoročni krediti od osiguravajućih društava (analitika po društvima a unutar toga po sklopljenim ugovorima)"
"2522","kp_rrif2522","l10n_hr_chart_template_rrif","liability_non_current",,"Dugoročni krediti od ostalih domaćih kreditnih institucija (fonda, i dr.)","Dugoročni krediti od ostalih domaćih kreditnih institucija (fonda, i dr.)"
"2523","kp_rrif2523","l10n_hr_chart_template_rrif","liability_non_current",,"Dugoročni krediti od inozemnih banaka i drugih inozemnih kreditnih institucija","Dugoročni krediti od inozemnih banaka i drugih inozemnih kreditnih institucija"
"2529","kp_rrif2529","l10n_hr_chart_template_rrif","liability_non_current",,"Dugoročne obveze za kamate","Dugoročne obveze za kamate"
"2530","kp_rrif2530","l10n_hr_chart_template_rrif","liability_non_current",,"Financijski najam vozila, brodova i dr.","Financijski najam vozila, brodova i dr."
"2531","kp_rrif2531","l10n_hr_chart_template_rrif","liability_non_current",,"Povratni financijski najam (npr. nekretnina brodova, tramvaja, vlakova i dr.)","Povratni financijski najam (npr. nekretnina brodova, tramvaja, vlakova i dr.)"
"2540","kp_rrif2540","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za dugoročne predujmove (avanse) za izgradnju objekata i postrojenja (bez PDV-a)","Obveze za dugoročne predujmove (avanse) za izgradnju objekata i postrojenja (bez PDV-a)"
"2541","kp_rrif2541","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za dugoročne predujmove za isporuke zaliha","Obveze za dugoročne predujmove za isporuke zaliha"
"2542","kp_rrif2542","l10n_hr_chart_template_rrif","liability_non_current",,"Ostale dugoročne obveze za predujmove","Ostale dugoročne obveze za predujmove"
"2550","kp_rrif2550","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze prema dobavljačima s rokom plaćanja duljim od godine dana (npr. kod izgradnje)","Obveze prema dobavljačima s rokom plaćanja duljim od godine dana (npr. kod izgradnje)"
"2551","kp_rrif2551","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze prema dobavljačima iz inozemstva (dugoročne)","Obveze prema dobavljačima iz inozemstva (dugoročne)"
"2552","kp_rrif2552","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze prema vjerovnicima iz ostalih poslovnih aktivnosti","Obveze prema vjerovnicima iz ostalih poslovnih aktivnosti"
"2553","kp_rrif2553","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze prema dobavljačima za zadržani dio iz jamstva","Obveze prema dobavljačima za zadržani dio iz jamstva"
"2560","kp_rrif2560","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za izdane dugoročne obveznice","Obveze za izdane dugoročne obveznice"
"2561","kp_rrif2561","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za dugoročne komercijalne vrijednosne papire","Obveze za dugoročne komercijalne vrijednosne papire"
"2562","kp_rrif2562","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za dugoročne mjenice","Obveze za dugoročne mjenice"
"2563","kp_rrif2563","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze po ostalim dugoroč. vrijednos. papirima","Obveze po ostalim dugoroč. vrijednos. papirima"
"2568","kp_rrif2568","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za kamate iz obveznica","Obveze za kamate iz obveznica"
"2569","kp_rrif2569","l10n_hr_chart_template_rrif","liability_non_current",,"Diskont na vrijednosne papire (kao razlika do nominalne vrijednosti)","Diskont na vrijednosne papire (kao razlika do nominalne vrijednosti)"
"2570","kp_rrif2570","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze prema poduzetnicima u kojima postoje sudjelujući interesi (do 20% udjela)-a1","Obveze prema poduzetnicima u kojima postoje sudjelujući interesi (do 20% udjela)-Analitika 1"
"2580","kp_rrif2580","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze prema Državi (realizirana jamstva, krediti)-a1","Obveze prema Državi (realizirana jamstva, krediti)-Analitika 1"
"2590","kp_rrif2590","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze prema zakladama, komorama i sl.","Obveze prema zakladama, komorama i sl."
"2591","kp_rrif2591","l10n_hr_chart_template_rrif","liability_non_current",,"Dugoročne obveze za porez","Dugoročne obveze za porez"
"2592","kp_rrif2592","l10n_hr_chart_template_rrif","liability_non_current",,"Dugoročne obveze za socijalno osiguranje","Dugoročne obveze za socijalno osiguranje"
"2593","kp_rrif2593","l10n_hr_chart_template_rrif","liability_non_current",,"Obveze za dugoročne jamčevine","Obveze za dugoročne jamčevine"
"2599","kp_rrif2599","l10n_hr_chart_template_rrif","liability_non_current",,"Ostale dugoročne obveze","Ostale dugoročne obveze"
"2600","kp_rrif2600","l10n_hr_chart_template_rrif","liability_non_current",,"Odgođena privremena razlika porezne obveze (analitika po godinama - HSFI t. 12.22. i MRS 12 t. 5.)","Odgođena privremena razlika porezne obveze (analitika po godinama - HSFI t. 12.22. i MRS 12 t. 5.)"
"2800","kp_rrif2800","l10n_hr_chart_template_rrif","liability_current",,"Rezerviranja za otpremnine","Rezerviranja za otpremnine"
"2801","kp_rrif2801","l10n_hr_chart_template_rrif","liability_current",,"Rezerviranja za mirovine","Rezerviranja za mirovine"
"2810","kp_rrif2810","l10n_hr_chart_template_rrif","liability_current",,"Dugor. rezerv. za odg. plać. poreza i dopr.","Dugor. rezerv. za odg. plać. poreza i dopr."
"2820","kp_rrif2820","l10n_hr_chart_template_rrif","liability_current",,"Dugoročna rezerviranja za troškove izdanih jamstava za prodana dobra","Dugoročna rezerviranja za troškove izdanih jamstava za prodana dobra"
"2821","kp_rrif2821","l10n_hr_chart_template_rrif","liability_current",,"Dugoročna rezerviranja za gubitke po započetim sudskim sporovima","Dugoročna rezerviranja za gubitke po započetim sudskim sporovima"
"2822","kp_rrif2822","l10n_hr_chart_template_rrif","liability_current",,"Dugor. rezerv. za obnovu prirodnog bogatstva-dug. rezer. za neiskor. g.o.(MRS19.t14.,HSFI13) -vidi 298","Dugor. rezerv. za obnovu prirodnog bogatstva - Dug. rezer. za neiskorištene g. o. (MRS 19. t. 14. i HSFI 13) - vidi račun 298"
"2824","kp_rrif2824","l10n_hr_chart_template_rrif","liability_current",,"Dugoročno rezerviranje za restrukturiranje (MRS 37 i HSFI t. 16.22.)","Dugoročno rezerviranje za restrukturiranje (MRS 37 i HSFI t. 16.22.)"
"2825","kp_rrif2825","l10n_hr_chart_template_rrif","liability_current",,"Rezerviranje za ugovore s poteškoćama (MRS 37, t. 66. i HSFI t. 16.21.)","Rezerviranje za ugovore s poteškoćama (MRS 37, t. 66. i HSFI t. 16.21.)"
"2829","kp_rrif2829","l10n_hr_chart_template_rrif","liability_current",,"Ostala dugoročna rezerviranja za rizike i dr.","Ostala dugoročna rezerviranja za rizike i dr."
"2900","kp_rrif2900","l10n_hr_chart_template_rrif","liability_current",,"Obračunani troškovi za koje nije primljena faktura (telefon, grijanje, el. energija, plin, voda i sl.)","Obračunani troškovi za koje nije primljena faktura (telefon, grijanje, el. energija, plin, voda i sl.)"
"2901","kp_rrif2901","l10n_hr_chart_template_rrif","liability_current",,"Obračunana najamnina iz operativnog (poslovnog) najma","Obračunana najamnina iz operativnog (poslovnog) najma"
"2902","kp_rrif2902","l10n_hr_chart_template_rrif","liability_current",,"Obračunani troškovi tekućeg održavanja","Obračunani troškovi tekućeg održavanja"
"2903","kp_rrif2903","l10n_hr_chart_template_rrif","liability_current",,"Obračunani troškovi reklame, propagande i sajmova","Obračunani troškovi reklame, propagande i sajmova"
"2904","kp_rrif2904","l10n_hr_chart_template_rrif","liability_current",,"Obračunane a neplaćene usluge korištene u obračunskom razdoblju","Obračunane a neplaćene usluge korištene u obračunskom razdoblju"
"2905","kp_rrif2905","l10n_hr_chart_template_rrif","liability_current",,"Obračunani rad po ugovoru o djelu, autor. hon.","Obračunani rad po ugovoru o djelu, autor. hon."
"2906","kp_rrif2906","l10n_hr_chart_template_rrif","liability_current",,"Obračunani kalo, rastep, kvar i lom","Obračunani kalo, rastep, kvar i lom"
"2907","kp_rrif2907","l10n_hr_chart_template_rrif","liability_current",,"Obračunani troškovi premije osiguranja","Obračunani troškovi premije osiguranja"
"2908","kp_rrif2908","l10n_hr_chart_template_rrif","liability_current",,"Uračunani troškovi službenih glasila i stručnih časopisa","Uračunani troškovi službenih glasila i stručnih časopisa"
"2909","kp_rrif2909","l10n_hr_chart_template_rrif","liability_current",,"Obračunani ostali troškovi poslovanja (prijevoz, bankovne usluge i platni promet, reprez. i dr.)","Obračunani ostali troškovi poslovanja (troškovi prijevoza, bankovne usluge i platni promet, reprezentacija i dr.)"
"2910","kp_rrif2910","l10n_hr_chart_template_rrif","liability_current",,"Obračunani troškovi franšiza, korištenih trgovačkih znakova, prava i sl.","Obračunani troškovi franšiza, korištenih trgovačkih znakova, prava i sl."
"2911","kp_rrif2911","l10n_hr_chart_template_rrif","liability_current",,"Obračunani troškovi licencija","Obračunani troškovi licencija"
"2912","kp_rrif2912","l10n_hr_chart_template_rrif","liability_current",,"Obračunani troškovi autorskih prava","Obračunani troškovi autorskih prava"
"2913","kp_rrif2913","l10n_hr_chart_template_rrif","liability_current",,"Odgođeno plaćanje troškova za ostala prava","Odgođeno plaćanje troškova za ostala prava"
"2920","kp_rrif2920","l10n_hr_chart_template_rrif","liability_current",,"Obračunani ovisni troškovi nabave (za koje nisu primljeni računi)-prijevoz, osiguranje, špedicija i dr.","Obračunani ovisni troškovi nabave (za koje nisu primljeni računi) - prijevoz, osiguranje, špedicija i dr. vanjski troškovi"
"2930","kp_rrif2930","l10n_hr_chart_template_rrif","liability_current",,"Odgođeni prihodi radi neizvjesnih troškova","Odgođeni prihodi radi neizvjesnih troškova"
"2931","kp_rrif2931","l10n_hr_chart_template_rrif","liability_current",,"Obračunani prihodi budućeg razdoblja koji su rezultat primjene računovodstvene politike","Obračunani prihodi budućeg razdoblja koji su rezultat primjene računovodstvene politike"
"2932","kp_rrif2932","l10n_hr_chart_template_rrif","liability_current",,"Odgođeno priznavanje prihoda kad se predviđa povrat robe","Odgođeno priznavanje prihoda kad se predviđa povrat robe"
"2933","kp_rrif2933","l10n_hr_chart_template_rrif","liability_current",,"Odgođeni prihodi iz otkupa tražbine (faktoring koji nije zarađen)","Odgođeni prihodi iz otkupa tražbine (faktoring koji nije zarađen)"
"2934","kp_rrif2934","l10n_hr_chart_template_rrif","liability_current",,"Odgođeni prihodi od kamata iz financijskog lizinga","Odgođeni prihodi od kamata iz financijskog lizinga"
"2935","kp_rrif2935","l10n_hr_chart_template_rrif","liability_current",,"Odgođeni prihodi iz operativnog lizinga","Odgođeni prihodi iz operativnog lizinga"
"2936","kp_rrif2936","l10n_hr_chart_template_rrif","liability_current",,"Obračunane (anticipativne) kamate na dane zajmove i zatezne kamate (za buduće razdoblje)","Obračunane (anticipativne) kamate na dane zajmove i zatezne kamate (za buduće razdoblje)"
"2937","kp_rrif2937","l10n_hr_chart_template_rrif","liability_current",,"Unaprijed obračunane ili naplaćene školarine","Unaprijed obračunane ili naplaćene školarine"
"2938","kp_rrif2938","l10n_hr_chart_template_rrif","liability_current",,"Odgođeni prihodi iz ortaštva","Odgođeni prihodi iz ortaštva"
"2939","kp_rrif2939","l10n_hr_chart_template_rrif","liability_current",,"Ostali odgođeni prihodi budućeg razdoblja","Ostali odgođeni prihodi budućeg razdoblja"
"2940","kp_rrif2940","l10n_hr_chart_template_rrif","liability_current",,"Odgođeni prihodi iz potpora za dug. nemat. i mat. imovinu (analitike po primitcima, investicijama)","Odgođeni prihodi iz državnih i lokalnih potpora za dugotrajnu nematerijalnu i materijalnu imovinu (analitike po namjenskim primitcima iz proračuna - po investicijama)"
"2941","kp_rrif2941","l10n_hr_chart_template_rrif","liability_current",,"Odgođeni prihodi iz potpora za zapošljavanje","Odgođeni prihodi iz potpora za zapošljavanje"
"2942","kp_rrif2942","l10n_hr_chart_template_rrif","liability_current",,"Odgođeno priznavanje prihoda za unaprijed naplaćene subvencije i potpore","Odgođeno priznavanje prihoda za unaprijed naplaćene subvencije i potpore"
"2943","kp_rrif2943","l10n_hr_chart_template_rrif","liability_current",,"Odgođeni prihodi ostalih potpora","Odgođeni prihodi ostalih potpora"
"2950","kp_rrif2950","l10n_hr_chart_template_rrif","liability_current",,"Odgođeni prihodi zbog rizika naplate (HSFI t. 15.71 u svezi s t. 15.24)","Odgođeni prihodi zbog rizika naplate (HSFI t. 15.71 u svezi s t. 15.24)"
"2951","kp_rrif2951","l10n_hr_chart_template_rrif","liability_current",,"Razgraničeni višak prihoda od prodaje i povratnog financijskog najma (MRS 17, t. 59.)","Razgraničeni višak prihoda od prodaje i povratnog financijskog najma (MRS 17, t. 59.)"
"2960","kp_rrif2960","l10n_hr_chart_template_rrif","liability_current",,"Odgođeni prihod s osnove nefakturiranih isporuka dobara i usluga-a1","Odgođeni prihod s osnove nefakturiranih isporuka dobara i usluga-Analitika 1"
"2970","kp_rrif2970","l10n_hr_chart_template_rrif","liability_current",,"Nerealizirani dobitci iz financijske imovine (HSFI 9 i MRS 39)-a1","Nerealizirani dobitci iz financijske imovine (HSFI 9 i MRS 39)-Analitika 1"
"2980","kp_rrif2980","l10n_hr_chart_template_rrif","liability_current",,"Rezerviranje troška za neiskorištene godišnje odmore-a1","Rezerviranje troška za neiskorištene godišnje odmore-Analitika 1"
"2999","kp_rrif2999","l10n_hr_chart_template_rrif","liability_current",,"Ostala pasivna vremenska razgraničenja troškova","Ostala pasivna vremenska razgraničenja troškova"
"3000","kp_rrif3000","l10n_hr_chart_template_rrif","asset_current",,"Fakturna cijena sirovina i materijala na putu","Fakturna cijena sirovina i materijala na putu"
"3001","kp_rrif3001","l10n_hr_chart_template_rrif","asset_current",,"Fakturna cijena sirovina i materijala","Fakturna cijena sirovina i materijala"
"3002","kp_rrif3002","l10n_hr_chart_template_rrif","asset_current",,"Fakturna cijena rezervnih dijelova","Fakturna cijena rezervnih dijelova"
"3003","kp_rrif3003","l10n_hr_chart_template_rrif","asset_current",,"Fakturna cijena sitnog inventara, autoguma i ambalaže","Fakturna cijena sitnog inventara, autoguma i ambalaže"
"3005","kp_rrif3005","l10n_hr_chart_template_rrif","asset_current",,"Materijal i dijelovi u preuzimanju (nema dokumentacije)","Materijal i dijelovi u preuzimanju (nema dokumentacije)"
"3010","kp_rrif3010","l10n_hr_chart_template_rrif","asset_current",,"Troškovi transporta","Troškovi transporta"
"3011","kp_rrif3011","l10n_hr_chart_template_rrif","asset_current",,"Troškovi ukrcaja i iskrcaja (fakturirani)","Troškovi ukrcaja i iskrcaja (fakturirani)"
"3012","kp_rrif3012","l10n_hr_chart_template_rrif","asset_current",,"Transportno osiguranje i čuvanje","Transportno osiguranje i čuvanje"
"3013","kp_rrif3013","l10n_hr_chart_template_rrif","asset_current",,"Posebni troškovi pakiranja - ambalaže","Posebni troškovi pakiranja - ambalaže"
"3014","kp_rrif3014","l10n_hr_chart_template_rrif","asset_current",,"Troškovi vlastitog transporta","Troškovi vlastitog transporta"
"3015","kp_rrif3015","l10n_hr_chart_template_rrif","asset_current",,"Troškovi vlastitog ukrcaja i iskrcaja","Troškovi vlastitog ukrcaja i iskrcaja"
"3016","kp_rrif3016","l10n_hr_chart_template_rrif","asset_current",,"Troškovi špeditera","Troškovi špeditera"
"3017","kp_rrif3017","l10n_hr_chart_template_rrif","asset_current",,"Troškovi atesta i kontrole","Troškovi atesta i kontrole"
"3018","kp_rrif3018","l10n_hr_chart_template_rrif","asset_current",,"Troškovi dorade i oplemenjivanja za vrijeme dovođenja na zalihu","Troškovi dorade i oplemenjivanja za vrijeme dovođenja na zalihu"
"3019","kp_rrif3019","l10n_hr_chart_template_rrif","asset_current",,"Ostali ovisni troškovi nabave","Ostali ovisni troškovi nabave"
"3020","kp_rrif3020","l10n_hr_chart_template_rrif","asset_current",,"Carina i druge uvozne pristojbe-a1","Carina i druge uvozne pristojbe-Analitika 1"
"3030","kp_rrif3030","l10n_hr_chart_template_rrif","asset_current",,"Posebni porezi (trošarine) koji se ne mogu odbiti-a1","Posebni porezi (trošarine) koji se ne mogu odbiti-Analitika 1"
"3090","kp_rrif3090","l10n_hr_chart_template_rrif","asset_current",,"Obračun nabave sirovina i materijala, dijelova i sitnog inventara koji se skladišti","Obračun nabave sirovina i materijala, dijelova i sitnog inventara koji se skladišti"
"3091","kp_rrif3091","l10n_hr_chart_template_rrif","asset_current",,"Obračun nabave sirovina i materijala i dijelova koji izravno terete troškove","Obračun nabave sirovina i materijala i dijelova koji izravno terete troškove"
"3100","kp_rrif3100","l10n_hr_chart_template_rrif","asset_current",,"Zalihe sirovina i materijala","Zalihe sirovina i materijala"
"3101","kp_rrif3101","l10n_hr_chart_template_rrif","asset_current",,"Zalihe goriva i maziva","Zalihe goriva i maziva"
"3102","kp_rrif3102","l10n_hr_chart_template_rrif","asset_current",,"Poluproizvodi za ugradnju ili proizvodnju","Poluproizvodi za ugradnju ili proizvodnju"
"3103","kp_rrif3103","l10n_hr_chart_template_rrif","asset_current",,"Uredski materijal i pribor","Uredski materijal i pribor"
"3104","kp_rrif3104","l10n_hr_chart_template_rrif","asset_current",,"Zalihe materijala s temelja povezane proizvodnje","Zalihe materijala s temelja povezane proizvodnje"
"3105","kp_rrif3105","l10n_hr_chart_template_rrif","asset_current",,"Zalihe ambalažnog materijala","Zalihe ambalažnog materijala"
"3106","kp_rrif3106","l10n_hr_chart_template_rrif","asset_current",,"Zalihe materijala kod kooperanata","Zalihe materijala kod kooperanata"
"3107","kp_rrif3107","l10n_hr_chart_template_rrif","asset_current",,"Materijal na zalihi u javnom ili drugom skladištu","Materijal na zalihi u javnom ili drugom skladištu"
"3108","kp_rrif3108","l10n_hr_chart_template_rrif","asset_current",,"Zalihe pića, hrane i dr. u ugostiteljstvu i hotelijerstvu","Zalihe pića, hrane i dr. u ugostiteljstvu i hotelijerstvu"
"3109","kp_rrif3109","l10n_hr_chart_template_rrif","asset_current",,"Zalihe otpadnog i rashodovanog materijala","Zalihe otpadnog i rashodovanog materijala"
"3110","kp_rrif3110","l10n_hr_chart_template_rrif","asset_current",,"Materijal u doradi, obradi i oplemenjivanju","Materijal u doradi, obradi i oplemenjivanju"
"3111","kp_rrif3111","l10n_hr_chart_template_rrif","asset_current",,"Materijal u manipulaciji i na putu","Materijal u manipulaciji i na putu"
"3112","kp_rrif3112","l10n_hr_chart_template_rrif","asset_current",,"Troškovi dorade, obrade i oplemenjivanja","Troškovi dorade, obrade i oplemenjivanja"
"3120","kp_rrif3120","l10n_hr_chart_template_rrif","asset_current",,"Materijal na doradi kod ortaka-a1","Materijal na doradi kod ortaka-Analitika 1"
"3130","kp_rrif3130","l10n_hr_chart_template_rrif","asset_current",,"Zalihe sjemena i sadnog materijala","Zalihe sjemena i sadnog materijala"
"3131","kp_rrif3131","l10n_hr_chart_template_rrif","asset_current",,"Zalihe komponenti za proizvodnju","Zalihe komponenti za proizvodnju"
"3180","kp_rrif3180","l10n_hr_chart_template_rrif","asset_current",,"Odstupanje od cijene zaliha-a1","Odstupanje od cijene zaliha-Analitika 1"
"3190","kp_rrif3190","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađenje zaliha sirovina i materijala-a1","Vrijednosno usklađenje zaliha sirovina i materijala-Analitika 1"
"3200","kp_rrif3200","l10n_hr_chart_template_rrif","asset_current",,"Rezervni dijelovi za servisne usluge","Rezervni dijelovi za servisne usluge"
"3201","kp_rrif3201","l10n_hr_chart_template_rrif","asset_current",,"Dijelovi i sklopovi za ugradnju (u proizvode)","Dijelovi i sklopovi za ugradnju (u proizvode)"
"3202","kp_rrif3202","l10n_hr_chart_template_rrif","asset_current",,"Rezervni dijelovi za tekuće i investicijsko održavanje","Rezervni dijelovi za tekuće i investicijsko održavanje"
"3203","kp_rrif3203","l10n_hr_chart_template_rrif","asset_current",,"Zalihe polovnih rezervnih dijelova","Zalihe polovnih rezervnih dijelova"
"3204","kp_rrif3204","l10n_hr_chart_template_rrif","asset_current",,"Zalihe otpadaka rezervnih dijelova","Zalihe otpadaka rezervnih dijelova"
"3280","kp_rrif3280","l10n_hr_chart_template_rrif","asset_current",,"Odstupanje od cijene dijelova na zalihi-a1","Odstupanje od cijene dijelova na zalihi-Analitika 1"
"3290","kp_rrif3290","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađenje zaliha rezervnih dijelova-a1","Vrijednosno usklađenje zaliha rezervnih dijelova-Analitika 1"
"3500","kp_rrif3500","l10n_hr_chart_template_rrif","asset_current",,"Sitan inventar na zalihi (analitika prema vrstama: alati, mjerni instrumenti, pribori,odjeća, i dr.)","Sitan inventar na zalihi (analitika prema vrstama: alati, mjerni instrumenti, pribori, ostala sredstva rada male vrijednosti, radna i zaštitna odjeća, protupožarna i sanitetska sredstva, i dr.)"
"3510","kp_rrif3510","l10n_hr_chart_template_rrif","asset_current",,"Ambalaža na zalihi (samo vlastita i višekratna, analitika prema vrstama)-a1","Ambalaža na zalihi (samo vlastita i višekratna, analitika prema vrstama)-Analitika 1"
"3520","kp_rrif3520","l10n_hr_chart_template_rrif","asset_current",,"Autogume na zalihi-a1","Autogume na zalihi-Analitika 1"
"3580","kp_rrif3580","l10n_hr_chart_template_rrif","asset_current",,"Odstupanje od cijene-a1","Odstupanje od cijene-Analitika 1"
"3590","kp_rrif3590","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađenje zaliha-a1","Vrijednosno usklađenje zaliha-Analitika 1"
"3600","kp_rrif3600","l10n_hr_chart_template_rrif","asset_current",,"Sitan inventar u uporabi-a1","Sitan inventar u uporabi-Analitika 1"
"3610","kp_rrif3610","l10n_hr_chart_template_rrif","asset_current",,"Ambalaža u uporabi-a1","Ambalaža u uporabi-Analitika 1"
"3620","kp_rrif3620","l10n_hr_chart_template_rrif","asset_current",,"Autogume u uporabi-a1","Autogume u uporabi-Analitika 1"
"3630","kp_rrif3630","l10n_hr_chart_template_rrif","asset_current",,"Otpis sitnog inventara-a1","Otpis sitnog inventara-Analitika 1"
"3640","kp_rrif3640","l10n_hr_chart_template_rrif","asset_current",,"Otpis ambalaže-a1","Otpis ambalaže-Analitika 1"
"3650","kp_rrif3650","l10n_hr_chart_template_rrif","asset_current",,"Otpis autoguma-a1","Otpis autoguma-Analitika 1"
"3690","kp_rrif3690","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađenje sitnog inventara, ambalaže i autoguma-a1","Vrijednosno usklađenje sitnog inventara, ambalaže i autoguma-Analitika 1"
"3700","kp_rrif3700","l10n_hr_chart_template_rrif","asset_current",,"Predujmovi dobavljačima materijala-a1","Predujmovi dobavljačima materijala-Analitika 1"
"3710","kp_rrif3710","l10n_hr_chart_template_rrif","asset_current",,"Predujmovi dobavljačima rezervnih dijelova-a1","Predujmovi dobavljačima rezervnih dijelova-Analitika 1"
"3720","kp_rrif3720","l10n_hr_chart_template_rrif","asset_current",,"Predujmovi dobavljačima sitnog inventara-a1","Predujmovi dobavljačima sitnog inventara-Analitika 1"
"3730","kp_rrif3730","l10n_hr_chart_template_rrif","asset_current",,"Predujmovi dani uvozniku za nabavu sirovina i materijala, dijelova i inventara-a1","Predujmovi dani uvozniku za nabavu sirovina i materijala, dijelova i inventara-Analitika 1"
"3740","kp_rrif3740","l10n_hr_chart_template_rrif","asset_current",,"Predujmovi inozemnim dobavljačima-a1","Predujmovi inozemnim dobavljačima-Analitika 1"
"3790","kp_rrif3790","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađenje danih predujmova-a1","Vrijednosno usklađenje danih predujmova-Analitika 1"
"4000","kp_rrif4000","l10n_hr_chart_template_rrif","expense",,"Osnovni materijali i sirovine","Osnovni materijali i sirovine"
"4001","kp_rrif4001","l10n_hr_chart_template_rrif","expense",,"Dijelovi i sklopovi","Dijelovi i sklopovi"
"4002","kp_rrif4002","l10n_hr_chart_template_rrif","expense",,"Poluproizvodi za ugradnju","Poluproizvodi za ugradnju"
"4003","kp_rrif4003","l10n_hr_chart_template_rrif","expense",,"Pomoćni materijali (mazivo, ljepila, svrdla, pile, noževi, brusne ploče i dr.)","Pomoćni materijali (mazivo, ljepila, svrdla, pile, noževi, brusne ploče i dr.)"
"4004","kp_rrif4004","l10n_hr_chart_template_rrif","expense",,"Potrošni materijal za čišćenje i održavanje","Potrošni materijal za čišćenje i održavanje"
"4005","kp_rrif4005","l10n_hr_chart_template_rrif","expense",,"Materijal za HTZ zaštitu, radna i zaštitna odjeća i obuća","Materijal za HTZ zaštitu, radna i zaštitna odjeća i obuća"
"4006","kp_rrif4006","l10n_hr_chart_template_rrif","expense",,"Materijal pogonske administracije i menadžmenta (uredski potrošni i sl.)","Materijal pogonske administracije i menadžmenta (uredski potrošni i sl.)"
"4007","kp_rrif4007","l10n_hr_chart_template_rrif","expense",,"Troškovi oblikovanja proizvoda za posebne kupce","Troškovi oblikovanja proizvoda za posebne kupce"
"4008","kp_rrif4008","l10n_hr_chart_template_rrif","expense",,"Materijali u pomoćnoj djelatnosti (za restoran i dr.)","Materijali u pomoćnoj djelatnosti (za restoran i dr.)"
"4009","kp_rrif4009","l10n_hr_chart_template_rrif","expense",,"Ostali izravni i opći troškovi pogona - uslužne jedinice (HSFI t. 10.17 i MRS 2, t. 10. do 19.)","Ostali izravni i opći troškovi pogona - uslužne jedinice (HSFI t. 10.17 i MRS 2, t. 10. do 19.)"
"4010","kp_rrif4010","l10n_hr_chart_template_rrif","expense",,"Uredski materijal (papir, registratori, olovke, tiskanice, toneri, ulošci, kalendari, rokovnici i sl.)","Uredski materijal (papir, registratori, olovke, tiskanice, toneri, ulošci, kalendari, rokovnici i sl.)"
"4011","kp_rrif4011","l10n_hr_chart_template_rrif","expense",,"Materijal i sredstva za čišćenje i održavanje","Materijal i sredstva za čišćenje i održavanje"
"4012","kp_rrif4012","l10n_hr_chart_template_rrif","expense",,"Troškovi otpisa sitnog inventara","Troškovi otpisa sitnog inventara"
"4013","kp_rrif4013","l10n_hr_chart_template_rrif","expense",,"Ambalažni materijal, vrpce za blagajne, blokovi papira, pisači, naljepnice, etikete i dr.","Ambalažni materijal, vrpce za blagajne, blokovi papira, pisači, naljepnice, etikete i dr."
"4014","kp_rrif4014","l10n_hr_chart_template_rrif","expense",,"Voda (izvorska) za piće","Voda (izvorska) za piće"
"4015","kp_rrif4015","l10n_hr_chart_template_rrif","expense",,"Uniformirana radna odjeća i obuća","Uniformirana radna odjeća i obuća"
"4016","kp_rrif4016","l10n_hr_chart_template_rrif","expense",,"Troškovi opomena","Troškovi opomena"
"4017","kp_rrif4017","l10n_hr_chart_template_rrif","expense",,"Troškovi ukrasnog bilja","Troškovi ukrasnog bilja"
"4019","kp_rrif4019","l10n_hr_chart_template_rrif","expense",,"Ostali materijalni troškovi trgovine","Ostali materijalni troškovi trgovine"
"4020","kp_rrif4020","l10n_hr_chart_template_rrif","expense",,"Troškovi projekta za temeljna istraživanja proizvoda","Troškovi projekta za temeljna istraživanja proizvoda"
"4030","kp_rrif4030","l10n_hr_chart_template_rrif","expense",,"Troškovi neodvojive ambalaže u proizvodnji (boce, limenke, kutije i dr.)","Troškovi neodvojive ambalaže u proizvodnji (boce, limenke, kutije i dr.)"
"4031","kp_rrif4031","l10n_hr_chart_template_rrif","expense",,"Troškovi paleta, gajbi i sl.","Troškovi paleta, gajbi i sl."
"4040","kp_rrif4040","l10n_hr_chart_template_rrif","expense",,"Troškovi sitnog inventara,","Troškovi sitnog inventara,"
"4041","kp_rrif4041","l10n_hr_chart_template_rrif","expense",,"Troškovi ambalaže (povratne, posebne) - otpis","Troškovi ambalaže (povratne, posebne) - otpis"
"4042","kp_rrif4042","l10n_hr_chart_template_rrif","expense",,"Troškovi autoguma (za kamione, autobuse, teretna vozila i strojeve)","Troškovi autoguma (za kamione, autobuse, teretna vozila i strojeve)"
"4044","kp_rrif4044","l10n_hr_chart_template_rrif","expense",,"Trokš. auto guma (neto + 30% PDV) za slučaj plaće","Trokš. auto guma (neto + 30% PDV) za slučaj plaće"
"4045","kp_rrif4045","l10n_hr_chart_template_rrif","expense",,"70% troška autoguma za os. automobile i dr. sredstva prijevoza za potrebe administr., uprave i prodaje","70% troška autoguma za osobne automobile i dr. sredstva prijevoza za potrebe administr., uprave i prodaje"
"4046","kp_rrif4046","l10n_hr_chart_template_rrif","expense",,"30% troška inventara i autoguma za osobne automobile +30% PDV-a","30% troška inventara i autoguma za osobne automobile +30% PDV-a"
"4050","kp_rrif4050","l10n_hr_chart_template_rrif","expense",,"Potrošeni rezervni dijelovi za popravak vlastite opreme","Potrošeni rezervni dijelovi za popravak vlastite opreme"
"4051","kp_rrif4051","l10n_hr_chart_template_rrif","expense",,"Materijal za održavanje opreme i objekata","Materijal za održavanje opreme i objekata"
"4054","kp_rrif4054","l10n_hr_chart_template_rrif","expense",,"Trošak rez. dijelova (neto + 30% PDV) za slučaj plaće","Trošak rez. dijelova (neto + 30% PDV) za slučaj plaće"
"4055","kp_rrif4055","l10n_hr_chart_template_rrif","expense",,"70% troškova rez. dijelova i mat. za automob., plovila i zrakopl.za prijevoz(čl.7.,st.1.,t.4.ZoPD)","70% troškova rezervnih dijelova i materijala za popravak automob., plovila i zrakopl. koji služe za osobni prijevoz poduzet. i zaposlenih (čl. 7., st. 1., t. 4. ZoPD)"
"4056","kp_rrif4056","l10n_hr_chart_template_rrif","expense",,"30% troška rezervnih dijelova i materijala za održavanje automobila i dr. za osobni prijevoz +30% PDV-a","30% troška rezervnih dijelova i materijala za održavanje automobila i dr. za osobni prijevoz +30% PDV-a"
"4057","kp_rrif4057","l10n_hr_chart_template_rrif","expense",,"Troškovi zamjene u jamstvenom roku","Troškovi zamjene u jamstvenom roku"
"4058","kp_rrif4058","l10n_hr_chart_template_rrif","expense",,"Potrošeni vlastiti proizvodi i roba za održavanje","Potrošeni vlastiti proizvodi i roba za održavanje"
"4059","kp_rrif4059","l10n_hr_chart_template_rrif","expense",,"Ostali troškovi rezervnih dijelova","Ostali troškovi rezervnih dijelova"
"4060","kp_rrif4060","l10n_hr_chart_template_rrif","expense",,"Električna energija","Električna energija"
"4061","kp_rrif4061","l10n_hr_chart_template_rrif","expense",,"Plin, para, briketi i drva","Plin, para, briketi i drva"
"4062","kp_rrif4062","l10n_hr_chart_template_rrif","expense",,"Mazut i ulje za loženje","Mazut i ulje za loženje"
"4063","kp_rrif4063","l10n_hr_chart_template_rrif","expense",,"Dizelsko gorivo, benzin i motorno ulje (za stroj. i sl.)","Dizelsko gorivo, benzin i motorno ulje (za stroj. i sl.)"
"4067","kp_rrif4067","l10n_hr_chart_template_rrif","expense",,"Trošak goriva za teretna vozila (kamione, autobuse, strojeve, brodove i sl.)","Trošak goriva za teretna vozila (kamione, autobuse, strojeve, brodove i sl.)"
"4069","kp_rrif4069","l10n_hr_chart_template_rrif","expense",,"Ostali troškovi energije u proizvodnji","Ostali troškovi energije u proizvodnji"
"4070","kp_rrif4070","l10n_hr_chart_template_rrif","expense",,"Trošak električne energije","Trošak električne energije"
"4071","kp_rrif4071","l10n_hr_chart_template_rrif","expense",,"Plin, toplinska energija, briketi, drva","Plin, toplinska energija, briketi, drva"
"4074","kp_rrif4074","l10n_hr_chart_template_rrif","expense",,"Gorivo za osob. aut. (neto + 30% PDV) za sluč. plaće","Gorivo za osob. aut. (neto + 30% PDV) za sluč. plaće"
"4075","kp_rrif4075","l10n_hr_chart_template_rrif","expense",,"70% troškova goriva za pogon automobila za osobni prijevoz(i automobila u najmu)","70% troškova diesela i benzina za pogon automobila, plovila i zrakoplova za osobni prijevoz poduzetnika i zaposlenih te za istu namjenu automobila u najmu"
"4076","kp_rrif4076","l10n_hr_chart_template_rrif","expense",,"30% goriva za osobni prijevoz +30% PDV-a","30% goriva za osobni prijevoz +30% PDV-a"
"4077","kp_rrif4077","l10n_hr_chart_template_rrif","expense",,"Trošak goriva za teretna vozila, strojeve i brodove","Trošak goriva za teretna vozila, strojeve i brodove"
"4079","kp_rrif4079","l10n_hr_chart_template_rrif","expense",,"Ostali troškovi energije","Ostali troškovi energije"
"4080","kp_rrif4080","l10n_hr_chart_template_rrif","expense",,"Troškovi energije na pomoćnim mjestima u proizvodnji-a1","Troškovi energije na pomoćnim mjestima u proizvodnji-Analitika 1"
"4090","kp_rrif4090","l10n_hr_chart_template_rrif","expense",,"Odstupanja od standardnog troška-a1","Odstupanja od standardnog troška-Analitika 1"
"4100","kp_rrif4100","l10n_hr_chart_template_rrif","expense",,"Troškovi telefona, interneta i sl.","Troškovi telefona, interneta i sl."
"4101","kp_rrif4101","l10n_hr_chart_template_rrif","expense",,"Poštanski troškovi","Poštanski troškovi"
"4102","kp_rrif4102","l10n_hr_chart_template_rrif","expense",,"Prijevozne usluge u cestovnom prometu","Prijevozne usluge u cestovnom prometu"
"4103","kp_rrif4103","l10n_hr_chart_template_rrif","expense",,"Prijevozne usluge željeznicom","Prijevozne usluge željeznicom"
"4104","kp_rrif4104","l10n_hr_chart_template_rrif","expense",,"Prijevozne usluge brodara","Prijevozne usluge brodara"
"4105","kp_rrif4105","l10n_hr_chart_template_rrif","expense",,"Prijevozne usluge zrakoplova","Prijevozne usluge zrakoplova"
"4106","kp_rrif4106","l10n_hr_chart_template_rrif","expense",,"Troškovi specijalnih prijevoza","Troškovi specijalnih prijevoza"
"4107","kp_rrif4107","l10n_hr_chart_template_rrif","expense",,"Usluge taksi- prijevoza","Usluge taksi- prijevoza"
"4108","kp_rrif4108","l10n_hr_chart_template_rrif","expense",,"Usluge dostave i logistike","Usluge dostave i logistike"
"4109","kp_rrif4109","l10n_hr_chart_template_rrif","expense",,"Ostale usluge prijevoza","Ostale usluge prijevoza"
"4110","kp_rrif4110","l10n_hr_chart_template_rrif","expense",,"Usluge dorade (oplemenjivanja), izrade, prerade i sl. u proizvodnji i izgradnji","Usluge dorade (oplemenjivanja), izrade, prerade i sl. u proizvodnji i izgradnji"
"4111","kp_rrif4111","l10n_hr_chart_template_rrif","expense",,"Usluge kooperanata na zajedničkim uslugama prema trećima","Usluge kooperanata na zajedničkim uslugama prema trećima"
"4112","kp_rrif4112","l10n_hr_chart_template_rrif","expense",,"Usluge studentskog i omladinskog servisa i na izradi proizvoda","Usluge studentskog i omladinskog servisa i na izradi proizvoda"
"4113","kp_rrif4113","l10n_hr_chart_template_rrif","expense",,"Usluge pripreme teksta za tisak, za web. i sl.","Usluge pripreme teksta za tisak, za web. i sl."
"4114","kp_rrif4114","l10n_hr_chart_template_rrif","expense",,"Grafičke usluge tiska i uveza","Grafičke usluge tiska i uveza"
"4115","kp_rrif4115","l10n_hr_chart_template_rrif","expense",,"Usluge hotela i smještaja radnika na terenu","Usluge hotela i smještaja radnika na terenu"
"4116","kp_rrif4116","l10n_hr_chart_template_rrif","expense",,"Usluge za iznajmljeni kapacitet","Usluge za iznajmljeni kapacitet"
"4117","kp_rrif4117","l10n_hr_chart_template_rrif","expense",,"Usluge rada vanjskog osoblja","Usluge rada vanjskog osoblja"
"4118","kp_rrif4118","l10n_hr_chart_template_rrif","expense",,"Usluge izrade ili popravka po ugovoru o djelu","Usluge izrade ili popravka po ugovoru o djelu"
"4119","kp_rrif4119","l10n_hr_chart_template_rrif","expense",,"Ostale vanjske usluge na izradi dobara i proizvodnih usluga","Ostale vanjske usluge na izradi dobara i proizvodnih usluga"
"4120","kp_rrif4120","l10n_hr_chart_template_rrif","expense",,"Nabavljene usluge tekućeg održavanja (bez vlastitog materijala i dijelova)","Nabavljene usluge tekućeg održavanja (bez vlastitog materijala i dijelova)"
"4121","kp_rrif4121","l10n_hr_chart_template_rrif","expense",,"Nabavljene usluge za investicijsko održavanje i popravke (bez vlastitog materijala i dijelova)","Nabavljene usluge za investicijsko održavanje i popravke (bez vlastitog materijala i dijelova)"
"4122","kp_rrif4122","l10n_hr_chart_template_rrif","expense",,"Usluge čišćenja i pranja","Usluge čišćenja i pranja"
"4123","kp_rrif4123","l10n_hr_chart_template_rrif","expense",,"Usluge održavanja softvera i web stranica","Usluge održavanja softvera i web stranica"
"41244","kp_rrif41244","l10n_hr_chart_template_rrif","expense",,"Servis osob. automob. (neto + 30% PDV) za slučaj plaće u naravi","Servis osob. automob. (neto + 30% PDV) za slučaj plaće u naravi"
"4125","kp_rrif4125","l10n_hr_chart_template_rrif","expense",,"70% usluga servisa za održavanje automobila za osobni prijevoz poduzetnika i zaposlenih","70% usluga servisa za održavanje automobila, plovila i zrakoplova koji služe za osobni prijevoz poduzetnika i zaposlenih"
"4126","kp_rrif4126","l10n_hr_chart_template_rrif","expense",,"30% usluga održavanja prijevoznih sredstava za osobni prijevoz + 30% PDV-a","30% usluga održavanja prijevoznih sredstava za osobni prijevoz + 30% PDV-a"
"4127","kp_rrif4127","l10n_hr_chart_template_rrif","expense",,"Usluge zaštite na radu i održavanja okoliša","Usluge zaštite na radu i održavanja okoliša"
"4128","kp_rrif4128","l10n_hr_chart_template_rrif","expense",,"Usluge zaštitara na čuvanju imovine i osoba","Usluge zaštitara na čuvanju imovine i osoba"
"4129","kp_rrif4129","l10n_hr_chart_template_rrif","expense",,"Ostale servisne usluge i usluge osoba","Ostale servisne usluge i usluge osoba"
"4130","kp_rrif4130","l10n_hr_chart_template_rrif","expense",,"70% troška registracije automobila, plovila i zrakoplova za prijevoz osoba poduz. (osim osiguranja)","70% troška registracije osobnih automobila, plovila i zrakoplova za prijevoz osoba poduzetnika (osim osiguranja)"
"41314","kp_rrif41314","l10n_hr_chart_template_rrif","expense",,"Troškovi registr. (neto + 30% PDV) - plaća","Troškovi registr. (neto + 30% PDV) - plaća"
"4132","kp_rrif4132","l10n_hr_chart_template_rrif","expense",,"Trošak registracije dostavnih i teret. vozila i autobusa (sveuk.) i automob. bez poreznog ograničenja","Trošak registracije dostavnih i teret. vozila i autobusa (sveukupno) i automob. bez poreznog ograničenja"
"4133","kp_rrif4133","l10n_hr_chart_template_rrif","expense",,"Troškovi registracije plovila","Troškovi registracije plovila"
"4134","kp_rrif4134","l10n_hr_chart_template_rrif","expense",,"Troškovi registracije zrakoplova","Troškovi registracije zrakoplova"
"4135","kp_rrif4135","l10n_hr_chart_template_rrif","expense",,"Troškovi dozvola za prometne smjerove","Troškovi dozvola za prometne smjerove"
"4136","kp_rrif4136","l10n_hr_chart_template_rrif","expense",,"Troškovi koncesija, licencija i dr. prava na prijevoz","Troškovi koncesija, licencija i dr. prava na prijevoz"
"4137","kp_rrif4137","l10n_hr_chart_template_rrif","expense",,"Troškovi nadoknada za ceste, takse i sl.","Troškovi nadoknada za ceste, takse i sl."
"4139","kp_rrif4139","l10n_hr_chart_template_rrif","expense",,"Ostali troškovi registracije prometala","Ostali troškovi registracije prometala"
"4140","kp_rrif4140","l10n_hr_chart_template_rrif","expense",,"Zakupnine - najamnine nekretnina","Zakupnine - najamnine nekretnina"
"4141","kp_rrif4141","l10n_hr_chart_template_rrif","expense",,"Zakupnine opreme","Zakupnine opreme"
"4142","kp_rrif4142","l10n_hr_chart_template_rrif","expense",,"Usluge operativnog (poslovnog) lizinga opreme","Usluge operativnog (poslovnog) lizinga opreme"
"4143","kp_rrif4143","l10n_hr_chart_template_rrif","expense",,"70% usluga operativnog lizinga automobila, brodova i zrakoplova za osobni prijevoz osoba poduzetnika","70% usluga operativnog lizinga automobila, brodova i zrakoplova za osobni prijevoz osoba poduzetnika"
"4144","kp_rrif4144","l10n_hr_chart_template_rrif","expense",,"Usluge operat. najma osob. automob. (neto + 30% PDV) za slučaj plaće","Usluge operat. najma osob. automob. (neto + 30% PDV) za slučaj plaće"
"4145","kp_rrif4145","l10n_hr_chart_template_rrif","expense",,"70% rent-á-car usluge prijevoza osoba","70% rent-á-car usluge prijevoza osoba"
"4146","kp_rrif4146","l10n_hr_chart_template_rrif","expense",,"30% usluga operativnog lizinga sred. za osobni prijevoz + 30% PDV-a","30% usluga operativnog lizinga sred. za osobni prijevoz + 30% PDV-a"
"4147","kp_rrif4147","l10n_hr_chart_template_rrif","expense",,"30% rent-á-car usluga prijevoza osoba + 30% PDV-a","30% rent-á-car usluga prijevoza osoba + 30% PDV-a"
"4148","kp_rrif4148","l10n_hr_chart_template_rrif","expense",,"Rent-á-car za prijevoz tereta","Rent-á-car za prijevoz tereta"
"4149","kp_rrif4149","l10n_hr_chart_template_rrif","expense",,"Usluge najma informatičke opreme","Usluge najma informatičke opreme"
"4150","kp_rrif4150","l10n_hr_chart_template_rrif","expense",,"Troškovi promidžbe putem tiskovina, TV, plakata i sl.","Troškovi promidžbe putem tiskovina, TV, plakata i sl."
"4151","kp_rrif4151","l10n_hr_chart_template_rrif","expense",,"Usluge promidžbenih agencija","Usluge promidžbenih agencija"
"4152","kp_rrif4152","l10n_hr_chart_template_rrif","expense",,"Troškovi promidžbe u inozemstvu","Troškovi promidžbe u inozemstvu"
"4153","kp_rrif4153","l10n_hr_chart_template_rrif","expense",,"Trošak sponzoriranja športa i kulture u cilju promidžbe","Trošak sponzoriranja športa i kulture u cilju promidžbe"
"4154","kp_rrif4154","l10n_hr_chart_template_rrif","expense",,"Usluge unapređenja prodaje","Usluge unapređenja prodaje"
"4155","kp_rrif4155","l10n_hr_chart_template_rrif","expense",,"Usluge istraživanja tržišta","Usluge istraživanja tržišta"
"4156","kp_rrif4156","l10n_hr_chart_template_rrif","expense",,"Usluge sajmova (nadoknada za prostor)","Usluge sajmova (nadoknada za prostor)"
"4157","kp_rrif4157","l10n_hr_chart_template_rrif","expense",,"Usluge oblikovanja i uređenja izložbenog i prodajnog prostora","Usluge oblikovanja i uređenja izložbenog i prodajnog prostora"
"4158","kp_rrif4158","l10n_hr_chart_template_rrif","expense",,"Troškovi promidžbe najmom medija (stranica portala i sl.)","Troškovi promidžbe najmom medija (stranica portala i sl.)"
"4159","kp_rrif4159","l10n_hr_chart_template_rrif","expense",,"Ostali troškovi promidžbe (osim reprezentacije, dnevnica na službenom putu i sl.)","Ostali troškovi promidžbe (osim reprezentacije, dnevnica na službenom putu i sl.)"
"4160","kp_rrif4160","l10n_hr_chart_template_rrif","expense",,"Troškovi drugih dohodaka (ugovora o djelu, akvizitera, trgov. putnika, konzultanata)","Troškovi drugih dohodaka (ugovora o djelu, akvizitera, trgov. putnika, konzultanata)"
"4161","kp_rrif4161","l10n_hr_chart_template_rrif","expense",,"Autorski honorari (pisana, govorna, prijevodi i dr.)","Autorski honorari (pisana, govorna, prijevodi i dr.)"
"4162","kp_rrif4162","l10n_hr_chart_template_rrif","expense",,"Usluge specijalističkog obrazovanja, znanstvenoistraživačke usluge, usluge informacija i sl.","Usluge specijalističkog obrazovanja, znanstvenoistraživačke usluge, usluge informacija i sl."
"4163","kp_rrif4163","l10n_hr_chart_template_rrif","expense",,"Konzultantske i savjetničke usluge","Konzultantske i savjetničke usluge"
"4164","kp_rrif4164","l10n_hr_chart_template_rrif","expense",,"Knjigovodstvene usluge","Knjigovodstvene usluge"
"4165","kp_rrif4165","l10n_hr_chart_template_rrif","expense",,"Usluge poreznih savjetnika","Usluge poreznih savjetnika"
"4166","kp_rrif4166","l10n_hr_chart_template_rrif","expense",,"Usluge revizije i procjene vrijednosti poduzeća","Usluge revizije i procjene vrijednosti poduzeća"
"4167","kp_rrif4167","l10n_hr_chart_template_rrif","expense",,"Odvjetničke, bilježničke i usluge izrade pravnih akata","Odvjetničke, bilježničke i usluge izrade pravnih akata"
"4168","kp_rrif4168","l10n_hr_chart_template_rrif","expense",,"Naknade za korištenje prava intelektualnog vlasništva (licencije, ind. prava., robni znak i sl.)","Naknade za korištenje prava intelektualnog vlasništva (licencije, ind. prava., robni znak i sl.)"
"4169","kp_rrif4169","l10n_hr_chart_template_rrif","expense",,"Usluge vještačenja, administracijske usluge, i dr. intelektualne usluge","Usluge vještačenja, administracijske usluge, i dr. intelektualne usluge"
"4170","kp_rrif4170","l10n_hr_chart_template_rrif","expense",,"Komunalna naknada (za financ. izgradnje)","Komunalna naknada (za financ. izgradnje)"
"4171","kp_rrif4171","l10n_hr_chart_template_rrif","expense",,"Odvoz smeća i fekalija","Odvoz smeća i fekalija"
"4172","kp_rrif4172","l10n_hr_chart_template_rrif","expense",,"Voda i odvodnja","Voda i odvodnja"
"4173","kp_rrif4173","l10n_hr_chart_template_rrif","expense",,"Održavanje zelenila","Održavanje zelenila"
"4174","kp_rrif4174","l10n_hr_chart_template_rrif","expense",,"Usluge tržnica","Usluge tržnica"
"4175","kp_rrif4175","l10n_hr_chart_template_rrif","expense",,"Garažiranje i parkiranje vozila","Garažiranje i parkiranje vozila"
"4176","kp_rrif4176","l10n_hr_chart_template_rrif","expense",,"Deratizacija i dezinfekcijske usluge","Deratizacija i dezinfekcijske usluge"
"4177","kp_rrif4177","l10n_hr_chart_template_rrif","expense",,"Dimnjačarske i ekološke usluge","Dimnjačarske i ekološke usluge"
"4178","kp_rrif4178","l10n_hr_chart_template_rrif","expense",,"Veterinarske, sanitarne i usluge zbrinjavanja otpada","Veterinarske, sanitarne i usluge zbrinjavanja otpada"
"4179","kp_rrif4179","l10n_hr_chart_template_rrif","expense",,"Ostale komunalne i ekološke usluge","Ostale komunalne i ekološke usluge"
"4180","kp_rrif4180","l10n_hr_chart_template_rrif","expense",,"70% vanjskih usluga ugošćenja (reprezentacije) + 70% PDV-a","70% vanjskih usluga ugošćenja (reprezentacije) + 70% PDV-a"
"4181","kp_rrif4181","l10n_hr_chart_template_rrif","expense",,"30% vanjskih usluga ugošćenja (reprezentacije)","30% vanjskih usluga ugošćenja (reprezentacije)"
"4184","kp_rrif4184","l10n_hr_chart_template_rrif","expense",,"Usluge posredovanja pri nabavi dobara i usluga","Usluge posredovanja pri nabavi dobara i usluga"
"4185","kp_rrif4185","l10n_hr_chart_template_rrif","expense",,"Usluge posredovanja pri prodaji dobara i usluga","Usluge posredovanja pri prodaji dobara i usluga"
"4186","kp_rrif4186","l10n_hr_chart_template_rrif","expense",,"Usluge agenata i detektiva","Usluge agenata i detektiva"
"4187","kp_rrif4187","l10n_hr_chart_template_rrif","expense",,"Troškovi provizija za usluge","Troškovi provizija za usluge"
"4190","kp_rrif4190","l10n_hr_chart_template_rrif","expense",,"Usluge kontrole kakvoće i atestiranja dobara","Usluge kontrole kakvoće i atestiranja dobara"
"4191","kp_rrif4191","l10n_hr_chart_template_rrif","expense",,"Usluge studentskog servisa","Usluge studentskog servisa"
"4192","kp_rrif4192","l10n_hr_chart_template_rrif","expense",,"Hotelske usluge (u agencijskim poslovima)","Hotelske usluge (u agencijskim poslovima)"
"4193","kp_rrif4193","l10n_hr_chart_template_rrif","expense",,"Vanjskotrgovačke usluge","Vanjskotrgovačke usluge"
"4194","kp_rrif4194","l10n_hr_chart_template_rrif","expense",,"Špediterske usluge pri izvozu i sl.","Špediterske usluge pri izvozu i sl."
"4195","kp_rrif4195","l10n_hr_chart_template_rrif","expense",,"Troškovi oglašavanja u tisku za slobodna radna mjesta, objava fin. izvješća i sl. (osim promidžbe)","Troškovi oglašavanja u tisku za slobodna radna mjesta, objava fin. izvješća i sl. (osim promidžbe)"
"4196","kp_rrif4196","l10n_hr_chart_template_rrif","expense",,"Troškovi korištenja javnih skladišta, luka, pristaništa, hlađenja i sl.","Troškovi korištenja javnih skladišta, luka, pristaništa, hlađenja i sl."
"4197","kp_rrif4197","l10n_hr_chart_template_rrif","expense",,"Trošak autoputa, tunela i mostarina","Trošak autoputa, tunela i mostarina"
"4198","kp_rrif4198","l10n_hr_chart_template_rrif","expense",,"Troškovi fotokopiranja, prijepisa, izrade naljepnica i fotografija i sl.","Troškovi fotokopiranja, prijepisa, izrade naljepnica i fotografija i sl."
"4199","kp_rrif4199","l10n_hr_chart_template_rrif","expense",,"Ostali nespomenuti vanjski troškovi - usluge","Ostali nespomenuti vanjski troškovi - usluge"
"4200","kp_rrif4200","l10n_hr_chart_template_rrif","expense",,"Troškovi neto plaća uprave i prodaje","Troškovi neto plaća uprave i prodaje"
"4201","kp_rrif4201","l10n_hr_chart_template_rrif","expense",,"Troškovi neto plaća proizvodnje","Troškovi neto plaća proizvodnje"
"4202","kp_rrif4202","l10n_hr_chart_template_rrif","expense",,"Ostali povremeni primitci","Ostali povremeni primitci"
"4210","kp_rrif4210","l10n_hr_chart_template_rrif","expense",,"Uprava i prodaja","Uprava i prodaja"
"4211","kp_rrif4211","l10n_hr_chart_template_rrif","expense",,"Proizvodnja","Proizvodnja"
"4212","kp_rrif4212","l10n_hr_chart_template_rrif","expense",,"Ostali povremeni primitci","Ostali povremeni primitci"
"4220","kp_rrif4220","l10n_hr_chart_template_rrif","expense",,"Uprava i prodaja","Uprava i prodaja"
"4221","kp_rrif4221","l10n_hr_chart_template_rrif","expense",,"Proizvodnja","Proizvodnja"
"4222","kp_rrif4222","l10n_hr_chart_template_rrif","expense",,"Ostali povremeni primitci","Ostali povremeni primitci"
"4230","kp_rrif4230","l10n_hr_chart_template_rrif","expense",,"Uprava i prodaja","Uprava i prodaja"
"4231","kp_rrif4231","l10n_hr_chart_template_rrif","expense",,"Proizvodnja","Proizvodnja"
"4232","kp_rrif4232","l10n_hr_chart_template_rrif","expense",,"Ostali povremeni primitci","Ostali povremeni primitci - potpore i sl."
"4235","kp_rrif4235","l10n_hr_chart_template_rrif","expense",,"Doprinosi za beneficirani radni staž","Doprinosi za beneficirani radni staž"
"4240","kp_rrif4240","l10n_hr_chart_template_rrif","expense",,"Bruto plaće (privremeno v. napom. 2)-a1","Bruto plaće (privremeno v. napom. 2)-Analitika 1"
"4300","kp_rrif4300","l10n_hr_chart_template_rrif","expense",,"Amortizacija izdataka za razvoj","Amortizacija izdataka za razvoj"
"4301","kp_rrif4301","l10n_hr_chart_template_rrif","expense",,"Amortizacija koncesije, patenata i dr. prava","Amortizacija koncesije, patenata i dr. prava"
"4302","kp_rrif4302","l10n_hr_chart_template_rrif","expense",,"Amortizacija softvera i ost. prava","Amortizacija softvera i ost. prava"
"4303","kp_rrif4303","l10n_hr_chart_template_rrif","expense",,"Amortizacija goodwila","Amortizacija goodwila"
"4304","kp_rrif4304","l10n_hr_chart_template_rrif","expense",,"Amortizacija ostale nematerijalne imovine","Amortizacija ostale nematerijalne imovine"
"4310","kp_rrif4310","l10n_hr_chart_template_rrif","expense",,"Amortizacija građevina","Amortizacija građevina"
"4311","kp_rrif4311","l10n_hr_chart_template_rrif","expense",,"Amortizacija postrojenja","Amortizacija postrojenja"
"4312","kp_rrif4312","l10n_hr_chart_template_rrif","expense",,"Amortizacija opreme","Amortizacija opreme"
"4313","kp_rrif4313","l10n_hr_chart_template_rrif","expense",,"Amortizacija alata i inventara","Amortizacija alata i inventara"
"4314","kp_rrif4314","l10n_hr_chart_template_rrif","expense",,"Amortizacija transportnih sredstava","Amortizacija transportnih sredstava"
"4315","kp_rrif4315","l10n_hr_chart_template_rrif","expense",,"Amortizacija brodova","Amortizacija brodova"
"4316","kp_rrif4316","l10n_hr_chart_template_rrif","expense",,"Amortizacija poljoprivredne opreme","Amortizacija poljoprivredne opreme"
"4319","kp_rrif4319","l10n_hr_chart_template_rrif","expense",,"Amortizacija ostale mater. imovine","Amortizacija ostale mater. imovine"
"4320","kp_rrif4320","l10n_hr_chart_template_rrif","expense",,"70% amortizacije osob. aut. i dr. sred. prijevoza","70% amortizacije osob. aut. i dr. sred. prijevoza"
"4321","kp_rrif4321","l10n_hr_chart_template_rrif","expense",,"30% amortizacije osob. aut. i dr. sred. prijevoza","30% amortizacije osob. aut. i dr. sred. prijevoza"
"4322","kp_rrif4322","l10n_hr_chart_template_rrif","expense",,"Dio amortizacije osob. aut. i dr. sred. prijevoza u vrijednosti iznad 400.000,00 kn","Dio amortizacije osob. aut. i dr. sred. prijevoza u vrijednosti iznad 400.000,00 kn"
"4323","kp_rrif4323","l10n_hr_chart_template_rrif","expense",,"Amortizacija (otpis) nepriznatog PDV-a","Amortizacija (otpis) nepriznatog PDV-a"
"4324","kp_rrif4324","l10n_hr_chart_template_rrif","expense",,"Amortiz. osob. aut. za sluč. plaće (neto + nepriz. PDV)","Amortiz. osob. aut. za sluč. plaće (neto + nepriz. PDV)"
"4330","kp_rrif4330","l10n_hr_chart_template_rrif","expense",,"Amortizacija građev. objekata","Amortizacija građev. objekata"
"4331","kp_rrif4331","l10n_hr_chart_template_rrif","expense",,"Amortizacija računala, računalne opreme i programa te računalne mreže","Amortizacija računala, računalne opreme i programa te računalne mreže"
"4332","kp_rrif4332","l10n_hr_chart_template_rrif","expense",,"Amortizacija osobnih automobila","Amortizacija osobnih automobila"
"4333","kp_rrif4333","l10n_hr_chart_template_rrif","expense",,"Amortizacija ostale opreme (pokućstvo, telefonija i dr.)","Amortizacija ostale opreme (pokućstvo, telefonija i dr.)"
"4340","kp_rrif4340","l10n_hr_chart_template_rrif","expense",,"Povećana amortizacija s temelja revalorizacije-a1","Povećana amortizacija s temelja revalorizacije-Analitika 1"
"4350","kp_rrif4350","l10n_hr_chart_template_rrif","expense",,"Amortizacija biološke imovine (vinogradi, voćnjaci, osnovno stado i sl.)-a1","Amortizacija biološke imovine (vinogradi, voćnjaci, osnovno stado i sl.)-Analitika 1"
"4360","kp_rrif4360","l10n_hr_chart_template_rrif","expense",,"Amortizacija iznad porezno dopuštene-a1","Amortizacija iznad porezno dopuštene-Analitika 1"
"4400","kp_rrif4400","l10n_hr_chart_template_rrif","expense",,"Vrijednosno usklađenje prava i dr.","Vrijednosno usklađenje prava i dr."
"4401","kp_rrif4401","l10n_hr_chart_template_rrif","expense",,"Vrijednosno usklađenje goodwill","Vrijednosno usklađenje goodwill"
"4402","kp_rrif4402","l10n_hr_chart_template_rrif","expense",,"Vrijednosno usklađenje ostale nemat. imov.","Vrijednosno usklađenje ostale nemat. imov."
"4410","kp_rrif4410","l10n_hr_chart_template_rrif","expense",,"Vrijednosno usklađenje nekretnina (028)","Vrijednosno usklađenje nekretnina (028)"
"4411","kp_rrif4411","l10n_hr_chart_template_rrif","expense",,"Vrijednosno usklađenje postrojenja, opreme, alata, pogonskog inventara i transportne imovine (038)","Vrijednosno usklađenje postrojenja, opreme, alata, pogonskog inventara i transportne imovine (038)"
"4412","kp_rrif4412","l10n_hr_chart_template_rrif","expense",,"Vrijednosno usklađenje biološke imovine (048)","Vrijednosno usklađenje biološke imovine (048)"
"4413","kp_rrif4413","l10n_hr_chart_template_rrif","expense",,"Vrijednosno usklađenje ulaganja u nekretnine (058)","Vrijednosno usklađenje ulaganja u nekretnine (058)"
"4414","kp_rrif4414","l10n_hr_chart_template_rrif","expense",,"Vrijednosno usklađenje ostale mater. imovine","Vrijednosno usklađenje ostale mater. imovine"
"4420","kp_rrif4420","l10n_hr_chart_template_rrif","expense",,"Vrijednosno usklađenje dugotrajnih potraživanja (veza sa 078)-a1","Vrijednosno usklađenje dugotrajnih potraživanja (veza sa 078)-Analitika 1"
"4440","kp_rrif4440","l10n_hr_chart_template_rrif","expense",,"Vrijednosno usklađenje depozita u bankama, mjenica, čekova i sl. (109 i dio 119)-a1","Vrijednosno usklađenje depozita u bankama, mjenica, čekova i sl. (109 i dio 119)-Analitika 1"
"4450","kp_rrif4450","l10n_hr_chart_template_rrif","expense",,"Vrijednosna usklađenja potraživanja od kupaca nenaplaćena dulje od 120 dana od dospijeća (veza s 129)","Vrijednosna usklađenja potraživanja od kupaca koja nisu naplaćena dulje od 120 dana od dospijeća (veza s 129)"
"4451","kp_rrif4451","l10n_hr_chart_template_rrif","expense",,"Vrijed. usklađenja utuženih kratk. pot. do 5000,00kn (prije zastare,ovršni,stečaj i sl.-dio129,139i159)","Vrijednosna usklađenja kratkoročnih potraživanja od kupaca i drugih koja su utužena (prije zastare, koja su u ovršnom postupku, otvoren je stečaj, nagodba i sl. - dio 129, 139 i 159) i svote do 5.000,00 kn"
"4452","kp_rrif4452","l10n_hr_chart_template_rrif","expense",,"Vrijednosna usklađenja zastarjelih potraživanja (dio sa 129, 139 i 159) - porezno nepriznata","Vrijednosna usklađenja zastarjelih potraživanja (dio sa 129, 139 i 159) - porezno nepriznata"
"4460","kp_rrif4460","l10n_hr_chart_template_rrif","expense",,"Vrijednosno usklađenje zaliha (veza s 319, 329, 359 i 369)-a1","Vrijednosno usklađenje zaliha (veza s 319, 329, 359 i 369)-Analitika 1"
"4470","kp_rrif4470","l10n_hr_chart_template_rrif","expense",,"Vrijednosno usklađenje danih predujmova (veza s 379)-a1","Vrijednosno usklađenje danih predujmova (veza s 379)-Analitika 1"
"4500","kp_rrif4500","l10n_hr_chart_template_rrif","expense",,"Troškovi dugoročnog rezerviranja za rizike u jamstvenom (garancijskom) roku (čl. 11., st. 2. ZoPD)-a1","Troškovi dugoročnog rezerviranja za rizike u jamstvenom (garancijskom) roku (čl. 11., st. 2. ZoPD)-Analitika 1"
"4510","kp_rrif4510","l10n_hr_chart_template_rrif","expense",,"Troškovi dugoročnog rezerviranja za gubitke po započetim sudskim sporovima (čl. 11., st. 2. ZoPD)-a1","Troškovi dugoročnog rezerviranja za gubitke po započetim sudskim sporovima (čl. 11., st. 2. ZoPD)-Analitika 1"
"4520","kp_rrif4520","l10n_hr_chart_template_rrif","expense",,"Troškovi dugoročnog rezerviranja za obnovu prirodnog bogatstva (čl. 11., st. 2. ZoPD)-a1","Troškovi dugoročnog rezerviranja za obnovu prirodnog bogatstva (čl. 11., st. 2. ZoPD)-Analitika 1"
"4530","kp_rrif4530","l10n_hr_chart_template_rrif","expense",,"Troškovi dugoročnog rezerviranja za otpremnine (čl. 11., st. 2. ZoPD)-a1","Troškovi dugoročnog rezerviranja za otpremnine (čl. 11., st. 2. ZoPD)-Analitika 1"
"4540","kp_rrif4540","l10n_hr_chart_template_rrif","expense",,"Troškovi dugoroč. rez. za neiskorišteni godišnji odmor (čl. 11., st. 5. ZoPD i MRS 19) - vidi rač. 298-a1","Troškovi dugoroč. rez. za neiskorišteni godišnji odmor (čl. 11., st. 5. ZoPD i MRS 19) - vidi rač. 298-Analitika 1"
"4550","kp_rrif4550","l10n_hr_chart_template_rrif","expense",,"Troškovi dugoročnog rezerviranja za restrukturiranje poduzeća (MRS 37, t. 72. i HSFI t. 16.22)-a1","Troškovi dugoročnog rezerviranja za restrukturiranje poduzeća (MRS 37, t. 72. i HSFI t. 16.22)-Analitika 1"
"4560","kp_rrif4560","l10n_hr_chart_template_rrif","expense",,"Troškovi dugoročnog rezerviranja za mirovine i slične troškove - obveze (MRS 19)-a1","Troškovi dugoročnog rezerviranja za mirovine i slične troškove - obveze (MRS 19)-Analitika 1"
"4570","kp_rrif4570","l10n_hr_chart_template_rrif","expense",,"Troškovi rezerviranja po štetnim ugovorima (HSFI 16.21.)-a1","Troškovi rezerviranja po štetnim ugovorima (HSFI 16.21.)-Analitika 1"
"4590","kp_rrif4590","l10n_hr_chart_template_rrif","expense",,"Troškovi ostalih dugoročnih rezerviranja i troškovi rizika-a1","Troškovi ostalih dugoročnih rezerviranja i troškovi rizika-Analitika 1"
"4600","kp_rrif4600","l10n_hr_chart_template_rrif","expense",,"Dnevnice za službena putovanja i troškovi noćenja u Hrvatskoj","Dnevnice za službena putovanja i troškovi noćenja u Hrvatskoj"
"4601","kp_rrif4601","l10n_hr_chart_template_rrif","expense",,"Dnevnice za službena putovanja u inozemstvu","Dnevnice za službena putovanja u inozemstvu"
"4602","kp_rrif4602","l10n_hr_chart_template_rrif","expense",,"Troškovi uporabe vlastitog automobila na službenom putu","Troškovi uporabe vlastitog automobila na službenom putu"
"4603","kp_rrif4603","l10n_hr_chart_template_rrif","expense",,"Terenski dodatak - pomorski dodatak","Terenski dodatak - pomorski dodatak"
"4604","kp_rrif4604","l10n_hr_chart_template_rrif","expense",,"Doprinos za zdravstveno osiguranje na službena putovanja u inozemstvo","Doprinos za zdravstveno osiguranje na službena putovanja u inozemstvo"
"4605","kp_rrif4605","l10n_hr_chart_template_rrif","expense",,"Troškovi noćenja (po računu hotela i dr.)","Troškovi noćenja (po računu hotela i dr.)"
"4606","kp_rrif4606","l10n_hr_chart_template_rrif","expense",,"Ostali troškovi na službenom putu (trošak autoceste, tunela, parkiranja, trajekta i dr.)","Ostali troškovi na službenom putu (trošak autoceste, tunela, parkiranja, trajekta i dr.)"
"4607","kp_rrif4607","l10n_hr_chart_template_rrif","expense",,"Troškovi službenog puta vanjskih suradnika (bruto s porezima i doprinosima)","Troškovi službenog puta vanjskih suradnika (bruto s porezima i doprinosima)"
"4610","kp_rrif4610","l10n_hr_chart_template_rrif","expense",,"Troškovi prijevoza na posao i s posla","Troškovi prijevoza na posao i s posla"
"4611","kp_rrif4611","l10n_hr_chart_template_rrif","expense",,"Loko vožnja - Nadoknada za uporabu privatnog automobila u poslovne svrhe za lokalnu vožnju","Nadoknada za uporabu privatnog automobila u poslovne svrhe za lokalnu vožnju"
"4612","kp_rrif4612","l10n_hr_chart_template_rrif","expense",,"Nadoknade za odvojeni život","Nadoknade za odvojeni život"
"46130","kp_rrif46130","l10n_hr_chart_template_rrif","expense",,"Stipendije i nagrade učenicima i studentima do neoporezivih svota","Stipendije i nagrade učenicima i studentima do neoporezivih svota"
"46131","kp_rrif46131","l10n_hr_chart_template_rrif","expense",,"Stipendije i nagrade učenicima i studentima iznad neoporezivih svota","Stipendije i nagrade učenicima i studentima iznad neoporezivih svota"
"4614","kp_rrif4614","l10n_hr_chart_template_rrif","expense",,"Otpremnine (odlazak u mirovinu, otkaz i teh.viška-čl.119.ZOR-a,ozljeda ili prof.bolesti (čl.80.ZOR-a)","Otpremnine (zbog odlaska radnika u mirovinu i otpremnine zbog danih otkaza i tehnološkog viška - čl. 119. ZOR-a, te otpremnine zbog ozljede ili profesionalne bolesti (čl. 80. ZOR-a)"
"4615","kp_rrif4615","l10n_hr_chart_template_rrif","expense",,"Darovi djeci i slične potpore (ako nisu dohodak)","Darovi djeci i slične potpore (ako nisu dohodak)"
"4616","kp_rrif4616","l10n_hr_chart_template_rrif","expense",,"Prigodne nagrade (božićnice,uskrsnice,u naravi do 400kn, regres, jubilarne i sl., do 2500kn god.)","Prigodne nagrade (božićnice, uskrsnice, dar u naravi (do 400,00 kn god., regres za god. odmor, jubilarne nagrade i sl., do 2.500,00 kn god.)"
"4617","kp_rrif4617","l10n_hr_chart_template_rrif","expense",,"Potpora zbog bolesti, invalidnosti, smrti, elementarnih nepogoda i sl.","Potpora zbog bolesti, invalidnosti, smrti, elementarnih nepogoda i sl."
"4618","kp_rrif4618","l10n_hr_chart_template_rrif","expense",,"Potpore i pomoći iznad neoporezivih svota","Potpore i pomoći iznad neoporezivih svota"
"4619","kp_rrif4619","l10n_hr_chart_template_rrif","expense",,"Ostali troškovi zaposlenika","Ostali troškovi zaposlenika"
"4620","kp_rrif4620","l10n_hr_chart_template_rrif","expense",,"Nadoknade članovima nadzornog odbora","Nadoknade članovima nadzornog odbora"
"4621","kp_rrif4621","l10n_hr_chart_template_rrif","expense",,"Nadoknade vanjskim članovima uprave","Nadoknade vanjskim članovima uprave"
"4622","kp_rrif4622","l10n_hr_chart_template_rrif","expense",,"Nadoknade stečajnim upraviteljima","Nadoknade stečajnim upraviteljima"
"4623","kp_rrif4623","l10n_hr_chart_template_rrif","expense",,"Nadoknade prokuristima, članovima skupštine društva i dr.","Nadoknade prokuristima, članovima skupštine društva i dr."
"4624","kp_rrif4624","l10n_hr_chart_template_rrif","expense",,"Troškovi usluga vanjske uprave (po računu)","Troškovi usluga vanjske uprave (po računu)"
"4625","kp_rrif4625","l10n_hr_chart_template_rrif","expense",,"Troškovi honorara vanjskim članovima uprave","Troškovi honorara vanjskim članovima uprave"
"4626","kp_rrif4626","l10n_hr_chart_template_rrif","expense",,"Godišnje nagrade članovima uprave","Godišnje nagrade članovima uprave"
"4630","kp_rrif4630","l10n_hr_chart_template_rrif","expense",,"70% troškova reprezentacije u darovima (robi i proizvodima + 70% PDV-a","70% troškova reprezentacije u darovima (robi i proizvodima + 70% PDV-a"
"4631","kp_rrif4631","l10n_hr_chart_template_rrif","expense",,"70% troškova vlastitih usluga za reprezentaciju + 70% PDV-a","70% troškova vlastitih usluga za reprezentaciju + 70% PDV-a"
"4632","kp_rrif4632","l10n_hr_chart_template_rrif","expense",,"70% troškova reprezentacije od uporabe vlastitih brodova, automobila, nekretnina i sl. + 70% PDV-a","70% troškova reprezentacije od uporabe vlastitih brodova, automobila, nekretnina i sl. + 70% PDV-a"
"4633","kp_rrif4633","l10n_hr_chart_template_rrif","expense",,"30% od svih neto-troškova reprezentacije (vlastitih)","30% od svih neto-troškova reprezentacije (vlastitih)"
"4635","kp_rrif4635","l10n_hr_chart_template_rrif","expense",,"Troškovi promidžbe (u katalozima, letcima, nagradne igre)","Troškovi promidžbe (u katalozima, letcima, nagradne igre)"
"4636","kp_rrif4636","l10n_hr_chart_template_rrif","expense",,"Troškovi promidžbe u proizvodima ili robi (""nije za prodaju"" do 80,00 kn )","Troškovi promidžbe u proizvodima ili robi (s oznakom ""nije za prodaju"" s nazivom tvrtke ili proizvoda pojedinačne vrijedn. do 80,00 kn - čaše, pepeljare, stoljnjaci, podmetači, olovke, rokovnici, upaljači, privjesci i sl.)"
"4640","kp_rrif4640","l10n_hr_chart_template_rrif","expense",,"Troškovi osiguranja dugotrajne materijalne i nematerijalne imovine","Troškovi osiguranja dugotrajne materijalne i nematerijalne imovine"
"4641","kp_rrif4641","l10n_hr_chart_template_rrif","expense",,"Premije osiguranja osoba (opasni poslovi, prenošenje novca, putnici i sl.)","Premije osiguranja osoba (opasni poslovi, prenošenje novca, putnici i sl.)"
"4642","kp_rrif4642","l10n_hr_chart_template_rrif","expense",,"Premije osiguranja prometnih sredstava (uključivo i kasko)","Premije osiguranja prometnih sredstava (uključivo i kasko)"
"4643","kp_rrif4643","l10n_hr_chart_template_rrif","expense",,"Transportno osiguranje dobara","Transportno osiguranje dobara"
"4644","kp_rrif4644","l10n_hr_chart_template_rrif","expense",,"Premije za zdravstveno osiguranje","Premije za zdravstveno osiguranje"
"4645","kp_rrif4645","l10n_hr_chart_template_rrif","expense",,"Troškovi premija životnog osiguranja (ugovaratelj i korisnik je trgovačko društvo)","Troškovi premija životnog osiguranja (ugovaratelj i korisnik je trgovačko društvo)"
"4646","kp_rrif4646","l10n_hr_chart_template_rrif","expense",,"Premije dobrovoljnog mirov. osig. (do 6000kn god. po radniku-čl.10.Zak. o porezu na dohodak -NN80/10)","Premije dobrovoljnog mirov. osig. (do 6.000 kn godišnje po radniku - čl. 10. Zak. o porezu na dohodak - Nar. nov., br. 80/10.)"
"4647","kp_rrif4647","l10n_hr_chart_template_rrif","expense",,"Premije za dokup mirovine zaposlenicima (III. stup) MRS 19 t. 43. i 44.","Premije za dokup mirovine zaposlenicima (III. stup) MRS 19 t. 43. i 44."
"4649","kp_rrif4649","l10n_hr_chart_template_rrif","expense",,"Premije za ostale oblike osiguranja","Premije za ostale oblike osiguranja"
"4650","kp_rrif4650","l10n_hr_chart_template_rrif","expense",,"Troškovi platnog prometa","Troškovi platnog prometa"
"4651","kp_rrif4651","l10n_hr_chart_template_rrif","expense",,"Troškovi provizija (pri kupnji deviza, brokeru i dr.)","Troškovi provizija (pri kupnji deviza, brokeru i dr.)"
"4652","kp_rrif4652","l10n_hr_chart_template_rrif","expense",,"Bankovne usluge (za inozemni platni promet, tečajnu maržu i sl.)","Bankovne usluge (za inozemni platni promet, tečajnu maržu i sl.)"
"4653","kp_rrif4653","l10n_hr_chart_template_rrif","expense",,"Troškovi provizija izdavatelja kreditnih kartica","Troškovi provizija izdavatelja kreditnih kartica"
"4654","kp_rrif4654","l10n_hr_chart_template_rrif","expense",,"Troškovi obrade kredita","Troškovi obrade kredita"
"4655","kp_rrif4655","l10n_hr_chart_template_rrif","expense",,"Troškovi akreditiva","Troškovi akreditiva"
"4656","kp_rrif4656","l10n_hr_chart_template_rrif","expense",,"Troškovi bankovne garancije","Troškovi bankovne garancije"
"4657","kp_rrif4657","l10n_hr_chart_template_rrif","expense",,"Ostali bankovni troškovi","Ostali bankovni troškovi"
"4660","kp_rrif4660","l10n_hr_chart_template_rrif","expense",,"Članarine komori (HGK ili HOK) i dopr. za javne ovlasti","Članarine komori (HGK ili HOK) i dopr. za javne ovlasti"
"4661","kp_rrif4661","l10n_hr_chart_template_rrif","expense",,"Članarine udrugama i strukovnim komorama","Članarine udrugama i strukovnim komorama"
"4662","kp_rrif4662","l10n_hr_chart_template_rrif","expense",,"Nadoknada za općekorisnu funkciju šuma (0,0525%)","Nadoknada za općekorisnu funkciju šuma (0,0525%)"
"4663","kp_rrif4663","l10n_hr_chart_template_rrif","expense",,"Članarina turističkoj zajednici","Članarina turističkoj zajednici"
"4664","kp_rrif4664","l10n_hr_chart_template_rrif","expense",,"Nadoknada za korištenje mineralnih sirovina","Nadoknada za korištenje mineralnih sirovina"
"4665","kp_rrif4665","l10n_hr_chart_template_rrif","expense",,"Pričuva za održavanje zgrade (Zakon o vlasništvu - Nar. nov., br. 91/96. do 38/09.)","Pričuva za održavanje zgrade (Zakon o vlasništvu - Nar. nov., br. 91/96. do 38/09.)"
"4666","kp_rrif4666","l10n_hr_chart_template_rrif","expense",,"Dozvole za korištenje autocesta, atesta, certifikata i sl.","Dozvole za korištenje autocesta, atesta, certifikata i sl."
"4667","kp_rrif4667","l10n_hr_chart_template_rrif","expense",,"Članarina za kreditne i potrošačke kartice","Članarina za kreditne i potrošačke kartice"
"4668","kp_rrif4668","l10n_hr_chart_template_rrif","expense",,"Spomenička renta","Spomenička renta"
"4669","kp_rrif4669","l10n_hr_chart_template_rrif","expense",,"Ostala davanja","Ostala davanja"
"4670","kp_rrif4670","l10n_hr_chart_template_rrif","expense",,"Porez na tvrtku odnosno naziv","Porez na tvrtku odnosno naziv"
"4671","kp_rrif4671","l10n_hr_chart_template_rrif","expense",,"Porez (imovinski) na cestovna vozila, plovne objekte i zrakoplove","Porez (imovinski) na cestovna vozila, plovne objekte i zrakoplove"
"4672","kp_rrif4672","l10n_hr_chart_template_rrif","expense",,"Porez na reklame koje se ističu na javnim mjestima","Porez na reklame koje se ističu na javnim mjestima"
"4673","kp_rrif4673","l10n_hr_chart_template_rrif","expense",,"Naknade Fondu za ambalažu (prema materijalu, po jed. proizv., povratna i poticajna ambalaža)","Naknade Fondu za ambalažu (prema materijalu, po jed. proizv., povratna i poticajna ambalaža)"
"4674","kp_rrif4674","l10n_hr_chart_template_rrif","expense",,"Porez na kuće za odmor","Porez na kuće za odmor"
"4675","kp_rrif4675","l10n_hr_chart_template_rrif","expense",,"Trošak poreza koji je ugovorno preuzet pri prodaji (npr. 5% p.p.n.)","Trošak poreza koji je ugovorno preuzet pri prodaji (npr. 5% p.p.n.)"
"46760","kp_rrif46760","l10n_hr_chart_template_rrif","expense",,"Trošak PDV-a iz vlastite potrošnje (ako već nije sadržan u trošku)","Trošak PDV-a iz vlastite potrošnje (ako već nije sadržan u trošku)"
"46761","kp_rrif46761","l10n_hr_chart_template_rrif","expense",,"Trošak PDV-a za koji je prestalo pravo na pretporez","Trošak PDV-a za koji je prestalo pravo na pretporez"
"46762","kp_rrif46762","l10n_hr_chart_template_rrif","expense",,"30% PDV-a na osobne automobile i dr. sredstva osobnog prijevoza","30% PDV-a na osobne automobile i dr. sredstva osobnog prijevoza"
"46763","kp_rrif46763","l10n_hr_chart_template_rrif","expense",,"PDV na osobne automobile i dr. sred. prijevoza na dio n. v. iznad 400.000,00 kn","PDV na osobne automobile i dr. sred. prijevoza na dio n. v. iznad 400.000,00 kn"
"4677","kp_rrif4677","l10n_hr_chart_template_rrif","expense",,"Porez po odbitku","Porez po odbitku"
"4678","kp_rrif4678","l10n_hr_chart_template_rrif","expense",,"Troškovi naknadno utvrđenih poreza (npr. PDV, posebni i dr. porezi, osim poreza na dobitak)","Troškovi naknadno utvrđenih poreza (npr. PDV, posebni i dr. porezi, osim poreza na dobitak)"
"46790","kp_rrif46790","l10n_hr_chart_template_rrif","expense",,"Porez i carina pri izvozu","Porez i carina pri izvozu"
"4680","kp_rrif4680","l10n_hr_chart_template_rrif","expense",,"Troškovi koncesije","Troškovi koncesije"
"4681","kp_rrif4681","l10n_hr_chart_template_rrif","expense",,"Troškovi franšiza, know howa, patenata, uporabe imena, znaka i dr.","Troškovi franšiza, know howa, patenata, uporabe imena, znaka i dr."
"4682","kp_rrif4682","l10n_hr_chart_template_rrif","expense",,"Troškovi prava na proizvodni i sl. postupak","Troškovi prava na proizvodni i sl. postupak"
"4683","kp_rrif4683","l10n_hr_chart_template_rrif","expense",,"Trošak prava na model, nacrt, formulu, plan, iskustvo i sl.","Trošak prava na model, nacrt, formulu, plan, iskustvo i sl."
"4684","kp_rrif4684","l10n_hr_chart_template_rrif","expense",,"Trošak HRT pretplate","Trošak HRT pretplate"
"4685","kp_rrif4685","l10n_hr_chart_template_rrif","expense",,"Troškovi licenciranih prava","Troškovi licenciranih prava"
"4686","kp_rrif4686","l10n_hr_chart_template_rrif","expense",,"Troškovi prava uporabe računalnih programa","Troškovi prava uporabe računalnih programa"
"4689","kp_rrif4689","l10n_hr_chart_template_rrif","expense",,"Ostali troškovi prava korištenja","Ostali troškovi prava korištenja"
"46900","kp_rrif46900","l10n_hr_chart_template_rrif","expense",,"Opće obrazovanje","Opće obrazovanje"
"46901","kp_rrif46901","l10n_hr_chart_template_rrif","expense",,"Posebno obrazovanje","Posebno obrazovanje"
"4691","kp_rrif4691","l10n_hr_chart_template_rrif","expense",,"Troškovi za priručnike, časopise i stručnu literaturu","Troškovi za priručnike, časopise i stručnu literaturu"
"4692","kp_rrif4692","l10n_hr_chart_template_rrif","expense",,"Troškovi službenih glasila","Troškovi službenih glasila"
"4693","kp_rrif4693","l10n_hr_chart_template_rrif","expense",,"Sudski troškovi i pristojbe","Sudski troškovi i pristojbe"
"4694","kp_rrif4694","l10n_hr_chart_template_rrif","expense",,"Troškovi zdravstvenog nadzora i kontrola proizvoda, robe, usluge i sl.","Troškovi zdravstvenog nadzora i kontrola proizvoda, robe, usluge i sl."
"4695","kp_rrif4695","l10n_hr_chart_template_rrif","expense",,"Troškovi obveznih liječničkih pregleda","Troškovi obveznih liječničkih pregleda"
"4696","kp_rrif4696","l10n_hr_chart_template_rrif","expense",,"Troškovi sistematskih kontrolnih liječničkih pregleda zaposlenika","Troškovi sistematskih kontrolnih liječničkih pregleda zaposlenika"
"4697","kp_rrif4697","l10n_hr_chart_template_rrif","expense",,"Troškovi osnivanja (bilježnik, sud, oglasi, odvjetnik)","Troškovi osnivanja (bilježnik, sud, oglasi, odvjetnik)"
"4698","kp_rrif4698","l10n_hr_chart_template_rrif","expense",,"Troškovi licenciranja, certifikata i sl.","Troškovi licenciranja, certifikata i sl."
"4699","kp_rrif4699","l10n_hr_chart_template_rrif","expense",,"Ostali nespomenuti nematerijalni troškovi (ulaznice za sajmove i dr.)","Ostali nespomenuti nematerijalni troškovi (ulaznice za sajmove i dr.)"
"4700","kp_rrif4700","l10n_hr_chart_template_rrif","expense",,"Ugovorene kamate","Ugovorene kamate"
"4701","kp_rrif4701","l10n_hr_chart_template_rrif","expense",,"Zatezne kamate (čl. 7., st. 1., t. 9. ZoPD)","Zatezne kamate (čl. 7., st. 1., t. 9. ZoPD)"
"4702","kp_rrif4702","l10n_hr_chart_template_rrif","expense",,"Kamate - porezno nepriznate","Kamate - porezno nepriznate"
"4703","kp_rrif4703","l10n_hr_chart_template_rrif","expense",,"Kamate koje se uračunavaju u zalihe (MRS 23. t.11. i HSFI t. 10.22.)","Kamate koje se uračunavaju u zalihe (MRS 23. t.11. i HSFI t. 10.22.)"
"4710","kp_rrif4710","l10n_hr_chart_template_rrif","expense",,"Tečajne razlike iz odnosa s povezanim društvima-a1","Tečajne razlike iz odnosa s povezanim društvima-Analitika 1"
"4720","kp_rrif4720","l10n_hr_chart_template_rrif","expense",,"Rashodi financiranja, troškovi popusta i naknadnih odobrenja s povezanim poduzetnicima","Rashodi financiranja, troškovi popusta i naknadnih odobrenja s povezanim poduzetnicima"
"4721","kp_rrif4721","l10n_hr_chart_template_rrif","expense",,"Troškovi usluga uprave koncerna i sl.","Troškovi usluga uprave koncerna i sl."
"4730","kp_rrif4730","l10n_hr_chart_template_rrif","expense",,"Kamate na kredite banaka","Kamate na kredite banaka"
"4731","kp_rrif4731","l10n_hr_chart_template_rrif","expense",,"Kamate iz lizing poslova","Kamate iz lizing poslova"
"4732","kp_rrif4732","l10n_hr_chart_template_rrif","expense",,"Kamate na zajmove pravnih osoba","Kamate na zajmove pravnih osoba"
"4733","kp_rrif4733","l10n_hr_chart_template_rrif","expense",,"Kamata na pozajmice članova društva i dioničara","Kamata na pozajmice članova društva i dioničara"
"4734","kp_rrif4734","l10n_hr_chart_template_rrif","expense",,"Kamate na zajmove od fizičkih osoba","Kamate na zajmove od fizičkih osoba"
"4735","kp_rrif4735","l10n_hr_chart_template_rrif","expense",,"Diskontne kamate po mjenicama i dr. vrijednosnim papirima","Diskontne kamate po mjenicama i dr. vrijednosnim papirima"
"4740","kp_rrif4740","l10n_hr_chart_template_rrif","expense",,"Zatezne kamate iz trgovačkih ugovora","Zatezne kamate iz trgovačkih ugovora"
"4741","kp_rrif4741","l10n_hr_chart_template_rrif","expense",,"Zatezne kamate na poreze, doprinose i dr. davanja","Zatezne kamate na poreze, doprinose i dr. davanja"
"4742","kp_rrif4742","l10n_hr_chart_template_rrif","expense",,"Zatezne kamate između povezanih osoba (por. neprizn.)","Zatezne kamate između povezanih osoba (por. neprizn.)"
"4743","kp_rrif4743","l10n_hr_chart_template_rrif","expense",,"Zatezne kamate po sudskim presudama","Zatezne kamate po sudskim presudama"
"4744","kp_rrif4744","l10n_hr_chart_template_rrif","expense",,"Ostale zatezne kamate","Ostale zatezne kamate"
"4750","kp_rrif4750","l10n_hr_chart_template_rrif","expense",,"Negativne tečajne razlike iz obveza za nabave u inozemstvu","Negativne tečajne razlike iz obveza za nabave u inozemstvu"
"4751","kp_rrif4751","l10n_hr_chart_template_rrif","expense",,"Negativne tečajne razlike iz kreditnih obveza","Negativne tečajne razlike iz kreditnih obveza"
"4752","kp_rrif4752","l10n_hr_chart_template_rrif","expense",,"Negativne tečajne razlike iz potraživanja u inozemstvu","Negativne tečajne razlike iz potraživanja u inozemstvu"
"4753","kp_rrif4753","l10n_hr_chart_template_rrif","expense",,"Negativne teč. razlike za ostalo (npr. iz blagajničkog, razlika kupovnog i srednjeg tečaja banke i dr.)","Negativne tečajne razlike za ostalo (npr. iz blagajničkog poslovanja, razlika između kupovnog i srednjeg tečaja banke i dr.)"
"4754","kp_rrif4754","l10n_hr_chart_template_rrif","expense",,"Negativne tečajne razlike nastale na stanjima deviznog računa i devizne blagajne","Negativne tečajne razlike nastale na stanjima deviznog računa i devizne blagajne"
"4760","kp_rrif4760","l10n_hr_chart_template_rrif","expense",,"Gubitci iz ulaganja u dionice, udjele i dr. vrij. papire (prodane ispod troška nabave - čl. 10. ZoPD-a1","Gubitci iz ulaganja u dionice, udjele, obveznice i dr. vrijednosne papire (koji su prodani ispod troška nabave - čl. 10. ZoPD-Analitika 1"
"4770","kp_rrif4770","l10n_hr_chart_template_rrif","expense",,"Troškovi diskonta pri prodaji potraživanja (faktoring)","Troškovi diskonta pri prodaji potraživanja (faktoring)"
"4771","kp_rrif4771","l10n_hr_chart_template_rrif","expense",,"Troškovi iz financijskih nagodbi","Troškovi iz financijskih nagodbi"
"4772","kp_rrif4772","l10n_hr_chart_template_rrif","expense",,"Ostali troškovi","Ostali troškovi"
"4780","kp_rrif4780","l10n_hr_chart_template_rrif","expense",,"Gubitci od smanjenja – vrijed. usklađenja fin. imovine za trgovanje (MRS 39. t. 55a. i HSFI t. 9.22b)","Gubitci od smanjenja - vrijednosnog usklađenja fin. imovine za trgovanje (MRS 39. t. 55a. i HSFI t. 9.22b)"
"4781","kp_rrif4781","l10n_hr_chart_template_rrif","expense",,"Troškovi smanjenja fin. imovine zbog ugovora o poteškoćama (HSFI t. 9.22a i MRS 39. t. 55b.)","Troškovi smanjenja fin. imovine zbog ugovora o poteškoćama (HSFI t. 9.22a i MRS 39. t. 55b.)"
"4782","kp_rrif4782","l10n_hr_chart_template_rrif","expense",,"Gubitci od smanjenja vrijednosti ostale financ. imov.","Gubitci od smanjenja vrijednosti ostale financ. imov."
"4790","kp_rrif4790","l10n_hr_chart_template_rrif","expense",,"Rashodi s osnove valutne klauzule po obvezama i kreditima","Rashodi s osnove valutne klauzule po obvezama i kreditima"
"4791","kp_rrif4791","l10n_hr_chart_template_rrif","expense",,"Rashodi s osnove usklađenja obveza zbog valutne i sl. klauzule (dobavljači, za predujmove i sl.)","Rashodi s osnove usklađenja obveza temeljem valutne i sl. klauzule (prema dobavljačima, za predujmove i sl.)"
"4792","kp_rrif4792","l10n_hr_chart_template_rrif","expense",,"Troškovi valutne klauzule iz tražbina ili obveza","Troškovi valutne klauzule iz tražbina ili obveza"
"4793","kp_rrif4793","l10n_hr_chart_template_rrif","expense",,"Troškovi burzovnih usluga, emisije vrijednosnih papira i sl.","Troškovi burzovnih usluga, emisije vrijednosnih papira i sl."
"4794","kp_rrif4794","l10n_hr_chart_template_rrif","expense",,"Troškovi carine inozemnog financijskog i operativnog lizinga","Troškovi carine inozemnog financijskog i operativnog lizinga"
"4799","kp_rrif4799","l10n_hr_chart_template_rrif","expense",,"Ostali nespomenuti financijski troškovi","Ostali nespomenuti financijski troškovi"
"4800","kp_rrif4800","l10n_hr_chart_template_rrif","expense",,"Naknadno odobreni popusti i odobrenja","Naknadno odobreni popusti i odobrenja"
"4801","kp_rrif4801","l10n_hr_chart_template_rrif","expense",,"Troškovi nagodbe (razlike iz sniženja)","Troškovi nagodbe (razlike iz sniženja)"
"4802","kp_rrif4802","l10n_hr_chart_template_rrif","expense",,"Troškovi nenadoknađenih jamstava za prodana dobra","Troškovi nenadoknađenih jamstava za prodana dobra"
"4803","kp_rrif4803","l10n_hr_chart_template_rrif","expense",,"Troškovi naknadnih reklamacija","Troškovi naknadnih reklamacija"
"4804","kp_rrif4804","l10n_hr_chart_template_rrif","expense",,"Troškovi uzoraka zbog kontrole i pregleda, izlaganje radi prodaje i sl.","Troškovi uzoraka zbog kontrole i pregleda, izlaganje radi prodaje i sl."
"4810","kp_rrif4810","l10n_hr_chart_template_rrif","expense",,"Otpisi nenaplaćenih jamstava i drugih osiguranja","Otpisi nenaplaćenih jamstava i drugih osiguranja"
"4811","kp_rrif4811","l10n_hr_chart_template_rrif","expense",,"Izravni otpisi nenaplaćenih potraživanja od kupaca i drugih koja nisu vrijednosno usklađena","Izravni otpisi nenaplaćenih potraživanja od kupaca i drugih koja nisu vrijednosno usklađena"
"4812","kp_rrif4812","l10n_hr_chart_template_rrif","expense",,"Troškovi ostalih otpisa","Troškovi ostalih otpisa"
"4820","kp_rrif4820","l10n_hr_chart_template_rrif","expense",,"Neamortizirana vrijednost rashodovane, uništene ili otuđene dugotrajne imovine","Neamortizirana vrijednost rashodovane, uništene ili otuđene dugotrajne imovine"
"4821","kp_rrif4821","l10n_hr_chart_template_rrif","expense",,"Otpisi imovine izvan uporabe - rashod","Otpisi imovine izvan uporabe - rashod"
"4822","kp_rrif4822","l10n_hr_chart_template_rrif","expense",,"Gubitak od prodane dug. imov. koja se ne amortizira","Gubitak od prodane dug. imov. koja se ne amortizira"
"4823","kp_rrif4823","l10n_hr_chart_template_rrif","expense",,"Gubitak od prodaje ost. dug. mat. i nemat. imovine","Gubitak od prodaje ost. dug. mat. i nemat. imovine"
"4824","kp_rrif4824","l10n_hr_chart_template_rrif","expense",,"Otpisi materijala i robe - rashod","Otpisi materijala i robe - rashod"
"4830","kp_rrif4830","l10n_hr_chart_template_rrif","expense",,"Manjkovi uslijed više sile (provalna krađa, elementarna nepogoda)","Manjkovi uslijed više sile (provalna krađa, elementarna nepogoda)"
"4831","kp_rrif4831","l10n_hr_chart_template_rrif","expense",,"Dopušteni manjkovi - kalo, rastep, kvar i lom na zalihama prema odlukama HGK, HOK ili internim aktima","Dopušteni manjkovi - kalo, rastep, kvar i lom na zalihama prema odlukama HGK, HOK ili internim aktima"
"4832","kp_rrif4832","l10n_hr_chart_template_rrif","expense",,"Prekomjerni manjkovi na zalihama (KRL) iznad normativa+PDV, prema HGK,(čl.7.,st.5.ZoPD)","Prekomjerni manjkovi na zalihama (kalo, rastep, kvar i lom) iznad normativa + PDV, prema odlukama HGK, HOK ili interno - skrivene isplate (čl. 7., st. 5. ZoPD)"
"4833","kp_rrif4833","l10n_hr_chart_template_rrif","expense",,"Manjkovi novca i vrijednosnih papira","Manjkovi novca i vrijednosnih papira"
"4839","kp_rrif4839","l10n_hr_chart_template_rrif","expense",,"Ostali manjkovi iz imovine","Ostali manjkovi iz imovine"
"4840","kp_rrif4840","l10n_hr_chart_template_rrif","expense",,"Troškovi kazni za prijestupe i prekršaje i sl. (čl. 7., st. 1., t. 7. ZoPD)","Troškovi kazni za prijestupe i prekršaje i sl. (čl. 7., st. 1., t. 7. ZoPD)"
"4841","kp_rrif4841","l10n_hr_chart_template_rrif","expense",,"Troškovi prisilne naplate poreza i dr. davanja (čl. 7., st. 1., t. 6. ZoPD)","Troškovi prisilne naplate poreza i dr. davanja (čl. 7., st. 1., t. 6. ZoPD)"
"4842","kp_rrif4842","l10n_hr_chart_template_rrif","expense",,"Penali, ležarine, dangubnine","Penali, ležarine, dangubnine"
"4843","kp_rrif4843","l10n_hr_chart_template_rrif","expense",,"Nadoknade šteta iz radnog odnosa (npr. za godišnji odmor, ozljede na radu, odštetne rente i sl.)","Nadoknade šteta iz radnog odnosa (npr. za neiskorišteni godišnji odmor, zbog ozljede na radu, odštetne rente i sl.)"
"4844","kp_rrif4844","l10n_hr_chart_template_rrif","expense",,"Nadoknade štete - troškovi po nagodbama i sudskim presudama - tužbama","Nadoknade štete - troškovi po nagodbama i sudskim presudama - tužbama"
"4845","kp_rrif4845","l10n_hr_chart_template_rrif","expense",,"Ugovorene kazne i penali zbog neizvršenja, propusta i sl.","Ugovorene kazne i penali zbog neizvršenja, propusta i sl."
"4846","kp_rrif4846","l10n_hr_chart_template_rrif","expense",,"Troškovi preuzetih obveza iz ugovora","Troškovi preuzetih obveza iz ugovora"
"4847","kp_rrif4847","l10n_hr_chart_template_rrif","expense",,"Kazne za parkiranje","Kazne za parkiranje"
"4849","kp_rrif4849","l10n_hr_chart_template_rrif","expense",,"Ostali izdatci za štete","Ostali izdatci za štete"
"4850","kp_rrif4850","l10n_hr_chart_template_rrif","expense",,"Naknadno utvrđeni troškovi - računi iz prethodnih godina","Naknadno utvrđeni troškovi - računi iz prethodnih godina"
"4851","kp_rrif4851","l10n_hr_chart_template_rrif","expense",,"Troškovi naknadnih razlika iz nabava","Troškovi naknadnih razlika iz nabava"
"4852","kp_rrif4852","l10n_hr_chart_template_rrif","expense",,"Ispravak pogrešaka prethodnih razdoblja","Ispravak pogrešaka prethodnih razdoblja"
"4860","kp_rrif4860","l10n_hr_chart_template_rrif","expense",,"Darovanje za općekorisne namjene (u novcu ili naravi do 2% od UP pr.god.  - čl. 7., st. 7. ZoPD)","Darovanje za općekorisne namjene (Darovi u novcu ili naravi do 2% od ukupnog prihoda prethodne godine za kulturu, znanost, odgoj i obrazovanje, zdravstvo, humanitarne, športske, vjerske, ekološke i dr. svrhe - čl. 7., st. 7. ZoPD)"
"4861","kp_rrif4861","l10n_hr_chart_template_rrif","expense",,"Darovanje za zdrav.potrebe (vanjskih osoba do 2% UPpr.god. a nije pokriveno osig.-čl.7.,st.8.ZoPD)","Darovanje za zdravstvene potrebe (darovanje vanjskih osoba do 2% od UP prethodne godine za operativne zahvate, liječenja, nabavu lijekova, ortopedskih pomagala i dr. što nije pokriveno osiguranjem - čl. 7., st. 8. ZoPD)"
"4870","kp_rrif4870","l10n_hr_chart_template_rrif","expense",,"Porezno nepriznata darovanja iznad 2% UP (iznad dop. s 486) i ino. udruga i sl. (čl.7.st.1.t.10 ZoPD)","Porezno nepriznata darovanja iznad 2% UP (za svrhe iznad dopuštenih s računa 486) i darovanja inozemnih udruga, ustanova i sl. (čl. 7. st. 1. t. 10. ZoPD)"
"4871","kp_rrif4871","l10n_hr_chart_template_rrif","expense",,"Darovi bez protučinidbe prim. i izdatci nisu u svezi s ostv.dobitka+PDV,osim na novac-čl7,st1.,t13ZoPD","Darovi (milodari), potpore i dotacije bez protučinidbe primatelja, te dr. izdatci koji nisu u svezi s ostvarivanjem dobitka (+PDV, osim na novac - čl. 7., st. 1., t. 13. ZoPD)"
"4872","kp_rrif4872","l10n_hr_chart_template_rrif","expense",,"Darovanje političkih stranaka i nezavisnih kandidata","Darovanje političkih stranaka i nezavisnih kandidata"
"4880","kp_rrif4880","l10n_hr_chart_template_rrif","expense",,"Troškovi iz ortačkog ugovora","Troškovi iz ortačkog ugovora"
"4881","kp_rrif4881","l10n_hr_chart_template_rrif","expense",,"Troškovi iz posredovanja","Troškovi iz posredovanja"
"4890","kp_rrif4890","l10n_hr_chart_template_rrif","expense",,"Rashodi utvrđeni u postupku nadzora (čl. 7. st. 1. t. 11. ZoPD) - skrivene isplate dobitka","Rashodi utvrđeni u postupku nadzora (čl. 7. st. 1. t. 11. ZoPD) - skrivene isplate dobitka - izuzimanje dioničara članova društva i fizičkih osoba (obrtnika) i s njima povez. osobama"
"4891","kp_rrif4891","l10n_hr_chart_template_rrif","expense",,"Troškovi -porezno nepriznati- koji nisu u svezi s ostvarivanjem dobitka(čl7, st1,t13.ZoPD-red.br.24PD) ","Troškovi koji nisu izravno u svezi s ostvarivanjem dobitka (čl. 7., st. 1., t. 13. ZoPD - red. br. 24 PD, npr. amortiz. imov. koja ne služi obavljanju djelatnosti, dodjela vlastitih dionica - udjela i dr.) - porezno nepriznati"
"4892","kp_rrif4892","l10n_hr_chart_template_rrif","expense",,"Troškovi povalstica i dr. oblici imovinskih koristi (čl. 7., st.1. t.10. ZoPD) - porezno nepriznati","Troškovi povalstica i dr. oblici imovinskih koristi (čl. 7., st.1. t.10. ZoPD) - porezno nepriznati"
"4899","kp_rrif4899","l10n_hr_chart_template_rrif","expense",,"Ostali nespomenuti poslovni rashodi","Ostali nespomenuti poslovni rashodi"
"4900","kp_rrif4900","l10n_hr_chart_template_rrif","expense",,"Raspored troškova za obračun proiz. i usluga (HSFI10, MRS2, MRS11)-uskladištivi troškovi (na 60,62i63)-a1","Raspored troškova za obračun proizvoda i usluga (prema HSFI 10 i MRS-u 2 i MRS-u 11) - uskladištivi troškovi (na račune 60, 62 i 63)-Analitika 1"
"4910","kp_rrif4910","l10n_hr_chart_template_rrif","expense",,"Raspored troškova za pokriće upravnih, administrativnih, prodajnih i drugih troškova (na rn 70 i 71)-a1","Raspored troškova za pokriće upravnih, administrativnih, prodajnih i drugih troškova (na račune 70 i 71)-Analitika 1"
"6000","kp_rrif6000","l10n_hr_chart_template_rrif","asset_current",,"Proizvodnja u tijeku (po serijama, nositeljima, mjestima, radnim nalozima i sl.)-a1","Proizvodnja u tijeku (razrada po serijama, nositeljima troškova, mjestima, pogonima, gradilištima, objektima, radnim nalozima i sl.)-Analitika 1"
"6010","kp_rrif6010","l10n_hr_chart_template_rrif","asset_current",,"Vrijednost usluga (u tijeku ili nedovršenih na datum bilance - MRS 2, t. 16.)-a1","Vrijednost usluga (u tijeku ili nedovršenih na datum bilance - MRS 2, t. 16.)-Analitika 1"
"6020","kp_rrif6020","l10n_hr_chart_template_rrif","asset_current",,"Vanjska proizvodnja (kooperacija i dr.)-a1","Vanjska proizvodnja (kooperacija i dr.)-Analitika 1"
"6050","kp_rrif6050","l10n_hr_chart_template_rrif","asset_current",,"Proizvodnja u slobodnoj zoni-a1","Proizvodnja u slobodnoj zoni-Analitika 1"
"6060","kp_rrif6060","l10n_hr_chart_template_rrif","asset_current",,"Proizvodnja u doradi i manipulaciji-a1","Proizvodnja u doradi i manipulaciji-Analitika 1"
"6070","kp_rrif6070","l10n_hr_chart_template_rrif","asset_current",,"Obustavljena proizvodnja-a1","Obustavljena proizvodnja-Analitika 1"
"6080","kp_rrif6080","l10n_hr_chart_template_rrif","asset_current",,"Proizvodnja u tijeku iz ortačkog ugovora-a1","Proizvodnja u tijeku iz ortačkog ugovora-Analitika 1"
"6090","kp_rrif6090","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađivanje proizvodnje - usluga-a1","Vrijednosno usklađivanje proizvodnje - usluga-Analitika 1"
"6100","kp_rrif6100","l10n_hr_chart_template_rrif","asset_current",,"Zalihe poluproizvoda (analitika prema osnovnim skupinama ili po stupnju dovršenosti)-a1","Zalihe poluproizvoda (analitika prema osnovnim skupinama ili po stupnju dovršenosti)-Analitika 1"
"6110","kp_rrif6110","l10n_hr_chart_template_rrif","asset_current",,"Nedovršeni proizvodi i poluproizvodi (analitika po vrstama proizvoda)-a1","Nedovršeni proizvodi i poluproizvodi (analitika po vrstama proizvoda)-Analitika 1"
"6190","kp_rrif6190","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađivanje nedovršenih proizvoda i poluproizvoda-a1","Vrijednosno usklađivanje nedovršenih proizvoda i poluproizvoda-Analitika 1"
"6200","kp_rrif6200","l10n_hr_chart_template_rrif","asset_current",,"Zalihe biološke proizvodnje u toku-a1","Zalihe biološke proizvodnje u toku-Analitika 1"
"6210","kp_rrif6210","l10n_hr_chart_template_rrif","asset_current",,"Biloška imovina za prodaju-a1","Biloška imovina za prodaju-Analitika 1"
"6290","kp_rrif6290","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađivanje biloške imovine-a1","Vrijednosno usklađivanje biloške imovine-Analitika 1"
"6300","kp_rrif6300","l10n_hr_chart_template_rrif","asset_current",,"Gotovi proizvodi na skladištu (razrada za svako skladište, pa po skupinama, tipovima, vrstama i sl.)-a1","Gotovi proizvodi na skladištu (razrada za svako skladište a unutar toga po skupinama, tipovima, vrstama i sl.)-Analitika 1"
"6310","kp_rrif6310","l10n_hr_chart_template_rrif","asset_current",,"Gotovi proizvodi u javnom skladištu, silosu, i dr.-a1","Gotovi proizvodi u javnom skladištu, silosu, i dr.-Analitika 1"
"6320","kp_rrif6320","l10n_hr_chart_template_rrif","asset_current",,"Gotovi proizvodi dani u komisijsku prodaju-a1","Gotovi proizvodi dani u komisijsku prodaju-Analitika 1"
"6330","kp_rrif6330","l10n_hr_chart_template_rrif","asset_current",,"Gotovi proizvodi dani u konsignacijsku prodaju-a1","Gotovi proizvodi dani u konsignacijsku prodaju-Analitika 1"
"6340","kp_rrif6340","l10n_hr_chart_template_rrif","asset_current",,"Gotovi proizvodi u doradi, obradi i manipulaciji-a1","Gotovi proizvodi u doradi, obradi i manipulaciji-Analitika 1"
"6350","kp_rrif6350","l10n_hr_chart_template_rrif","asset_current",,"Gotovi proizvodi u slobodnoj zoni-a1","Gotovi proizvodi u slobodnoj zoni-Analitika 1"
"6360","kp_rrif6360","l10n_hr_chart_template_rrif","asset_current",,"Zalihe nekurentnih proizvoda i otpadaka-a1","Zalihe nekurentnih proizvoda i otpadaka-Analitika 1"
"6370","kp_rrif6370","l10n_hr_chart_template_rrif","asset_current",,"Gotovi proizvodi u izložbenim prostorima-a1","Gotovi proizvodi u izložbenim prostorima-Analitika 1"
"6380","kp_rrif6380","l10n_hr_chart_template_rrif","asset_current",,"Gotovi proizvodi iz ortaštava-a1","Gotovi proizvodi iz ortaštava-Analitika 1"
"6390","kp_rrif6390","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađivanje zaliha gotovih proizvoda-a1","Vrijednosno usklađivanje zaliha gotovih proizvoda-Analitika 1"
"6400","kp_rrif6400","l10n_hr_chart_template_rrif","asset_current",,"Gotovi proizvodi u prodaji u vlastitim prodavaonicama (analitika po prodavaonicama)-a1","Gotovi proizvodi u prodaji u vlastitim prodavaonicama (analitika po prodavaonicama)-Analitika 1"
"6410","kp_rrif6410","l10n_hr_chart_template_rrif","asset_current",,"Uračunani PDV u vrijednosti proizvoda-a1","Uračunani PDV u vrijednosti proizvoda-Analitika 1"
"6420","kp_rrif6420","l10n_hr_chart_template_rrif","asset_current",,"Uračunani porez na luksuz-a1","Uračunani porez na luksuz-Analitika 1"
"6480","kp_rrif6480","l10n_hr_chart_template_rrif","asset_current",,"Uračunana marža u prodajnoj cijeni gotovih proizvoda-a1","Uračunana marža u prodajnoj cijeni gotovih proizvoda-Analitika 1"
"6490","kp_rrif6490","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađivanje gotovih proizvoda u prodavaonicama-a1","Vrijednosno usklađivanje gotovih proizvoda u prodavaonicama-Analitika 1"
"6500","kp_rrif6500","l10n_hr_chart_template_rrif","asset_current",,"Kupovna cijena robe od dobavljača-a1","Kupovna cijena robe od dobavljača-Analitika 1"
"6510","kp_rrif6510","l10n_hr_chart_template_rrif","asset_current",,"Troškovi transporta","Troškovi transporta"
"6511","kp_rrif6511","l10n_hr_chart_template_rrif","asset_current",,"Troškovi ukrcaja i iskrcaja (fakturirani)","Troškovi ukrcaja i iskrcaja (fakturirani)"
"6512","kp_rrif6512","l10n_hr_chart_template_rrif","asset_current",,"Transportno osiguranje","Transportno osiguranje"
"6513","kp_rrif6513","l10n_hr_chart_template_rrif","asset_current",,"Troškovi posebnog pakiranja - ambalaže","Troškovi posebnog pakiranja - ambalaže"
"6514","kp_rrif6514","l10n_hr_chart_template_rrif","asset_current",,"Troškovi vlastitog transporta (ne više od tarife javnog prijevoza) dovođenja robe na prodajnu lokaciju","Troškovi vlastitog transporta (ne više od tarife javnog prijevoza) dovođenja robe na prodajnu lokaciju"
"6515","kp_rrif6515","l10n_hr_chart_template_rrif","asset_current",,"Troškovi oblikovanja za posebne kupce","Troškovi oblikovanja za posebne kupce"
"6516","kp_rrif6516","l10n_hr_chart_template_rrif","asset_current",,"Troškovi čuvanja i rukovanja (u fazi nabave)","Troškovi čuvanja i rukovanja (u fazi nabave)"
"6517","kp_rrif6517","l10n_hr_chart_template_rrif","asset_current",,"Špediterski i bankarski troškovi","Špediterski i bankarski troškovi"
"6518","kp_rrif6518","l10n_hr_chart_template_rrif","asset_current",,"Nadoknada uvozniku za uslugu uvoza","Nadoknada uvozniku za uslugu uvoza"
"6519","kp_rrif6519","l10n_hr_chart_template_rrif","asset_current",,"Ostali troškovi kupnje (pregledi, atesti i troškovi u svezi s dovođenjem robe na zalihu -HSFI10 i MRS2)","Ostali troškovi kupnje (pregledi, atesti i dr. troškovi u svezi s dovođenjem robe na zalihu - HSFI 10 i MRS 2)"
"6520","kp_rrif6520","l10n_hr_chart_template_rrif","asset_current",,"Carina i druge uvozne pristojbe za robu-a1","Carina i druge uvozne pristojbe za robu-Analitika 1"
"6530","kp_rrif6530","l10n_hr_chart_template_rrif","asset_current",,"Posebni porezi (trošarine)-a1","Posebni porezi (trošarine)-Analitika 1"
"6590","kp_rrif6590","l10n_hr_chart_template_rrif","asset_current",,"Obračun nabave - trošak kupnje-a1","Obračun nabave - trošak kupnje-Analitika 1"
"6600","kp_rrif6600","l10n_hr_chart_template_rrif","asset_current",,"Roba u vlastitom veleprodajnom skladištu (analitika po skladištima)","Roba u vlastitom veleprodajnom skladištu (analitika po skladištima)"
"6601","kp_rrif6601","l10n_hr_chart_template_rrif","asset_current",,"Zaliha otpadaka od robe","Zaliha otpadaka od robe"
"6610","kp_rrif6610","l10n_hr_chart_template_rrif","asset_current",,"Roba u tuđem skladištu","Roba u tuđem skladištu"
"6611","kp_rrif6611","l10n_hr_chart_template_rrif","asset_current",,"Roba u tuđim silosima, hladnjačama i sl.","Roba u tuđim silosima, hladnjačama i sl."
"6612","kp_rrif6612","l10n_hr_chart_template_rrif","asset_current",,"Roba u izložbenim prostorima","Roba u izložbenim prostorima"
"6620","kp_rrif6620","l10n_hr_chart_template_rrif","asset_current",,"Roba dana u komisijsku ili konsignacijsku prodaju-a1","Roba dana u komisijsku ili konsignacijsku prodaju-Analitika 1"
"66300","kp_rrif66300","l10n_hr_chart_template_rrif","asset_current",,"Roba u prodavaonici A s PDV-om 23%","Roba u prodavaonici A s PDV-om 23%"
"66301","kp_rrif66301","l10n_hr_chart_template_rrif","asset_current",,"Roba u prodavaonici A s PDV-om 10%","Roba u prodavaonici A s PDV-om 10%"
"66302","kp_rrif66302","l10n_hr_chart_template_rrif","asset_current",,"Roba u prodavaonici A s PDV-om 0%, itd.","Roba u prodavaonici A s PDV-om 0%, itd."
"6640","kp_rrif6640","l10n_hr_chart_template_rrif","asset_current",,"Uračunani PDV (analitika po prodajnim mjestima i poreznim stopama)","Uračunani PDV (analitika po prodajnim mjestima i poreznim stopama)"
"6641","kp_rrif6641","l10n_hr_chart_template_rrif","asset_current",,"Uračunani porez na luksuz","Uračunani porez na luksuz"
"6650","kp_rrif6650","l10n_hr_chart_template_rrif","asset_current",,"Vlastita roba u carinskom skladištu tipa ""D""","Vlastita roba u carinskom skladištu tipa ""D"""
"6651","kp_rrif6651","l10n_hr_chart_template_rrif","asset_current",,"Roba u slobodnoj zoni","Roba u slobodnoj zoni"
"6660","kp_rrif6660","l10n_hr_chart_template_rrif","asset_current",,"Vrijednost robe u doradi","Vrijednost robe u doradi"
"6661","kp_rrif6661","l10n_hr_chart_template_rrif","asset_current",,"Troškovi u svezi s doradom","Troškovi u svezi s doradom"
"6670","kp_rrif6670","l10n_hr_chart_template_rrif","asset_current",,"Roba na putu-a1","Roba na putu-Analitika 1"
"6680","kp_rrif6680","l10n_hr_chart_template_rrif","asset_current",,"Razlika u cijeni robe na skladištu (analitički po skupinama robe s istom maržom)","Razlika u cijeni robe na skladištu (analitički po skupinama robe s istom maržom)"
"6681","kp_rrif6681","l10n_hr_chart_template_rrif","asset_current",,"Uračunana marža robe u prodavaonici","Uračunana marža robe u prodavaonici"
"6690","kp_rrif6690","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađenje zbog pada cijena, smanjenja uporabljivosti i sl.","Vrijednosno usklađenje zbog pada cijena, smanjenja uporabljivosti i sl."
"6691","kp_rrif6691","l10n_hr_chart_template_rrif","asset_current",,"Prepravljene cijene robe zbog monetarnih oscilacija","Prepravljene cijene robe zbog monetarnih oscilacija"
"6700","kp_rrif6700","l10n_hr_chart_template_rrif","asset_current",,"Dani predujmovi za nabavu robe-a1","Dani predujmovi za nabavu robe-Analitika 1"
"6710","kp_rrif6710","l10n_hr_chart_template_rrif","asset_current",,"Dani predujmovi uvozniku za nabavu robe-a1","Dani predujmovi uvozniku za nabavu robe-Analitika 1"
"6720","kp_rrif6720","l10n_hr_chart_template_rrif","asset_current",,"Dani predujmovi za robu povezanom društvu-a1","Dani predujmovi za robu povezanom društvu-Analitika 1"
"6790","kp_rrif6790","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađenje danih predujmova za robu-a1","Vrijednosno usklađenje danih predujmova za robu-Analitika 1"
"6800","kp_rrif6800","l10n_hr_chart_template_rrif","asset_current",,"Nabavna vrijednost nekretnina za preprodaju (s porezom na promet)-a1","Nabavna vrijednost nekretnina za preprodaju (s porezom na promet)-Analitika 1"
"6810","kp_rrif6810","l10n_hr_chart_template_rrif","asset_current",,"Troškovi dodatnog uređenja - dorade-a1","Troškovi dodatnog uređenja - dorade-Analitika 1"
"6820","kp_rrif6820","l10n_hr_chart_template_rrif","asset_current",,"Umjetnine u prodaji-a1","Umjetnine u prodaji-Analitika 1"
"6830","kp_rrif6830","l10n_hr_chart_template_rrif","asset_current",,"Nekretnine za prodaju (uređene)-a1","Nekretnine za prodaju (uređene)-Analitika 1"
"6840","kp_rrif6840","l10n_hr_chart_template_rrif","asset_current",,"Uračunani PDV u umjetnine-a1","Uračunani PDV u umjetnine-Analitika 1"
"6870","kp_rrif6870","l10n_hr_chart_template_rrif","asset_current",,"Predujmovi za kupnju nekretnina radi daljnje prodaje-a1","Predujmovi za kupnju nekretnina radi daljnje prodaje-Analitika 1"
"6880","kp_rrif6880","l10n_hr_chart_template_rrif","asset_current",,"Uračunana razlika u cijeni nekretnina i umjetnina za daljnju prodaju-a1","Uračunana razlika u cijeni nekretnina i umjetnina za daljnju prodaju-Analitika 1"
"6890","kp_rrif6890","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađivanje nekretnina i umjetnina u prometu i predujmova-a1","Vrijednosno usklađivanje nekretnina i umjetnina u prometu i predujmova-Analitika 1"
"6900","kp_rrif6900","l10n_hr_chart_template_rrif","asset_current",,"Nematerijalna imovina namijenjena prodaji-a1","Nematerijalna imovina namijenjena prodaji-Analitika 1"
"6910","kp_rrif6910","l10n_hr_chart_template_rrif","asset_current",,"Skupina imovine za prodaju (npr. pogon, poslov. jedinica i sl. )","Skupina imovine za prodaju (npr. pogon, poslov. jedinica i sl. )"
"6990","kp_rrif6990","l10n_hr_chart_template_rrif","asset_current",,"Vrijednosno usklađenje dugotrajne imovine namijenjena prodaji-a1","Vrijednosno usklađenje dugotrajne imovine namijenjena prodaji-Analitika 1"
"7000","kp_rrif7000","l10n_hr_chart_template_rrif","expense",,"Trošak zaliha prodanih proizvoda (60, 62, 63 i 64)-a1","Trošak zaliha prodanih proizvoda (60, 62, 63 i 64)-Analitika 1"
"7010","kp_rrif7010","l10n_hr_chart_template_rrif","expense",,"Troškovi realiziranih usluga (490 i 601)-a1","Troškovi realiziranih usluga (490 i 601)-Analitika 1"
"7020","kp_rrif7020","l10n_hr_chart_template_rrif","expense",,"Troškovi neiskorištenog kapaciteta (HSFI t. 10.18. i MRS 2 t. 13)-a1","Troškovi neiskorištenog kapaciteta (HSFI t. 10.18. i MRS 2 t. 13)-Analitika 1"
"7030","kp_rrif7030","l10n_hr_chart_template_rrif","expense",,"Troškovi nabavne vrijednosti materijala, dijelova, inventara i otpadaka","Troškovi nabavne vrijednosti materijala, dijelova, inventara i otpadaka"
"7031","kp_rrif7031","l10n_hr_chart_template_rrif","expense",,"Troškovi manjkova materijala, dijelova i inventara kojom se tereti odgovorna osoba","Troškovi manjkova materijala, dijelova i inventara kojom se tereti odgovorna osoba"
"7040","kp_rrif7040","l10n_hr_chart_template_rrif","expense",,"Dopušteni manjkovi -porezno priznati- tehnološki KRL i škart u proizvodnji (sa skupina 60, 62, 63 i 64)","Dopušteni manjkovi - tehnološki kalo, rastep, kvar i lom i škart u proizvodnji (sa skupina 60, 62, 63 i 64) - porezno priznati"
"7041","kp_rrif7041","l10n_hr_chart_template_rrif","expense",,"Prekomjerni manjkovi-porezno priznati teh.KRL i škart u proiz.(60,62,63i64 -HSFI t10.21. i MRS2,t14)","Prekomjerni manjkovi - tehnološki kalo, rastep, kvar, lom i škart u proizvodnji (60, 62, 63 i 64 - HSFI t. 10.21. i MRS 2, t. 14.) a porezno dopušteni po očevidu i sl. - porezno priznati"
"7042","kp_rrif7042","l10n_hr_chart_template_rrif","expense",,"Prekomjerni manjkovi proizvoda (HSFI t. 10.2. i MRS 2, t. 16.) - porezno nepriznati","Prekomjerni manjkovi proizvoda (HSFI t. 10.2. i MRS 2, t. 16.) - porezno nepriznati"
"7043","kp_rrif7043","l10n_hr_chart_template_rrif","expense",,"Razlika višeg troška proizvodnje od neto-vrij. koja se može realizirati (HSFIt.10.35. i MRS2t.28.-33.)","Razlika višeg troška proizvodnje od neto-vrijednosti koja se može realizirati (HSFI t. 10.35. i MRS 2, t. 28. do 33.)"
"7044","kp_rrif7044","l10n_hr_chart_template_rrif","expense",,"Troškovi isporučenih proizvoda u jamstvenom roku (zamjena)","Troškovi isporučenih proizvoda u jamstvenom roku (zamjena)"
"7050","kp_rrif7050","l10n_hr_chart_template_rrif","expense",,"Greškom neiskazani rashodi proteklih razdoblja-a1","Greškom neiskazani rashodi proteklih razdoblja-Analitika 1"
"7060","kp_rrif7060","l10n_hr_chart_template_rrif","expense",,"Gubitci iz ugovora o izgradnj (MRS 11, t. 36.)-a1","Gubitci iz ugovora o izgradnj (MRS 11, t. 36.)-Analitika 1"
"7070","kp_rrif7070","l10n_hr_chart_template_rrif","expense",,"Troškovi iz ugovora o ortaštvu-a1","Troškovi iz ugovora o ortaštvu-Analitika 1"
"7080","kp_rrif7080","l10n_hr_chart_template_rrif","expense",,"Troškovi vrijed. uskl. proizvod. u tijeku(609), poluproizvoda(629) i zaliha got. proizvoda (639 i 649)-a1","Troškovi vrijednosnog usklađenja proizvodnje u tijeku (609), poluproizvoda (629) i zaliha gotovih proizvoda (639 i 649)-Analitika 1"
"7100","kp_rrif7100","l10n_hr_chart_template_rrif","expense",,"Trošak prodane robe u tuzemstvu","Trošak prodane robe u tuzemstvu"
"7101","kp_rrif7101","l10n_hr_chart_template_rrif","expense",,"Trošak prodane robe u inozemstvu","Trošak prodane robe u inozemstvu"
"7102","kp_rrif7102","l10n_hr_chart_template_rrif","expense",,"Troškovi prodane robe u tranzitu","Troškovi prodane robe u tranzitu"
"7103","kp_rrif7103","l10n_hr_chart_template_rrif","expense",,"Trošak prodane robe u poslov. jed.","Trošak prodane robe u poslov. jed."
"7110","kp_rrif7110","l10n_hr_chart_template_rrif","expense",,"Nabavna vrijednost prodanih nekretnina i umjetnina-a1","Nabavna vrijednost prodanih nekretnina i umjetnina-Analitika 1"
"7120","kp_rrif7120","l10n_hr_chart_template_rrif","expense",,"Troškovi dugotr. imov. namijenjeni prodaji-a1","Troškovi dugotr. imov. namijenjeni prodaji-Analitika 1"
"7130","kp_rrif7130","l10n_hr_chart_template_rrif","expense",,"Kalo, rastep, kvar i lom u dopuštenoj visini prema Pravilniku HGK - porezno priznati","Kalo, rastep, kvar i lom u dopuštenoj visini prema Pravilniku HGK - porezno priznati"
"7131","kp_rrif7131","l10n_hr_chart_template_rrif","expense",,"Manjkovi uslijed više sile (provalne krađe, poplava, požar, potres i sl.) - porezno priznati","Manjkovi uslijed više sile (provalne krađe, poplava, požar, potres i sl.) - porezno priznati"
"7132","kp_rrif7132","l10n_hr_chart_template_rrif","expense",,"Manjkovi i otpisi trgovačke robe po očevidu PU i sl. - porezno priznati","Manjkovi i otpisi trgovačke robe po očevidu PU i sl. - porezno priznati"
"7133","kp_rrif7133","l10n_hr_chart_template_rrif","expense",,"Prekomjerni kalo, rastep, kvar i lom + PDV - porezno nepriznati","Prekomjerni kalo, rastep, kvar i lom + PDV - porezno nepriznati"
"7134","kp_rrif7134","l10n_hr_chart_template_rrif","expense",,"Manjak robe na teret odgovorne osobe","Manjak robe na teret odgovorne osobe"
"7140","kp_rrif7140","l10n_hr_chart_template_rrif","expense",,"Troškovi zamjene robe u jamstvenom roku-a1","Troškovi zamjene robe u jamstvenom roku-Analitika 1"
"7150","kp_rrif7150","l10n_hr_chart_template_rrif","expense",,"Greškom neiskazani rashodi prodane robe u proteklim razdobljima u trgovini-a1","Greškom neiskazani rashodi prodane robe u proteklim razdobljima u trgovini-Analitika 1"
"7180","kp_rrif7180","l10n_hr_chart_template_rrif","expense",,"Troškovi vrijednosnog usklađenja trgovačke robe i predujmova (669, 679, 689)-a1","Troškovi vrijednosnog usklađenja trgovačke robe i predujmova (669, 679, 689)-Analitika 1"
"7190","kp_rrif7190","l10n_hr_chart_template_rrif","expense",,"Troškovi vrijednosnog usklađenja dugotrajne imovine namijenjene prodaji (699)-a1","Troškovi vrijednosnog usklađenja dugotrajne imovine namijenjene prodaji (699)-Analitika 1"
"7200","kp_rrif7200","l10n_hr_chart_template_rrif","expense",,"Troškovi uprave, prodaje, administracije (491)-a1","Troškovi uprave, prodaje, administracije (491)-Analitika 1"
"7210","kp_rrif7210","l10n_hr_chart_template_rrif","expense",,"Ostali poslovni rashodi - nespomenuti-a1","Ostali poslovni rashodi - nespomenuti-Analitika 1"
"7300","kp_rrif7300","l10n_hr_chart_template_rrif","expense",,"Izvanredni rashodi od prodaje dugotrajne imovine (HSFI t. 8.35.)-a1","Izvanredni rashodi od prodaje dugotrajne imovine (HSFI t. 8.35.)-Analitika 1"
"7310","kp_rrif7310","l10n_hr_chart_template_rrif","expense",,"Izvanredni otpisi od otuđenja imovine, nastali neočekivano i u visokoj vrijednosti (HSFt4.7. i MRS10t9)-a1","Izvanredni otpisi od otuđenja imovine koji su nastali neočekivano i u visokoj vrijednosti (HSFI t. 4.7. i MRS 10. t. 9.)-Analitika 1"
"7320","kp_rrif7320","l10n_hr_chart_template_rrif","expense",,"Gubitci zbog izvlaštenja ili zbog prirodnih katastrofa na važnom dijelu imovine-a1","Gubitci zbog izvlaštenja ili zbog prirodnih katastrofa na važnom dijelu imovine-Analitika 1"
"7330","kp_rrif7330","l10n_hr_chart_template_rrif","expense",,"Izvanredni rashodi iz ostalih rijetkih i neobičnih događaja ili transakcija-a1","Izvanredni rashodi iz ostalih rijetkih i neobičnih događaja ili transakcija-Analitika 1"
"7340","kp_rrif7340","l10n_hr_chart_template_rrif","expense",,"Izvanredne kazne, penali, odštete, naknadno utvrđ. obveze i sl.-a1","Izvanredne kazne, penali, odštete, naknadno utvrđ. obveze i sl.-Analitika 1"
"7350","kp_rrif7350","l10n_hr_chart_template_rrif","expense",,"Nerealizirani gubitci-a1","Nerealizirani gubitci-Analitika 1"
"7370","kp_rrif7370","l10n_hr_chart_template_rrif","expense",,"Gubitci od procjene biološke imovine-a1","Gubitci od procjene biološke imovine-Analitika 1"
"7400","kp_rrif7400","l10n_hr_chart_template_rrif","expense",,"Udio u gubitku povezanih društava","Udio u gubitku povezanih društava"
"7401","kp_rrif7401","l10n_hr_chart_template_rrif","expense",,"Udio u gubitku ortaka","Udio u gubitku ortaka"
"7450","kp_rrif7450","l10n_hr_chart_template_rrif","income",,"Udio u dobitku povezanih društava","Udio u dobitku povezanih društava"
"7451","kp_rrif7451","l10n_hr_chart_template_rrif","income",,"Udio u dobitku ortaštva","Udio u dobitku ortaštva"
"7500","kp_rrif7500","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje proizvoda od redovne prodaje","Prihodi od prodaje proizvoda od redovne prodaje"
"7501","kp_rrif7501","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje proizvoda na kredit ili otplatu","Prihodi od prodaje proizvoda na kredit ili otplatu"
"7502","kp_rrif7502","l10n_hr_chart_template_rrif","income",,"Prihodi ostvareni u slobodnoj zoni","Prihodi ostvareni u slobodnoj zoni"
"7503","kp_rrif7503","l10n_hr_chart_template_rrif","income",,"Prodaja u vlastitim prodavaonicama","Prodaja u vlastitim prodavaonicama"
"7504","kp_rrif7504","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje stanova i dr. građevina","Prihodi od prodaje stanova i dr. građevina"
"7505","kp_rrif7505","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje poluproizvoda i nedovršenih proizvoda","Prihodi od prodaje poluproizvoda i nedovršenih proizvoda"
"7506","kp_rrif7506","l10n_hr_chart_template_rrif","income",,"Prihodi ostvareni u posl. jed. na području posebne držav. skrbi i u Vukovaru","Prihodi ostvareni u posl. jed. na području posebne držav. skrbi i u Vukovaru"
"7507","kp_rrif7507","l10n_hr_chart_template_rrif","income",,"Prihod od povratne naknade za ambalažu (bez PDV-a)","Prihod od povratne naknade za ambalažu (bez PDV-a)"
"7508","kp_rrif7508","l10n_hr_chart_template_rrif","income",,"Prihod od prodaje otpadaka iz proizvodnje","Prihod od prodaje otpadaka iz proizvodnje"
"7509","kp_rrif7509","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje poluproizvoda i nedovršenih proizvoda","Prihodi od prodaje poluproizvoda i nedovršenih proizvoda"
"7510","kp_rrif7510","l10n_hr_chart_template_rrif","income",,"Prihodi od servisnih usluga, usluga popravaka i sl. usluga","Prihodi od servisnih usluga, usluga popravaka i sl. usluga"
"7511","kp_rrif7511","l10n_hr_chart_template_rrif","income",,"Prihodi od restorana i gostionica","Prihodi od restorana i gostionica"
"7512","kp_rrif7512","l10n_hr_chart_template_rrif","income",,"Prihodi od hotela i noćenja","Prihodi od hotela i noćenja"
"7513","kp_rrif7513","l10n_hr_chart_template_rrif","income",,"Prihodi od knjigovodstvenih, usluga poreznog savjetovanja, revizorskih, konzultantskih i dr. usluga","Prihodi od knjigovodstvenih, usluga poreznog savjetovanja, revizorskih, konzultantskih i dr. usluga"
"7514","kp_rrif7514","l10n_hr_chart_template_rrif","income",,"Prihodi od usluga prijevoza","Prihodi od usluga prijevoza"
"7515","kp_rrif7515","l10n_hr_chart_template_rrif","income",,"Prihodi od komunalnih usluga","Prihodi od komunalnih usluga"
"7516","kp_rrif7516","l10n_hr_chart_template_rrif","income",,"Prihodi od promidžbenih usluga","Prihodi od promidžbenih usluga"
"7517","kp_rrif7517","l10n_hr_chart_template_rrif","income",,"Prihodi od usluga zaštite i istraživanja","Prihodi od usluga zaštite i istraživanja"
"7518","kp_rrif7518","l10n_hr_chart_template_rrif","income",,"Prihodi od programskih usluga","Prihodi od programskih usluga"
"7519","kp_rrif7519","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje ostalih usluga","Prihodi od prodaje ostalih usluga"
"7520","kp_rrif7520","l10n_hr_chart_template_rrif","income",,"Prihodi od graditeljskih usluga - iz ugovora o izgradnji (građevina, postrojenja, brodova i sl.)-a1","Prihodi od graditeljskih usluga - iz ugovora o izgradnji (građevina, postrojenja, brodova i sl.)-Analitika 1"
"7530","kp_rrif7530","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje dobara u inozemstvo (moguća analitika po vrstama proizvoda i zemljama)-a1","Prihodi od prodaje dobara u inozemstvo (moguća analitika po vrstama proizvoda i zemljama)-Analitika 1"
"7540","kp_rrif7540","l10n_hr_chart_template_rrif","income",,"Prihodi iz međunarodne plovidbe brodovima (čl. 26., st. 10. ZoPD)","Prihodi iz međunarodne plovidbe brodovima (čl. 26., st. 10. ZoPD)"
"7541","kp_rrif7541","l10n_hr_chart_template_rrif","income",,"Prihodi od internetskih usluga za inozemstvo","Prihodi od internetskih usluga za inozemstvo"
"7542","kp_rrif7542","l10n_hr_chart_template_rrif","income",,"Prihodi od poslovnih jedinica u inozemstvu","Prihodi od poslovnih jedinica u inozemstvu"
"7543","kp_rrif7543","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje turističkih usluga u inozem.","Prihodi od prodaje turističkih usluga u inozem."
"7550","kp_rrif7550","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje proizvoda i usluga poduzet. u kojima postoje sudjelujući interesi (do 20% udjela)-a1","Prihodi od prodaje proizvoda i usluga poduzetnicima u kojima postoje sudjelujući interesi (do 20% udjela)-Analitika 1"
"7560","kp_rrif7560","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje proizvoda i usluga ovisnim društvima (analitika po društvima)-a1","Prihodi od prodaje proizvoda i usluga ovisnim društvima (analitika po društvima)-Analitika 1"
"7570","kp_rrif7570","l10n_hr_chart_template_rrif","income",,"Prihodi od najmova i zakupa-a1","Prihodi od najmova i zakupa-Analitika 1"
"7580","kp_rrif7580","l10n_hr_chart_template_rrif","income",,"Prihodi iz ortaštva-a1","Prihodi iz ortaštva-Analitika 1"
"7590","kp_rrif7590","l10n_hr_chart_template_rrif","income",,"Ostali prihodi od prodaje učinaka-a1","Ostali prihodi od prodaje učinaka-Analitika 1"
"7600","kp_rrif7600","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje robe na veliko (analitika po prodajnim mjestima)","Prihodi od prodaje robe na veliko (analitika po prodajnim mjestima)"
"7601","kp_rrif7601","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje uvezene robe na veliko","Prihodi od prodaje uvezene robe na veliko"
"7602","kp_rrif7602","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje robe u tranzitu","Prihodi od prodaje robe u tranzitu"
"7603","kp_rrif7603","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje robe na malo (analitika po prodav.)","Prihodi od prodaje robe na malo (analitika po prodav.)"
"7604","kp_rrif7604","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje robe u povezanim društvima","Prihodi od prodaje robe u povezanim društvima"
"7605","kp_rrif7605","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje robe dane u komisiju ili konsignaciju","Prihodi od prodaje robe dane u komisiju ili konsignaciju"
"7606","kp_rrif7606","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje u poslov. jed. na području posebne držav. skrbi i Vukovaru","Prihodi od prodaje u poslov. jed. na području posebne držav. skrbi i Vukovaru"
"7607","kp_rrif7607","l10n_hr_chart_template_rrif","income",,"Prihodi od povratne naknade za ambalažu (bez PDV-a)","Prihodi od povratne naknade za ambalažu (bez PDV-a)"
"7608","kp_rrif7608","l10n_hr_chart_template_rrif","income",,"Prihodi od prometa nekretnina","Prihodi od prometa nekretnina"
"7610","kp_rrif7610","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje robe na inozemnom tržištu-a1","Prihodi od prodaje robe na inozemnom tržištu-Analitika 1"
"7620","kp_rrif7620","l10n_hr_chart_template_rrif","income",,"Prihodi od provizija","Prihodi od provizija"
"7621","kp_rrif7621","l10n_hr_chart_template_rrif","income",,"Prihodi od franšiza i robnih znakova","Prihodi od franšiza i robnih znakova"
"7622","kp_rrif7622","l10n_hr_chart_template_rrif","income",,"Prihodi od usluge posredovanja","Prihodi od usluge posredovanja"
"7623","kp_rrif7623","l10n_hr_chart_template_rrif","income",,"Prihodi od davanja mišljenja","Prihodi od davanja mišljenja"
"7630","kp_rrif7630","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje nekurentne robe (robe u kvaru, ošte-ćenju, demodirana i sl.)-a1","Prihodi od prodaje nekurentne robe (robe u kvaru, ošte-ćenju, demodirana i sl.)-Analitika 1"
"7640","kp_rrif7640","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje robe na robni kredit","Prihodi od prodaje robe na robni kredit"
"7641","kp_rrif7641","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje robe na potrošački kredit","Prihodi od prodaje robe na potrošački kredit"
"7650","kp_rrif7650","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje robe poduzetnicima u kojima postoje sudjelujući interesi (do 20% udjela)-a1","Prihodi od prodaje robe poduzetnicima u kojima postoje sudjelujući interesi (do 20% udjela)-Analitika 1"
"7660","kp_rrif7660","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje robe ovisnim društvima-a1","Prihodi od prodaje robe ovisnim društvima-Analitika 1"
"7670","kp_rrif7670","l10n_hr_chart_template_rrif","income",,"Prihodi od dane (prodane) robe u financijski lizing (najam)-a1","Prihodi od dane (prodane) robe u financijski lizing (najam)-Analitika 1"
"7680","kp_rrif7680","l10n_hr_chart_template_rrif","income",,"Prihodi od preprodaje nekretnina i umjetnina-a1","Prihodi od preprodaje nekretnina i umjetnina-Analitika 1"
"7690","kp_rrif7690","l10n_hr_chart_template_rrif","income",,"Prihodi od prikupljanja ambalaže","Prihodi od prikupljanja ambalaže"
"7691","kp_rrif7691","l10n_hr_chart_template_rrif","income",,"Prihodi od zbrinjavanja otpada","Prihodi od zbrinjavanja otpada"
"7692","kp_rrif7692","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje korisnog otpada","Prihodi od prodaje korisnog otpada"
"7700","kp_rrif7700","l10n_hr_chart_template_rrif","income",,"Prihodi od kamata od povezanih poduzetnika","Prihodi od kamata od povezanih poduzetnika"
"7701","kp_rrif7701","l10n_hr_chart_template_rrif","income",,"Prihodi od tečajnih razlika od povez. poduzet.","Prihodi od tečajnih razlika od povez. poduzet."
"7702","kp_rrif7702","l10n_hr_chart_template_rrif","income",,"Prihodi od dividende (dobitka) iz udjela u povezanim poduzetnicima","Prihodi od dividende (dobitka) iz udjela u povezanim poduzetnicima"
"7703","kp_rrif7703","l10n_hr_chart_template_rrif","income",,"Prihodi od valutne (indeksne) klauzule iz odnosa s povezanim poduzetnicima","Prihodi od valutne (indeksne) klauzule iz odnosa s povezanim poduzetnicima"
"7704","kp_rrif7704","l10n_hr_chart_template_rrif","income",,"Ostali financijski prihodi od povezanih poduzet","Ostali financijski prihodi od povezanih poduzet"
"7705","kp_rrif7705","l10n_hr_chart_template_rrif","income",,"Ostali financijski prihodi od povez. poduzetnika","Ostali financijski prihodi od povez. poduzetnika"
"7706","kp_rrif7706","l10n_hr_chart_template_rrif","income",,"Dobitci od prodaje udjela i dionica od povezanih poduzetnika","Dobitci od prodaje udjela i dionica od povezanih poduzetnika"
"7710","kp_rrif7710","l10n_hr_chart_template_rrif","income",,"Prihodi od redovnih kamata","Prihodi od redovnih kamata"
"7711","kp_rrif7711","l10n_hr_chart_template_rrif","income",,"Prihodi od zateznih kamata","Prihodi od zateznih kamata"
"7712","kp_rrif7712","l10n_hr_chart_template_rrif","income",,"Prihodi od kamata iz financijske imovine i dr.","Prihodi od kamata iz financijske imovine i dr."
"7713","kp_rrif7713","l10n_hr_chart_template_rrif","income",,"Kamate na depozite i jamčevine","Kamate na depozite i jamčevine"
"7720","kp_rrif7720","l10n_hr_chart_template_rrif","income",,"Pozitivne tečajne razlike iz tražbina i stanja deviza na računu","Pozitivne tečajne razlike iz tražbina i stanja deviza na računu"
"7721","kp_rrif7721","l10n_hr_chart_template_rrif","income",,"Pozitivne tečajne razlike iz nižih obveza prema inozemstvu","Pozitivne tečajne razlike iz nižih obveza prema inozemstvu"
"7722","kp_rrif7722","l10n_hr_chart_template_rrif","income",,"Prihodi od ostalih tečajnih razlika","Prihodi od ostalih tečajnih razlika"
"7730","kp_rrif7730","l10n_hr_chart_template_rrif","income",,"Prihodi od dividendi","Prihodi od dividendi"
"7731","kp_rrif7731","l10n_hr_chart_template_rrif","income",,"Prihodi od udjela u dobitku u d.o.o. i dr.","Prihodi od udjela u dobitku u d.o.o. i dr."
"7740","kp_rrif7740","l10n_hr_chart_template_rrif","income",,"Prihodi iz udjela u investicijskim i dr. fondovima","Prihodi iz udjela u investicijskim i dr. fondovima"
"7741","kp_rrif7741","l10n_hr_chart_template_rrif","income",,"Dobitci od prodaje dionica i udjela (s nepovez. društvima) i dr. vrijed. papira","Dobitci od prodaje dionica i udjela (s nepovez. društvima) i dr. vrijed. papira"
"7742","kp_rrif7742","l10n_hr_chart_template_rrif","income",,"Prihodi od primjene valutne (indeksne) klauzule","Prihodi od primjene valutne (indeksne) klauzule"
"7743","kp_rrif7743","l10n_hr_chart_template_rrif","income",,"Ostali financijski prihodi iz odnosa s nepovezanim poduzetnicima i dr. osobama","Ostali financijski prihodi iz odnosa s nepovezanim poduzetnicima i dr. osobama"
"7744","kp_rrif7744","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje ostale financijske imovine","Prihodi od prodaje ostale financijske imovine"
"7750","kp_rrif7750","l10n_hr_chart_template_rrif","income",,"Financijski prihodi (dividende, dobitci) iz udjela u društvima do 20%","Financijski prihodi (dividende, dobitci) iz udjela u društvima do 20%"
"7751","kp_rrif7751","l10n_hr_chart_template_rrif","income",,"Prihodi od kamata, teč. razlika i dr. fin. prihoda iz odnosa s pridruženim poduz. i od udjela do 20%","Prihodi od kamata, tečajnih razlika i dr. fin. prihoda iz odnosa s pridruženim poduzetnicima i od društava u kojima se drži udio do 20%"
"7760","kp_rrif7760","l10n_hr_chart_template_rrif","income",,"Prihodi iz procjene fin. imovine namijenjene za trgovanje (HSFI 15. t. 15.52. i MRS 39. t. 55a. i dr.)","Prihodi iz procjene fin. imovine namijenjene za trgovanje (HSFI 15. t. 15.52. i MRS 39. t. 55a. i dr.)"
"7761","kp_rrif7761","l10n_hr_chart_template_rrif","income",,"Dobitci iz promjene fer vrijednosti ulaganja u nekretnine (HSFI 7. i MRS 40)","Dobitci iz promjene fer vrijednosti ulaganja u nekretnine (HSFI 7. i MRS 40)"
"7762","kp_rrif7762","l10n_hr_chart_template_rrif","income",,"Dobitci iz promjene vrijednosti ostale imovine","Dobitci iz promjene vrijednosti ostale imovine"
"7770","kp_rrif7770","l10n_hr_chart_template_rrif","income",,"Prihodi negativnog goodwilla-a1","Prihodi negativnog goodwilla-Analitika 1"
"7780","kp_rrif7780","l10n_hr_chart_template_rrif","income",,"Prihodi iz burzovnih transakcija","Prihodi iz burzovnih transakcija"
"7781","kp_rrif7781","l10n_hr_chart_template_rrif","income",,"Prihodi iz faktoringa, forwarda, opcija i sl.","Prihodi iz faktoringa i sl."
"7782","kp_rrif7782","l10n_hr_chart_template_rrif","income",,"Prihodi od neplaćenih obveza - davanja (za šume, poreze, članarine, naknade i dr.)","Prihodi od neplaćenih obveza - davanja (za šume, poreze, članarine, naknade i dr.)"
"7783","kp_rrif7783","l10n_hr_chart_template_rrif","income",,"Prihodi iz fin. leasinga","Prihodi iz fin. leasinga"
"7784","kp_rrif7784","l10n_hr_chart_template_rrif","income",,"Prihodi od naplate životnog osiguranja","Prihodi od naplate životnog osiguranja"
"7800","kp_rrif7800","l10n_hr_chart_template_rrif","income",,"Otpisi obveza prema dobavljačima, obveza za primljene predujmove i sl.","Otpisi obveza prema dobavljačima, obveza za primljene predujmove i sl."
"7801","kp_rrif7801","l10n_hr_chart_template_rrif","income",,"Otpis obveza prema kreditorima","Otpis obveza prema kreditorima"
"7802","kp_rrif7802","l10n_hr_chart_template_rrif","income",,"Otpis obveza prema zaposlenicima","Otpis obveza prema zaposlenicima"
"7803","kp_rrif7803","l10n_hr_chart_template_rrif","income",,"Prihodi od zastare obveza","Prihodi od zastare obveza"
"7805","kp_rrif7805","l10n_hr_chart_template_rrif","income",,"Otpis ostalih obveza","Otpis ostalih obveza"
"7807","kp_rrif7807","l10n_hr_chart_template_rrif","income",,"Prihodi od naknadnih odobrenja - sniženja i popusta od dobavljača i dr.","Prihodi od naknadnih odobrenja - sniženja i popusta od dobavljača i dr."
"7810","kp_rrif7810","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje otpisanih i rashodovanih sredstava rada (alata, opreme i sl.)","Prihodi od prodaje otpisanih i rashodovanih sredstava rada (alata, opreme i sl.)"
"7811","kp_rrif7811","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje dugotrajne materijalne imovine (iz uporabe a amortizirane)","Prihodi od prodaje dugotrajne materijalne imovine (iz uporabe i amortiz.)"
"7812","kp_rrif7812","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje dugotr. nemater. imovine (trgov. znak, patent i dr.) - iz uporabe","Prihodi od prodaje dugotr. nemater. imovine (trgov. znak, patent i dr.) - iz uporabe"
"7813","kp_rrif7813","l10n_hr_chart_template_rrif","income",,"Prihodi od ranije otpisanih zaliha po novoj procjeni (HSFI t. 10.38. i MRS 2, t. 33.)","Prihodi od ranije otpisanih zaliha po novoj procjeni (HSFI t. 10.38. i MRS 2, t. 33.)"
"7814","kp_rrif7814","l10n_hr_chart_template_rrif","income",,"Inventurni viškovi na robi, proizvodima i zalihama sirovina, materijala, dijelova i dugotrajne imovine","Inventurni viškovi na robi, proizvodima i zalihama sirovina, materijala, dijelova i dugotrajne imovine"
"7815","kp_rrif7815","l10n_hr_chart_template_rrif","income",,"Viškovi u blagajni (novac, vrijed. papiri i dr.)","Viškovi u blagajni (novac, vrijed. papiri i dr.)"
"7816","kp_rrif7816","l10n_hr_chart_template_rrif","income",,"Viškovi iz neidentificiranih novčanih doznaka u tekućem poslovanju","Viškovi iz neidentificiranih novčanih doznaka u tekućem poslovanju"
"7817","kp_rrif7817","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje  ulaganja u nekretnine (MRS 40. t. 69.)","Prihodi od  ulaganja u nekretnine (MRS 40. t. 69.)"
"7818","kp_rrif7818","l10n_hr_chart_template_rrif","income",,"Prihodi od procjene prometa (utrška) po nalazu poreznog nadzora","Prihodi od procjene prometa (utrška) po nalazu poreznog nadzora"
"7819","kp_rrif7819","l10n_hr_chart_template_rrif","income",,"Prihodi od ostalih primitaka bez nadoknade","Prihodi od ostalih primitaka bez nadoknade"
"7820","kp_rrif7820","l10n_hr_chart_template_rrif","income",,"Prihodovanje dugoročnih rezerviranja","Prihodovanje dugoročnih rezerviranja"
"7821","kp_rrif7821","l10n_hr_chart_template_rrif","income",,"Naknadno utvrđeni prihodi","Naknadno utvrđeni prihodi"
"7822","kp_rrif7822","l10n_hr_chart_template_rrif","income",,"Ukidanje pasiv. vrem. razgraničenja","Ukidanje pasiv. vrem. razgraničenja"
"7824","kp_rrif7824","l10n_hr_chart_template_rrif","income",,"Prihodi od ukidanja troškova od kojih se odustalo","Prihodi od ukidanja troškova od kojih se odustalo"
"7825","kp_rrif7825","l10n_hr_chart_template_rrif","income",,"Prihodi od naknadno naplaćenih jamstava - garancija","Prihodi od naknadno naplaćenih jamstava - garancija"
"7826","kp_rrif7826","l10n_hr_chart_template_rrif","income",,"Prihodi od naknadno naplaćenih potraživanja iz prethodnih godina","Prihodi od naknadno naplaćenih potraživanja iz prethodnih godina"
"7827","kp_rrif7827","l10n_hr_chart_template_rrif","income",,"Prihodi od naplate iz ugovora po naknadnim priznanjima iz prošlih godina","Prihodi od naplate iz ugovora po naknadnim priznanjima iz prošlih godina"
"7828","kp_rrif7828","l10n_hr_chart_template_rrif","income",,"Prihodi od naknadno naplaćenih reklamacija","Prihodi od naknadno naplaćenih reklamacija"
"7829","kp_rrif7829","l10n_hr_chart_template_rrif","income",,"Prihodi od zaprimanja dobara koja su prodana u prethodnom obrač. razdoblju","Prihodi od zaprimanja dobara koja su prodana u prethodnom obrač. razdoblju"
"7830","kp_rrif7830","l10n_hr_chart_template_rrif","income",,"Prihodi od refundacije za rad radnika","Prihodi od refundacije za rad radnika"
"7831","kp_rrif7831","l10n_hr_chart_template_rrif","income",,"Prihodi od naknada šteta iz tekućeg poslovanja","Prihodi od naknada šteta iz tekućeg poslovanja"
"7832","kp_rrif7832","l10n_hr_chart_template_rrif","income",,"Prihodi od subvencija","Prihodi od subvencija"
"7833","kp_rrif7833","l10n_hr_chart_template_rrif","income",,"Prihodi od dotacija i pomoći","Prihodi od dotacija i pomoći"
"7834","kp_rrif7834","l10n_hr_chart_template_rrif","income",,"Prihodi s osnove basplatnog primitka opreme, nekretnina, zaliha i potraživanja","Prihodi s osnove basplatnog primitka opreme, nekretnina, zaliha i potraživanja"
"7835","kp_rrif7835","l10n_hr_chart_template_rrif","income",,"Prihodi od naknada za jamstva","Prihodi od naknada za jamstva"
"7836","kp_rrif7836","l10n_hr_chart_template_rrif","income",,"Prihodi od prefakturiranih troškova (npr. komunalnih, premija osiguranja i dr.)","Prihodi od prefakturiranih troškova (npr. komunalnih, premija osiguranja i dr.)"
"7837","kp_rrif7837","l10n_hr_chart_template_rrif","income",,"Prihodi za pokriće gubitka","Prihodi za pokriće gubitka"
"7838","kp_rrif7838","l10n_hr_chart_template_rrif","income",,"Prihodi za manjkove od odgovornih osoba","Prihodi za manjkove od odgovornih osoba"
"7839","kp_rrif7839","l10n_hr_chart_template_rrif","income",,"Prihodi od ostalih nadoknada iz poslovanja","Prihodi od ostalih nadoknada iz poslovanja"
"7840","kp_rrif7840","l10n_hr_chart_template_rrif","income",,"Prihodi od revalorizacije zaliha","Prihodi od revalorizacije zaliha"
"7841","kp_rrif7841","l10n_hr_chart_template_rrif","income",,"Prihodi od revalorizacije financijske imovine raspoložive za prodaju","Prihodi od revalorizacije financijske imovine raspoložive za prodaju"
"7842","kp_rrif7842","l10n_hr_chart_template_rrif","income",,"Prihodi od ukidanja gubitka (MRS 36)","Prihodi od ukidanja gubitka (MRS 36)"
"7843","kp_rrif7843","l10n_hr_chart_template_rrif","income",,"Prihodi od procjene zaliha i dr. imovine","Prihodi od procjene zaliha i dr. imovine"
"7844","kp_rrif7844","l10n_hr_chart_template_rrif","income",,"Prihodi od procjene ostale imovine","Prihodi od procjene ostale imovine"
"7850","kp_rrif7850","l10n_hr_chart_template_rrif","income",,"Prihodi s osnove povrata poreza na promet","Prihodi s osnove povrata poreza na promet"
"7851","kp_rrif7851","l10n_hr_chart_template_rrif","income",,"Prihodi od ugovorenih i naplaćenih penala zbog neizvršenja roka u isporuci","Prihodi od ugovorenih i naplaćenih penala zbog neizvršenja roka u isporuci"
"7852","kp_rrif7852","l10n_hr_chart_template_rrif","income",,"Prihodi od nagrada za proizvod, uslugu, oblik i sl.","Prihodi od nagrada za proizvod, uslugu, oblik i sl."
"7853","kp_rrif7853","l10n_hr_chart_template_rrif","income",,"Prihodi od (nevraćenih) kaucija i depozita","Prihodi od (nevraćenih) kaucija i depozita"
"7854","kp_rrif7854","l10n_hr_chart_template_rrif","income",,"Prihodi od kapara, odustatnina i sl.","Prihodi od kapara, odustatnina i sl."
"7855","kp_rrif7855","l10n_hr_chart_template_rrif","income",,"Prihodi od vraćenih premija osiguranja","Prihodi od vraćenih premija osiguranja"
"7856","kp_rrif7856","l10n_hr_chart_template_rrif","income",,"Prihodi od financ. inženjeringa(projekt., financ. i izgr., kupoprodaja poduzeća i sl.) i provizija","Prihodi od financijskog inženjeringa (npr. projektiranje, financiranje i izgr. objekata, kupnja i prodaja poduzeća i sl.) i provizija"
"7857","kp_rrif7857","l10n_hr_chart_template_rrif","income",,"Prihodi od prodaje prava (patenata, licencija, koncesija, rente, imena, znaka i sl.)","Prihodi od prodaje prava (patenata, licencija, koncesija, rente, imena, znaka i sl.)"
"7858","kp_rrif7858","l10n_hr_chart_template_rrif","income",,"Prihodi od naplate šteta uništene imovine (požarom, poplavom i dr. višom silom)","Prihodi od naplate šteta uništene imovine (požarom, poplavom i dr. višom silom)"
"7859","kp_rrif7859","l10n_hr_chart_template_rrif","income",,"Prihodi od naplate šteta po sudskim procesima (zbog oduzete imovine,zlouporabe znaka,imena,prava i dr.)","Prihodi od naplate šteta po sudskim procesima (npr. zbog oduzete imovine, zlouporabe znaka, imena, prava i dr.)"
"7860","kp_rrif7860","l10n_hr_chart_template_rrif","income",,"Prihodi od državnih potpora za pokriće troškova","Prihodi od državnih potpora za pokriće troškova"
"7861","kp_rrif7861","l10n_hr_chart_template_rrif","income",,"Prihodi (odgođeni) od državnih potpora za investicije (sredstva)","Prihodi od državnih potpora za investicije (sredstva)"
"7862","kp_rrif7862","l10n_hr_chart_template_rrif","income",,"Prihodi od državnih potpora za ostale određene namjene","Prihodi od državnih potpora za ostale određene namjene"
"7870","kp_rrif7870","l10n_hr_chart_template_rrif","income",,"Dobitci od prirasta biološke imovine","Dobitci od prirasta biološke imovine"
"7871","kp_rrif7871","l10n_hr_chart_template_rrif","income",,"Dobitci od procjene biološke imovine","Dobitci od procjene biološke imovine"
"7872","kp_rrif7872","l10n_hr_chart_template_rrif","income",,"Dobitci od procjene poljoprivrednih proizvoda","Dobitci od procjene poljoprivrednih proizvoda"
"7880","kp_rrif7880","l10n_hr_chart_template_rrif","income",,"Prihodi uporabe vlastitih proizvoda i usluga za troškove","Prihodi uporabe vlastitih proizvoda i usluga za troškove"
"7881","kp_rrif7881","l10n_hr_chart_template_rrif","income",,"Prihodi od uporabe vlastitih proizvoda i usluga za dugotrajnu imovinu","Prihodi od uporabe vlastitih proizvoda i usluga za dugotrajnu imovinu"
"7890","kp_rrif7890","l10n_hr_chart_template_rrif","income",,"Prihod od izvanredne prodaje značajnog dijela imovine","Prihod od izvanredne prodaje značajnog dijela imovine"
"7891","kp_rrif7891","l10n_hr_chart_template_rrif","income",,"Prihod od dugotrajne materijalne imovine namijenjene prodaji","Prihod od dugotrajne materijalne imovine namijenjene prodaji"
"7892","kp_rrif7892","l10n_hr_chart_template_rrif","income",,"Prihod od izvansudskih nagodbi","Prihod od izvansudskih nagodbi"
"7899","kp_rrif7899","l10n_hr_chart_template_rrif","income",,"Ostali nepredviđeni prihodi","Ostali nepredviđeni prihodi"
"7900","kp_rrif7900","l10n_hr_chart_template_rrif","off_balance",,"Razlika prihoda i rashoda (iz cjelokupnog poslovanja)-a1","Razlika prihoda i rashoda (iz cjelokupnog poslovanja)-Analitika 1"
"8000","kp_rrif8000","l10n_hr_chart_template_rrif","off_balance",,"Dobitak prije oporezivanja-a1","Dobitak prije oporezivanja-Analitika 1"
"8010","kp_rrif8010","l10n_hr_chart_template_rrif","off_balance",,"Gubitak prije oporezivanja-a1","Gubitak prije oporezivanja-Analitika 1"
"8030","kp_rrif8030","l10n_hr_chart_template_rrif","off_balance",,"Porez na dobitak (gubitak)-a1","Porez na dobitak (gubitak)-Analitika 1"
"8040","kp_rrif8040","l10n_hr_chart_template_rrif","off_balance",,"Dobitak razdoblja (poslije poreza)","Dobitak razdoblja (poslije poreza)"
"8041","kp_rrif8041","l10n_hr_chart_template_rrif","off_balance",,"Gubitak razdoblja","Gubitak razdoblja"
"8100","kp_rrif8100","l10n_hr_chart_template_rrif","off_balance",,"Dobitak pripisan imateljima kapitala matice-a1","Dobitak pripisan imateljima kapitala matice-Analitika 1"
"8110","kp_rrif8110","l10n_hr_chart_template_rrif","off_balance",,"Gubitak koji tereti imatelje kapitala matice-a1","Gubitak koji tereti imatelje kapitala matice-Analitika 1"
"8130","kp_rrif8130","l10n_hr_chart_template_rrif","off_balance",,"Dobitak pripisan manjinskom interesu-a1","Dobitak pripisan manjinskom interesu-Analitika 1"
"8140","kp_rrif8140","l10n_hr_chart_template_rrif","off_balance",,"Gubitak koji tereti manjinski interes-a1","Gubitak koji tereti manjinski interes-Analitika 1"
"9000","kp_rrif9000","l10n_hr_chart_template_rrif","liability_current",,"Upisani temeljni kapital članova d.o.o. (analitika po članovima)","Upisani temeljni kapital članova d.o.o. (analitika po članovima)"
"9001","kp_rrif9001","l10n_hr_chart_template_rrif","liability_current",,"Temeljni dionički kapital (obične dionice)","Temeljni dionički kapital (obične dionice)"
"9002","kp_rrif9002","l10n_hr_chart_template_rrif","liability_current",,"Temeljni dionički kapital (povlaštene dionice)","Temeljni dionički kapital (povlaštene dionice)"
"9010","kp_rrif9010","l10n_hr_chart_template_rrif","liability_current",,"Upisani temeljni kapital manjinskih članova-a1","Upisani temeljni kapital manjinskih članova-Analitika 1"
"9020","kp_rrif9020","l10n_hr_chart_template_rrif","liability_current",,"Upisani temeljni kapital koji je pozvan za uplatu (analitika po upisnicima)","Upisani temeljni kapital koji je pozvan za uplatu (analitika po upisnicima)"
"9030","kp_rrif9030","l10n_hr_chart_template_rrif","liability_current",,"Kapital (ulozi) članova javnog trgovačkog društva-a1","Kapital (ulozi) članova javnog trgovačkog društva-Analitika 1"
"9040","kp_rrif9040","l10n_hr_chart_template_rrif","liability_current",,"Kapital (ulozi) komanditora komanditnog društva-a1","Kapital (ulozi) komanditora komanditnog društva-Analitika 1"
"9050","kp_rrif9050","l10n_hr_chart_template_rrif","liability_current",,"Državni kapital u udjelima-a1","Državni kapital u udjelima-Analitika 1"
"9060","kp_rrif9060","l10n_hr_chart_template_rrif","liability_current",,"Kapital članova zadruge-a1","Kapital članova zadruge-Analitika 1"
"9100","kp_rrif9100","l10n_hr_chart_template_rrif","liability_current",,"Uplaćeni udjeli - dionice iznad svote temeljnog kapitala-a1","Uplaćeni udjeli - dionice iznad svote temeljnog kapitala-Analitika 1"
"9110","kp_rrif9110","l10n_hr_chart_template_rrif","liability_current",,"Kapitalne pričuve iz dodatnih uplata radi stjecanja posebnih prava u društvu(ili zamjenjivih obveznica)-a1","Kapitalne pričuve iz dodatnih uplata radi stjecanja posebnih prava u društvu (ili zamjenjivih obveznica)-Analitika 1"
"9120","kp_rrif9120","l10n_hr_chart_template_rrif","liability_current",,"Kapitalne pričuve iz uplata dodatnih činidbi-a1","Kapitalne pričuve iz uplata dodatnih činidbi-Analitika 1"
"9130","kp_rrif9130","l10n_hr_chart_template_rrif","liability_current",,"Kapitalne pričuve iz ostatka pri smanjenju temeljnog kapitala-a1","Kapitalne pričuve iz ostatka pri smanjenju temeljnog kapitala-Analitika 1"
"9140","kp_rrif9140","l10n_hr_chart_template_rrif","liability_current",,"Kapitalne pričuve iz drugih izvora-a1","Kapitalne pričuve iz drugih izvora-Analitika 1"
"9150","kp_rrif9150","l10n_hr_chart_template_rrif","liability_current",,"Kapitalni dobitak iz prodaje vlastitih udjela - dionica-a1","Kapitalni dobitak iz prodaje vlastitih udjela - dionica-Analitika 1"
"9160","kp_rrif9160","l10n_hr_chart_template_rrif","liability_current",,"Kapitalni dobitak na prodane emitirane dionice-a1","Kapitalni dobitak na prodane emitirane dionice-Analitika 1"
"9170","kp_rrif9170","l10n_hr_chart_template_rrif","liability_current",,"Kapitalne pričuve iz ulaganja tajnog člana društva (čl. 148. ZTD-a)-a1","Kapitalne pričuve iz ulaganja tajnog člana društva (čl. 148. ZTD-a)-Analitika 1"
"9190","kp_rrif9190","l10n_hr_chart_template_rrif","liability_current",,"Kapital iz ulaganja obrtnika dobitaša-a1","Kapital iz ulaganja obrtnika dobitaša-Analitika 1"
"9200","kp_rrif9200","l10n_hr_chart_template_rrif","liability_current",,"Pričuve prema ZTD","Pričuve prema ZTD"
"9201","kp_rrif9201","l10n_hr_chart_template_rrif","liability_current",,"Pričuve za povećanje temeljnog kapitala","Pričuve za povećanje temeljnog kapitala"
"9202","kp_rrif9202","l10n_hr_chart_template_rrif","liability_current",,"Pričuve za pokriće gubitka","Pričuve za pokriće gubitka"
"9210","kp_rrif9210","l10n_hr_chart_template_rrif","liability_current",,"Pričuve za opcijske dionice za zaposlenike","Pričuve za opcijske dionice za zaposlenike"
"9211","kp_rrif9211","l10n_hr_chart_template_rrif","liability_current",,"Pričuve za otkupljene dionice radi obeštećenja dioničara","Pričuve za otkupljene dionice radi obeštećenja dioničara"
"9212","kp_rrif9212","l10n_hr_chart_template_rrif","liability_current",,"Pričuve za otkup dionica da bi se spriječila šteta i dr.","Pričuve za otkup dionica da bi se spriječila šteta i dr."
"9213","kp_rrif9213","l10n_hr_chart_template_rrif","liability_current",,"Pričuve za vlastite (trezorske) dionice i udjele","Pričuve za vlastite (trezorske) dionice i udjele"
"9220","kp_rrif9220","l10n_hr_chart_template_rrif","liability_current",,"Otkupljene vlastite dionice","Otkupljene vlastite dionice"
"9221","kp_rrif9221","l10n_hr_chart_template_rrif","liability_current",,"Otkupljeni vlastiti udjeli","Otkupljeni vlastiti udjeli"
"9222","kp_rrif9222","l10n_hr_chart_template_rrif","liability_current",,"Vlastite dionice za opcijsku namjenu","Vlastite dionice za opcijsku namjenu"
"9230","kp_rrif9230","l10n_hr_chart_template_rrif","liability_current",,"Pričuve za održavanje financijske stabilnosti društva, za razvojne aktivnosti i sl.","Pričuve za održavanje financijske stabilnosti društva, za razvojne aktivnosti i sl."
"9231","kp_rrif9231","l10n_hr_chart_template_rrif","liability_current",,"Pričuve za restrukturiranje","Pričuve za restrukturiranje"
"9232","kp_rrif9232","l10n_hr_chart_template_rrif","liability_current",,"Pričuve radi održavanja boniteta strukture izvora financiranja","Pričuve radi održavanja boniteta strukture izvora financiranja"
"9240","kp_rrif9240","l10n_hr_chart_template_rrif","liability_current",,"Pričuve za pokriće gubitka u poslovanju","Pričuve za pokriće gubitka u poslovanju"
"9241","kp_rrif9241","l10n_hr_chart_template_rrif","liability_current",,"Pričuve za nagrade i sl.","Pričuve za nagrade i sl."
"9242","kp_rrif9242","l10n_hr_chart_template_rrif","liability_current",,"Slobodne pričuve iz neraspoređenog dobitka","Slobodne pričuve iz neraspoređenog dobitka"
"9243","kp_rrif9243","l10n_hr_chart_template_rrif","liability_current",,"Pričuve za nerealizirane dobitke iz udjela","Pričuve za nerealizirane dobitke iz udjela"
"9300","kp_rrif9300","l10n_hr_chart_template_rrif","liability_current",,"Revalorizacijske pričuve iz procjene dug. nemat. i mat. imovine (HSFIt.6.36.iMRS16,t.39.iMRS38,t.85)","Revalorizacijske pričuve iz procjene dugotrajne nematerijalne i materijalne imovine (HSFI t. 6.36. i MRS 16, t. 39. i MRS 38, t. 85.) (analitika prema revaloriziranim imovinskim stavkama)"
"9301","kp_rrif9301","l10n_hr_chart_template_rrif","liability_current",,"Pričuve iz revalorizacije financijske imovine","Pričuve iz revalorizacije financijske imovine"
"9303","kp_rrif9303","l10n_hr_chart_template_rrif","liability_current",,"Ostale revalorizacijske pričuve","Ostale revalorizacijske pričuve"
"9310","kp_rrif9310","l10n_hr_chart_template_rrif","liability_current",,"Revalorizacijske pričuve iz razdoblja od 2001. do 2004.","Revalorizacijske pričuve iz razdoblja od 2001. do 2004."
"9311","kp_rrif9311","l10n_hr_chart_template_rrif","liability_current",,"Revalorizacijske pričuve nastale do kraja 2000.","Revalorizacijske pričuve nastale do kraja 2000."
"9320","kp_rrif9320","l10n_hr_chart_template_rrif","liability_current",,"Pričuve iz tečajnih razlika od ulaganja u inozemno poslovanje (MRS 21. t. 39.)-a1","Pričuve iz tečajnih razlika od ulaganja u inozemno poslovanje (MRS 21. t. 39.)-Analitika 1"
"9330","kp_rrif9330","l10n_hr_chart_template_rrif","liability_current",,"Dobitci/gubitci od udjela u kapitalu do prestanka priznavanja (MRS 1. t. 95. i MRS 39. 55.b)-a1","Dobitci/gubitci od udjela u kapitalu do prestanka priznavanja (MRS 1. t. 95. i MRS 39. 55.b)-Analitika 1"
"9340","kp_rrif9340","l10n_hr_chart_template_rrif","liability_current",,"Dobitci/gubitci iz zaštite novčanog toka do reklasifikacije (MRS 1. t. 95. i MRS 39. t. 100.)-a1","Dobitci/gubitci iz zaštite novčanog toka do reklasifikacije (MRS 1. t. 95. i MRS 39. t. 100.)-Analitika 1"
"9350","kp_rrif9350","l10n_hr_chart_template_rrif","liability_current",,"Ostali akumulirani sveobuhvatni dobitci/gubitci - pričuve (MRS 1. t. 96.)-a1","Ostali akumulirani sveobuhvatni dobitci/gubitci - pričuve (MRS 1. t. 96.)-Analitika 1"
"94000","kp_rrif94000","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak članova društva (analitika po članovima)","Zadržani dobitak članova društva (analitika po članovima)"
"94001","kp_rrif94001","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak - neisplaćena dividenda","Zadržani dobitak - neisplaćena dividenda"
"94002","kp_rrif94002","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak koji se izuzima od isplate (npr. za investicije u imovinu)","Zadržani dobitak koji se izuzima od isplate (npr. za investicije u imovinu)"
"94003","kp_rrif94003","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak koji čeka raspored","Zadržani dobitak koji čeka raspored"
"94004","kp_rrif94004","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak za privatne troškove članova društva","Zadržani dobitak za privatne troškove članova društva"
"94005","kp_rrif94005","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak za manjinske članove društva","Zadržani dobitak za manjinske članove društva"
"94006","kp_rrif94006","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak tajnog člana društva","Zadržani dobitak tajnog člana društva"
"94010","kp_rrif94010","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak članova društva (analitika po članovima)","Zadržani dobitak članova društva (analitika po članovima)"
"94011","kp_rrif94011","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak - neispl. dividende","Zadržani dobitak - neispl. dividende"
"94012","kp_rrif94012","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak izuzet od isplate (npr. za investicije u imovinu)","Zadržani dobitak izuzet od isplate (npr. za investicije u imovinu)"
"94013","kp_rrif94013","l10n_hr_chart_template_rrif","liability_current",,"Neraspoređeni zadržani dobitak","Neraspoređeni zadržani dobitak"
"94020","kp_rrif94020","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak članova društva (analitika po članovima)","Zadržani dobitak članova društva (analitika po članovima)"
"94021","kp_rrif94021","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak - neispl. dividenda","Zadržani dobitak - neispl. dividenda"
"94022","kp_rrif94022","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak izuzet od isplate (npr. za investicije u imovinu)","Zadržani dobitak izuzet od isplate (npr. za investicije u imovinu)"
"94023","kp_rrif94023","l10n_hr_chart_template_rrif","liability_current",,"Neraspoređeni zadržani dobitak","Neraspoređeni zadržani dobitak"
"9403","kp_rrif9403","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak oblikovan iz realizirane rev. pričuve","Zadržani dobitak oblikovan iz realizirane rev. pričuve"
"9404","kp_rrif9404","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak po prijedlogu uprave i N.O.","Zadržani dobitak po prijedlogu uprave i N.O."
"9405","kp_rrif9405","l10n_hr_chart_template_rrif","liability_current",,"Zadržani dobitak iz negativnog goodwilla","Zadržani dobitak iz negativnog goodwilla"
"9410","kp_rrif9410","l10n_hr_chart_template_rrif","liability_current",,"Preneseni gubitak (iz godine 200_.)","Preneseni gubitak (iz godine 200_.)"
"9411","kp_rrif9411","l10n_hr_chart_template_rrif","liability_current",,"Preneseni gubitak (iz godine 201_.)","Preneseni gubitak (iz godine 201_.)"
"9417","kp_rrif9417","l10n_hr_chart_template_rrif","liability_current",,"Gubitak (dio) koji je iznad kapitala","Gubitak (dio) koji je iznad kapitala"
"9500","kp_rrif9500","l10n_hr_chart_template_rrif","liability_current",,"Dobitak financijske godine (neraspoređen)","Dobitak financijske godine (neraspoređen)"
"9501","kp_rrif9501","l10n_hr_chart_template_rrif","liability_current",,"Dobitak za isplate i izuzimanja u tijeku godine (analitika po članovima)","Dobitak za isplate i izuzimanja u tijeku godine (analitika po članovima)"
"9502","kp_rrif9502","l10n_hr_chart_template_rrif","liability_current",,"Dobitak - dividenda financijske godine (analitika prema članovima društva - dioničarima)","Dobitak - dividenda financijske godine (analitika prema članovima društva - dioničarima)"
"9504","kp_rrif9504","l10n_hr_chart_template_rrif","liability_current",,"Dobitak koji se privremeno ne isplaćuje po prijedlogu uprave i N.O.","Dobitak koji se privremeno ne isplaćuje po prijedlogu uprave i N.O."
"9505","kp_rrif9505","l10n_hr_chart_template_rrif","liability_current",,"Dobitak za manjinske članove društva","Dobitak za manjinske članove društva"
"9506","kp_rrif9506","l10n_hr_chart_template_rrif","liability_current",,"Dobitak za tajnog člana društva","Dobitak za tajnog člana društva"
"9507","kp_rrif9507","l10n_hr_chart_template_rrif","liability_current",,"Dobitak za nagrade zaposlenicima","Dobitak za nagrade zaposlenicima"
"9509","kp_rrif9509","l10n_hr_chart_template_rrif","liability_current",,"Dobitak iz privremenih razlika (odgođeni porezi)","Dobitak iz privremenih razlika (odgođeni porezi)"
"9510","kp_rrif9510","l10n_hr_chart_template_rrif","liability_current",,"Gubitak koji se pokriva","Gubitak koji se pokriva"
"9511","kp_rrif9511","l10n_hr_chart_template_rrif","liability_current",,"Nepokriveni gubitak","Nepokriveni gubitak"
"9512","kp_rrif9512","l10n_hr_chart_template_rrif","liability_current",,"Gubitak iz privremenih razlika","Gubitak iz privremenih razlika"
"9600","kp_rrif9600","l10n_hr_chart_template_rrif","liability_current",,"Kapital manjinskog društva (privremeno u postupku konsolidacije kao dio koji nije pod kontrolom matice)","Kapital manjinskog društva (ovaj račun privremeno se rabi u postupku konsolidacije društva kao dio kapitala koji nije pod kontrolom matice)"
"9900","kp_rrif9900","l10n_hr_chart_template_rrif","off_balance",,"Primljena roba u komisiju i konsignaciju (tuđa)","Primljena roba u komisiju i konsignaciju (tuđa)"
"9901","kp_rrif9901","l10n_hr_chart_template_rrif","off_balance",,"Materijal i roba u doradi (tuđa)","Materijal i roba u doradi (tuđa)"
"9902","kp_rrif9902","l10n_hr_chart_template_rrif","off_balance",,"Pozajmica strojeva i alata","Pozajmica strojeva i alata"
"9903","kp_rrif9903","l10n_hr_chart_template_rrif","off_balance",,"Zaštitna odjeća i obuća na korištenju","Zaštitna odjeća i obuća na korištenju"
"9904","kp_rrif9904","l10n_hr_chart_template_rrif","off_balance",,"Roba u skladištu (tuđa)","Roba u skladištu (tuđa)"
"9905","kp_rrif9905","l10n_hr_chart_template_rrif","off_balance",,"Ambalaža na korištenju (tuđa)","Ambalaža na korištenju (tuđa)"
"9906","kp_rrif9906","l10n_hr_chart_template_rrif","off_balance",,"Vlasništvo ortačke zajednice","Vlasništvo ortačke zajednice"
"9907","kp_rrif9907","l10n_hr_chart_template_rrif","off_balance",,"Materijali za doradne - lohn-poslove","Materijali za doradne - lohn-poslove"
"9908","kp_rrif9908","l10n_hr_chart_template_rrif","off_balance",,"Roba u izvozu","Roba u izvozu"
"9909","kp_rrif9909","l10n_hr_chart_template_rrif","off_balance",,"Zgrade i zemljišta u zakupu","Zgrade i zemljišta u zakupu"
"9910","kp_rrif9910","l10n_hr_chart_template_rrif","off_balance",,"Prava na korištenja","Prava na korištenja"
"9911","kp_rrif9911","l10n_hr_chart_template_rrif","off_balance",,"Krediti ugovoreni","Krediti ugovoreni"
"9912","kp_rrif9912","l10n_hr_chart_template_rrif","off_balance",,"Hipoteka na tuđoj imovini","Hipoteka na tuđoj imovini"
"9913","kp_rrif9913","l10n_hr_chart_template_rrif","off_balance",,"Materijalna prava","Materijalna prava"
"9914","kp_rrif9914","l10n_hr_chart_template_rrif","off_balance",,"Prava po loro akreditivima (domaći partneri)","Prava po loro akreditivima (domaći partneri)"
"9915","kp_rrif9915","l10n_hr_chart_template_rrif","off_balance",,"Prava po loro akreditivima (inozemni partneri)","Prava po loro akreditivima (inozemni partneri)"
"9916","kp_rrif9916","l10n_hr_chart_template_rrif","off_balance",,"Prava na ratne reparacije i štete od oduzete imovine","Prava na ratne reparacije i štete od oduzete imovine"
"9920","kp_rrif9920","l10n_hr_chart_template_rrif","off_balance",,"Primljeni čekovi, mjenice za osiguranje otplate anuiteta za dobivene robne i financijske kredite","Primljeni čekovi, mjenice za osiguranje otplate anuiteta za dobivene robne i financijske kredite"
"9921","kp_rrif9921","l10n_hr_chart_template_rrif","off_balance",,"Primljena jamstva vjerovnika kao instrumenata plaćanja","Primljena jamstva vjerovnika kao instrumenata plaćanja"
"9922","kp_rrif9922","l10n_hr_chart_template_rrif","off_balance",,"Primljene zadužnice","Primljene zadužnice"
"9923","kp_rrif9923","l10n_hr_chart_template_rrif","off_balance",,"Ostali vrijednosni papiri koji nisu stavljeni u optjecaj","Ostali vrijednosni papiri koji nisu stavljeni u optjecaj"
"9924","kp_rrif9924","l10n_hr_chart_template_rrif","off_balance",,"Izdane zadužnice","Izdane zadužnice"
"9925","kp_rrif9925","l10n_hr_chart_template_rrif","off_balance",,"Izdane mjenice","Izdane mjenice"
"9926","kp_rrif9926","l10n_hr_chart_template_rrif","off_balance",,"Korištene garancije u tijeku","Korištene garancije u tijeku"
"9927","kp_rrif9927","l10n_hr_chart_template_rrif","off_balance",,"Tražbine od kupaca iz zastupničke prodaje","Tražbine od kupaca iz zastupničke prodaje"
"9930","kp_rrif9930","l10n_hr_chart_template_rrif","off_balance",,"Obveznice i druge vrijednosti na skladištu (blagajni)","Obveznice i druge vrijednosti na skladištu (blagajni)"
"9931","kp_rrif9931","l10n_hr_chart_template_rrif","off_balance",,"Vrijednosni papiri na čuvanju (obveznice, dionice)","Vrijednosni papiri na čuvanju (obveznice, dionice)"
"9932","kp_rrif9932","l10n_hr_chart_template_rrif","off_balance",,"Blokovi ulaznica","Blokovi ulaznica"
"9933","kp_rrif9933","l10n_hr_chart_template_rrif","off_balance",,"Prodajna mjesta za izdane obveznice","Prodajna mjesta za izdane obveznice"
"9940","kp_rrif9940","l10n_hr_chart_template_rrif","off_balance",,"Investicija - ulaganje - rashodi","Investicija - ulaganje - rashodi"
"9941","kp_rrif9941","l10n_hr_chart_template_rrif","off_balance",,"Prihod - priljev","Prihod - priljev"
"9942","kp_rrif9942","l10n_hr_chart_template_rrif","off_balance",,"Dobitak","Dobitak"
"9943","kp_rrif9943","l10n_hr_chart_template_rrif","off_balance",,"Porezi i druga davanja","Porezi i druga davanja"
"9944","kp_rrif9944","l10n_hr_chart_template_rrif","off_balance",,"Čisti dobitak","Čisti dobitak"
"9947","kp_rrif9947","l10n_hr_chart_template_rrif","off_balance",,"Trošak kamata budućeg razdoblja","Trošak kamata budućeg razdoblja"
"9950","kp_rrif9950","l10n_hr_chart_template_rrif","off_balance",,"Obveze prema vlasnicima robe u komisiji i konsignaciji (tuđa sredstva)","Obveze prema vlasnicima robe u komisiji i konsignaciji (tuđa sredstva)"
"9951","kp_rrif9951","l10n_hr_chart_template_rrif","off_balance",,"Vlasnici materijala i robe u doradi","Vlasnici materijala i robe u doradi"
"9952","kp_rrif9952","l10n_hr_chart_template_rrif","off_balance",,"Vlasnici pozajmljenih strojeva i alata","Vlasnici pozajmljenih strojeva i alata"
"9953","kp_rrif9953","l10n_hr_chart_template_rrif","off_balance",,"Skladište zaštitne odjeće i obuće","Skladište zaštitne odjeće i obuće"
"9954","kp_rrif9954","l10n_hr_chart_template_rrif","off_balance",,"Vlasnici robe u našim skladištima","Vlasnici robe u našim skladištima"
"9955","kp_rrif9955","l10n_hr_chart_template_rrif","off_balance",,"Vlasnici ambalaže u korištenju","Vlasnici ambalaže u korištenju"
"9956","kp_rrif9956","l10n_hr_chart_template_rrif","off_balance",,"Ortaci - vlasništvo ortačke zajednice","Ortaci - vlasništvo ortačke zajednice"
"9957","kp_rrif9957","l10n_hr_chart_template_rrif","off_balance",,"Obveze za materijale u doradi - lohnu","Obveze za materijale u doradi - lohnu"
"9958","kp_rrif9958","l10n_hr_chart_template_rrif","off_balance",,"Obveze za robu u izvozu (tuđa roba)","Obveze za robu u izvozu (tuđa roba)"
"9959","kp_rrif9959","l10n_hr_chart_template_rrif","off_balance",,"Vlasnici zemljišta i zgrada u zakupu","Vlasnici zemljišta i zgrada u zakupu"
"9960","kp_rrif9960","l10n_hr_chart_template_rrif","off_balance",,"Izvor prava na korištenje","Izvor prava na korištenje"
"9961","kp_rrif9961","l10n_hr_chart_template_rrif","off_balance",,"Krediti odobreni","Krediti odobreni"
"9962","kp_rrif9962","l10n_hr_chart_template_rrif","off_balance",,"Dužnici po hipoteci","Dužnici po hipoteci"
"9963","kp_rrif9963","l10n_hr_chart_template_rrif","off_balance",,"Ulagači u materijalna prava","Ulagači u materijalna prava"
"9964","kp_rrif9964","l10n_hr_chart_template_rrif","off_balance",,"Izvori prava loro-akreditiva (domaći partneri)","Izvori prava loro-akreditiva (domaći partneri)"
"9965","kp_rrif9965","l10n_hr_chart_template_rrif","off_balance",,"Izvori prava loro-akreditiva (inozemni partneri)","Izvori prava loro-akreditiva (inozemni partneri)"
"9966","kp_rrif9966","l10n_hr_chart_template_rrif","off_balance",,"Ratne reparacije i nadoknade šteta","Ratne reparacije i nadoknade šteta"
"9967","kp_rrif9967","l10n_hr_chart_template_rrif","off_balance",,"Obveze zastupnika iz prodaje prema nalogodavcu","Obveze zastupnika iz prodaje prema nalogodavcu"
"9970","kp_rrif9970","l10n_hr_chart_template_rrif","off_balance",,"Obveze za čekove i mjenice za osiguranje otplate anuiteta za robne i financijske kredite","Obveze za čekove i mjenice za osiguranje otplate anuiteta za robne i financijske kredite"
"9971","kp_rrif9971","l10n_hr_chart_template_rrif","off_balance",,"Jamstva od dužnika kao instrument plaćanja","Jamstva od dužnika kao instrument plaćanja"
"9972","kp_rrif9972","l10n_hr_chart_template_rrif","off_balance",,"Obveze za primljene zadužnice","Obveze za primljene zadužnice"
"9973","kp_rrif9973","l10n_hr_chart_template_rrif","off_balance",,"Ostali vrijednosni papiri koji nisu stavljeni u optjecaj","Ostali vrijednosni papiri koji nisu stavljeni u optjecaj"
"9974","kp_rrif9974","l10n_hr_chart_template_rrif","off_balance",,"Obveze za izdane zadužnice","Obveze za izdane zadužnice"
"9975","kp_rrif9975","l10n_hr_chart_template_rrif","off_balance",,"Obveze za izdane mjenice","Obveze za izdane mjenice"
"9976","kp_rrif9976","l10n_hr_chart_template_rrif","off_balance",,"Obveze za korištene garancije u tijeku","Obveze za korištene garancije u tijeku"
"9980","kp_rrif9980","l10n_hr_chart_template_rrif","off_balance",,"Obveznice i druge vrijednosnice","Obveznice i druge vrijednosnice"
"9981","kp_rrif9981","l10n_hr_chart_template_rrif","off_balance",,"Vrijednosni papiri (obveznice, dionice)","Vrijednosni papiri (obveznice, dionice)"
"9982","kp_rrif9982","l10n_hr_chart_template_rrif","off_balance",,"Vrijednost zalihe blokova ulaznica","Vrijednost zalihe blokova ulaznica"
"9983","kp_rrif9983","l10n_hr_chart_template_rrif","off_balance",,"Prodajna mjesta za izdane obveznice","Prodajna mjesta za izdane obveznice"
"9987","kp_rrif9987","l10n_hr_chart_template_rrif","off_balance",,"Kamate sadržane u vrijednosnicama ili obračunima","Kamate sadržane u vrijednosnicama ili obračunima"
"9990","kp_rrif9990","l10n_hr_chart_template_rrif","off_balance",,"Obveze prema ulagačima","Obveze prema ulagačima"
"9991","kp_rrif9991","l10n_hr_chart_template_rrif","off_balance",,"Izvori prihoda - priljeva","Izvori prihoda - priljeva"
"9992","kp_rrif9992","l10n_hr_chart_template_rrif","off_balance",,"Ukalkulirani - planirani dobitak","Ukalkulirani - planirani dobitak"
"9993","kp_rrif9993","l10n_hr_chart_template_rrif","off_balance",,"Obveze za porez i druga javna davanja","Obveze za porez i druga javna davanja"
"9994","kp_rrif9994","l10n_hr_chart_template_rrif","off_balance",,"Obveze iz planiranog čistog dobitka","Obveze iz planiranog čistog dobitka"

```

## File: data\account.tax.group.csv

```csv
id,name,country_id/id
tax_group_0,PDV 0%,base.hr
tax_group_10,PDV 10%,base.hr
tax_group_25,PDV 25%,base.hr
tax_group_30,PDV 30%,base.hr
tax_group_70,PDV 70%,base.hr

```

## File: data\account_chart_tag_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <!-- Account Chart Template -->
        <record id="l10n_hr_chart_template_rrif" model="account.chart.template">
            <field name="property_account_receivable_id"    ref="kp_rrif1200"/>
            <field name="property_account_payable_id"       ref="kp_rrif2200"/>
            <field name="property_account_expense_categ_id" ref="kp_rrif4199"/>
            <field name="property_account_income_categ_id"  ref="kp_rrif7500"/>
            <field name="income_currency_exchange_account_id" ref="kp_rrif1050"/>
            <field name="expense_currency_exchange_account_id" ref="kp_rrif4754"/>
            <field name="default_pos_receivable_account_id" ref="kp_rrif1213" />
        </record>

</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_hr.l10n_hr_chart_template_rrif')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_fiscal_position_data.xml

```xml
<?xml version="1.0"  encoding="utf-8"?>
<odoo>
		<record id="account_fiscal_position_template_rrif_domairpartneri0" model="account.fiscal.position.template">
			<field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
			<field name="name">R1 partneri</field>
		</record>

		<record id="account_fiscal_position_template_rrif_2" model="account.fiscal.position.template">
			<field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
			<field name="name">R2 partneri</field>
		</record>

		<record id="account_fiscal_position_template_rrif_3" model="account.fiscal.position.template">
			<field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
			<field name="name">EU Inozemni</field>
		</record>

		<record id="account_fiscal_position_template_rrif_4" model="account.fiscal.position.template">
			<field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
			<field name="name">Inozemni</field>
		</record>


       <!-- R2 Partneri -->
		
		<record id="account_fiscal_position_tax_template_rrif_3" model="account.fiscal.position.tax.template">
			<field name="position_id" ref="account_fiscal_position_template_rrif_2"/>
			<field name="tax_src_id" ref="rrif_pp_25"/>
			<field name="tax_dest_id" ref="rrif_ppr2_25"/> 
		</record>

		<record id="account_fiscal_position_tax_template_rrif_3" model="account.fiscal.position.tax.template">
			<field name="position_id" ref="account_fiscal_position_template_rrif_2"/>
			<field name="tax_src_id" ref="rrif_pp_25usl"/>
			<field name="tax_dest_id" ref="rrif_ppr2_25"/>
		</record>

		<record id="account_fiscal_position_account_template_rrif_1" model="account.fiscal.position.account.template">
			<field name="position_id" ref="account_fiscal_position_template_rrif_2"/>
			<field name="account_dest_id" ref="kp_rrif2220"/>
			<field name="account_src_id" ref="kp_rrif2200"/>
		</record>


       <!-- INO Partneri -->

		<record id="account_fiscal_position_account_template_rrif_2" model="account.fiscal.position.account.template">
			<field name="position_id" ref="account_fiscal_position_template_rrif_3"/>
			<field name="account_dest_id" ref="kp_rrif1210"/>
			<field name="account_src_id" ref="kp_rrif1200"/>
		</record>
		<record id="account_fiscal_position_account_template_rrif_3" model="account.fiscal.position.account.template">
			<field name="position_id" ref="account_fiscal_position_template_rrif_3"/>
			<field name="account_dest_id" ref="kp_rrif2210"/>
			<field name="account_src_id" ref="kp_rrif2200"/>
		</record>

		<record id="account_fiscal_position_tax_template_rrif_4" model="account.fiscal.position.tax.template">
			<field name="position_id" ref="account_fiscal_position_template_rrif_3"/>
			<field name="tax_src_id" ref="rrif_pdv_25"/>
			<field name="tax_dest_id" ref="rrif_pdv_osl_izvoz_0"/>
		</record>
		<record id="account_fiscal_position_tax_template_rrif_5" model="account.fiscal.position.tax.template">
			<field name="position_id" ref="account_fiscal_position_template_rrif_3"/>
			<field name="tax_src_id" ref="rrif_pdv_25usl"/>
			<field name="tax_dest_id" ref="rrif_pdv_osl_izvoz_0"/>
		</record>
		
		<record id="account_fiscal_position_tax_template_rrif_6" model="account.fiscal.position.tax.template">
			<field name="position_id" ref="account_fiscal_position_template_rrif_3"/>
			<field name="tax_src_id" ref="rrif_pp_25"/>
			<field name="tax_dest_id" ref="rrif_pp_uvoz_0"/>
		</record>
		<record id="account_fiscal_position_tax_template_rrif_7" model="account.fiscal.position.tax.template">
			<field name="position_id" ref="account_fiscal_position_template_rrif_3"/>
			<field name="tax_src_id" ref="rrif_pp_25usl"/>
			<field name="tax_dest_id" ref="rrif_pp_uvoz_samopdv_25usl"/>
		</record>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.hr"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="account_tax_report_line_ostali_porezi" model="account.report.line">
                <field name="name">Ostali porezi, carine, trošarine i sl.</field>
                <field name="aggregation_formula">POREZ_NA_POTROSNJU.balance</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_porez" model="account.report.line">
                        <field name="name">Porez na potrošnju</field>
                        <field name="code">POREZ_NA_POTROSNJU</field>
                        <field name="expression_ids">
                            <record id="account_tax_report_line_porez_formula" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">external</field>
                                <field name="formula">sum</field>
                                <field name="subformula">editable;rounding=2</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="account_tax_report_line_pdv" model="account.report.line">
                <field name="name">PDV</field>
                <field name="aggregation_formula">(((koje.balance + (izvozne.balance + isporuke.balance + tuzemne.balance + ostale.balance) + isporuke_po.balance) + (izdani_10.balance + izdani_25.balance)) + (pretporez_10.balance + (0.7 * pretporez_25_70.balance + 0.3 * pretporez_25_30.balance) + placeni_uvozu.balance + placeni_usluge_10.balance + placeni_usluge_25.balance + pretporez_0.balance)) + (((izdani_racuni.balance + izdani_racuni_25.balance) + (pretporez_10_tax.balance + pretporez_25_tax.balance + placeni_uvozu_tax.balance + placeni_usluge_10_tax.balance + placeni_usluge_25_ta.balance))) + ((nepriznati_pretporez_25_other.balance + (0.3 * nepriznati_pretporez_25_30.balance) + (0.7 * nepriznati_pretporez_25_70.balance)) + pretporez_koji.balance + (nepriznati_pretporez_25_tax.balance) + (pretporez_koji_neplaceni.balance + pretporez_koji_neplaceni_25.balance + pretporez_koji_neplaceni_usluge_25.balance))</field>
                <field name="children_ids">
                    <record id="account_tax_report_line_obrazac_pdv" model="account.report.line">
                        <field name="name">OBRAZAC PDV</field>
                        <field name="aggregation_formula">(((koje.balance + (izvozne.balance + isporuke.balance + tuzemne.balance + ostale.balance) + isporuke_po.balance) + (izdani_10.balance + izdani_25.balance)) + (pretporez_10.balance + (0.7 * pretporez_25_70.balance + 0.3 * pretporez_25_30.balance) + placeni_uvozu.balance + placeni_usluge_10.balance + placeni_usluge_25.balance + pretporez_0.balance)) + (((izdani_racuni.balance + izdani_racuni_25.balance) + (pretporez_10_tax.balance + pretporez_25_tax.balance + placeni_uvozu_tax.balance + placeni_usluge_10_tax.balance + placeni_usluge_25_ta.balance)))</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_osnovica" model="account.report.line">
                                <field name="name">O S N O V I C A</field>
                                <field name="aggregation_formula">((koje.balance + (izvozne.balance + isporuke.balance + tuzemne.balance + ostale.balance) + isporuke_po.balance) + (izdani_10.balance + izdani_25.balance)) + (pretporez_10.balance + (0.7 * pretporez_25_70.balance + 0.3 * pretporez_25_30.balance) + placeni_uvozu.balance + placeni_usluge_10.balance + placeni_usluge_25.balance + pretporez_0.balance)</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_obracun" model="account.report.line">
                                        <field name="name">Obračun isporuka (I+II)</field>
                                        <field name="aggregation_formula">OBRACUN_I_ISPORUKE_NE_PODLIJEZU__OSLOBODENE.balance + OBRACUN_II_OPOREZIVE_ISPORUKE.balance</field>
                                        <field name="children_ids">
                                            <record id="account_tax_report_line_obracun_isporuke" model="account.report.line">
                                                <field name="name">I. Isporuke ne podliježu / oslobođene</field>
                                                <field name="code">OBRACUN_I_ISPORUKE_NE_PODLIJEZU__OSLOBODENE</field>
                                                <field name="aggregation_formula">koje.balance + I2_OSLOBODENE_UKUPNO.balance + isporuke_po.balance</field>
                                                <field name="children_ids">
                                                    <record id="account_tax_report_line_koje" model="account.report.line">
                                                        <field name="name">I.1. Koje ne podliježu oporezivanju</field>
                                                        <field name="code">koje</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_koje_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">I.1. Koje ne podliježu oporezivanju</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_oslobodene" model="account.report.line">
                                                        <field name="name">I.2. Oslobođene ukupno</field>
                                                        <field name="code">I2_OSLOBODENE_UKUPNO</field>
                                                        <field name="aggregation_formula">izvozne.balance + isporuke.balance + tuzemne.balance + ostale.balance</field>
                                                        <field name="children_ids">
                                                            <record id="account_tax_report_line_izvozne" model="account.report.line">
                                                                <field name="name">I.2.1. Izvozne</field>
                                                                <field name="code">izvozne</field>
                                                                <field name="expression_ids">
                                                                    <record id="account_tax_report_line_izvozne_tag" model="account.report.expression">
                                                                        <field name="label">balance</field>
                                                                        <field name="engine">tax_tags</field>
                                                                        <field name="formula">I.2.1. Izvozne</field>
                                                                    </record>
                                                                </field>
                                                            </record>
                                                            <record id="account_tax_report_line_isporuke" model="account.report.line">
                                                                <field name="name">I.2.2. Isporuke dobara</field>
                                                                <field name="code">isporuke</field>
                                                                <field name="expression_ids">
                                                                    <record id="account_tax_report_line_isporuke_tag" model="account.report.expression">
                                                                        <field name="label">balance</field>
                                                                        <field name="engine">tax_tags</field>
                                                                        <field name="formula">I.2.2. Isporuke dobara</field>
                                                                    </record>
                                                                </field>
                                                            </record>
                                                            <record id="account_tax_report_line_tuzemne" model="account.report.line">
                                                                <field name="name">I.2.3. Tuzemne - bez prava odbitka</field>
                                                                <field name="code">tuzemne</field>
                                                                <field name="expression_ids">
                                                                    <record id="account_tax_report_line_tuzemne_tag" model="account.report.expression">
                                                                        <field name="label">balance</field>
                                                                        <field name="engine">tax_tags</field>
                                                                        <field name="formula">I.2.3. Tuzemne  - bez prava odbitka</field>
                                                                    </record>
                                                                </field>
                                                            </record>
                                                            <record id="account_tax_report_line_ostale" model="account.report.line">
                                                                <field name="name">I.2.4. Ostale – s pravom odbitka</field>
                                                                <field name="code">ostale</field>
                                                                <field name="expression_ids">
                                                                    <record id="account_tax_report_line_ostale_tag" model="account.report.expression">
                                                                        <field name="label">balance</field>
                                                                        <field name="engine">tax_tags</field>
                                                                        <field name="formula">I.2.4. Ostale – s pravom odbitka</field>
                                                                    </record>
                                                                </field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_isporuke_po" model="account.report.line">
                                                        <field name="name">I.3. Isporuke po stopi od 0%</field>
                                                        <field name="code">isporuke_po</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_isporuke_po_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">I.3. Isporuke po stopi od 0%</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_oporezive" model="account.report.line">
                                                <field name="name">II. Oporezive isporuke</field>
                                                <field name="code">OBRACUN_II_OPOREZIVE_ISPORUKE</field>
                                                <field name="aggregation_formula">izdani_10.balance + OBRACUN_II2_IZDANI_RACUNI_PO_STOPI_22_I_23.balance + izdani_25.balance + II4_NENAPLACENI_IZVOZ.balance + II5_OSLOBODENJE_IZVOZA__PUTNICKI_PROMET.balance</field>
                                                <field name="children_ids">
                                                    <record id="account_tax_report_line_izdani_10" model="account.report.line">
                                                        <field name="name">II.1 Izdani računi po stopi 10%</field>
                                                        <field name="code">izdani_10</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_izdani_10_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">II.1 Izdani računi po stopi 10% (osnovica)</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_izdani_22_i_23" model="account.report.line">
                                                        <field name="name">II.2 Izdani računi po stopi 22% i 23%</field>
                                                        <field name="code">OBRACUN_II2_IZDANI_RACUNI_PO_STOPI_22_I_23</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_izdani_22_i_23_formula" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">external</field>
                                                                <field name="formula">sum</field>
                                                                <field name="subformula">editable;rounding=2</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_izdani_25" model="account.report.line">
                                                        <field name="name">II.3 Izdani računi po stopi 25%</field>
                                                        <field name="code">izdani_25</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_izdani_25_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">II.3 Izdani računi po stopi 25% (osnovica)</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_nenaplaceni" model="account.report.line">
                                                        <field name="name">II.4 Nenaplaćeni izvoz</field>
                                                        <field name="code">II4_NENAPLACENI_IZVOZ</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_nenaplaceni_formula" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">external</field>
                                                                <field name="formula">sum</field>
                                                                <field name="subformula">editable;rounding=2</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_oslobodenje" model="account.report.line">
                                                        <field name="name">II.5 Oslobođenje izvoza – putnički promet</field>
                                                        <field name="code">II5_OSLOBODENJE_IZVOZA__PUTNICKI_PROMET</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_oslobodenje_formula" model="account.report.expression">
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
                                    <record id="account_tax_report_line_obracunani" model="account.report.line">
                                        <field name="name">III. OBRAČUNANI PRETPOREZ</field>
                                        <field name="aggregation_formula">pretporez_10.balance + (0.7 * pretporez_25_70.balance + 0.3 * pretporez_25_30.balance) + placeni_uvozu.balance + placeni_usluge_10.balance + placeni_usluge_25.balance + pretporez_0.balance</field>
                                        <field name="children_ids">
                                            <record id="account_tax_report_line_pretporez_10" model="account.report.line">
                                                <field name="name">III.1. Pretporez 10%</field>
                                                <field name="code">pretporez_10</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_pretporez_10_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">III.1. Pretporez 10% (osnovica)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_pretporez_22_i_23" model="account.report.line">
                                                <field name="name">III.2. Pretporez 22% i23%</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_pretporez_22_i_23_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">III.2. Pretporez 22% i23%</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_pretporez_25" model="account.report.line">
                                                <field name="name">III.3. Pretporez 25%</field>
                                                <field name="aggregation_formula">0.3 * pretporez_25_30.balance + 0.7 * pretporez_25_70.balance</field>
                                                <field name="children_ids">
                                                    <record id="account_tax_report_line_pretporez_25_30" model="account.report.line">
                                                        <field name="name">III.3. Pretporez 25% (30%)</field>
                                                        <field name="code">pretporez_25_30</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_pretporez_25_30_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">III.3. Pretporez 25% (30%)</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_pretporez_25_70" model="account.report.line">
                                                        <field name="name">III.3. Pretporez 25% (70%)</field>
                                                        <field name="code">pretporez_25_70</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_pretporez_25_70_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">III.3. Pretporez 25% (70%)</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_placeni_uvozu" model="account.report.line">
                                                <field name="name">III.4. Plaćeni PP pri uvozu</field>
                                                <field name="code">placeni_uvozu</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_placeni_uvozu_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">III.4. Plaćeni PP pri uvozu (osnovica)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_placeni_usluge_10" model="account.report.line">
                                                <field name="name">III.5. Plaćeni PP na ino usluge 10%</field>
                                                <field name="code">placeni_usluge_10</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_placeni_usluge_10_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">III.5. Plaćeni PP na ino usluge 10% (osnovica)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_placeni_usluge_22_i_23" model="account.report.line">
                                                <field name="name">III.6. Plaćeni PP na ino usluge 22% i 23%</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_placeni_usluge_22_i_23_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">III.6. Plaćeni PP na ino usluge 22% i 23%</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_placeni_usluge_25" model="account.report.line">
                                                <field name="name">III.7. Plaćeni PP na ino usluge 25%</field>
                                                <field name="code">placeni_usluge_25</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_placeni_usluge_25_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">III.7. Plaćeni PP na ino usluge 25% (osnovica)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_pretporez_0" model="account.report.line">
                                                <field name="name">III.0. Pretporez 0%</field>
                                                <field name="code">pretporez_0</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_pretporez_0_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">III.0. Pretporez 0%</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_pdv_porez" model="account.report.line">
                                <field name="name">PDV – POREZ – razlika za uplatu/preplata</field>
                                <field name="aggregation_formula">VI_POREZNA_OBVEZA_U_RAZDOBLJU.balance + UPLACENO_U_RAZDOBLJU.balance</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_porezna" model="account.report.line">
                                        <field name="name">VI. POREZNA OBVEZA U RAZDOBLJU</field>
                                        <field name="code">VI_POREZNA_OBVEZA_U_RAZDOBLJU</field>
                                        <field name="aggregation_formula">POREZNA_I_ISPORUKE_NE_PODLIJEZU__OSLOBODENE.balance + POREZNA_II_OPOREZIVE_ISPORUKE.balance + III_OBRACUNANI_PRETPOREZ.balance</field>
                                        <field name="children_ids">
                                            <record id="account_tax_report_line_isporuke_ne" model="account.report.line">
                                                <field name="name">I. Isporuke ne podliježu / oslobođene</field>
                                                <field name="code">POREZNA_I_ISPORUKE_NE_PODLIJEZU__OSLOBODENE</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_isporuke_ne_formula" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">external</field>
                                                        <field name="formula">sum</field>
                                                        <field name="subformula">editable;rounding=2</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_oporezive_isporuke" model="account.report.line">
                                                <field name="name">II. Oporezive isporuke</field>
                                                <field name="code">POREZNA_II_OPOREZIVE_ISPORUKE</field>
                                                <field name="aggregation_formula">izdani_racuni.balance + POREZNA_II2_IZDANI_RACUNI_PO_STOPI_22_I_23.balance + izdani_racuni_25.balance + II4_OSLOBODENJE_IZVOZA__PUTNICKI_PROMET.balance</field>
                                                <field name="children_ids">
                                                    <record id="account_tax_report_line_izdani_racuni" model="account.report.line">
                                                        <field name="name">II.1 Izdani računi po stopi 10%</field>
                                                        <field name="code">izdani_racuni</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_izdani_racuni_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">II.1 Izdani računi po stopi 10% (porez)</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_izdani_racuni_22_i_23" model="account.report.line">
                                                        <field name="name">II.2 Izdani računi po stopi 22% i 23%</field>
                                                        <field name="code">POREZNA_II2_IZDANI_RACUNI_PO_STOPI_22_I_23</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_izdani_racuni_22_i_23_formula" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">external</field>
                                                                <field name="formula">sum</field>
                                                                <field name="subformula">editable;rounding=2</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_izdani_racuni_25" model="account.report.line">
                                                        <field name="name">II.3 Izdani računi po stopi 25%</field>
                                                        <field name="code">izdani_racuni_25</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_izdani_racuni_25_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">II.3 Izdani računi po stopi 25% (porez)</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_oslobodenje_izvoza" model="account.report.line">
                                                        <field name="name">II.4 Oslobođenje izvoza – putnički promet</field>
                                                        <field name="code">II4_OSLOBODENJE_IZVOZA__PUTNICKI_PROMET</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_oslobodenje_izvoza_formula" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">external</field>
                                                                <field name="formula">sum</field>
                                                                <field name="subformula">editable;rounding=2</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_obracunani_pretporez" model="account.report.line">
                                                <field name="name">III. OBRAČUNANI PRETPOREZ</field>
                                                <field name="code">III_OBRACUNANI_PRETPOREZ</field>
                                                <field name="aggregation_formula">pretporez_10_tax.balance + III2_PRETPOREZ_22_I_23.balance + pretporez_25_tax.balance + placeni_uvozu_tax.balance + placeni_usluge_10_tax.balance + III6_PLACENI_PP_NA_INO_USLUGE_22_I_23.balance + placeni_usluge_25_ta.balance + III8_ISPRAVCI_PRETPOREZA.balance</field>
                                                <field name="children_ids">
                                                    <record id="account_tax_report_line_pretporez_10_tax" model="account.report.line">
                                                        <field name="name">III.1. Pretporez 10%</field>
                                                        <field name="code">pretporez_10_tax</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_pretporez_10_tax_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">III.1. Pretporez 10% (porez)</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_pretporez_22_i_23_tax" model="account.report.line">
                                                        <field name="name">III.2. Pretporez 22% i 23%</field>
                                                        <field name="code">III2_PRETPOREZ_22_I_23</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_pretporez_22_i_23_tax_formula" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">external</field>
                                                                <field name="formula">sum</field>
                                                                <field name="subformula">editable;rounding=2</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_pretporez_25_tax" model="account.report.line">
                                                        <field name="name">III.3. Pretporez 25%</field>
                                                        <field name="code">pretporez_25_tax</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_pretporez_25_tax_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">III.3. Pretporez 25%</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_placeni_uvozu_tax" model="account.report.line">
                                                        <field name="name">III.4. Plaćeni PP pri uvozu</field>
                                                        <field name="code">placeni_uvozu_tax</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_placeni_uvozu_tax_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">III.4. Plaćeni PP pri uvozu (porez)</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_placeni_usluge_10_tax" model="account.report.line">
                                                        <field name="name">III.5. Plaćeni PP na ino usluge 10%</field>
                                                        <field name="code">placeni_usluge_10_tax</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_placeni_usluge_10_tax_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">III.5. Plaćeni PP na ino usluge 10% (porez)</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_placeni_usluge_22_i_23_tax" model="account.report.line">
                                                        <field name="name">III.6. Plaćeni PP na ino usluge 22% i 23%</field>
                                                        <field name="code">III6_PLACENI_PP_NA_INO_USLUGE_22_I_23</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_placeni_usluge_22_i_23_tax_formula" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">external</field>
                                                                <field name="formula">sum</field>
                                                                <field name="subformula">editable;rounding=2</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_placeni_usluge_25_tax" model="account.report.line">
                                                        <field name="name">III.7. Plaćeni PP na ino usluge 25%</field>
                                                        <field name="code">placeni_usluge_25_ta</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_placeni_usluge_25_tax_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">III.7. Plaćeni PP na ino usluge 25% (porez)</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_ispravci" model="account.report.line">
                                                        <field name="name">III.8. Ispravci pretporeza</field>
                                                        <field name="code">III8_ISPRAVCI_PRETPOREZA</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_ispravci_formula" model="account.report.expression">
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
                                    <record id="account_tax_report_line_uplaceno" model="account.report.line">
                                        <field name="name">Uplaćeno u razdoblju</field>
                                        <field name="code">UPLACENO_U_RAZDOBLJU</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_uplaceno_formula" model="account.report.expression">
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
                    <record id="account_tax_report_line_pdv_ostalo" model="account.report.line">
                        <field name="name">PDV - OSTALO</field>
                        <field name="aggregation_formula">(nepriznati_pretporez_25_other.balance + (0.3 * nepriznati_pretporez_25_30.balance) + (0.7 * nepriznati_pretporez_25_70.balance)) + pretporez_koji.balance + (nepriznati_pretporez_25_tax.balance) + (pretporez_koji_neplaceni.balance + pretporez_koji_neplaceni_25.balance + pretporez_koji_neplaceni_usluge_25.balance)</field>
                        <field name="children_ids">
                            <record id="account_tax_report_line_ostali_podaci" model="account.report.line">
                                <field name="name">Ostali podaci</field>
                                <field name="aggregation_formula">1_ZA_ISPRAVAK_PRETPOREZA.balance + 2_OTUDENJESTJECANJE_GOSPODARSKE_CJELINE_ILI_POGONA.balance + 3_NABAVA_DOBARA_I_USLUGA_ZA_REPREZENTACIJU.balance + 4_NABAVA_OSOBNIH_VOZILA_I_DRUGIH_SREDSTAVA_ZA_OSOBNI_PRIJEVOZ.balance + 5_OSNOVICA_ZA_OBRACUN_VLASTITE_POTROSNJE_ZA_OSOBNA_VOZILA_NABAV.balance</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_ispravak" model="account.report.line">
                                        <field name="name">1. ZA ISPRAVAK PRETPOREZA</field>
                                        <field name="code">1_ZA_ISPRAVAK_PRETPOREZA</field>
                                        <field name="aggregation_formula">11_NABAVA_NEKRETNINA.balance + 12_PRODAJA_NEKRETNINA.balance + 13_NABAVA_OSOBNIH_VOZILA.balance + 14_PRODAJA_OSOBNIH_VOZILA.balance + 15_NABAVA_DUGOTRAJNE_IMOVINE.balance + 16_PRODAJA_DUGOTRAJNE_IMOVINE.balance</field>
                                        <field name="children_ids">
                                            <record id="account_tax_report_line_nabava" model="account.report.line">
                                                <field name="name">1.1. NABAVA NEKRETNINA</field>
                                                <field name="code">11_NABAVA_NEKRETNINA</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_nabava_formula" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">external</field>
                                                        <field name="formula">sum</field>
                                                        <field name="subformula">editable;rounding=2</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_prodaja" model="account.report.line">
                                                <field name="name">1.2. PRODAJA NEKRETNINA</field>
                                                <field name="code">12_PRODAJA_NEKRETNINA</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_prodaja_formula" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">external</field>
                                                        <field name="formula">sum</field>
                                                        <field name="subformula">editable;rounding=2</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_nabava_osobnih" model="account.report.line">
                                                <field name="name">1.3. NABAVA OSOBNIH VOZILA</field>
                                                <field name="code">13_NABAVA_OSOBNIH_VOZILA</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_nabava_osobnih_formula" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">external</field>
                                                        <field name="formula">sum</field>
                                                        <field name="subformula">editable;rounding=2</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_prodaja_osobnih" model="account.report.line">
                                                <field name="name">1.4. PRODAJA OSOBNIH VOZILA</field>
                                                <field name="code">14_PRODAJA_OSOBNIH_VOZILA</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_prodaja_osobnih_formula" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">external</field>
                                                        <field name="formula">sum</field>
                                                        <field name="subformula">editable;rounding=2</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_nabava_dugotrajne" model="account.report.line">
                                                <field name="name">1.5. NABAVA DUGOTRAJNE IMOVINE</field>
                                                <field name="code">15_NABAVA_DUGOTRAJNE_IMOVINE</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_nabava_dugotrajne_formula" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">external</field>
                                                        <field name="formula">sum</field>
                                                        <field name="subformula">editable;rounding=2</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_prodaja_dugotrajne" model="account.report.line">
                                                <field name="name">1.6. PRODAJA DUGOTRAJNE IMOVINE</field>
                                                <field name="code">16_PRODAJA_DUGOTRAJNE_IMOVINE</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_prodaja_dugotrajne_formula" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">external</field>
                                                        <field name="formula">sum</field>
                                                        <field name="subformula">editable;rounding=2</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_otudenje" model="account.report.line">
                                        <field name="name">2. OTUĐENJE/STJECANJE GOSPODARSKE CJELINE ILI POGONA</field>
                                        <field name="code">2_OTUDENJESTJECANJE_GOSPODARSKE_CJELINE_ILI_POGONA</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_otudenje_formula" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">external</field>
                                                <field name="formula">sum</field>
                                                <field name="subformula">editable;rounding=2</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_nabava_dobara" model="account.report.line">
                                        <field name="name">3. NABAVA DOBARA I USLUGA ZA REPREZENTACIJU</field>
                                        <field name="code">3_NABAVA_DOBARA_I_USLUGA_ZA_REPREZENTACIJU</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_nabava_dobara_formula" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">external</field>
                                                <field name="formula">sum</field>
                                                <field name="subformula">editable;rounding=2</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_nabava_osobnih_vozila" model="account.report.line">
                                        <field name="name">4. NABAVA OSOBNIH VOZILA I DRUGIH SREDSTAVA ZA OSOBNI PRIJEVOZ</field>
                                        <field name="code">4_NABAVA_OSOBNIH_VOZILA_I_DRUGIH_SREDSTAVA_ZA_OSOBNI_PRIJEVOZ</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_nabava_osobnih_vozila_formula" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">external</field>
                                                <field name="formula">sum</field>
                                                <field name="subformula">editable;rounding=2</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_osnovica_za_obracun" model="account.report.line">
                                        <field name="name">5. OSNOVICA ZA OBRAČUN VLASTITE POTROŠNJE ZA OSOBNA VOZILA NABAV</field>
                                        <field name="code">5_OSNOVICA_ZA_OBRACUN_VLASTITE_POTROSNJE_ZA_OSOBNA_VOZILA_NABAV</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_osnovica_za_obracun_formula" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">external</field>
                                                <field name="formula">sum</field>
                                                <field name="subformula">editable;rounding=2</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="account_tax_report_line_nepriznati_pretporez" model="account.report.line">
                                <field name="name">NEPRIZNATI PRETPOREZ</field>
                                <field name="aggregation_formula">0.3 * nepriznati_pretporez_25_30.balance + 0.7 * nepriznati_pretporez_25_70.balance + nepriznati_pretporez_25_other.balance + pretporez_koji.balance</field>
                                <field name="children_ids">
                                    <record id="account_tax_report_line_nepriznati_pretporez_osnovica_3070" model="account.report.line">
                                        <field name="name">NEPRIZNATI PRETPOREZ (osnovica 30 ili 70%)</field>
                                        <field name="aggregation_formula">0.3 * nepriznati_pretporez_25_30.balance + 0.7 * nepriznati_pretporez_25_70.balance + nepriznati_pretporez_25_other.balance</field>
                                        <field name="children_ids">
                                            <record id="account_tax_report_line_nepriznati_pretporez_10" model="account.report.line">
                                                <field name="name">Nepriznati pretporez 10% (o)</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_nepriznati_pretporez_10_formula" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">external</field>
                                                        <field name="formula">sum</field>
                                                        <field name="subformula">editable;rounding=2</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_nepriznati_pretporez_22_i_23" model="account.report.line">
                                                <field name="name">Nepriznati pretporez 22% i 23% (o)</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_nepriznati_pretporez_22_i_23_formula" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">external</field>
                                                        <field name="formula">sum</field>
                                                        <field name="subformula">editable;rounding=2</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_nepriznati_pretporez_25" model="account.report.line">
                                                <field name="name">Nepriznati pretporez 25% (o)</field>
                                                <field name="aggregation_formula">0.3 * nepriznati_pretporez_25_30.balance + 0.7 * nepriznati_pretporez_25_70.balance + nepriznati_pretporez_25_other.balance</field>
                                                <field name="children_ids">
                                                    <record id="account_tax_report_line_nepriznati_pretporez_25_30" model="account.report.line">
                                                        <field name="name">Nepriznati pretporez 25% (o) (30%)</field>
                                                        <field name="code">nepriznati_pretporez_25_30</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_nepriznati_pretporez_25_30_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">Nepriznati pretporez 25% (o) (30%)</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_nepriznati_pretporez_25_70" model="account.report.line">
                                                        <field name="name">Nepriznati pretporez 25% (o) (70%)</field>
                                                        <field name="code">nepriznati_pretporez_25_70</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_nepriznati_pretporez_25_70_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">Nepriznati pretporez 25% (o) (70%)</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_nepriznati_pretporez_25_other" model="account.report.line">
                                                        <field name="name">Nepriznati pretporez 25% (o)</field>
                                                        <field name="code">nepriznati_pretporez_25_other</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_nepriznati_pretporez_25_other_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">Nepriznati pretporez 25% (o)</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_pretporez_koji" model="account.report.line">
                                        <field name="name">Pretporez koji još nije priznan (uključivo i neplaćeni R-2)</field>
                                        <field name="code">pretporez_koji</field>
                                        <field name="expression_ids">
                                            <record id="account_tax_report_line_pretporez_koji_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Pretporez koji još nije priznan (uključivo i neplaćeni R-2)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_nepriznati_pretporez_porez" model="account.report.line">
                                        <field name="name">NEPRIZNATI PRETPOREZ (porez)</field>
                                        <field name="aggregation_formula">NEPRIZNATI_PRETPOREZ_30_ILI_70.balance</field>
                                        <field name="children_ids">
                                            <record id="account_tax_report_line_nepriznati_pretporez_3070" model="account.report.line">
                                                <field name="name">NEPRIZNATI PRETPOREZ (30 ili 70%)</field>
                                                <field name="code">NEPRIZNATI_PRETPOREZ_30_ILI_70</field>
                                                <field name="aggregation_formula">NEPRIZNATI_PRETPOREZ_10_P.balance + NEPRIZNATI_PRETPOREZ_22_I_23_P.balance + nepriznati_pretporez_25_tax.balance</field>
                                                <field name="children_ids">
                                                    <record id="account_tax_report_line_nepriznati_pretporez_10_tax" model="account.report.line">
                                                        <field name="name">Nepriznati pretporez 10% (p)</field>
                                                        <field name="code">NEPRIZNATI_PRETPOREZ_10_P</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_nepriznati_pretporez_10_tax_formula" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">external</field>
                                                                <field name="formula">sum</field>
                                                                <field name="subformula">editable;rounding=2</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_nepriznati_pretporez_22_i_23_tax" model="account.report.line">
                                                        <field name="name">Nepriznati pretporez 22% i 23% (p)</field>
                                                        <field name="code">NEPRIZNATI_PRETPOREZ_22_I_23_P</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_nepriznati_pretporez_22_i_23_tax_formula" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">external</field>
                                                                <field name="formula">sum</field>
                                                                <field name="subformula">editable;rounding=2</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                    <record id="account_tax_report_line_nepriznati_pretporez_25_tax" model="account.report.line">
                                                        <field name="name">Nepriznati pretporez 25% (p)</field>
                                                        <field name="code">nepriznati_pretporez_25_tax</field>
                                                        <field name="expression_ids">
                                                            <record id="account_tax_report_line_nepriznati_pretporez_25_tax_tag" model="account.report.expression">
                                                                <field name="label">balance</field>
                                                                <field name="engine">tax_tags</field>
                                                                <field name="formula">Nepriznati pretporez 25% (p)</field>
                                                            </record>
                                                        </field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="account_tax_report_line_pretporez_koji_ukupno" model="account.report.line">
                                        <field name="name">Pretporez koji još nije priznan (ukupno)</field>
                                        <field name="aggregation_formula">pretporez_koji_neplaceni.balance + pretporez_koji_neplaceni_25.balance + pretporez_koji_neplaceni_usluge_25.balance + PRETPOREZ_KOJI_JOS_NIJE_PRIZNAN_NEPLACENE_INO_USLUGE_10.balance</field>
                                        <field name="children_ids">
                                            <record id="account_tax_report_line_pretporez_koji_neplaceni" model="account.report.line">
                                                <field name="name">Pretporez koji još nije priznan (neplaćeni R2)</field>
                                                <field name="code">pretporez_koji_neplaceni</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_pretporez_koji_neplaceni_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">Pretporez koji još nije priznan (neplaćeni R2)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_pretporez_koji_neplaceni_25" model="account.report.line">
                                                <field name="name">Pretporez koji još nije priznan (neplaćeni UVOZ dobara 25%)</field>
                                                <field name="code">pretporez_koji_neplaceni_25</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_pretporez_koji_neplaceni_25_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">Pretporez koji još nije priznan (neplaćeni UVOZ dobara 25%)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_pretporez_koji_neplaceni_usluge_25" model="account.report.line">
                                                <field name="name">Pretporez koji još nije priznan (neplaćene INO usluge 25%)</field>
                                                <field name="code">pretporez_koji_neplaceni_usluge_25</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_pretporez_koji_neplaceni_usluge_25_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">Pretporez koji još nije priznan (neplaćene INO usluge 25%)</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="account_tax_report_line_pretporez_koji_neplaceni_usluge_10" model="account.report.line">
                                                <field name="name">Pretporez koji još nije priznan (neplaćene INO usluge 10%)</field>
                                                <field name="code">PRETPOREZ_KOJI_JOS_NIJE_PRIZNAN_NEPLACENE_INO_USLUGE_10</field>
                                                <field name="expression_ids">
                                                    <record id="account_tax_report_line_pretporez_koji_neplaceni_usluge_10_formula" model="account.report.expression">
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
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="rrif_pdv_25" model="account.tax.template">
        <field name="description">PDV 25%</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">25% PDV</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="10"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_25_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif24003'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_racuni_25_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_25_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif24003'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_racuni_25_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pdv_25usl" model="account.tax.template">
        <field name="description">PDV 25% Usluge</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">25% PDV usluge</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_25_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif24003'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_racuni_25_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_25_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif24003'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_racuni_25_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pdv_10" model="account.tax.template">
        <field name="description">PDV 10%</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">10% PDV</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif24000'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_racuni_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif24000'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_racuni_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pdv_0" model="account.tax.template">
        <field name="description">PDV  0%</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">0% PDV</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_isporuke_po_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_isporuke_po_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="rrif_pdv_avans_25" model="account.tax.template">
        <field name="description">PDV za predujam 25%</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">25% PDV (za predujam)</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_25_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif24013'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_racuni_25_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_25_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif24013'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_racuni_25_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pdv_avans_10" model="account.tax.template">
        <field name="description">PDV za predujam 10%</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">10% PDV (za predujam)</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif24010'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_racuni_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif24010'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_racuni_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pdv_avans_0" model="account.tax.template">
        <field name="description">PDV za predujam 0%</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">0% PDV (za predujam)</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_isporuke_po_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_isporuke_po_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="rrif_pdv_nezar_isp_25" model="account.tax.template">
        <field name="description">PDV po nezaračunanim isporukama 25%</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">25% PDV za nezaračunane isp.</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_25_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif24033'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_racuni_25_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_25_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif24033'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_racuni_25_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pdv_nezar_isp_10" model="account.tax.template">
        <field name="description">PDV po nezaračunanim isporukama 10%</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">10% PDV za nezaračunane isp.</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif24030'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_racuni_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif24030'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izdani_racuni_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pdv_nezar_isp_0" model="account.tax.template">
        <field name="description">PDV po nezaračunanim isporukama 0%</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">0% PDV za nezaračunane isp.</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_isporuke_po_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_isporuke_po_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="rrif_pdv_nepod_0" model="account.tax.template">
        <field name="description">1. KOJE NE PODLIJEŽU OPOREZIVANJU (čl. 2. u svezi s čl. 5 i čl. 8 st. 7 Zakona)</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">0% Ne podliježe op.</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_koje_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_koje_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="rrif_pdv_osl_izvoz_0" model="account.tax.template">
        <field name="description">2.1. IZVOZNE - s pravom na odbitak pretporeza (čl. 13. st. 1. toč. 1. i čl. 14. Zakona)</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">0% osl. izvozne</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izvozne_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_izvozne_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="rrif_pdv_osl_medpri_0" model="account.tax.template">
        <field name="description">2.2. U VEZI S MEĐUNARODNIM PRIJEVOZOM  (čl. 13.b Zakona)</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">0% osl. međ. prijevoz</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_isporuke_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_isporuke_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="rrif_pdv_osl_tuz_0" model="account.tax.template">
        <field name="description">2.3. TUZEMNE - bez prava na odbitak pretporeza (čl. 11. i čl. 11a Zakona)</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">0% osl tuzemne</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_tuzemne_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_tuzemne_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="rrif_pdv_osl_ost_0" model="account.tax.template">
        <field name="description">2.4. OSTALO (čl. 13. st. 1. toč. 2. I čl. 13a Zakona) </field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">0% osl. ostalo</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_ostale_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_ostale_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="rrif_pp_25" model="account.tax.template">
        <field name="description">Pretporez 25% PDV</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">25% PDV pretporez</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="10"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14003'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_25_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14003'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_25_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pp_25usl" model="account.tax.template">
        <field name="description">Pretporez 25% PDV Usluge</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">25% PDV pretporez usluge</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14003'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_25_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14003'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_25_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pp_10" model="account.tax.template">
        <field name="description">Pretporez 10% PDV</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">10% PDV pretporez</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="10"/>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14000'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_10_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14000'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_10_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pp_0" model="account.tax.template">
        <field name="description">Pretporez 0% PDV</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">0% PDV pretporez</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_0_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_0_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="rrif_pp_avans_25" model="account.tax.template">
        <field name="description">Pretporez za predujam 25% PDV</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">25% PDV pretporez za predujam</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14013'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_25_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14013'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_25_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pp_avans_0" model="account.tax.template">
        <field name="description">Pretporez za predujam 0% PDV</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">0% PDV pretporez za predujam</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_0_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_0_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="rrif_pp_uvoz_25" model="account.tax.template">
        <field name="description">Plaćeni PDV 25% pri uvozu dobara</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">25% uvoz dobara</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_uvozu_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14023'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_uvozu_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_uvozu_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14023'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_uvozu_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pp_uvoz_10" model="account.tax.template">
        <field name="description">Plaćeni PDV 10% pri uvozu dobara</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">10% uvoz dobara</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="10"/>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_uvozu_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14020'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_uvozu_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_uvozu_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14020'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_uvozu_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pp_uvoz_0" model="account.tax.template">
        <field name="description">Plaćeni PDV 0% pri uvozu dobara</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">0% uvoz dobara</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_uvozu_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_uvozu_tag')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="rrif_pp_ino_25" model="account.tax.template">
        <field name="description">Plaćeni PDV 25% na usluge inozemnih poduzetnika</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">25% ino. usluge</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_usluge_25_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14033'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_usluge_25_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_uvozu_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14033'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_usluge_25_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pp_ino_10" model="account.tax.template">
        <field name="description">Plaćeni PDV 10% na usluge inozemnih poduzetnika</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">10% ino. usluge</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_usluge_10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14030'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_usluge_10_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_uvozu_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14030'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_placeni_usluge_10_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_ppr2_25" model="account.tax.template">
        <field name="description">25% R-2 dobavljač</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">25% R-2 dobavljač</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_koji_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif1409'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_koji_neplaceni_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_koji_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif1409'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_koji_neplaceni_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pp_uvoz_samopdv_25" model="account.tax.template">
        <field name="description">Samo PDV kod uvoza 25%</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">Samo PDV kod uvoza 25%</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_other_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif4199'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_koji_neplaceni_25_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_other_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif4199'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_koji_neplaceni_25_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_pp_uvoz_samopdv_25usl" model="account.tax.template">
        <field name="description">Samo PDV kod uvoza 25% usluge</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">Samo PDV kod uvoza 25% usluge</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_other_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif4199'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_koji_neplaceni_usluge_25_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_other_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif4199'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_koji_neplaceni_usluge_25_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_ppdnp1_3070_1" model="account.tax.template">
        <field name="description">pp 30% Nepriznat 70% Priznat</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">pp 30% Nepriznat 70% Priznat</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_25_70_tag'), ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_30_tag')],
            }),
            (0,0, {
                'factor_percent': 30,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif4199'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_tax_tag')],
            }),
            (0,0, {
                'factor_percent': 70,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14002'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_25_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_25_70_tag'), ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_30_tag')],
            }),
            (0,0, {
                'factor_percent': 30,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif4199'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_tax_tag')],
            }),
            (0,0, {
                'factor_percent': 70,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14002'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_25_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_ppdnp1_7030_1" model="account.tax.template">
        <field name="description">pp 70% Nepriznat 30% Priznat</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">pp 70% Nepriznat 30% Priznat</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_70_tag'), ref('l10n_hr.account_tax_report_line_pretporez_25_30_tag')],
            }),
            (0,0, {
                'factor_percent': 30,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14002'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_25_tax_tag')],
            }),
            (0,0, {
                'factor_percent': 70,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif4199'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_70_tag'), ref('l10n_hr.account_tax_report_line_pretporez_25_30_tag')],
            }),
            (0,0, {
                'factor_percent': 30,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif14002'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_25_tax_tag')],
            }),
            (0,0, {
                'factor_percent': 70,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif4199'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_ppdnp1_3070_1_r2" model="account.tax.template">
        <field name="description">R2 pp 30% Nepriznat 70% Priznat</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">R2 pp 30% Nepriznat 70% Priznat</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_25_70_tag'), ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_30_tag')],
            }),
            (0,0, {
                'factor_percent': 30,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif4199'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_koji_neplaceni_tag')],
            }),
            (0,0, {
                'factor_percent': 70,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif1408'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_25_70_tag'), ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_30_tag')],
            }),
            (0,0, {
                'factor_percent': 30,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif4199'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_koji_neplaceni_tag')],
            }),
            (0,0, {
                'factor_percent': 70,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif1408'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_tax_tag')],
            }),
        ]"/>
    </record>
    <record id="rrif_ppdnp1_7030_1_r2" model="account.tax.template">
        <field name="description">R2 pp 70% Nepriznat 30% Priznat</field>
        <field name="chart_template_id" ref="l10n_hr_chart_template_rrif"/>
        <field name="name">R2 pp 70% Nepriznat 30% Priznat</field>
        <field name="amount">25</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="sequence" eval="100"/>
        <field name="tax_group_id" ref="tax_group_25"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_70_tag'), ref('l10n_hr.account_tax_report_line_pretporez_25_30_tag')],
            }),
            (0,0, {
                'factor_percent': 30,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif1408'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_koji_neplaceni_tag')],
            }),
            (0,0, {
                'factor_percent': 70,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif4199'),
                'plus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_tax_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_70_tag'), ref('l10n_hr.account_tax_report_line_pretporez_25_30_tag')],
            }),
            (0,0, {
                'factor_percent': 30,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif1408'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_pretporez_koji_neplaceni_tag')],
            }),
            (0,0, {
                'factor_percent': 70,
                'repartition_type': 'tax',
                'account_id': ref('kp_rrif4199'),
                'minus_report_expression_ids': [ref('l10n_hr.account_tax_report_line_nepriznati_pretporez_25_tax_tag')],
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\l10n_hr_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <!-- Account Chart Template -->
        <record id="l10n_hr_chart_template_rrif" model="account.chart.template">
            <field name="name">RRIF-ov računski plan za poduzetnike</field>
            <field name="bank_account_code_prefix">101</field>
            <field name="cash_account_code_prefix">102</field>
            <field name="transfer_account_code_prefix">1009</field>
            <field name="currency_id" ref="base.HRK"/>
            <field name="code_digits">0</field>
            <field name="country_id" ref="base.hr"/>
            <field name="use_storno_accounting" eval="True"/>
        </record>

    </data>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="-0.41" y="6.65" width="61.9" height="33.53" maskUnits="userSpaceOnUse">
      <rect x="6.18" y="7.55" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="1000" height="500" transform="translate(-0.41 6.65) scale(0.06 0.07)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAA+gAAAIeCAYAAAAoHyVUAAAACXBIWXMAALLbAACy2wGBYJJrAAAgAElEQVR4Xuzdd5SkZZn+8e+bKofOPd09wyRmhjjkjCSRoOsK/sC0C4qCygroGhZZUXEVMaKyuKLrohJWRFFBEQRBJKPkYYYweXpC51hdVW+96ffH05hWGHUXLfD6nNOnm6G6QnedM3O99/3ct5VAgoiIiIiIiIj8Vdnbu4GIiIiIiIiIvPAU0EVERERERESagAK6iIiIiIiISBNQQBcRERERERFpAgroIiIiIiIiIk1AAV1ERERERESkCSigi4iIiIiIiDQBBXQRERERERGRJqCALiIiIiIiItIEFNBFREREREREmoACuoiIiIiIiEgTUEAXERERERERaQIK6CIiIiIiIiJNQAFdREREREREpAkooIuIiIiIiIg0AQV0ERERERERkSaggC4iIiIiIiLSBBTQRURERERERJqAArqIiIiIiIhIE1BAFxEREREREWkCCugiIiIiIiIiTUABXURERERERKQJKKCLiIiIiIiINAEFdBEREREREZEmoIAuIiIiIiIi0gQU0EVERERERESagAK6iIiIiIiISBNQQBcRERERERFpAgroIiIiIiIiIk1AAV1ERERERESkCSigi4iIiIiIiDQBBXQRERERERGRJqCALiIiIiIiItIEFNBFREREREREmoACuoiIiIiIiEgTUEAXERERERERaQIK6CIiIiIiIiJNQAFdREREREREpAkooIuIiIiIiIg0AQV0ERERERERkSaggC4iIiIiIiLSBBTQRURERERERJqAArqIiIiIiIhIE1BAFxEREREREWkCCugiIiIiIiIiTUABXURERERERKQJKKCLiIiIiIiINAEFdBEREREREZEmoIAuIiIiIiIi0gQU0EVERERERESagAK6iIiIiIiISBNQQBcRERERERFpAgroIiIiIiIiIk1AAV1ERERERESkCSigi4iIiIiIiDQBBXQRERERERGRJqCALiIiIiIiItIEFNBFREREREREmoACuoiIiIiIiEgTUEAXERERERERaQIK6CIiIiIiIiJNQAFdREREREREpAkooIuIiIiIiIg0AQV0ERERERERkSaggC4iIiIiIiLSBBTQRURERERERJqAArqIiIiIiIhIE1BAFxERERER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6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMAGBDgAAABMQ6AAAADABgQ4AAAATEOgAAAAwAYEOAAAAExDoAAAAMIE1yd//qRcBAAAA/7v+C12EZlnbJM+cAAAAAElFTkSuQmCC"/>
    </g>
  </g>
</svg>

```

