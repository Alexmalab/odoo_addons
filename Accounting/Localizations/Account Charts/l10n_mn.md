# Odoo Module: l10n_mn

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
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    "name" : "Mongolia - Accounting",
    "version" : "1.1",
    'category': 'Accounting/Localizations/Account Charts',
    "author" : "BumanIT LLC, Odoo S.A.",
    "description": """
This is the module to manage the accounting chart for Mongolia.
===============================================================

    * the Mongolia Official Chart of Accounts,
    * the Tax Code Chart for Mongolia
    * the main taxes used in Mongolia

Financial requirement contributor: Baskhuu Lodoikhuu. BumanIT LLC
""",
    "depends": ['account'],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account.account.tag.csv',
        'data/account.account.template.csv',
        'data/account.tax.group.csv',
        'data/account_tax_template_data.xml',
        'data/account_fiscal_position_template_data.xml',
        'data/account.chart.template.csv',
        'data/account_chart_template_configuration_data.xml',
        'data/l10n_mn_chart_template_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account.account.tag.csv

```csv
id,name,applicability,country_id:id
account.account_tag_operating,CF: Үндсэн: Бусад,accounts,
account.account_tag_financing,CF: Хөрөнгө оруулалт: Бусад Урт хугацаат хөрөнгө,accounts,
account.account_tag_investing,CF: Санхүү: Ногдол ашиг,accounts,
account_tag_operating_product,CF: Үндсэн: Бараа үйлчилгээ,accounts,
account_tag_operating_loyalty,CF: Үндсэн: Эрхийн шимтгэл хураамж,accounts,
account_tag_operating_insurance,CF: Үндсэн: Нийгмийн Даатгал,accounts,
account_tag_operating_insurance2,CF: Үндсэн: Бусад Даатгал,accounts,
account_tag_operating_taxrefund,CF: Үндсэн: Татвар,accounts,
account_tag_operating_funding,CF: Үндсэн: Татаас санхүүжилт,accounts,
account_tag_operating_employee,CF: Үндсэн: Ажилчид,accounts,
account_tag_operating_cost,CF: Үндсэн: Ашиглалтын зардал,accounts,
account_tag_operating_transport,CF: Үндсэн: Түлш тээвэр сэлбэг,accounts,
account_tag_operating_interest,CF: Үндсэн: Зээлийн хүү,accounts,
account_tag_investing_fasset,CF: Хөрөнгө оруулалт: Үндсэн хөрөнгө,accounts,
account_tag_investing_iasset,CF: Хөрөнгө оруулалт: Биет бус хөрөнгө,accounts,
account_tag_investing_invest,CF: Хөрөнгө оруулалт: Хөрөнгө оруулалт,accounts,
account_tag_investing_loan,CF: Хөрөнгө оруулалт: Зээл урьдчилгаа,accounts,
account_tag_investing_interest,CF: Хөрөнгө оруулалт: Зээлийн хүү,accounts,
account_tag_investing_dividends,CF: Хөрөнгө оруулалт: Ногдол ашиг,accounts,
account_tag_financing_loan,"CF: Санхүү: Зээл, өрийн үнэт цаас",accounts,
account_tag_financing_rental,CF: Санхүү: Санхүүгийн түрээс,accounts,
account_tag_financing_stock,CF: Санхүү: Хувьцаа өмчийн бусад үнэт цаас,accounts,
account_tag_financing_donation,CF: Санхүү: Хандив,accounts,
account_tag_exchange,CF: Валютын ханшийн зөрүү,accounts,
account_cashflow_tag_111,"CF:[1.1.1] Бараа борлуулсан, үйлчилгээ үзүүлсний орлого",accounts,
account_cashflow_tag_112,"CF:[1.1.2] Эрхийн шимтгэл, хураамж, төлбөрийн орлого",accounts,
account_cashflow_tag_113,"CF:[1.1.3] Даатгалын нөхвөрөөс хүлээн авсан мөнгө",accounts,
account_cashflow_tag_114,"CF:[1.1.4] Буцаан авсан албан татвар",accounts,
account_cashflow_tag_115,"CF:[1.1.5] Татаас, санхүүжилтийн орлого",accounts,
account_cashflow_tag_116,"CF:[1.1.6] Бусад мөнгөн орлого",accounts,
account_cashflow_tag_121,"CF:[1.2.1] Ажиллагчдад төлсөн",accounts,
account_cashflow_tag_122,"CF:[1.2.2] Нийгмийн даатгалын байгууллагад төлсөн",accounts,
account_cashflow_tag_123,"CF:[1.2.3] Бараа материал худалдан авахад төлсөн",accounts,
account_cashflow_tag_124,"CF:[1.2.4] Ашиглалтын зардалд төлсөн",accounts,
account_cashflow_tag_125,"CF:[1.2.5] Түлш шатахуун, тээврийн хөлс, сэлбэг хэрэгсэлд төлсөн",accounts,
account_cashflow_tag_126,"CF:[1.2.6] Хүүний төлбөрт төлсөн",accounts,
account_cashflow_tag_127,"CF:[1.2.7] Татварын байгууллагад төлсөн",accounts,
account_cashflow_tag_128,"CF:[1.2.8] Даатгалын төлбөрт төлсөн",accounts,
account_cashflow_tag_129,"CF:[1.2.9] Бусад мөнгөн зарлага",accounts,
account_cashflow_tag_211_221,"CF:[2.1.1] Үндсэн хөрөнгө борлуулсны орлого",accounts,
account_cashflow_tag_212_222,"CF:[2.1.2] Биет бус хөрөнгө борлуулсны орлого",accounts,
account_cashflow_tag_213_223,"CF:[2.1.3] Хөрөнгө оруулалт борлуулсны орлого",accounts,
account_cashflow_tag_214_224,"CF:[2.1.4] Бусад урт хугацаат хөрөнгө борлуулсны орлого",accounts,
account_cashflow_tag_215_225,"CF:[2.1.5] Бусдад олгосон зээл, мөнгөн урьдчилгааны буцаан төлөлт",accounts,
account_cashflow_tag_216,"CF:[2.1.6] Хүлээн авсан хүүний орлого",accounts,
account_cashflow_tag_217,"CF:[2.1.7] Хүлээн авсан ногдол ашиг",accounts,
account_cashflow_tag_221,"CF:[2.2.1] Үндсэн хөрөнгө олж эзэмшихэд төлсөн",accounts,
account_cashflow_tag_222,"CF:[2.2.2] Биет бус хөрөнгө олж эзэмшихэд төлсөн",accounts,
account_cashflow_tag_223,"CF:[2.2.3] Хөрөнгө оруулалт олж эзэмшихэд төлсөн",accounts,
account_cashflow_tag_224,"CF:[2.2.4] Бусад урт хугацаат хөрөнгө олж эзэмшихэд төлсөн",accounts,
account_cashflow_tag_225,"CF:[2.2.5] Бусдад олгосон зээл болон урьдчилгаа",accounts,
account_cashflow_tag_311_321,"CF:[3.1.1] Зээл авсан, өрийн үнэт цаас гаргаснаас хүлээн авсан",accounts,
account_cashflow_tag_312_323,"CF:[3.1.2] Хувьцаа болон өмчийн бусад үнэт цаас гаргаснаас хүлээн авсан",accounts,
account_cashflow_tag_313,"CF:[3.1.3] Төрөл бүрийн хандив",accounts,
account_cashflow_tag_321,"CF:[3.2.1] Зээл, өрийн үнэт цаасны төлбөрт төлсөн мөнгө",accounts,
account_cashflow_tag_322,"CF:[3.2.2] Санхүүгийн түрээсийн өглөгт төлсөн",accounts,
account_cashflow_tag_323,"CF:[3.2.3] Хувьцаа буцаан худалдаж авахад төлсөн",accounts,
account_cashflow_tag_324,"CF:[3.2.4] Төлсөн ногдол ашиг",accounts,
account_cashflow_tag_325,"CF:[3.2.5] Төрөл бүрийн хандив, тусламж болон торгууль төлсөн",accounts,
account_cashflow_tag_400,"CF:[4] Валютын ханшийн зөрүү",accounts,
vat_report_tag2,ТТ3а: НӨАТ-аас чөлөөлөгдөх борлуулалтын орлого (2),taxes,base.mn
vat_report_tag5,ТТ3a: Дотоодын зах зээлд борлуулсан барааны борлуулалтын орлого (5),taxes,base.mn
vat_report_tag6,ТТ3a: Бусад барааны борлуулалтын орлого (6),taxes,base.mn
vat_report_tag7,ТТ3a: Эрх борлуулсны орлого (7),taxes,base.mn
vat_report_tag8,ТТ3a: Татан буугдах үед үлдсэн барааны орлого (8),taxes,base.mn
vat_report_tag9,ТТ3a: Өрийн төлбөрт шилжүүлсэн барааны орлого (9),taxes,base.mn
vat_report_tag11,ТТ3a: Наториатын үйлчилгээний орлого (хуучирсан),taxes,base.mn
vat_report_tag12,ТТ3a: Өрийн төлбөрт тооцсон ажил үйлчилгээний орлого (11),taxes,base.mn
vat_report_tag13,"ТТ3a: Цахилгаан,  дулаан, ус, шуудан, холбооны үйлчилгээ үзүүлсний орлого (12)",taxes,base.mn
vat_report_tag14,"ТТ3a: Бараа түрээслүүлэх, эзэмшүүлэх, ашиглуулах үйлчилгээний орлого (13)",taxes,base.mn
vat_report_tag15,"ТТ3a: Байр түрээслүүлэх, эзэмшүүлэх, ашиглуулах үйлчилгээний орлого (14)",taxes,base.mn
vat_report_tag16,"ТТ3a: Үл хөдлөх, хөдлөх хөрөнгө түрээслүүлэх, эзэмшүүлэх ашиглуулах үйлчилгээий орлого (15)",taxes,base.mn
vat_report_tag17,"ТТ3a: Шинэ бүтээл, загвар, зохиогчийн эрх, барааны тэмдэг худалдсаны орлого (16)",taxes,base.mn
vat_report_tag18,"ТТ3a: Хонжворт сугалаа, төлбөрт таавар, бооцоот тоглоомын орлого (17)",taxes,base.mn
vat_report_tag19,ТТ3a: Зуучлалын үйлчилгээний орлого (18),taxes,base.mn
vat_report_tag20,"ТТ3a: Бусдаас авсан хүү, торгууль, алдангийн орлого (19)",taxes,base.mn
vat_report_tag21,ТТ3a: Хөрөнгийн үнэлгээний үйлчилгээний орлого (20),taxes,base.mn
vat_report_tag22,"ТТ3a: Төсвийн санхүүжилт, татаас, урамшууллын орлого (21)",taxes,base.mn
vat_report_tag23,"ТТ3a: Өмгөөлөл, хууль зүйн үйлчилгээний орлого (22)",taxes,base.mn
vat_report_tag24,"ТТ3a: Үсчин, гоо сайхан, засвар үйлчилгээ, угаалга, хими цэвэрлэгээний үйлчилгээний орлого (23)",taxes,base.mn
vat_report_tag25,ТТ3a: Хуулийн 13-т зааснаас бусад үйлчилгээний орлого (24),taxes,base.mn
vat_report_tag28,ТТ3a: Экспортонд гаргасан барааны борлуулалтын орлого (27),taxes,base.mn
vat_report_tag29,ТТ3a: Экспортолсон үйлчилгээний борлуулалтын орлого (28),taxes,base.mn
vat_report_tag33,"ТТ3а: Худалдан авсан нийт бараа, ажил үйлчилгээ (32)",taxes,base.mn
vat_report_tag35,"ТТ3a: НӨАТ-тэй авсан Импортын бараа, ажил үйлчилгээний худалдан авалт (34)",taxes,base.mn
vat_report_tag36,"ТТ3a: Дотоодын зах зээлээс НӨАТ-тэй авсан бараа, ажил, үйлчилгээ (35)",taxes,base.mn
vat_report_tag37,"ТТ3a: Суутган төлөгчөөр бүртгүүлэх үед НӨАТ-тэй авсан бараа, үйлчилгээ (36)",taxes,base.mn
vat_report_tag37a,"ТТ3a: Барилга байгууламж: Үндсэн хөрөнгө бэлтгэхэд зориулсан импортын худалдан авалт (37)",taxes,base.mn
vat_report_tag37b,"ТТ3a: Машин, тоног төхөөрөмж: Үндсэн хөрөнгө бэлтгэхэд зориулсан импортын худалдан авалт (38)",taxes,base.mn
vat_report_tag37c,"ТТ3a: Бусад: Үндсэн хөрөнгө бэлтгэхэд зориулсан импортын худалдан авалт (39)",taxes,base.mn
vat_report_tag37d,"ТТ3a: Хайгууланы үнэлгээ: Үндсэн хөрөнгө бэлтгэхэд зориулсан импортын худалдан авалт (40)",taxes,base.mn
vat_report_tag38,"ТТ3a: Мал аж ахуй, тариалан эрхлэгчээс НӨАТ-тэй авсан бараа, үйлчилгээ (41)",taxes,base.mn
vat_report_tag41,"ТТ3а: Суудлын автомашин, түүний эд анги сэлбэгт төлсөн  НӨАТ (44)",taxes,base.mn
vat_report_tag42,"ТТ3а: Ажлын хэрэгцээнд зориулж авсан бараа, үйлчилгээнд төлсөн НӨАТ (45)",taxes,base.mn
vat_report_tag43,"ТТ3а: Үндсэн хөрөнгө бэлтгэхэд зориулж импортоор авсан бараа, үйлчилгээнд төлсөн  НӨАТ (хуучирсан)",taxes,base.mn
vat_report_tag44,"ТТ3а: Чөлөөлөгдөх үйлдвэрлэл, үйлчилгээнд зориулж авсан бараа, ажил, үйлчилгээнд төлсөн НӨАТ (46)",taxes,base.mn
vat_report_tag45,"ТТ3а: Хайгуулын ажил болон ашиглалтын өмнөх үйл ажиллагаанд зориулж импортоор авсан бараа, ажил, үйлчилгээнд төлсөн НӨАТ (47)",taxes,base.mn
vat_report_tag46,"ТТ3а: Импортоор оруулсан бусад хамааралгүй бараа, ажил, үйлчилгээнд төлсөн НӨАТ (48)",taxes,base.mn
vat_report_tag48,ТТ3а: Cанхүүгийн түрээсээр дотоодын зах зээлд борлуулсны орлого (50),taxes,base.mn
vat_report_tag49,"ТТ3а: Факторинг, форфайтинг хэлцлийн үйлчилгээний орлого (51)",taxes,base.mn
vat_report_tag51,ТТ3а: Санхүүгийн түрээсийн зүйлийг худалдан авахад төлсөн түрээсийн төлбөр (53),taxes,base.mn
vat_report_tag52,"ТТ3а: Факторинг, форфайтинг хэлцлийг авахад төлсөн төлбөр (54)",taxes,base.mn
vat_report_tag56a,"ТТ3a: Борлуулалтын орлогын буцаалт, хөнгөлөлт (58)",taxes,base.mn
vat_report_tag57a,"ТТ3a: Борлуулалтын орлогын буцаалт, хөнгөлөлт - татварын дүн (59)",taxes,base.mn
vat_report_tag58a,"ТТ3a: Худалдан авалтын буцаалт, хөнгөлөлт (60)",taxes,base.mn
vat_report_tag59a,"ТТ3a: Худалдан авалтын буцаалт, хөнгөлөлт - татварын дүн (61)",taxes,base.mn
vat_report_tag58,"ТТ3а: Борлуулалтын орлогын буцаалт, хөнгөлөлт (58)",accounts,
vat_report_tag59,"ТТ3а: Худалдан авалтын буцаалт, хөнгөлөлт (59)",accounts,
tax_report_tag2,ТТ02: Татвараас чөлөөлөгдөх орлого (2),accounts,
tax_report_tag5,ТТ02: Бусад орлогын дүн (5),accounts,
tax_report_tag7,ТТ02: Үндсэн ажил үйлчилгээний орлого (7),accounts,
tax_report_tag8,ТТ02: Туслах ажил үйлчилгээний орлого (8),accounts,
tax_report_tag9,"ТТ02: Хувьцаа, үнэт цаас борлуулсны орлого (9)",accounts,
tax_report_tag10,"ТТ02: Үнэ төлбөргүйгээр бусдаас авсан бараа, үйлчилгээ (10)",accounts,
tax_report_tag11,ТТ02: Биет бус хөрөнгө борлуулсны орлого (11),accounts,
tax_report_tag12,ТТ02: Техникийн зөвлөх үйлчилгээний орлого (12),accounts,
tax_report_tag13,"ТТ02: Гэрээний хүү, торгууль, хохирлын нөхөн төлбөрийн орлого (13)",accounts,
tax_report_tag14,ТТ02: Гадаад валютын ханшийн зөрүүгийн бодит орлого (14),accounts,
tax_report_tag15,ТТ02: Хөдлөх болон үл хөдлөх эд хөрөнгийн түрээсийн орлого (15),accounts,
tax_report_tag16,ТТ02: Хөдлөх эд хөрөнгө борлуулсны орлого (16),accounts,
tax_report_tag17,ТТ02: Албан татвар ногдох бусад орлого (17),accounts,
tax_report_tag22,ТТ02: Татвар ногдох орлогоос хасагдахгүй зардал (22),accounts,
tax_report_tag23,ТТ02: Татвар ногдох орлогыг бууруулах (23),accounts,
tax_report_tag25,ТТ02: Сайн дурын даатгалын хураамжийн хэтрэлт (25),accounts,
tax_report_tag27,ТТ02: Өмнөх жилийн татварын алдагдлаас тайлант хугацаанд шилжүүлсэн дүн (27),accounts,
tax_report_tag30,ТТ02: Татварын хөнгөлөлт (30),accounts,
tax_report_tag32,"ТТ02: Эротик хэвлэл, бичлэг, тоглолт борлуулалтын орлого (32)",accounts,
tax_report_tag34,"ТТ02: Таавар, бооцоот тоглоом, хонжворт сугалааны орлого (34)",accounts,
tax_report_tag35,"ТТ02: Таавар, бооцоот тоглоом, хонжворт сугалааны орлого олоход зарцуулсан зардал (35)",accounts,
tax_report_tag36,ТТ02: Хонжворт олгосон мөнгө болон барааны үнэ (36),accounts,
tax_report_tag39,ТТ02: Хүүгийн орлого (39),accounts,
tax_report_tag41,"ТТ02: Гадаад улсад олсон ногдол ашиг, хүүгийн орлого (41)",accounts,
tax_report_tag43,ТТ02: Гадаад улсад олсон орлого (43),accounts,
tax_report_tag46,ТТ02: Ногдол ашгийн орлого (46),accounts,
tax_report_tag48,ТТ02: Эрхийн шимтгэлийн орлого (48),accounts,
tax_report_tag50,ТТ02: Эрх борлуулсны орлого (50),accounts,
tax_report_tag52,ТТ02: Үл хөдлөх хөрөнгө борлуулсны орлого (52),accounts,

```

## File: data\account.account.template.csv

```csv
id,code,name,account_type,tag_ids/id,reconcile,chart_template_id:id
account_template_1001_0101,10010101,Бэлэн мөнгө,asset_cash,,0,mn_chart_1
account_template_1003_0101,10030101,Жижиг мөнгөн сан,asset_cash,,0,mn_chart_1
account_template_1101_0101,11010101,Банкинд байгаа мөнгө,asset_cash,,0,mn_chart_1
account_template_1103_0101,11030101,Хугацаагүй хадгаламж,asset_cash,,0,mn_chart_1
account_template_1104_0101,11040101,Банкны төлбөрийн темринал,asset_cash,,0,mn_chart_1
account_template_1109_0101,11090101,Замд яваа мөнгө,asset_current,,1,mn_chart_1
account_template_1110_0101,11100101,Бусад мөнгө түүнтэй адилтгах хөрөнгө,asset_current,,0,mn_chart_1
account_template_1201_0101,12010101,Худалдааны авлага ААН,asset_receivable,account_cashflow_tag_111,1,mn_chart_1
account_template_1201_0201,12010201,Худалдааны авлага Хувь хүн,asset_receivable,account_cashflow_tag_111,1,mn_chart_1
account_template_1201_0202,12010202,Худалдааны авлага Хувь хүн (PoS),asset_receivable,account_cashflow_tag_111,1,mn_chart_1
account_template_1201_0301,12010301,Худалдааны авлага Холбоотой тал,asset_receivable,account_cashflow_tag_111,1,mn_chart_1
account_template_1202_0101,12020101,Авлагын бичиг,asset_prepayments,account_cashflow_tag_123,0,mn_chart_1
account_template_1203_0101,12030101,Найдваргүй авлагын хасагдуулга,asset_prepayments,account_cashflow_tag_123,0,mn_chart_1
account_template_1204_0101,12040101,Урьдчилж төлсөн ААНОАТатвар,asset_prepayments,account_cashflow_tag_127,0,mn_chart_1
account_template_1204_0201,12040201,Урьдчилж төлсөн ХХОАТатвар,asset_prepayments,account_cashflow_tag_127,0,mn_chart_1
account_template_1204_0301,12040301,Урьдчилж төлсөн НӨАТатвар,asset_prepayments,account_cashflow_tag_127,0,mn_chart_1
account_template_1204_0302,12040302,Хойшлуулсан НӨАТатвар /Барилга байгууламж/,asset_prepayments,,0,l10n_mn.mn_chart_1
account_template_1204_0303,12040303,Хойшлуулсан НӨАТатвар /Машин тоног төхөөрөмж/,asset_prepayments,,0,l10n_mn.mn_chart_1
account_template_1204_0304,12040304,Хойшлуулсан НӨАТатвар /Хайгуулын үнэлгээ/,asset_prepayments,,0,l10n_mn.mn_chart_1
account_template_1204_0401,12040401,Урьдчилж төлсөн Гаалийн албан татвар,asset_prepayments,account_cashflow_tag_127,0,mn_chart_1
account_template_1204_0501,12040501,Урьдчилж төлсөн Үл хөдлөх хөрөнгийн албан татвар,asset_prepayments,account_cashflow_tag_127,0,mn_chart_1
account_template_1204_0601,12040601,Урьдчилж төлсөн Газрын төлбөр,asset_prepayments,account_cashflow_tag_127,0,mn_chart_1
account_template_1204_0701,12040701,Урьдчилж төлсөн АТӨЯХАТатвар,asset_prepayments,account_cashflow_tag_127,0,mn_chart_1
account_template_1204_0801,12040801,Урьдчилж төлсөн Агаарын бохирдлын татвар,asset_prepayments,account_cashflow_tag_127,0,mn_chart_1
account_template_1204_0901,12040901,Урьдчилж төлсөн Онцгой албан татвар,asset_prepayments,account_cashflow_tag_127,0,mn_chart_1
account_template_1204_1001,12041001,Урьдчилж төлсөн Байгалийн нөөц ашиглалтын төлбөр,asset_prepayments,account_cashflow_tag_127,0,mn_chart_1
account_template_1204_9901,12049901,Урьдчилж төлсөн Бусад татвар,asset_prepayments,account_cashflow_tag_127,0,mn_chart_1
account_template_1204_9902,12049902,Урьдчилж төлсөн татварууд - Харилцах данс,asset_current,account_cashflow_tag_127,0,mn_chart_1
account_template_1205_0101,12050101,"Урьдчилж төлсөн ЭМ, НДШ",asset_prepayments,account_cashflow_tag_122,0,mn_chart_1
account_template_1205_0201,12050201,Урьдчилж төлсөн Сайн дурын даатгалын хураамж,asset_prepayments,account_cashflow_tag_128,0,mn_chart_1
account_template_1206_0101,12060101,"Цалингийн урьдчилгаа, богино хугацаат зээл",asset_receivable,account_cashflow_tag_215_225,1,mn_chart_1
account_template_1206_0201,12060201,"Төлбөр, дутагдал",asset_receivable,account_cashflow_tag_116,1,mn_chart_1
account_template_1206_0301,12060301,Худалдааны бус авлага ААН,asset_receivable,account_cashflow_tag_116,1,mn_chart_1
account_template_1206_0401,12060401,Худалдааны бус авлага Хувь хүн,asset_receivable,account_cashflow_tag_116,1,mn_chart_1
account_template_1206_0501,12060501,Худалдааны бус авлага Холбоотой тал,asset_receivable,account_cashflow_tag_116,1,mn_chart_1
account_template_1206_0601,12060601,Богино хугацаат зээлийн авлага ААН,asset_receivable,account_cashflow_tag_116,1,mn_chart_1
account_template_1206_0701,12060701,Богино хугацаат зээлийн авлага Хувь хүн,asset_receivable,account_cashflow_tag_116,1,mn_chart_1
account_template_1206_0801,12060801,Богино хугацаат зээлийн авлага Холбоотой тал,asset_receivable,account_cashflow_tag_116,1,mn_chart_1
account_template_1206_0901,12060901,Ногдол ашгийн авлага,asset_receivable,account_cashflow_tag_217,1,mn_chart_1
account_template_1206_1001,12061001,Барьцаалсан авлага,asset_receivable,account_cashflow_tag_116,1,mn_chart_1
account_template_1206_1101,12061101,Хүүний авлага,asset_receivable,account_cashflow_tag_116,1,mn_chart_1
account_template_1206_1201,12061201,Үндсэн хөрөнгө худалдааны авлага,asset_receivable,account_cashflow_tag_211_221,1,mn_chart_1
account_template_1206_1301,12061301,Биет бус хөрөнгө худалдааны авлага,asset_receivable,account_cashflow_tag_212_222,1,mn_chart_1
account_template_1206_1401,12061401,Хөрөнгө оруулалт худалдааны авлага,asset_receivable,account_cashflow_tag_213_223,1,mn_chart_1
account_template_1206_9901,12069901,Бусад худалдааны бус авлага,asset_receivable,account_cashflow_tag_116,1,mn_chart_1
account_template_1301_0101,13010101,Түргэн борлогдох үнэт цаас,asset_current,,0,mn_chart_1
account_template_1301_0201,13010201,Хугацаат хадгаламж,asset_current,,0,mn_chart_1
account_template_1302_0101,13020101,Бусад дэлгэрэнгүй орлого болох бодит үнэ цэнээр илэрхийлэх санхүүгийн хөрөнгө,asset_current,,0,mn_chart_1
account_template_1303_0101,13030101,Хорогдуулсан өртөгөөр илэрхийлэгдэх санхүүгийн өртөг,asset_current,,0,mn_chart_1
account_template_1401_0101,14010101,Агуулахад байгаа бараа материал,asset_current,,0,mn_chart_1
account_template_1401_0401,14010401,Татан авалтын өртөгийн зөрүү,asset_current,,0,mn_chart_1
account_template_1401_0701,14010701,Амьтан,asset_current,,0,mn_chart_1
account_template_1401_0801,14010801,Мал,asset_current,,0,mn_chart_1
account_template_1401_0901,14010901,Ургамал,asset_current,,0,mn_chart_1
account_template_1402_0101,14020101,Агуулахад байгаа түүхий эд материал,asset_current,,0,mn_chart_1
account_template_1403_0301,14030301,Агуулахад байгаа үндсэн хөрөнгийн хэсэг,asset_current,,0,mn_chart_1
account_template_1403_0401,14030401,Агуулахад байгаа хангамжийн материал,asset_current,,0,mn_chart_1
account_template_1403_0501,14030501,Ашиглалтанд байгаа хангамжийн материал,asset_current,,0,mn_chart_1
account_template_1403_0502,14030502,Ашиглалтанд байгаа хангамжийн материалын элэгдэл,asset_current,,0,mn_chart_1
account_template_1404_0101,14040101,Дуусаагүй үйлдвэрлэл - Шууд материал,asset_current,,0,mn_chart_1
account_template_1404_0201,14040201,Дуусаагүй үйлдвэрлэл - Шууд хөдөлмөр,asset_current,,0,mn_chart_1
account_template_1404_0301,14040301,Дуусаагүй үйлдвэрлэл - НЗ,asset_current,,0,mn_chart_1
account_template_1404_0401,14040401,Хагас боловсруулсан бүтээгдэхүүн,asset_current,,0,mn_chart_1
account_template_1405_0101,14050101,Агентад байгаа бараа,asset_current,,0,mn_chart_1
account_template_1406_0101,14060101,Замд яваа бараа,asset_current,,0,mn_chart_1
account_template_1407_0101,14070101,Ирж буй барааны өртөгийн түр данс,asset_current,,0,l10n_mn.mn_chart_1
account_template_1408_0101,14080101,Гарч буй барааны өртөгийн түр данс,asset_current,,0,l10n_mn.mn_chart_1
account_template_1501_0101,15010101,Урьдчилж төлсөн түрээс,asset_prepayments,account_cashflow_tag_124,0,mn_chart_1
account_template_1501_0201,15010201,Урьдчилж төлсөн хураамж,asset_prepayments,,0,mn_chart_1
account_template_1501_0301,15010301,Урьдчилж төлсөн цалин,asset_prepayments,,0,mn_chart_1
account_template_1501_0901,15010901,Урьдчилж төлсөн бусад зардал,asset_prepayments,account_cashflow_tag_129,0,mn_chart_1
account_template_1502_0101,15020101,Бэлтгэн нийлүүлэгчид төлсөн урьдчилгаа,asset_prepayments,account_cashflow_tag_123,0,mn_chart_1
account_template_1502_0201,15020201,"Дараа тайлангийн урьдчилгаа, томилолт",asset_prepayments,account_cashflow_tag_124,0,mn_chart_1
account_template_1502_0301,15020301,Үндсэн хөрөнгө худалдан авахад төлсөн урьдчилгаа,asset_prepayments,account_cashflow_tag_221,0,mn_chart_1
account_template_1502_0401,15020401,Биет бус хөрөнгө худалдан авахад төлсөн урьдчилгаа,asset_prepayments,account_cashflow_tag_222,0,mn_chart_1
account_template_1502_0501,15020501,Хөрөнгө оруулалт худалдан авахад төлсөн урьдчилгаа,asset_prepayments,account_cashflow_tag_223,0,mn_chart_1
account_template_1502_0601,15020601,Хангамжийн зүйлс худалдан авахад төлсөн урьдчилгаа,asset_prepayments,account_cashflow_tag_123,0,mn_chart_1
account_template_1502_0901,15020901,Бусад худалдааны бус урьдчилгаа,asset_prepayments,account_cashflow_tag_129,0,mn_chart_1
account_template_1601_0101,16010101,Бусад эргэлтийн хөрөнгө,asset_current,,0,mn_chart_1
account_template_1701_0101,17010101,Борлуулах зорилгоор эзэмшиж буй эргэлтийн бус хөрөнгө,asset_current,,0,mn_chart_1
account_template_2001_0101,20010101,Биет хөрөнгө - Газрын сайржуулалт,asset_fixed,,0,mn_chart_1
account_template_2001_0201,20010201,Биет хөрөнгө - Газрын сайжруулалтын элэгдэл,asset_fixed,,0,mn_chart_1
account_template_2002_0101,20020101,"Биет хөрөнгө - Барилга, байгууламж",asset_fixed,,0,mn_chart_1
account_template_2002_0201,20020201,"Биет хөрөнгө - Барилга, байгууламжын элэгдэл",asset_fixed,,0,mn_chart_1
account_template_2003_0101,20030101,Биет хөрөнгө - Машин тоног төхөөрөмж,asset_fixed,,0,mn_chart_1
account_template_2003_0201,20030201,Биет хөрөнгө - Машин тоног төхөөрөмжийн элэгдэл,asset_fixed,,0,mn_chart_1
account_template_2004_0101,20040101,Биет хөрөнгө - Тээврийн хэрэгсэл,asset_fixed,,0,mn_chart_1
account_template_2004_0201,20040201,Биет хөрөнгө - Тээврийн хэрэгслийн элэгдэл,asset_fixed,,0,mn_chart_1
account_template_2005_0101,20050101,Биет хөрөнгө - Тавилга эд хогшил,asset_fixed,,0,mn_chart_1
account_template_2005_0201,20050201,Биет хөрөнгө - Тавилга эд хогшлын элэгдэл,asset_fixed,,0,mn_chart_1
account_template_2006_0101,20060101,"Биет хөрөнгө - Компьютер, дагалдах хэрэгсэл",asset_fixed,,0,mn_chart_1
account_template_2006_0201,20060201,"Биет хөрөнгө - Компьютер, дагалдах хэрэгслийн элэгдэл",asset_fixed,,0,mn_chart_1
account_template_2008_0101,20080101,Биет хөрөнгө - Дуусаагүй барилга,asset_fixed,,0,mn_chart_1
account_template_2008_0201,20080201,Биет хөрөнгө - Дуусаагүй барилгын элэгдэл,asset_fixed,,0,mn_chart_1
account_template_2009_0101,20090101,Биет хөрөнгө - Бусад хөрөнгө,asset_fixed,,0,mn_chart_1
account_template_2009_0201,20090201,Биет хөрөнгө - Бусад хөрөнгийн элэгдэл,asset_fixed,,0,mn_chart_1
account_template_2101_0101,21010101,Биет бус хөрөнгө - Зохиогчийн эрх,asset_fixed,,0,mn_chart_1
account_template_2101_0201,21010201,Биет бус хөрөнгө - Зохиогчийн эрхийн хорогдол,asset_fixed,,0,mn_chart_1
account_template_2102_0101,21020101,Биет бус хөрөнгө - Програм хангамж,asset_fixed,,0,mn_chart_1
account_template_2102_0201,21020201,Биет бус хөрөнгө - Програм хангамжын хорогдол,asset_fixed,,0,mn_chart_1
account_template_2103_0101,21030101,Биет бус хөрөнгө - Патент,asset_fixed,,0,mn_chart_1
account_template_2103_0201,21030201,Биет бус хөрөнгө - Патентын хорогдол,asset_fixed,,0,mn_chart_1
account_template_2104_0101,21040101,Биет бус хөрөнгө - Барааны тэмдэг,asset_fixed,,0,mn_chart_1
account_template_2104_0201,21040201,Биет бус хөрөнгө - Барааны тэмдэгийн хорогдол,asset_fixed,,0,mn_chart_1
account_template_2105_0101,21050101,Биет бус хөрөнгө - Тусгай зөвшөөрөл,asset_fixed,,0,mn_chart_1
account_template_2105_0201,21050201,Биет бус хөрөнгө - Тусгай зөвшөөрлийн хорогдол,asset_fixed,,0,mn_chart_1
account_template_2109_0101,21090101,Биет бус хөрөнгө - Бусад хөрөнгө,asset_fixed,,0,mn_chart_1
account_template_2109_0201,21090201,Биет бус хөрөнгө - Бусад хөрөнгийн элэгдэл,asset_fixed,,0,mn_chart_1
account_template_2401_0101,24010101,Биологийн хөрөнгө - Мал амьтад,asset_fixed,,0,mn_chart_1
account_template_2401_0201,24010201,Биологийн хөрөнгө - Мал амьтадын хорогдол,asset_fixed,,0,mn_chart_1
account_template_2402_0101,24020101,Биологийн хөрөнгө - Ургамал,asset_fixed,,0,mn_chart_1
account_template_2402_0201,24020201,Биологийн хөрөнгө - Ургамалын хорогдол,asset_fixed,,0,mn_chart_1
account_template_2201_0101,22010101,Ашиг алдагдлаарх бодит үнэ цэнээр илэрхийлэх санхүүгийн хөрөнгө,asset_non_current,account_cashflow_tag_211_221,0,mn_chart_1
account_template_2204_0101,22040101,"Хараат компани, хамтарсан үйлдвэрт оруулсан хөрөнгө оруулалт",asset_non_current,account_cashflow_tag_211_221,0,mn_chart_1
account_template_2205_0101,22050101,Охин компанид оруулсан хөрөнгө оруулалт,asset_non_current,account_cashflow_tag_211_221,0,mn_chart_1
account_template_2301_0101,23010101,Хайгуул ба үнэлгээний хөрөнгө,asset_non_current,account_cashflow_tag_211_221,0,mn_chart_1
account_template_2501_0101,25010101,Хойшлогдсон татварын хөрөнгө,asset_non_current,,0,mn_chart_1
account_template_2601_0101,26010101,Хөрөнгө оруулалтын зориулалттай үндсэн хөрөнгө,asset_non_current,account_cashflow_tag_211_221,0,mn_chart_1
account_template_2601_0201,26010201,Үндсэн хөрөнгийн үнэ цэнийн бууралт,asset_non_current,,0,mn_chart_1
account_template_2701_0101,27010101,Урт хугацаат зээлийн авлага ААН,asset_non_current,account_cashflow_tag_212_222,0,mn_chart_1
account_template_2701_0201,27010201,Урт хугацаат зээлийн авлага Хувь хүн,asset_non_current,account_cashflow_tag_212_222,0,mn_chart_1
account_template_2701_0301,27010301,Урт хугацаат зээлийн авлага Холбоотой тал,asset_non_current,account_cashflow_tag_212_222,0,mn_chart_1
account_template_2709_0101,27090101,Бусад эргэлтийн бус хөрөнгө,asset_non_current,,0,mn_chart_1
account_template_3101_0101,31010101,Худалдааны БН-д өгөх өглөг ААН,liability_payable,account_cashflow_tag_123,1,mn_chart_1
account_template_3101_0201,31010201,Худалдааны БН-д өгөх өглөг Хувь хүн,liability_payable,account_cashflow_tag_123,1,mn_chart_1
account_template_3101_0301,31010301,Худалдааны БН-д өгөх өглөг Холбоотой тал,liability_payable,account_cashflow_tag_123,1,mn_chart_1
account_template_3201_0101,32010101,Урьдчилж орсон худалдааны орлого,liability_current,account_cashflow_tag_123,1,mn_chart_1
account_template_3202_0101,32020101,Урьдчилж орсон түрээсийн орлого,liability_current,account_cashflow_tag_123,1,mn_chart_1
account_template_3203_0101,32030101,Урьдчилж орсон зээлийн хүүгийн орлого,liability_current,account_cashflow_tag_129,1,mn_chart_1
account_template_3301_0101,33010101,Цалингийн өглөг,liability_payable,account_cashflow_tag_121,1,mn_chart_1
account_template_3401_0101,34010101,Татварын өглөг - ААНОАТатвар,liability_current,account_cashflow_tag_127,0,mn_chart_1
account_template_3401_0201,34010201,Татварын өглөг - НӨАТатвар,liability_current,account_cashflow_tag_127,0,mn_chart_1
account_template_3401_0301,34010301,Татварын өглөг - ХХОАТатвар,liability_current,account_cashflow_tag_127,0,mn_chart_1
account_template_3401_0401,34010401,Татварын өглөг - Гаалийн албан татвар,liability_current,account_cashflow_tag_127,0,mn_chart_1
account_template_3401_0501,34010501,Татварын өглөг - Үл хөдлөх хөрөнгийн албан татвар,liability_current,account_cashflow_tag_127,0,mn_chart_1
account_template_3401_0601,34010601,Татварын өглөг - Газрын албан татвар,liability_current,account_cashflow_tag_127,0,mn_chart_1
account_template_3401_0701,34010701,Татварын өглөг - АТӨЯХАТатвар,liability_current,account_cashflow_tag_127,0,mn_chart_1
account_template_3401_0801,34010801,Татварын өглөг - Агаарын бохирдлын татвар,liability_current,account_cashflow_tag_127,0,mn_chart_1
account_template_3401_0901,34010901,Татварын өглөг - Онцгой албан татвар,liability_current,account_cashflow_tag_127,0,mn_chart_1
account_template_3401_1001,34011001,Татварын өглөг - Байгалийн нөөц ашиглалтын татвар,liability_current,account_cashflow_tag_127,0,mn_chart_1
account_template_3401_1101,34011101,Татварын өглөг – НХАТатвар,liability_current,account_cashflow_tag_127,0,mn_chart_1
account_template_3401_9901,34019901,Татварын өглөг - Бусад татвар,liability_current,account_cashflow_tag_127,0,mn_chart_1
account_template_3401_9902,34019902,Татварын өглөг - Харилцах данс,liability_current,account_cashflow_tag_127,0,mn_chart_1
account_template_3402_0101,34020101,"ЭМ, НДШ-ийн өглөг",liability_current,account_cashflow_tag_122,0,mn_chart_1
account_template_3402_0201,34020201,Сайн дурын даатгалын өглөг,liability_current,account_cashflow_tag_128,0,mn_chart_1
account_template_3501_0101,35010101,Банкны богино хугацаат зээлийн өглөг,liability_payable,account_cashflow_tag_127,1,mn_chart_1
account_template_3501_0201,35010201,ББСБ-ын богино хугацаат зээлийн өглөг,liability_payable,account_cashflow_tag_127,1,mn_chart_1
account_template_3501_0301,35010301,Хувь хүний богино хугацаат зээлийн өглөг,liability_payable,account_cashflow_tag_127,1,mn_chart_1
account_template_3502_0101,35020101,Банкны богино хугацаат зээлийн хүү,liability_current,l10n_mn.account_cashflow_tag_127,0,l10n_mn.mn_chart_1
account_template_3502_0201,35020201,ББСБ-ын богино хугацаат зээлийн хүү,liability_current,l10n_mn.account_cashflow_tag_127,0,l10n_mn.mn_chart_1
account_template_3502_0301,35020301,Хувь хүний богино хугацаат зээлийн хүү,liability_current,l10n_mn.account_cashflow_tag_127,0,l10n_mn.mn_chart_1
account_template_3601_0101,36010101,Ногдол ашгийн өглөг,liability_current,,0,mn_chart_1
account_template_3701_0101,37010101,Нөөц өр төлбөр,liability_current,account_cashflow_tag_222,0,mn_chart_1
account_template_3801_0101,38010101,Даатгалын өглөг,liability_payable,account_cashflow_tag_128,1,mn_chart_1
account_template_3802_0101,38020101,Гишүүнчлэлийн өглөг,liability_payable,account_cashflow_tag_129,1,mn_chart_1
account_template_3803_0101,38030101,Ажилчдад өгөх өглөг,liability_payable,account_cashflow_tag_121,1,mn_chart_1
account_template_3804_0101,38040101,Хувь хүмүүст өгөх өглөг,liability_payable,account_cashflow_tag_129,1,mn_chart_1
account_template_3805_0101,38050101,"Баталгаа, барьцааны өглөг",liability_payable,account_cashflow_tag_129,1,mn_chart_1
account_template_3806_0101,38060101,Худалдааны бус бэлтгэн нийлүүлэгчид өгөх өглөг - ААН,liability_payable,account_cashflow_tag_129,1,mn_chart_1
account_template_3806_0201,38060201,Худалдааны бус бэлтгэн нийлүүлэгчид өгөх өглөг - Хувь хүн,liability_payable,account_cashflow_tag_129,1,mn_chart_1
account_template_3806_0301,38060301,Худалдааны бус бэлтгэн нийлүүлэгчид өгөх өглөг - Холбоотой тал,liability_payable,account_cashflow_tag_129,1,mn_chart_1
account_template_3806_0401,38060401,Худалдан авсан үндсэн хөрөнгийн өглөг,liability_payable,account_cashflow_tag_128,1,mn_chart_1
account_template_3806_0501,38060501,Худалдан авсан хөрөнгө оруулалтын өглөг,liability_payable,account_cashflow_tag_128,1,mn_chart_1
account_template_3809_0101,38090101,Бусад богино хугацаат өр төлбөр,liability_current,account_cashflow_tag_129,0,mn_chart_1
account_template_3901_0101,39010101,Борлуулах зорилгоор эзэмшиж буй эргэлтийн бус хөрөнгөнд хамаарах өр,liability_current,account_cashflow_tag_222,0,mn_chart_1
account_template_4001_0101,40010101,Банкны урт хугацаат зээл,liability_non_current,account_cashflow_tag_212_222,0,mn_chart_1
account_template_4001_0201,40010201,ББСБ-ын урт хугацаат зээл,liability_non_current,account_cashflow_tag_212_222,0,mn_chart_1
account_template_4001_0301,40010301,Хувь хүнээс авсан урт хугацаат зээл,liability_non_current,account_cashflow_tag_212_222,0,mn_chart_1
account_template_4001_0401,40010401,Холбоотой талын урт хугацаат зээл,liability_non_current,account_cashflow_tag_212_222,0,mn_chart_1
account_template_4002_0101,40020101,Урт хугацаат нөөц өр төлбөр,liability_non_current,account_cashflow_tag_212_222,0,mn_chart_1
account_template_4003_0101,40030101,Хойшлогдсон татварын өр,liability_non_current,account_cashflow_tag_212_222,0,mn_chart_1
account_template_4009_0101,40090101,Бусад урт хугацаат өр төлбөр,liability_non_current,account_cashflow_tag_212_222,0,mn_chart_1
account_template_4101_0101,41010101,Эздийн өмч - төрийн,equity,,0,mn_chart_1
account_template_4102_0101,41020101,Эздийн өмч - хувийн,equity,,0,mn_chart_1
account_template_4103_0101,41030101,Эздийн өмч - хувьцаат,equity,,0,mn_chart_1
account_template_4201_0101,42010101,Халаасны хувьцаа,equity,,0,mn_chart_1
account_template_4301_0101,43010101,Нэмж төлөгдсөн капитал,equity,,0,mn_chart_1
account_template_4401_0101,44010101,Хөрөнгийн дахин үнэлгээний нэмэгдэл,equity,,0,mn_chart_1
account_template_4501_0101,45010101,Гадаад валютын хөрвүүлэлтийн нөөц,equity,,0,mn_chart_1
account_template_4509_0101,45090101,Эздийн өмчийн бусад хэсэг,equity,,0,mn_chart_1
account_template_4601_0101,46010101,Тайлант үеийн хуримтлагдсан ашиг,equity_unaffected,,0,mn_chart_1
account_template_5101_0101,51010101,Борлуулалтын орлого,income,account_cashflow_tag_111,0,mn_chart_1
account_template_5101_0201,51010201,Борлуулалтын орлого (НӨАТ-с чөлөөлөгдөх),income,account_cashflow_tag_111,0,mn_chart_1
account_template_5102_0101,51020101,Түрээсийн орлого,income_other,account_cashflow_tag_111,0,mn_chart_1
account_template_5103_0101,51030101,Хүүний орлого,income_other,account_cashflow_tag_127,0,mn_chart_1
account_template_5104_0101,51040101,Ногдол ашгийн орлого,income_other,account_cashflow_tag_214_224,0,mn_chart_1
account_template_5105_0101,51050101,Эрхийн шимтгэлийн орлого,income_other,account_cashflow_tag_112,0,mn_chart_1
account_template_5201_0101,52010101,Тооллогын зөрүүний олз,income_other,,0,mn_chart_1
account_template_5201_0201,52010201,Тооллогын зөрүүний алдагдал,income_other,,0,mn_chart_1
account_template_5201_0901,52010901,Бусад дэлгэрэнгүй орлого,income_other,account_cashflow_tag_116,0,mn_chart_1
account_template_5202_0101,52020101,Үнэ шилжилтийн хоёрдогч тохируулгын үр дүнд бий болсон үнийн зөрүү,income_other,,0,l10n_mn.mn_chart_1
account_template_5203_0101,52030101,"Гадаад, дотоодын үнэт цаасны анхдагч болон хоёрдогч зах зээлд нээлттэй арилжаалах өрийн хэрэгсэл, нэгж эрх худалдан авсан албан татвар төлөгчид олгосон хүүгийн орлого",income_other,account_cashflow_tag_129,0,l10n_mn.mn_chart_1
account_template_5204_0101,52040101,"Монгол Улсын арилжааны банкны дотоодын эх үүсвэрээс татсан зээл, өрийн хэрэгсэлд олгосон хүүгийн орлого",income_other,account_cashflow_tag_129,0,l10n_mn.mn_chart_1
account_template_5205_0101,52050101,Төлөөний газраар дамжуулан Монгол Улсад үйл ажиллагаа явуулдаг Монгол Улсад байрладаггүй албан татвар төлөгчид тухайн төлөөний газраас шилжүүлсэн орлого,income_other,,0,l10n_mn.mn_chart_1
account_template_5205_0201,52050201,Төлөөний газрын үйл ажиллагаатай холбогдох орлогыг Монгол Улсад байрладаггүй этгээдэд шилжүүлсэн орлого,income_other,,0,l10n_mn.mn_chart_1
account_template_5205_0301,52050301,Тухайн татварын жилд төлөөний газраас өөрийн толгой аж ахуйн нэгжид шилжүүлсэн ашиг,income_other,,0,l10n_mn.mn_chart_1
account_template_5205_0401,52050401,Газрын тосны бүтээгдэхүүний өөрт ногдох борлуулалтаас гадаадад шилжүүлсэн чөлөөлөгдөх орлого,income_other,,0,l10n_mn.mn_chart_1
account_template_5205_0501,52050501,Байгаль орчинд нөлөөлөх байдлын үнэлгээний тухай хуулийн дагуу буцаан олголтын орлого,income_other,,0,l10n_mn.mn_chart_1
account_template_5205_0601,52050601,Даатгалын нөхөн төлбөрийн орлого,income_other,account_cashflow_tag_113,0,l10n_mn.mn_chart_1
account_template_5301_0101,53010101,Валютын ханшийн зөрүүний хэрэгжсэн олз,income_other,account_cashflow_tag_400,0,mn_chart_1
account_template_5301_0201,53010201,Валютын ханшийн зөрүүний хэрэгжээгүй олз,income_other,account_cashflow_tag_400,0,mn_chart_1
account_template_5302_0101,53020101,Валютын ханшийн зөрүүний хэрэгжсэн гарз,income_other,account_cashflow_tag_400,0,mn_chart_1
account_template_5302_0201,53020201,Валютын ханшийн зөрүүний хэрэгжээгүй гарз,income_other,account_cashflow_tag_400,0,mn_chart_1
account_template_5401_0101,54010101,Үндсэн хөрөнгө борлуулсны олз,income_other,,0,mn_chart_1
account_template_5401_0201,54010201,Үндсэн хөрөнгө борлуулсны гарз,income_other,,0,mn_chart_1
account_template_5402_0101,54020101,Барилга байгууламжын хөрөнгө борлуулсны түр данс,income_other,,0,l10n_mn.mn_chart_1
account_template_5402_0201,54020201,Бусад үндсэн хөрөнгө борлуулсны түр данс,income_other,,0,l10n_mn.mn_chart_1
account_template_5501_0101,55010101,Биет бус хөрөнгө борлуулсны олз,income_other,,0,mn_chart_1
account_template_5501_0201,55010201,Биет бус хөрөнгө борлуулсны гарз,income_other,,0,mn_chart_1
account_template_5502_0101,55020101,Биет бус хөрөнгө борлуулсны түр данс,income_other,,0,l10n_mn.mn_chart_1
account_template_5601_0101,56010101,Хөрөнгө оруулалт борлуулсны олз,income_other,,0,mn_chart_1
account_template_5601_0201,56010201,Хөрөнгө оруулалт борлуулсны гарз,income_other,,0,mn_chart_1
account_template_5701_0101,57010101,Хугацаанаасаа өмнө төлсний хөнгөлөлтийн олз,income_other,,0,mn_chart_1
account_template_5701_0201,57010201,Хугацаанаасаа өмнө төлсний хөнгөлөлтийн алдагдал,income_other,,0,mn_chart_1
account_template_5901_0101,59010101,Үндсэн бус үйл ажиллагааны бусад олз,income_other,,0,mn_chart_1
account_template_5901_0201,59010201,Үндсэн бус үйл ажиллагааны бусад гарз,income_other,,0,mn_chart_1
account_template_6001_0101,60010101,Борлуулалтын хөнгөлөлт,income,,0,mn_chart_1
account_template_6002_0101,60020101,Борлуулалтын урамшуулал,income,,0,mn_chart_1
account_template_6003_0101,60030101,Борлуулалтын буцаалт,income,,0,mn_chart_1
account_template_6004_0101,60040101,Бэлнээр төлсөний хөнгөлөлт,income,,0,mn_chart_1
account_template_6005_0101,60050101,Хугацаанаас өмнө төлсний хөнгөлөлт,income,,0,mn_chart_1
account_template_6006_0101,60060101,Борлуулалтын үнийн бууралт,income,,0,mn_chart_1
account_template_6101_0101,61010101,"Борлуулсан бараа, бүтээгдэхүүний өртөг",expense_direct_cost,,0,mn_chart_1
account_template_6102_0101,61020101,Борлуулсан үйлчилгээний өртөг,expense_direct_cost,,0,mn_chart_1
account_template_7001_0101,70010101,Удирд - Үндсэн цалингийн зардал,expense,,0,mn_chart_1
account_template_7001_0201,70010201,Удирд - Нэмэгдэл цалингийн зардал,expense,,0,mn_chart_1
account_template_7002_0101,70020101,Удирд - ЭМД-ийн зардал,expense,,0,mn_chart_1
account_template_7002_0201,70020201,Удирд - НДШ-ийн зардал,expense,,0,mn_chart_1
account_template_7002_0301,70020301,Удирд - Банкны шимтгэл хураамжын зардал,expense,,0,mn_chart_1
account_template_7002_0401,70020401,Удирд - Гадаад томилолтын зардал,expense,,0,mn_chart_1
account_template_7002_0402,70020402,Удирд - Дотоод томилолтын зардал,expense,,0,mn_chart_1
account_template_7002_0501,70020501,Удирд - Бичиг хэргийн зардал,expense,,0,mn_chart_1
account_template_7002_0601,70020601,Удирд - Шуудан холбооны зардал,expense,,0,mn_chart_1
account_template_7002_0602,70020602,Удирд - Суурин утасны зардал,expense,,0,mn_chart_1
account_template_7002_0603,70020603,Удирд - Үүрэн утасны зардал,expense,,0,mn_chart_1
account_template_7002_0604,70020604,Удирд - Интернетийн зардал,expense,,0,mn_chart_1
account_template_7002_0701,70020701,Удирд - Мэргэжлийн үйлчилгээний зардал,expense,,0,mn_chart_1
account_template_7002_0801,70020801,Удирд - Хүний нөөцийн зардал,expense,,0,mn_chart_1
account_template_7002_0901,70020901,Удирд - Сонин сэтгүүл захиалгын зардал,expense,,0,mn_chart_1
account_template_7002_1001,70021001,Удирд - Даатгалын зардал,expense,,0,mn_chart_1
account_template_7002_1101,70021101,Удирд - Ашиглалтын зардал,expense,,0,mn_chart_1
account_template_7002_1201,70021201,Удирд - Засварын зардал,expense,,0,mn_chart_1
account_template_7002_1301,70021301,Удирд - Элэгдэл хорогдлын зардал,expense,,0,mn_chart_1
account_template_7002_1401,70021401,Удирд - Түрээсийн зардал,expense,account_cashflow_tag_124,0,mn_chart_1
account_template_7002_1501,70021501,Удирд - Харуул хамгаалалтын зардал,expense,account_cashflow_tag_124,0,mn_chart_1
account_template_7002_1601,70021601,Удирд - Цэвэрлэгээ үйлчилгээний зардал,expense,account_cashflow_tag_124,0,mn_chart_1
account_template_7002_1701,70021701,Удирд - Тээврийн зардал,expense,account_cashflow_tag_125,0,mn_chart_1
account_template_7002_1801,70021801,Удирд - Шатахууны зардал,expense,account_cashflow_tag_125,0,mn_chart_1
account_template_7002_1901,70021901,Удирд - Хүлээн авалтын зардал,expense,,0,mn_chart_1
account_template_7002_2001,70022001,Удирд - Зар сурталчилгааны зардал,expense,,0,mn_chart_1
account_template_7002_2101,70022101,Удирд - Төсөл хөгжлийн зардал,expense,,0,mn_chart_1
account_template_7002_2201,70022201,"Удирд - Судалгаа, туршилтын зардал",expense,,0,mn_chart_1
account_template_7002_2301,70022301,"Удирд - Хүргэлт, түгээлтийн зардал",expense,,0,mn_chart_1
account_template_7002_9901,70029901,Удирд - Үйл ажиллагааны бусад зардал,expense,,0,mn_chart_1
account_template_7101_0101,71010101,Борл - Үндсэн цалингийн зардал,expense,,0,mn_chart_1
account_template_7101_0201,71010201,Борл - Нэмэгдэл цалингийн зардал,expense,,0,mn_chart_1
account_template_7102_0101,71020101,Борл - ЭМД-ийн зардал,expense,,0,mn_chart_1
account_template_7102_0201,71020201,Борл - НДШ-ийн зардал,expense,,0,mn_chart_1
account_template_7102_0301,71020301,Борл - Банкны шимтгэл хураамжын зардал,expense,,0,mn_chart_1
account_template_7102_0401,71020401,Борл - Гадаад томилолтын зардал,expense,,0,mn_chart_1
account_template_7102_0402,71020402,Борл - Дотоод томилолтын зардал,expense,,0,mn_chart_1
account_template_7102_0501,71020501,Борл - Бичиг хэргийн зардал,expense,,0,mn_chart_1
account_template_7102_0601,71020601,Борл - Шуудан холбооны зардал,expense,,0,mn_chart_1
account_template_7102_0602,71020602,Борл - Суурин утасны зардал,expense,,0,mn_chart_1
account_template_7102_0603,71020603,Борл - Үүрэн утасны зардал,expense,,0,mn_chart_1
account_template_7102_0604,71020604,Борл - Интернетийн зардал,expense,,0,mn_chart_1
account_template_7102_0701,71020701,Борл - Мэргэжлийн үйлчилгээний зардал,expense,,0,mn_chart_1
account_template_7102_0801,71020801,Борл - Хүний нөөцийн зардал,expense,,0,mn_chart_1
account_template_7102_0901,71020901,Борл - Сонин сэтгүүл захиалгын зардал,expense,,0,mn_chart_1
account_template_7102_1001,71021001,Борл - Даатгалын зардал,expense,,0,mn_chart_1
account_template_7102_1101,71021101,Борл - Ашиглалтын зардал,expense,,0,mn_chart_1
account_template_7102_1201,71021201,Борл - Засварын зардал,expense,,0,mn_chart_1
account_template_7102_1301,71021301,Борл - Элэгдэл хорогдлын зардал,expense,,0,mn_chart_1
account_template_7102_1401,71021401,Борл - Түрээсийн зардал,expense,,0,mn_chart_1
account_template_7102_1501,71021501,Борл - Харуул хамгаалалтын зардал,expense,,0,mn_chart_1
account_template_7102_1601,71021601,Борл - Цэвэрлэгээ үйлчилгээний зардал,expense,,0,mn_chart_1
account_template_7102_1701,71021701,Борл - Тээврийн зардал,expense,account_cashflow_tag_125,0,mn_chart_1
account_template_7102_1801,71021801,Борл - Шатахууны зардал,expense,account_cashflow_tag_125,0,mn_chart_1
account_template_7102_1901,71021901,Борл - Хүлээн авалтын зардал,expense,,0,mn_chart_1
account_template_7102_2001,71022001,Борл - Зар сурталчилгааны зардал,expense,,0,mn_chart_1
account_template_7102_2101,71022101,Борл - Төсөл хөгжлийн зардал,expense,,0,mn_chart_1
account_template_7102_2201,71022201,"Борл - Судалгаа, туршилтын зардал",expense,,0,mn_chart_1
account_template_7102_2301,71022301,"Борл - Хүргэлт, түгээлтийн зардал",expense,,0,mn_chart_1
account_template_7102_9901,71029901,Борл - Үйл ажиллагааны бусад зардал,expense,,0,mn_chart_1
account_template_7201_0101,72010101,Эргэлтийн хөрөнгө санхүүжүүлэх зээлийн хүүгийн зардал,expense,account_cashflow_tag_129,0,mn_chart_1
account_template_7202_0101,72020101,Үндсэн хөрөнгө санхүүжүүлэх зээлийн хүүгийн зардал,expense,account_cashflow_tag_129,0,mn_chart_1
account_template_7203_0101,72030101,Төсөл санхүүжүүлэх зээлийн хүүгийн зардал,expense,account_cashflow_tag_129,0,mn_chart_1
account_template_7301_0101,73010101,"Хайгуул, үнэлгээний зардал",expense,,0,mn_chart_1
account_template_7401_0101,74010101,"Алданги, торгуулийн зардал",expense,,0,mn_chart_1
account_template_7402_0101,74020101,"Хандив, тусламжийн зардал",expense,,0,mn_chart_1
account_template_7403_0101,74030101,Найдваргүй авлагын зардал,expense,,0,mn_chart_1
account_template_7404_0101,74040101,Болзошгүй алдагдлын санд төвлөрүүлсэн хөрөнгийн зардал,expense,,0,mn_chart_1
account_template_7405_0101,74050101,Франчайзын зардал,expense,,0,mn_chart_1
account_template_7406_0101,74060101,Банкны үйлчилгээний шимтгэлийн зардал,expense,,0,mn_chart_1
account_template_7407_0101,74070101,Хөрөнгө акталж устгасны зардал,expense,,0,mn_chart_1
account_template_7408_0101,74080101,Үйлдвэрийн ослын зардал,expense,,0,mn_chart_1
account_template_7409_0101,74090101,"Туслах үйлдвэрлэл, үйлчилгээний зардал",expense,,0,mn_chart_1
account_template_7410_0101,74100101,Үндсэн хөрөнгийн үнэ цэнийн бууралтын зардал,expense,,0,mn_chart_1
account_template_7499_0101,74990101,Бусад ерөнхий зардал,expense,,0,mn_chart_1
account_template_9101_0101,91010101,Тайлант үеийн орлогын албан татварын зардал,expense,,0,mn_chart_1
account_template_9102_0101,91020101,Хойшлогдсон татварын зардал,expense,,0,mn_chart_1
account_template_9103_0101,91030101,Татвар ногдохгүй бусад дэлгэрэнгүй орлого,expense,,0,mn_chart_1
account_template_9301_0101,93010101,Эхний үлдэгдлийн эх үүсвэр,expense,,0,mn_chart_1

```

## File: data\account.chart.template.csv

```csv
id,property_account_receivable_id/id,property_account_payable_id/id,property_account_expense_categ_id/id,property_account_income_categ_id/id,income_currency_exchange_account_id/id,expense_currency_exchange_account_id/id,property_stock_account_input_categ_id/id,property_stock_account_output_categ_id/id,property_stock_valuation_account_id/id,default_pos_receivable_account_id/id
mn_chart_1,account_template_1201_0201,account_template_3101_0201,account_template_6101_0101,account_template_5101_0101,account_template_5301_0201,account_template_5302_0201,account_template_1407_0101,account_template_1408_0101,account_template_1401_0101,account_template_1201_0202

```

## File: data\account.tax.group.csv

```csv
id,name,sequence,country_id/id
account_tax_group1,НӨАТ ногдох бараа,10,base.mn
account_tax_group2,НӨАТ ногдох үйлчилгээ,10,base.mn
account_tax_group3,Экспортын борлуулалт,11,base.mn
account_tax_group4,"Худалдан авсан бараа, үйлчилгээ",12,base.mn
account_tax_group5,Санхүүгийн түрээсийн зүйл,13,base.mn
account_tax_group6,НӨАТ-с чөлөөлөгдөх,14,base.mn

```

## File: data\account_chart_template_configuration_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">

    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_mn.mn_chart_1')]"/>
    </function>

</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="mn_chart_1" model="account.chart.template">
        <field name="name">Mongolia</field>
        <field name="bank_account_code_prefix">11</field>
        <field name="cash_account_code_prefix">10</field>
        <field name="transfer_account_code_prefix">1109</field>
        <field name="code_digits">8</field>
        <field name="currency_id" ref="base.MNT"/>
        <field name="use_anglo_saxon" eval="True"/>
        <field name="country_id" ref="base.mn"/>
    </record>

</odoo>

```

## File: data\account_fiscal_position_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- = = = = = = = = = = = = = = = -->
    <!-- Fiscal Position Templates     -->
    <!-- = = = = = = = = = = = = = = = -->
    <record id="fiscal_position_vat0" model="account.fiscal.position.template">
        <field name="name">Дотоодын борлуулалт - Худалдан авалт</field>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="auto_apply" eval="True" />
        <field name="country_id" ref="base.mn"/>
    </record>

    <record id="fiscal_position_vat2" model="account.fiscal.position.template">
        <field name="name">НӨАТ 0% (3000000)</field>
        <field name="chart_template_id" ref="mn_chart_1"/>
    </record>

    <record id="fiscal_position_vat3" model="account.fiscal.position.template">
        <field name="name">НӨАТ-гүй (2000000)</field>
        <field name="chart_template_id" ref="mn_chart_1"/>
    </record>

    <record id="fiscal_position_vat4" model="account.fiscal.position.template">
        <field name="name">Гадаад борлуулалт - Худалдан авалт</field>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="auto_apply" eval="True" />
    </record>

    <!-- = = = = = = = = = = = = = = = -->
    <!-- Fiscal Position Tax Templates -->
    <!-- = = = = = = = = = = = = = = = -->

    <record id="fp_tax_template_vat2_sale_vat1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat1" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat4" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat5" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat6" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat7" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat9" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat10" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat11" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat11" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat12" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat12" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

        <record id="fp_tax_template_vat2_sale_vat13" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat13" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat14" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat15" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat15" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat16" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat16" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat17" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat17" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat18" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat18" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat19" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat19" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat20" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat20" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat21" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat2_sale_vat22" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat2" />
        <field name="tax_src_id" ref="account_tax_sale_vat22" />
        <field name="tax_dest_id" ref="account_tax_sale_vat2" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat1" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat4" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat5" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat6" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat7" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat9" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat10" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat11" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat11" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat12" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat12" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat13" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat13" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat14" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat15" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_dest_id" ref="account_tax_sale_vat15" />
        <field name="tax_src_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat16" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat16" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat17" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat17" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat18" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat18" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat19" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat19" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat20" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat20" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat21" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat3_sale_vat22" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat3" />
        <field name="tax_src_id" ref="account_tax_sale_vat22" />
        <field name="tax_dest_id" ref="account_tax_sale_vat3" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat1" />
        <field name="tax_dest_id" ref="account_tax_sale_vat23" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat4" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat4" />
        <field name="tax_dest_id" ref="account_tax_sale_vat23" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat5" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat5" />
        <field name="tax_dest_id" ref="account_tax_sale_vat23" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat6" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat6" />
        <field name="tax_dest_id" ref="account_tax_sale_vat23" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat7" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat7" />
        <field name="tax_dest_id" ref="account_tax_sale_vat23" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat9" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat9" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat10" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat10" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat11" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat11" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat12" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat12" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat13" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat13" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat14" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat14" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat15" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_dest_id" ref="account_tax_sale_vat15" />
        <field name="tax_src_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat16" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat16" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat17" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat17" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat18" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat18" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat19" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat19" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat20" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat20" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat21" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat21" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_sale_vat22" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_sale_vat22" />
        <field name="tax_dest_id" ref="account_tax_sale_vat24" />
    </record>

    <record id="fp_tax_template_vat4_purchase_vat1" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_vat4" />
        <field name="tax_src_id" ref="account_tax_purchase_vat1" />
        <field name="tax_dest_id" ref="account_tax_purchase_vat2" />
    </record>

</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="account_tax_sale_vat1" model="account.tax.template">
        <field name="name">НӨАТ бараа</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">1</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group1"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag5')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag5')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag5')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag5')],
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat1" model="account.tax.template">
        <field name="name">НӨАТ-тэй худалдан авалт</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">1</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag36'),ref('vat_report_tag33')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag36')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag36'),ref('vat_report_tag33')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag36')],
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat2" model="account.tax.template">
        <field name="name">НӨАТ-гүй худалдан авалт</field>
        <field name="amount">0.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">0%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="account_tax_sale_vat2" model="account.tax.template">
        <field name="name">НӨАТ 0% борлуулалт</field>
        <field name="amount">0.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">0%</field>
        <field name="tax_group_id" ref="account_tax_group6"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag2')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag2')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="account_tax_sale_vat3" model="account.tax.template">
        <field name="name">НӨАТ чөлөөлөх борлуулалт</field>
        <field name="amount">0.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">10</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">0%</field>
        <field name="tax_group_id" ref="account_tax_group6"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag2')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag2')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="account_tax_sale_vat4" model="account.tax.template">
        <field name="name">НӨАТ бусад барааны борлуулалт</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group1"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag6')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag6')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag6')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag6')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat5" model="account.tax.template">
        <field name="name">НӨАТ эрх борлуулалт</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group1"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag7')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag7')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag7')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag7')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat6" model="account.tax.template">
        <field name="name">НӨАТ татан буугдах үеийн борлуулалт</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group1"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag8')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag8')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag8')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag8')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat7" model="account.tax.template">
        <field name="name">НӨАТ өрийн төлбөрт өгсөн бараа</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group1"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag9')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag9')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag9')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag9')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat9" model="account.tax.template">
        <field name="name">НӨАТ өрийн төлбөрт өгсөн үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag12')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag12')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag12')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag12')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat10" model="account.tax.template">
        <field name="name">НӨАТ цахилгаан, дулаан, ус, шуудан, холбоо</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag13')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag13')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag13')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag13')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat11" model="account.tax.template">
        <field name="name">НӨАТ бараа түрээслүүлэх, эзэмшүүлэх</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag14')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag14')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag14')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag14')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat12" model="account.tax.template">
        <field name="name">НӨАТ байр түрээслүүлэх, эзэмшүүлэх</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag15')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag15')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag15')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag15')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat13" model="account.tax.template">
        <field name="name">НӨАТ хөрөнгө түрээслүүлэх, эзэмшүүлэх</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag16')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag16')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag16')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag16')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat14" model="account.tax.template">
        <field name="name">НӨАТ бүтээл, загвар, зохиогчийн эрх</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag17')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag17')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag17')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag17')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat15" model="account.tax.template">
        <field name="name">НӨАТ хонжворт сугалаа, таавар, бооцоо</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag18')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag18')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag18')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag18')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat16" model="account.tax.template">
        <field name="name">НӨАТ зуучлалын үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag19')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag19')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag19')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag19')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat17" model="account.tax.template">
        <field name="name">НӨАТ авсан хүү торгууль, алданги</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag20')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag20')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag20')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag20')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat18" model="account.tax.template">
        <field name="name">НӨАТ хөрөнгийн үнэлгээний үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag21')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag21')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag21')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag21')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat19" model="account.tax.template">
        <field name="name">НӨАТ төсвийн санхүүжилт, татаас</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag22')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag22')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag22')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag22')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat20" model="account.tax.template">
        <field name="name">НӨАТ хууль зүйн үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag23')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag23')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag23')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag23')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat21" model="account.tax.template">
        <field name="name">НӨАТ үсчин, гоо сайхан, угаалга, цэвэрлэгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag24')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag24')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag24')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag24')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat22" model="account.tax.template">
        <field name="name">НӨАТ бусад үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group2"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag25')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag25')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag25')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag25')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat23" model="account.tax.template">
        <field name="name">НӨАТ 0% экспорт бараа</field>
        <field name="amount">0.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">30</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">0%</field>
        <field name="tax_group_id" ref="account_tax_group3"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag28')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag28')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="account_tax_sale_vat24" model="account.tax.template">
        <field name="name">НӨАТ 0% экспорт үйлчилгээ</field>
        <field name="amount">0.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">30</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">0%</field>
        <field name="tax_group_id" ref="account_tax_group3"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag29')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag29')],
            }),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat3" model="account.tax.template">
        <field name="name">НӨАТ импортоор авсан бараа, үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag35')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag35')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag35')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag35')],
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat4" model="account.tax.template">
        <field name="name">НӨАТ төлөгчөөр бүртгүүлэх үед худалдан авсан</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag37')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag37')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag37')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag37')],
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat5" model="account.tax.template">
        <field name="name">НӨАТ мал аж ахуй эрхлэгчээс худалдан авсан</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag38')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag38')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag38')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag38')],
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat6" model="account.tax.template">
        <field name="name">НӨАТ автомашин, эд анги худалдан авалт (дотоодын)</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'), ref('vat_report_tag36')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag41')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'), ref('vat_report_tag36')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag41')],
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat7" model="account.tax.template">
        <field name="name">НӨАТ ажлын хэрэгцээнд авсан бараа (дотоодын)</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'), ref('vat_report_tag36')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag42')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'), ref('vat_report_tag36')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag42')],
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat8" model="account.tax.template">
        <field name="name">НӨАТ Машин, тоног төхөөрөмж бэлтгэхэд зориулж импортоор худалдан авсан</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'), ref('vat_report_tag37b')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'), ref('vat_report_tag37b')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat9" model="account.tax.template">
        <field name="name">НӨАТ чөлөөлөгдөх үйлдвэрлэлд зориулсан бараа, үйлчилгээ (дотоодын)</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'), ref('vat_report_tag36')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag44')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'), ref('vat_report_tag36')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag44')],
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat10" model="account.tax.template">
        <field name="name">НӨАТ хайгуулын үнэлгээний хөрөнгө бэлтгэхэд зориулсан импортын бараа, үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag45')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag45')],
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat11" model="account.tax.template">
        <field name="name">НӨАТ импортын бусад хамааралгүй бараа, үйлчилгээ (дотоодын)</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'), ref('vat_report_tag36')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag46')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'), ref('vat_report_tag36')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag46')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat25" model="account.tax.template">
        <field name="name">НӨАТ дотоодод зарсан санхүүгийн түрээс</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">50</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group5"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag48')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag48')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag48')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag48')],
            }),
        ]"/>
    </record>
    <record id="account_tax_sale_vat26" model="account.tax.template">
        <field name="name">НӨАТ Факторинг, форфайтинг хэлцлийн үйлчилгээ</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">50</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group5"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag49')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag49')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag49')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_3401_0201'),
                'tag_ids': [ref('vat_report_tag49')],
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat12" model="account.tax.template">
        <field name="name">НӨАТ Санхүүгийн түрээсийн худалдан авалт</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">50</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group5"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'), ref('vat_report_tag51')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag51')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'), ref('vat_report_tag51')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag51')],
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat13" model="account.tax.template">
        <field name="name">НӨАТ Факторинг, форфайтинг худалдан авалт</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">50</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group5"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'), ref('vat_report_tag52')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag52')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'), ref('vat_report_tag52')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
                'tag_ids': [ref('vat_report_tag52')],
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat14" model="account.tax.template">
        <field name="name">НӨАТ Барилга байгууламж бэлтгэхэд зориулж импортоор худалдан авсан</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag37a')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag37a')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat15" model="account.tax.template">
        <field name="name">НӨАТ Бусад үндсэн хөрөнгө бэлтгэхэд зориулж импортоор худалдан авсан</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag37c')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag37c')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
            }),
        ]"/>
    </record>
    <record id="account_tax_purchase_vat16" model="account.tax.template">
        <field name="name">НӨАТ Ашиглалтын ажиллагаанд зориулж импортоор худалдан авсан</field>
        <field name="amount">10.0</field>
        <field name="price_include" eval="1"/>
        <field name="sequence">40</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="description">10%</field>
        <field name="tax_group_id" ref="account_tax_group4"/>
        <field name="chart_template_id" ref="mn_chart_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag37d')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [ref('vat_report_tag33'),ref('vat_report_tag37d')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('account_template_1204_0301'),
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\l10n_mn_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="mn_chart_1" model="account.chart.template">
            <field name="account_journal_early_pay_discount_loss_account_id" ref="account_template_5701_0201"/>
            <field name="account_journal_early_pay_discount_gain_account_id" ref="account_template_5701_0101"/>
            <field name="property_tax_payable_account_id" ref="account_template_3401_9902"/>
            <field name="property_tax_receivable_account_id" ref="account_template_1204_9902"/>
        </record>
    </data>
</odoo>

```

## File: migrations\1.1\post-migrate.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details
from odoo import api, SUPERUSER_ID, Command
from odoo.addons.account.models.chart_template import update_taxes_from_templates


def migrate(cr, version):
    env = api.Environment(cr, SUPERUSER_ID, {})

    # Set the new cashflow statement tags based on the templates
    account_templates = env['account.account.template'].search([('chart_template_id', '=', env.ref('l10n_mn.mn_chart_1').id)])
    for company in env['res.company'].search([('chart_template_id', '=', env.ref('l10n_mn.mn_chart_1').id)]):
        for account in env['account.account'].search([('company_id', '=', company.id)]):
            matching_template = account_templates.filtered(lambda account_template: account.code.startswith(account_template.code))
            if matching_template:
                cashflow_tag = matching_template.tag_ids
                if cashflow_tag:
                    account.write({'tag_ids': [Command.link(tag_id) for tag_id in cashflow_tag.ids]})

    # Update all taxes with the new tax repartition lines on templates.
    update_taxes_from_templates(cr, 'l10n_mn.mn_chart_1')

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.7" y="8.55" width="51.6" height="29.9" maskUnits="userSpaceOnUse">
      <rect x="6.19" y="8.95" width="48.45" height="28.72" rx="1" style="fill: #fff"/>
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
      <image width="1024" height="512" transform="translate(4.7 8.55) scale(0.05 0.06)" xlink:href="data:image/png;base64,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"/>
    </g>
  </g>
</svg>

```

