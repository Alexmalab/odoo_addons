# Odoo Module: l10n_hu

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
    'name': 'Hungary - Accounting',
    'website': 'https://www.odoo.com/documentation/17.0/applications/finance/fiscal_localizations.html',
    'icon': '/account/static/description/l10n.png',
    'countries': ['hu'],
    'version': '3.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Accounting chart and localization for Hungary
    """,
    'depends': [
        'account',
        'base_vat',
    ],
    'data': [
        'data/account_tax_report_data.xml',
        'data/res.bank.csv',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo auto_sequence="1">
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.hu"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_alap" model="account.report.line">
                <field name="name">Tax base</field>
                <field name="aggregation_formula">BASE_PAYABLE.balance + BASE_RECOVERABLE.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_base_pay" model="account.report.line">
                        <field name="name">Payable</field>
                        <field name="code">BASE_PAYABLE</field>
                        <field name="aggregation_formula">BASE_PAY_EXPORTS.balance + BASE_PAY_INTRA.balance + BASE_PAY_EXEMPT_PROPERTY.balance + BASE_PAY_EXEMPT_TAX.balance + BASE_PAY_EXEMPT.balance + BASE_PAY_27.balance + BASE_PAY_18.balance + BASE_PAY_5.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_alap_fiz_export" model="account.report.line">
                                <field name="name">Exports</field>
                                <field name="code">BASE_PAY_EXPORTS</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_export_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_exports</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_fiz_eu" model="account.report.line">
                                <field name="name">Intra-community</field>
                                <field name="code">BASE_PAY_INTRA</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_eu_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_intra</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_fiz_targyi" model="account.report.line">
                                <field name="name">Exempt from property tax</field>
                                <field name="code">BASE_PAY_EXEMPT_PROPERTY</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_targyi_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_exempt_property</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_fiz_alanyi" model="account.report.line">
                                <field name="name">Exempt from tax</field>
                                <field name="code">BASE_PAY_EXEMPT_TAX</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_alanyi_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_exempt_tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_fiz_koron_kivuli" model="account.report.line">
                                <field name="name">Exempt</field>
                                <field name="code">BASE_PAY_EXEMPT</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_koron_kivuli_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_exempt</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_fiz_afa_27" model="account.report.line">
                                <field name="name">27% VAT</field>
                                <field name="code">BASE_PAY_27</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_afa_27_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_27</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_fiz_afa_18" model="account.report.line">
                                <field name="name">18% VAT</field>
                                <field name="code">BASE_PAY_18</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_afa_18_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_18</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_fiz_afa_5" model="account.report.line">
                                <field name="name">5% VAT</field>
                                <field name="code">BASE_PAY_5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_fiz_afa_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_pay_5</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_base_rec" model="account.report.line">
                        <field name="name">Recoverable</field>
                        <field name="code">BASE_RECOVERABLE</field>
                        <field name="aggregation_formula">BASE_REC_IMPORT.balance + BASE_REC_INTRA.balance + BASE_REC_REVERSE.balance + BASE_REC_EXEMPT_MATERIAL.balance + BASE_REC_EXEMPT_TAX.balance + BASE_REC_EXEMPT.balance + BASE_REC_27.balance + BASE_REC_18.balance + BASE_REC_5.balance + BASE_REC_COMPENSATION.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_alap_import" model="account.report.line">
                                <field name="name">Import</field>
                                <field name="code">BASE_REC_IMPORT</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_import_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_import</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_viss" model="account.report.line">
                                <field name="name">Intra-community</field>
                                <field name="code">BASE_REC_INTRA</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_viss_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_intra</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_forditott" model="account.report.line">
                                <field name="name">Reverse</field>
                                <field name="code">BASE_REC_REVERSE</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_forditott_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_reverse</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_viss_targyi" model="account.report.line">
                                <field name="name">Exempt from material tax</field>
                                <field name="code">BASE_REC_EXEMPT_MATERIAL</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_viss_targyi_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_exempt_material</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_viss_alanyi" model="account.report.line">
                                <field name="name">Exempt from tax</field>
                                <field name="code">BASE_REC_EXEMPT_TAX</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_viss_alanyi_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_exempt_tax</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_viss_koron_kivuli" model="account.report.line">
                                <field name="name">Exempt</field>
                                <field name="code">BASE_REC_EXEMPT</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_viss_koron_kivuli_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_exempt</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_viss_27" model="account.report.line">
                                <field name="name">27% VAT</field>
                                <field name="code">BASE_REC_27</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_viss_27_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_27</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_viss_18" model="account.report.line">
                                <field name="name">18% VAT</field>
                                <field name="code">BASE_REC_18</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_viss_18_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_18</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_viss_5" model="account.report.line">
                                <field name="name">5% VAT</field>
                                <field name="code">BASE_REC_5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_viss_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_alap_komp" model="account.report.line">
                                <field name="name">Compensation Surcharge</field>
                                <field name="code">BASE_REC_COMPENSATION</field>
                                <field name="expression_ids">
                                    <record id="tax_report_alap_komp_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">base_rec_compensation</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_fizetndo" model="account.report.line">
                <field name="name">VAT payable/recoverable</field>
                <field name="aggregation_formula">VAT_PAYABLE.balance + VAT_RECOVERABLE.balance</field>
                <field name="hierarchy_level">0</field>
                <field name="children_ids">
                    <record id="tax_report_vat_pay" model="account.report.line">
                        <field name="name">Payable</field>
                        <field name="code">VAT_PAYABLE</field>
                        <field name="aggregation_formula">VAT_PAY_27.balance + VAT_PAY_18.balance + VAT_PAY_5.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_fizetndo_27" model="account.report.line">
                                <field name="name">27% VAT</field>
                                <field name="code">VAT_PAY_27</field>
                                <field name="expression_ids">
                                    <record id="tax_report_fizetndo_27_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vat_pay_27</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_fizetndo_18" model="account.report.line">
                                <field name="name">18% VAT</field>
                                <field name="code">VAT_PAY_18</field>
                                <field name="expression_ids">
                                    <record id="tax_report_fizetndo_18_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vat_pay_18</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_fizetndo_5" model="account.report.line">
                                <field name="name">5% VAT</field>
                                <field name="code">VAT_PAY_5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_fizetndo_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vat_pay_5</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_vat_rec" model="account.report.line">
                        <field name="name">Recoverable</field>
                        <field name="code">VAT_RECOVERABLE</field>
                        <field name="aggregation_formula">VAT_REC_27.balance + VAT_REC_18.balance + VAT_REC_5.balance + VAT_REC_COMPENSATION.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_fizetndo_viss_27" model="account.report.line">
                                <field name="name">27% VAT</field>
                                <field name="code">VAT_REC_27</field>
                                <field name="expression_ids">
                                    <record id="tax_report_fizetndo_viss_27_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vat_rec_27</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_fizetndo_viss_18" model="account.report.line">
                                <field name="name">18% VAT</field>
                                <field name="code">VAT_REC_18</field>
                                <field name="expression_ids">
                                    <record id="tax_report_fizetndo_viss_18_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vat_rec_18</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_fizetndo_viss_5" model="account.report.line">
                                <field name="name">5% VAT</field>
                                <field name="code">VAT_REC_5</field>
                                <field name="expression_ids">
                                    <record id="tax_report_fizetndo_viss_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vat_rec_5</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_fizetndo_viss_komp" model="account.report.line">
                                <field name="name">Compensation Surcharge</field>
                                <field name="code">VAT_REC_COMPENSATION</field>
                                <field name="expression_ids">
                                    <record id="tax_report_fizetndo_viss_komp_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">vat_rec_compensation</field>
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

## File: data\res.bank.csv

```csv
"id","name","bic","country/id","zip","city","street","email","phone","active"
"BKCHHUHBXXX","Bank of China (Hungária) Hitelintézet Rt.","BKCHHUHBXXX","base.hu",1051,"Budapest","József Nádor tér 7.","service_hu@bank-of-china.com","+3614299200","True"
"BNPAHUHX","BNP Paribas Hungária Bank Rt.","BNPAHUHX","base.hu",1051,"Budapest","Teréz krt. 55-57.","csd_hungary@bnpparibas.com","+3613746300","True"
"BUDAHUHB","Budapest Hitel- és Fejlesztési Bank Rt.","BUDAHUHB","base.hu",1138,"Budapest","Váci út 188."," info@budapestbank.hu","+3614506060","False"
"CIBHHUHB","CIB Közép-Európai Nemzetközi Bank Zrt.","CIBHHUHB","base.hu",1027,"Budapest","Medve utca 4-14","cib@cib.hu","+3614242242","True"
"CITIHUHX","Citibank Rt.","CITIHUHX","base.hu",1051,"Budapest","Szabadság tér 7.",,"+3613745000","True"
"COBAHUHX","Commerzbank Zártkörűen Működő Rt.","COBAHUHX","base.hu",1054,"Budapest","Széchenyi rakpart 8.","info.budapest@commerzbank.com","+3613748100","False"
"DEUTHU2B","Deutsche Bank Zártkörűen Működő Rt.","DEUTHU2B","base.hu",1054,"Budapest","Hold utca 27.","db.hungary@db.com","+3613013700","True"
"FHKBHUHB","FHB Kereskedelmi Bank Zrt.","FHKBHUHB","base.hu",1082,"Budapest","Üllői út 48."," info@fhb.hu","+3614529100","False"
"GIBAHUHB","ERSTE Bank Hungary Zrt.","GIBAHUHB","base.hu",1138,"Budapest","Népfürdő u. 24-26.","erste@erstebank.hu","+3612980222","True"
"GNBAHUHB","Gránit Bank Zrt.","GNBAHUHB","base.hu",1095,"Budapest","Lechner Ödön fasor 8.","info@granitbank.hu","+3615100527","True"
"INCNHUHB","IC Bank Rt.","INCNHUHB","base.hu",1088,"Budapest","Rákóczi út 1-3."," level@bancopopolare.hu ","+3640200515","False"
"INGBHUHB","ING Bank N.V. Magyarországi Fióktelepe","INGBHUHB","base.hu",1068,"Budapest","Dózsa György út 84/b.","communications.hu@ingbank.com","+3612358700","True"
"OKHBHUHB","K&H Bank Zrt.","OKHBHUHB","base.hu",1095,"Budapest","Lechner Ödön fasor 9.","bank@kh.hu","+36203353355","True"
"HBWEHUHB","MagNet Magyar Közösségi Bank Zrt.","HBWEHUHB","base.hu",1062,"Budapest","Andrássy út 98.","info@magnetbank.hu","+3614288888","True"
"MANEHUHB","Magyar Nemzeti Bank","MANEHUHB","base.hu",1054,"Budapest","Szabadság tér 8-9.","info@mnb.hu","+3614282600","True"
"MKKBHUHBXXX","MHB Bank Nyrt.","MKKBHUHBXXX","base.hu",1056,"Budapest","Váci utca 38.","+3612687173","ugyfelszolgalat@mbhbank.hu","True"
"OTPVHUHB","OTP Bank Nyrt.","OTPVHUHB","base.hu",1051,"Budapest","Nádor utca 16.","otpbank@otpbank.hu","+3614735000","True"
"UBRTHUHB","RAIFFEISEN Bank Zrt.","UBRTHUHB","base.hu",1054,"Budapest","Akadémia utca 6.","info@raiffeisen.hu","+3680488588","True"
"MAVOHUHB","Sberbank Magyarország Zrt.","MAVOHUHB","base.hu",1088,"Budapest","Rákóczi út 7.","info@sberbank.hu","+3614114200","False"
"TAKBHUHB","TakarékBank Zrt.","TAKBHUHB","base.hu",1122,"Budapest","Pethényi  köz 10.","info@tbank.hu","False"
"BACXHUHB","UniCredit Bank Hungary Zrt.","BACXHUHB","base.hu",1054,"Budapest","Szabadság tér 5-6.","info@unicreditgroup.hu","+3613253200","True"

```

## File: data\template\account.account-hu.csv

```csv
"id","code","name","account_type","reconcile","name@hu"
"l10n_hu_111","111","Activation value of founding reorganization","asset_non_current","False","Az alapító átszervezés aktiválási értéke"
"l10n_hu_112","112","Activated value for experimental development","asset_non_current","False","Aktivált érték kísérleti fejlesztéshez"
"l10n_hu_113","113","Property rights","asset_non_current","False","Tulajdonjogok"
"l10n_hu_114","114","Intellectual products","asset_non_current","False","Szellemi termékek"
"l10n_hu_115","115","Business or goodwill","asset_non_current","False","Üzleti vagy jóindulatú"
"l10n_hu_117","117","Value adjustments in respect of intangible assets","asset_non_current","False","Értékmódosítások az immateriális javak tekintetében"
"l10n_hu_118","118","Unscheduled amortization and reversal of intangible assets","asset_non_current","False","Nem tervezett amortizáció és immateriális javak visszaírása"
"l10n_hu_119","119","Planned amortization of intangible assets","asset_non_current","False","Az immateriális javak tervezett amortizációja"
"l10n_hu_121","121","Land","asset_fixed","False","Föld"
"l10n_hu_122","122","Land, realization","asset_fixed","False","Föld, megvalósítás"
"l10n_hu_123","123","Buildings, ownership shares","asset_fixed","False","Épületek, tulajdonrészek"
"l10n_hu_124","124","Other structures","asset_fixed","False","Egyéb szerkezetek"
"l10n_hu_125","125","Real estate and buildings outside the plant","asset_fixed","False","Ingatlanok és épületek az üzemen kívül"
"l10n_hu_126","126","Link to real estate for property rights","asset_fixed","False","Link az ingatlanhoz a tulajdonjogokhoz"
"l10n_hu_127","127","Real estate value adjustment","asset_fixed","False","Ingatlan értékhelyesbítés"
"l10n_hu_128","128","Unplanned depreciation of real estate and its reversal","asset_non_current","False","Az ingatlanok nem tervezett értékcsökkenése és annak visszaírása"
"l10n_hu_129","129","Planned depreciation of real estate","asset_non_current","False","Az ingatlanok tervezett értékcsökkenése"
"l10n_hu_131","131","Production machines, equipment, tools, production tools","asset_fixed","False","Gyártógépek, berendezések, szerszámok, gyártóeszközök"
"l10n_hu_132","132","Vehicles directly involved in production","asset_fixed","False","A gyártásban közvetlenül részt vevő járművek"
"l10n_hu_137","137","Value adjustment of technical equipment, machines, vehicles","asset_fixed","False","Műszaki berendezések, gépek, járművek értékhelyesbítése"
"l10n_hu_138","138","Unscheduled depreciation and reversal of technical equipment, machinery and vehicles","asset_non_current","False","Műszaki berendezések, gépek és járművek előre nem tervezett értékcsökkenése, visszaírás"
"l10n_hu_139","139","Depreciation of technical equipment, machinery and vehicles according to plan","asset_non_current","False","Műszaki berendezések, gépek, járművek terv szerinti értékcsökkenése"
"l10n_hu_141","141","Operating (business) machines, equipment, facilities","asset_fixed","False","Üzemeltető (üzleti) gépek, berendezések, létesítmények"
"l10n_hu_142","142","Other vehicles","asset_fixed","False","Egyéb járművek"
"l10n_hu_143","143","Office, administrative and plant equipment","asset_fixed","False","Irodai, adminisztratív és üzemi berendezések"
"l10n_hu_144","144","Off-site equipment, facilities, vehicles","asset_fixed","False","Telephelyen kívüli berendezések, létesítmények, járművek"
"l10n_hu_145","145","Welfare equipment, fittings and works of art","asset_fixed","False","Jóléti berendezések, felszerelések és műalkotások"
"l10n_hu_147","147","Value adjustments of other plant, equipment and vehicles","asset_fixed","False","Egyéb üzemek, berendezések és járművek értékhelyesbítései"
"l10n_hu_148","148","Unscheduled depreciation and reversal of other plant, equipment and vehicles","asset_non_current","False","Egyéb berendezések, berendezések és járművek előre nem tervezett értékcsökkenése és visszaírás"
"l10n_hu_149","149","Depreciation of other plant, equipment and vehicles as planned","asset_non_current","False","Egyéb üzemek, berendezések és járművek terv szerinti értékcsökkenése"
"l10n_hu_151","151","Breeding animals","asset_non_current","False","Tenyészállatok"
"l10n_hu_152","152","Work animals","asset_non_current","False","Munkaállatok"
"l10n_hu_153","153","Other animals","asset_non_current","False","Más állatok"
"l10n_hu_157","157","Value adjustment of breeding animals","asset_non_current","False","Tenyészállatok értékhelyesbítése"
"l10n_hu_158","158","Unscheduled depreciation and reversal of breeding animals","asset_non_current","False","A tenyészállatok nem tervezett értékcsökkenése és visszaírása"
"l10n_hu_159","159","Depreciation of breeding animals as planned","asset_non_current","False","Tenyészállatok terv szerinti értékcsökkenése"
"l10n_hu_161","161","Work in progress","asset_non_current","False","Munka folyamatban"
"l10n_hu_162","162","Renovations","asset_non_current","False","Felújítások"
"l10n_hu_168","168","Investments are unplanned depreciated","asset_non_current","False","A beruházások nem tervezett értékcsökkenést mutatnak"
"l10n_hu_171","171","Long-term interest in an associate","asset_non_current","False","Hosszú távú érdeklődés egy munkatárs iránt"
"l10n_hu_172","172","Permanent significant ownership interest","asset_non_current","False","Tartós jelentős tulajdoni részesedé"
"l10n_hu_173","173","Other permanent holdings","asset_non_current","False","Egyéb állandó üzemek"
"l10n_hu_177","177","Value adjustments in respect of shares","asset_non_current","False","A részvények értékhelyesbítése"
"l10n_hu_178","178","Valuation difference on non-current holdings","asset_non_current","False","Befektetett állományok értékelési különbözete"
"l10n_hu_179","179","Impairment and reversal of shares","asset_non_current","False","Részvények értékvesztése és visszaírás"
"l10n_hu_181","181","Government bonds","asset_non_current","False","Államkötvények"
"l10n_hu_182","182","Securities of affiliated enterprises","asset_non_current","False","Kapcsolt vállalkozások értékpapírjai"
"l10n_hu_183","183","Securities of enterprises with a significant shareholding","asset_non_current","False","Jelentős részesedéssel rendelkező vállalkozások értékpapírjai"
"l10n_hu_184","184","Securities of other enterprises","asset_non_current","False","Egyéb vállalkozások értékpapírjai"
"l10n_hu_185","185","Long-term discount securities","asset_non_current","False","Hosszú lejáratú diszkont értékpapírok"
"l10n_hu_188","188","Valuation difference on debt securities","asset_non_current","False","Hitelviszonyt megtestesítő értékpapírok értékelési különbözete"
"l10n_hu_189","189","Impairment and reversal of securities","asset_non_current","False","Értékpapírok értékvesztése és visszaírása"
"l10n_hu_191","191","Resistant loans to affiliates","asset_non_current","False","Ellenálló kölcsönök leányvállalatoknak"
"l10n_hu_192","192","Long-term loans granted to other participating enterprises","asset_non_current","False","Más részt vevő vállalkozásoknak nyújtott hosszú lejáratú hitelek"
"l10n_hu_193","193","Other long-term loans","asset_non_current","False","Egyéb hosszú lejáratú hitelek"
"l10n_hu_194","194","Long-term bank deposits in affiliated enterprises","asset_non_current","False","Hosszú lejáratú bankbetétek kapcsolt vállalkozásokban"
"l10n_hu_195","195","Long-term bank deposits in other participating enterprises","asset_non_current","False","Hosszú lejáratú bankbetétek más részt vevő vállalkozásoknál"
"l10n_hu_196","196","Other long-term bank deposits","asset_non_current","False","Egyéb hosszú lejáratú bankbetétek"
"l10n_hu_197","197","Long-term receivables from finance leases","asset_non_current","False","Hosszú lejáratú követelések pénzügyi lízingből"
"l10n_hu_199","199","Impairment and reversal of long-term loans (and bank deposits)","asset_non_current","False","Hosszú lejáratú hitelek (és bankbetétek) értékvesztése és visszaírása"
"l10n_hu_211","211","Raw materials","asset_current","False","Nyersanyagok"
"l10n_hu_221","221","Excipients","asset_current","False","Segédanyagok"
"l10n_hu_222","222","Fuels and fuels","asset_current","False","Üzemanyagok és üzemanyagok"
"l10n_hu_223","223","Maintenance materials","asset_current","False","Karbantartási anyagok"
"l10n_hu_224","224","Building materials","asset_current","False","Építőanyagok"
"l10n_hu_225","225","Depreciable assets within one year","asset_current","False","Egy éven belül amortizálható eszközök"
"l10n_hu_226","226","Materials reclassified from property, plant and equipment","asset_current","False","Az ingatlanok, gépek és berendezések közül átsorolt anyagok"
"l10n_hu_227","227","Other materials","asset_current","False","Más anyagok"
"l10n_hu_228","228","Price difference of materials","asset_current","False","Anyagok árkülönbsége"
"l10n_hu_229","229","Impairment and reversal of materials","asset_non_current","False","Anyagok értékvesztése és visszaírása"
"l10n_hu_231","231","Work in progress","asset_current","False","Munka folyamatban"
"l10n_hu_235","235","Half-done products","asset_current","False","Félkész termékek"
"l10n_hu_238","238","Difference in inventories of semi-finished products","asset_current","False","Különbség a félkész termékek készleteiben"
"l10n_hu_239","239","Work in progress and impairment of semi-finished products","asset_non_current","False","Folyamatban lévő termelés és félkész termékek értékvesztése"
"l10n_hu_241","241","Adult animals","asset_current","False","Felnőtt állatok"
"l10n_hu_242","242","Fattening animals","asset_current","False","Hízó állatok"
"l10n_hu_243","243","Other animals","asset_current","False","Más állatok"
"l10n_hu_246","246","Hired animals","asset_current","False","Bérelt állatok"
"l10n_hu_248","248","Animal inventory value difference","asset_current","False","Állatkészlet értékkülönbség"
"l10n_hu_249","249","Impairment and reversal of animals","asset_non_current","False","Az állatok károsodása és visszafordítása"
"l10n_hu_251","251","Finished products","asset_current","False","Elkészült termékek"
"l10n_hu_258","258","Difference in inventories of finished goods","asset_current","False","Különbség a késztermékek készleteiben"
"l10n_hu_259","259","Impairment and reversal of finished products","asset_non_current","False","Késztermékek értékvesztése és visszaírása"
"l10n_hu_261","261","Goods at purchase price","asset_current","False","Áruk vételáron"
"l10n_hu_262","262","Goods at settlement prices","asset_current","False","Áruk elszámoló áron"
"l10n_hu_263","263","Price difference of goods","asset_current","False","Az áruk árkülönbsége"
"l10n_hu_264","264","Goods at selling price","asset_current","False","Áruk eladási áron"
"l10n_hu_265","265","Goods margin","asset_current","False","Áruk árrés"
"l10n_hu_266","266","Goods stored abroad and handed over to a commission","asset_current","False","Külföldön tárolt és bizományosnak átadott áru"
"l10n_hu_267","267","Goods reclassified from property, plant and equipment","asset_current","False","Az ingatlanok, gépek és berendezések közül átsorolt áruk"
"l10n_hu_269","269","Impairment and reversal of commercial goods","asset_non_current","False","Kereskedelmi áruk értékvesztése és visszaírása"
"l10n_hu_271","271","Intermediate services","asset_current","False","Köztes szolgáltatások"
"l10n_hu_279","279","Impairment and reversal of intermediary services","asset_non_current","False","A közvetítő szolgáltatások értékvesztése és visszaírása"
"l10n_hu_281","281","Own containers","asset_current","False","Saját konténere"
"l10n_hu_282","282","Foreign packaging","asset_current","False","Külföldi csomagolás"
"l10n_hu_288","288","Price difference for deposit packages","asset_current","False","Betéti csomagok árkülönbözete"
"l10n_hu_289","289","Impairment and reversal of deposit-bearing packages","asset_non_current","False","Betétdíjas csomagok értékvesztése, visszaírása"
"l10n_hu_311","311","Domestic receivables (in huf)","asset_receivable","True","Belföldi követelések (forintban)"
"l10n_hu_312","312","Domestic receivables (in foreign currency)","asset_receivable","True","Belföldi követelések (devizában)"
"l10n_hu_315","315","Impairment and reversal of domestic receivables","asset_non_current","False","Belföldi követelések értékvesztése, visszaírása"
"l10n_hu_316","316","Foreign receivables (in huf)","asset_receivable","True","Külföldi követelések (forintban)"
"l10n_hu_317","317","Foreign claims (in foreign currency)","asset_receivable","True","Külföldi követelések (devizában)"
"l10n_hu_318","318","Valuation difference of receivables","asset_cash","False","A követelések értékelési különbözete"
"l10n_hu_319","319","Impairment and reversal of foreign receivables","asset_non_current","False","Külföldi követelések értékvesztése és visszaírása"
"l10n_hu_321","321","Receivables from associates","asset_current","False","Követelések munkatársakkal szemben"
"l10n_hu_322","322","Receivables from enterprises with significant ownership interest","asset_current","False","Követelések jelentős tulajdoni hányaddal rendelkező vállalkozásokkal szemben"
"l10n_hu_323","323","Subscribed but not yet paid-up capital relationship from business","asset_current","False","Vállalkozásból jegyzett, de még be nem fizetett tőkeviszony"
"l10n_hu_324","324","Subscribed but not yet paid-up capital from a company with a significant shareholding","asset_current","False","Jegyzett, de még be nem fizetett tőke jelentős részesedéssel rendelkező társaságtól"
"l10n_hu_325","325","Impairment and reversal of receivables from associates","asset_non_current","False","Társult vállalkozásokkal szembeni követelések értékvesztése és visszaírása"
"l10n_hu_326","326","Impairment and reversal of receivables from a significant ownership interest","asset_non_current","False","Jelentős tulajdonosi részesedésből származó követelések értékvesztése és visszaírása"
"l10n_hu_331","331","Receivables from other participating interests","asset_current","False","Követelések egyéb érdekeltséggel szemben"
"l10n_hu_332","332","Subscribed but not yet paid-up capital from other participating undertakings","asset_current","False","Más részt vevő vállalkozásoktól jegyzett, de még be nem fizetett tőke"
"l10n_hu_333","333","Subscribed but not yet paid-up capital from a non-participating company","asset_current","False","Jegyzett, de még be nem fizetett tőke egy nem részt vevő társaságtól"
"l10n_hu_339","339","Impairment and reversal of receivables from other participating interests","asset_non_current","False","Egyéb részesedéssel szembeni követelések értékvesztése és visszaírása"
"l10n_hu_341","341","Domestic bills","asset_current","False","Belföldi számlák"
"l10n_hu_345","345","Impairment and reversal of domestic accounts receivable","asset_non_current","False","Belföldi követelések értékvesztése és visszaírása"
"l10n_hu_346","346","Foreign exchange receivables","asset_current","False","Devizakövetelések"
"l10n_hu_349","349","Impairment and reversal of foreign exchange receivables","asset_non_current","False","Devizakövetelések értékvesztése és visszaírása"
"l10n_hu_351","351","Advances on intangible assets","asset_current","False","Előlegek immateriális javakra"
"l10n_hu_352","352","Advances on investments","asset_current","False","Beruházási előlegek"
"l10n_hu_353","353","Advances on stocks","asset_current","False","Előlegek a részvényeken"
"l10n_hu_354","354","Advances on services","asset_current","False","Előrelépések a szolgáltatások terén"
"l10n_hu_355","355","Other advances given","asset_current","False","Egyéb előlegek adottak"
"l10n_hu_359","359","Impairment and reversal of specific advances","asset_non_current","False","Egyedi előlegek értékvesztése és visszaírása"
"l10n_hu_361","361","Claims on employees","asset_current","False","Az alkalmazottakkal szembeni követelések"
"l10n_hu_3611","3611","Advances paid to employees","asset_current","False","Az alkalmazottaknak fizetett előlegek"
"l10n_hu_3612","3612","Prescribed debts","asset_current","False","Előírt tartozások"
"l10n_hu_3613","3613","Other accounts with employees","asset_current","False","Egyéb számlák alkalmazottakkal"
"l10n_hu_362","362","Budget allocation requirements","asset_current","False","Költségvetési elosztási követelmények"
"l10n_hu_363","363","Fulfillment of budget allocation needs","asset_current","False","Költségvetési előirányzati igények teljesítése"
"l10n_hu_364","364","Short-term borrowed funds","asset_current","False","Rövid lejáratú kölcsönzött pénzeszközök"
"l10n_hu_365","365","Receivables purchased and received","asset_current","False","Vásárolt és átvett követelések"
"l10n_hu_366","366","Receivables related to shares and securities","asset_current","False","Részvényekkel és értékpapírokkal kapcsolatos követelések"
"l10n_hu_367","367","Receivables related to futures, options and swaps","asset_current","False","Határidős, opciós és swap ügyletekkel kapcsolatos követelések"
"l10n_hu_368","368","Miscellaneous other receivables","asset_current","False","Egyéb egyéb követelések"
"l10n_hu_3681","3681","Commission settlements","asset_current","False","Bizományos elszámolások"
"l10n_hu_3683","3683","Vat on export purchases","asset_current","False","Export vásárlások áfája"
"l10n_hu_3684","3684","Debtors","asset_current","False","Adósok"
"l10n_hu_3685","3685","Claims on insurance undertakings","asset_current","False","Biztosítókkal szembeni követelések"
"l10n_hu_3686","3686","Barter transaction settlement account","asset_current","False","Barter tranzakciós elszámolási számla"
"l10n_hu_3687","3687","Exchange differences settlement account","asset_current","False","Árfolyam-különbözet elszámolási számla"
"l10n_hu_3688","3688","Intangible assets and tangible assets settlement account","asset_current","False","Immateriális javak és tárgyi eszközök elszámolási száml"
"l10n_hu_3689","3689","Other claims not raised","asset_current","False","Egyéb követelések nem merültek fel"
"l10n_hu_369","369","Impairment and reversal of other receivables","asset_non_current","False","Egyéb követelések értékvesztése és visszaírása"
"l10n_hu_371","371","Participation in an associate","asset_current","False","Részvétel egy munkatársban"
"l10n_hu_3711","3711","Interests in associates purchased for sale","asset_current","False","Érdeklődés eladásra vásárolt munkatársak iránt"
"l10n_hu_3719","3719","Impairment and reversal of interests in associates","asset_non_current","False","A társult vállalkozásokban lévő érdekeltségek értékvesztése és visszavonás"
"l10n_hu_372","372","Significant ownership","asset_current","False","Jelentős tulajdonjog"
"l10n_hu_3721","3721","Significant ownership interests purchased for sale","asset_current","False","Eladásra vásárolt jelentős tulajdoni részesedés"
"l10n_hu_3729","3729","Impairment and reversal of significant ownership interests","asset_non_current","False","Jelentős tulajdonosi érdekeltségek értékvesztése és visszaírása"
"l10n_hu_373","373","Other shares","asset_current","False","Egyéb részvények"
"l10n_hu_3731","3731","Other shares purchased for sale","asset_current","False","Egyéb eladásra vásárolt részvények"
"l10n_hu_3739","3739","Impairment and reversal of other interests","asset_non_current","False","Egyéb érdekeltségek értékvesztése és visszavonása"
"l10n_hu_374","374","Own shares, own shares","asset_current","False","Saját részvények, saját részvények"
"l10n_hu_3741","3741","Repurchased own shares, business shares","asset_current","False","Visszavásárolt saját részvények, üzletrészek"
"l10n_hu_3749","3749","Impairment of own shares, own shares","asset_non_current","False","Saját részvények, saját részvények értékvesztése"
"l10n_hu_375","375","Debt securities held for trading","asset_current","False","Kereskedési céllal tartott hitelviszonyt megtestesítő értékpapírok"
"l10n_hu_3751","3751","Debt securities purchased for sale","asset_current","False","Eladásra vásárolt hitelviszonyt megtestesítő értékpapírok"
"l10n_hu_3759","3759","Impairment and reversal of negotiable debt securities","asset_non_current","False","Forgatóképes hitelviszonyt megtestesítő értékpapírok értékvesztése és visszaírása"
"l10n_hu_378","378","Valuation difference on securities","asset_current","False","Értékpapírok értékelési különbözete"
"l10n_hu_379","379","Securities settlement account","asset_current","False","Értékpapír elszámolási számla"
"l10n_hu_381","381","Checkout","asset_current","False","Pénztár"
"l10n_hu_382","382","Currency office","asset_current","False","Valutahivatal"
"l10n_hu_383","383","Checks","asset_current","False","Ellenőrzések"
"l10n_hu_384","384","Settlement deposit account","asset_current","False","Elszámolási betétszámla"
"l10n_hu_385","385","Separate deposit accounts","asset_current","False","Külön betétszámlák"
"l10n_hu_386","386","Foreign currency deposit account","asset_current","False","Devizabetétszámla"
"l10n_hu_389","389","Internal relocation","asset_current","False","Belső költöztetés"
"l10n_hu_391","391","Accruals and deferrals","asset_current","False","Passzív időbeli elhatárolások"
"l10n_hu_392","392","Accruals and deferrals of costs and expenses","asset_current","False","Költségek és ráfordítások passzív időbeli elhatárolásai"
"l10n_hu_393","393","Deferred expenses","asset_current","False","Levont szakszervezeti díj"
"l10n_hu_411","411","Registered capital","equity","False","Jegyzett tőke"
"l10n_hu_412","412","Capital reserve","equity","False","Tőketartalék"
"l10n_hu_413","413","Profit reserve","equity","False","Profittartalék"
"l10n_hu_414","414","Reserved reserve","equity","False","Fenntartott tartalék"
"l10n_hu_417","417","Valuation reserve","equity","False","Értékelési tartalék"
"l10n_hu_4171","4171","Valuation reserve for value adjustments","equity","False","Értékelési tartalék értékhelyesbítéshez"
"l10n_hu_4172","4172","Fair value measurement reserve","equity","False","Valós érték mérési tartalék"
"l10n_hu_421","421","Provision for contingent liabilities","liability_non_current","False","Céltartalék függő kötelezettségekre"
"l10n_hu_422","422","Provision for future expenses","liability_non_current","False","Céltartalék a jövőbeni kiadásokra"
"l10n_hu_429","429","Other provisions","liability_non_current","False","Egyéb rendelkezések"
"l10n_hu_431","431","Subordinated liabilities to an associate","liability_non_current","False","Alárendelt kötelezettségek társult vállalkozással szemben"
"l10n_hu_432","432","Subordinated liabilities to an enterprise that has a significant ownership interest","liability_non_current","False","Hátrasorolt kötelezettségek jelentős tulajdonosi részesedéssel rendelkező vállalkozással szemben"
"l10n_hu_433","433","Subordinated liabilities to other participating interests","liability_non_current","False","Alárendelt kötelezettségek egyéb részesedéssel szemben"
"l10n_hu_434","434","Subordinated liabilities to other enterprise","liability_non_current","False","Hátrasorolt kötelezettségek más vállalkozással szemben"
"l10n_hu_441","441","Long-term loans","liability_non_current","False","Hosszú lejáratú hitelek"
"l10n_hu_442","442","Convertible bonds","liability_non_current","False","Átváltható kötvények"
"l10n_hu_443","443","Debt from bond issue","liability_non_current","False","Kötvénykibocsátásból származó adósság"
"l10n_hu_444","444","Investment and development loans","liability_non_current","False","Beruházási és fejlesztési hitelek"
"l10n_hu_445","445","Other long-term loans","liability_non_current","False","Egyéb hosszú lejáratú hitelek"
"l10n_hu_446","446","Long-term liabilities to an associate","liability_non_current","False","Hosszú lejáratú kötelezettségek társult vállalkozással szemben"
"l10n_hu_447","447","Long-term liabilities to a significant shareholder","liability_non_current","False","Hosszú lejáratú kötelezettségek jelentős részvényessel szemben"
"l10n_hu_448","448","Non-current liabilities to participating interests","liability_non_current","False","Hosszú lejáratú kötelezettségek részesedéssel szemben"
"l10n_hu_449","449","Other long-term liabilities","liability_non_current","False","Egyéb hosszú lejáratú kötelezettségek"
"l10n_hu_4491","4491","Leases due to finance leases","liability_non_current","False","Lízingek pénzügyi lízing miatt"
"l10n_hu_451","451","Short-term loans","liability_current","False","Rövid lejáratú hitelek"
"l10n_hu_4511","4511","Short-term loans from convertible and convertible bonds","liability_current","False","Rövid lejáratú hitelek átváltoztatható és átváltoztatható kötvényekből"
"l10n_hu_452","452","Short-term loans","liability_current","False","Rövid lejáratú hitelek"
"l10n_hu_453","453","Advances received from customers","liability_current","False","Az ügyfelektől kapott előlegek"
"l10n_hu_454","454","Suppliers","liability_payable","True","Szállítók"
"l10n_hu_455","455","Investment suppliers","liability_current","False","Befektetési beszállítók"
"l10n_hu_456","456","Accounts payable","liability_current","False","Kötelezett számlák"
"l10n_hu_457","457","Current liabilities to associates","liability_current","False","Rövid lejáratú kötelezettségek társult vállalkozásokkal szemben"
"l10n_hu_458","458","Current liabilities to companies with significant ownership interests","liability_current","False","Rövid lejáratú kötelezettségek jelentős tulajdonosi érdekeltséggel rendelkező társaságokkal szemben"
"l10n_hu_459","459","Current liabilities to other participating interests","liability_current","False","Rövid lejáratú kötelezettségek egyéb részesedéssel szemben"
"l10n_hu_461","461","Corporate tax","liability_current","False","Vállalati adó"
"l10n_hu_462","462","Personal income tax accounting","liability_current","False","Személyi jövedelemadó elszámolás"
"l10n_hu_463","463","Budget payment obligations","liability_current","False","Költségvetési fizetési kötelezettségek"
"l10n_hu_464","464","Fulfillment of budget payment obligations","liability_current","False","Költségvetési fizetési kötelezettségek teljesítése"
"l10n_hu_465","465","Customs and import vat debts","liability_current","False","Vám- és behozatali áfatartozások"
"l10n_hu_466","466","Vat charged in advance","liability_current","False","ÁFA előre felszámított"
"l10n_hu_467","467","Value added tax payable","liability_current","False","Fizetendő hozzáadottérték-adó"
"l10n_hu_468","468","Financial accounting for value added tax","liability_current","False","Az általános forgalmi adó pénzügyi elszámolása"
"l10n_hu_469","469","Local taxes settlement account","liability_current","False","Helyi adók elszámolási száml"
"l10n_hu_471","471","Subordinated liabilities to other enterprise","liability_current","False","Hátrasorolt kötelezettségek más vállalkozással szemben"
"l10n_hu_472","472","Unpaid allowances","liability_current","False","Ki nem fizetett juttatások"
"l10n_hu_473","473","Social security contribution obligation","liability_current","False","Társadalombiztosítási járulékfizetési kötelezettség"
"l10n_hu_474","474","Payment obligations related to segregated funds","liability_current","False","Az elkülönített alapokhoz kapcsolódó fizetési kötelezettségek"
"l10n_hu_475","475","Liabilities vis-à-vis asset management organizations","liability_current","False","Kötelezettségek vagyonkezelő szervezetekkel szemben"
"l10n_hu_476","476","Other current liabilities to employees and members","liability_current","False","Egyéb rövid lejáratú kötelezettségek alkalmazottakkal és tagokkal szemben"
"l10n_hu_4761","4761","Compensation","liability_current","False","Kártérítés"
"l10n_hu_4762","4762","Judicial disqualification","liability_current","False","Bírói kizárás"
"l10n_hu_4764","4764","Deducted union fee","liability_current","False","Levont szakszervezeti díj"
"l10n_hu_4765","4765","Private pension fund contribution obligations","liability_current","False","Magánnyugdíjpénztári befizetési kötelezettségek"
"l10n_hu_478","478","Liabilities related to shares and securities","liability_current","False","Részvényekkel és értékpapírokkal kapcsolatos kötelezettségek"
"l10n_hu_479","479","Miscellaneous current liabilities","liability_current","False","Egyéb rövid lejáratú kötelezettségek"
"l10n_hu_4791","4791","Liabilities to insurance institutions","liability_current","False","Biztosító intézményekkel szembeni kötelezettségek"
"l10n_hu_4792","4792","Lenders","liability_current","False","Hitelezők"
"l10n_hu_4796","4796","Valuation difference on liabilities","liability_current","False","A kötelezettségek értékelési különbözete"
"l10n_hu_481","481","Accruals and deferrals","liability_current","False","Passzív időbeli elhatárolások"
"l10n_hu_482","482","Accruals and deferrals of costs and expenses","liability_current","False","Költségek és ráfordítások passzív időbeli elhatárolásai"
"l10n_hu_483","483","Deferred income","liability_current","False","Halasztott bevétel"
"l10n_hu_511","511","Cost of materials purchased","expense_direct_cost","False","A vásárolt anyagok költsége"
"l10n_hu_5111","5111","Raw material costs","expense_direct_cost","False","Nyersanyag költségek"
"l10n_hu_5112","5112","Excipient costs","expense_direct_cost","False","Segédanyagok költségei"
"l10n_hu_5113","5113","Fuel costs","expense_direct_cost","False","Üzemanyag költségek"
"l10n_hu_5114","5114","Cost of consumables, plant, equipment and other assets that are depreciated within one year","expense_direct_cost","False","A fogyóeszközök, gépek, berendezések és egyéb eszközök bekerülési értéke, amelyeket egy éven belül amortizálnak"
"l10n_hu_5115","5115","The cost of using work clothes that wear out within a year, protective clothing","expense_direct_cost","False","Egy éven belül elhasználódó munkaruha, védőruha használatának költsége"
"l10n_hu_5116","5116","Costs of forms and office supplies","expense_direct_cost","False","Nyomtatványok és irodaszerek költségei"
"l10n_hu_5117","5117","Fuel costs","expense_direct_cost","False","Üzemanyag költségek"
"l10n_hu_5118","5118","Costs of electricity and water consumption","expense_direct_cost","False","Villany- és vízfogyasztás költségei"
"l10n_hu_5119","5119","Other material use costs","expense_direct_cost","False","Egyéb anyaghasználati költségek"
"l10n_hu_512","512","Costs of depreciable assets within one year","expense_direct_cost","False","Az amortizálható eszközök egy éven belüli költsége"
"l10n_hu_522","522","Rent cost","expense_direct_cost","False","Bérleti költség"
"l10n_hu_523","523","Shipping, handling, loading and storage costs","expense_direct_cost","False","Szállítási, kezelési, rakodási és tárolási költségek"
"l10n_hu_524","524","Repair, maintenance costs","expense_direct_cost","False","Javítási, karbantartási költségek"
"l10n_hu_525","525","Costs of publications on electronic media","expense_direct_cost","False","Elektronikus médiában megjelent publikációk költségei"
"l10n_hu_526","526","Expenses for newspapers and books","expense_direct_cost","False","Újságok és könyvek költségei"
"l10n_hu_527","527","Postage, telephone, internet and other telecommunications charges","expense_direct_cost","False","Posta, telefon, internet és egyéb távközlési díjak"
"l10n_hu_528","528","Laundry, dry cleaning, cleaning costs","expense_direct_cost","False","Mosoda, vegytisztítás, takarítás költsége"
"l10n_hu_529","529","Costs of other services used","expense_direct_cost","False","Egyéb igénybe vett szolgáltatások költségei"
"l10n_hu_5291","5291","Costs of photocopying and reproduction","expense_direct_cost","False","A fénymásolás és sokszorosítás költségei"
"l10n_hu_5292","5292","District heating costs","expense_direct_cost","False","Távfűtési költségek"
"l10n_hu_5293","5293","Cost of warranty repairs performed with another company","expense_direct_cost","False","Más céggel végzett garanciális javítások költsége"
"l10n_hu_5294","5294","Organization of exhibitions, demonstrations, fairs","expense_direct_cost","False","Kiállítások, bemutatók, vásárok szervezése"
"l10n_hu_5295","5295","Advertising, publicity, propaganda costs","expense_direct_cost","False","Reklám, reklám, propaganda költségek"
"l10n_hu_5296","5296","Expenditure on education and training","expense_direct_cost","False","Oktatási és képzési kiadások"
"l10n_hu_531","531","Official administration and service fees and charges","expense_direct_cost","False","Hivatalos adminisztrációs és szolgáltatási díjak és díjak"
"l10n_hu_532","532","Financial and investment service fees","expense_direct_cost","False","Pénzügyi és befektetési szolgáltatási díjak"
"l10n_hu_5321","5321","Bank charges","expense_direct_cost","False","Bankköltségek"
"l10n_hu_533","533","Insurance fee","expense_direct_cost","False","Biztosítási díj"
"l10n_hu_534","534","Taxes, contributions, product charges to be accounted for as costs","expense_direct_cost","False","Költségként elszámolandó adók, járulékok, termékdíjak"
"l10n_hu_539","539","Miscellaneous other costs","expense_direct_cost","False","Egyéb egyéb költségek"
"l10n_hu_541","541","Wage cost","expense_direct_cost","False","Bérköltség"
"l10n_hu_542","542","Remuneration for the personal contribution of the owner","expense_direct_cost","False","A tulajdonos személyes hozzájárulásának díjazása"
"l10n_hu_551","551","Personal payments made to employees and members","expense_direct_cost","False","Személyes kifizetések az alkalmazottaknak és tagoknak"
"l10n_hu_5511","5511","Sick leave fee, sick pay charged to the employer, sick pay supplement","expense_direct_cost","False","Betegszabadság díj, munkáltatót terhelő táppénz, táppénz kiegészítés"
"l10n_hu_5512","5512","Severance pay","expense_direct_cost","False","Végkielégítés"
"l10n_hu_5513","5513","Reimbursement of other travel expenses","expense_direct_cost","False","Egyéb utazási költségek megtérítése"
"l10n_hu_5514","5514","Daily subsistence allowance","expense_direct_cost","False","Napidíj"
"l10n_hu_5515","5515","Remuneration supplement for disabled workers, paid benefits","expense_direct_cost","False","Fogyatékos munkavállalók bérpótléka, fizetett juttatások"
"l10n_hu_5516","5516","Holiday allowance","expense_direct_cost","False","Üdülési támogatás"
"l10n_hu_5517","5517","Housing subsidy, rent contribution","expense_direct_cost","False","Lakhatási támogatás, lakbér hozzájárulás"
"l10n_hu_5518","5518","Anniversary reward, object reward","expense_direct_cost","False","Jubileumi jutalom, tárgyi jutalom"
"l10n_hu_552","552","Welfare and cultural costs","expense_direct_cost","False","Jóléti és kulturális költségek"
"l10n_hu_559","559","Other personal payments","expense_direct_cost","False","Egyéb személyes fizetések"
"l10n_hu_5591","5591","Accident, life and pension insurance premiums paid by the employer","expense_direct_cost","False","A munkáltató által fizetett baleset-, élet- és nyugdíjbiztosítási díjak"
"l10n_hu_5592","5592","Employer membership fee contribution paid by an employer to a voluntary fund","expense_direct_cost","False","Munkáltatói tagdíj-hozzájárulás, amelyet a munkáltató fizet az önkéntes pénztárba"
"l10n_hu_5593","5593","Personal income tax on employers","expense_direct_cost","False","A munkáltatókat terhelő személyi jövedelemadó"
"l10n_hu_5594","5594","Employers' contribution to early retirement","expense_direct_cost","False","A munkaadók hozzájárulása a korengedményes nyugdíjhoz"
"l10n_hu_5595","5595","Invention fee, patent purchase price and utilization fee","expense_direct_cost","False","Feltalálási díj, szabadalom vételár és hasznosítási díj"
"l10n_hu_5596","5596","Fees for royalties, author's and other copyrighted works and related contributions","expense_direct_cost","False","Jogdíjak, szerzői és egyéb szerzői jog által védett művek és kapcsolódó hozzájárulások díjai"
"l10n_hu_5597","5597","Scholarships paid","expense_direct_cost","False","Kifizetett ösztöndíjak"
"l10n_hu_5598","5598","Entertainment expenses, meal allowance","expense_direct_cost","False","Szórakozási költségek, étkezési költségtérítés"
"l10n_hu_5599","5599","Insurance premiums for employees","expense_direct_cost","False","Biztosítási díjak az alkalmazottak számára"
"l10n_hu_561","561","Social contribution tax","expense_direct_cost","False","Szociális hozzájárulási adó"
"l10n_hu_564","564","Vocational training contribution","expense_direct_cost","False","Szakképzési hozzájárulás"
"l10n_hu_565","565","Rehabilitation contribution","expense_direct_cost","False","Rehabilitációs hozzájárulás"
"l10n_hu_571","571","Depreciation according to plan","expense_depreciation","False","Terv szerinti értékcsökkenés"
"l10n_hu_572","572","Depreciation of small assets (below huf 200 thousand individual purchase value) according to plan","expense_depreciation","False","Kis eszközök (200 ezer Ft egyedi beszerzési érték alatti) terv szerinti értékcsökkenése"
"l10n_hu_581","581","Change in stocks of own production","income","False","Saját termelés készleteinek változása"
"l10n_hu_582","582","Capitalized value of self-produced assets","income","False","Saját termelésű eszközök aktivált értéke"
"l10n_hu_811","811","Material cost","expense_direct_cost","False","Anyagköltség"
"l10n_hu_812","812","Value of services used","expense_direct_cost","False","Az igénybe vett szolgáltatások értéke"
"l10n_hu_813","813","Value of other services","expense_direct_cost","False","Egyéb szolgáltatások értéke"
"l10n_hu_814","814","Purchase value of goods sold","expense_direct_cost","False","Az eladott áruk beszerzési értéke"
"l10n_hu_815","815","Value of services sold (brokered)","expense_direct_cost","False","Eladott (közvetített) szolgáltatások értéke"
"l10n_hu_821","821","Wage cost","expense_direct_cost","False","Bérköltség"
"l10n_hu_822","822","Other payments of a personal nature","expense_direct_cost","False","Egyéb személyes jellegű kifizetések"
"l10n_hu_823","823","Wage contributions","expense_direct_cost","False","Bérjárulékok"
"l10n_hu_851","851","Sales and distribution costs","expense","False","Értékesítési és forgalmazási költségek"
"l10n_hu_852","852","Administrative expenses","expense","False","Igazgatási költségek"
"l10n_hu_853","853","Other overheads","expense","False","Egyéb rezsi"
"l10n_hu_861","861","Realized loss on sale of intangible assets and property, plant and equipment","expense","False","Az immateriális javak és tárgyi eszközök értékesítéséből származó realizált veszteség"
"l10n_hu_862","862","Book value of receivables sold, transferred (assigned)","expense","False","Eladott, átruházott (engedményezett) követelések könyv szerinti értéke"
"l10n_hu_863","863","Expenses related to the financial year for events occurring before the balance sheet date","expense","False","Pénzügyi évhez kapcsolódó ráfordítások a mérleg fordulónapja előtt bekövetkezett események miatt"
"l10n_hu_8631","8631","Damage-related payments, amounts payable","expense","False","Kárral kapcsolatos kifizetések, fizetendő összegek"
"l10n_hu_8632","8632","Fines, penalties, late fees, default interest, damages","expense","False","Bírság, kötbér, késedelmi pótlék, késedelmi kamat, kártérítés"
"l10n_hu_864","864","Subsequent - indirectly related - financially settled discount","expense","False","Utólagos - közvetetten kapcsolódó - pénzügyileg rendezett kedvezmény"
"l10n_hu_865","865","Provisioning","expense","False","Ellátás"
"l10n_hu_866","866","Impairment, unplanned depreciation","expense","False","Értékvesztés, nem tervezett értékcsökkenés"
"l10n_hu_8661","8661","Impairment of inventories","expense","False","Készletek értékvesztése"
"l10n_hu_8662","8662","Impairment of receivables","expense","False","Követelések értékvesztése"
"l10n_hu_8663","8663","Unscheduled amortization of intangible assets","expense","False","Immateriális javak előre nem tervezett amortizációja"
"l10n_hu_8664","8664","Depreciation of property, plant and equipment is unplanned","expense","False","Az ingatlanok, gépek és berendezések értékcsökkenése nem tervezett"
"l10n_hu_867","867","Taxes, fees, contributions","expense","False","Adók, illetékek, járulékok"
"l10n_hu_868","868","Expenses of exceptional size or occurrence","expense","False","Rendkívüli nagyságrendű vagy előfordulású költségek"
"l10n_hu_8681","8681","The book value of assets brought into the company","expense","False","A társaságba bevitt eszközök könyv szerinti értéke"
"l10n_hu_8682","8682","Carrying amount of a receivable that is uncollectible","expense","False","Behajthatatlan követelés könyv szerinti értéke"
"l10n_hu_8683","8683","Contractual amount of debt assumption","expense","False","A tartozás átvállalásának szerződés szerinti összege"
"l10n_hu_8684","8684","Financially supported aid transferred without obligation to repay","expense","False","Visszafizetési kötelezettség nélkül átutalt pénzügyileg támogatott támogatás"
"l10n_hu_8685","8685","Amount of aid granted for development purposes","expense","False","Fejlesztési célra nyújtott támogatás összege"
"l10n_hu_8686","8686","Book value of assets transferred free of charge","expense","False","Az ingyenesen átadott eszközök könyv szerinti értéke"
"l10n_hu_8687","8687","Cost of services provided free of charge","expense","False","Az ingyenesen nyújtott szolgáltatások költsége"
"l10n_hu_869","869","Miscellaneous other expenses","expense","False","Egyéb egyéb költségek"
"l10n_hu_8691","8691","Amount written off for bad debt","expense","False","Rossz adósság miatt leírt összeg"
"l10n_hu_8692","8692","Book value of missing, destroyed intangible assets and property, plant and equipment","expense","False","Hiányzó, megsemmisült immateriális javak és tárgyi eszközök könyv szerinti értéke"
"l10n_hu_8693","8693","Book value of missing, destroyed inventories derecognised","expense","False","A hiányzó, megsemmisült készletek könyv szerinti értéke kivezetve"
"l10n_hu_8694","8694","Loss-weighted inventory difference for commercial goods","expense","False","Kereskedelmi áruk veszteséggel súlyozott készletkülönbsége"
"l10n_hu_871","871","Expenses and exchange losses on shares","expense","False","Részvények ráfordításai és árfolyamveszteségei"
"l10n_hu_8711","8711","Equity costs, exchange losses for associates","expense","False","Tőkeköltségek, társult vállalkozások árfolyamveszteségei"
"l10n_hu_872","872","Expenses from fixed financial assets (securities, loans), exchange rate losses","expense","False","Befektetett pénzügyi eszközök (értékpapírok, hitelek) ráfordításai, árfolyamveszteségek"
"l10n_hu_8721","8721","It is derived from the expenses of fixed financial assets (securities, loans) and from the exchange rate losses of affiliated companies.","expense","False","A befektetett pénzügyi eszközök (értékpapírok, hitelek) ráfordításaiból, valamint a kapcsolt vállalkozások árfolyamveszteségéből származik."
"l10n_hu_873","873","Interest payable (interest paid) and interest - like expenses","expense","False","Fizetendő kamat (fizetett kamat) és kamatszerű kiadások"
"l10n_hu_8731","8731","Interest payable (interest paid) and interest - like expenses to an associate","expense","False","Fizetendő kamat (fizetett kamat) és kamat – hasonló költségek egy társult vállalkozás számára"
"l10n_hu_874","874","Impairment of shares, securities, bank deposits","expense","False","Részvények, értékpapírok, bankbetétek értékvesztése"
"l10n_hu_875","875","Foreign exchange loss on investment, sale and redemption of investment in current assets","expense","False","A forgóeszközökbe történő befektetés, értékesítés és visszaváltás árfolyamvesztesége"
"l10n_hu_876","876","Exchange rate loss at conversion and valuation","expense","False","Árfolyamveszteség átváltáskor és értékeléskor"
"l10n_hu_8761","8761","Exchange rate loss on the conversion of foreign exchange and currency inventories into huf","expense","False","Árfolyamveszteség a deviza- és devizakészletek forintra történő átváltásán"
"l10n_hu_8762","8762","Foreign exchange losses on assets and liabilities denominated in foreign currencies","expense","False","Devizában denominált eszközök és kötelezettségek árfolyamvesztesége"
"l10n_hu_8763","8763","Aggregate foreign exchange loss on the measurement of assets and liabilities denominated in foreign currencies at the balance sheet date","expense","False","Összesített árfolyamveszteség a devizában fennálló eszközök és kötelezettségek mérlegfordulónapi értékeléséből"
"l10n_hu_877","877","Other exchange losses, option fees","expense","False","Egyéb árfolyamveszteségek, opciós díjak"
"l10n_hu_878","878","Expenditure on purchased receivables","expense","False","Vásárolt követelésekkel kapcsolatos kiadások"
"l10n_hu_879","879","Other financial expenses","expense","False","Egyéb pénzügyi kiadások"
"l10n_hu_8791","8791","Valuation difference other financial expenses","expense","False","Értékelési különbözet egyéb pénzügyi ráfordítások"
"l10n_hu_891","891","Corporate tax","expense","False","Vállalati adó"
"l10n_hu_911","911","Domestic sales revenue","income","False","Belföldi árbevétel"
"l10n_hu_931","931","Export sales revenue","income","False","Export árbevétel"
"l10n_hu_961","961","Realized gains on the sale of intangible assets and property, plant and equipment","income","False","Az immateriális javak és ingatlanok, gépek és berendezések értékesítéséből származó realizált nyereség"
"l10n_hu_962","962","Recognized value of receivables sold, assigned (assigned)","income","False","Eladott, engedményezett (engedményezett) követelések elismert értéke"
"l10n_hu_963","963","Other income related to the financial year financially settled up to the balance sheet date","income","False","A pénzügyi évhez kapcsolódó egyéb bevételek a mérleg fordulónapjáig pénzügyileg rendezve"
"l10n_hu_9631","9631","Proceeds from claims","income","False","A követelésekből származó bevétel"
"l10n_hu_9632","9632","Fines, fines, penalties, default interest, damages received","income","False","Bírság, bírság, kötbér, késedelmi kamatok, kapott kártérítés"
"l10n_hu_9633","9633","Amounts received on receivables classified as irrecoverable and written off","income","False","A behajthatatlannak minősített és leírt követelésekre kapott összegek"
"l10n_hu_964","964","Subsequent - indirectly related - financially settled discount","income","False","Utólagos - közvetetten kapcsolódó - pénzügyileg rendezett kedvezmény"
"l10n_hu_965","965","Use of provision (decrease, termination)","income","False","Céltartalék felhasználása (csökkentés, megszüntetés)"
"l10n_hu_966","966","Reversal of impairment, unplanned depreciation","income","False","Értékvesztés visszaírása, nem tervezett értékcsökkenés"
"l10n_hu_9661","9661","Reversal of impairment, unplanned depreciation","income","False","Értékvesztés visszaírása, nem tervezett értékcsökkenés"
"l10n_hu_9662","9662","Reversal of impairment of receivables","income","False","Követelések értékvesztésének visszaírása"
"l10n_hu_9663","9663","Unscheduled depreciation of intangible assets","income","False","Az immateriális javak nem tervezett értékcsökkenése"
"l10n_hu_9664","9664","Unscheduled depreciation of property, plant and equipment","income","False","Az ingatlanok, gépek és berendezések nem tervezett értékcsökkenése"
"l10n_hu_967","967","Aid received without obligation to repay","income","False","Visszafizetési kötelezettség nélkül kapott támogatás"
"l10n_hu_968","968","Revenues of exceptional size or occurrence","income","False","Kivételes nagyságrendű vagy előfordulású bevételek"
"l10n_hu_9681","9681","The value of the assets brought into the company as specified in the memorandum of association","income","False","A társaságba bevitt eszközök alapító okiratban meghatározott értéke"
"l10n_hu_9682","9682","Value of liabilities waived","income","False","Elengedett kötelezettségek értéke"
"l10n_hu_9683","9683","Liability assumed by a third party in the course of a debt assumption","income","False","Harmadik fél által a tartozás átvállalása során vállalt felelősség"
"l10n_hu_9684","9684","Funds received and received without obligation to repay","income","False","Visszafizetési kötelezettség nélkül kapott és átvett pénzeszközök"
"l10n_hu_9685","9685","Funds received definitively for development purposes without a repayment obligation","income","False","Visszafizetési kötelezettség nélkül véglegesen fejlesztési célra átvett források"
"l10n_hu_9686","9686","Market value of assets received free of charge, received as gifts or bequests","income","False","Az ingyenesen átvett, ajándékba vagy hagyatékba kapott eszközök piaci értéke"
"l10n_hu_9687","9687","Market value of services received free of charge","income","False","Az ingyenesen igénybe vett szolgáltatások piaci értéke"
"l10n_hu_969","969","Miscellaneous other revenue","income","False","Egyéb egyéb bevételek"
"l10n_hu_9691","9691","Amount of compensation confirmed by the insurer","income","False","A biztosító által visszaigazolt kártérítés összege"
"l10n_hu_971","971","Dividends and shares received (due)","income_other","False","Kapott osztalék és részvény (esedékes)"
"l10n_hu_9711","9711","Dividends and interests received from associates","income_other","False","Partnerektől kapott osztalékok és kamatok"
"l10n_hu_972","972","Foreign exchange gains on the sale of shares","income_other","False","Részvények eladásából származó árfolyamnyereség"
"l10n_hu_9721","9721","Foreign exchange gains on interests sold to associates","income_other","False","A társult vállalkozásoknak eladott részesedések árfolyamnyeresége"
"l10n_hu_973","973","Interest and foreign exchange gains on financial assets","income_other","False","Pénzügyi eszközök kamat- és árfolyamnyeresége"
"l10n_hu_9731","9731","Interest on fixed financial assets, foreign exchange gains from associates","income_other","False","Befektetett pénzügyi eszközök kamatai, társult vállalkozásoktól származó árfolyamnyereség"
"l10n_hu_974","974","Other interest receivable and similar income","income_other","False","Egyéb kamatkövetelések és hasonló bevételek"
"l10n_hu_9741","9741","Other interest receivable and similar income from an associate","income_other","False","Egyéb kamatkövetelések és hasonló bevételek társult vállalkozástól"
"l10n_hu_975","975","Foreign exchange gain on investment, sale and redemption of securities","income_other","False","Devizaárfolyam-nyereség befektetésen, értékpapírok eladásán és visszaváltásán"
"l10n_hu_976","976","Exchange rate gains on exchange and valuation","income_other","False","Átváltási és értékelési árfolyamnyereség"
"l10n_hu_9761","9761","Foreign exchange gains on the translation of foreign exchange and currency inventories into huf","income_other","False","Deviza- és devizakészletek forintra történő átváltásából származó árfolyamnyereség"
"l10n_hu_9762","9762","Foreign exchange gains and losses on assets and liabilities denominated in foreign currencies","income_other","False","Devizában denominált eszközök és kötelezettségek árfolyamnyeresége és -vesztesége"
"l10n_hu_9763","9763","Aggregate foreign exchange gains on the measurement of assets and liabilities denominated in foreign currencies at the balance sheet date","income_other","False","A devizában denominált eszközök és kötelezettségek mérlegfordulónapi értékeléséből származó összesített árfolyamnyereség"
"l10n_hu_977","977","Other foreign exchange gains, option fee income","income_other","False","Egyéb árfolyamnyereség, opciós díjbevétel"
"l10n_hu_978","978","Revenue from purchased receivables","income_other","False","Vásárolt követelésekből származó bevétel"
"l10n_hu_979","979","Other financial income","income_other","False","Egyéb pénzügyi bevételek"
"l10n_hu_9791","9791","Valuation difference on other financial income","income_other","False","Egyéb pénzügyi bevételek értékelési különbözete"

```

## File: data\template\account.fiscal.position-hu.csv

```csv
"id","name","sequence","auto_apply","vat_required","country_id","country_group_id","tax_ids/tax_src_id","tax_ids/tax_dest_id","name@hu"
"fiscal_position_hu_exempt","Exempt taxpayer","","","","","","F27","FA","Mentes adózó"
"","","","","","","","F18","FA",""
"","","","","","","","F5","FA",""
"","","","","","","","V27","VA",""
"","","","","","","","V27TE","VA",""
"","","","","","","","V18","VA",""
"","","","","","","","V5","VA",""
"fiscal_position_hu_national","Domestic","10","1","1","base.hu","","","","Belföldi"
"fiscal_position_hu_eu_private","EU partner private","20","1","","","base.europe","","","EU partner magán"
"fiscal_position_hu_eu","EU partner","30","1","1","","base.europe","F27","FEUT",""
"","","","","","","","F18","FEUT",""
"","","","","","","","F5","FEUT",""
"","","","","","","","V27","VEU27T",""
"","","","","","","","V27TE","VEU27TE",""
"fiscal_position_hu_eu_out","Partner outside the EU","40","1","","","","F27","FEXT","EU-n kívüli partner"
"","","","","","","","F18","FEUT",""
"","","","","","","","F5","FEXT",""
"","","","","","","","V27","VIMK",""
"","","","","","","","V27TE","VIMK",""

```

## File: data\template\account.group-hu.csv

```csv
"id","code_prefix_start","code_prefix_end","name","name@hu"
"l10n_hu_group_1","1","","Fixed assets","Befektetett eszközök"
"l10n_hu_group_11","11","","Intangible assets","Immateriális javak"
"l10n_hu_group_12","12","","Real estate and associated rights","Ingatlan és kapcsolódó jogok"
"l10n_hu_group_13","13","","Technical equipment, machinery, vehicles","Műszaki berendezések, gépek, járművek"
"l10n_hu_group_14","14","","Other equipment, equipment and vehicles","Egyéb berendezések, felszerelések és járművek"
"l10n_hu_group_15","15","","Breeding animals","Tenyészállatok"
"l10n_hu_group_16","16","","Investments, renovations","Beruházások, felújítások"
"l10n_hu_group_17","17","","Ownership investments (shares)","Tulajdonosi befektetések (részvények)"
"l10n_hu_group_18","18","","Debt securities","Hitelviszonyt megtestesítő értékpapíro"
"l10n_hu_group_19","19","","Long-term loans","Hosszú lejáratú hitelek"
"l10n_hu_group_2","2","","Stocks","Készletek"
"l10n_hu_group_21-22","21","22","Materials","Anyagok"
"l10n_hu_group_23","23","","Work in progress and semi-finished products","Befejezetlen termelés és félkész termékek"
"l10n_hu_group_24","24","","Adult, fattening and other animals","Felnőtt, hízó és egyéb állatok"
"l10n_hu_group_25","25","","Finished products","Elkészült termékek"
"l10n_hu_group_26","26","","Commercial goods","Kereskedelmi áruk"
"l10n_hu_group_27","27","","Intermediate services","Köztes szolgáltatások"
"l10n_hu_group_28","28","","Deposit feeds","Betéti hírcsatornák"
"l10n_hu_group_3","3","","Receivables, securities, cash and current accounts","Követelések, értékpapírok, készpénz és folyószámlák"
"l10n_hu_group_31","31","","Receivables from supply of goods and services (customers)","Termékértékesítésből és szolgáltatásnyújtásból származó követelések (vevők)"
"l10n_hu_group_32","32","","Claims against associated and significant ownership","Társult és jelentős tulajdonjoggal szembeni követelések"
"l10n_hu_group_33","33","","Claims against other participating undertakings","Más részt vevő vállalkozásokkal szembeni követelések"
"l10n_hu_group_34","34","","Receivables","Követelések"
"l10n_hu_group_35","35","","Advances given","Adott előlegek"
"l10n_hu_group_36","36","","Other claims","Egyéb követelések"
"l10n_hu_group_37","37","","Securities","Értékpapír"
"l10n_hu_group_38","38","","Funds","Alapok"
"l10n_hu_group_39","39","","Active definitions","Aktív definíciók"
"l10n_hu_group_4","4","","Sources","Források"
"l10n_hu_group_41","41","","Equity","Saját tőke"
"l10n_hu_group_42","42","","Provisions","Rendelkezések"
"l10n_hu_group_43","43","","Subsequent liabilities","Későbbi kötelezettségek"
"l10n_hu_group_44","44","","Long-term liabilities","Hosszú lejáratú kötelezettségek"
"l10n_hu_group_45-47","45","47","Short-term liabilities","Rövid lejáratú kötelezettségek"
"l10n_hu_group_48","48","","Liabilities","Kötelezettségek"
"l10n_hu_group_5","5","","Costs","Költségek"
"l10n_hu_group_51","51","","Material cost","Anyagköltség"
"l10n_hu_group_52","52","","Costs of services used","Az igénybe vett szolgáltatások költségei"
"l10n_hu_group_53","53","","Costs of other services","Egyéb szolgáltatások költségei"
"l10n_hu_group_54","54","","Wage cost","Bérköltség"
"l10n_hu_group_55","55","","Other personal payments","Egyéb személyes fizetések"
"l10n_hu_group_56","56","","Contributions","Hozzájárulások"
"l10n_hu_group_57","57","","Description of impairment","Az értékvesztés leírása"
"l10n_hu_group_58","58","","Own work capitalized value","Saját munka aktivált értéke"
"l10n_hu_group_59","59","","Non-cost transfer account","Nem költség átutalási számla"
"l10n_hu_group_6","6","","Costs, overall costs","Költségek, összköltségek"
"l10n_hu_group_61","61","","Costs of repair-maintenance plants","Javító-karbantartó üzemek költségei"
"l10n_hu_group_62","62","","Costs of establishments (units)","Az alapítás költségei (egység)"
"l10n_hu_group_63","63","","Machinery costs","A gépek költségei"
"l10n_hu_group_64-65","64","65","Overall costs of operating management","A működési menedzsment összköltsége"
"l10n_hu_group_66","66","","Overall costs of central management","A központi irányítás összköltsége"
"l10n_hu_group_67","67","","Sales and distribution costs","Értékesítési és forgalmazási költségek"
"l10n_hu_group_68","68","","Allocated other overall costs","Felosztott egyéb összköltség"
"l10n_hu_group_69","69","","Cost positions transfer of cost types","Költségpozíciók költségtípusok átvitele"
"l10n_hu_group_7","7","","Costs of activities","Tevékenységek költségei"
"l10n_hu_group_71-74","71","74","Costs of activities","Tevékenységek költségei"
"l10n_hu_group_75","75","","Costs of the service","A szolgáltatás költségei"
"l10n_hu_group_76","76","","Production costs of centers","A központok előállítási költségei"
"l10n_hu_group_77-78","77","78","Marketing costs","Marketing költségek"
"l10n_hu_group_79","79","","Transfer of costs of activities","Tevékenységek költségeinek áthárítása"
"l10n_hu_group_8","8","","Accounted costs of sales and expenses","Elszámolt értékesítési költségek és kiadások"
"l10n_hu_group_81","81","","Material expenses","Anyagi kiadások"
"l10n_hu_group_82","82","","Personnel expenses","Személyi jellegű ráfordítások"
"l10n_hu_group_83","83","","Description of impairment","Az értékvesztés leírása"
"l10n_hu_group_85","85","","Indirect costs of sale","Az értékesítés közvetett költségei"
"l10n_hu_group_86","86","","Other expenses","Más költségek"
"l10n_hu_group_87","87","","Expenditure on financial operations","Pénzügyi műveletek kiadásai"
"l10n_hu_group_89","89","","Taxes on profit","Nyereségadók"
"l10n_hu_group_9","9","","Sales revenue and revenue","Árbevétel és bevétel"
"l10n_hu_group_91-92","91","92","Turnover of domestic sales","Belföldi értékesítés forgalma"
"l10n_hu_group_93-94","93","94","Export sales revenue","Export árbevétel"
"l10n_hu_group_96","96","","Other revenues","Egyéb bevételek"
"l10n_hu_group_97","97","","Revenue from financial operations","Pénzügyi műveletekből származó bevétel"
"l10n_hu_group_0","0","","Registration accounts","Regisztrációs számlák"

```

## File: data\template\account.tax-hu.csv

```csv
"id","sequence","name","description","invoice_label","price_include","amount","amount_type","type_tax_use","tax_group_id","repartition_line_ids/factor_percent","repartition_line_ids/repartition_type","repartition_line_ids/document_type","repartition_line_ids/tag_ids","repartition_line_ids/account_id","description@hu","invoice_label@hu"
"F27","10","27%","27% VAT","27%","False","27.0","percent","sale","tax_group_afa_27","100","base","invoice","+base_pay_27","","27% ÁFA","27%"
"","","","","","","","","","","100","tax","invoice","+vat_pay_27","l10n_hu_467","",""
"","","","","","","","","","","100","base","refund","-base_pay_27","","",""
"","","","","","","","","","","100","tax","refund","-vat_pay_27","l10n_hu_467","",""
"F18","20","18%","18% VAT","18%","False","18.0","percent","sale","tax_group_afa_18","100","base","invoice","+base_pay_18","","18% ÁFA","18%"
"","","","","","","","","","","100","tax","invoice","+vat_pay_18","l10n_hu_467","",""
"","","","","","","","","","","100","base","refund","-base_pay_18","","",""
"","","","","","","","","","","100","tax","refund","-vat_pay_18","l10n_hu_467","",""
"F5","30","5%","5% VAT","5%","False","5.0","percent","sale","tax_group_afa_5","100","base","invoice","+base_pay_5","","5% ÁFA","5%"
"","","","","","","","","","","100","tax","invoice","+vat_pay_5","l10n_hu_467","",""
"","","","","","","","","","","100","base","refund","-base_pay_5","","",""
"","","","","","","","","","","100","tax","refund","-vat_pay_5","l10n_hu_467","",""
"FA","40","0% AAM","0% Personal tax exemption","0%","False","0.0","percent","sale","tax_group_afa_0","100","base","invoice","+base_pay_exempt_tax","","0% Alanyi Adómentes","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_pay_exempt_tax","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"FT","50","0% TAM","0% Tax-exempt activity","0%","False","0.0","percent","sale","tax_group_afa_0","100","base","invoice","+base_pay_exempt_property","","0% Tárgyi Adómentes","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_pay_exempt_property","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"FKKS","60","0% S EXEMPT","0% Services Exempt","0%","False","0.0","percent","sale","tax_group_afa_0","100","base","invoice","+base_pay_exempt","","0% ÁFA körön kívüli szolgáltatás","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_pay_exempt","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"FKKT","61","0% G EXEMPT","0% Goods Exempt","0%","False","0.0","percent","sale","tax_group_afa_0","100","base","invoice","+base_pay_exempt","","0% ÁFA körön kívüli termék értékesítés","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_pay_exempt","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"FF","70","0% R","0% Domestic reverse charge","0%","False","0.0","percent","sale","tax_group_afa_0","100","base","invoice","+base_rec_reverse","","0% Fordított ÁFA","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_rec_reverse","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"FEUSZ","80","0% EU S","0% Intra-community delivery of services","0%","False","0.0","percent","sale","tax_group_afa_0","100","base","invoice","+base_pay_intra","","0% EU szolgáltatás nyújtás, adómentes","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_pay_intra","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"FEUT","81","0% EU G","0% Goods Intra-community Deliveries","0%","False","0.0","percent","sale","tax_group_afa_0","100","base","invoice","+base_pay_intra","","0% EU termékértékesítés, adómentes","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_pay_intra","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"FEXS","90","0% S EX","0% Extra-community delivery of services","0%","False","0.0","percent","sale","tax_group_afa_0","100","base","invoice","+base_pay_exports","","0% Export szolgáltatás nyújtás","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_pay_exports","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"FEXT","91","0% G EX","0% Extra-community deliver of goods","0%","False","0.0","percent","sale","tax_group_afa_0","100","base","invoice","+base_pay_exports","","0% Export termékértékesítés","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_pay_exports","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"V27","100","27%","27% VAT","27%","False","27.0","percent","purchase","tax_group_afa_27","100","base","invoice","+base_rec_27","","27% ÁFA","27%"
"","","","","","","","","","","100","tax","invoice","+vat_rec_27","l10n_hu_466","",""
"","","","","","","","","","","100","base","refund","-base_rec_27","","",""
"","","","","","","","","","","100","tax","refund","-vat_rec_27","l10n_hu_466","",""
"V27TE","110","27% PPE","27% Property, plant and equipment","27%","False","27.0","percent","purchase","tax_group_afa_27","100","base","invoice","+base_rec_27","","27% Tárgyi eszköz","27%"
"","","","","","","","","","","100","tax","invoice","+vat_rec_27","l10n_hu_466","",""
"","","","","","","","","","","100","base","refund","-base_rec_27","","",""
"","","","","","","","","","","100","tax","refund","-vat_rec_27","l10n_hu_466","",""
"V18","120","18%","18% VAT","18%","False","18.0","percent","purchase","tax_group_afa_18","100","base","invoice","+base_rec_18","","18% ÁFA","18%"
"","","","","","","","","","","100","tax","invoice","+vat_rec_18","l10n_hu_466","",""
"","","","","","","","","","","100","base","refund","-base_rec_18","","",""
"","","","","","","","","","","100","tax","refund","-vat_rec_18","l10n_hu_466","",""
"V5","130","5%","5% VAT","5%","False","5.0","percent","purchase","tax_group_afa_5","100","base","invoice","+base_rec_5","","5% ÁFA","5%"
"","","","","","","","","","","100","tax","invoice","+vat_rec_5","l10n_hu_466","",""
"","","","","","","","","","","100","base","refund","-base_rec_5","","",""
"","","","","","","","","","","100","tax","refund","-vat_rec_5","l10n_hu_466","",""
"VKOMP7","140","7% CS","7% Compensation Surcharge","7% CS","False","7.0","percent","purchase","tax_group_afa_komp","100","base","invoice","+base_rec_compensation","","7% Kompenzációs felár","7% KF"
"","","","","","","","","","","100","tax","invoice","+vat_rec_compensation","l10n_hu_466","",""
"","","","","","","","","","","100","base","refund","-base_rec_compensation","","",""
"","","","","","","","","","","100","tax","refund","-vat_rec_compensation","l10n_hu_466","",""
"VKOMP12","150","12% CS","12% Compensation Surcharge","12% CS","False","12.0","percent","purchase","tax_group_afa_komp","100","base","invoice","+base_rec_compensation","","12% Kompenzációs felár","12% KF"
"","","","","","","","","","","100","tax","invoice","+vat_rec_compensation","l10n_hu_466","",""
"","","","","","","","","","","100","base","refund","-base_rec_compensation","","",""
"","","","","","","","","","","100","tax","refund","-vat_rec_compensation","l10n_hu_466","",""
"VA","160","0% AAM","0% Personal tax exemption","0%","False","0.0","percent","purchase","tax_group_afa_0","100","base","invoice","+base_rec_exempt_tax","","0% Alanyi Adómentes","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_rec_exempt_tax","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"VT","170","0% TAM","0% Tax exempt activity","0%","False","0.0","percent","purchase","tax_group_afa_0","100","base","invoice","+base_rec_exempt_material","","0% Tárgyi Adómentes","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_rec_exempt_material","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"VKKS","180","0% S EXEMPT","0% Services Exempt","0%","False","0.0","percent","purchase","tax_group_afa_0","100","base","invoice","+base_rec_exempt","","0% Áfa körön kívüli szolgáltatás","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_rec_exempt","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"VKKT","181","0% G EXEMPT","0% Goods Exempt","0%","False","0.0","percent","purchase","tax_group_afa_0","100","base","invoice","+base_rec_exempt","","0% Áfa körön kívüli termék beszerzés","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_rec_exempt","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"VF","190","0% R","0% Domestic Reverse charge","0%","False","0.0","percent","purchase","tax_group_afa_0","100","base","invoice","+base_rec_reverse","","0% Fordított ÁFA","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_rec_reverse","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"VEU27S","200","27% EU S","27% Services Intra-community Acquisition","27%","False","27.0","percent","purchase","tax_group_afa_27","100","base","invoice","+base_rec_intra","","27% EU termék beszerzés","27%"
"","","","","","","","","","","-100","tax","invoice","+vat_pay_27","l10n_hu_467","",""
"","","","","","","","","","","100","tax","invoice","+vat_rec_27","l10n_hu_466","",""
"","","","","","","","","","","100","base","refund","-base_rec_intra","","",""
"","","","","","","","","","","-100","tax","refund","-vat_pay_27","l10n_hu_467","",""
"","","","","","","","","","","100","tax","refund","-vat_rec_27","l10n_hu_466","",""
"VEU27T","201","27% EU G","27% Goods Intra-community Acquisition","27%","False","27.0","percent","purchase","tax_group_afa_27","100","base","invoice","+base_rec_intra","","27% EU szolgáltatás","27%"
"","","","","","","","","","","-100","tax","invoice","+vat_pay_27","l10n_hu_467","",""
"","","","","","","","","","","100","tax","invoice","+vat_rec_27","l10n_hu_466","",""
"","","","","","","","","","","100","base","refund","-base_rec_intra","","",""
"","","","","","","","","","","-100","tax","refund","-vat_pay_27","l10n_hu_467","",""
"","","","","","","","","","","100","tax","refund","-vat_rec_27","l10n_hu_466","",""
"VEU27TE","210","27% EU P","27% ICA (Property)","27%","False","27.0","percent","purchase","tax_group_afa_27","100","base","invoice","+base_rec_intra","","27% EU Tárgyi eszköz","27%"
"","","","","","","","","","","-100","tax","invoice","+vat_pay_27","l10n_hu_467","",""
"","","","","","","","","","","100","tax","invoice","+vat_rec_27","l10n_hu_466","",""
"","","","","","","","","","","100","base","refund","-base_rec_intra","","",""
"","","","","","","","","","","-100","tax","refund","-vat_pay_27","l10n_hu_467","",""
"","","","","","","","","","","100","tax","refund","-vat_rec_27","l10n_hu_466","",""
"VEUM","220","0% ICA","0% Intra-community Acquisition","0%","False","0.0","percent","purchase","tax_group_afa_0","100","base","invoice","+base_rec_intra","","0% EU Adómentes szolgáltatás","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_rec_intra","","",""
"","","","","","","","","","","100","tax","refund","","","",""
"VIMS","230","27% EX","27% Import","27%","False","27.0","percent","purchase","tax_group_afa_27","100","base","invoice","+base_rec_import","","27% Import szolgáltatás","27%"
"","","","","","","","","","","-100","tax","invoice","+vat_pay_27","l10n_hu_467","",""
"","","","","","","","","","","100","tax","invoice","+vat_rec_27","l10n_hu_466","",""
"","","","","","","","","","","100","base","refund","-base_rec_import","","",""
"","","","","","","","","","","-100","tax","refund","-vat_pay_27","l10n_hu_467","",""
"","","","","","","","","","","100","tax","refund","-vat_rec_27","l10n_hu_466","",""
"VIMK","240","27% EX S","27% Import (Sourcing)","27%","False","27.0","percent","purchase","tax_group_afa_27","100","base","invoice","+base_rec_import","","27% Import beszerzés","27%"
"","","","","","","","","","","100","tax","invoice","+vat_rec_27","l10n_hu_466","",""
"","","","","","","","","","","100","base","refund","-base_rec_import","","",""
"","","","","","","","","","","100","tax","refund","-vat_rec_27","l10n_hu_466","",""
"VIM","250","0% EX","0% Import","0%","False","0.0","percent","purchase","tax_group_afa_0","100","base","invoice","+base_rec_import","","0% Import adómentes szolgáltatás","0%"
"","","","","","","","","","","100","tax","invoice","","","",""
"","","","","","","","","","","100","base","refund","-base_rec_import","","",""
"","","","","","","","","","","100","tax","refund","","","",""

```

## File: data\template\account.tax.group-hu.csv

```csv
"id","name","country_id","tax_payable_account_id","tax_receivable_account_id","name@hu"
"tax_group_afa_27","27% VAT","base.hu","l10n_hu_468","l10n_hu_468","27% ÁFA"
"tax_group_afa_18","18% VAT","base.hu","l10n_hu_468","l10n_hu_468","18% ÁFA"
"tax_group_afa_5","5% VAT","base.hu","l10n_hu_468","l10n_hu_468","5% ÁFA"
"tax_group_afa_0","0% VAT","base.hu","l10n_hu_468","l10n_hu_468","0% ÁFA"
"tax_group_afa_komp","Compensation surcharge","base.hu","l10n_hu_468","l10n_hu_468","Kompenzációs felár"

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import api, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    @api.depends('country_code', 'move_type')
    def _compute_show_delivery_date(self):
        # EXTENDS 'account'
        super()._compute_show_delivery_date()
        for move in self:
            if move.country_code == 'HU':
                move.show_delivery_date = move.is_sale_document()

    def _post(self, soft=True):
        res = super()._post(soft)
        for move in self:
            if move.country_code == 'HU' and move.is_sale_document() and not move.delivery_date:
                move.delivery_date = move.invoice_date
        return res

```

## File: models\res_partner.py

```python
from odoo import api, fields, models


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_hu_eu_vat = fields.Char(compute='_compute_l10n_hu_eu_vat')

    @api.depends('vat')
    def _compute_l10n_hu_eu_vat(self):
        for partner in self:
            if partner.country_code == 'HU' and partner.vat:
                partner.l10n_hu_eu_vat = partner._convert_hu_local_to_eu_vat(partner.vat)
            else:
                partner.l10n_hu_eu_vat = False

```

## File: models\template_hu.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo import models
from odoo.addons.account.models.chart_template import template


class AccountChartTemplate(models.AbstractModel):
    _inherit = 'account.chart.template'

    @template('hu')
    def _get_hu_template_data(self):
        return {
            'property_account_receivable_id': 'l10n_hu_311',
            'property_account_payable_id': 'l10n_hu_454',
            'property_account_expense_categ_id': 'l10n_hu_811',
            'property_account_income_categ_id': 'l10n_hu_911',
            'code_digits': '6',
        }

    @template('hu', 'res.company')
    def _get_hu_res_company(self):
        return {
            self.env.company.id: {
                'account_fiscal_country_id': 'base.hu',
                'tax_calculation_rounding_method': 'round_globally',
                'bank_account_code_prefix': '384',
                'cash_account_code_prefix': '381',
                'transfer_account_code_prefix': '389',
                'income_currency_exchange_account_id': 'l10n_hu_976',
                'expense_currency_exchange_account_id': 'l10n_hu_876',
                'account_sale_tax_id': 'F27',
                'account_purchase_tax_id': 'V27',
            },
        }

```

## File: models\__init__.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from . import template_hu
from . import account_move
from . import res_partner

```

