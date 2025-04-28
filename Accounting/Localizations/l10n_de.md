# Odoo Module: l10n_de

Category: Accounting/Localizations

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import models

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Germany - Accounting',
    "version": "2.0",
    'author': 'openbig.org',
    'website': 'http://www.openbig.org',
    'category': 'Accounting/Localizations',
    'description': """
Dieses  Modul beinhaltet einen deutschen Kontenrahmen basierend auf dem SKR03.
==============================================================================

German accounting chart and localization.
    """,
    'depends': [
        'account',
        'base_iban',
        'base_vat',
        'l10n_din5008',
    ],
    'data': [
        'data/account_account_tags_data.xml',
        'views/account_view.xml',
        'views/res_company_views.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_account_tags_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.de"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_de_tag_01" model="account.report.line">
                <field name="name">Bemessungsgrundlage</field>
                <field name="sequence">10</field>
                <field name="aggregation_formula"></field>
                <field name="children_ids">
                    <record id="tax_report_de_tag_17" model="account.report.line">
                        <field name="name">Anmeldung der Umsatzsteuer-Vorauszahlung</field>
                        <field name="code">I_ANMELDUNG_DER_UMSATZSTEUERVORAUSZAHLUNG_ZEILE_17_GRUNDLAGE</field>
                        <field name="sequence">20</field>
                        <field name="aggregation_formula"></field>
                        <field name="children_ids">
                            <record id="tax_report_de_tag_18" model="account.report.line">
                                <field name="name">Lieferungen und sonstige Leistungen</field>
                                <field name="code">LIEFERUNGEN_UND_SONSTIGE_LEISTUNGEN_ZEILE_18_GRUNDLAGE</field>
                                <field name="sequence">30</field>
                                <field name="aggregation_formula">STEUERFREIE_UMSATZE_MIT_VORSTEUERABZUG_ZEILE_19.balance + DE_48.balance + STEUERPFLICHTIGE_UMSATZE_ZEILE_25.balance + AGG_DE_31.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_de_tag_19" model="account.report.line">
                                        <field name="name">Steuerpflichtige Umsätze</field>
                                        <field name="code">STEUERFREIE_UMSATZE_MIT_VORSTEUERABZUG_ZEILE_19</field>
                                        <field name="sequence">40</field>
                                        <field name="aggregation_formula">DE_81_BASE.balance + DE_86_BASE.balance + DE_87.balance + DE_35.balance + DE_77.balance + DE_76.balance</field>
                                        <field name="children_ids">
                                            <record id="tax_report_de_tag_81" model="account.report.line">
                                                <field name="name">81. zum Steuersatz von 19 % (zeile 12)</field>
                                                <field name="code">DE_81_BASE</field>
                                                <field name="sequence">50</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_81_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">81_BASE</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_86" model="account.report.line">
                                                <field name="name">86. zum Steuersatz von 7 % (zeile 13)</field>
                                                <field name="code">DE_86_BASE</field>
                                                <field name="sequence">60</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_86_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">86_BASE</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_87" model="account.report.line">
                                                <field name="name">87. zum Steuersatz von 0 % (zeile 14)</field>
                                                <field name="code">DE_87</field>
                                                <field name="sequence">70</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_87_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">87</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_35" model="account.report.line">
                                                <field name="name">35. zu anderen Steuersätzen (zeile 15)</field>
                                                <field name="code">DE_35</field>
                                                <field name="sequence">80</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_35_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">35</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_77" model="account.report.line">
                                                <field name="name">77. Lieferungen land- und forstwirtschaftlicher Betriebe nach § 24 UStG an Abnehmer mit USt-IdNr. (zeile 16)</field>
                                                <field name="code">DE_77</field>
                                                <field name="sequence">90</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_77_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">77</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_76" model="account.report.line">
                                                <field name="name">76. Umsätze, für die eine Steuer nach § 24 UStG zu entrichten ist (zeile 17)</field>
                                                <field name="code">DE_76</field>
                                                <field name="sequence">100</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_76_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">76</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_25" model="account.report.line">
                                        <field name="name">Steuerfreie Umsätze mit Vorsteuerabzug</field>
                                        <field name="code">STEUERPFLICHTIGE_UMSATZE_ZEILE_25</field>
                                        <field name="sequence">110</field>
                                        <field name="aggregation_formula">DE_41.balance + DE_44.balance + DE_49.balance + DE_43.balance</field>
                                        <field name="children_ids">
                                            <record id="tax_report_de_tag_41" model="account.report.line">
                                                <field name="name">41. an Abnehmer mit USt-IdNr (zeile 18)</field>
                                                <field name="code">DE_41</field>
                                                <field name="sequence">120</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_41_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">41</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_44" model="account.report.line">
                                                <field name="name">44. neuer Fahrzeuge an Abnehmer ohne USt-IdNr (zeile 19)</field>
                                                <field name="code">DE_44</field>
                                                <field name="sequence">130</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_44_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">44</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_49" model="account.report.line">
                                                <field name="name">49. neuer Fahrzeuge außerhalb eines Unternehmens (zeile 20)</field>
                                                <field name="code">DE_49</field>
                                                <field name="sequence">140</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_49_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">49</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_43" model="account.report.line">
                                                <field name="name">43. Weitere steuerfreie Umsätze mit Vorsteuerabzug (zeile 21)</field>
                                                <field name="code">DE_43</field>
                                                <field name="sequence">150</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_43_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">43</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_24" model="account.report.line">
                                        <field name="name">48. Steuerfreie Umsätze ohne Vorsteuerabzug (zeile 22)</field>
                                        <field name="code">DE_48</field>
                                        <field name="sequence">160</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_24_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">48</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_31" model="account.report.line">
                                        <field name="name">Innergemeinschaftliche Erwerbe</field>
                                        <field name="code">AGG_DE_31</field>
                                        <field name="sequence">170</field>
                                        <field name="aggregation_formula">DE_91.balance + DE_89_BASE.balance + DE_93_BASE.balance + DE_90.balance + DE_95.balance + DE_94.balance</field>
                                        <field name="children_ids">
                                            <record id="tax_report_de_tag_91" model="account.report.line">
                                                <field name="name">91. Steuerfreie innergemeinschaftliche Erwerbe (zeile 23)</field>
                                                <field name="code">DE_91</field>
                                                <field name="sequence">180</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_91_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">91</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_89" model="account.report.line">
                                                <field name="name">89. Steuerpflichtige innergemeinschaftliche Erwerbe zum Steuersatz von 19 % (zeile 24)</field>
                                                <field name="code">DE_89_BASE</field>
                                                <field name="sequence">190</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_89_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">89_BASE</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_93" model="account.report.line">
                                                <field name="name">93. zum Steuersatz von 7 % (zeile 25)</field>
                                                <field name="code">DE_93_BASE</field>
                                                <field name="sequence">200</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_93_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">93_BASE</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_90" model="account.report.line">
                                                <field name="name">90. zum Steuersatz von 0 % (zeile 26)</field>
                                                <field name="code">DE_90</field>
                                                <field name="sequence">210</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_90_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">90</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_95" model="account.report.line">
                                                <field name="name">95. zu anderen Steuersätzen (zeile 27)</field>
                                                <field name="code">DE_95</field>
                                                <field name="sequence">220</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_95_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">95</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_94" model="account.report.line">
                                                <field name="name">94. neuer Fahrzeuge von Lieferern ohne (zeile 28)</field>
                                                <field name="code">DE_94</field>
                                                <field name="sequence">230</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_94_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">94</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_de_tag_46" model="account.report.line">
                                <field name="name">Leistungsempfänger als Steuerschuldner</field>
                                <field name="code">LEISTUNGSEMPFANGER_ALS_STEUERSCHULDNER_ZEILE_46_GRUNDLAGE</field>
                                <field name="sequence">240</field>
                                <field name="aggregation_formula">DE_46.balance + DE_73.balance + DE_84.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_de_tag_48" model="account.report.line">
                                        <field name="name">46. Steuerpflichtige sonstige Leistungen eines im übrigen Gemeinschaftsgebiet ansässigen Unternehmers (zeile 29)</field>
                                        <field name="code">DE_46</field>
                                        <field name="sequence">250</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_48_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">46</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_73" model="account.report.line">
                                        <field name="name">73. Lieferungen sicherungsübereigneter Gegenstände und Umsätze, die unter das GrEStG fallen (zeile 30)</field>
                                        <field name="code">DE_73</field>
                                        <field name="sequence">260</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_73_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">73</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_84" model="account.report.line">
                                        <field name="name">84. Andere Leistungen (zeile 31)</field>
                                        <field name="code">DE_84</field>
                                        <field name="sequence">270</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_84_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">84</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_de_tag_37" model="account.report.line">
                                <field name="name">Ergänzende Angaben zu Umsätzen</field>
                                <field name="code">ERGANZENDE_ANGABEN_ZU_UMSATZEN_ZEILE_37_GRUNDLAGE</field>
                                <field name="sequence">280</field>
                                <field name="aggregation_formula">DE_42.balance + DE_60.balance + DE_21.balance + DE_45_BASE.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_de_tag_42" model="account.report.line">
                                        <field name="name">42. Dreiecksgeschäften (zeile 32)</field>
                                        <field name="code">DE_42</field>
                                        <field name="sequence">280</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_42_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">42</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_60" model="account.report.line">
                                        <field name="name">60. Übrige steuerpflichtige Umsätze, für die der Leistungsempfänger die Steuer nach § 13b Abs. 5 UStG schuldet (zeile 33)</field>
                                        <field name="code">DE_60</field>
                                        <field name="sequence">290</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_60_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">60</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_21" model="account.report.line">
                                        <field name="name">21. Nicht steuerbare sonstige Leistungen (zeile 34)</field>
                                        <field name="code">DE_21</field>
                                        <field name="sequence">300</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_21_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">21</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_45" model="account.report.line">
                                        <field name="name">45. Übrige nicht steuerbare Umsätze (zeile 35)</field>
                                        <field name="code">DE_45_BASE</field>
                                        <field name="sequence">310</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_45_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">45_BASE</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_de_tag_02" model="account.report.line">
                <field name="name">Steuer</field>
                <field name="sequence">320</field>
                <field name="aggregation_formula">I_ANMELDUNG_DER_UMSATZSTEUERVORAUSZAHLUNG_ZEILE_17_STEUER.balance</field>
                <field name="children_ids">
                    <record id="tax_report_de_tax_tag_17" model="account.report.line">
                        <field name="name">Anmeldung der Umsatzsteuer-Vorauszahlung</field>
                        <field name="code">I_ANMELDUNG_DER_UMSATZSTEUERVORAUSZAHLUNG_ZEILE_17_STEUER</field>
                        <field name="sequence">330</field>
                        <field name="aggregation_formula">LIEFERUNGEN_UND_SONSTIGE_LEISTUNGEN_ZEILE_18_STEUER.balance + LEISTUNGSEMPFANGER_ALS_STEUERSCHULDNER_ZEILE_46.balance + ABZIEHBARE_VORSTEUERBETRAGE_ZEILE_55.balance + ANDERE_STEUERBETRAGE_ZEILE_64.balance + DE_39.balance + DE_83.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_de_tax_tag_18" model="account.report.line">
                                <field name="name">Lieferungen und sonstige Leistungen</field>
                                <field name="code">LIEFERUNGEN_UND_SONSTIGE_LEISTUNGEN_ZEILE_18_STEUER</field>
                                <field name="sequence">340</field>
                                <field name="aggregation_formula">STEUERPFLICHT_UMSATZE_STEUER.balance + INNERGEMEINSCHAFTLICHE_ERWERBE_ZEILE_31_STEUER.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_de_tax_tag_19" model="account.report.line">
                                        <field name="name">Steuerpflichtige Umsätze</field>
                                        <field name="code">STEUERPFLICHT_UMSATZE_STEUER</field>
                                        <field name="sequence">350</field>
                                        <field name="aggregation_formula">DE_81_TAX.balance + DE_86_TAX.balance + DE_36.balance + DE_80.balance</field>
                                        <field name="children_ids">
                                            <record id="tax_report_de_tag_26" model="account.report.line">
                                                <field name="name">81. zum Steuersatz von 19 % (zeile 12)</field>
                                                <field name="code">DE_81_TAX</field>
                                                <field name="sequence">360</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_26_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">81_TAX</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_27" model="account.report.line">
                                                <field name="name">86. zum Steuersatz von 7 % (zeile 13)</field>
                                                <field name="code">DE_86_TAX</field>
                                                <field name="sequence">370</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_27_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">86_TAX</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_36" model="account.report.line">
                                                <field name="name">36. zu anderen Steuersatzen (zeile 15)</field>
                                                <field name="code">DE_36</field>
                                                <field name="sequence">380</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_36_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">36</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_80" model="account.report.line">
                                                <field name="name">80. Umsatze, fur die eine Steuer nach § 24 UStG zu entrichten ist (zeile 17)</field>
                                                <field name="code">DE_80</field>
                                                <field name="sequence">390</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_80_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">80</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tax_tag_31" model="account.report.line">
                                        <field name="name">Innergemeinschaftliche Erwerbe</field>
                                        <field name="code">INNERGEMEINSCHAFTLICHE_ERWERBE_ZEILE_31_STEUER</field>
                                        <field name="sequence">400</field>
                                        <field name="aggregation_formula">DE_89_TAX.balance + DE_93_TAX.balance + DE_98.balance + DE_96.balance</field>
                                        <field name="children_ids">
                                            <record id="tax_report_de_tag_33" model="account.report.line">
                                                <field name="name">89. zum Steuersatz von 19 % (zeile 24)</field>
                                                <field name="code">DE_89_TAX</field>
                                                <field name="sequence">410</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_33_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">89_TAX</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_34" model="account.report.line">
                                                <field name="name">93. zum Steuersatz von 7 % (zeile 25)</field>
                                                <field name="code">DE_93_TAX</field>
                                                <field name="sequence">420</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_34_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">93_TAX</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_98" model="account.report.line">
                                                <field name="name">98. zu anderen Steuersatzen (zeile 27)</field>
                                                <field name="code">DE_98</field>
                                                <field name="sequence">430</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_98_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">98</field>
                                                    </record>
                                                </field>
                                            </record>
                                            <record id="tax_report_de_tag_96" model="account.report.line">
                                                <field name="name">96. neuer Fahrzeuge von Lieferern ohne USt-IdNr. zum allgemeinen Steuersatz (zeile 28)</field>
                                                <field name="code">DE_96</field>
                                                <field name="sequence">440</field>
                                                <field name="expression_ids">
                                                    <record id="tax_report_de_tag_96_tag" model="account.report.expression">
                                                        <field name="label">balance</field>
                                                        <field name="engine">tax_tags</field>
                                                        <field name="formula">96</field>
                                                    </record>
                                                </field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_de_tax_tag_46" model="account.report.line">
                                <field name="name">Leistungsempfanger als Steuerschuldner</field>
                                <field name="code">LEISTUNGSEMPFANGER_ALS_STEUERSCHULDNER_ZEILE_46</field>
                                <field name="sequence">450</field>
                                <field name="aggregation_formula">DE_47.balance + DE_74.balance + DE_85.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_de_tag_47" model="account.report.line">
                                        <field name="name">47. Steuerpflichtige sonstige Leistungen eines im übrigen Gemeinschaftsgebietansässigen Unternehmers (zeile 29)</field>
                                        <field name="code">DE_47</field>
                                        <field name="sequence">460</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_47_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">47</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_74" model="account.report.line">
                                        <field name="name">74. Lieferungen sicherungsübereigneter Gegenstände und Umsätze, die unter das GrEStG fallen (zeile 30)</field>
                                        <field name="code">DE_74</field>
                                        <field name="sequence">470</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_74_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">74</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_85" model="account.report.line">
                                        <field name="name">85. Andere Leistungen (zeile 31)</field>
                                        <field name="code">DE_85</field>
                                        <field name="sequence">480</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_85_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">85</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_de_tax_tag_55" model="account.report.line">
                                <field name="name">Abziehbare Vorsteuerbetrage</field>
                                <field name="code">ABZIEHBARE_VORSTEUERBETRAGE_ZEILE_55</field>
                                <field name="sequence">490</field>
                                <field name="aggregation_formula">DE_66.balance + DE_61.balance + DE_62.balance + DE_67.balance + DE_63.balance + DE_64.balance + DE_59.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_de_tag_66" model="account.report.line">
                                        <field name="name">66. Vorsteuerbeträge aus Rechnungen von anderen Unternehmern (zeile 37)</field>
                                        <field name="code">DE_66</field>
                                        <field name="sequence">500</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_66_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">66</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_61" model="account.report.line">
                                        <field name="name">61. Vorsteuerbeträge aus dem innergemeinschaftlichen Erwerb von Gegenständen (zeile 38)</field>
                                        <field name="code">DE_61</field>
                                        <field name="sequence">510</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_61_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">61</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_62" model="account.report.line">
                                        <field name="name">62. Entstandene Einfuhrumsatzsteuer (zeile 39)</field>
                                        <field name="code">DE_62</field>
                                        <field name="sequence">520</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_62_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">62</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_67" model="account.report.line">
                                        <field name="name">67. Vorsteuerbeträge aus Leistungen im Sinne des § 13b UStG (zeile 40)</field>
                                        <field name="code">DE_67</field>
                                        <field name="sequence">530</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_67_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">67</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_63" model="account.report.line">
                                        <field name="name">63. Vorsteuerbeträge, die nach allgemeinen Durchschnittssätzen berechnet sind (zeile 41)</field>
                                        <field name="code">DE_63</field>
                                        <field name="sequence">540</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_63_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">63</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_59" model="account.report.line">
                                        <field name="name">59. Vorsteuerabzug für innergemeinschaftliche Lieferungen neuer Fahrzeuge außerhalb eines Unternehmens (zeile 42)</field>
                                        <field name="code">DE_59</field>
                                        <field name="sequence">550</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_59_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">59</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_64" model="account.report.line">
                                        <field name="name">64. Berichtigung des Vorsteuerabzugs (zeile 43)</field>
                                        <field name="code">DE_64</field>
                                        <field name="sequence">560</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_64_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">64</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_de_tax_tag_64" model="account.report.line">
                                <field name="name">Andere Steuerbetrage</field>
                                <field name="code">ANDERE_STEUERBETRAGE_ZEILE_64</field>
                                <field name="sequence">570</field>
                                <field name="aggregation_formula">DE_65.balance + DE_69.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_de_tag_65" model="account.report.line">
                                        <field name="name">65. Steuer infolge Wechsels der Besteuerungsform sowie Nachsteuer auf versteuerte Anzahlungen u. ä. wegen Steuersatzänderung (zeile 45)</field>
                                        <field name="code">DE_65</field>
                                        <field name="sequence">580</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_65_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">65</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_de_tag_69" model="account.report.line">
                                        <field name="name">69. In Rechnungen unrichtig oder unberechtigt ausgewiesene Steuerbeträge (zeile 46)</field>
                                        <field name="code">DE_69</field>
                                        <field name="sequence">590</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_de_tag_69_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">69</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_de_tag_39" model="account.report.line">
                                <field name="name">39. Abzug der festgesetzten Sondervorauszahlung für Dauerfristverlängerung (zeile 48)</field>
                                <field name="code">DE_39</field>
                                <field name="sequence">600</field>
                                <field name="expression_ids">
                                    <record id="tax_report_de_tag_39_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">39</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_de_tag_83" model="account.report.line">
                                <field name="name">83. Verbleibende Umsatzsteuer-Vorauszahlung (zeile 49)</field>
                                <field name="code">DE_83</field>
                                <field name="sequence">610</field>
                                <field name="expression_ids">
                                    <record id="tax_report_de_tag_83_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">83</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_de_tag_71" model="account.report.line">
                <field name="name">Minderung</field>
                <field name="sequence">620</field>
                <field name="children_ids">
                    <record id="tax_report_de_tag_50" model="account.report.line">
                        <field name="name">50. Minderung der Bemessungsgrundlage (zeile 50)</field>
                        <field name="code">50</field>
                        <field name="sequence">630</field>
                        <field name="expression_ids">
                            <record id="tax_report_de_tag_50_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">50</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_de_tag_37_74" model="account.report.line">
                        <field name="name">37. Minderung der abziehbaren Vorsteuerbeträge (zeile 51)</field>
                        <field name="code">37</field>
                        <field name="sequence">640</field>
                        <field name="expression_ids">
                            <record id="tax_report_de_tag_37_74_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">37</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
    <record id="tag_de_intracom_community_delivery" model="account.account.tag">
        <field name="name">Innergemeinschaftliche Lieferung</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.de"/>
    </record>
    <record id="tag_de_intracom_community_supplies" model="account.account.tag">
        <field name="name">Sonstige Leistungen</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.de"/>
    </record>
    <record id="tag_de_intracom_ABC" model="account.account.tag">
        <field name="name">Dreiecksgeschäfte</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.de"/>
    </record>
    <record id="tag_de_pl_01" model="account.account.tag">
        <field name="name">G&amp;V: 1-Umsatzerlöse</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_02" model="account.account.tag">
        <field name="name">G&amp;V: 2-Erhöhung oder Verminderung des Bestands an fertigen und unfertigen Erzeugnissen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_03" model="account.account.tag">
        <field name="name">G&amp;V: 3-Andere aktivierte Eigenleistungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_04" model="account.account.tag">
        <field name="name">G&amp;V: 4-Sonstige betrieliche Erträge</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_05" model="account.account.tag">
        <field name="name">G&amp;V: 5-Materialaufwand</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_06" model="account.account.tag">
        <field name="name">G&amp;V: 6-Personalaufwand</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_07" model="account.account.tag">
        <field name="name">G&amp;V: 7-Abschreibungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_08_1" model="account.account.tag">
        <field name="name">G&amp;V: 8.1-Raumkosten</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_08_2" model="account.account.tag">
        <field name="name">G&amp;V: 8.2-Versicherungen, Beiträge und Abgaben</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_08_3" model="account.account.tag">
        <field name="name">G&amp;V: 8.3-Reparaturen und Instandhaltungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_08_4" model="account.account.tag">
        <field name="name">G&amp;V: 8.4-Fahrzeugkosten</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_08_5" model="account.account.tag">
        <field name="name">G&amp;V: 8.5-Werbe- und Reisekosten</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_08_6" model="account.account.tag">
        <field name="name">G&amp;V: 8.6-Kosten der Warenabgabe</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_08_7" model="account.account.tag">
        <field name="name">G&amp;V: 8.7-verschiedene betriebliche Kosten</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_09" model="account.account.tag">
        <field name="name">G&amp;V: 9-Erträge aus Beteiligungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_10" model="account.account.tag">
        <field name="name">G&amp;V: 10-Erträge aus anderen Wertpapieren und Ausleihungen des Finanzanlagevermögens</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_11" model="account.account.tag">
        <field name="name">G&amp;V: 11-Sonstige Zinsen und ähnliche Erträge</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_12" model="account.account.tag">
        <field name="name">G&amp;V: 12-Abschreibungen auf Finanzanlagen und auf Wertpapiere des Umlaufvermögens</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_13" model="account.account.tag">
        <field name="name">G&amp;V: 13-Zinsen und ähnliche Aufwendungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_14" model="account.account.tag">
        <field name="name">G&amp;V: 14-Steuern vom Einkommen und Ertrag</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_pl_15" model="account.account.tag">
        <field name="name">G&amp;V: 15-Sonstige Steuern</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_A_I_1" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: A I 1-Selbst geschaffene gewerbliche Schutzrechte und ähnliche Rechte und Werte</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_A_I_2" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: A I 2-Konzessionen, Lizenzen und ähnliche Rechte und Werte</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_A_I_3" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: A I 3-Geschäfts- oder Firmenwert</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_A_I_4" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: A I 4-geleistete Anzahlungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_A_II_1" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: A II 1-Grundstücke. grundstücksgleiche Rechte und Bauten</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_A_II_2" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: A II 2-Technische  Anlagen und Maschinen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_A_II_3" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: A II 3-Andere Anlagen. Betriebs- und Geschäftsausstattung</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_A_II_4" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: A II 4-Geleistete Anzahlungen und Anlagen im Bau</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_A_III_1" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: A III 1-Anteile an verbundenen Unternehmen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_A_III_2" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: A III 2-Ausleihungen an verbundene Unternehmen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_A_III_3" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: A III 3-Beteiligungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_A_III_4" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: A III 4-Ausleihungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_A_III_5" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: A III 5-Wertpapiere des Anlagevermögens</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_A_III_6" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: A III 6-sonstige Ausleihungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_B_I_1" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: B I 1-Roh-, Hilfs- und Betriebsstoffe</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_B_I_2" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: B I 2-Unfertige Erzeugnisse, unfertige Leistungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_B_I_3" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: B I 3-Fertige Erzeugnisse und Waren</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_B_I_4" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: B I 4-Geleistete Anzahlungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_B_II_1" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: B II 1-Forderungen aus Lieferungen und Leistungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_B_II_2" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: B II 2-Forderungen gegen verbundene Unternehmen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_B_II_3" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: B II 3-Forderungen gegen Unternehmen, mit denen ein Beteiligungsverhältnis besteht</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_B_II_4" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: B II 4-Sonstige Vermögensgegenstände</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_B_III_1" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: B III 1-Anteile an verbundenen Unternehmen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_B_III_2" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: B III 2-sonstige Wertpapiere</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_B_IV" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: B IV-Kassenbestand, Bundesbankguthaben, Guthaben bei Kreditinstituten und Schecks</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_C" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: C-Rechnungsabgrenzungsposten</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_D" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: D-Aktive latente Steuern</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_asset_bs_E" model="account.account.tag">
        <field name="name">Bilanz-Aktiva: E-Aktiver Unterschiedsbetrag aus der Vermögensverrechnung</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_A_I" model="account.account.tag">
        <field name="name">Bilanz-Passiva: A I-Gezeichnetes Kapital</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_A_II" model="account.account.tag">
        <field name="name">Bilanz-Passiva: A II-Kapitalrücklage</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_A_III_1" model="account.account.tag">
        <field name="name">Bilanz-Passiva: A III 1-Gesetzliche Rücklage</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_A_III_2" model="account.account.tag">
        <field name="name">Bilanz-Passiva: A III 2-Rücklage für Anteile an einem herrschenden oder mehrheitlich beteiligten Unternehmen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_A_III_3" model="account.account.tag">
        <field name="name">Bilanz-Passiva: A III 3-Satzungsmäßige Rücklagen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_A_III_4" model="account.account.tag">
        <field name="name">Bilanz-Passiva: A III 4-Andere Gewinnrücklagen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_A_IV" model="account.account.tag">
        <field name="name">Bilanz-Passiva: A IV-Gewinnvortrag/Verlustvortrag</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_A_V" model="account.account.tag">
        <field name="name">Bilanz-Passiva: A V-Jahresüberschuß/Jahresfehlbetrag</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_B_1" model="account.account.tag">
        <field name="name">Bilanz-Passiva: B 1-Rückstellungen für Pensionen und ähnliche Verpflichtungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_B_2" model="account.account.tag">
        <field name="name">Bilanz-Passiva: B 2-Steuerrückstellungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_B_3" model="account.account.tag">
        <field name="name">Bilanz-Passiva: B 3-Sonstige Rückstellungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_C_1" model="account.account.tag">
        <field name="name">Bilanz-Passiva: C 1-Anleihen, davon konvertibeln</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_C_2" model="account.account.tag">
        <field name="name">Bilanz-Passiva: C 2-Verbindlichkeiten gegenüber Kreditinstituten</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_C_3" model="account.account.tag">
        <field name="name">Bilanz-Passiva: C 3-Erhaltene Anzahlungen auf Bestellungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_C_4" model="account.account.tag">
        <field name="name">Bilanz-Passiva: C 4-Verbindlichkeiten aus Lieferungen und Leistungen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_C_5" model="account.account.tag">
        <field name="name">Bilanz-Passiva: C 5-Verbindlichkeiten aus der Annahme gezogener Wechsel und der Ausstellung eigener Wechsel</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_C_6" model="account.account.tag">
        <field name="name">Bilanz-Passiva: C 6-Verbindlichkeiten gegenüber verbundenen Unternehmen</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_C_7" model="account.account.tag">
        <field name="name">Bilanz-Passiva: C 7-Verbindlichkeiten gegenüber Unternehmen, mit denen ein Beteiligungsverhältnis besteht</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_C_8" model="account.account.tag">
        <field name="name">Bilanz-Passiva: C 8-Sonstige Verbindlichkeiten, davon aus Steuern, davon im Rahmen der sozialen Sicherheit</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_D" model="account.account.tag">
        <field name="name">Bilanz-Passiva: D-Rechnungsabgrenzungsposten</field>
        <field name="applicability">accounts</field>
    </record>
    <record id="tag_de_liabilities_bs_E" model="account.account.tag">
        <field name="name">Bilanz-Passiva: E-Passive latente Steuern</field>
        <field name="applicability">accounts</field>
    </record>
</odoo>

```

## File: migrations\1.1\post-migrate_update_amls.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details
from odoo import api, SUPERUSER_ID


def migrate(cr, version):
    # The tax report line 68 has been removed as it does not appear in tax report anymore.
    # But, it was referenced in the account.sales.report
    # So, we update amls of this line only, to make this report consistent.

    env = api.Environment(cr, SUPERUSER_ID, {})
    country = env['res.country'].search([('code', '=', 'DE')], limit=1)
    tags_68 = env['account.account.tag']._get_tax_tags('68', country.id)
    tags_60 = env['account.account.tag']._get_tax_tags('60', country.id)

    if tags_68.filtered(lambda tag: tag.tax_negate):
        cr.execute(
            """
            UPDATE account_account_tag_account_move_line_rel
               SET account_account_tag_id = %s
             WHERE account_account_tag_id IN %s;
            """,
            [
                tags_60.filtered(lambda tag: tag.tax_negate)[0].id,
                tuple(tags_68.filtered(lambda tag: tag.tax_negate).ids)
            ]
        )

    if tags_68.filtered(lambda tag: not tag.tax_negate):
        cr.execute(
            """
            UPDATE account_account_tag_account_move_line_rel
               SET account_account_tag_id = %s
             WHERE account_account_tag_id IN %s;
            """,
            [
                tags_60.filtered(lambda tag: not tag.tax_negate)[0].id,
                tuple(tags_68.filtered(lambda tag: not tag.tax_negate).ids)
            ]
        )

    cr.execute(
        r"""
        UPDATE account_move_line
           SET tax_audit = REGEXP_REPLACE(tax_audit, '(?<=(^|\s))68:', '60:')
          FROM (
              SELECT aml.id as aml_id
                FROM account_move_line aml
                JOIN account_account_tag_account_move_line_rel aml_tag_rel ON aml_tag_rel.account_move_line_id = aml.id
               WHERE aml_tag_rel.account_account_tag_id IN %s
               ) aml
         WHERE id = aml.aml_id
        """, [tuple(tags_60.ids)]
    )

```

## File: migrations\2.0\pre-migrate.py

```python
# -*- coding: utf-8 -*-


def rename_tag(cr, old_tag, new_tag):
    cr.execute(
        """UPDATE ir_model_data
           SET name=%s
           WHERE module='l10n_de' AND name=%s
        """,
        (new_tag, old_tag),
    )


def migrate(cr, version):
    # By deleting tag B from ir_model_data we ensure that the ORM won't try to remove this record.
    # This is done because the tag might be already used as a FK somewhere else.
    cr.execute(
        """DELETE FROM ir_model_data
           WHERE module='l10n_de'
           AND name='tag_de_liabilities_bs_B'
        """
    )

    # As some people already upgraded, they will have renamed the C_1 tag to B_1. This doesn't come from the script but from the
    # account_account_tags_data.xml. If they try to run this script now in the fix they get the error that B_1 already exists.
    # To fix this we can check if it exists or not, and if it does then we don't run the script. This means that the ones
    # that upgraded won't have the old tags data transferred to the new tags but they will still be able to have the updated sheet.
    cr.execute(
        """SELECT 1 FROM ir_model_data
           WHERE module='l10n_de' AND name='tag_de_liabilities_bs_B_1'
        """)
    if cr.rowcount:
        # If the script didn't run, we should remove the tags that have been replaced from ir_model_data too so they're
        # not deleted by the ORM if they were already used.
        cr.execute(
            """DELETE FROM ir_model_data
               WHERE module='l10n_de'
               AND name IN ('tag_de_liabilities_bs_F', 'tag_de_liabilities_bs_D_1', 'tag_de_liabilities_bs_D_2',
               'tag_de_liabilities_bs_D_3', 'tag_de_liabilities_bs_D_4', 'tag_de_liabilities_bs_D_5',
               'tag_de_liabilities_bs_D_6', 'tag_de_liabilities_bs_D_7', 'tag_de_liabilities_bs_D_8')
            """
        )
        return
    rename_tag(cr, "tag_de_liabilities_bs_C_1", "tag_de_liabilities_bs_B_1")
    rename_tag(cr, "tag_de_liabilities_bs_C_2", "tag_de_liabilities_bs_B_2")
    rename_tag(cr, "tag_de_liabilities_bs_C_3", "tag_de_liabilities_bs_B_3")
    rename_tag(cr, "tag_de_liabilities_bs_D_1", "tag_de_liabilities_bs_C_1")
    rename_tag(cr, "tag_de_liabilities_bs_D_2", "tag_de_liabilities_bs_C_2")
    rename_tag(cr, "tag_de_liabilities_bs_D_3", "tag_de_liabilities_bs_C_3")
    rename_tag(cr, "tag_de_liabilities_bs_D_4", "tag_de_liabilities_bs_C_4")
    rename_tag(cr, "tag_de_liabilities_bs_D_5", "tag_de_liabilities_bs_C_5")
    rename_tag(cr, "tag_de_liabilities_bs_D_6", "tag_de_liabilities_bs_C_6")
    rename_tag(cr, "tag_de_liabilities_bs_D_7", "tag_de_liabilities_bs_C_7")
    rename_tag(cr, "tag_de_liabilities_bs_D_8", "tag_de_liabilities_bs_C_8")
    rename_tag(cr, "tag_de_liabilities_bs_E", "tag_de_liabilities_bs_D")
    rename_tag(cr, "tag_de_liabilities_bs_F", "tag_de_liabilities_bs_E")

```

## File: models\account_journal.py

```python
# -*- coding: utf-8 -*-

from odoo import api, models

class AccountJournal(models.Model):
    _inherit = "account.journal"

    @api.model
    def _prepare_liquidity_account_vals(self, company, code, vals):
        res = super()._prepare_liquidity_account_vals(company, code, vals)

        if company.account_fiscal_country_id.code == 'DE':
            tag_ids = res.get('tag_ids', [])
            tag_ids.append((4, self.env.ref('l10n_de.tag_de_asset_bs_B_IV').id))
            res['tag_ids'] = tag_ids

        return res

```

## File: models\chart_template.py

```python
# -*- coding: utf-8 -*-
from odoo import models, Command, _


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    # Write paperformat and report template used on company
    def _load(self, company):
        res = super(AccountChartTemplate, self)._load(company)
        if self in [
            self.env.ref('l10n_de_skr03.l10n_de_chart_template', raise_if_not_found=False),
            self.env.ref('l10n_de_skr04.l10n_chart_de_skr04', raise_if_not_found=False)
        ]:
            company.write({
                'external_report_layout_id': self.env.ref('l10n_din5008.external_layout_din5008').id,
                'paperformat_id': self.env.ref('l10n_din5008.paperformat_euro_din').id
            })

            outstanding_receipt = company.account_journal_payment_debit_account_id
            outstanding_payment = company.account_journal_payment_credit_account_id

            asset_tag = self.env.ref('l10n_de.tag_de_asset_bs_B_II_4')
            outstanding_receipt['tag_ids'] += asset_tag
            outstanding_payment['tag_ids'] += asset_tag

        return res

    def _prepare_transfer_account_template(self):
        res = super(AccountChartTemplate, self)._prepare_transfer_account_template(None)
        if self in [
            self.env.ref('l10n_de_skr03.l10n_de_chart_template', raise_if_not_found=False),
            self.env.ref('l10n_de_skr04.l10n_chart_de_skr04', raise_if_not_found=False)
        ]:
            tag_ids = res.get('tag_ids', [])
            tag_ids += [Command.link(self.env.ref('l10n_de.tag_de_asset_bs_B_II_4').id)]
            res['tag_ids'] = tag_ids

        return res

    def _create_liquidity_journal_suspense_account(self, company, code_digits):
        if self not in [
            self.env.ref('l10n_de_skr03.l10n_de_chart_template', raise_if_not_found=False),
            self.env.ref('l10n_de_skr04.l10n_chart_de_skr04', raise_if_not_found=False)
        ]:
            return super()._create_liquidity_journal_suspense_account(company, code_digits)
        return self.env['account.account'].create({
            'name': _("Bank Suspense Account"),
            'code': self.env['account.account']._search_new_account_code(company, code_digits, company.bank_account_code_prefix or ''),
            'account_type': 'asset_current',
            'company_id': company.id,
            'tag_ids': self.env.ref('l10n_de.tag_de_asset_bs_B_IV')
        })

```

## File: models\datev.py

```python
from odoo import fields, models

class AccountTaxTemplate(models.Model):
    _inherit = 'account.tax.template'

    l10n_de_datev_code = fields.Char(size=4)

    def _get_tax_vals(self, company, tax_template_to_tax):
        vals = super(AccountTaxTemplate, self)._get_tax_vals(company, tax_template_to_tax)
        vals['l10n_de_datev_code'] = self.l10n_de_datev_code
        return vals

class AccountTax(models.Model):
    _inherit = "account.tax"

    l10n_de_datev_code = fields.Char(size=4, help="4 digits code use by Datev")


class ProductTemplate(models.Model):
    _inherit = "product.template"

    def _get_product_accounts(self):
        """ As taxes with a different rate need a different income/expense account, we add this logic in case people only use
         invoicing to not be blocked by the above constraint"""
        result = super(ProductTemplate, self)._get_product_accounts()
        company = self.env.company
        if company.account_fiscal_country_id.code == "DE":
            if not self.property_account_income_id:
                taxes = self.taxes_id.filtered(lambda t: t.company_id == company)
                if not result['income'] or (result['income'].tax_ids and taxes and taxes[0] not in result['income'].tax_ids):
                    result_income = self.env['account.account'].search([('internal_group', '=', 'income'), ('deprecated', '=', False),
                                                                   ('tax_ids', 'in', taxes.ids)], limit=1)
                    result['income'] = result_income or result['income']
            if not self.property_account_expense_id:
                supplier_taxes = self.supplier_taxes_id.filtered(lambda t: t.company_id == company)
                if not result['expense'] or (result['expense'].tax_ids and supplier_taxes and supplier_taxes[0] not in result['expense'].tax_ids):
                    result_expense = self.env['account.account'].search([('internal_group', '=', 'expense'), ('deprecated', '=', False),
                                                                   ('tax_ids', 'in', supplier_taxes.ids)], limit=1)
                    result['expense'] = result_expense or result['expense']
        return result

```

## File: models\ir_actions_report.py

```python
from odoo import models


class IrActionsReport(models.Model):
    _inherit = 'ir.actions.report'

    def _get_rendering_context(self, report, docids, data):
        data = super()._get_rendering_context(report, docids, data)
        data['din_header_spacing'] = report.get_paperformat().header_spacing
        return data

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import models, fields, api, _
from odoo.exceptions import ValidationError
import stdnum.de.stnr
import stdnum.exceptions


class ResCompany(models.Model):
    _inherit = 'res.company'

    l10n_de_stnr = fields.Char(string="St.-Nr.", help="Steuernummer. Scheme: ??FF0BBBUUUUP, e.g.: 2893081508152 https://de.wikipedia.org/wiki/Steuernummer")
    l10n_de_widnr = fields.Char(string="W-IdNr.", help="Wirtschafts-Identifikationsnummer.")

    @api.depends('country_code')
    @api.constrains('state_id', 'l10n_de_stnr')
    def _validate_l10n_de_stnr(self):
        for record in self:
            record.get_l10n_de_stnr_national()

    def get_l10n_de_stnr_national(self):
        self.ensure_one()
        national_steuer_nummer = None

        if self.l10n_de_stnr and self.country_code == 'DE':
            try:
                national_steuer_nummer = stdnum.de.stnr.to_country_number(self.l10n_de_stnr, self.state_id.name)
            except stdnum.exceptions.InvalidComponent:
                raise ValidationError(_("Your company's SteuerNummer is not compatible with your state"))
            except stdnum.exceptions.InvalidFormat:
                if stdnum.de.stnr.is_valid(self.l10n_de_stnr, self.state_id.name):
                    national_steuer_nummer = self.l10n_de_stnr
                else:
                    raise ValidationError(_("Your company's SteuerNummer is not valid"))

        elif self.l10n_de_stnr:
            national_steuer_nummer = self.l10n_de_stnr

        return national_steuer_nummer

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import account_journal
from . import datev
from . import chart_template
from . import ir_actions_report
from . import res_company

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="7.22" width="50.4" height="32.76" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.5" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="1920" height="1152" transform="translate(4.8 7.22) scale(0.03 0.03)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAB4AAAATgCAYAAAAWvl8BAAAACXBIWXMAAaXYAAGl2AHj38NIAAAgAElEQVR4XuzbQQ2AQBAEwYXgXywGQMK9t1P1HgeduWbmGwAAAAAAAADWu08DAAAAAAAAAHYQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACAiOc9LQAAAAAAAABYwQMYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAAAAIEIABgAAAAAAAIgQgAEAAAAAAAAiBGAAAAAAAACACAEYAAAAAICffTuoAQAEYiAI/sUh6ZDAl2xm3nWwKQAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABF7zprXCAAAAAAAAID/eQADAAAAAAAARAjAAEFtkKQAAA/CSURBVAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAAAAAECEAAwAAAAAAAAQIQADAAAAAAAARAjAAAAAAAAAABECMAAAAADc9uxABgAAAGCQv/U9vtIIAAAmBDAAAAAAAADAhAAGAAAAAAAAmBDAAAAAAAAAABMCGAAAAAAAAGBCAAMAAAAAAABMCGAAAAAAAACACQEMAAAAAAAAMCGAAQAAAAAAACYEMAAAAAAAAMCEAAYAAAAAAACYEMAAAAAAAAAAEwIYAAAAAAAAYEIAAwAAAAAAAEwIYAAAAAAAAIAJAQwAAAAAAAAwIYABAAAAAAAAJgQwAAAAAAAAwIQABgAAAAAAAJgQwAAAAAAAAAATAhgAAAAAAABgQgADAAAAAAAATAhgAAAAAAAAgAkBDAAAAAAAADAhgAEAAAAAAAAmBDAAAAAAAADAhAAGAAAAAAAAmBDAAAAAAAAAABMCGAAAAAAAAGBCAAMAAAAAAABMCGAAAAAAAACACQEMAAAAAAAAMCGAAQAAAAAAACYEMAAAAAAAAMCEAAYAAAAAAACYEMAAAAAAAAAAEwIYAAAAAAAAYEIAAwAAAAAAAEwIYAAAAAAAAIAJAQwAAAAAAAAwIYABAAAAAAAAJgQwAAAAAAAAwIQABgAAAAAAAJgQwAAAAAAAAAATAhgAAAAAAABgQgADAAAAAAAATAhgAAAAAAAAgAkBDAAAAAAAADAhgAEAAAAAAAAmBDAAAAAAAADAhAAGAAAAAAAAmBDAAAAAAAAAABMCGAAAAAAAAGBCAAMAAAAAAABMCGAAAAAAAACACQEMAAAAAAAAMCGAAQAAAAAAACYEMAAAAAAAAMCEAAYAAAAAAACYEMAAAAAAAAAAEwIYAAAAAAAAYEIAAwAAAAAAAEwIYAAAAAAAAIAJAQwAAAAAAAAwIYABAAAAAAAAJgQwAAAAAAAAwIQABgAAAAAAAJgQwAAAAAAAAAATAhgAAAAAAABgQgADAAAAAAAATAhgAAAAAAAAgAkBDAAAAAAAADAhgAEAAAAAAAAmBDAAAAAAAADAhAAGAAAAAAAAmBDAAAAAAAAAABMCGAAAAAAAAGBCAAMAAAAAAABMCGAAAAAAAACACQEMAAAAAAAAMCGAAQAAAAAAACYEMAAAAAAAAMCEAAYAAAAAAACYEMAAAAAAAAAAEwIYAAAAAAAAYEIAAwAAAAAAAEwIYAAAAAAAAIAJAQwAAAAAAAAwIYABAAAAAAAAJgQwAAAAAAAAwIQABgAAAAAAAJgQwAAAAAAAAAATAhgAAAAAAABgQgADAAAAAAAATAhgAAAAAAAAgAkBDAAAAAAAADAhgAEAAAAAAAAmBDAAAAAAAADAhAAGAAAAAAAAmBDAAAAAAAAAABMCGAAAAAAAAGBCAAMAAAAAAABMCGAAAAAAAACACQEMAAAAAAAAMCGAAQAAAAAAACYEMAAAAAAAAMCEAAYAAAAAAACYEMAAAAAAAAAAEwIYAAAAAAAAYEIAAwAAAAAAAEwIYAAAAAAAAIAJAQwAAAAAAAAwIYABAAAAAAAAJgQwAAAAAAAAwIQABgAAAAAAAJgQwAAAAAAAAAATAhgAAAAAAABgQgADAAAAAAAATAhgAAAAAAAAgAkBDAAAAAAAADAhgAEAAAAAAAAmBDAAAAAAAADAhAAGAAAAAAAAmBDAAAAAAAAAABMCGAAAAAAAAGBCAAMAAAAAAABMCGAAAAAAAACACQEMAAAAAAAAMCGAAQAAAAAAACYEMAAAAAAAAMCEAAYAAAAAAACYEMAAAAAAAAAAEwIYAAAAAAAAYEIAAwAAAAAAAEwIYAAAAAAAAIAJAQwAAAAAAAAwIYABAAAAAAAAJgQwAAAAAAAAwIQABgAAAAAAAJgQwAAAAAAAAAATAhgAAAAAAABgQgADAAAAAAAATAhgAAAAAAAAgAkBDAAAAAAAADAhgAEAAAAAAAAmBDAAAAAAAADAhAAGAAAAAAAAmBDAAAAAAAAAABMCGAAAAAAAAGBCAAMAAAAAAABMCGAAAAAAAACACQEMAAAAAAAAMCGAAQAAAAAAACYEMAAAAAAAAMCEAAYAAAAAAACYEMAAAAAAAAAAEwIYAAAAAAAAYCIKRg5/dFtv5QAAAABJRU5ErkJggg=="/>
    </g>
  </g>
</svg>

```

## File: views\account_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="view_account_tax_form_inherit" model="ir.ui.view">
            <field name="name">account.tax.form</field>
            <field name="model">account.tax</field>
            <field name="inherit_id" ref="account.view_tax_form"/>
            <field name="arch" type="xml">
                <field name="include_base_amount" position="after">
                    <field name="l10n_de_datev_code" attrs="{'invisible': [('country_code', '!=', 'DE')]}"/>
                </field>
            </field>
        </record>
</odoo>
```

## File: views\res_company_views.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
        <record id="res_company_form_l10n_de" model="ir.ui.view">
            <field name="name">res.company.form</field>
            <field name="model">res.company</field>
            <field name="inherit_id" ref="account.view_company_form"/>
            <field name="arch" type="xml">
                <field name="vat" position="after">
                    <field name="l10n_de_stnr" attrs="{'invisible':[('account_enabled_tax_country_ids', 'not in', %(base.de)d)]}"/>
                    <field name="l10n_de_widnr" attrs="{'invisible':[('country_code', '!=', 'DE')]}"/>
                </field>
            </field>
        </record>
</odoo>

```

