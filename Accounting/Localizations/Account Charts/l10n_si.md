# Odoo Module: l10n_si

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright: (C) 2012 - Mentis d.o.o., Dravograd

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright: (C) 2012 - Mentis d.o.o., Dravograd

{
    "name": "Slovenian - Accounting",
    "version": "1.1",
    "author": "Mentis d.o.o.",
    "website": "http://www.mentis.si",
    'category': 'Accounting/Localizations/Account Charts',
    "description": "Kontni načrt za gospodarske družbe",
    "depends": [
        "account",
        "base_iban",
    ],
    "data": [
        "data/l10n_si_chart_data.xml",
        "data/account.account.template.csv",
        "data/account.chart.template.csv",
        "data/account.tax.group.csv",
        "data/account_tax_report_data.xml",
        "data/account_tax_data.xml",
        "data/account.fiscal.position.template.csv",
        "data/account.fiscal.position.account.template.csv",
        "data/account.fiscal.position.tax.template.csv",
        "data/account_chart_template_data.xml",
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
code,id,user_type_id/id,name,reconcile,chart_template_id/id
000000,gd_acc_000000,account.data_account_type_current_assets,"DOBRO IME",False,gd_chart
002000,gd_acc_002000,account.data_account_type_current_assets,"ODLOŽENI STROŠKI RAZVIJANJA",False,gd_chart
003000,gd_acc_003000,account.data_account_type_current_assets,"PREMOŽENJSKE PRAVICE",False,gd_chart
005000,gd_acc_005000,account.data_account_type_current_assets,"DRUGA NEOPREDMETENA SREDSTVA (TUDI EMISIJSKI KUPONI)",False,gd_chart
007000,gd_acc_007000,account.data_account_type_current_assets,"DOLGOROČNE AKTIVNE ČASOVNE RAZMEJITVE",False,gd_chart
008000,gd_acc_008000,account.data_account_type_current_assets,"POPRAVEK VREDNOSTI NEOPREDMETENIH SREDSTEV ZARADI AMORTIZIRANJA",False,gd_chart
009000,gd_acc_009000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI NEOPREDMETENIH SREDSTEV",False,gd_chart
010000,gd_acc_010000,account.data_account_type_current_assets,"NALOŽBENE NEPREMIČNINE, VREDNOTENE PO MODELU NABAVNE VREDNOSTI",False,gd_chart
011000,gd_acc_011000,account.data_account_type_current_assets,"NALOŽBENE NEPREMIČNINE, VREDNOTENE PO MODELU POŠTENE VREDNOSTI",False,gd_chart
015000,gd_acc_015000,account.data_account_type_current_assets,"POPRAVEK VREDNOSTI NALOŽBENIH NEPREMIČNIN ZARADI AMORTIZIRANJA",False,gd_chart
019000,gd_acc_019000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI NALOŽBENIH NEPREMIČNIN",False,gd_chart
020000,gd_acc_020000,account.data_account_type_current_assets,"ZEMLJIŠČA, VREDNOTENA PO MODELU NABAVNE VREDNOSTI",False,gd_chart
021000,gd_acc_021000,account.data_account_type_current_assets,"ZGRADBE, VREDNOTENE PO MODELU NABAVNE VREDNOSTI",False,gd_chart
022000,gd_acc_022000,account.data_account_type_current_assets,"ZEMLJIŠČA, VREDNOTENA PO MODELU PREVREDNOTENJA",False,gd_chart
023000,gd_acc_023000,account.data_account_type_current_assets,"ZGRADBE, VREDNOTENE PO MODELU PREVREDNOTENJA",False,gd_chart
027000,gd_acc_027000,account.data_account_type_current_assets,"NEPREMIČNINE V GRADNJI OZIROMA IZDELAVI",False,gd_chart
031000,gd_acc_031000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI ZEMLJIŠČ",False,gd_chart
032000,gd_acc_032000,account.data_account_type_current_assets,"POPRAVEK VREDNOSTI ZEMLJIŠČ ZARADI AMORTIZIRANJA (KAMNOLOMI, ODLAGALIŠČA ODPADKOV)",False,gd_chart
035000,gd_acc_035000,account.data_account_type_current_assets,"POPRAVEK VREDNOSTI ZGRADB ZARADI AMORTIZIRANJA",False,gd_chart
039000,gd_acc_039000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI ZGRADB",False,gd_chart
040000,gd_acc_040000,account.data_account_type_current_assets,"OPREMA IN NADOMESTNI DELI, VREDNOTENI PO MODELU NABAVNE VREDNOSTI",False,gd_chart
041000,gd_acc_041000,account.data_account_type_current_assets,"DROBNI INVENTAR",False,gd_chart
042000,gd_acc_042000,account.data_account_type_current_assets,"OPREMA IN NADOMESTNI DELI, VREDNOTENI PO MODELU PREVREDNOTENJA",False,gd_chart
043000,gd_acc_043000,account.data_account_type_current_assets,"BIOLOŠKA SREDSTVA",False,gd_chart
044000,gd_acc_044000,account.data_account_type_current_assets,"VLAGANJA V OPREDMETENA OSNOVNA SREDSTVA V TUJI LASTI",False,gd_chart
045000,gd_acc_045000,account.data_account_type_current_assets,"DRUGA OPREDMETENA OSNOVNA SREDSTVA",False,gd_chart
047000,gd_acc_047000,account.data_account_type_current_assets,"OPREMA IN DRUGA OPREDMETENA OSNOVNA SREDSTVA V GRADNJI OZIROMA IZDELAVI",False,gd_chart
050000,gd_acc_050000,account.data_account_type_current_assets,"POPRAVEK VREDNOSTI OPREME IN NADOMESTNIH DELOV ZARADI AMORTIZIRANJA",False,gd_chart
051000,gd_acc_051000,account.data_account_type_current_assets,"POPRAVEK VREDNOSTI DROBNEGA INVENTARJA ZARADI AMORTIZIRANJA",False,gd_chart
052000,gd_acc_052000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI OPREME IN NADOMESTNIH DELOV",False,gd_chart
053000,gd_acc_053000,account.data_account_type_current_assets,"POPRAVEK VREDNOSTI BIOLOŠKIH SREDSTEV",False,gd_chart
054000,gd_acc_054000,account.data_account_type_current_assets,"POPRAVEK VREDNOSTI VLAGANJ V OPREDMETENA OSNOVNA SREDSTVA V TUJI LASTI",False,gd_chart
055000,gd_acc_055000,account.data_account_type_current_assets,"POPRAVEK VREDNOSTI DRUGIH OPREDMETENIH OSNOVNIH SREDSTEV ZARADI AMORTIZIRANJA",False,gd_chart
059000,gd_acc_059000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI DRUGIH OPREDMETENIH OSNOVNIH SREDSTEV",False,gd_chart
060000,gd_acc_060000,account.data_account_type_current_assets,"DOLGOROČNE FINANČNE NALOŽBE V DELNICE IN DELEŽE DRUŽB V SKUPINI, RAZPOREJENE IN IZMERJENE PO NABAVNI VREDNOSTI",False,gd_chart
061000,gd_acc_061000,account.data_account_type_current_assets,"DOLGOROČNE FINANČNE NALOŽBE V DELNICE IN DELEŽE DRUŽB V SKUPINI, RAZPOREJENE IN IZMERJENE PO POŠTENI VREDNOSTI PREK POSLOVNEGA IZIDA",False,gd_chart
062000,gd_acc_062000,account.data_account_type_current_assets,"DOLGOROČNE FINANČNE NALOŽBE V DELNICE IN DELEŽE DRUŽB V SKUPINI, RAZPOREJENE IN IZMERJENE PO POŠTENI VREDNOSTI PREK KAPITALA",False,gd_chart
063000,gd_acc_063000,account.data_account_type_current_assets,"DOLGOROČNE FINANČNE NALOŽBE V DELNICE IN DELEŽE PRIDRUŽENIH DRUŽB IN SKUPAJ OBVLADOVANIH DRUŽB, RAZPOREJENE IN IZMERJENE PO NABAVNI VREDNOSTI",False,gd_chart
064000,gd_acc_064000,account.data_account_type_current_assets,"DOLGOROČNE FINANČNE NALOŽBE V DELNICE IN DELEŽE PRIDRUŽENIH DRUŽB IN SKUPAJ OBVLADOVANIH DRUŽB, RAZPOREJENE IN IZMERJENE PO POŠTENI VREDNOSTI PREK POSLOVNEGA IZIDA",False,gd_chart
065000,gd_acc_065000,account.data_account_type_current_assets,"DOLGOROČNE FINANČNE NALOŽBE V DELNICE IN DELEŽE PRIDRUŽENIH DRUŽB IN SKUPAJ OBVLADOVANIH DRUŽB, RAZPOREJENE IN IZMERJENE PO POŠTENI VREDNOSTI PREK KAPITALA",False,gd_chart
066000,gd_acc_066000,account.data_account_type_current_assets,"DRUGE DOLGOROČNE FINANČNE NALOŽBE, RAZPOREJENE IN IZMERJENE PO NABAVNI VREDNOSTI",False,gd_chart
067000,gd_acc_067000,account.data_account_type_current_assets,"DRUGE DOLGOROČNE FINANČNE NALOŽBE, RAZPOREJENE IN IZMERJENE PO POŠTENI VREDNOSTI PREK POSLOVNEGA IZIDA",False,gd_chart
068000,gd_acc_068000,account.data_account_type_current_assets,"DRUGE DOLGOROČNE FINANČNE NALOŽBE, RAZPOREJENE IN IZMERJENE PO POŠTENI VREDNOSTI PREK KAPITALA",False,gd_chart
069000,gd_acc_069000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI DOLGOROČNIH FINANČNIH NALOŽB",False,gd_chart
070000,gd_acc_070000,account.data_account_type_current_assets,"DOLGOROČNA POSOJILA, DANA NA PODLAGI POSOJILNIH POGODB DRUŽBAM V SKUPINI, VKLJUČNO Z DOLGOROČNIMI TERJATVAMI IZ FINANČNEGA NAJEMA",False,gd_chart
071000,gd_acc_071000,account.data_account_type_current_assets,"DOLGOROČNA POSOJILA, DANA NA PODLAGI POSOJILNIH POGODB PRIDRUŽENIM DRUŽBAM IN SKUPAJ OBVLADOVANIM DRUŽBAM, VKLJUČNO Z DOLGOROČNIMI TERJATVAMI IZ FINANČNEGA NAJEMA",False,gd_chart
072000,gd_acc_072000,account.data_account_type_current_assets,"DOLGOROČNA POSOJILA, DANA DRUGIM, VKLJUČNO Z DOLGOROČNIMI TERJATVAMI IZ FINANČNEGA NAJEMA",False,gd_chart
073000,gd_acc_073000,account.data_account_type_current_assets,"DOLGOROČNA POSOJILA, DANA Z ODKUPOM OBVEZNIC OD DRUŽB V SKUPINI",False,gd_chart
074000,gd_acc_074000,account.data_account_type_current_assets,"DOLGOROČNA POSOJILA, DANA Z ODKUPOM OBVEZNIC OD PRIDRUŽENIH DRUŽB IN SKUPAJ OBVLADOVANIH DRUŽB",False,gd_chart
075000,gd_acc_075000,account.data_account_type_current_assets,"DOLGOROČNA POSOJILA, DANA Z ODKUPOM OBVEZNIC OD DRUGIH",False,gd_chart
076000,gd_acc_076000,account.data_account_type_current_assets,"DOLGOROČNE TERJATVE ZA NEVPLAČANI VPOKLICANI KAPITAL",False,gd_chart
077000,gd_acc_077000,account.data_account_type_current_assets,"DRUGA DOLGOROČNO VLOŽENA SREDSTVA",False,gd_chart
078000,gd_acc_078000,account.data_account_type_current_assets,"DANI DOLGOROČNI DEPOZITI",False,gd_chart
079000,gd_acc_079000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI DANIH DOLGOROČNIH POSOJIL",False,gd_chart
080000,gd_acc_080000,account.data_account_type_current_assets,"DOLGOROČNI BLAGOVNI KREDITI, DANI V DRŽAVI",False,gd_chart
081000,gd_acc_081000,account.data_account_type_current_assets,"DOLGOROČNI BLAGOVNI KREDITI, DANI V TUJINI",False,gd_chart
082000,gd_acc_082000,account.data_account_type_current_assets,"DANI DOLGOROČNI POTROŠNIŠKI KREDITI",False,gd_chart
083000,gd_acc_083000,account.data_account_type_current_assets,"DANI DOLGOROČNI PREDUJMI",False,gd_chart
084000,gd_acc_084000,account.data_account_type_current_assets,"DANE DOLGOROČNE VARŠČINE",False,gd_chart
086000,gd_acc_086000,account.data_account_type_current_assets,"DRUGE DOLGOROČNE POSLOVNE TERJATVE",False,gd_chart
089000,gd_acc_089000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI DOLGOROČNIH POSLOVNIH TERJATEV",False,gd_chart
090000,gd_acc_090000,account.data_account_type_current_assets,"TERJATVE ZA ODLOŽENI DAVEK IZ ODBITNIH ZAČASNIH RAZLIK",False,gd_chart
091000,gd_acc_091000,account.data_account_type_current_assets,"TERJATVE ZA ODLOŽENI DAVEK IZ NEIZRABLJENIH DAVČNIH IZGUB, PRENESENIH V NASLEDNJA DAVČNA OBDOBJA",False,gd_chart
092000,gd_acc_092000,account.data_account_type_current_assets,"TERJATVE ZA ODLOŽENI DAVEK IZ DAVČNIH DOBROPISOV, PRENESENIH V NASLEDNJA DAVČNA OBDOBJA",False,gd_chart
100000,gd_acc_100000,account.data_account_type_current_assets,"DENARNA SREDSTVA V BLAGAJNI, RAZEN DEVIZNIH SREDSTEV",False,gd_chart
101000,gd_acc_101000,account.data_account_type_current_assets,"DEVIZNA SREDSTVA V BLAGAJNI",False,gd_chart
102000,gd_acc_102000,account.data_account_type_current_assets,"IZDANI ČEKI (ODBITNA POSTAVKA)",False,gd_chart
103000,gd_acc_103000,account.data_account_type_current_assets,"PREJETI ČEKI",False,gd_chart
104000,gd_acc_104000,account.data_account_type_current_assets,"NETVEGANI TAKOJ UDENARLJIVI DOLŽNIŠKI VREDNOSTNI PAPIRJI",False,gd_chart
110000,gd_acc_110000,account.data_account_type_current_assets,"DENARNA SREDSTVA NA RAČUNIH, RAZEN DEVIZNIH",False,gd_chart
111000,gd_acc_111000,account.data_account_type_current_assets,"KRATKOROČNI DEPOZITI OZIROMA DEPOZITI NA ODPOKLIC, RAZEN DEVIZNIH",False,gd_chart
112000,gd_acc_112000,account.data_account_type_current_assets,"DEVIZNA SREDSTVA NA RAČUNIH",False,gd_chart
113000,gd_acc_113000,account.data_account_type_current_assets,"KRATKOROČNI DEVIZNI DEPOZITI OZIROMA DEVIZNI DEPOZITI NA ODPOKLIC",False,gd_chart
114000,gd_acc_114000,account.data_account_type_current_assets,"DENARNA SREDSTVA NA POSEBNIH RAČUNIH OZIROMA ZA POSEBNE NAMENE",False,gd_chart
120000,gd_acc_120000,account.data_account_type_receivable,"KRATKOROČNE TERJATVE DO KUPCEV V DRŽAVI",True,gd_chart
121000,gd_acc_121000,account.data_account_type_receivable,"KRATKOROČNE TERJATVE DO KUPCEV V TUJINI",True,gd_chart
122000,gd_acc_122000,account.data_account_type_current_assets,"KRATKOROČNI BLAGOVNI KREDITI, DANI KUPCEM V DRŽAVI",False,gd_chart
123000,gd_acc_123000,account.data_account_type_current_assets,"KRATKOROČNI BLAGOVNI KREDITI, DANI KUPCEM V TUJINI",False,gd_chart
124000,gd_acc_124000,account.data_account_type_current_assets,"KRATKOROČNI POTROŠNIŠKI KREDITI, DANI KUPCEM V DRŽAVI",False,gd_chart
125000,gd_acc_125000,account.data_account_type_receivable,"KRATKOROČNE TERJATVE DO KUPCEV V DRŽAVI (POS)",True,gd_chart
129000,gd_acc_129000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI KRATKOROČNIH TERJATEV DO KUPCEV",False,gd_chart
130000,gd_acc_130000,account.data_account_type_current_assets,"KRATKOROČNI PREDUJMI, DANI ZA OPREDMETENA OSNOVNA SREDSTVA",False,gd_chart
131000,gd_acc_131000,account.data_account_type_current_assets,"KRATKOROČNI PREDUJMI, DANI ZA NEOPREDMETENA SREDSTVA",False,gd_chart
132000,gd_acc_132000,account.data_account_type_current_assets,"KRATKOROČNI PREDUJMI, DANI ZA ZALOGE MATERIALA IN BLAGA TER ŠE NE OPRAVLJENE STORITVE",False,gd_chart
133000,gd_acc_133000,account.data_account_type_receivable,"DRUGI DANI KRATKOROČNI PREDUJMI IN PREPLAČILA",True,gd_chart
134000,gd_acc_134000,account.data_account_type_current_assets,"DANE KRATKOROČNE VARŠČINE",False,gd_chart
139000,gd_acc_139000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI DANIH KRATKOROČNIH PREDUJMOV IN VARŠČIN",False,gd_chart
140000,gd_acc_140000,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE DO IZVOZNIKOV",False,gd_chart
141000,gd_acc_141000,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE IZ UVOZA ZA TUJ RAČUN",False,gd_chart
142000,gd_acc_142000,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE IZ KOMISIJSKE IN KONSIGNACIJSKE PRODAJE",False,gd_chart
145000,gd_acc_145000,account.data_account_type_current_assets,"DRUGE KRATKOROČNE TERJATVE IZ POSLOVANJA ZA TUJ RAČUN",False,gd_chart
149000,gd_acc_149000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI KRATKOROČNIH TERJATEV IZ POSLOVANJA ZA TUJ RAČUN",False,gd_chart
150000,gd_acc_150000,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA OBRESTI",False,gd_chart
151000,gd_acc_151000,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA DIVIDENDE",False,gd_chart
152000,gd_acc_152000,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA DRUGE DELEŽE V DOBIČKU",False,gd_chart
155000,gd_acc_155000,account.data_account_type_current_assets,"DRUGE KRATKOROČNE TERJATVE, POVEZANE S FINANČNIMI PRIHODKI",False,gd_chart
159000,gd_acc_159000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI KRATKOROČNIH TERJATEV, POVEZANIH S FINANČNIMI PRIHODKI",False,gd_chart
160001,gd_acc_160001,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA NEODBITNI DDV",False,gd_chart
160002,gd_acc_160002,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA ODBITNI DDV 9,5%",False,gd_chart
160003,gd_acc_160003,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA ODBITNI DDV 22%",False,gd_chart
160004,gd_acc_160004,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA ODBITNI DDV 9,5% V EU",False,gd_chart
160005,gd_acc_160005,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA ODBITNI DDV 22% V EU",False,gd_chart
160006,gd_acc_160006,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA ODBITNI DDV 9,5% IZVEN EU",False,gd_chart
160007,gd_acc_160007,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA ODBITNI DDV 22% IZVEN EU",False,gd_chart
160008,gd_acc_160008,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA ODBITNI DDV 9,5% UVOZ",False,gd_chart
160009,gd_acc_160009,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA ODBITNI DDV 22% UVOZ",False,gd_chart
161000,gd_acc_161000,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA DAVEK OD DOHODKOV PRAVNIH OSEB, VKLJUČNO Z DAVKOM, PLAČANIM V TUJINI",False,gd_chart
162000,gd_acc_162000,account.data_account_type_current_assets,"DRUGE KRATKOROČNE TERJATVE DO DRŽAVNIH IN DRUGIH INŠTITUCIJ",False,gd_chart
165000,gd_acc_165000,account.data_account_type_current_assets,"OSTALE KRATKOROČNE TERJATVE",False,gd_chart
166000,gd_acc_166000,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA DDV, VRNJEN TUJCEM",False,gd_chart
167000,gd_acc_167000,account.data_account_type_current_assets,"KRATKOROČNE TERJATVE ZA DDV, PLAČAN V TUJINI",False,gd_chart
169000,gd_acc_169000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI DRUGIH KRATKOROČNIH TERJATEV",False,gd_chart
170000,gd_acc_170000,account.data_account_type_current_assets,"KRATKOROČNE FINANČNE NALOŽBE V DELNICE IN DELEŽE DRUŽB V SKUPINI, RAZPOREJENE IN IZMERJENE PO NABAVNI VREDNOSTI",False,gd_chart
171000,gd_acc_171000,account.data_account_type_current_assets,"KRATKOROČNE FINANČNE NALOŽBE V DELNICE IN DELEŽE DRUŽB V SKUPINI, RAZPOREJENE IN IZMERJENE PO POŠTENI VREDNOSTI PREK POSLOVNEGA IZIDA",False,gd_chart
172000,gd_acc_172000,account.data_account_type_current_assets,"KRATKOROČNE FINANČNE NALOŽBE V DELNICE IN DELEŽE DRUŽB V SKUPINI, RAZPOREJENE IN IZMERJENE PO POŠTENI VREDNOSTI PREK KAPITALA",False,gd_chart
173000,gd_acc_173000,account.data_account_type_current_assets,"KRATKOROČNE FINANČNE NALOŽBE V DELNICE IN DELEŽE PRIDRUŽENIH DRUŽB IN SKUPAJ OBVLADOVANIH DRUŽB, RAZPOREJENE IN IZMERJENE PO NABAVNI VREDNOSTI",False,gd_chart
174000,gd_acc_174000,account.data_account_type_current_assets,"KRATKOROČNE FINANČNE NALOŽBE V DELNICE IN DELEŽE PRIDRUŽENIH DRUŽB IN SKUPAJ OBVLADOVANIH DRUŽB, RAZPOREJENE IN IZMERJENE PO POŠTENI VREDNOSTI PREK POSLOVNEGA IZIDA",False,gd_chart
175000,gd_acc_175000,account.data_account_type_current_assets,"KRATKOROČNE FINANČNE NALOŽBE V DELNICE IN DELEŽE PRIDRUŽENIH DRUŽB IN SKUPAJ OBVLADOVANIH DRUŽB, RAZPOREJENE IN IZMERJENE PO POŠTENI VREDNOSTI PREK KAPITALA",False,gd_chart
176000,gd_acc_176000,account.data_account_type_current_assets,"DRUGE KRATKOROČNE FINANČNE NALOŽBE, RAZPOREJENE IN IZMERJENE PO NABAVNI VREDNOSTI",False,gd_chart
177000,gd_acc_177000,account.data_account_type_current_assets,"DRUGE KRATKOROČNE FINANČNE NALOŽBE, RAZPOREJENE IN IZMERJENE PO POŠTENI VREDNOSTI PREK POSLOVNEGA IZIDA",False,gd_chart
178000,gd_acc_178000,account.data_account_type_current_assets,"DRUGE KRATKOROČNE FINANČNE NALOŽBE, RAZPOREJENE IN IZMERJENE PO POŠTENI VREDNOSTI PREK KAPITALA",False,gd_chart
179000,gd_acc_179000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI KRATKOROČNIH FINANČNIH NALOŽB",False,gd_chart
180000,gd_acc_180000,account.data_account_type_current_assets,"KRATKOROČNA POSOJILA, DANA NA PODLAGI POSOJILNIH POGODB DRUŽBAM V SKUPINI",False,gd_chart
181000,gd_acc_181000,account.data_account_type_current_assets,"KRATKOROČNA POSOJILA, DANA NA PODLAGI POSOJILNIH POGODB PRIDRUŽENIM DRUŽBAM IN SKUPAJ OBVLADOVANIM DRUŽBAM",False,gd_chart
182000,gd_acc_182000,account.data_account_type_current_assets,"KRATKOROČNA POSOJILA, DANA DRUGIM",False,gd_chart
183000,gd_acc_183000,account.data_account_type_current_assets,"KRATKOROČNI DEPOZITI V BANKAH IN DRUGIH FINANČNIH ORGANIZACIJAH",False,gd_chart
184000,gd_acc_184000,account.data_account_type_current_assets,"KRATKOROČNO DANA POSOJILA Z ODKUPOM OBVEZNIC",False,gd_chart
185000,gd_acc_185000,account.data_account_type_current_assets,"PREJETE MENICE",False,gd_chart
186000,gd_acc_186000,account.data_account_type_current_assets,"KRATKOROČNO DANA POSOJILA Z ODKUPOM DRUGIH DOLŽNIŠKIH VREDNOSTNIH PAPIRJEV",False,gd_chart
187000,gd_acc_187000,account.data_account_type_current_assets,"KRATKOROČNO NEVPLAČANI VPOKLICANI KAPITAL",False,gd_chart
189000,gd_acc_189000,account.data_account_type_current_assets,"OSLABITEV VREDNOSTI KRATKOROČNIH POSOJIL",False,gd_chart
190000,gd_acc_190000,account.data_account_type_current_assets,"KRATKOROČNO ODLOŽENI STROŠKI OZIROMA ODHODKI",False,gd_chart
191000,gd_acc_191000,account.data_account_type_current_assets,"KRATKOROČNO NEZARAČUNANI PRIHODKI",False,gd_chart
192000,gd_acc_192000,account.data_account_type_current_assets,"VREDNOTNICE",False,gd_chart
195000,gd_acc_195000,account.data_account_type_current_assets,"DDV OD PREJETIH PREDUJMOV",False,gd_chart
210000,gd_acc_210000,account.data_account_type_current_liabilities,"OBVEZNOSTI, VKLJUČENE V SKUPINE ZA ODTUJITEV",False,gd_chart
220000,gd_acc_220000,account.data_account_type_payable,"KRATKOROČNE OBVEZNOSTI (DOLGOVI) DO DOBAVITELJEV V DRŽAVI",True,gd_chart
221000,gd_acc_221000,account.data_account_type_payable,"KRATKOROČNE OBVEZNOSTI (DOLGOVI) DO DOBAVITELJEV V TUJINI",True,gd_chart
222000,gd_acc_222000,account.data_account_type_current_liabilities,"KRATKOROČNI BLAGOVNI KREDITI, PREJETI V DRŽAVI",False,gd_chart
223000,gd_acc_223000,account.data_account_type_current_liabilities,"KRATKOROČNI BLAGOVNI KREDITI, PREJETI V TUJINI",False,gd_chart
224000,gd_acc_224000,account.data_account_type_current_liabilities,"KRATKOROČNE OBVEZNOSTI (DOLGOVI) ZA NEZARAČUNANE BLAGO IN STORITVE",False,gd_chart
230000,gd_acc_230000,account.data_account_type_payable,"PREJETI KRATKOROČNI PREDUJMI",True,gd_chart
231000,gd_acc_231000,account.data_account_type_current_liabilities,"PREJETE KRATKOROČNE VARŠČINE",False,gd_chart
240000,gd_acc_240000,account.data_account_type_current_liabilities,"KRATKOROČNE OBVEZNOSTI IZ IZVOZA ZA TUJ RAČUN",False,gd_chart
241000,gd_acc_241000,account.data_account_type_current_liabilities,"KRATKOROČNE OBVEZNOSTI DO UVOZNIKOV",False,gd_chart
242000,gd_acc_242000,account.data_account_type_current_liabilities,"KRATKOROČNE OBVEZNOSTI IZ KOMISIJSKE IN KONSIGNACIJSKE PRODAJE",False,gd_chart
245000,gd_acc_245000,account.data_account_type_current_liabilities,"DRUGE KRATKOROČNE OBVEZNOSTI IZ POSLOVANJA ZA TUJ RAČUN",False,gd_chart
250000,gd_acc_250000,account.data_account_type_current_liabilities,"KRATKOROČNE OBVEZNOSTI ZA VRAČUNANE IN NEOBRAČUNANE PLAČE",False,gd_chart
251000,gd_acc_251000,account.data_account_type_current_liabilities,"KRATKOROČNE OBVEZNOSTI ZA ČISTE PLAČE IN NADOMESTILA PLAČ",False,gd_chart
253000,gd_acc_253000,account.data_account_type_current_liabilities,"KRATKOROČNE OBVEZNOSTI ZA PRISPEVKE IZ KOSMATIH PLAČ IN NADOMESTIL PLAČ",False,gd_chart
254000,gd_acc_254000,account.data_account_type_current_liabilities,"KRATKOROČNE OBVEZNOSTI ZA DAVKE IZ KOSMATIH PLAČ IN NADOMESTIL PLAČ",False,gd_chart
255000,gd_acc_255000,account.data_account_type_current_liabilities,"KRATKOROČNE OBVEZNOSTI ZA DRUGE PREJEMKE IZ DELOVNEGA RAZMERJA",False,gd_chart
256000,gd_acc_256000,account.data_account_type_current_liabilities,"KRATKOROČNE OBVEZNOSTI ZA PRISPEVKE IZ DRUGIH PREJEMKOV IZ DELOVNEGA RAZMERJA, KI SE NE OBRAČUNAVAJO SKUPAJ S PLAČAMI",False,gd_chart
257000,gd_acc_257000,account.data_account_type_current_liabilities,"KRATKOROČNE OBVEZNOSTI ZA DAVEK IZ DRUGIH PREJEMKOV IZ DELOVNEGA RAZMERJA, KI SE NE OBRAČUNAVAJO SKUPAJ S PLAČAMI",False,gd_chart
260001,gd_acc_260001,account.data_account_type_current_liabilities,"OBVEZNOSTI ZA ZARAČUNANI DDV 9,5%",False,gd_chart
260002,gd_acc_260002,account.data_account_type_current_liabilities,"OBVEZNOSTI ZA ZARAČUNANI DDV 22%",False,gd_chart
260003,gd_acc_260003,account.data_account_type_current_liabilities,"OBVEZNOSTI ZA ZARAČUNANI DDV 9,5% V EU",False,gd_chart
260004,gd_acc_260004,account.data_account_type_current_liabilities,"OBVEZNOSTI ZA ZARAČUNANI DDV 22% V EU",False,gd_chart
260005,gd_acc_260005,account.data_account_type_current_liabilities,"OBVEZNOSTI ZA ZARAČUNANI DDV 9,5% IZVEN EU",False,gd_chart
260006,gd_acc_260006,account.data_account_type_current_liabilities,"OBVEZNOSTI ZA ZARAČUNANI DDV 22% IZVEN EU",False,gd_chart
261000,gd_acc_261000,account.data_account_type_current_liabilities,"OBVEZNOSTI ZA DDV, CARINO IN DRUGE DAJATVE OD UVOŽENEGA BLAGA",False,gd_chart
262000,gd_acc_262000,account.data_account_type_current_liabilities,"OBVEZNOSTI ZA PRISPEVKE IZPLAČEVALCA",False,gd_chart
263000,gd_acc_263000,account.data_account_type_current_liabilities,"OBVEZNOSTI ZA DAVEK OD IZPLAČANIH PLAČ",False,gd_chart
264000,gd_acc_264000,account.data_account_type_current_liabilities,"OBVEZNOSTI ZA DAVEK OD DOHODKOV",False,gd_chart
265000,gd_acc_265000,account.data_account_type_current_liabilities,"OBVEZNOSTI ZA DAVČNI ODTEGLJAJ",False,gd_chart
266000,gd_acc_266000,account.data_account_type_current_liabilities,"DRUGE KRATKOROČNE OBVEZNOSTI DO DRŽAVNIH IN DRUGIH INŠTITUCIJ",False,gd_chart
270000,gd_acc_270000,account.data_account_type_current_liabilities,"KRATKOROČNA POSOJILA, DOBLJENA PRI DRUŽBAH V SKUPINI",False,gd_chart
271000,gd_acc_271000,account.data_account_type_current_liabilities,"KRATKOROČNA POSOJILA, DOBLJENA PRI PRIDRUŽENIH DRUŽBAH IN SKUPAJ OBVLADOVANIH DRUŽBAH",False,gd_chart
272000,gd_acc_272000,account.data_account_type_current_liabilities,"KRATKOROČNA POSOJILA, DOBLJENA PRI BANKAH IN DRUŽBAH V DRŽAVI",False,gd_chart
273000,gd_acc_273000,account.data_account_type_current_liabilities,"KRATKOROČNA POSOJILA, DOBLJENA PRI BANKAH IN DRUŽBAH V TUJINI",False,gd_chart
274000,gd_acc_274000,account.data_account_type_current_liabilities,"KRATKOROČNE FINANČNE OBVEZNOSTI V ZVEZI Z OBVEZNICAMI",False,gd_chart
275000,gd_acc_275000,account.data_account_type_current_liabilities,"KRATKOROČNE OBVEZNOSTI V ZVEZI Z RAZDELITVIJO POSLOVNEGA IZIDA",False,gd_chart
276000,gd_acc_276000,account.data_account_type_current_liabilities,"KRATKOROČNE FINANČNE OBVEZNOSTI DO FIZIČNIH OSEB",False,gd_chart
278000,gd_acc_278000,account.data_account_type_current_liabilities,"OBVEZNOSTI IZ VPLAČILA KAPITALA DO VPISA V SODNI REGISTER",False,gd_chart
279000,gd_acc_279000,account.data_account_type_current_liabilities,"DRUGE KRATKOROČNE FINANČNE OBVEZNOSTI",False,gd_chart
280000,gd_acc_280000,account.data_account_type_current_liabilities,"KRATKOROČNE OBVEZNOSTI ZA OBRESTI",False,gd_chart
281000,gd_acc_281000,account.data_account_type_current_liabilities,"KRATKOROČNE MENIČNE OBVEZNOSTI",False,gd_chart
282000,gd_acc_282000,account.data_account_type_current_liabilities,"KRATKOROČNE OBVEZNOSTI V ZVEZI Z ODTEGLJAJI OD PLAČ IN NADOMESTIL PLAČ ZAPOSLENCEM",False,gd_chart
285000,gd_acc_285000,account.data_account_type_current_liabilities,"OSTALE KRATKOROČNE POSLOVNE OBVEZNOSTI",False,gd_chart
290000,gd_acc_290000,account.data_account_type_current_liabilities,"VNAPREJ VRAČUNANI STROŠKI OZIROMA ODHODKI",False,gd_chart
291000,gd_acc_291000,account.data_account_type_current_liabilities,"KRATKOROČNO ODLOŽENI PRIHODKI",False,gd_chart
295000,gd_acc_295000,account.data_account_type_current_liabilities,"DDV OD DANIH PREDUJMOV",False,gd_chart
300000,gd_acc_300000,account.data_account_type_current_assets,"VREDNOST SUROVIN IN MATERIALA PO OBRAČUNIH DOBAVITELJEV",False,gd_chart
301000,gd_acc_301000,account.data_account_type_current_assets,"ODVISNI STROŠKI NABAVE SUROVIN IN MATERIALA",False,gd_chart
302000,gd_acc_302000,account.data_account_type_current_assets,"CARINA IN DRUGE UVOZNE DAVŠČINE OD SUROVIN IN MATERIALA",False,gd_chart
303000,gd_acc_303000,account.data_account_type_current_assets,"DDV IN DRUGE DAVŠČINE OD SUROVIN IN MATERIALA",False,gd_chart
309000,gd_acc_309000,account.data_account_type_current_assets,"OBRAČUN NABAVE SUROVIN IN MATERIALA",False,gd_chart
310000,gd_acc_310000,account.data_account_type_current_assets,"ZALOGE SUROVIN IN MATERIALA V SKLADIŠČU",False,gd_chart
312000,gd_acc_312000,account.data_account_type_current_assets,"ZALOGE SUROVIN IN MATERIALA NA POTI",False,gd_chart
316000,gd_acc_316000,account.data_account_type_current_assets,"ZALOGE SUROVIN IN MATERIALA V DODELAVI IN PREDELAVI",False,gd_chart
319000,gd_acc_319000,account.data_account_type_current_assets,"ODMIKI OD CEN ZALOG SUROVIN IN MATERIALA",False,gd_chart
320000,gd_acc_320000,account.data_account_type_current_assets,"ZALOGE DROBNEGA INVENTARJA IN EMBALAŽE V SKLADIŠČU",False,gd_chart
321000,gd_acc_321000,account.data_account_type_current_assets,"ZALOGE DROBNEGA INVENTARJA IN EMBALAŽE, DANE V UPORABO",False,gd_chart
329000,gd_acc_329000,account.data_account_type_current_assets,"ODMIKI OD CEN DROBNEGA INVENTARJA IN EMBALAŽE",False,gd_chart
400000,gd_acc_400000,account.data_account_type_expenses,"STROŠKI MATERIALA",False,gd_chart
401000,gd_acc_401000,account.data_account_type_expenses,"STROŠKI POMOŽNEGA MATERIALA",False,gd_chart
402000,gd_acc_402000,account.data_account_type_expenses,"STROŠKI ENERGIJE",False,gd_chart
403000,gd_acc_403000,account.data_account_type_expenses,"STROŠKI NADOMESTNIH DELOV ZA OSNOVNA SREDSTVA IN MATERIALA ZA VZDRŽEVANJE OSNOVNIH SREDSTEV",False,gd_chart
404000,gd_acc_404000,account.data_account_type_expenses,"ODPIS DROBNEGA INVENTARJA IN EMBALAŽE",False,gd_chart
405000,gd_acc_405000,account.data_account_type_expenses,"USKLADITEV STROŠKOV MATERIALA IN DROBNEGA INVENTARJA ZARADI UGOTOVLJENIH POPISNIH RAZLIK",False,gd_chart
406000,gd_acc_406000,account.data_account_type_expenses,"STROŠKI PISARNIŠKEGA MATERIALA IN STROKOVNE LITERATURE",False,gd_chart
407000,gd_acc_407000,account.data_account_type_expenses,"DRUGI STROŠKI MATERIALA",False,gd_chart
410000,gd_acc_410000,account.data_account_type_expenses,"STROŠKI STORITEV PRI USTVARJANJU PROIZVODOV IN OPRAVLJANJU STORITEV",False,gd_chart
411000,gd_acc_411000,account.data_account_type_expenses,"STROŠKI TRANSPORTNIH STORITEV",False,gd_chart
412000,gd_acc_412000,account.data_account_type_expenses,"STROŠKI STORITEV V ZVEZI Z VZDRŽEVANJEM",False,gd_chart
413000,gd_acc_413000,account.data_account_type_expenses,"NAJEMNINE",False,gd_chart
414000,gd_acc_414000,account.data_account_type_expenses,"POVRAČILA STROŠKOV ZAPOSLENCEM V ZVEZI Z DELOM",False,gd_chart
415000,gd_acc_415000,account.data_account_type_expenses,"STROŠKI PLAČILNEGA PROMETA, STROŠKI BANČNIH STORITEV, STROŠKI POSLOV IN ZAVAROVALNE PREMIJE",False,gd_chart
416000,gd_acc_416000,account.data_account_type_expenses,"STROŠKI INTELEKTUALNIH IN OSEBNIH STORITEV",False,gd_chart
417000,gd_acc_417000,account.data_account_type_expenses,"STROŠKI SEJMOV, REKLAME IN REPREZENTANCE",False,gd_chart
418000,gd_acc_418000,account.data_account_type_expenses,"STROŠKI STORITEV FIZIČNIH OSEB, KI NE OPRAVLJAJO DEJAVNOSTI, SKUPAJ Z DAJATVAMI, KI BREMENIJO PODJETJE (STROŠKI PO POGODBAH O DELU, AVTORSKIH POGODBAH, SEJNINE ZAPOSLENCEM IN DRUGIM OSEBAM …)",False,gd_chart
419000,gd_acc_419000,account.data_account_type_expenses,"STROŠKI DRUGIH STORITEV",False,gd_chart
430000,gd_acc_430000,account.data_account_type_expenses,"AMORTIZACIJA NEOPREDMETENIH SREDSTEV",False,gd_chart
431000,gd_acc_431000,account.data_account_type_expenses,"AMORTIZACIJA ZGRADB",False,gd_chart
432000,gd_acc_432000,account.data_account_type_expenses,"AMORTIZACIJA OPREME IN NADOMESTNIH DELOV",False,gd_chart
433000,gd_acc_433000,account.data_account_type_expenses,"AMORTIZACIJA DROBNEGA INVENTARJA",False,gd_chart
434000,gd_acc_434000,account.data_account_type_expenses,"AMORTIZACIJA DRUGIH OPREDMETENIH OSNOVNIH SREDSTEV",False,gd_chart
435000,gd_acc_435000,account.data_account_type_expenses,"AMORTIZACIJA NALOŽBENIH NEPREMIČNIN",False,gd_chart
440000,gd_acc_440000,account.data_account_type_expenses,"REZERVACIJE ZA STROŠKE REORGANIZACIJE PODJETJA",False,gd_chart
441000,gd_acc_441000,account.data_account_type_expenses,"REZERVACIJE ZA DANA JAMSTVA",False,gd_chart
442000,gd_acc_442000,account.data_account_type_expenses,"REZERVACIJE ZA KOČLJIVE POGODBE",False,gd_chart
443000,gd_acc_443000,account.data_account_type_expenses,"REZERVACIJE ZA POKOJNINE, JUBILEJNE NAGRADE IN ODPRAVNINE OB UPOKOJITVI",False,gd_chart
449000,gd_acc_449000,account.data_account_type_expenses,"REZERVACIJE ZA POKRIVANJE DRUGIH OBVEZNOSTI IZ PRETEKLEGA POSLOVANJA",False,gd_chart
450000,gd_acc_450000,account.data_account_type_expenses,"STROŠKI OBRESTI",False,gd_chart
470000,gd_acc_470000,account.data_account_type_expenses,"PLAČE ZAPOSLENCEV",False,gd_chart
471000,gd_acc_471000,account.data_account_type_expenses,"NADOMESTILA PLAČ ZAPOSLENCEV",False,gd_chart
472000,gd_acc_472000,account.data_account_type_expenses,"STROŠKI DODATNEGA POKOJNINSKEGA ZAVAROVANJA ZAPOSLENCEV",False,gd_chart
473000,gd_acc_473000,account.data_account_type_expenses,"REGRES ZA LETNI DOPUST, BONITETE, POVRAČILA (ZA PREVOZ NA DELO IN Z NJEGA, ZA PREHRANO, ZA LOČENO ŽIVLJENJE) IN DRUGI PREJEMKI ZAPOSLENCEV",False,gd_chart
474000,gd_acc_474000,account.data_account_type_expenses,"DELODAJALČEVI PRISPEVKI OD PLAČ, NADOMESTIL PLAČ, BONITET, POVRAČIL IN DRUGIH PREJEMKOV ZAPOSLENCEV",False,gd_chart
475000,gd_acc_475000,account.data_account_type_expenses,"DRUGE DELODAJALČEVE DAJATVE OD PLAČ, NADOMESTIL PLAČ, BONITET, POVRAČIL IN DRUGIH PREJEMKOV ZAPOSLENCEV",False,gd_chart
476000,gd_acc_476000,account.data_account_type_expenses,"NAGRADE VAJENCEM SKUPAJ Z DAJATVAMI, KI BREMENIJO PODJETJE",False,gd_chart
480000,gd_acc_480000,account.data_account_type_expenses,"DAJATVE, KI NISO ODVISNE OD STROŠKOV DELA ALI DRUGIH VRST STROŠKOV",False,gd_chart
481000,gd_acc_481000,account.data_account_type_expenses,"IZDATKI ZA VARSTVO OKOLJA",False,gd_chart
482000,gd_acc_482000,account.data_account_type_expenses,"NAGRADE DIJAKOM IN ŠTUDENTOM NA DELOVNI PRAKSI SKUPAJ Z DAJATVAMI",False,gd_chart
483000,gd_acc_483000,account.data_account_type_expenses,"ŠTIPENDIJE DIJAKOM IN ŠTUDENTOM",False,gd_chart
484000,gd_acc_484000,account.data_account_type_expenses,"Foreigh izguba deviznega",False,gd_chart
489000,gd_acc_489000,account.data_account_type_expenses,"OSTALI STROŠKI",False,gd_chart
490000,gd_acc_490000,account.data_account_type_expenses,"PRENOS STROŠKOV V ZALOGE",False,gd_chart
491000,gd_acc_491000,account.data_account_type_expenses,"PRENOS STROŠKOV NEPOSREDNO V ODHODKE",False,gd_chart
600000,gd_acc_600000,account.data_account_type_current_assets,"NEDOKONČANA PROIZVODNJA",False,gd_chart
601000,gd_acc_601000,account.data_account_type_current_assets,"NEDOKONČANE STORITVE",False,gd_chart
602000,gd_acc_602000,account.data_account_type_current_assets,"POLIZDELKI",False,gd_chart
604000,gd_acc_604000,account.data_account_type_current_assets,"PROIZVODNJA V DODELAVI IN PREDELAVI",False,gd_chart
609000,gd_acc_609000,account.data_account_type_current_assets,"ODMIKI OD CEN NEDOKONČANIH PROIZVODNJE IN STORITEV",False,gd_chart
630000,gd_acc_630000,account.data_account_type_current_assets,"PROIZVODI V LASTNEM SKLADIŠČU",False,gd_chart
631000,gd_acc_631000,account.data_account_type_current_assets,"PROIZVODI V TUJEM SKLADIŠČU",False,gd_chart
632000,gd_acc_632000,account.data_account_type_current_assets,"PROIZVODI NA POTI",False,gd_chart
633000,gd_acc_633000,account.data_account_type_current_assets,"PROIZVODI V LASTNI PRODAJALNI",False,gd_chart
634000,gd_acc_634000,account.data_account_type_current_assets,"VRAČUNANI DDV OD PROIZVODOV V PRODAJALNI",False,gd_chart
635000,gd_acc_635000,account.data_account_type_current_assets,"PROIZVODI V DODELAVI IN PREDELAVI",False,gd_chart
639000,gd_acc_639000,account.data_account_type_current_assets,"ODMIKI OD CEN PROIZVODOV",False,gd_chart
650000,gd_acc_650000,account.data_account_type_current_assets,"VREDNOST BLAGA PO OBRAČUNIH DOBAVITELJEV",False,gd_chart
651000,gd_acc_651000,account.data_account_type_current_assets,"ODVISNI STROŠKI NABAVE BLAGA",False,gd_chart
659000,gd_acc_659000,account.data_account_type_current_assets,"OBRAČUN NABAVE BLAGA",False,gd_chart
660000,gd_acc_660000,account.data_account_type_current_assets,"BLAGO V LASTNEM SKLADIŠČU",False,gd_chart
661000,gd_acc_661000,account.data_account_type_current_assets,"BLAGO V TUJEM SKLADIŠČU",False,gd_chart
662000,gd_acc_662000,account.data_account_type_current_assets,"BLAGO NA POTI",False,gd_chart
663000,gd_acc_663000,account.data_account_type_current_assets,"BLAGO V LASTNI PRODAJALNI",False,gd_chart
664000,gd_acc_664000,account.data_account_type_current_assets,"DDV, VRAČUNAN V ZALOGAH BLAGA",False,gd_chart
669000,gd_acc_669000,account.data_account_type_current_assets,"VRAČUNANA RAZLIKA V CENAH ZALOG BLAGA",False,gd_chart
670000,gd_acc_670000,account.data_account_type_current_assets,"OPREDMETENA OSNOVNA SREDSTVA, NAMENJENA PRODAJI",False,gd_chart
671000,gd_acc_671000,account.data_account_type_current_assets,"NALOŽBENE NEPREMIČNINE, VREDNOTENE PO MODELU NABAVNE VREDNOSTI, NAMENJENE PRODAJI",False,gd_chart
672000,gd_acc_672000,account.data_account_type_current_assets,"DRUGA NEKRATKOROČNA SREDSTVA, NAMENJENA PRODAJI",False,gd_chart
673000,gd_acc_673000,account.data_account_type_current_assets,"SREDSTVA DELA DENAR USTVARJAJOČE ENOTE, NAMENJENA PRODAJI",False,gd_chart
674000,gd_acc_674000,account.data_account_type_current_assets,"SREDSTVA DENAR USTVARJAJOČE ENOTE, NAMENJENA PRODAJI",False,gd_chart
700000,gd_acc_700000,account.data_account_type_expenses,"VREDNOST PRODANIH POSLOVNIH UČINKOV",False,gd_chart
701000,gd_acc_701000,account.data_account_type_expenses,"VREDNOST USREDSTVENIH LASTNIH PROIZVODOV IN STORITEV",False,gd_chart
702000,gd_acc_702000,account.data_account_type_expenses,"NABAVNA VREDNOST PRODANIH MATERIALA IN BLAGA",False,gd_chart
703000,gd_acc_703000,account.data_account_type_expenses,"DRUGI POSLOVNI ODHODKI",False,gd_chart
710000,gd_acc_710000,account.data_account_type_expenses,"VREDNOST PRODANIH POSLOVNIH UČINKOV",False,gd_chart
711000,gd_acc_711000,account.data_account_type_expenses,"NABAVNA VREDNOST PRODANIH MATERIALA IN BLAGA",False,gd_chart
712000,gd_acc_712000,account.data_account_type_expenses,"STROŠKI PRODAJANJA",False,gd_chart
713000,gd_acc_713000,account.data_account_type_expenses,"STROŠKI SPLOŠNIH DEJAVNOSTI (NABAVE IN UPRAVE)",False,gd_chart
714000,gd_acc_714000,account.data_account_type_expenses,"DRUGI STROŠKI, KI SE NE ZADRŽUJEJO V ZALOGAH",False,gd_chart
720000,gd_acc_720000,account.data_account_type_expenses,"PREVREDNOTOVALNI POSLOVNI ODHODKI V ZVEZI Z NEOPREDMETENIMI SREDSTVI, OPREDMETENIMI OSNOVNIMI SREDSTVI IN NALOŽBENIMI NEPREMIČNINAMI RAZPOREJENIMI IN IZMERJENIMI PO MODELU NABAVNE VREDNOSTI",False,gd_chart
721000,gd_acc_721000,account.data_account_type_expenses,"PREVREDNOTOVALNI POSLOVNI ODHODKI V ZVEZI S KRATKOROČNIMI SREDSTVI, RAZEN S FINANČNIMI NALOŽBAMI",False,gd_chart
722000,gd_acc_722000,account.data_account_type_expenses,"PREVREDNOTOVALNI POSLOVNI ODHODKI V ZVEZI S STROŠKI DELA",False,gd_chart
740000,gd_acc_740000,account.data_account_type_expenses,"ODHODKI IZ POSOJIL, PREJETIH OD DRUŽB V SKUPINI",False,gd_chart
741000,gd_acc_741000,account.data_account_type_expenses,"ODHODKI IZ POSOJIL, PREJETIH OD BANK",False,gd_chart
742000,gd_acc_742000,account.data_account_type_expenses,"ODHODKI IZ IZDANIH OBVEZNIC",False,gd_chart
743000,gd_acc_743000,account.data_account_type_expenses,"ODHODKI IZ DRUGIH FINANČNIH OBVEZNOSTI",False,gd_chart
744000,gd_acc_744000,account.data_account_type_expenses,"ODHODKI IZ POSLOVNIH OBVEZNOSTI DO DRUŽB V SKUPINI",False,gd_chart
745000,gd_acc_745000,account.data_account_type_expenses,"ODHODKI IZ OBVEZNOSTI DO DOBAVITELJEV IN MENIČNIH OBVEZNOSTI",False,gd_chart
746000,gd_acc_746000,account.data_account_type_expenses,"ODHODKI IZ DRUGIH POSLOVNIH OBVEZNOSTI",False,gd_chart
747000,gd_acc_747000,account.data_account_type_expenses,"ODHODKI IZ SREDSTEV, RAZPOREJENIH PO POŠTENI VREDNOSTI PREK POSLOVNEGA IZIDA",False,gd_chart
748000,gd_acc_748000,account.data_account_type_expenses,"ODHODKI IZ OSLABITVE FINANČNIH NALOŽB",False,gd_chart
749000,gd_acc_749000,account.data_account_type_expenses,"ODHODKI IZ ODPRAVE PRIPOZNANJA FINANČNIH NALOŽB",False,gd_chart
750000,gd_acc_750000,account.data_account_type_expenses,"ODHODKI IZ VREDNOTENJA NALOŽBENIH NEPREMIČNIN PO MODELU POŠTENE VREDNOSTI",False,gd_chart
751000,gd_acc_751000,account.data_account_type_expenses,"ODHODKI IZ ODTUJITVE NALOŽBENIH NEPREMIČNIN, IZMERJENIH PO POŠTENI VREDNOSTI",False,gd_chart
752000,gd_acc_752000,account.data_account_type_expenses,"DENARNE KAZNI",False,gd_chart
753000,gd_acc_753000,account.data_account_type_expenses,"ODŠKODNINE",False,gd_chart
759000,gd_acc_759000,account.data_account_type_expenses,"OSTALI ODHODKI",False,gd_chart
760000,gd_acc_760000,account.data_account_type_revenue,"PRIHODKI OD PRODAJE PROIZVODOV IN STORITEV NA DOMAČEM TRGU",False,gd_chart
761000,gd_acc_761000,account.data_account_type_revenue,"PRIHODKI OD PRODAJE PROIZVODOV IN STORITEV NA TUJEM TRGU",False,gd_chart
762000,gd_acc_762000,account.data_account_type_revenue,"PRIHODKI OD PRODAJE TRGOVSKEGA BLAGA IN MATERIALA NA DOMAČEM TRGU",False,gd_chart
763000,gd_acc_763000,account.data_account_type_revenue,"PRIHODKI OD PRODAJE TRGOVSKEGA BLAGA IN MATERIALA NA TUJEM TRGU",False,gd_chart
765000,gd_acc_765000,account.data_account_type_revenue,"PRIHODKI OD NAJEMNIN",False,gd_chart
766000,gd_acc_766000,account.data_account_type_revenue,"PRIHODKI OD ODPRAVE REZERVACIJ",False,gd_chart
767000,gd_acc_767000,account.data_account_type_revenue,"PRIHODKI OD POSLOVNIH ZDRUŽITEV (PRESEŽEK IZ PREVREDNOTENJA - SLABO IME)",False,gd_chart
768000,gd_acc_768000,account.data_account_type_revenue,"DRUGI PRIHODKI, POVEZANI S POSLOVNIMI UČINKI (SUBVENCIJE, DOTACIJE, REGRESI, KOMPENZACIJE, PREMIJE ...)",False,gd_chart
769000,gd_acc_769000,account.data_account_type_revenue,"PREVREDNOTOVALNI POSLOVNI PRIHODKI",False,gd_chart
770000,gd_acc_770000,account.data_account_type_revenue,"FINANČNI PRIHODKI IZ DELEŽEV V DRUŽBAH V SKUPINI",False,gd_chart
771000,gd_acc_771000,account.data_account_type_revenue,"FINANČNI PRIHODKI IZ DELEŽEV V PRIDRUŽENIH DRUŽBAH IN SKUPAJ OBVLADOVANIH DRUŽBAH",False,gd_chart
772000,gd_acc_772000,account.data_account_type_revenue,"FINANČNI PRIHODKI IZ DELEŽEV V DRUGIH DRUŽBAH",False,gd_chart
773000,gd_acc_773000,account.data_account_type_revenue,"FINANČNI PRIHODKI IZ DRUGIH NALOŽB",False,gd_chart
774000,gd_acc_774000,account.data_account_type_revenue,"FINANČNI PRIHODKI IZ POSOJIL, DANIH DRUŽBAM V SKUPINI",False,gd_chart
775000,gd_acc_775000,account.data_account_type_revenue,"FINANČNI PRIHODKI IZ POSOJIL, DANIH DRUGIM (TUDI OD DEPOZITOV)",False,gd_chart
776000,gd_acc_776000,account.data_account_type_revenue,"FINANČNI PRIHODKI IZ POSLOVNIH TERJATEV DO DRUŽB V SKUPINI",False,gd_chart
777000,gd_acc_777000,account.data_account_type_revenue,"FINANČNI PRIHODKI IZ POSLOVNIH TERJATEV DO DRUGIH",False,gd_chart
778000,gd_acc_778000,account.data_account_type_revenue,"FINANČNI PRIHODKI IZ FINANČNIH SREDSTEV, RAZPOREJENIH PO POŠTENI VREDNOSTI PREK POSLOVNEGA IZIDA",False,gd_chart
780000,gd_acc_780000,account.data_account_type_revenue,"PRIHODKI IZ VREDNOTENJA NALOŽBENIH NEPREMIČNIN PO POŠTENI VREDNOSTI",False,gd_chart
781000,gd_acc_781000,account.data_account_type_revenue,"PRIHODKI IZ ODTUJITVE NALOŽBENIH NEPREMIČNIN, IZMERJENIH PO POŠTENI VREDNOSTI",False,gd_chart
785000,gd_acc_785000,account.data_account_type_revenue,"SUBVENCIJE, DOTACIJE IN PODOBNI PRIHODKI, KI NISO POVEZANI S POSLOVNIMI UČINKI",False,gd_chart
786000,gd_acc_786000,account.data_account_type_revenue,"PREJETE ODŠKODNINE",False,gd_chart
787000,gd_acc_787000,account.data_account_type_revenue,"PREJETE KAZNI",False,gd_chart
789000,gd_acc_789000,account.data_account_type_revenue,"OSTALI PRIHODKI",False,gd_chart
800000,gd_acc_800000,account.data_account_type_current_liabilities,"DOBIČEK ALI IZGUBA PRED OBDAVČITVIJO",False,gd_chart
810000,gd_acc_810000,account.data_account_type_current_liabilities,"DAVEK OD DOHODKA",False,gd_chart
812000,gd_acc_812000,account.data_account_type_current_liabilities,"DRUGI DAVKI, KI NISO IZKAZANI V DRUGIH POSTAVKAH",False,gd_chart
813000,gd_acc_813000,account.data_account_type_current_liabilities,"PRIHODKI (ODHODKI) IZ NASLOVA ODLOŽENEGA DAVKA",False,gd_chart
815000,gd_acc_815000,account.data_account_type_current_liabilities,"ČISTI DOBIČEK POSLOVNEGA LETA",False,gd_chart
820000,gd_acc_820000,account.data_account_type_current_liabilities,"ČISTI DOBIČEK ZA KRITJE PRENESENIH IZGUB",False,gd_chart
821000,gd_acc_821000,account.data_account_type_current_liabilities,"ČISTI DOBIČEK ZA OBLIKOVANJE ZAKONSKIH REZERV",False,gd_chart
822000,gd_acc_822000,account.data_account_type_current_liabilities,"ČISTI DOBIČEK ZA OBLIKOVANJE REZERV ZA LASTNE DELNICE OZIROMA DELEŽE",False,gd_chart
823000,gd_acc_823000,account.data_account_type_current_liabilities,"ČISTI DOBIČEK ZA OBLIKOVANJE STATUTARNIH REZERV",False,gd_chart
824000,gd_acc_824000,account.data_account_type_current_liabilities,"ČISTI DOBIČEK ZA DRUGE REZERVE IZ DOBIČKA",False,gd_chart
829000,gd_acc_829000,account.data_account_type_current_liabilities,"PRENOS NEUPORABLJENEGA DELA ČISTEGA DOBIČKA POSLOVNEGA LETA",False,gd_chart
890000,gd_acc_890000,account.data_account_type_current_liabilities,"IZGUBA TEKOČEGA LETA",False,gd_chart
899000,gd_acc_899000,account.data_account_type_current_liabilities,"PRENOS IZGUBE TEKOČEGA LETA",False,gd_chart
900000,gd_acc_900000,account.data_account_type_current_liabilities,"OSNOVNI DELNIŠKI KAPITAL - NAVADNE DELNICE",False,gd_chart
901000,gd_acc_901000,account.data_account_type_current_liabilities,"OSNOVNI DELNIŠKI KAPITAL - PREDNOSTNE DELNICE",False,gd_chart
902000,gd_acc_902000,account.data_account_type_current_liabilities,"OSNOVNI KAPITAL - KAPITALSKI DELEŽI",False,gd_chart
903000,gd_acc_903000,account.data_account_type_current_liabilities,"OSNOVNI KAPITAL - KAPITALSKA VLOGA",False,gd_chart
909000,gd_acc_909000,account.data_account_type_current_liabilities,"NEVPOKLICANI KAPITAL (ODBITNA POSTAVKA)",False,gd_chart
910000,gd_acc_910000,account.data_account_type_current_liabilities,"VPLAČILA NAD NAJMANJŠIMI EMISIJSKIMI ZNESKI DELNIC OZIROMA DELEŽEV (VPLAČANI PRESEŽEK KAPITALA)",False,gd_chart
911000,gd_acc_911000,account.data_account_type_current_liabilities,"VPLAČILA NAD KNJIGOVODSKO VREDNOSTJO PRI ODTUJITVI ZAČASNO ODKUPLJENIH LASTNIH DELNIC OZIROMA DELEŽEV",False,gd_chart
912000,gd_acc_912000,account.data_account_type_current_liabilities,"VPLAČILA NAD NAJMANJŠIM EMISIJSKIM ZNESKOM KAPITALA, PRIDOBLJENA Z IZDAJO ZAMENLJIVIH OBVEZNIC IN OBVEZNIC Z DELNIŠKO NAKUPNO OPCIJO",False,gd_chart
913000,gd_acc_913000,account.data_account_type_current_liabilities,"VPLAČILA ZA PRIDOBITEV DODATNIH PRAVIC IZ DELNIC OZIROMA DELEŽEV",False,gd_chart
914000,gd_acc_914000,account.data_account_type_current_liabilities,"DRUGA VPLAČILA KAPITALA NA PODLAGI STATUTA",False,gd_chart
915000,gd_acc_915000,account.data_account_type_current_liabilities,"ZNESKI IZ POENOSTAVLJENEGA ZMANJŠANJA OSNOVNEGA KAPITALA IN ZNESKI ZMANJŠANJA OSNOVNEGA KAPITALA Z UMIKOM DELNIC OZIROMA DELEŽEV",False,gd_chart
916000,gd_acc_916000,account.data_account_type_current_liabilities,"SPLOŠNI PREVREDNOTOVALNI POPRAVEK KAPITALA",False,gd_chart
917000,gd_acc_917000,account.data_account_type_current_liabilities,"ZNESKI IZ UČINKOV POTRJENE PRISILNE PORAVNAVE",False,gd_chart
920000,gd_acc_920000,account.data_account_type_current_liabilities,"ZAKONSKE REZERVE",False,gd_chart
921000,gd_acc_921000,account.data_account_type_current_liabilities,"REZERVE ZA LASTNE DELNICE OZIROMA LASTNE POSLOVNE DELEŽE",False,gd_chart
922000,gd_acc_922000,account.data_account_type_current_liabilities,"STATUTARNE REZERVE",False,gd_chart
923000,gd_acc_923000,account.data_account_type_current_liabilities,"DRUGE REZERVE IZ DOBIČKA",False,gd_chart
929000,gd_acc_929000,account.data_account_type_current_liabilities,"PRIDOBLJENE LASTNE DELNICE OZIROMA LASTNI POSLOVNI DELEŽI (ODBITNA POSTAVKA)",False,gd_chart
930000,gd_acc_930000,account.data_account_type_current_liabilities,"PRENESENI ČISTI DOBIČEK IZ PREJŠNJIH LET",False,gd_chart
931000,gd_acc_931000,account.data_account_type_current_liabilities,"PRENESENA ČISTA IZGUBA IZ PREJŠNJIH LET",False,gd_chart
932000,gd_acc_932000,account.data_account_type_current_liabilities,"NEUPORABLJENI DEL ČISTEGA DOBIČKA POSLOVNEGA LETA",False,gd_chart
933000,gd_acc_933000,account.data_account_type_current_liabilities,"ČISTA IZGUBA POSLOVNEGA LETA",False,gd_chart
934000,gd_acc_934000,account.data_account_type_current_liabilities,"PRENOS IZ PRESEŽKA IZ PREVREDNOTENJA",False,gd_chart
950000,gd_acc_950000,account.data_account_type_current_liabilities,"PRESEŽEK IZ PREVREDNOTENJA ZEMLJIŠČ",False,gd_chart
951000,gd_acc_951000,account.data_account_type_current_liabilities,"PRESEŽEK IZ PREVREDNOTENJA ZGRADB",False,gd_chart
952000,gd_acc_952000,account.data_account_type_current_liabilities,"PRESEŽEK IZ PREVREDNOTENJA OPREME",False,gd_chart
953000,gd_acc_953000,account.data_account_type_current_liabilities,"PRESEŽEK IZ PREVREDNOTENJA NEOPREDMETENIH SREDSTEV",False,gd_chart
954000,gd_acc_954000,account.data_account_type_current_liabilities,"PRESEŽEK IZ PREVREDNOTENJA DOLGOROČNIH FINANČNIH NALOŽB",False,gd_chart
955000,gd_acc_955000,account.data_account_type_current_liabilities,"PRESEŽEK IZ PREVREDNOTENJA KRATKOROČNIH FINANČNIH NALOŽB",False,gd_chart
959000,gd_acc_959000,account.data_account_type_current_liabilities,"POPRAVEK VREDNOSTI PRESEŽKOV IZ PREVREDNOTENJA ZA ODLOŽENI DAVEK",False,gd_chart
960000,gd_acc_960000,account.data_account_type_current_liabilities,"REZERVACIJE ZA STROŠKE REORGANIZACIJE PODJETJA",False,gd_chart
961000,gd_acc_961000,account.data_account_type_current_liabilities,"REZERVACIJE ZA POKRIVANJE PRIHODNJIH STROŠKOV OZIROMA ODHODKOV ZARADI RAZGRADNJE IN PONOVNE VZPOSTAVITVE PRVOTNEGA STANJA TER DRUGE PODOBNE REZERVACIJE",False,gd_chart
962000,gd_acc_962000,account.data_account_type_current_liabilities,"REZERVACIJE ZA KOČLJIVE POGODBE",False,gd_chart
963000,gd_acc_963000,account.data_account_type_current_liabilities,"REZERVACIJE ZA POKOJNINE, JUBILEJNE NAGRADE IN ODPRAVNINE OB UPOKOJITVI",False,gd_chart
964000,gd_acc_964000,account.data_account_type_current_liabilities,"REZERVACIJE ZA DANA JAMSTVA",False,gd_chart
965000,gd_acc_965000,account.data_account_type_current_liabilities,"DRUGE REZERVACIJE IZ NASLOVA DOLGOROČNO VNAPREJ VRAČUNANIH STROŠKOV",False,gd_chart
966000,gd_acc_966000,account.data_account_type_current_liabilities,"PREJETE DRŽAVNE PODPORE",False,gd_chart
967000,gd_acc_967000,account.data_account_type_current_liabilities,"PREJETE DONACIJE",False,gd_chart
968000,gd_acc_968000,account.data_account_type_current_liabilities,"DRUGE DOLGOROČNE PASIVNE ČASOVNE RAZMEJITVE",False,gd_chart
970000,gd_acc_970000,account.data_account_type_current_liabilities,"DOLGOROČNA POSOJILA, DOBLJENA PRI DRUŽBAH V SKUPINI",False,gd_chart
971000,gd_acc_971000,account.data_account_type_current_liabilities,"DOLGOROČNA POSOJILA, DOBLJENA PRI PRIDRUŽENIH DRUŽBAH",False,gd_chart
972000,gd_acc_972000,account.data_account_type_current_liabilities,"DOLGOROČNA POSOJILA, DOBLJENA PRI BANKAH IN DRUŽBAH V DRŽAVI",False,gd_chart
973000,gd_acc_973000,account.data_account_type_current_liabilities,"DOLGOROČNA POSOJILA, DOBLJENA PRI BANKAH IN DRUŽBAH V TUJINI",False,gd_chart
974000,gd_acc_974000,account.data_account_type_current_liabilities,"DOLGOROČNE FINANČNE OBVEZNOSTI V ZVEZI Z OBVEZNICAMI",False,gd_chart
975000,gd_acc_975000,account.data_account_type_current_liabilities,"DOLGOROČNI DOLGOVI IZ FINANČNEGA NAJEMA",False,gd_chart
976000,gd_acc_976000,account.data_account_type_current_liabilities,"DOLGOROČNE FINANČNE OBVEZNOSTI DO FIZIČNIH OSEB",False,gd_chart
979000,gd_acc_979000,account.data_account_type_current_liabilities,"DRUGE DOLGOROČNE FINANČNE OBVEZNOSTI",False,gd_chart
980000,gd_acc_980000,account.data_account_type_current_liabilities,"DOLGOROČNI KREDITI, DOBLJENI NA PODLAGI KREDITNIH POGODB OD DRUŽB V SKUPINI",False,gd_chart
981000,gd_acc_981000,account.data_account_type_current_liabilities,"DOLGOROČNI KREDITI, DOBLJENI NA PODLAGI KREDITNIH POGODB OD PRIDRUŽENIH DRUŽB IN SKUPAJ OBVLADOVANIH DRUŽB",False,gd_chart
982000,gd_acc_982000,account.data_account_type_current_liabilities,"DOLGOROČNI KREDITI, DOBLJENI OD DRUGIH DOMAČIH DOBAVITELJEV",False,gd_chart
983000,gd_acc_983000,account.data_account_type_current_liabilities,"DOLGOROČNI KREDITI, DOBLJENI OD DRUGIH TUJIH DOBAVITELJEV",False,gd_chart
985000,gd_acc_985000,account.data_account_type_current_liabilities,"DOLGOROČNE MENIČNE OBVEZNOSTI",False,gd_chart
986000,gd_acc_986000,account.data_account_type_current_liabilities,"DOLGOROČNI DOBLJENI PREDUJMI IN VARŠČINE",False,gd_chart
988000,gd_acc_988000,account.data_account_type_current_liabilities,"OBVEZNOSTI ZA ODLOŽENI DAVEK",False,gd_chart
989000,gd_acc_989000,account.data_account_type_current_liabilities,"DRUGE DOLGOROČNE POSLOVNE OBVEZNOSTI",False,gd_chart
990000,gd_acc_990000,account.data_account_type_current_liabilities,"NAJETA, IZPOSOJENA IN ZAKUPLJENA (TUJA) SREDSTVA",False,gd_chart
991000,gd_acc_991000,account.data_account_type_current_liabilities,"MENICE IN DRUGI VREDNOSTNI PAPIRJI, PREJETI ZA ZAVAROVANJE PLAČIL",False,gd_chart
992000,gd_acc_992000,account.data_account_type_current_liabilities,"BLAGO, PREJETO V KOMISIJSKO IN KONSIGNACIJSKO PRODAJO",False,gd_chart
993000,gd_acc_993000,account.data_account_type_current_liabilities,"VREDNOTNICE, IZDANE ZA OBRAČUNAVANJE ZNOTRAJ PRAVNE OSEBE",False,gd_chart
994000,gd_acc_994000,account.data_account_type_current_liabilities,"DRUGI AKTIVNI ZUNAJBILANČNI KONTI",False,gd_chart
995000,gd_acc_995000,account.data_account_type_current_liabilities,"LASTNIKI NAJETIH, IZPOSOJENIH IN ZAKUPLJENIH SREDSTEV",False,gd_chart
996000,gd_acc_996000,account.data_account_type_current_liabilities,"DOLŽNIKI, KI SO ZAVAROVALI PLAČILA Z MENICAMI IN DRUGIMI VREDNOSTNIMI PAPIRJI",False,gd_chart
997000,gd_acc_997000,account.data_account_type_current_liabilities,"OBVEZNOSTI IZ BLAGA, PREJETEGA V KOMISIJSKO IN KONSIGNACIJSKO PRODAJO",False,gd_chart
998000,gd_acc_998000,account.data_account_type_current_liabilities,"NOMINALNA VREDNOST VREDNOTNIC, IZDANIH ZA OBRAČUNAVANJE ZNOTRAJ PRAVNE OSEBE",False,gd_chart
999000,gd_acc_999000,account.data_account_type_current_liabilities,"DRUGI PASIVNI ZUNAJBILANČNI KONTI",False,gd_chart

```

## File: data\account.chart.template.csv

```csv
"id","property_account_receivable_id/id","property_account_payable_id/id","property_account_expense_categ_id/id","property_account_income_categ_id/id",income_currency_exchange_account_id/id,expense_currency_exchange_account_id/id,default_pos_receivable_account_id/id
"gd_chart","gd_acc_120000","gd_acc_220000","gd_acc_702000","gd_acc_762000","gd_acc_777000","gd_acc_484000","gd_acc_125000"


```

## File: data\account.fiscal.position.account.template.csv

```csv
"id","position_id/id","account_src_id/id","account_dest_id/id"
"gd_fp_eu_acc1","gd_fp_eu","gd_acc_120000","gd_acc_121000"
"gd_fp_eu_acc2","gd_fp_eu","gd_acc_220000","gd_acc_221000"
"gd_fp_eu_acc3","gd_fp_eu","gd_acc_760000","gd_acc_761000"
"gd_fp_eu_acc4","gd_fp_eu","gd_acc_762000","gd_acc_763000"
"gd_fp_ne_acc1","gd_fp_ne","gd_acc_120000","gd_acc_121000"
"gd_fp_ne_acc2","gd_fp_ne","gd_acc_220000","gd_acc_221000"
"gd_fp_ne_acc3","gd_fp_ne","gd_acc_760000","gd_acc_761000"
"gd_fp_ne_acc4","gd_fp_ne","gd_acc_762000","gd_acc_763000"


```

## File: data\account.fiscal.position.tax.template.csv

```csv
"id","position_id/id","tax_src_id/id","tax_dest_id/id"
"gd_fp_eu_tax1","gd_fp_eu","gd_taxp_3","gd_taxp_st_2"
"gd_fp_eu_tax2","gd_fp_eu","gd_taxp_2","gd_taxp_st_1"
"gd_fp_eu_tax3","gd_fp_eu","gd_taxr_3","gd_taxr_1"
"gd_fp_eu_tax4","gd_fp_eu","gd_taxr_2","gd_taxr_1"
"gd_fp_ne_tax1","gd_fp_ne","gd_taxp_3","gd_taxp_1"
"gd_fp_ne_tax2","gd_fp_ne","gd_taxp_2","gd_taxp_1"
"gd_fp_ne_tax3","gd_fp_ne","gd_taxr_3","gd_taxr_1"
"gd_fp_ne_tax4","gd_fp_ne","gd_taxr_2","gd_taxr_1"


```

## File: data\account.fiscal.position.template.csv

```csv
"id","chart_template_id/id","name","sequence","vat_required","auto_apply","country_id:id","country_group_id:id"
"gd_fp_do","gd_chart","Domači partner",1,1,1,base.si,
"gd_fp_do1","gd_chart","Zasebno EU",2,0,1,,base.europe
"gd_fp_eu","gd_chart","Partner EU",3,1,1,,base.europe
"gd_fp_ne","gd_chart","Partner izven EU / EU brez DŠ",4,0,1,,

```

## File: data\account.tax.group.csv

```csv
id,name,country_id/id
tax_group_0,DDV 0%,base.si
tax_group_95,DDV 9.5%,base.si
tax_group_22,DDV 22%,base.si

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_si.gd_chart')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="gd_taxr_1" model="account.tax.template">
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="name">Izstopni DDV 0%</field>
        <field name="description">DDV-I-0</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_zo_dvd_odb_izstopni_opr')],
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
                'minus_report_line_ids': [ref('tax_report_zo_dvd_odb_izstopni_opr')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="gd_taxr_2" model="account.tax.template">
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="name">Izstopni DDV 9.5%</field>
        <field name="description">DDV-I-9.5</field>
        <field name="amount_type">percent</field>
        <field name="amount">9.5</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_zo_dvd_odb_izstopni_zni')],
            }),
            (0,0, {
                'factor_percent': 100,
                'account_id': ref('gd_acc_260001'),
                'repartition_type': 'tax',
                'plus_report_line_ids': [ref('tax_report_zn_dvd_odb_izstopni_zni')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_zo_dvd_odb_izstopni_zni')],
            }),
            (0,0, {
                'factor_percent': 100,
                'account_id': ref('gd_acc_260001'),
                'repartition_type': 'tax',
                'minus_report_line_ids': [ref('tax_report_zn_dvd_odb_izstopni_zni')],
            }),
        ]"/>
    </record>

    <record id="gd_taxr_3" model="account.tax.template">
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="name">Izstopni DDV 22%</field>
        <field name="description">DDV-I-22</field>
        <field name="amount_type">percent</field>
        <field name="amount">22</field>
        <field name="type_tax_use">sale</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_zo_dvd_odb_izstopni_osn')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260002'),
                'plus_report_line_ids': [ref('tax_report_zn_dvd_odb_izstopni_osn')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_zo_dvd_odb_izstopni_osn')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260002'),
                'minus_report_line_ids': [ref('tax_report_zn_dvd_odb_izstopni_osn')],
            }),
        ]"/>
    </record>

    <record id="gd_taxp_1" model="account.tax.template">
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="name">Vstopni DDV 0%</field>
        <field name="description">DDV-V-0</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_zo_dvd_odb_vstopni_opr')],
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
                'minus_report_line_ids': [ref('tax_report_zo_dvd_odb_vstopni_opr')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record id="gd_taxp_2" model="account.tax.template">
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="name">Vstopni DDV 9.5%</field>
        <field name="description">DDV-V-9.5</field>
        <field name="amount_type">percent</field>
        <field name="amount">9.5</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_zo_dvd_odb_vstopni_zni')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160002'),
                'plus_report_line_ids': [ref('tax_report_zn_dvd_odb_vstopni_zni')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_zo_dvd_odb_vstopni_zni')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160002'),
                'minus_report_line_ids': [ref('tax_report_zn_dvd_odb_vstopni_zni')],
            }),
        ]"/>
    </record>

    <record id="gd_taxp_3" model="account.tax.template">
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="name">Vstopni DDV 22%</field>
        <field name="description">DDV-V-22</field>
        <field name="amount_type">percent</field>
        <field name="amount">22</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_zo_dvd_odb_vstopni_osn')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160003'),
                'plus_report_line_ids': [ref('tax_report_zn_dvd_odb_vstopni_osn')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_zo_dvd_odb_vstopni_osn')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160003'),
                'minus_report_line_ids': [ref('tax_report_zn_dvd_odb_vstopni_osn')],
            }),
        ]"/>
    </record>

    <record id="gd_taxp_nr_1" model="account.tax.template">
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="name">Vstopni DDV 9.5% - neodbitni</field>
        <field name="description">DDV-V-9.5-NE</field>
        <field name="amount_type">percent</field>
        <field name="amount">9.5</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="tax_group_95"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_zo_dvd_neo_sdtopni_zni')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160001'),
                'plus_report_line_ids': [ref('tax_report_zn_dvd_neo_vstopni_zni')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_zo_dvd_neo_sdtopni_zni')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160001'),
                'minus_report_line_ids': [ref('tax_report_zn_dvd_neo_vstopni_zni')],
            }),
        ]"/>
    </record>

    <record id="gd_taxp_nr_2" model="account.tax.template">
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="name">Vstopni DDV 22% - neodbitni</field>
        <field name="description">DDV-V-22-NE</field>
        <field name="amount_type">percent</field>
        <field name="amount">22</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include" eval="1"/>
        <field name="tax_group_id" ref="tax_group_22"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_zo_dvd_neo_sdtopni_osno')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160001'),
                'plus_report_line_ids': [ref('tax_report_zn_dvd_neo_vstopni_osnnovna')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_zo_dvd_neo_sdtopni_osno')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160001'),
                'minus_report_line_ids': [ref('tax_report_zn_dvd_neo_vstopni_osnnovna')],
            }),
        ]"/>
    </record>

    <record id="gd_taxp_st_1" model="account.tax.template">
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="name">Vstopni DDV 9.5% - samoobdavčitev</field>
        <field name="description">DDV-V-9.5-SO</field>
        <field name="amount_type">percent</field>
        <field name="amount">9.5</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_zo_dvd_odb_vstopni_zni')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160002'),
                'plus_report_line_ids': [ref('tax_report_zn_dvd_odb_vstopni_zni')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260001'),
                'plus_report_line_ids': [ref('tax_report_zn_dvd_odb_izstopni_zni')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_zo_dvd_odb_vstopni_zni')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160002'),
                'minus_report_line_ids': [ref('tax_report_zn_dvd_odb_vstopni_zni')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260001'),
                'minus_report_line_ids': [ref('tax_report_zn_dvd_odb_izstopni_zni')],
            }),
        ]"/>
    </record>

    <record id="gd_taxp_st_2" model="account.tax.template">
        <field name="chart_template_id" ref="gd_chart"/>
        <field name="name">Vstopni DDV 22% - samoobdavčitev</field>
        <field name="description">DDV-V-22-SO</field>
        <field name="amount_type">percent</field>
        <field name="amount">22</field>
        <field name="type_tax_use">purchase</field>
        <field name="price_include" eval="0"/>
        <field name="tax_group_id" ref="tax_group_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_zo_dvd_odb_vstopni_osn')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160003'),
                'plus_report_line_ids': [ref('tax_report_zn_dvd_odb_vstopni_osn')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260002'),
                'plus_report_line_ids': [ref('tax_report_zn_dvd_odb_izstopni_osn')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_zo_dvd_odb_vstopni_osn')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_160003'),
                'minus_report_line_ids': [ref('tax_report_zn_dvd_odb_vstopni_osn')],
            }),
            (0,0, {
                'factor_percent': -100,
                'repartition_type': 'tax',
                'account_id': ref('gd_acc_260002'),
                'minus_report_line_ids': [ref('tax_report_zn_dvd_odb_izstopni_osn')],
            }),
        ]"/>
    </record>

</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.si"/>
    </record>

	<record id="tax_report_zo_dvd" model="account.tax.report.line">
        <field name="name">Znesek osnov za DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_zo_dvd_neo" model="account.tax.report.line">
        <field name="name">Neodbitni DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_zo_dvd"/>
    </record>

    <record id="tax_report_zo_dvd_neo_sdtopni" model="account.tax.report.line">
        <field name="name">Vstopni DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_zo_dvd_neo"/>
    </record>

    <record id="tax_report_zo_dvd_neo_sdtopni_osno" model="account.tax.report.line">
        <field name="name">Nabave po osnovni stopnji DDV</field>
        <field name="tag_name">Nabave po osnovni stopnji DDV (Neodbitni)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_zo_dvd_neo_sdtopni"/>
    </record>

    <record id="tax_report_zo_dvd_neo_sdtopni_zni" model="account.tax.report.line">
        <field name="name">Nabave po znižani stopnji DDV</field>
        <field name="tag_name">Nabave po znižani stopnji DDV (Neodbitni)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_zo_dvd_neo_sdtopni"/>
    </record>

    <record id="tax_report_zo_dvd_odb" model="account.tax.report.line">
        <field name="name">Odbitni DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_zo_dvd"/>
    </record>

    <record id="tax_report_zo_dvd_odb_izstopni" model="account.tax.report.line">
        <field name="name">Izstopni DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_zo_dvd_odb"/>
    </record>

    <record id="tax_report_zo_dvd_odb_izstopni_opr" model="account.tax.report.line">
        <field name="name">Prodaja oproščena DDV</field>
        <field name="tag_name">Prodaja oproščena DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_zo_dvd_odb_izstopni"/>
    </record>

    <record id="tax_report_zo_dvd_odb_izstopni_osn" model="account.tax.report.line">
        <field name="name">Prodaja po osnovni stopnji DDV</field>
        <field name="tag_name">Prodaja po osnovni stopnji DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_zo_dvd_odb_izstopni"/>
    </record>

    <record id="tax_report_zo_dvd_odb_izstopni_zni" model="account.tax.report.line">
        <field name="name">Prodaja po znižani stopnji DDV</field>
        <field name="tag_name">Prodaja po znižani stopnji DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_zo_dvd_odb_izstopni"/>
    </record>

    <record id="tax_report_zo_dvd_odb_vstopni" model="account.tax.report.line">
        <field name="name">Vstopni DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_zo_dvd_odb"/>
    </record>

    <record id="tax_report_zo_dvd_odb_vstopni_opr" model="account.tax.report.line">
        <field name="name">Nabave oproščene DDV</field>
        <field name="tag_name">Nabave oproščene DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_zo_dvd_odb_vstopni"/>
    </record>

    <record id="tax_report_zo_dvd_odb_vstopni_osn" model="account.tax.report.line">
        <field name="name">Nabave po osnovni stopnji DDV</field>
        <field name="tag_name">Nabave po osnovni stopnji DDV (Vstopni)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_zo_dvd_odb_vstopni"/>
    </record>

    <record id="tax_report_zo_dvd_odb_vstopni_zni" model="account.tax.report.line">
        <field name="name">Nabave po znižani stopnji DDV</field>
        <field name="tag_name">Nabave po znižani stopnji DDV (Vstopni)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_zo_dvd_odb_vstopni"/>
    </record>

    <record id="tax_report_zn_dvd" model="account.tax.report.line">
        <field name="name">Znesek DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_zn_dvd_neo" model="account.tax.report.line">
        <field name="name">Neodbitni DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_zn_dvd"/>
    </record>

    <record id="tax_report_zn_dvd_neo_vstopni" model="account.tax.report.line">
        <field name="name">Vstopni DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_zn_dvd_neo"/>
    </record>

    <record id="tax_report_zn_dvd_neo_vstopni_osnnovna" model="account.tax.report.line">
        <field name="name">Neodbitni DDV – osnovna stopnja</field>
        <field name="tag_name">Neodbitni DDV – osnovna stopnja</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_zn_dvd_neo_vstopni"/>
    </record>

    <record id="tax_report_zn_dvd_neo_vstopni_zni" model="account.tax.report.line">
        <field name="name">Neodbitni DDV – znižana stopnja</field>
        <field name="tag_name">Neodbitni DDV – znižana stopnja</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_zn_dvd_neo_vstopni"/>
    </record>

    <record id="tax_report_zn_dvd_odb" model="account.tax.report.line">
        <field name="name">Odbitni DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_zn_dvd"/>
    </record>

    <record id="tax_report_zn_dvd_odb_izstopni" model="account.tax.report.line">
        <field name="name">Izstopni DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_zn_dvd_odb"/>
    </record>

    <record id="tax_report_zn_dvd_odb_izstopni_osn" model="account.tax.report.line">
        <field name="name">Izstopni DDV - osnovna stopnja</field>
        <field name="tag_name">Izstopni DDV - osnovna stopnja</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_zn_dvd_odb_izstopni"/>
    </record>

    <record id="tax_report_zn_dvd_odb_izstopni_zni" model="account.tax.report.line">
        <field name="tag_name">Izstopni DDV - znižana stopnja</field>
        <field name="name">Izstopni DDV - znižana stopnja</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_zn_dvd_odb_izstopni"/>
    </record>

    <record id="tax_report_zn_dvd_odb_vstopni" model="account.tax.report.line">
        <field name="name">Vstopni DDV</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_zn_dvd_odb"/>
    </record>

    <record id="tax_report_zn_dvd_odb_vstopni_osn" model="account.tax.report.line">
        <field name="name">Vstopni DDV - osnovna stopnja</field>
        <field name="tag_name">Vstopni DDV - osnovna stopnja</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_zn_dvd_odb_vstopni"/>
    </record>

    <record id="tax_report_zn_dvd_odb_vstopni_zni" model="account.tax.report.line">
        <field name="name">Vstopni DDV - znižana stopnja</field>
        <field name="tag_name">Vstopni DDV - znižana stopnja</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_zn_dvd_odb_vstopni"/>
    </record>

</odoo>

```

## File: data\l10n_si_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_si_statements_menu" name="Slovenia" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>
    <!-- Chart Template -->
    <record id="gd_chart" model="account.chart.template">
        <field name="name">Kontni načrt za gospodarske družbe</field>
        <field name="bank_account_code_prefix">110</field>
        <field name="cash_account_code_prefix">100</field>
        <field name="transfer_account_code_prefix">109</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.EUR"/>
        <field name="country_id" ref="base.si"/>
    </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="3.6" y="4.92" width="64.8" height="35.1" maskUnits="userSpaceOnUse">
      <rect x="5.09" y="7.8" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="1200" height="600" transform="translate(3.6 4.92) scale(0.05 0.06)" xlink:href="data:image/png;base64,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JmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJAmYAEAAACQJmABAAAAkCZgAQAAAJA2CaWEqQPwbk8AAAAASUVORK5CYII="/>
    </g>
  </g>
</svg>

```

