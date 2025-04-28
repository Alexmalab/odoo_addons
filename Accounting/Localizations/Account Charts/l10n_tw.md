# Odoo Module: l10n_tw

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import models

```

## File: __manifest__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
{
    'name': 'Taiwan - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['tw'],
    'author': 'Odoo PS',
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the base module to manage the accounting chart for Taiwan in Odoo.
==============================================================================
    """,
    'depends': [
        'account',
        'base_address_extended',
    ],
    'data': [
        'data/res.country.state.csv',
        'data/res_currency_data.xml',
        'data/res_country_data.xml',
        'data/res.city.csv',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\res.city.csv

```csv
id,country_id/id,name,zipcode,state_id/id
city_tw_100,base.tw,中正區,100,l10n_tw.state_tw_tpc
city_tw_103,base.tw,大同區,103,l10n_tw.state_tw_tpc
city_tw_104,base.tw,中山區,104,l10n_tw.state_tw_tpc
city_tw_105,base.tw,松山區,105,l10n_tw.state_tw_tpc
city_tw_106,base.tw,大安區,106,l10n_tw.state_tw_tpc
city_tw_108,base.tw,萬華區,108,l10n_tw.state_tw_tpc
city_tw_110,base.tw,信義區,110,l10n_tw.state_tw_tpc
city_tw_111,base.tw,士林區,111,l10n_tw.state_tw_tpc
city_tw_112,base.tw,北投區,112,l10n_tw.state_tw_tpc
city_tw_114,base.tw,內湖區,114,l10n_tw.state_tw_tpc
city_tw_115,base.tw,南港區,115,l10n_tw.state_tw_tpc
city_tw_116,base.tw,文山區,116,l10n_tw.state_tw_tpc
city_tw_200,base.tw,仁愛區,200,l10n_tw.state_tw_klc
city_tw_201,base.tw,信義區,201,l10n_tw.state_tw_klc
city_tw_202,base.tw,中正區,202,l10n_tw.state_tw_klc
city_tw_203,base.tw,中山區,203,l10n_tw.state_tw_klc
city_tw_204,base.tw,安樂區,204,l10n_tw.state_tw_klc
city_tw_205,base.tw,暖暖區,205,l10n_tw.state_tw_klc
city_tw_206,base.tw,七堵區,206,l10n_tw.state_tw_klc
city_tw_207,base.tw,萬里區,207,l10n_tw.state_tw_klc
city_tw_208,base.tw,金山區,208,l10n_tw.state_tw_klc
city_tw_209,base.tw,南竿鄉,209,l10n_tw.state_tw_lcc
city_tw_210,base.tw,北竿鄉,210,l10n_tw.state_tw_lcc
city_tw_211,base.tw,莒光鄉,211,l10n_tw.state_tw_lcc
city_tw_212,base.tw,東引鄉,212,l10n_tw.state_tw_lcc
city_tw_220,base.tw,板橋區,220,l10n_tw.state_tw_ntpc
city_tw_221,base.tw,汐止區,221,l10n_tw.state_tw_ntpc
city_tw_222,base.tw,深坑區,222,l10n_tw.state_tw_ntpc
city_tw_223,base.tw,石碇區,223,l10n_tw.state_tw_ntpc
city_tw_224,base.tw,瑞芳區,224,l10n_tw.state_tw_ntpc
city_tw_226,base.tw,平溪區,226,l10n_tw.state_tw_ntpc
city_tw_227,base.tw,雙溪區,227,l10n_tw.state_tw_ntpc
city_tw_228,base.tw,貢寮區,228,l10n_tw.state_tw_ntpc
city_tw_231,base.tw,新店區,231,l10n_tw.state_tw_ntpc
city_tw_232,base.tw,坪林區,232,l10n_tw.state_tw_ntpc
city_tw_233,base.tw,烏來區,233,l10n_tw.state_tw_ntpc
city_tw_234,base.tw,永和區,234,l10n_tw.state_tw_ntpc
city_tw_235,base.tw,中和區,235,l10n_tw.state_tw_ntpc
city_tw_236,base.tw,土城區,236,l10n_tw.state_tw_ntpc
city_tw_237,base.tw,三峽區,237,l10n_tw.state_tw_ntpc
city_tw_238,base.tw,樹林區,238,l10n_tw.state_tw_ntpc
city_tw_239,base.tw,鶯歌區,239,l10n_tw.state_tw_ntpc
city_tw_241,base.tw,三重區,241,l10n_tw.state_tw_ntpc
city_tw_242,base.tw,新莊區,242,l10n_tw.state_tw_ntpc
city_tw_243,base.tw,泰山區,243,l10n_tw.state_tw_ntpc
city_tw_244,base.tw,林口區,244,l10n_tw.state_tw_ntpc
city_tw_247,base.tw,蘆洲區,247,l10n_tw.state_tw_ntpc
city_tw_248,base.tw,五股區,248,l10n_tw.state_tw_ntpc
city_tw_249,base.tw,八里區,249,l10n_tw.state_tw_ntpc
city_tw_251,base.tw,淡水區,251,l10n_tw.state_tw_ntpc
city_tw_252,base.tw,三芝區,252,l10n_tw.state_tw_ntpc
city_tw_253,base.tw,石門區,253,l10n_tw.state_tw_ntpc
city_tw_260,base.tw,宜蘭市,260,l10n_tw.state_tw_ilh
city_tw_261,base.tw,頭城鎮,261,l10n_tw.state_tw_ilh
city_tw_262,base.tw,礁溪鄉,262,l10n_tw.state_tw_ilh
city_tw_263,base.tw,壯圍鄉,263,l10n_tw.state_tw_ilh
city_tw_264,base.tw,員山鄉,264,l10n_tw.state_tw_ilh
city_tw_265,base.tw,羅東鎮,265,l10n_tw.state_tw_ilh
city_tw_266,base.tw,三星鄉,266,l10n_tw.state_tw_ilh
city_tw_267,base.tw,大同鄉,267,l10n_tw.state_tw_ilh
city_tw_268,base.tw,五結鄉,268,l10n_tw.state_tw_ilh
city_tw_269,base.tw,冬山鄉,269,l10n_tw.state_tw_ilh
city_tw_270,base.tw,蘇澳鎮,270,l10n_tw.state_tw_ilh
city_tw_272,base.tw,南澳鄉,272,l10n_tw.state_tw_ilh
city_tw_290,base.tw,釣魚臺,290,l10n_tw.state_tw_ilh
city_tw_300,base.tw,北區,300,l10n_tw.state_tw_hct
city_tw_300_1,base.tw,東區,300,l10n_tw.state_tw_hct
city_tw_300_2,base.tw,香山區,300,l10n_tw.state_tw_hct
city_tw_302,base.tw,竹北市,302,l10n_tw.state_tw_hch
city_tw_303,base.tw,湖口鄉,303,l10n_tw.state_tw_hch
city_tw_304,base.tw,新豐鄉,304,l10n_tw.state_tw_hch
city_tw_305,base.tw,新埔鎮,305,l10n_tw.state_tw_hch
city_tw_306,base.tw,關西鎮,306,l10n_tw.state_tw_hch
city_tw_307,base.tw,芎林鄉,307,l10n_tw.state_tw_hch
city_tw_308,base.tw,寶山鄉,308,l10n_tw.state_tw_hch
city_tw_310,base.tw,竹東鎮,310,l10n_tw.state_tw_hch
city_tw_311,base.tw,五峰鄉,311,l10n_tw.state_tw_hch
city_tw_312,base.tw,橫山鄉,312,l10n_tw.state_tw_hch
city_tw_313,base.tw,尖石鄉,313,l10n_tw.state_tw_hch
city_tw_314,base.tw,北埔鄉,314,l10n_tw.state_tw_hch
city_tw_315,base.tw,峨眉鄉,315,l10n_tw.state_tw_hch
city_tw_320,base.tw,中壢區,320,l10n_tw.state_tw_tyc
city_tw_324,base.tw,平鎮區,324,l10n_tw.state_tw_tyc
city_tw_325,base.tw,龍潭區,325,l10n_tw.state_tw_tyc
city_tw_326,base.tw,楊梅區,326,l10n_tw.state_tw_tyc
city_tw_327,base.tw,新屋區,327,l10n_tw.state_tw_tyc
city_tw_328,base.tw,觀音區,328,l10n_tw.state_tw_tyc
city_tw_330,base.tw,桃園區,330,l10n_tw.state_tw_tyc
city_tw_333,base.tw,龜山區,333,l10n_tw.state_tw_tyc
city_tw_334,base.tw,八德區,334,l10n_tw.state_tw_tyc
city_tw_335,base.tw,大溪區,335,l10n_tw.state_tw_tyc
city_tw_336,base.tw,復興區,336,l10n_tw.state_tw_tyc
city_tw_337,base.tw,大園區,337,l10n_tw.state_tw_tyc
city_tw_338,base.tw,蘆竹區,338,l10n_tw.state_tw_tyc
city_tw_350,base.tw,竹南鎮,350,l10n_tw.state_tw_mlh
city_tw_351,base.tw,頭份市,351,l10n_tw.state_tw_mlh
city_tw_352,base.tw,三灣鄉,352,l10n_tw.state_tw_mlh
city_tw_353,base.tw,南庄鄉,353,l10n_tw.state_tw_mlh
city_tw_354,base.tw,獅潭鄉,354,l10n_tw.state_tw_mlh
city_tw_356,base.tw,後龍鎮,356,l10n_tw.state_tw_mlh
city_tw_357,base.tw,通霄鎮,357,l10n_tw.state_tw_mlh
city_tw_358,base.tw,苑裡鎮,358,l10n_tw.state_tw_mlh
city_tw_360,base.tw,苗栗市,360,l10n_tw.state_tw_mlh
city_tw_361,base.tw,造橋鄉,361,l10n_tw.state_tw_mlh
city_tw_362,base.tw,頭屋鄉,362,l10n_tw.state_tw_mlh
city_tw_363,base.tw,公館鄉,363,l10n_tw.state_tw_mlh
city_tw_364,base.tw,大湖鄉,364,l10n_tw.state_tw_mlh
city_tw_365,base.tw,泰安鄉,365,l10n_tw.state_tw_mlh
city_tw_366,base.tw,銅鑼鄉,366,l10n_tw.state_tw_mlh
city_tw_367,base.tw,三義鄉,367,l10n_tw.state_tw_mlh
city_tw_368,base.tw,西湖鄉,368,l10n_tw.state_tw_mlh
city_tw_369,base.tw,卓蘭鎮,369,l10n_tw.state_tw_mlh
city_tw_400,base.tw,中區,400,l10n_tw.state_tw_tcc
city_tw_401,base.tw,東區,401,l10n_tw.state_tw_tcc
city_tw_402,base.tw,南區,402,l10n_tw.state_tw_tcc
city_tw_403,base.tw,西區,403,l10n_tw.state_tw_tcc
city_tw_404,base.tw,北區,404,l10n_tw.state_tw_tcc
city_tw_406,base.tw,北屯區,406,l10n_tw.state_tw_tcc
city_tw_407,base.tw,西屯區,407,l10n_tw.state_tw_tcc
city_tw_408,base.tw,南屯區,408,l10n_tw.state_tw_tcc
city_tw_411,base.tw,太平區,411,l10n_tw.state_tw_tcc
city_tw_412,base.tw,大里區,412,l10n_tw.state_tw_tcc
city_tw_413,base.tw,霧峰區,413,l10n_tw.state_tw_tcc
city_tw_414,base.tw,烏日區,414,l10n_tw.state_tw_tcc
city_tw_420,base.tw,豐原區,420,l10n_tw.state_tw_tcc
city_tw_421,base.tw,后里區,421,l10n_tw.state_tw_tcc
city_tw_422,base.tw,石岡區,422,l10n_tw.state_tw_tcc
city_tw_423,base.tw,東勢區,423,l10n_tw.state_tw_tcc
city_tw_424,base.tw,和平區,424,l10n_tw.state_tw_tcc
city_tw_426,base.tw,新社區,426,l10n_tw.state_tw_tcc
city_tw_427,base.tw,潭子區,427,l10n_tw.state_tw_tcc
city_tw_428,base.tw,大雅區,428,l10n_tw.state_tw_tcc
city_tw_429,base.tw,神岡區,429,l10n_tw.state_tw_tcc
city_tw_432,base.tw,大肚區,432,l10n_tw.state_tw_tcc
city_tw_433,base.tw,沙鹿區,433,l10n_tw.state_tw_tcc
city_tw_434,base.tw,龍井區,434,l10n_tw.state_tw_tcc
city_tw_435,base.tw,梧棲區,435,l10n_tw.state_tw_tcc
city_tw_436,base.tw,清水區,436,l10n_tw.state_tw_tcc
city_tw_437,base.tw,大甲區,437,l10n_tw.state_tw_tcc
city_tw_438,base.tw,外埔區,438,l10n_tw.state_tw_tcc
city_tw_439,base.tw,大安區,439,l10n_tw.state_tw_tcc
city_tw_500,base.tw,彰化市,500,l10n_tw.state_tw_chh
city_tw_502,base.tw,芬園鄉,502,l10n_tw.state_tw_chh
city_tw_503,base.tw,花壇鄉,503,l10n_tw.state_tw_chh
city_tw_504,base.tw,秀水鄉,504,l10n_tw.state_tw_chh
city_tw_505,base.tw,鹿港鎮,505,l10n_tw.state_tw_chh
city_tw_506,base.tw,福興鄉,506,l10n_tw.state_tw_chh
city_tw_507,base.tw,線西鄉,507,l10n_tw.state_tw_chh
city_tw_508,base.tw,和美鎮,508,l10n_tw.state_tw_chh
city_tw_509,base.tw,伸港鄉,509,l10n_tw.state_tw_chh
city_tw_510,base.tw,員林市,510,l10n_tw.state_tw_chh
city_tw_511,base.tw,社頭鄉,511,l10n_tw.state_tw_chh
city_tw_512,base.tw,永靖鄉,512,l10n_tw.state_tw_chh
city_tw_513,base.tw,埔心鄉,513,l10n_tw.state_tw_chh
city_tw_514,base.tw,溪湖鎮,514,l10n_tw.state_tw_chh
city_tw_515,base.tw,大村鄉,515,l10n_tw.state_tw_chh
city_tw_516,base.tw,埔鹽鄉,516,l10n_tw.state_tw_chh
city_tw_520,base.tw,田中鎮,520,l10n_tw.state_tw_chh
city_tw_521,base.tw,北斗鎮,521,l10n_tw.state_tw_chh
city_tw_522,base.tw,田尾鄉,522,l10n_tw.state_tw_chh
city_tw_523,base.tw,埤頭鄉,523,l10n_tw.state_tw_chh
city_tw_524,base.tw,溪州鄉,524,l10n_tw.state_tw_chh
city_tw_525,base.tw,竹塘鄉,525,l10n_tw.state_tw_chh
city_tw_526,base.tw,二林鎮,526,l10n_tw.state_tw_chh
city_tw_527,base.tw,大城鄉,527,l10n_tw.state_tw_chh
city_tw_528,base.tw,芳苑鄉,528,l10n_tw.state_tw_chh
city_tw_530,base.tw,二水鄉,530,l10n_tw.state_tw_chh
city_tw_540,base.tw,南投市,540,l10n_tw.state_tw_ntc
city_tw_541,base.tw,中寮鄉,541,l10n_tw.state_tw_ntc
city_tw_542,base.tw,草屯鎮,542,l10n_tw.state_tw_ntc
city_tw_544,base.tw,國姓鄉,544,l10n_tw.state_tw_ntc
city_tw_545,base.tw,埔里鎮,545,l10n_tw.state_tw_ntc
city_tw_546,base.tw,仁愛鄉,546,l10n_tw.state_tw_ntc
city_tw_551,base.tw,名間鄉,551,l10n_tw.state_tw_ntc
city_tw_552,base.tw,集集鎮,552,l10n_tw.state_tw_ntc
city_tw_553,base.tw,水里鄉,553,l10n_tw.state_tw_ntc
city_tw_555,base.tw,魚池鄉,555,l10n_tw.state_tw_ntc
city_tw_556,base.tw,信義鄉,556,l10n_tw.state_tw_ntc
city_tw_557,base.tw,竹山鎮,557,l10n_tw.state_tw_ntc
city_tw_558,base.tw,鹿谷鄉,558,l10n_tw.state_tw_ntc
city_tw_600,base.tw,東區,600,l10n_tw.state_tw_cic
city_tw_600_1,base.tw,西區,600,l10n_tw.state_tw_cic
city_tw_602,base.tw,番路鄉,602,l10n_tw.state_tw_cih
city_tw_603,base.tw,梅山鄉,603,l10n_tw.state_tw_cih
city_tw_604,base.tw,竹崎鄉,604,l10n_tw.state_tw_cih
city_tw_605,base.tw,阿里山鄉,605,l10n_tw.state_tw_cih
city_tw_606,base.tw,中埔鄉,606,l10n_tw.state_tw_cih
city_tw_607,base.tw,大埔鄉,607,l10n_tw.state_tw_cih
city_tw_608,base.tw,水上鄉,608,l10n_tw.state_tw_cih
city_tw_611,base.tw,鹿草鄉,611,l10n_tw.state_tw_cih
city_tw_612,base.tw,太保市,612,l10n_tw.state_tw_cih
city_tw_613,base.tw,朴子市,613,l10n_tw.state_tw_cih
city_tw_614,base.tw,東石鄉,614,l10n_tw.state_tw_cih
city_tw_615,base.tw,六腳鄉,615,l10n_tw.state_tw_cih
city_tw_616,base.tw,新港鄉,616,l10n_tw.state_tw_cih
city_tw_621,base.tw,民雄鄉,621,l10n_tw.state_tw_cih
city_tw_622,base.tw,大林鎮,622,l10n_tw.state_tw_cih
city_tw_623,base.tw,溪口鄉,623,l10n_tw.state_tw_cih
city_tw_624,base.tw,義竹鄉,624,l10n_tw.state_tw_cih
city_tw_625,base.tw,布袋鎮,625,l10n_tw.state_tw_cih
city_tw_630,base.tw,斗南鎮,630,l10n_tw.state_tw_ylh
city_tw_631,base.tw,大埤鄉,631,l10n_tw.state_tw_ylh
city_tw_632,base.tw,虎尾鎮,632,l10n_tw.state_tw_ylh
city_tw_633,base.tw,土庫鎮,633,l10n_tw.state_tw_ylh
city_tw_634,base.tw,褒忠鄉,634,l10n_tw.state_tw_ylh
city_tw_635,base.tw,東勢鄉,635,l10n_tw.state_tw_ylh
city_tw_636,base.tw,台西鄉,636,l10n_tw.state_tw_ylh
city_tw_637,base.tw,崙背鄉,637,l10n_tw.state_tw_ylh
city_tw_638,base.tw,麥寮鄉,638,l10n_tw.state_tw_ylh
city_tw_640,base.tw,斗六市,640,l10n_tw.state_tw_ylh
city_tw_643,base.tw,林內鄉,643,l10n_tw.state_tw_ylh
city_tw_646,base.tw,古坑鄉,646,l10n_tw.state_tw_ylh
city_tw_647,base.tw,莿桐鄉,647,l10n_tw.state_tw_ylh
city_tw_648,base.tw,西螺鎮,648,l10n_tw.state_tw_ylh
city_tw_649,base.tw,二崙鄉,649,l10n_tw.state_tw_ylh
city_tw_651,base.tw,北港鎮,651,l10n_tw.state_tw_ylh
city_tw_652,base.tw,水林鄉,652,l10n_tw.state_tw_ylh
city_tw_653,base.tw,口湖鄉,653,l10n_tw.state_tw_ylh
city_tw_654,base.tw,四湖鄉,654,l10n_tw.state_tw_ylh
city_tw_655,base.tw,元長鄉,655,l10n_tw.state_tw_ylh
city_tw_700,base.tw,中西區,700,l10n_tw.state_tw_tnh
city_tw_701,base.tw,東區,701,l10n_tw.state_tw_tnh
city_tw_702,base.tw,南區,702,l10n_tw.state_tw_tnh
city_tw_704,base.tw,北區,704,l10n_tw.state_tw_tnh
city_tw_708,base.tw,安平區,708,l10n_tw.state_tw_tnh
city_tw_709,base.tw,安南區,709,l10n_tw.state_tw_tnh
city_tw_710,base.tw,永康區,710,l10n_tw.state_tw_tnh
city_tw_711,base.tw,歸仁區,711,l10n_tw.state_tw_tnh
city_tw_712,base.tw,新化區,712,l10n_tw.state_tw_tnh
city_tw_713,base.tw,左鎮區,713,l10n_tw.state_tw_tnh
city_tw_714,base.tw,玉井區,714,l10n_tw.state_tw_tnh
city_tw_715,base.tw,楠西區,715,l10n_tw.state_tw_tnh
city_tw_716,base.tw,南化區,716,l10n_tw.state_tw_tnh
city_tw_717,base.tw,仁德區,717,l10n_tw.state_tw_tnh
city_tw_718,base.tw,關廟區,718,l10n_tw.state_tw_tnh
city_tw_719,base.tw,龍崎區,719,l10n_tw.state_tw_tnh
city_tw_720,base.tw,官田區,720,l10n_tw.state_tw_tnh
city_tw_721,base.tw,麻豆區,721,l10n_tw.state_tw_tnh
city_tw_722,base.tw,佳里區,722,l10n_tw.state_tw_tnh
city_tw_723,base.tw,西港區,723,l10n_tw.state_tw_tnh
city_tw_724,base.tw,七股區,724,l10n_tw.state_tw_tnh
city_tw_725,base.tw,將軍區,725,l10n_tw.state_tw_tnh
city_tw_726,base.tw,學甲區,726,l10n_tw.state_tw_tnh
city_tw_727,base.tw,北門區,727,l10n_tw.state_tw_tnh
city_tw_730,base.tw,新營區,730,l10n_tw.state_tw_tnh
city_tw_731,base.tw,後壁區,731,l10n_tw.state_tw_tnh
city_tw_732,base.tw,白河區,732,l10n_tw.state_tw_tnh
city_tw_733,base.tw,東山區,733,l10n_tw.state_tw_tnh
city_tw_734,base.tw,六甲區,734,l10n_tw.state_tw_tnh
city_tw_735,base.tw,下營區,735,l10n_tw.state_tw_tnh
city_tw_736,base.tw,柳營區,736,l10n_tw.state_tw_tnh
city_tw_737,base.tw,鹽水區,737,l10n_tw.state_tw_tnh
city_tw_741,base.tw,善化區,741,l10n_tw.state_tw_tnh
city_tw_742,base.tw,大內區,742,l10n_tw.state_tw_tnh
city_tw_743,base.tw,山上區,743,l10n_tw.state_tw_tnh
city_tw_744,base.tw,新市區,744,l10n_tw.state_tw_tnh
city_tw_745,base.tw,安定區,745,l10n_tw.state_tw_tnh
city_tw_800,base.tw,新興區,800,l10n_tw.state_tw_khc
city_tw_801,base.tw,前金區,801,l10n_tw.state_tw_khc
city_tw_802,base.tw,苓雅區,802,l10n_tw.state_tw_khc
city_tw_803,base.tw,鹽埕區,803,l10n_tw.state_tw_khc
city_tw_804,base.tw,鼓山區,804,l10n_tw.state_tw_khc
city_tw_805,base.tw,旗津區,805,l10n_tw.state_tw_khc
city_tw_806,base.tw,前鎮區,806,l10n_tw.state_tw_khc
city_tw_807,base.tw,三民區,807,l10n_tw.state_tw_khc
city_tw_811,base.tw,楠梓區,811,l10n_tw.state_tw_khc
city_tw_812,base.tw,小港區,812,l10n_tw.state_tw_khc
city_tw_813,base.tw,左營區,813,l10n_tw.state_tw_khc
city_tw_814,base.tw,仁武區,814,l10n_tw.state_tw_khc
city_tw_815,base.tw,大社區,815,l10n_tw.state_tw_khc
city_tw_817,base.tw,東沙群島,817,l10n_tw.state_tw_khc
city_tw_819,base.tw,南沙群島,819,l10n_tw.state_tw_khc
city_tw_820,base.tw,岡山區,820,l10n_tw.state_tw_khc
city_tw_821,base.tw,路竹區,821,l10n_tw.state_tw_khc
city_tw_822,base.tw,阿蓮區,822,l10n_tw.state_tw_khc
city_tw_823,base.tw,田寮區,823,l10n_tw.state_tw_khc
city_tw_824,base.tw,燕巢區,824,l10n_tw.state_tw_khc
city_tw_825,base.tw,橋頭區,825,l10n_tw.state_tw_khc
city_tw_826,base.tw,梓官區,826,l10n_tw.state_tw_khc
city_tw_827,base.tw,彌陀區,827,l10n_tw.state_tw_khc
city_tw_828,base.tw,永安區,828,l10n_tw.state_tw_khc
city_tw_829,base.tw,湖內區,829,l10n_tw.state_tw_khc
city_tw_830,base.tw,鳳山區,830,l10n_tw.state_tw_khc
city_tw_831,base.tw,大寮區,831,l10n_tw.state_tw_khc
city_tw_832,base.tw,林園區,832,l10n_tw.state_tw_khc
city_tw_833,base.tw,鳥松區,833,l10n_tw.state_tw_khc
city_tw_840,base.tw,大樹區,840,l10n_tw.state_tw_khc
city_tw_842,base.tw,旗山區,842,l10n_tw.state_tw_khc
city_tw_843,base.tw,美濃區,843,l10n_tw.state_tw_khc
city_tw_844,base.tw,六龜區,844,l10n_tw.state_tw_khc
city_tw_845,base.tw,內門區,845,l10n_tw.state_tw_khc
city_tw_846,base.tw,杉林區,846,l10n_tw.state_tw_khc
city_tw_847,base.tw,甲仙區,847,l10n_tw.state_tw_khc
city_tw_848,base.tw,桃源區,848,l10n_tw.state_tw_khc
city_tw_849,base.tw,那瑪夏區,849,l10n_tw.state_tw_khc
city_tw_851,base.tw,茂林區,851,l10n_tw.state_tw_khc
city_tw_852,base.tw,茄萣區,852,l10n_tw.state_tw_khc
city_tw_880,base.tw,馬公市,880,l10n_tw.state_tw_phc
city_tw_881,base.tw,西嶼鄉,881,l10n_tw.state_tw_phc
city_tw_882,base.tw,望安鄉,882,l10n_tw.state_tw_phc
city_tw_883,base.tw,七美鄉,883,l10n_tw.state_tw_phc
city_tw_884,base.tw,白沙鄉,884,l10n_tw.state_tw_phc
city_tw_885,base.tw,湖西鄉,885,l10n_tw.state_tw_phc
city_tw_890,base.tw,金沙鎮,890,l10n_tw.state_tw_kmc
city_tw_891,base.tw,金湖鎮,891,l10n_tw.state_tw_kmc
city_tw_892,base.tw,金寧鄉,892,l10n_tw.state_tw_kmc
city_tw_893,base.tw,金城鎮,893,l10n_tw.state_tw_kmc
city_tw_894,base.tw,烈嶼鄉,894,l10n_tw.state_tw_kmc
city_tw_896,base.tw,烏坵鄉,896,l10n_tw.state_tw_kmc
city_tw_900,base.tw,屏東市,900,l10n_tw.state_tw_pth
city_tw_901,base.tw,三地門鄉,901,l10n_tw.state_tw_pth
city_tw_902,base.tw,霧台鄉,902,l10n_tw.state_tw_pth
city_tw_903,base.tw,瑪家鄉,903,l10n_tw.state_tw_pth
city_tw_904,base.tw,九如鄉,904,l10n_tw.state_tw_pth
city_tw_905,base.tw,里港鄉,905,l10n_tw.state_tw_pth
city_tw_906,base.tw,高樹鄉,906,l10n_tw.state_tw_pth
city_tw_907,base.tw,鹽埔鄉,907,l10n_tw.state_tw_pth
city_tw_908,base.tw,長治鄉,908,l10n_tw.state_tw_pth
city_tw_909,base.tw,麟洛鄉,909,l10n_tw.state_tw_pth
city_tw_911,base.tw,竹田鄉,911,l10n_tw.state_tw_pth
city_tw_912,base.tw,內埔鄉,912,l10n_tw.state_tw_pth
city_tw_913,base.tw,萬丹鄉,913,l10n_tw.state_tw_pth
city_tw_920,base.tw,潮州鎮,920,l10n_tw.state_tw_pth
city_tw_921,base.tw,泰武鄉,921,l10n_tw.state_tw_pth
city_tw_922,base.tw,來義鄉,922,l10n_tw.state_tw_pth
city_tw_923,base.tw,萬巒鄉,923,l10n_tw.state_tw_pth
city_tw_924,base.tw,崁頂鄉,924,l10n_tw.state_tw_pth
city_tw_925,base.tw,新埤鄉,925,l10n_tw.state_tw_pth
city_tw_926,base.tw,南州鄉,926,l10n_tw.state_tw_pth
city_tw_927,base.tw,林邊鄉,927,l10n_tw.state_tw_pth
city_tw_928,base.tw,東港鎮,928,l10n_tw.state_tw_pth
city_tw_929,base.tw,琉球鄉,929,l10n_tw.state_tw_pth
city_tw_931,base.tw,佳冬鄉,931,l10n_tw.state_tw_pth
city_tw_932,base.tw,新園鄉,932,l10n_tw.state_tw_pth
city_tw_940,base.tw,枋寮鄉,940,l10n_tw.state_tw_pth
city_tw_941,base.tw,枋山鄉,941,l10n_tw.state_tw_pth
city_tw_942,base.tw,春日鄉,942,l10n_tw.state_tw_pth
city_tw_943,base.tw,獅子鄉,943,l10n_tw.state_tw_pth
city_tw_944,base.tw,車城鄉,944,l10n_tw.state_tw_pth
city_tw_945,base.tw,牡丹鄉,945,l10n_tw.state_tw_pth
city_tw_946,base.tw,恆春鎮,946,l10n_tw.state_tw_pth
city_tw_947,base.tw,滿州鄉,947,l10n_tw.state_tw_pth
city_tw_950,base.tw,台東市,950,l10n_tw.state_tw_tth
city_tw_951,base.tw,綠島鄉,951,l10n_tw.state_tw_tth
city_tw_952,base.tw,蘭嶼鄉,952,l10n_tw.state_tw_tth
city_tw_953,base.tw,延平鄉,953,l10n_tw.state_tw_tth
city_tw_954,base.tw,卑南鄉,954,l10n_tw.state_tw_tth
city_tw_955,base.tw,鹿野鄉,955,l10n_tw.state_tw_tth
city_tw_956,base.tw,關山鎮,956,l10n_tw.state_tw_tth
city_tw_957,base.tw,海端鄉,957,l10n_tw.state_tw_tth
city_tw_958,base.tw,池上鄉,958,l10n_tw.state_tw_tth
city_tw_959,base.tw,東河鄉,959,l10n_tw.state_tw_tth
city_tw_961,base.tw,成功鎮,961,l10n_tw.state_tw_tth
city_tw_962,base.tw,長濱鄉,962,l10n_tw.state_tw_tth
city_tw_963,base.tw,太麻里鄉,963,l10n_tw.state_tw_tth
city_tw_964,base.tw,金峰鄉,964,l10n_tw.state_tw_tth
city_tw_965,base.tw,大武鄉,965,l10n_tw.state_tw_tth
city_tw_966,base.tw,達仁鄉,966,l10n_tw.state_tw_tth
city_tw_970,base.tw,花蓮市,970,l10n_tw.state_tw_hlh
city_tw_971,base.tw,新城鄉,971,l10n_tw.state_tw_hlh
city_tw_972,base.tw,秀林鄉,972,l10n_tw.state_tw_hlh
city_tw_973,base.tw,吉安鄉,973,l10n_tw.state_tw_hlh
city_tw_974,base.tw,壽豐鄉,974,l10n_tw.state_tw_hlh
city_tw_975,base.tw,鳳林鎮,975,l10n_tw.state_tw_hlh
city_tw_976,base.tw,光復鄉,976,l10n_tw.state_tw_hlh
city_tw_977,base.tw,豐濱鄉,977,l10n_tw.state_tw_hlh
city_tw_978,base.tw,瑞穗鄉,978,l10n_tw.state_tw_hlh
city_tw_979,base.tw,萬榮鄉,979,l10n_tw.state_tw_hlh
city_tw_981,base.tw,玉里鎮,981,l10n_tw.state_tw_hlh
city_tw_982,base.tw,卓溪鄉,982,l10n_tw.state_tw_hlh
city_tw_983,base.tw,富里鄉,983,l10n_tw.state_tw_hlh

```

## File: data\res.country.state.csv

```csv
id,country_id/id,name,code
state_tw_chh,base.tw,彰化縣,CHH
state_tw_cic,base.tw,嘉義市,CIC
state_tw_cih,base.tw,嘉義縣,CIH
state_tw_hch,base.tw,新竹縣,HCH
state_tw_hct,base.tw,新竹市,HCT
state_tw_hlh,base.tw,花蓮縣,HLH
state_tw_ilh,base.tw,宜蘭縣,ILH
state_tw_khc,base.tw,高雄市,KHC
state_tw_klc,base.tw,基隆市,KLC
state_tw_kmc,base.tw,金門縣,KMC
state_tw_lcc,base.tw,連江縣,LCC
state_tw_mlh,base.tw,苗栗縣,MLH
state_tw_ntc,base.tw,南投縣,NTC
state_tw_ntpc,base.tw,新北市,NTPC
state_tw_phc,base.tw,澎湖縣,PHC
state_tw_pth,base.tw,屏東縣,PTH
state_tw_tcc,base.tw,台中市,TCC
state_tw_tnh,base.tw,台南市,TNH
state_tw_tpc,base.tw,台北市,TPC
state_tw_tth,base.tw,台東縣,TTH
state_tw_tyc,base.tw,桃園市,TYC
state_tw_ylh,base.tw,雲林縣,YLH

```

## File: data\res_country_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="base.tw" model="res.country">
        <field name="enforce_cities" eval="1" />
    </record>
</odoo>

```

## File: data\res_currency_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="base.TWD" model="res.currency">
            <field name="active" eval="True" />
            <field name="position">before</field>
        </record>
    </data>
</odoo>

```

## File: data\template\account.account-tw.csv

```csv
"id","code","name","account_type","reconcile","name@zh_TW"
"tw_118100","118100","Notes receivable","asset_receivable","True","應收票據"
"tw_119100","119100","Accounts receivable","asset_receivable","True","應收帳款"
"tw_119150","119150","Accounts receivable (PoS)","asset_receivable","True","應收帳款(POS)"
"tw_121100","121100","Earned revenue receivable","asset_receivable","True","應收收益"
"tw_123100","123100","Merchandise inventory","asset_current","False","商品存貨"
"tw_123150","123150","Other inventory (pending acceptance)","asset_current","True","其他存貨(已出貨待驗收)"
"tw_123500","123500","Finished goods","asset_current","False","製成品"
"tw_123700","123700","Work in progress","asset_current","False","在製品"
"tw_123900","123900","Raw materials","asset_current","False","原料"
"tw_124000","124000","Supplies","asset_current","False","物料"
"tw_124500","124500","Stock Interim Account (Input)","asset_current","False",""
"tw_124600","124600","Stock Interim Account (Output)","asset_current","False",""
"tw_126400","126400","Office supplies","asset_current","False","用品盤存"
"tw_126500","126500","Other prepaid expenses","asset_current","False","其他預付費用"
"tw_126600","126600","Prepayment for purchases","asset_current","False","預付貨款"
"tw_126800","126800","Business tax paid (or Input VAT)","asset_current","False","進項稅額"
"tw_126900","126900","Overpaid sales tax","asset_current","False","留抵稅額"
"tw_127000","127000","Other prepayments","asset_current","False","其他預付款項"
"tw_128100","128100","Temporary payments","asset_current","False","暫付款"
"tw_128200","128200","Payment on behalf of others","asset_current","False","代付款"
"tw_128400","128400","Refundable deposits","asset_current","False","存出保證金"
"tw_137100","137100","Investments accounted for using equity method","asset_non_current","False","採用權益法之投資"
"tw_139100","139100","Land, cost","asset_non_current","False","土地－成本"
"tw_141100","141100","Buildings and structures, cost","asset_non_current","False","房屋及建築－成本"
"tw_141300","141300","Accumulated depreciation, buildings and structures","asset_non_current","False","累計折舊－房屋及建築"
"tw_142100","142100","Machinery and equipment, cost","asset_non_current","False","機器設備－成本"
"tw_142200","142200","Accumulated depreciation, machinery and equipment","asset_non_current","False","累計折舊－機器設備"
"tw_143100","143100","Office equipment, cost","asset_non_current","False","辦公設備－成本"
"tw_143200","143200","Accumulated depreciation, office equipment","asset_non_current","False","累計折舊－辦公設備"
"tw_144100","144100","Leased assets, cost","asset_non_current","False","租賃資產－成本"
"tw_144200","144200","Accumulated depreciation, leased assets","asset_non_current","False","累計折舊－租賃資產"
"tw_155100","155100","Other intangible assets, net","asset_non_current","False","其他無形資產"
"tw_155200","155200","Accumulated amortization, other intangible assets","asset_non_current","False","累計攤銷－其他無形資產"
"tw_156100","156100","Deferred tax assets","asset_non_current","False","遞延所得稅資產"
"tw_158200","158200","Prepayments for business facilities","asset_non_current","False","預付設備款"
"tw_158300","158300","Guarantee deposits paid","asset_non_current","False","存出保證金"
"tw_158400","158400","Owner (shareholder) accounts, debit","asset_non_current","False","業主(股東)往來"
"tw_158600","158600","Other non-current assets, others","asset_non_current","False","其他非流動資產－其他"
"tw_211200","211200","Bank loan","liability_current","False","銀行借款"
"tw_216100","216100","Notes payable","liability_payable","True","應付票據"
"tw_217100","217100","Accounts payable","liability_payable","True","應付帳款"
"tw_219400","219400","Business tax payable","liability_payable","True","應付營業稅"
"tw_219700","219700","Other accrued expenses","liability_payable","True","其他應付費用"
"tw_219900","219900","Payable on machinery and equipment","liability_payable","True","應付設備款"
"tw_220100","220100","Dividends payable","liability_payable","True","應付股利"
"tw_220400","220400","Business tax received (or Output VAT)","liability_current","False","銷項稅額"
"tw_222100","222100","Advance sales receipts","liability_current","False","預收貨款"
"tw_222300","222300","Other advance receipts","liability_current","False","其他預收款"
"tw_225100","225100","Temporary credits","liability_current","False","暫收款"
"tw_225200","225200","Receipts under custody","liability_current","False","代收款"
"tw_239300","239300","Owner (shareholder) accounts, credit","liability_current","False","業主(股東)往來"
"tw_311100","311100","Ordinary share","equity","False","普通股股本"
"tw_321100","321100","Capital surplus, additional paid-in capital arising from ordinary share","equity","False","資本公積－普通股股票溢價"
"tw_335100","335100","Accumulated profit and loss","equity","False","累積盈虧"
"tw_341500","341500","Other equity interest, others","equity","False","其他權益－其他"
"tw_411100","411100","Sales revenue","income","False","銷貨收入"
"tw_411300","411300","Sales returns","income","False","銷貨退回"
"tw_411400","411400","Sales discounts and allowances","income","False","銷貨折讓"
"tw_414100","414100","Other operating revenue","income","False","其他營業收入"
"tw_412100","412100","Labor Income","income","False","勞務收入"
"tw_413100","413100","Engineering Income","income_other","False","工程收入"
"tw_423200","423200","Engineering Income refund or discount","income_other","False","工程收入退回及折讓"
"tw_511100","511100","Cost of sales","expense_direct_cost","False","銷貨成本"
"tw_512100","512100","Purchase of goods","expense_direct_cost","False","進貨"
"tw_512300","512300","Purchases returns","expense_direct_cost","False","進貨退出"
"tw_512400","512400","Purchases discounts and allowances","expense_direct_cost","False","進貨折讓"
"tw_513100","513100","Purchase of raw materials","expense_direct_cost","False","進料"
"tw_513300","513300","Raw materials purchase returns","expense_direct_cost","False","進料退出"
"tw_513400","513400","Raw materials purchase discounts and allowances","expense_direct_cost","False","進料折讓"
"tw_514100","514100","Direct labor","expense_direct_cost","False","直接人工"
"tw_515200","515200","Rent expense","expense_direct_cost","False","租金支出"
"tw_515700","515700","Repairs and maintenance expense","expense_direct_cost","False","修繕費"
"tw_515800","515800","Packing expenses","expense_direct_cost","False","包裝費"
"tw_515900","515900","Utilities expense","expense_direct_cost","False","水電瓦斯費"
"tw_516000","516000","Insurance expense","expense_direct_cost","False","保險費"
"tw_516100","516100","Processing expense","expense_direct_cost","False","加工費"
"tw_516300","516300","Depreciations","expense_direct_cost","False","折舊"
"tw_516500","516500","Meal expense","expense_direct_cost","False","伙食費"
"tw_516800","516800","Indirect materials","expense_direct_cost","False","間接材料"
"tw_516900","516900","Other overheads","expense_direct_cost","False","其他製造費用"
"tw_591100","591100","Other operating costs","expense_direct_cost","False","其他營業成本"
"tw_611100","611100","Wages and salaries","expense","False","薪資支出"
"tw_611200","611200","Rent expense","expense","False","租金支出"
"tw_611300","611300","Stationery supplies","expense","False","文具用品"
"tw_611400","611400","Traveling Expense","expense","False","旅費"
"tw_611500","611500","Freight","expense","False","運費"
"tw_611600","611600","Postage expenses","expense","False","郵電費"
"tw_611700","611700","Repairs and maintenance expense","expense","False","修繕費"
"tw_611800","611800","Advertisement expense","expense","False","廣告費"
"tw_611900","611900","Utilities expense","expense","False","水電瓦斯費"
"tw_612000","612000","Insurance expense","expense","False","保險費"
"tw_612100","612100","Entertainment expense","expense","False","交際費"
"tw_612200","612200","Donation expense","expense","False","捐贈"
"tw_612300","612300","Taxes","expense","False","稅捐"
"tw_612400","612400","Losses on doubtful debts","expense","False","呆帳損失"
"tw_612500","612500","Depreciations","expense","False","折舊"
"tw_612600","612600","Depletions and amortizations","expense","False","各項耗竭及攤提"
"tw_612700","612700","Losses on export sales","expense","False","外銷損失"
"tw_612800","612800","Meal expense","expense","False","伙食費"
"tw_612900","612900","Employee benefits/welfare","expense","False","職工福利"
"tw_613000","613000","Research and development expense","expense","False","研究發展費用"
"tw_613100","613100","Commissions expense","expense","False","佣金支出"
"tw_613200","613200","Training expense","expense","False","訓練費"
"tw_613300","613300","Services expense","expense","False","勞務費"
"tw_613400","613400","Other operating expenses","expense","False","其他營業費用"
"tw_711100","711100","Interest revenue","income_other","False","利息收入"
"tw_712100","712100","Rent income","income_other","False","租金收入"
"tw_714100","714100","Dividend revenue","income_other","False","股利收入"
"tw_715100","715100","Interest expense","expense","False","利息費用"
"tw_717100","717100","Investment income accounted for using equity method","income_other","False","採權益法認列之投資利益"
"tw_717200","717200","Investment loss accounted for using equity method","expense","False","採權益法認列之投資損失"
"tw_718100","718100","Foreign exchange gains","income_other","False","兌換利益"
"tw_718200","718200","Foreign exchange losses","expense","False","兌換損失"
"tw_718500","718500","Cash difference gains","income_other","False","現金差異利益"
"tw_718600","718600","Cash difference losses","expense","False","現金差異損失"
"tw_719100","719100","Gains on disposals of investment property","income_other","False","處分投資性不動產利益"
"tw_719200","719200","Losses on disposals of investment property","expense","False","處分投資性不動產損失"
"tw_720100","720100","Net gain or loss on disposals of property, plant and equipment","income_other","False","處分不動產、廠房及設備利益"
"tw_720200","720200","Gains on disposals of property, plant and equipment","expense","False","處分不動產、廠房及設備損失"
"tw_723200","723200","Commissions revenue","income_other","False","佣金收入"
"tw_724700","724700","Other revenue","income_other","False","其他收入"
"tw_821100","821100","Tax expense (income)","expense","False","所得稅費用(或利益)"
"tw_841100","841100","Closed Units' asset or profit (post tax)","income","False","停業單位資產或處分群組處分損益（稅後)"
"tw_871100","871100","Unrealized gains and losses on available-for-sale financial assets","income_other","False","備供出售金融資產未實現損益"

```

## File: data\template\account.tax-tw.csv

```csv
"id","description","sequence","name","invoice_label","type_tax_use","amount_type","amount","price_include","tax_group_id","include_base_amount","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/account_id","description@zh_TW"
"tw_tax_sale_5","Sale (5%)","1","5%","GST Sales","sale","percent","5.0","False","tax_group_gst_5","","base","invoice","","銷項5%"
"","","","","","","","","","","","tax","invoice","tw_220400",""
"","","","","","","","","","","","base","refund","",""
"","","","","","","","","","","","tax","refund","tw_220400",""
"tw_tax_sale_inc_5","GST Inc Sale (5%)","2","5% INC","GST Inclusive Sale","sale","percent","5.0","True","tax_group_gst_5","1","base","invoice","","銷項5%-內含"
"","","","","","","","","","","","tax","invoice","tw_220400",""
"","","","","","","","","","","","base","refund","",""
"","","","","","","","","","","","tax","refund","tw_220400",""
"tw_tax_purchase_5","Purchase (5%)","1","5%","GST Purchase","purchase","percent","5.0","False","tax_group_gst_5","","base","invoice","","進項5%"
"","","","","","","","","","","","tax","invoice","tw_126800",""
"","","","","","","","","","","","base","refund","",""
"","","","","","","","","","","","tax","refund","tw_126800",""
"tw_tax_purchase_inc_5","GST Inc Purchase (5%)","","5% INC","GST Inclusive Purchases","purchase","percent","5.0","True","tax_group_gst_5","1","base","invoice","","進項5%-內含"
"","","","","","","","","","","","tax","invoice","tw_126800",""
"","","","","","","","","","","","base","refund","",""
"","","","","","","","","","","","tax","refund","tw_126800",""

```

## File: data\template\account.tax.group-tw.csv

```csv
"id","name","country_id","name@zh_TW"
"tax_group_gst_5","GST 5%","base.tw","營業稅"

```

## File: models\template_tw.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('tw')
    def _get_tw_template_data(self):
        return {
            'code_digits': '6',
            'property_account_receivable_id': 'tw_119100',
            'property_account_payable_id': 'tw_217100',
            'property_account_expense_categ_id': 'tw_511100',
            'property_account_income_categ_id': 'tw_411100',
            'property_stock_account_input_categ_id': 'tw_124500',
            'property_stock_account_output_categ_id': 'tw_124600',
            'property_stock_valuation_account_id': 'tw_123100',
        }

    @template('tw', 'res.company')
    def _get_tw_res_company(self):
        return {
            self.env.company.id: {
                'anglo_saxon_accounting': True,
                'account_fiscal_country_id': 'base.tw',
                'bank_account_code_prefix': '1113',
                'cash_account_code_prefix': '1111',
                'transfer_account_code_prefix': '1114',
                'account_default_pos_receivable_account_id': 'tw_119150',
                'income_currency_exchange_account_id': 'tw_718100',
                'expense_currency_exchange_account_id': 'tw_718200',
                'account_journal_early_pay_discount_loss_account_id': 'tw_411400',
                'account_journal_early_pay_discount_gain_account_id': 'tw_512400',
                'default_cash_difference_income_account_id': 'tw_718500',
                'default_cash_difference_expense_account_id': 'tw_718600',
                'account_sale_tax_id': 'tw_tax_sale_5',
                'account_purchase_tax_id': 'tw_tax_purchase_5',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_tw

```

