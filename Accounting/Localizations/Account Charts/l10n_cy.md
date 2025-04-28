# Odoo Module: l10n_cy

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
from . import models

```

## File: __manifest__.py

```python
{
    'name': 'Cyprus - Accounting',
    'icon': '/account/static/description/l10n.png',
    'countries': ['cy'],
    'description': """
Basic package for Cyprus that contains the chart of accounts, taxes, tax reports,...
    """,
    'license': 'LGPL-3',
    'version': '1.0',
    'category': 'Accounting/Localizations/Account Charts',
    'depends': [
        'account',
        'base_vat',
    ],
    'data': [
        'data/menuitem_data.xml',
        'data/account_tax_report_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
}

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8" ?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.cy"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_1" model="account.report.line">
                <field name="name">1. VAT due on sales and other outputs</field>
                <field name="code">cy_1</field>
                <field name="expression_ids">
                    <record id="tax_report_line_1_amount" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">1</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_2" model="account.report.line">
                <field name="name">2. VAT due on acquisitions from other EU Member States</field>
                <field name="code">cy_2</field>
                <field name="expression_ids">
                    <record id="tax_report_line_2_amount" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">2</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_3" model="account.report.line">
                <field name="name">3. Total output VAT (sum of boxes 1 and 2)</field>
                <field name="hierarchy_level">0</field>
                <field name="code">cy_3</field>
                <field name="expression_ids">
                    <record id="tax_report_line_3_amount" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">cy_1.balance + cy_2.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_4" model="account.report.line">
                <field name="name">4. Input VAT (including acquisitions from other EU Member States)</field>
                <field name="code">cy_4</field>
                <field name="expression_ids">
                    <record id="tax_report_line_4_amount" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">4</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_5" model="account.report.line">
                <field name="name">5. VAT payable or refundable (difference between box 4 and 3)</field>
                <field name="code">cy_5</field>
                <field name="expression_ids">
                    <record id="tax_report_line_5_amount" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">cy_4.balance - cy_3.balance</field>
                        <field name="green_on_positive">False</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_6" model="account.report.line">
                <field name="name">6. Value of total sales and other outputs (excluding VAT)(including box 10)</field>
                <field name="code">cy_6</field>
                <field name="expression_ids">
                    <record id="tax_report_line_6_amount" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">cy_6.sub_balance + cy_10.balance</field>
                    </record>
                    <record id="tax_report_line_6_sub_amount" model="account.report.expression">
                        <field name="label">sub_balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">6</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_7" model="account.report.line">
                <field name="name">7. Value of total purchases and other inputs (excluding VAT)</field>
                <field name="code">cy_7</field>
                <field name="expression_ids">
                    <record id="tax_report_line_7_amount" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">7</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_8A" model="account.report.line">
                <field name="name">8A. Total value of all supplies of goods (and directly related services) to other EU Member States</field>
                <field name="code">cy_8A</field>
                <field name="expression_ids">
                    <record id="tax_report_line_8A_amount" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">8A</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_8B" model="account.report.line">
                <field name="name">8B. Total value of services provided to taxable persons in other EU Member States</field>
                <field name="code">cy_8B</field>
                <field name="expression_ids">
                    <record id="tax_report_line_8B_amount" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">8B</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_9" model="account.report.line">
                <field name="name">9. Total value of sales taxed at the rate of 0% (other than those included in box 8A)</field>
                <field name="code">cy_9</field>
                <field name="expression_ids">
                    <record id="tax_report_line_9_amount" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">9</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_10" model="account.report.line">
                <field name="name">10. Total value of sales which are outside the scope of Cyprus VAT</field>
                <field name="code">cy_10</field>
                <field name="expression_ids">
                    <record id="tax_report_line_10_amount" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">external</field>
                        <field name="formula">sum</field>
                        <field name="subformula">editable</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_11A" model="account.report.line">
                <field name="name">11A. Total value of acquisitions of goods (and directly related services) from other EU Member States</field>
                <field name="code">cy_11A</field>
                <field name="expression_ids">
                    <record id="tax_report_line_11A_amount" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">11A</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_11B" model="account.report.line">
                <field name="name">11B. Total value of services received from taxable persons residence in other EU Member States</field>
                <field name="code">cy_11B</field>
                <field name="expression_ids">
                    <record id="tax_report_line_11B_amount" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">tax_tags</field>
                        <field name="formula">11B</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\menuitem_data.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<odoo>
    <menuitem id="account_reports_cy_statements_menu" name="Cyprus" parent="account.menu_finance_reports" sequence="5" groups="account.group_account_readonly"/>
</odoo>

```

## File: data\template\account.account-cy.csv

```csv
"id","code","name","account_type","reconcile","name@tr","name@gr"
"cy_0010","0010","Patents","asset_non_current","False","Patentler","Διπλώματα ευρεσιτεχνίας"
"cy_0020","0020","Trademarks","asset_non_current","False","Ticari markalar","Εμπορικά σήματα"
"cy_0030","0030","Licenses","asset_non_current","False","Lisanslar","Άδειες"
"cy_0040","0040","Software","asset_non_current","False","Yazılım","Λογισμικό"
"cy_0050","0050","Accumulated patents amortization","asset_non_current","False","Birikmiş patent amortismanı","Σωρευμένες αποσβέσεις διπλωμάτων ευρεσιτεχνίας"
"cy_0051","0051","Accumulated trademarks amortization","asset_non_current","False","Birikmiş ticari marka amortismanı","Συσσωρευμένη απόσβεση εμπορικών σημάτων"
"cy_0052","0052","Accumulated licenses amortization","asset_non_current","False","Birikmiş lisans amortismanı","Σωρευμένες αποσβέσεις αδειών"
"cy_0053","0053","Accumulated software amortization","asset_non_current","False","Birikmiş yazılım amortismanı","Συσσωρευμένη απόσβεση λογισμικού"
"cy_0060","0060","Freehold property","asset_fixed","False","Mülkiyet","Ελεύθερη ιδιοκτησία"
"cy_0070","0070","Leasehold property","asset_fixed","False","Kiralık mülk","Μίσθωση ακινήτων"
"cy_0080","0080","Plant and machinery","asset_fixed","False","Tesis ve makineler","Εγκαταστάσεις και μηχανήματα"
"cy_0090","0090","Office equipment","asset_fixed","False","Ofis malzemesi","Εξοπλισμός γραφείου"
"cy_0100","0100","Computers","asset_fixed","False","Bilgisayarlar","Υπολογιστές"
"cy_0110","0110","Furniture and fixtures","asset_fixed","False","Mobilya ve demirbaşlar","Έπιπλα και φωτιστικά"
"cy_0120","0120","Warehouse equipment","asset_fixed","False","Depo ekipmanları","Εξοπλισμός αποθήκης"
"cy_0130","0130","Motor vehicles","asset_fixed","False","Motorlu Taşıtlar","Μηχανοκίνητα οχήματα"
"cy_0140","0140","Accumulated leasehold property depreciation","asset_fixed","False","Birikmiş özel mülk amortismanı","Σωρευμένες αποσβέσεις μισθωμένων ακινήτων"
"cy_0141","0141","Accumulated plant and machinery depreciation","asset_fixed","False","Birikmiş tesis ve makine amortismanı","Σωρευμένες αποσβέσεις εγκαταστάσεων και μηχανημάτων"
"cy_0142","0142","Accumulated office equipment depreciation","asset_fixed","False","Birikmiş ofis ekipmanı amortismanı","Σωρευμένες αποσβέσεις εξοπλισμού γραφείου"
"cy_0143","0143","Accumulated computers depreciation","asset_fixed","False","Birikmiş bilgisayar amortismanı","Σωρευμένες αποσβέσεις υπολογιστών"
"cy_0144","0144","Accumulated furniture and fixtures depreciation","asset_fixed","False","Birikmiş mobilya ve demirbaş amortismanı","Σωρευμένες αποσβέσεις επίπλων και ειδών"
"cy_0145","0145","Accumulated warehouse equipment depreciation","asset_fixed","False","Birikmiş depo ekipmanı amortismanı","Σωρευμένες αποσβέσεις εξοπλισμού αποθήκης"
"cy_0146","0146","Accumulated motor vehicles depreciation","asset_fixed","False","Birikmiş motorlu taşıt amortismanı","Σωρευμένες αποσβέσεις μηχανοκίνητων οχημάτων"
"cy_0150","0150","Long-term loan","asset_non_current","False","Uzun vadeli kredi","Μακροπρόθεσμο δάνειο"
"cy_0160","0160","Other long-term investments","asset_non_current","False","Diğer uzun vadeli yatırımlar","Άλλες μακροπρόθεσμες επενδύσεις"
"cy_1000","1000","Stock","asset_current","False","Stoklamak","Στοκ"
"cy_1010","1010","Work in progress","asset_current","False","Çalışma devam ediyor","Εργασία σε εξέλιξη"
"cy_1020","1020","Raw materials","asset_current","False","İşlenmemiş içerikler","Πρώτες ύλες"
"cy_1100","1100","Accounts receivable","asset_receivable","True","Alacak hesapları","Εισπρακτέοι λογαριασμοί"
"cy_1090","1090","Advance payments","asset_prepayments","False","Peşin ödemeler","Προκαταβολές"
"cy_1110","1110","Overpayment","asset_current","False","Fazla ödeme","Υπερπληρωμή"
"cy_1120","1120","Other debtors","asset_receivable","True","Diğer borçlular","Άλλοι οφειλέτες"
"cy_1140","1140","Accrued income","asset_receivable","True","Gelir tahakkukları","Δεδουλευμένα έσοδα"
"cy_1150","1150","Employee advances (business expenses)","asset_receivable","True","Çalışan avansları (iş giderleri)","Προκαταβολές εργαζομένων (επιχειρηματικά έξοδα)"
"cy_1160","1160","Prepaid expenses","asset_current","False","Önceden ödenmiş giderler","Προπληρωθέντα έξοδα"
"cy_1170","1170","Short-term loans","asset_current","False","Kısa vadeli krediler","Βραχυπρόθεσμα δάνεια"
"cy_1180","1180","Other short-term investments","asset_current","False","Diğer kısa vadeli yatırımlar","Άλλες βραχυπρόθεσμες επενδύσεις"
"cy_1210","1210","Deposit account","asset_current","False","Mevduat hesabı","Καταθετικός λογαριασμός"
"cy_1230","1230","Petty cash","asset_cash","False","Küçük kasa","Μικρό ταμείο"
"cy_1240","1240","Credit card","asset_cash","False","Kredi kartı","Πιστωτική κάρτα"
"cy_1250","1250","In-transit account","asset_current","False","Transit hesap","Λογαριασμός κατά τη μεταφορά"
"cy_1260","1260","Suspense account","asset_current","False","Askıya alma hesabı","Λογαριασμός αναστολής"
"cy_2100","2100","Accounts payable","liability_payable","True","Ödenebilir hesaplar","Πληρωτέοι λογαριασμοί"
"cy_2120","2120","Other creditors","liability_payable","True","Diğer alacaklılar","Άλλοι πιστωτές"
"cy_2130","2130","Reimbursable expenses","liability_current","False","Geri ödenebilir giderler","Επιστρεφόμενα έξοδα"
"cy_2140","2140","Accruals","liability_current","False","Tahakkuklar","δεδουλευμένα"
"cy_2150","2150","Customer overpayment","liability_current","False","Müşteri fazla ödemesi","Υπερπληρωμή πελάτη"
"cy_2160","2160","Deposit received","liability_current","False","Alınan depozito","Λήφθηκε κατάθεση"
"cy_2200","2200","VAT control account","liability_current","False","KDV kontrol hesabı","Λογαριασμός ελέγχου ΦΠΑ"
"cy_2201","2201","Output VAT (Sales)","liability_current","False","Çıkış KDV'si (Satış)","ΦΠΑ εκροών (Πωλήσεις)"
"cy_2202","2202","Input VAT (Purchases)","liability_current","False","Girilen KDV (Satın Alma İşlemleri)","ΦΠΑ εισροών (Αγορές)"
"cy_2203","2203","VAT reverse charge","liability_current","False","KDV ters ücreti","Αντίστροφη επιβάρυνση ΦΠΑ"
"cy_2204","2204","Import VAT","liability_current","False","İthalat KDV'si","ΦΠΑ εισαγωγής"
"cy_2209","2209","VAT holding account","liability_current","False","KDV saklama hesabı","Λογαριασμός αποθέματος ΦΠΑ"
"cy_2210","2210","PAYE/NIC","liability_current","False","ÖDEME/NIC","PAYE/NIC"
"cy_2220","2220","Income Tax","liability_current","False","Gelir vergisi","Φόρος εισοδήματος"
"cy_2250","2250","Net wages","liability_payable","True","Net ücret","Καθαρός μισθός"
"cy_2260","2260","Dividends payable","liability_current","False","Ödenecek borç","Μερίσματα πληρωτέα"
"cy_2270","2270","Deferred revenue","liability_current","False","Ertelenmiş gelir","Αναβαλλόμενα έσοδα"
"cy_2280","2280","Short-term loans","liability_current","False","Kısa vadeli krediler","Βραχυπρόθεσμα δάνεια"
"cy_2300","2300","Long-term loans","liability_non_current","False","Uzun vadeli krediler","Μακροπρόθεσμα δάνεια"
"cy_3000","3000","Share capital","equity","False","Sermaye","Μετοχικό κεφάλαιο"
"cy_3090","3090","Opening balances","equity","False","Açılış bakiyeleri","Ανοιχτά υπόλοιπα"
"cy_3100","3100","Reserves","equity","False","Rezervler","Αποθεματικά"
"cy_3200","3200","Profit and loss account","equity_unaffected","False","Kar ve zarar hesabı","Λογαριασμός κερδών και ζημιών"
"cy_3201","3201","Net profit/loss current period","equity","False","Net kar/zarar cari dönem","Καθαρά κέρδη/ζημίες τρέχουσα περίοδος"
"cy_4000","4000","Sales","income","False","Satış","Εκπτώσεις"
"cy_4090","4090","Discounts allowed","expense","False","İndirimlere izin veriliyor","Εκπτώσεις δοθείσες"
"cy_4200","4200","Sales of assets","income_other","False","Varlık satışı","Πωλήσεις περιουσιακών στοιχείων"
"cy_4900","4900","Other sales","income_other","False","Diğer satışlar","Άλλες πωλήσεις"
"cy_4901","4901","Royalties received","income_other","False","Alınan telif hakları","Λήφθηκαν δικαιώματα"
"cy_4902","4902","Commissions received","income_other","False","Alınan komisyonlar","Λήφθηκαν προμήθειες"
"cy_4903","4903","Insurance claims","income_other","False","Sigorta talepleri","Ασφαλιστικές απαιτήσεις"
"cy_4904","4904","Rent income","income_other","False","Kira geliri","Έσοδα από ενοίκια"
"cy_4905","4905","Distribution and carriage","income_other","False","Dağıtım ve taşıma","Διανομή και μεταφορά"
"cy_4906","4906","Bank interest received","income_other","False","Alınan banka faizi","Τραπεζικοί τόκοι που εισπράχθηκαν"
"cy_5000","5000","Cost of goods","expense_direct_cost","False","Malların maliyeti","Κόστος αγαθών"
"cy_5010","5010","Stock adjustments","expense_direct_cost","False","Stok ayarlamaları","Προσαρμογές μετοχών"
"cy_5090","5090","Discounts taken","income_other","False","Alınan indirimler","Πραγματοποιήθηκαν εκπτώσεις"
"cy_5100","5100","Carriage","expense","False","Taşıma","Μεταφορά"
"cy_5101","5101","Import duty","expense","False","İthalat vergisi","Εισαγωγικός δασμός"
"cy_5102","5102","Transport insurance","expense","False","Nakliye sigortası","Ασφάλιση μεταφοράς"
"cy_5103","5103","Packaging","expense","False","Ambalajlama","Συσκευασία"
"cy_5104","5104","Miscellaneous purchases","expense","False","Çeşitli satın almalar","Διάφορες αγορές"
"cy_6000","6000","Productive labour","expense","False","Üretken emek","Παραγωγική εργασία"
"cy_6001","6001","Sales labour","expense","False","Satış emeği","Εργασία πωλήσεων"
"cy_6002","6002","Sub-contractors","expense","False","Taşeronlar","Υπεργολάβοι"
"cy_6100","6100","Sales commissions","expense","False","Satış komisyonları","Προμήθειες πωλήσεων"
"cy_6200","6200","Sales promotions","expense","False","Satış promosyonu","Προωθήσεις πωλήσεων"
"cy_6201","6201","Advertising","expense","False","Reklam","Διαφήμιση"
"cy_6202","6202","Gifts and samples","expense","False","Hediyeler ve örnekler","Δώρα και δείγματα"
"cy_6203","6203","PR (literature & brochures)","expense","False","Halkla İlişkiler (literatür & broşürler)","PR (λογοτεχνία & μπροσούρες)"
"cy_6900","6900","Miscellaneous expenses","expense","False","Çeşitli masraflar","Διάφορα έξοδα"
"cy_7000","7000","Gross wages","expense","False","Brüt ücret","Ακαθάριστους μισθούς"
"cy_7001","7001","Directors salaries","expense","False","Yönetici maaşları","Μισθοί διευθυντών"
"cy_7002","7002","Directors bonus","expense","False","Yönetmen bonusu","Μπόνους σκηνοθέτη"
"cy_7003","7003","Staff bonus","expense","False","Personel bonusu","Μπόνους προσωπικού"
"cy_7005","7005","Recruitment fees","expense","False","İşe alım ücretleri","Αμοιβές πρόσληψης"
"cy_7006","7006","Employers n.i.","expense","False","İşverenler","Εργοδότες n.i."
"cy_7007","7007","Employers pensions","expense","False","İşveren emekli maaşları","Συντάξεις εργοδοτών"
"cy_7008","7008","Employee benefits healthcare","expense","False","Çalışan","Οι εργαζόμενοι επωφελούνται από την υγειονομική περίθαλψη"
"cy_7009","7009","Employee benefits phi life assurance","expense","False","Çalışanlara sağlanan faydalar phi hayat sigortası","Παροχές εργαζομένων phi ασφάλιση ζωής"
"cy_7010","7010","SSP reclaimed","expense","False","SSP geri alındı","Το SSP ανακτήθηκε"
"cy_7011","7011","SMP reclaimed","expense","False","SMP geri kazanıldı","Το SMP ανακτήθηκε"
"cy_7012","7012","Staff entertainment","expense","False","Personel eğlencesi","Ψυχαγωγία προσωπικού"
"cy_7100","7100","Rent","expense","False","Kira","Ενοίκιο"
"cy_7102","7102","Water rates","expense","False","Su oranları","Τιμές νερού"
"cy_7103","7103","General rates","expense","False","Genel oranlar","Γενικές τιμές"
"cy_7104","7104","Premises & liability insurance","expense","False","Tesis ve sorumluluk sigortası","Ασφάλιση χώρων & αστικής ευθύνης"
"cy_7190","7190","Utilities","expense","False","Araçlar","Βοηθητικά προγράμματα"
"cy_7200","7200","Electricity","expense","False","Elektrik","Ηλεκτρική ενέργεια"
"cy_7201","7201","Gas","expense","False","Gaz","Αέριο"
"cy_7202","7202","Oil","expense","False","Yağ","Λάδι"
"cy_7203","7203","Other heating costs","expense","False","Diğer ısıtma maliyetleri","Άλλα έξοδα θέρμανσης"
"cy_7300","7300","Car fuel & oil","expense","False","Araba yakıtı ve yağı","Καύσιμα & λάδια αυτοκινήτου"
"cy_7301","7301","Repairs and servicing","expense","False","Onarım ve servis","Επισκευές και σέρβις"
"cy_7302","7302","Licenses & mot's","expense","False","Lisanslar ve mot'lar","Άδειες & μηχανήματα"
"cy_7303","7303","Vehicle insurance","expense","False","Araç sigortası","Ασφάλιση οχήματος"
"cy_7304","7304","Miscellaneous motor expenses","expense","False","Çeşitli motor masrafları","Διάφορα έξοδα κινητήρα"
"cy_7400","7400","Traveling","expense","False","Seyahat","Ταξίδια"
"cy_7401","7401","Car hire","expense","False","Araba kiralama","Ενοικίαση αυτοκινήτων"
"cy_7402","7402","Hotels","expense","False","Oteller","Ξενοδοχεία"
"cy_7403","7403","Entertainment","expense","False","Eğlence","Ψυχαγωγία"
"cy_7405","7405","Overseas traveling","expense","False","Yurtdışı seyahati","Ταξίδια στο εξωτερικό"
"cy_7406","7406","Subsistence & refreshments","expense","False","Harcama ve içecekler","Διατροφή & αναψυκτικά"
"cy_7407","7407","Travel insurance","expense","False","Seyahat sigortası","Ταξιδιωτική ασφάλιση"
"cy_7490","7490","Other operating expenses","expense","False","Diğer işletme giderleri","Λοιπά λειτουργικά έξοδα"
"cy_7500","7500","Printing","expense","False","Baskı","Εκτύπωση"
"cy_7501","7501","Postage","expense","False","Posta ücreti","Ταχυδρομικά τέλη"
"cy_7502","7502","Telephone","expense","False","Telefon","Τηλέφωνο"
"cy_7503","7503","Internet","expense","False","internet","Διαδίκτυο"
"cy_7504","7504","Office stationery","expense","False","Ofis kırtasiye","Χαρτικά γραφείου"
"cy_7505","7505","Books & subscriptions","expense","False","Kitaplar ve abonelikler","Βιβλία και συνδρομές"
"cy_7506","7506","Training costs","expense","False","Eğitim maliyetleri","Κόστος εκπαίδευσης"
"cy_7507","7507","Donations","expense","False","Bağışlar","Δωρεές"
"cy_7508","7508","Computer software","expense","False","Bilgisayar yazılımı","Λογισμικό Ηλεκτρονικών Υπολογιστών"
"cy_7600","7600","Legal fees","expense","False","Yasal ücretler","Νομικές αμοιβές"
"cy_7601","7601","Audit and accountancy fees","expense","False","Denetim ve muhasebe ücretleri","Αμοιβές ελέγχου και λογιστικής"
"cy_7602","7602","Consultancy fees","expense","False","Danışmanlık ücretleri","Συμβουλευτικές αμοιβές"
"cy_7603","7603","Professional fees","expense","False","Profesyonel ücretler","Επαγγελματικές αμοιβές"
"cy_7604","7604","Memberships","expense","False","Üyelikler","Συνδρομές"
"cy_7700","7700","Equipment hire","expense","False","Ekipman kiralama","Ενοικίαση εξοπλισμού"
"cy_7701","7701","Office machine maintenance","expense","False","Ofis makinesi bakımı","Συντήρηση μηχανών γραφείου"
"cy_7800","7800","Repairs and renewals","expense","False","Onarımlar ve yenilemeler","Επισκευές και ανανεώσεις"
"cy_7810","7810","Premises expenses","expense","False","Tesis giderleri","Έξοδα χώρων"
"cy_7900","7900","Bank interest paid","expense","False","Ödenen banka faizi","Καταβληθέντες τραπεζικοί τόκοι"
"cy_7901","7901","Bank charges","expense","False","banka masrafları","τραπεζικές χρεώσεις"
"cy_7902","7902","Currency charges","expense","False","Döviz ücretleri","Χρεώσεις συναλλάγματος"
"cy_7903","7903","Loan interest paid","expense","False","Ödenen kredi faizi","Πληρωμένοι τόκοι δανείου"
"cy_7904","7904","H.P. interest","expense","False","H.P. faiz","ΙΠΠΟΔΥΝΑΜΗ. ενδιαφέρον"
"cy_7905","7905","Credit charges","expense","False","Kredi ücretleri","Πιστωτικές χρεώσεις"
"cy_7910","7910","Exchange rate realized gains/losses","income_other","False","Döviz kurunun gerçekleşen kazançları/zararları","Συναλλαγματική ισοτιμία πραγματοποιήθηκαν κέρδη/ζημίες"
"cy_7911","7911","Exchange rate unrealized gains/losses","income_other","False","Döviz kuru gerçekleşmemiş kar/zarar","Μη πραγματοποιηθέντα κέρδη/ζημίες συναλλαγματικών ισοτιμιών"
"cy_8001","8001","Patents amortization","expense_depreciation","False","Patent amortismanı","Απόσβεση διπλωμάτων ευρεσιτεχνίας"
"cy_8002","8002","Trademarks amortization","expense_depreciation","False","Ticari marka amortismanı","Απόσβεση εμπορικών σημάτων"
"cy_8003","8003","Licenses amortization","expense_depreciation","False","Lisans amortismanı","Απόσβεση αδειών"
"cy_8004","8004","Software amortization","expense_depreciation","False","Yazılım amortismanı","Απόσβεση λογισμικού"
"cy_8006","8006","Leasehold property depreciation","expense_depreciation","False","Kiralık mülk amortismanı","Απόσβεση ακινήτων"
"cy_8007","8007","Plant and machinery depreciation","expense_depreciation","False","Tesis ve makine amortismanı","Απόσβεση φυτών και μηχανημάτων"
"cy_8008","8008","Office equipment depreciation","expense_depreciation","False","Ofis ekipmanı amortismanı","Απόσβεση εξοπλισμού γραφείου"
"cy_8009","8009","Computers depreciation","expense_depreciation","False","Bilgisayar amortismanı","Απόσβεση υπολογιστών"
"cy_8010","8010","Furniture and fixtures depreciation","expense_depreciation","False","Mobilya ve demirbaş amortismanı","Αποσβέσεις επίπλων και φωτιστικών"
"cy_8011","8011","Warehouse equipment depreciation","expense_depreciation","False","Depo ekipmanı amortismanı","Απόσβεση εξοπλισμού αποθήκης"
"cy_8012","8012","Motor vehicles depreciation","expense_depreciation","False","Motorlu taşıtların amortismanı","Απόσβεση μηχανοκίνητων οχημάτων"
"cy_8100","8100","Bad debt write off","expense","False","Kötü borç siliniyor","Διαγραφή επισφαλών χρεών"
"cy_8102","8102","Bad debt provision","expense","False","Şüpheli alacak karşılığı","Πρόβλεψη για επισφαλείς απαιτήσεις"
"cy_8200","8200","Other non-operating income or expenses","expense","False","Diğer faaliyet dışı gelir veya giderler","Άλλα μη λειτουργικά έσοδα ή έξοδα"
"cy_8300","8300","Disposals","expense","False","Bertaraflar","Διαθέσεις"
"cy_8400","8400","Income tax expenses","expense","False","Gelir vergisi giderleri","Έξοδα φόρου εισοδήματος"

```

## File: data\template\account.fiscal.position-cy.csv

```csv
"id","sequence","name","name@gr","name@tr","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","account_ids/account_src_id","account_ids/account_dest_id"
"fiscal_position_cy_exempt","1","Exempt taxpayer","Απαλλασσόμενος φορολογούμενος","Vergiden muaf mükellef","","","","","VAT_S_IN_CY_19_G","EXEMPT_S","",""
"","","","","","","","","","VAT_S_IN_CY_19_S","EXEMPT_S","",""
"","","","","","","","","","VAT_S_IN_CY_9_G","EXEMPT_S","",""
"","","","","","","","","","VAT_S_IN_CY_9_S","EXEMPT_S","",""
"","","","","","","","","","VAT_S_IN_CY_5_G","EXEMPT_S","",""
"","","","","","","","","","VAT_S_IN_CY_5_S","EXEMPT_S","",""
"","","","","","","","","","VAT_S_IN_CY_3_G","EXEMPT_S","",""
"","","","","","","","","","VAT_S_IN_CY_3_S","EXEMPT_S","",""
"","","","","","","","","","VAT_P_IN_CY_19_G","EXEMPT_P","",""
"","","","","","","","","","VAT_P_IN_CY_19_S","EXEMPT_P","",""
"","","","","","","","","","VAT_P_IN_CY_9_G","EXEMPT_P","",""
"","","","","","","","","","VAT_P_IN_CY_9_S","EXEMPT_P","",""
"","","","","","","","","","VAT_P_IN_CY_5_G","EXEMPT_P","",""
"","","","","","","","","","VAT_P_IN_CY_5_S","EXEMPT_P","",""
"","","","","","","","","","VAT_P_IN_CY_3_G","EXEMPT_P","",""
"","","","","","","","","","VAT_P_IN_CY_3_S","EXEMPT_P","",""
"fiscal_position_cy_national","2","Domestic","Οικιακός","Yerel","1","1","base.cy","","","","",""
"fiscal_position_cy_person_private","3","Private partner","Ιδιωτικός συνεργάτης","Özel ortak","1","","","base.europe","","","",""
"fiscal_position_cy_eu","4","EU partner","εταίρο της ΕΕ","AB ortağı","1","1","","base.europe","VAT_S_IN_CY_19_G","VAT_S_IN_EU_19_G_ES","",""
"","","","","","","","","","VAT_S_IN_CY_19_S","VAT_S_IN_EU_19_S_ESSS","",""
"","","","","","","","","","VAT_S_IN_CY_9_G","VAT_S_IN_EU_0_G_EZ","",""
"","","","","","","","","","VAT_S_IN_CY_9_S","VAT_S_IN_EU_19_S_ESSS","",""
"","","","","","","","","","VAT_S_IN_CY_5_G","VAT_S_IN_EU_0_G_EZ","",""
"","","","","","","","","","VAT_S_IN_CY_5_S","VAT_S_IN_EU_19_S_ESSS","",""
"","","","","","","","","","VAT_S_IN_CY_3_G","VAT_S_IN_EU_0_G_EZ","",""
"","","","","","","","","","VAT_S_IN_CY_3_S","VAT_S_IN_EU_19_S_ESSS","",""
"","","","","","","","","","VAT_P_IN_CY_19_G","VAT_P_IN_EU_0_G_EZ","",""
"","","","","","","","","","VAT_P_IN_CY_19_S","VAT_P_IN_EU_19_S_ESSP","",""
"","","","","","","","","","VAT_P_IN_CY_9_G","VAT_P_IN_EU_0_G_EZ","",""
"","","","","","","","","","VAT_P_IN_CY_9_S","VAT_P_IN_EU_19_S_ESSP","",""
"","","","","","","","","","VAT_P_IN_CY_5_G","VAT_P_IN_EU_0_G_EZ","",""
"","","","","","","","","","VAT_P_IN_CY_5_S","VAT_P_IN_EU_19_S_ESSP","",""
"","","","","","","","","","VAT_P_IN_CY_3_G","VAT_P_IN_EU_0_G_EZ","",""
"","","","","","","","","","VAT_P_IN_CY_3_S","VAT_P_IN_EU_19_S_ESSP","",""
"fiscal_position_cy_abroad","5","Partner outside of EU","Συνεργάτης εκτός ΕΕ","AB dışındaki ortak","1","","","","VAT_S_IN_CY_19_G","VAT_S_OUT_EU","",""
"","","","","","","","","","VAT_S_IN_CY_19_S","VAT_S_OUT_EU","",""
"","","","","","","","","","VAT_S_IN_CY_9_G","VAT_S_OUT_EU","",""
"","","","","","","","","","VAT_S_IN_CY_9_S","VAT_S_OUT_EU","",""
"","","","","","","","","","VAT_S_IN_CY_5_G","VAT_S_OUT_EU","",""
"","","","","","","","","","VAT_S_IN_CY_5_S","VAT_S_OUT_EU","",""
"","","","","","","","","","VAT_S_IN_CY_3_G","VAT_S_OUT_EU","",""
"","","","","","","","","","VAT_S_IN_CY_3_S","VAT_S_OUT_EU","",""
"","","","","","","","","","VAT_P_IN_CY_19_G","VAT_P_OUT_EU_G","",""
"","","","","","","","","","VAT_P_IN_CY_19_S","VAT_P_OUT_EU_S","",""
"","","","","","","","","","VAT_P_IN_CY_9_G","VAT_P_OUT_EU_G","",""
"","","","","","","","","","VAT_P_IN_CY_9_S","VAT_P_OUT_EU_S","",""
"","","","","","","","","","VAT_P_IN_CY_5_G","VAT_P_OUT_EU_G","",""
"","","","","","","","","","VAT_P_IN_CY_5_S","VAT_P_OUT_EU_S","",""
"","","","","","","","","","VAT_P_IN_CY_3_G","VAT_P_OUT_EU_G","",""
"","","","","","","","","","VAT_P_IN_CY_3_S","VAT_P_OUT_EU_S","",""

```

## File: data\template\account.tax-cy.csv

```csv
"id","sequence","description","invoice_label","name","price_include","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@gr","description@tr"
"VAT_S_IN_CY_19_G","1","Standard rate of 19% for goods sold in country","19%","19%","False","19.0","percent","sale","tax_group_19","100","base","invoice","+6","","Τυπικός συντελεστής 19% για προϊόντα που πωλούνται στη χώρα","Ülkede satılan mallar için %19'luk standart oran"
"","","","","","","","","","","100","tax","invoice","+1","cy_2201","",""
"","","","","","","","","","","100","base","refund","-6","","",""
"","","","","","","","","","","100","tax","refund","-1","cy_2201","",""
"VAT_S_IN_CY_19_S","2","Standard rate of 19% for services sold in country","19% S","19% S","False","19.0","percent","sale","tax_group_19","100","base","invoice","+6","","Τυπικός συντελεστής 19% για υπηρεσίες που πωλούνται στη χώρα","Ülkede satılan hizmetler için %19'luk standart oran"
"","","","","","","","","","","100","tax","invoice","+1","cy_2201","",""
"","","","","","","","","","","100","base","refund","-6","","",""
"","","","","","","","","","","100","tax","refund","-1","cy_2201","",""
"VAT_P_IN_CY_19_G","3","Standard rate of 19% for goods purchased in country","19%","19%","False","19.0","percent","purchase","tax_group_19","100","base","invoice","+7","","Τυπικός συντελεστής 19% για αγαθά που αγοράζονται στη χώρα","Ülkede satın alınan mallar için %19'luk standart oran"
"","","","","","","","","","","100","tax","invoice","+4","cy_2202","",""
"","","","","","","","","","","100","base","refund","-7","","",""
"","","","","","","","","","","100","tax","refund","-4","cy_2202","",""
"VAT_P_IN_CY_19_S","4","Standard rate of 19% for services purchased in country","19% S","19% S","False","19.0","percent","purchase","tax_group_19","100","base","invoice","+7","","Τυπικός συντελεστής 19% για υπηρεσίες που αγοράζονται στη χώρα","Ülkede satın alınan hizmetler için %19'luk standart oran"
"","","","","","","","","","","100","tax","invoice","+4","cy_2202","",""
"","","","","","","","","","","100","base","refund","-7","","",""
"","","","","","","","","","","100","tax","refund","-4","cy_2202","",""
"VAT_S_IN_CY_9_G","5","Standard rate of 9% for goods sold in country","9%","9%","False","9.0","percent","sale","tax_group_9","100","base","invoice","+6","","Τυπικός συντελεστής 9% για προϊόντα που πωλούνται στη χώρα","Ülkede satılan mallar için %9'luk standart oran"
"","","","","","","","","","","100","tax","invoice","+1","cy_2201","",""
"","","","","","","","","","","100","base","refund","-6","","",""
"","","","","","","","","","","100","tax","refund","-1","cy_2201","",""
"VAT_S_IN_CY_9_S","6","Reduced rate of 9% for services sold in country","9% S","9% S","False","9.0","percent","sale","tax_group_9","100","base","invoice","+6","","Μειωμένο ποσοστό 9% για υπηρεσίες που πωλούνται στη χώρα","Ülkede satılan hizmetler için %9'luk indirimli oran"
"","","","","","","","","","","100","tax","invoice","+1","cy_2201","",""
"","","","","","","","","","","100","base","refund","-6","","",""
"","","","","","","","","","","100","tax","refund","-1","cy_2201","",""
"VAT_P_IN_CY_9_G","7","Reduced rate of 9% for goods purchased in country","9%","9%","False","9.0","percent","purchase","tax_group_9","100","base","invoice","+7","","Μειωμένο ποσοστό 9% για αγαθά που αγοράζονται στη χώρα","Ülkede satın alınan mallar için indirimli oran %9"
"","","","","","","","","","","100","tax","invoice","+4","cy_2202","",""
"","","","","","","","","","","100","base","refund","-7","","",""
"","","","","","","","","","","100","tax","refund","-4","cy_2202","",""
"VAT_P_IN_CY_9_S","8","Reduced rate of 9% for services purchased in country","9% S","9% S","False","9.0","percent","purchase","tax_group_9","100","base","invoice","+7","","Μειωμένο ποσοστό 9% για υπηρεσίες που αγοράζονται στη χώρα","Ülkede satın alınan hizmetler için %9 indirimli oran"
"","","","","","","","","","","100","tax","invoice","+4","cy_2202","",""
"","","","","","","","","","","100","base","refund","-7","","",""
"","","","","","","","","","","100","tax","refund","-4","cy_2202","",""
"VAT_S_IN_CY_5_G","9","Reduced rate of 5% for goods sold in country","5%","5%","False","5.0","percent","sale","tax_group_5","100","base","invoice","+6","","Μειωμένος συντελεστής 5% για προϊόντα που πωλούνται στη χώρα","Ülkede satılan mallar için %5 indirimli oran"
"","","","","","","","","","","100","tax","invoice","+1","cy_2201","",""
"","","","","","","","","","","100","base","refund","-6","","",""
"","","","","","","","","","","100","tax","refund","-1","cy_2201","",""
"VAT_S_IN_CY_5_S","10","Reduced rate of 5% for services sold in country","5% S","5% S","False","5.0","percent","sale","tax_group_5","100","base","invoice","+6","","Μειωμένο ποσοστό 5% για υπηρεσίες που πωλούνται στη χώρα","Ülkede satılan hizmetler için %5 indirimli oran"
"","","","","","","","","","","100","tax","invoice","+1","cy_2201","",""
"","","","","","","","","","","100","base","refund","-6","","",""
"","","","","","","","","","","100","tax","refund","-1","cy_2201","",""
"VAT_P_IN_CY_5_G","11","Reduced rate of 5% for goods purchased in country","5%","5%","False","5.0","percent","purchase","tax_group_5","100","base","invoice","+7","","Μειωμένο ποσοστό 5% για προϊόντα που αγοράζονται στη χώρα","Ülkede satın alınan mallar için %5 indirimli oran"
"","","","","","","","","","","100","tax","invoice","+4","cy_2202","",""
"","","","","","","","","","","100","base","refund","-7","","",""
"","","","","","","","","","","100","tax","refund","-4","cy_2202","",""
"VAT_P_IN_CY_5_S","12","Reduced rate of 5% for services purchased in country","5% S","5% S","False","5.0","percent","purchase","tax_group_5","100","base","invoice","+7","","Μειωμένο ποσοστό 5% για υπηρεσίες που αγοράζονται στη χώρα","Ülkede satın alınan hizmetler için %5 indirimli oran"
"","","","","","","","","","","100","tax","invoice","+4","cy_2202","",""
"","","","","","","","","","","100","base","refund","-7","","",""
"","","","","","","","","","","100","tax","refund","-4","cy_2202","",""
"VAT_S_IN_CY_3_G","13","Reduced rate of 3% for goods sold in country","3%","3%","False","3.0","percent","sale","tax_group_3","100","base","invoice","+6","","Μειωμένος συντελεστής 3% για προϊόντα που πωλούνται στη χώρα","Ülkede satılan mallar için indirimli oran %3"
"","","","","","","","","","","100","tax","invoice","+1","cy_2201","",""
"","","","","","","","","","","100","base","refund","-6","","",""
"","","","","","","","","","","100","tax","refund","-1","cy_2201","",""
"VAT_S_IN_CY_3_S","14","Reduced rate of 3% for services sold in country","3% S","3% S","False","3.0","percent","sale","tax_group_3","100","base","invoice","+6","","Μειωμένο ποσοστό 3% για υπηρεσίες που πωλούνται στη χώρα","Ülkede satılan hizmetler için %3'lük indirimli oran"
"","","","","","","","","","","100","tax","invoice","+1","cy_2201","",""
"","","","","","","","","","","100","base","refund","-6","","",""
"","","","","","","","","","","100","tax","refund","-1","cy_2201","",""
"VAT_P_IN_CY_3_G","15","Reduced rate of 3% for goods purchased in country","3%","3%","False","3.0","percent","purchase","tax_group_3","100","base","invoice","+7","","Μειωμένο ποσοστό 3% για προϊόντα που αγοράζονται στη χώρα","Ülkede satın alınan mallar için indirimli oran %3"
"","","","","","","","","","","100","tax","invoice","+4","cy_2202","",""
"","","","","","","","","","","100","base","refund","-7","","",""
"","","","","","","","","","","100","tax","refund","-4","cy_2202","",""
"VAT_P_IN_CY_3_S","16","Reduced rate of 3% for services purchased in country","3% S","3% S","False","3.0","percent","purchase","tax_group_3","100","base","invoice","+7","","Μειωμένο ποσοστό 3% για υπηρεσίες που αγοράζονται στη χώρα","Ülkede satın alınan hizmetler için %3 indirimli oran"
"","","","","","","","","","","100","tax","invoice","+4","cy_2202","",""
"","","","","","","","","","","100","base","refund","-7","","",""
"","","","","","","","","","","100","tax","refund","-4","cy_2202","",""
"VAT_S_IN_CY_0","17","Zero rate for goods and services sold in country","0%","0%","False","0.0","percent","sale","tax_group_0","100","base","invoice","+6","","Μηδενικός συντελεστής για αγαθά και υπηρεσίες που πωλούνται στη χώρα","Ülkede satılan mal ve hizmetler için sıfır oran"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-6","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"VAT_P_IN_CY_0","18","Zero rate for goods and services purchased in country","0%","0%","False","0.0","percent","purchase","tax_group_0","100","base","invoice","+7","","Μηδενικός συντελεστής για αγαθά και υπηρεσίες που αγοράζονται στη χώρα","Ülkede satın alınan mal ve hizmetler için sıfır oran"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-7","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"RC_S_IN_CY","19","Reverse charge rate for goods and services sold in country","19% RC","19% RC","False","19.0","percent","sale","tax_group_19","100","base","invoice","+6","","Ποσοστό αντίστροφης χρέωσης για αγαθά και υπηρεσίες που πωλούνται στη χώρα","Ülkede satılan mal ve hizmetler için ters ücret oranı"
"","","","","","","","","","","100","tax","invoice","","cy_2203","",""
"","","","","","","","","","","100","base","refund","-6","","",""
"","","","","","","","","","","100","tax","refund","","cy_2203","",""
"RC_P_IN_CY","20","Reverse charge rate for goods and services purchased in country","19% RC","19% RC","False","19.0","percent","purchase","tax_group_19","100","base","invoice","+7","","Ποσοστό αντίστροφης χρέωσης για αγαθά και υπηρεσίες που αγοράζονται στη χώρα","Ülkede satın alınan mal ve hizmetler için ters ücret oranı"
"","","","","","","","","","","100","tax","invoice","+1||+4","cy_2203","",""
"","","","","","","","","","","100","base","refund","-7","","",""
"","","","","","","","","","","100","tax","refund","-1||-4","cy_2203","",""
"EXEMPT_S","21","Exempt rate for goods and services sold in country","0% E","0% E","False","0.0","percent","sale","tax_group_0","100","base","invoice","+6","","Ποσοστό απαλλαγής για αγαθά και υπηρεσίες που πωλούνται στη χώρα","Ülkede satılan mal ve hizmetler için muafiyet oranı"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-6","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"EXEMPT_P","22","Exempt rate for goods and services purchased in country","0% E","0% E","False","0.0","percent","purchase","tax_group_0","100","base","invoice","+7","","Ποσοστό απαλλαγής για αγαθά και υπηρεσίες που αγοράζονται στη χώρα","Ülkede satın alınan mal ve hizmetler için muafiyet oranı"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-7","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"VAT_S_OUT_EU","23","VAT rate for goods and services sold outside of the EU","0% OEU","0% OEU","False","0.0","percent","sale","tax_group_0","100","base","invoice","+6||+9","","Συντελεστής ΦΠΑ για αγαθά και υπηρεσίες που πωλούνται εκτός ΕΕ","AB dışında satılan mal ve hizmetler için KDV oranı"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-6||-9","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"VAT_P_OUT_EU_G","24","VAT rate for goods purchased from outside the EU","0% OEU","0% OEU","False","0.0","percent","purchase","tax_group_0","100","base","invoice","+7","","Συντελεστής ΦΠΑ για αγαθά που αγοράζονται από χώρες εκτός ΕΕ","AB dışından satın alınan mallar için KDV oranı"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-7","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"VAT_P_OUT_EU_S","25","VAT rate for services purchased from outside the EU","19% OEU","19% OEU","False","19.0","percent","purchase","tax_group_19","100","base","invoice","+6||+7","","Συντελεστής ΦΠΑ για υπηρεσίες που αγοράζονται εκτός ΕΕ","AB dışından satın alınan hizmetler için KDV oranı"
"","","","","","","","","","","100","tax","invoice","+1||+4","cy_2204","",""
"","","","","","","","","","","100","base","refund","-6||-7","","",""
"","","","","","","","","","","100","tax","refund","-1||-4","cy_2204","",""
"VAT_S_IN_EU_19_G_ES","26","VAT rate for sales of goods in EU","19% EU","19% EU","False","19.0","percent","sale","tax_group_19","100","base","invoice","+6||+8A","","Συντελεστής ΦΠΑ για πωλήσεις αγαθών στην ΕΕ","AB'de mal satışlarında KDV oranı"
"","","","","","","","","","","100","tax","invoice","","cy_2201","",""
"","","","","","","","","","","100","base","refund","-6||-8A","","",""
"","","","","","","","","","","100","tax","refund","","cy_2201","",""
"VAT_P_IN_EU_19_G_ES","27","VAT rate for purchases of goods within EU","19% EU","19% EU","False","19.0","percent","purchase","tax_group_19","100","base","invoice","+7||+11A","","Συντελεστής ΦΠΑ για αγορές αγαθών εντός ΕΕ","AB dahilinde mal alımlarında KDV oranı"
"","","","","","","","","","","100","tax","invoice","+2||+4","cy_2202","",""
"","","","","","","","","","","100","base","refund","-7||-11A","","",""
"","","","","","","","","","","100","tax","refund","-2||-4","cy_2202","",""
"VAT_S_IN_EU_0_G_EZ","28","Zero rate for sales of goods in EU","0% EU","0% EU","False","0.0","percent","sale","tax_group_0","100","base","invoice","+6||+8A","","Μηδενικός συντελεστής για πωλήσεις αγαθών στην ΕΕ","AB'de mal satışlarında sıfır oran"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-6||-8A","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"VAT_P_IN_EU_0_G_EZ","29","Zero rate for purchases of goods in EU","0% EU","0% EU","False","0.0","percent","purchase","tax_group_0","100","base","invoice","+7","","Μηδενικός συντελεστής για αγορές αγαθών στην ΕE","AB'de mal alımlarında sıfır oran"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-7","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"VAT_S_IN_EU_19_S_ESSS","30","VAT rate for sales of services in EU","19% EU S","19% EU S","False","19.0","percent","sale","tax_group_19","100","base","invoice","+6||+8B","","Συντελεστής ΦΠΑ για πωλήσεις υπηρεσιών στην ΕE","AB'de hizmet satışlarında KDV oranı"
"","","","","","","","","","","100","tax","invoice","","cy_2201","",""
"","","","","","","","","","","100","base","refund","-6||-8B","","",""
"","","","","","","","","","","100","tax","refund","","cy_2201","",""
"VAT_P_IN_EU_19_S_ESSS","31","VAT rate for purchases of services in EU","19% EU S","19% EU S","False","19.0","percent","purchase","tax_group_19","100","base","invoice","+7||+11B","","Συντελεστής ΦΠΑ για αγορές υπηρεσιών στην ΕE","AB'de hizmet satın alımlarında KDV oranı"
"","","","","","","","","","","100","tax","invoice","+1||+4","cy_2202","",""
"","","","","","","","","","","100","base","refund","-7||-11B","","",""
"","","","","","","","","","","100","tax","refund","-1||-4","cy_2202","",""
"VAT_S_IN_EU_19_S_ESSP","32","VAT rate for sales of services in EU with reverse charge","19% EU RC","19% EU RC","False","19.0","percent","sale","tax_group_19","100","base","invoice","+8B","","Συντελεστής ΦΠΑ για πωλήσεις υπηρεσιών στην ΕΕ με αντίστροφη χρέωση","AB'de ters ödemeli hizmet satışları için KDV oranı"
"","","","","","","","","","","100","tax","invoice","","cy_2201","",""
"","","","","","","","","","","100","base","refund","-8B","","",""
"","","","","","","","","","","100","tax","refund","","cy_2201","",""
"VAT_P_IN_EU_19_S_ESSP","33","VAT rate for purchases of services in EU with reverse charge","19% EU RC","19% EU RC","False","19.0","percent","purchase","tax_group_19","100","base","invoice","+6||+7||+11B","","Συντελεστής ΦΠΑ για αγορές υπηρεσιών στην ΕΕ με αντίστροφη χρέωση","AB'de ters ödemeli hizmet satın alımlarında KDV oranı"
"","","","","","","","","","","100","tax","invoice","+1||+4","cy_2202","",""
"","","","","","","","","","","100","base","refund","-6||-7||-11B","","",""
"","","","","","","","","","","100","tax","refund","-1||-4","cy_2202","",""

```

## File: data\template\account.tax.group-cy.csv

```csv
"id","name","country_id","name@gr","name@tr","tax_receivable_account_id","tax_payable_account_id"
"tax_group_19","VAT 19%","base.cy","ΦΠΑ 19%","KDV %19","cy_1100","cy_2100"
"tax_group_9","VAT 9%","base.cy","ΦΠΑ 9%","KDV 9%","cy_1100","cy_2100"
"tax_group_5","VAT 5%","base.cy","ΦΠΑ 5%","KDV 5%","cy_1100","cy_2100"
"tax_group_3","VAT 3%","base.cy","ΦΠΑ 3%","KDV 3%","cy_1100","cy_2100"
"tax_group_0","VAT 0%","base.cy","ΦΠΑ 0%","KDV 0%","cy_1100","cy_2100"

```

## File: models\template_cy.py

```python
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('cy')
    def _get_cy_template_data(self):
        return {
            'code_digits': '4',
            'property_account_receivable_id': 'cy_1100',
            'property_account_payable_id': 'cy_2100',
            'property_account_expense_categ_id': 'cy_5100',
            'property_account_income_categ_id': 'cy_4000',
        }

    @template('cy', 'res.company')
    def _get_cy_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.cy',
                'bank_account_code_prefix': '1200',
                'cash_account_code_prefix': '1231',
                'transfer_account_code_prefix': '1261',
                'account_default_pos_receivable_account_id': 'cy_1100',
                'income_currency_exchange_account_id': 'cy_7910',
                'expense_currency_exchange_account_id': 'cy_7910',
                'account_sale_tax_id': 'VAT_S_IN_CY_19_G',
                'account_purchase_tax_id': 'VAT_P_IN_CY_19_G',
            },
        }

```

## File: models\__init__.py

```python
from . import template_cy

```

