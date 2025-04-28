# Odoo Module: l10n_tr

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
    'name': 'Turkey - Accounting',
    'version': '1.1',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Türkiye için Tek düzen hesap planı şablonu Odoo Modülü.
==========================================================

Bu modül kurulduktan sonra, Muhasebe yapılandırma sihirbazı çalışır
    * Sihirbaz sizden hesap planı şablonu, planın kurulacağı şirket, banka hesap
      bilgileriniz, ilgili para birimi gibi bilgiler isteyecek.
    """,
    'author': 'Ahmet Altınışık, Can Tecim',
    'maintainer':'https://launchpad.net/~openerp-turkey, http://www.cantecim.com',
    'depends': [
        'account',
    ],
    'data': [
        'data/l10n_tr_chart_data.xml',
        'data/account.account.template.csv',
        'data/l10n_tr_chart_post_data.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_template_data.xml',
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
"id","name","code","user_type_id/id","chart_template_id/id","reconcile"
"tr101","Alınan Çekler","101","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr108","Diğer Hazır Değerler","108","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr110","Hisse Senetleri","110","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr111","Özel Kesim Tahvil Senet Ve Bonoları","111","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr112","Kamu Kesimi Tahvil, Senet ve Bonoları","112","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr118","Diğer Menkul Kıymetler","118","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr119","Menkul Kıymetler Değer Düşüklüğü Karşılığı(-)","119","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr120","Alıcılar","120","account.data_account_type_receivable","l10n_tr.l10ntr_tek_duzen_hesap","True"
"tr121","Alacak Senetleri","121","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr122","Alacak Senetleri Reeskontu(-)","122","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr123","Alıcılar (PoS)","123","account.data_account_type_receivable","l10n_tr.l10ntr_tek_duzen_hesap","True"
"tr124","Kazanılmamış Finansal Kiralama Faiz Gelirleri(-)","124","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr126","Verilen Depozito ve Teminatlar","126","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr127","Diğer Ticari Alacaklar","127","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr128","Şüpheli Ticari Alacaklar","128","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr129","Şüpheli Ticari Alacaklar Karşılığı","129","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr131","Ortaklardan Alacaklar","131","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr132","İştiraklerden Alacaklar","132","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr133","Bağlı Ortaklıklardan Alacaklar","133","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr135","Personelden Alacaklar","135","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr136","Diğer Çeşitli Alacaklar","136","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr137","Diğer Alacak Senetleri Reeskontu(-)","137","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr138","Şüpheli Diğer Alacaklar","138","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr139","Şüpheli Diğer Alacaklar Karşılığı(-)","139","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr150","İlk Madde Malzeme","150","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr151","Yarı Mamuller","151","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr152","Mamuller","152","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr153","Ticari Mallar","153","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr158","Stok Değer Düşüklüğü Karşılığı(-)","158","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr159","Verilen Sipariş Avansları","159","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr170","Yıllara Yaygın İnşaat Ve Onarım Maliyetleri","170","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr179","Taşeronlara Verilen Avanslar","179","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr180","Gelecek Aylara Ait Giderler","180","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr181","Gelir Tahakkukları","181","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr190","Devreden KDV","190","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr191","İndirilecek KDV","191","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr192","Diğer KDV","192","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr193","Peşin Ödenen Vergiler Ve Fonlar","193","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr195","İş Avansları","195","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr196","Personel Avansları","196","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr197","Sayım Ve Tesellüm Noksanları","197","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr198","Diğer Çeşitli Dönen Varlıklar","198","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr199","Diğer Dönen Varlıklar Karşılığı(-)","199","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr220","Alıcılar","220","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr221","Alacak Senetleri","221","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr222","Alacak Senetleri Reeskontu(-)","222","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr224","Kazaqnılmamış Finansal Kiralama Faiz Gelirleri(-)","224","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr226","Verilen Depozito Ve Teminatlar","226","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr229","Şüpheli Ticari Alacaklar Karşılığı(-)","229","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr231","Ortaklardan Alacaklar","231","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr232","İştiraklerden Alacaklar","232","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr233","Bağlı Ortaklıklardan Alacaklar","233","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr235","Personelden Alacaklar","235","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr236","Diğer Çeşitli Alacaklar","236","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr237","Diğer Alacak Senetleri Reeskontu(-)","237","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr240","Bağlı Menkul Kıymetler","240","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr241","Bağlı Menkul Kıymetler Değer Düşüklüğü Karşılığı(-)","241","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr242","İştirakler","242","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr243","İştiraklere Sermaye Taahhütleri(-)","243","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr244","İştirakler Sermaye Payları Değer Düşüklüğü Karşılığı(-)","244","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr245","Bağlı Ortaklıklar","245","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr246","Bağlı Ortaklıklara Sermaye Taahhütleri(-)","246","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr247","Bağlı Ortaklıklar Sermaye Payları Değer Düşüklüğü Karşılığı(-)","247","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr248","Diğer Mali Duran Varlıklar","248","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr249","Diğer Mali Duran Varlıklar Karşılığı(-)","249","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr250","Arazi Ve Arsalar","250","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr251","Yer Altı Ve Yer Üstü Düzenleri","251","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr252","Binalar","252","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr253","Tesis, Makine Ve Cihazlar","253","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr254","Taşıtlar","254","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr255","Demirbaşlar","255","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr256","Diğer Maddi Duran Varlıklar","256","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr257","Birikmiş Amortismanlar(-)","257","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr258","Yapılmakta Olan Yatırımlar","258","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr259","Verilen Avanslar","259","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr260","Haklar","260","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr261","Şerefiye","261","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr262","Kuruluş Ve Örgütlenme Giderleri","262","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr263","Araştırma Ve Geliştirme Giderleri","263","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr264","Özel Maliyetler","264","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr267","Diğer Maddi Olmayan Duran Varlıklar","267","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr268","Birikmiş Amortismanlar(-)","268","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr269","Verilen Avanslar","269","account.data_account_type_fixed_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr271","Arama Giderleri","271","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr272","Hazırlık Ve Geliştirme Giderleri","272","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr277","Diğer Özel Tükenmeye Tabi Varlıklar","277","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr278","Birikmiş Tükenme Payları(-)","278","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr279","Verilen Avanslar","279","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr280","Gelecek Yıllara Ait Giderler","280","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr281","Gelir Tahakkukları","281","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr291","Gelecek Yıllarda İndirilecek KDV","291","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr292","Diğer KDV","292","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr293","Gelecek Yıllar İhtiyacı Stoklar","293","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr294","Elden Çıkarılacak Stoklar Ve Maddi Duran Varlıklar","294","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr295","Peşin Ödenen Vergi Ve Fonlar","295","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr297","Diğer Çeşitli Duran Varlıklar","297","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr298","Stok Değer Düşüklüğü Karşılığı(-)","298","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr299","Birikmiş Amortismanlar(-)","299","account.data_account_type_current_assets","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr300","Banka Kredileri","300","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr301","Finansal Kiralama İşlemlerinden Borçlar","301","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr302","Ertelenmiş Finansal Kiralama Borçlanma Maliyetleri(-)","302","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr303","Uzun Vadeli Kredilerin Anapara Taksitleri Ve Faizleri","303","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr304","Tahvil Anapara Borç, Taksit Ve Faizleri","304","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr305","Çıkarılan Bonolar Ve Senetler","305","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr306","Çıkarılmış Diğer Menkul Kıymetler","306","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr308","Menkul Kıymetler İhraç Farkı(-)","308","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr309","Diğer Mali Borçlar","309","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr320","Satıcılar","320","account.data_account_type_payable","l10n_tr.l10ntr_tek_duzen_hesap","True"
"tr321","Borç Senetleri","321","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr322","Borç Senetleri Reeskontu(-)","322","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr326","Alınan Depozito Ve Teminatlar","326","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr329","Diğer Ticari Borçlar","329","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr331","Ortaklara Borçlar","331","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr332","İştiraklere Borçlar","332","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr333","Bağlı Ortaklıklara Borçlar","333","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr335","Personele Borçlar","335","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr336","Diğer Çeşitli Borçlar","336","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr337","Diğer Borç Senetleri Reeskontu(-)","337","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr340","Alınan Sipariş Avansları","340","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr349","Alınan Diğer Avanslar","349","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr350","350 Yıllara Yaygın İnşaat Ve Onarım Hakedişleri Bedelleri","350","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr360","Ödenecek Vergi Ve Fonlar","360","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr361","Ödenecek Sosyal Güvenlük Kesintileri","361","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr368","Vadesi Geçmiş, Ertelenmiş Veya Taksitlendirilmiş Vergi Ve Diğer Yükümlülükler","368","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr369","Ödenecek Diğer Yükümlülükler","369","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr370","Dönem Kârı Vergi Ve Diğer Yasal Yükümlülük Karşılıkları","370","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr371","Dönem Kârının Peşin Ödenen Vergi Ve Diğer Yükümlülükler(-)","371","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr372","Kıdem Tazminatı Karşılığı","372","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr373","Maliyet Giderleri Karşılığı","373","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr379","Diğer Borç Ve Gider Karşılıkları","379","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr380","Gelecek Aylara Ait Gelirler","380","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr381","Gider Tahakkukları","381","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr391","Hesaplanan KDV","391","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr392","Diğer KDV","392","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr393","Merkez Ve Şubeler Cari Hesabı","393","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr397","Sayım Ve Tesellüm Fazlaları","397","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr399","Diğer Çeşitli Yabancı Kaynaklar","399","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr400","Banka Kredileri","400","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr401","Finansal Kiralama İşlemlerinden Borçlar","401","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr402","Ertelenmiş Finansal Kiralama Borçlanma Maliyetleri(-)","402","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr405","Çıkarılmış Tahviller","405","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr407","Çıkarılmış Diğer Menkul Kıymetler","407","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr408","Menkul Kıymetler İhraç Farkı(-)","408","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr409","Diğer Mali Borçlar","409","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr420","Satıcılar","420","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr421","Borç Senetleri","421","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr422","Borç Senetleri Reeskontu(-)","422","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr426","Alınan Depozito Ve Teminatlar","426","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr429","Diğer Ticari Borçlar","429","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr431","Ortaklara Borçlar","431","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr432","İştiraklere Borçlar","432","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr433","Bağlı Ortaklıklara Borçlar","433","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr436","Diğer Çeşitli Borçlar","436","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr437","Diğer Borç Senetleri Reeskontu(-)","437","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr438","Kamuya Olan Ertelenmiş Veya Taksitlendirilmiş Borçlar","438","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr440","Alınan Sipariş Avansları","440","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr449","Alınan Diğer Avanslar","449","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr472","Kıdem Tazminatı Karşılığı","472","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr479","Diğer Borç Ve Gider Karşılıkları","479","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr492","Gelecek Yıllara Ertelenmiş Veya Terkin Edilecek KDV","492","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr493","Tesise Katılma Payları","493","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr499","Diğer Çeşitli Uzun Vadeli Yabancı Kaynaklar","499","account.data_account_type_current_liabilities","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr500","Sermaye","500","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr501","Ödenmiş Sermaye(-)","501","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr520","Hisse Senetleri İhraç Primleri","520","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr521","Hisse Senedi İptal Kârları","521","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr522","Maddi Duran Varlık Yeniden Değerlenme Artışları","522","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr524","Maliyet Artışları Fonu","524","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr529","Diğer Sermaye Yedekleri","529","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr540","Yasal Yedekler","540","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr541","Statü Yedekleri","541","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr542","Olağanüstü Yedekler","542","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr548","Diğer Kâr Yedekleri","548","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr549","Özel Fonlar","549","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr570","Geçmiş Yıllar Kârları","570","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr580","Geçmiş Yıllar Zararları(-)","580","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr590","Dönem Net Kârı","590","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr591","Dönem Net Zararı(-)","591","account.data_account_type_equity","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr600","Yurt İçi Satışlar","600","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr601","Yurt Dışı Satışlar","601","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr602","Diğer Gelirler","602","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr610","Satıştan İadeler(-)","610","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr611","Satış İndirimleri(-)","611","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr612","Diğer İndirimler","612","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr620","Satılan Mamuller Maliyeti(-)","620","account.data_account_type_direct_costs","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr621","Satılan Ticari Mallar Maliyeti(-)","621","account.data_account_type_direct_costs","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr622","Satılan Hizmet Maliyeti(-)","622","account.data_account_type_direct_costs","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr623","Diğer Satışların Maliyeti(-)","623","account.data_account_type_direct_costs","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr630","Araştırma Ve Geliştirme Giderleri(-)","630","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr631","Pazarlama Satış Ve Dağıtım Giderleri(-)","631","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr632","Genel Yönetim Giderleri(-)","632","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr640","İştiraklerden Temettü Gelirleri","640","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr641","Bağlı Ortaklıklardan Temettü Gelirleri","641","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr642","Faiz Gelirleri","642","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr643","Komisyon Gelirleri","643","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr644","Konusu Kalmayan Karşılıklar","644","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr645","Menkul Kıymet Satış Kârları","645","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr646","Kambiyo Kârları","646","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr647","Reeskont Faiz Gelirleri","647","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr648","Enflasyon Düzeltme Kârları","648","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr649","Diğer Olağan Gelir Ve Kârlar","649","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr653","Komisyon Giderleri(-)","653","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr654","Karşılık Giderleri(-)","654","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr655","Menkul Kıymet Satış Zararları(-)","655","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr656","Kambiyo Zararları(-)","656","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr657","Reeskont Faiz Giderleri(-)","657","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr658","Enflasyon Düzeltmesi Zararları(-)","658","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr659","Diğer Olağan Gider Ve Zararlar(-)","659","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr660","Kısa Vadeli Borçlanma Giderleri(-)","660","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr661","Uzun Vadeli Borçlanma Giderleri(-)","661","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr671","Önceki Dönem Gelir Ve Kârları","671","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr679","Diğer Olağan Dışı Gelir Ve Kârlar","679","account.data_account_type_revenue","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr680","Çalışmayan Kısım Gider Ve Zararları(-)","680","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr681","Önceki Dönem Gider Ve Zararları(-)","681","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr689","Diğer Olağan Dışı Gider Ve Zararlar(-)","689","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr690","Dönem Kârı Veya Zararı","690","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr691","Dönem Kârı Vergi Ve Diğer Yasal Yükümlülük Karşılıkları(-)","691","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr692","Dönem Net Kârı Veya Zararı","692","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr697","Yıllara Yaygın İnşaat Ve Enflasyon Düzeltme Hesabı","697","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr698","Enflasyon Düzeltme Hesabı","698","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr700","Maliyet Muhasebesi Bağlantı Hesabı","700","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr701","Maliyet Muhasebesi Yansıtma Hesabı","701","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr710","Direk İlk Madde Ve Malzeme Giderleri Hesabı","710","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr711","Direkt İlk Madde Ve Malzeme Yansıtma Hesabı","711","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr712","Direkt İlk Madde Ve Malzeme Fiyat Farkı","712","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr713","Direkt İlk Madde Ve Malzeme Miktar Farkı","713","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr720","Direkt İşçilik Giderleri","720","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr721","Direkt İşçilik Giderleri Yansıtma Hesabı","721","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr722","Direkt İşçilik Ücret Farkları","722","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr723","Direkt İşçilik Süre Farkları","723","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr730","Genel Üretim Giderleri","730","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr731","Genel Üretim Giderleri Yansıtma Hesabı","731","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr732","Genel Üretim Giderleri Bütçe Farkları","732","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr733","Genel Üretim Giderleri Verimlilik Giderleri","733","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr734","Genel Üretim Giderleri Kapasite Farkları","734","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr740","Hizmet Üretim Maliyeti","740","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr741","Hizmet Üretim Maliyeti Yansıtma Hesabı","741","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr742","Hizmet Üretim Maliyeti Fark Hesapları","742","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr760","Atraştırma Ve Geliştirme Giderleri","760","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr761","Pazarlama Satış Ve Dagıtım Giderleri Yansıtma Hesabı","761","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr762","Pazarlama Satış Ve Dağıtım Giderleri Fark Hesabı","762","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr770","Genel Yönetim Giderleri","770","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr771","Genel Yönetim Giderleri Yansıtma Hesabı","771","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr772","Genel Yönetim Gider Farkları Hesabı","772","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr780","Finansman Giderleri","780","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr781","Finansman Giderleri Yansıtma Hesabı","781","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
"tr782","Finansman Giderleri Fark Hesabı","782","account.data_account_type_expenses","l10n_tr.l10ntr_tek_duzen_hesap","False"
```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_tr.l10ntr_tek_duzen_hesap')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_kdv_18" model="account.tax.group">
            <field name="name">KDV %18</field>
            <field name="country_id" ref="base.tr"/>
        </record>
        <record id="tax_group_kdv_20" model="account.tax.group">
            <field name="name">KDV 20%</field>
            <field name="country_id" ref="base.tr"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- account.tax.template (pre-2023) -->
    <record id="tr_kdv_satis_sale_18" model="account.tax.template">
        <field name="sequence">11</field>
        <field name="description">KDV %18(sale)</field>
        <field name="name">KDV %18(sale)</field>
        <field name="price_include" eval="0"/>
        <field name="amount">18</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="l10ntr_tek_duzen_hesap"/>
        <field name="tax_group_id" ref="tax_group_kdv_18"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('tr391'),
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
                'account_id': ref('tr191'),
            }),
        ]"/>
    </record>

    <record id="tr_kdv_satis_purchase_18" model="account.tax.template">
        <field name="sequence">11</field>
        <field name="description">KDV %18(purchase)</field>
        <field name="name">KDV %18(purchase)</field>
        <field name="price_include" eval="0"/>
        <field name="amount">18</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="l10ntr_tek_duzen_hesap"/>
        <field name="tax_group_id" ref="tax_group_kdv_18"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('tr391'),
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
                'account_id': ref('tr191'),
            }),
        ]"/>
    </record>

    <!-- account.tax.template (introduced in July 2023) -->
    <record id="tr_kdv_satis_sale_20" model="account.tax.template">
        <field name="sequence">13</field>
        <field name="description">KDV 20%</field>
        <field name="name">20%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="l10ntr_tek_duzen_hesap"/>
        <field name="tax_group_id" ref="tax_group_kdv_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('tr391'),
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
                'account_id': ref('tr191'),
            }),
        ]"/>
    </record>

    <record id="tr_kdv_satis_purchase_20" model="account.tax.template">
        <field name="sequence">14</field>
        <field name="description">KDV 20%</field>
        <field name="name">20%</field>
        <field name="price_include" eval="0"/>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="l10ntr_tek_duzen_hesap"/>
        <field name="tax_group_id" ref="tax_group_kdv_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5,0,0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('tr391'),
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
                'account_id': ref('tr191'),
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\l10n_tr_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Chart template of l10n_tr -->
    <record id="l10ntr_tek_duzen_hesap" model="account.chart.template">
        <field name="name">Tek Düzen Hesap Planı</field>
        <field name="bank_account_code_prefix">102</field>
        <field name="cash_account_code_prefix">100</field>
        <field name="transfer_account_code_prefix">103</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.TRY"/>
        <field name="country_id" ref="base.tr"/>
    </record>

</odoo>

```

## File: data\l10n_tr_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Chart template -->
    <record id="l10ntr_tek_duzen_hesap" model="account.chart.template">
        <field name="property_account_receivable_id" ref="tr120"/>
        <field name="property_account_payable_id" ref="tr320"/>
        <field name="property_account_expense_categ_id" ref="tr150"/>
        <field name="property_account_income_categ_id" ref="tr600"/>
        <field name="income_currency_exchange_account_id" ref="tr646"/>
        <field name="expense_currency_exchange_account_id" ref="tr656"/>
        <field name="default_pos_receivable_account_id" ref="tr123" />
    </record>
</odoo>

```

## File: migrations\1.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.addons.account.models.chart_template import update_taxes_from_templates


def migrate(cr, version):
    update_taxes_from_templates(cr, 'l10n_tr.l10ntr_tek_duzen_hesap')

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="5.95" width="50.4" height="35.1" maskUnits="userSpaceOnUse">
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
      <image width="125" height="83" transform="translate(4.8 5.95) scale(0.4 0.42)" xlink:href="data:image/png;base64,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"/>
    </g>
  </g>
</svg>

```

