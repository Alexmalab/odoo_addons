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
    ],
    'data': [
        'data/account_account_tags_data.xml',
        'data/menuitem_data.xml',
        'views/account_view.xml',
        'views/res_company_views.xml',
        'report/din5008_report.xml',
        'data/report_layout.xml',
    ],
    'assets': {
        'web.report_assets_common': [
            'l10n_de/static/src/**/*',
        ],
    },
    'license': 'LGPL-3',
}

```

## File: data\account_account_tags_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.de"/>
    </record>

    <!-- First level, one for base, one for tax -->
    <record id="tax_report_de_tag_01" model="account.tax.report.line">
        <field name="name">Bemessungsgrundlage</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="formula">None</field>
    </record>
    <record id="tax_report_de_tag_02" model="account.tax.report.line">
        <field name="name">Steuer</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="formula">None</field>
    </record>


    <!-- BASE -->
    <record id="tax_report_de_tag_17" model="account.tax.report.line">
        <field name="name">I. Anmeldung der Umsatzsteuer-Vorauszahlung</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tag_01"/>
        <field name="formula">None</field>
    </record>

    <record id="tax_report_de_tag_18" model="account.tax.report.line">
        <field name="name">Lieferungen und sonstige Leistungen</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tag_17"/>
    </record>

    <record id="tax_report_de_tag_19" model="account.tax.report.line">
        <field name="name">Steuerpflichtige Umsätze</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tag_18"/>
    </record>

    <record id="tax_report_de_tag_81" model="account.tax.report.line">
        <field name="name">81. zum Steuersatz von 19 % (zeile 12)</field>
        <field name="tag_name">81_BASE</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tag_19"/>
        <field name="code">81</field>
    </record>
    <record id="tax_report_de_tag_86" model="account.tax.report.line">
        <field name="name">86. zum Steuersatz von 7 % (zeile 13)</field>
        <field name="tag_name">86_BASE</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tag_19"/>
        <field name="code">86</field>
    </record>
    <record id="tax_report_de_tag_87" model="account.tax.report.line">
        <field name="name">87. zum Steuersatz von 0 % (zeile 14)</field>
        <field name="tag_name">87_BASE</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">30</field>
        <field name="parent_id" ref="tax_report_de_tag_19"/>
        <field name="code">87</field>
    </record>
    <record id="tax_report_de_tag_35" model="account.tax.report.line">
        <field name="name">35. zu anderen Steuersätzen (zeile 15)</field>
        <field name="tag_name">35</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">40</field>
        <field name="parent_id" ref="tax_report_de_tag_19"/>
        <field name="code">35</field>
    </record>
    <record id="tax_report_de_tag_77" model="account.tax.report.line">
        <field name="name">77. Lieferungen land- und forstwirtschaftlicher Betriebe nach § 24 UStG an Abnehmer mit USt-IdNr. (zeile 16)</field>
        <field name="tag_name">77</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">50</field>
        <field name="parent_id" ref="tax_report_de_tag_19"/>
        <field name="code">77</field>
    </record>
    <record id="tax_report_de_tag_76" model="account.tax.report.line">
        <field name="name">76. Umsätze, für die eine Steuer nach § 24 UStG zu entrichten ist (zeile 17)</field>
        <field name="tag_name">76</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">60</field>
        <field name="parent_id" ref="tax_report_de_tag_19"/>
        <field name="code">76</field>
    </record>

    <record id="tax_report_de_tag_25" model="account.tax.report.line">
        <field name="name">Steuerfreie Umsätze mit Vorsteuerabzug</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tag_18"/>
    </record>

    <record id="tax_report_de_tag_41" model="account.tax.report.line">
        <field name="name">41. an Abnehmer mit USt-IdNr (zeile 18)</field>
        <field name="tag_name">41</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tag_25"/>
        <field name="code">41</field>
    </record>
    <record id="tax_report_de_tag_44" model="account.tax.report.line">
        <field name="name">44. neuer Fahrzeuge an Abnehmer ohne USt-IdNr (zeile 19)</field>
        <field name="tag_name">44</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tag_25"/>
        <field name="code">44</field>
    </record>
    <record id="tax_report_de_tag_49" model="account.tax.report.line">
        <field name="name">49. neuer Fahrzeuge außerhalb eines Unternehmens (zeile 20)</field>
        <field name="tag_name">49</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">30</field>
        <field name="parent_id" ref="tax_report_de_tag_25"/>
        <field name="code">49</field>
    </record>
    <record id="tax_report_de_tag_43" model="account.tax.report.line">
        <field name="name">43. Weitere steuerfreie Umsätze mit Vorsteuerabzug (zeile 21)</field>
        <field name="tag_name">43</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">40</field>
        <field name="parent_id" ref="tax_report_de_tag_25"/>
        <field name="code">43</field>
    </record>

    <record id="tax_report_de_tag_24" model="account.tax.report.line">
        <field name="name">48. Steuerfreie Umsätze ohne Vorsteuerabzug (zeile 22)</field>
        <field name="tag_name">48</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">30</field>
        <field name="parent_id" ref="tax_report_de_tag_18"/>
        <field name="code">48</field>
    </record>

    <record id="tax_report_de_tag_31" model="account.tax.report.line">
        <field name="name">Innergemeinschaftliche Erwerbe</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">40</field>
        <field name="parent_id" ref="tax_report_de_tag_18"/>
    </record>

    <record id="tax_report_de_tag_91" model="account.tax.report.line">
        <field name="name">91. Steuerfreie innergemeinschaftliche Erwerbe (zeile 23)</field>
        <field name="tag_name">91</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tag_31"/>
        <field name="code">91</field>
    </record>
    <record id="tax_report_de_tag_89" model="account.tax.report.line">
        <field name="name">89. Steuerpflichtige innergemeinschaftliche Erwerbe zum Steuersatz von 19 % (zeile 24)</field>
        <field name="tag_name">89_BASE</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tag_31"/>
        <field name="code">89</field>
    </record>
    <record id="tax_report_de_tag_93" model="account.tax.report.line">
        <field name="name">93. zum Steuersatz von 7 % (zeile 25)</field>
        <field name="tag_name">93_BASE</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">30</field>
        <field name="parent_id" ref="tax_report_de_tag_31"/>
        <field name="code">93</field>
    </record>
    <record id="tax_report_de_tag_90" model="account.tax.report.line">
        <field name="name">90. zum Steuersatz von 0 % (zeile 26)</field>
        <field name="tag_name">90_BASE</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">40</field>
        <field name="parent_id" ref="tax_report_de_tag_31"/>
        <field name="code">90</field>
    </record>
    <record id="tax_report_de_tag_95" model="account.tax.report.line">
        <field name="name">95. zu anderen Steuersätzen (zeile 27)</field>
        <field name="tag_name">95</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">50</field>
        <field name="parent_id" ref="tax_report_de_tag_31"/>
        <field name="code">95</field>
    </record>
    <record id="tax_report_de_tag_94" model="account.tax.report.line">
        <field name="name">94. neuer Fahrzeuge von Lieferern ohne (zeile 28)</field>
        <field name="tag_name">94</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">60</field>
        <field name="parent_id" ref="tax_report_de_tag_31"/>
        <field name="code">94</field>
    </record>

    <record id="tax_report_de_tag_46" model="account.tax.report.line">
        <field name="name">Leistungsempfänger als Steuerschuldner</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tag_17"/>
    </record>

    <record id="tax_report_de_tag_48" model="account.tax.report.line">
        <field name="name">46. Steuerpflichtige sonstige Leistungen eines im übrigen Gemeinschaftsgebiet ansässigen Unternehmers (zeile 29)</field>
        <field name="tag_name">46</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tag_46"/>
        <field name="code">46</field>
    </record>
    <record id="tax_report_de_tag_73" model="account.tax.report.line">
        <field name="name">73. Lieferungen sicherungsübereigneter Gegenstände und Umsätze, die unter das GrEStG fallen (zeile 30)</field>
        <field name="tag_name">73</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tag_46"/>
        <field name="code">73</field>
    </record>
    <record id="tax_report_de_tag_84" model="account.tax.report.line">
        <field name="name">84. Andere Leistungen (zeile 31)</field>
        <field name="tag_name">84</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">30</field>
        <field name="parent_id" ref="tax_report_de_tag_46"/>
        <field name="code">84</field>
    </record>

    <record id="tax_report_de_tag_37" model="account.tax.report.line">
        <field name="name">Ergänzende Angaben zu Umsätzen</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">30</field>
        <field name="parent_id" ref="tax_report_de_tag_17"/>
    </record>

    <record id="tax_report_de_tag_42" model="account.tax.report.line">
        <field name="name">42. Dreiecksgeschäften (zeile 32)</field>
        <field name="tag_name">42</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tag_37"/>
        <field name="code">42</field>
    </record>
    <record id="tax_report_de_tag_60" model="account.tax.report.line">
        <field name="name">60. Übrige steuerpflichtige Umsätze, für die der Leistungsempfänger die Steuer nach § 13b Abs. 5 UStG schuldet (zeile 33)</field>
        <field name="tag_name">60</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tag_37"/>
        <field name="code">60</field>
    </record>
    <record id="tax_report_de_tag_21" model="account.tax.report.line">
        <field name="name">21. Nicht steuerbare sonstige Leistungen (zeile 34)</field>
        <field name="tag_name">21</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">30</field>
        <field name="parent_id" ref="tax_report_de_tag_37"/>
        <field name="code">21</field>
    </record>
    <record id="tax_report_de_tag_45" model="account.tax.report.line">
        <field name="name">45. Übrige nicht steuerbare Umsätze (zeile 35)</field>
        <field name="tag_name">45_BASE</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">40</field>
        <field name="parent_id" ref="tax_report_de_tag_37"/>
        <field name="code">45</field>
    </record>

    <!-- TAX -->

    <record id="tax_report_de_tax_tag_17" model="account.tax.report.line">
        <field name="name">Anmeldung der Umsatzsteuer-Vorauszahlung</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tag_02"/>
        <field name="code"></field>
    </record>

    <record id="tax_report_de_tax_tag_18" model="account.tax.report.line">
        <field name="name">Lieferungen und sonstige Leistungen</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_17"/>
    </record>

    <record id="tax_report_de_tax_tag_19" model="account.tax.report.line">
        <field name="name">Steuerpflichtige Umsätze</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_18"/>
    </record>
    <!-- Row 20 - 21 - 22 - 24 -->
    <record id="tax_report_de_tag_26" model="account.tax.report.line">
        <field name="name">zum Steuersatz von 19 % (zeile 12)</field>
        <field name="tag_name">81_TAX</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_19"/>
        <field name="code"></field>
    </record>
    <record id="tax_report_de_tag_27" model="account.tax.report.line">
        <field name="name">zum Steuersatz von 7 % (zeile 13)</field>
        <field name="tag_name">86_TAX</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_19"/>
        <field name="code"></field>
    </record>
    <record id="tax_report_de_tag_36" model="account.tax.report.line">
        <field name="name">36. zu anderen Steuersatzen (zeile 15)</field>
        <field name="tag_name">36</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">30</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_19"/>
        <field name="code">36</field>
    </record>
    <record id="tax_report_de_tag_80" model="account.tax.report.line">
        <field name="name">80. Umsatze, fur die eine Steuer nach § 24 UStG zu entrichten ist (zeile 17)</field>
        <field name="tag_name">80</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">40</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_19"/>
        <field name="code">80</field>
    </record>

    <record id="tax_report_de_tax_tag_31" model="account.tax.report.line">
        <field name="name">Innergemeinschaftliche Erwerbe</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_18"/>
    </record>

    <record id="tax_report_de_tag_33" model="account.tax.report.line">
        <field name="name">89. zum Steuersatz von 19 % (zeile 24)</field>
        <field name="tag_name">89_TAX</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_31"/>
        <field name="code"></field>
    </record>
    <record id="tax_report_de_tag_34" model="account.tax.report.line">
        <field name="name">93. zum Steuersatz von 7 % (zeile 25)</field>
        <field name="tag_name">93_TAX</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_31"/>
        <field name="code"></field>
    </record>
    <record id="tax_report_de_tag_98" model="account.tax.report.line">
        <field name="name">98. zu anderen Steuersatzen (zeile 27)</field>
        <field name="tag_name">98</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">30</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_31"/>
        <field name="code">98</field>
    </record>
    <record id="tax_report_de_tag_96" model="account.tax.report.line">
        <field name="name">96. neuer Fahrzeuge von Lieferern ohne USt-IdNr. zum allgemeinen Steuersatz (zeile 28)</field>
        <field name="tag_name">96</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">40</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_31"/>
        <field name="code">96</field>
    </record>

    <record id="tax_report_de_tax_tag_46" model="account.tax.report.line">
        <field name="name">Leistungsempfänger als Steuerschuldner</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_17"/>
    </record>

    <record id="tax_report_de_tag_47" model="account.tax.report.line">
        <field name="name">47. Steuerpflichtige sonstige Leistungen eines im übrigen Gemeinschaftsgebietansässigen Unternehmers (zeile 29)</field>
        <field name="tag_name">47</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_46"/>
        <field name="code">47</field>
    </record>
    <record id="tax_report_de_tag_74" model="account.tax.report.line">
        <field name="name">74. Lieferungen sicherungsübereigneter Gegenstände und Umsätze, die unter das GrEStG fallen (zeile 30)</field>
        <field name="tag_name">74</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_46"/>
        <field name="code">74</field>
    </record>
    <record id="tax_report_de_tag_85" model="account.tax.report.line">
        <field name="name">85. Andere Leistungen eines im Ausland ansässigen Unternehmers (zeile 31)</field>
        <field name="tag_name">85</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">30</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_46"/>
        <field name="code">85</field>
    </record>

    <record id="tax_report_de_tax_tag_55" model="account.tax.report.line">
        <field name="name">Abziehbare Vorsteuerbetrage</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">30</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_17"/>
    </record>

    <record id="tax_report_de_tag_66" model="account.tax.report.line">
        <field name="name">66. Vorsteuerbeträge aus Rechnungen von anderen Unternehmern (zeile 37)</field>
        <field name="tag_name">66</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_55"/>
        <field name="code">66</field>
    </record>
    <record id="tax_report_de_tag_61" model="account.tax.report.line">
        <field name="name">61. Vorsteuerbeträge aus dem innergemeinschaftlichen Erwerb von Gegenständen (zeile 38)</field>
        <field name="tag_name">61</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_55"/>
        <field name="code">61</field>
    </record>
    <record id="tax_report_de_tag_62" model="account.tax.report.line">
        <field name="name">62. Entstandene Einfuhrumsatzsteuer (zeile 39)</field>
        <field name="tag_name">62</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">30</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_55"/>
        <field name="code">62</field>
    </record>
    <record id="tax_report_de_tag_67" model="account.tax.report.line">
        <field name="name">67. Vorsteuerbeträge aus Leistungen im Sinne des § 13b UStG (zeile 40)</field>
        <field name="tag_name">67</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">40</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_55"/>
        <field name="code">67</field>
    </record>
    <record id="tax_report_de_tag_63" model="account.tax.report.line">
        <field name="name">63. Vorsteuerbeträge, die nach allgemeinen Durchschnittssätzen berechnet sind (zeile 41)</field>
        <field name="tag_name">63</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">50</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_55"/>
        <field name="code">63</field>
    </record>
    <record id="tax_report_de_tag_59" model="account.tax.report.line">
        <field name="name">59. Vorsteuerabzug für innergemeinschaftliche Lieferungen neuer Fahrzeuge außerhalb eines Unternehmens (zeile 42)</field>
        <field name="tag_name">59</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">60</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_55"/>
        <field name="code">59</field>
    </record>
    <record id="tax_report_de_tag_64" model="account.tax.report.line">
        <field name="name">64. Berichtigung des Vorsteuerabzugs (zeile 43)</field>
        <field name="tag_name">64</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">70</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_55"/>
        <field name="code">64</field>
    </record>

    <record id="tax_report_de_tax_tag_64" model="account.tax.report.line">
        <field name="name">Andere Steuerbetrage</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">40</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_17"/>
    </record>

    <record id="tax_report_de_tag_65" model="account.tax.report.line">
        <field name="name">65. Steuer infolge Wechsels der Besteuerungsform sowie Nachsteuer auf versteuerte Anzahlungen u. ä. wegen Steuersatzänderung (zeile 45)</field>
        <field name="tag_name">65</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_64"/>
        <field name="code">65</field>
    </record>
    <record id="tax_report_de_tag_69" model="account.tax.report.line">
        <field name="name">69. In Rechnungen unrichtig oder unberechtigt ausgewiesene Steuerbeträge (zeile 46)</field>
        <field name="tag_name">69</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_64"/>
        <field name="code">69</field>
    </record>
    <record id="tax_report_de_tag_39" model="account.tax.report.line">
        <field name="name">39. Abzug der festgesetzten Sondervorauszahlung für Dauerfristverlängerung (zeile 48)</field>
        <field name="tag_name">39</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">60</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_17"/>
        <field name="code">39</field>
    </record>
    <record id="tax_report_de_tag_83" model="account.tax.report.line">
        <field name="name">83. Verbleibende Umsatzsteuer-Vorauszahlung (zeile 68)</field>
        <field name="tag_name">83</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">70</field>
        <field name="parent_id" ref="tax_report_de_tax_tag_17"/>
        <field name="code">83</field>
    </record>

    <record id="tax_report_de_tag_71" model="account.tax.report.line">
        <field name="name">Minderung</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">30</field>
        <field name="formula">None</field>
    </record>
    <record id="tax_report_de_tag_50" model="account.tax.report.line">
        <field name="name">50. Minderung der Bemessungsgrundlage (zeile 50)</field>
        <field name="tag_name">50</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">10</field>
        <field name="parent_id" ref="tax_report_de_tag_71"/>
        <field name="code">50</field>
    </record>
    <record id="tax_report_de_tag_37_74" model="account.tax.report.line">
        <field name="name">37. Minderung der abziehbaren Vorsteuerbeträge (zeile 51)</field>
        <field name="tag_name">37</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence">20</field>
        <field name="parent_id" ref="tax_report_de_tag_71"/>
        <field name="code">37</field>
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

    <!-- Profit and loss tags -->
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

    <!-- Balance sheet tags -->
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

## File: data\menuitem_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_de_statements_menu" name="Germany" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>
</odoo>

```

## File: data\report_layout.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="report_layout_din5008" model="report.layout">
            <field name="view_id" ref="l10n_de.external_layout_din5008"/>
            <field name="image">/l10n_de/static/img/preview_din.png</field>
            <field name="pdf">/l10n_de/static/pdf/preview_din.pdf</field>
            <field name="name">DIN 5008</field>
        </record>
    </data>
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
    tags_60 = env.ref('l10n_de.tax_report_de_tag_60').tag_ids

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

## File: models\account_move.py

```python
from odoo import models, fields, _
from odoo.tools import format_date


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_de_template_data = fields.Binary(compute='_compute_l10n_de_template_data')
    l10n_de_document_title = fields.Char(compute='_compute_l10n_de_document_title')
    l10n_de_addresses = fields.Binary(compute='_compute_l10n_de_addresses')

    def _compute_l10n_de_template_data(self):
        for record in self:
            record.l10n_de_template_data = data = []
            if record.name:
                data.append((_("Invoice No."), record.name))
            if record.invoice_date:
                data.append((_("Invoice Date"), format_date(self.env, record.invoice_date)))
            if record.invoice_date_due:
                data.append((_("Due Date"), format_date(self.env, record.invoice_date_due)))
            if record.invoice_origin:
                data.append((_("Source"), record.invoice_origin))
            if record.ref:
                data.append((_("Reference"), record.ref))

    def _compute_l10n_de_document_title(self):
        for record in self:
            record.l10n_de_document_title = ''
            if record.move_type == 'out_invoice':
                if record.state == 'posted':
                    record.l10n_de_document_title = _('Invoice')
                elif record.state == 'draft':
                    record.l10n_de_document_title = _('Draft Invoice')
                elif record.state == 'cancel':
                    record.l10n_de_document_title = _('Cancelled Invoice')
            elif record.move_type == 'out_refund':
                record.l10n_de_document_title = _('Credit Note')
            elif record.move_type == 'in_refund':
                record.l10n_de_document_title = _('Vendor Credit Note')
            elif record.move_type == 'in_invoice':
                record.l10n_de_document_title = _('Vendor Bill')

    def _compute_l10n_de_addresses(self):
        for record in self:
            record.l10n_de_addresses = data = []
            if 'partner_shipping_id' not in record._fields:
                data.append((_("Invoicing Address:"), record.partner_id))
            elif record.partner_shipping_id == record.partner_id:
                data.append((_("Invoicing and Shipping Address:"), record.partner_shipping_id))
            elif record.move_type in ("in_invoice", "in_refund") or not record.partner_shipping_id:
                data.append((_("Invoicing and Shipping Address:"), record.partner_id))
            else:
                data.append((_("Shipping Address:"), record.partner_shipping_id))
                data.append((_("Invoicing Address:"), record.partner_id))

    def check_field_access_rights(self, operation, field_names):
        field_names = super().check_field_access_rights(operation, field_names)
        return [field_name for field_name in field_names if field_name not in {
            'l10n_de_addresses',
        }]

```

## File: models\base_document_layout.py

```python
from odoo import models, fields, _
from odoo.tools import format_date


class BaseDocumentLayout(models.TransientModel):
    _inherit = 'base.document.layout'

    street = fields.Char(related='company_id.street', readonly=True)
    street2 = fields.Char(related='company_id.street2', readonly=True)
    zip = fields.Char(related='company_id.zip', readonly=True)
    city = fields.Char(related='company_id.city', readonly=True)
    company_registry = fields.Char(related='company_id.company_registry', readonly=True)
    bank_ids = fields.One2many(related='company_id.partner_id.bank_ids', readonly=True)
    account_fiscal_country_id = fields.Many2one(related='company_id.account_fiscal_country_id', readonly=True)
    l10n_de_template_data = fields.Binary(compute='_compute_l10n_de_template_data')
    l10n_de_document_title = fields.Char(compute='_compute_l10n_de_document_title')

    def _compute_l10n_de_template_data(self):
        self.l10n_de_template_data = [
            (_("Invoice No."), 'INV/2021/12345'),
            (_("Invoice Date"), format_date(self.env, fields.Date.today())),
            (_("Due Date"), format_date(self.env, fields.Date.add(fields.Date.today(), days=7))),
            (_("Reference"), 'SO/2021/45678'),
        ]

    def _compute_l10n_de_document_title(self):
        self.l10n_de_document_title = _('Invoice')

```

## File: models\chart_template.py

```python
# -*- coding: utf-8 -*-
from odoo import api, models


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    @api.model
    def _prepare_transfer_account_for_direct_creation(self, name, company):
        res = super(AccountChartTemplate, self)._prepare_transfer_account_for_direct_creation(name, company)
        if company.account_fiscal_country_id.code == 'DE':
            xml_id = self.env.ref('l10n_de.tag_de_asset_bs_B_III_2').id
            res.setdefault('tag_ids', [])
            res['tag_ids'].append((4, xml_id))
        return res

    # Write paperformat and report template used on company
    def _load(self, sale_tax_rate, purchase_tax_rate, company):
        res = super(AccountChartTemplate, self)._load(sale_tax_rate, purchase_tax_rate, company)
        if company.account_fiscal_country_id.code == 'DE':
            company.write({'external_report_layout_id': self.env.ref('l10n_de.external_layout_din5008').id,
            'paperformat_id': self.env.ref('l10n_de.paperformat_euro_din').id})
        return res

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
                    result['income'] = self.env['account.account'].search([('internal_group', '=', 'income'), ('deprecated', '=', False),
                                                                   ('tax_ids', 'in', taxes.ids)], limit=1)
            if not self.property_account_expense_id:
                supplier_taxes = self.supplier_taxes_id.filtered(lambda t: t.company_id == company)
                if not result['expense'] or (result['expense'].tax_ids and supplier_taxes and supplier_taxes[0] not in result['expense'].tax_ids):
                    result['expense'] = self.env['account.account'].search([('internal_group', '=', 'expense'), ('deprecated', '=', False),
                                                                   ('tax_ids', 'in', supplier_taxes.ids)], limit=1)
        return result

```

## File: models\hr_timesheet.py

```python
from odoo import models, fields, api, _

class AccountAnalyticLine(models.Model):
    _inherit = 'account.analytic.line'

    l10n_de_template_data = fields.Binary(compute='_compute_l10n_de_template_data')
    l10n_de_document_title = fields.Char(compute='_compute_l10n_de_document_title')

    def _compute_l10n_de_template_data(self):
        for record in self:
            record.l10n_de_template_data = []

    def _compute_l10n_de_document_title(self):
        for record in self:
            record.l10n_de_document_title = ''


```

## File: models\ir_actions_report.py

```python
from odoo import models


class IrActionsReport(models.Model):
    _inherit = 'ir.actions.report'

    def _get_rendering_context(self, docids, data):
        data = super()._get_rendering_context(docids, data)
        data['din_header_spacing'] = self.get_paperformat().header_spacing
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
from . import base_document_layout
from . import chart_template
from . import ir_actions_report
from . import account_move
from . import res_company
from . import hr_timesheet

```

## File: report\din5008_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

        <!-- New report paperformat for din5008 format -->
        <record id="paperformat_euro_din_a" model="report.paperformat">
            <field name="name">European A4 for DIN5008 Type A</field>
            <field name="default" eval="False" />
            <field name="format">A4</field>
            <field name="orientation">Portrait</field>
            <field name="margin_top">27</field>
            <field name="margin_bottom">40</field>
            <field name="margin_left">20</field>
            <field name="margin_right">10</field>
            <field name="dpi">70</field>
            <field name="header_line" eval="False" />
            <field name="header_spacing">27</field>
        </record>

        <record id="paperformat_euro_din" model="report.paperformat">
            <field name="name">European A4 for DIN5008 Type B</field>
            <field name="default" eval="False" />
            <field name="format">A4</field>
            <field name="orientation">Portrait</field>
            <field name="margin_top">45</field>
            <field name="margin_bottom">40</field>
            <field name="margin_left">20</field>
            <field name="margin_right">10</field>
            <field name="dpi">70</field>
            <field name="header_line" eval="False" />
            <field name="header_spacing">45</field>
        </record>

        <!-- New report layout for din5008 format -->
        <template id="external_layout_din5008">
            <div>
                <div t-attf-class="header din_page o_company_#{company.id}_layout #{'din_page_pdf' if report_type == 'pdf' else ''}">
                    <table class="company_header" t-att-style="'height: %dmm;' % (din_header_spacing or 27)">
                        <tr>
                            <td><h3 class="mt0" t-field="company.report_header"/></td>
                            <td><img t-if="company.logo" t-att-src="image_data_uri(company.logo)" t-att-style="'max-height: %dmm;' % (din_header_spacing or 27)"/></td>
                        </tr>
                    </table>
                </div>

                <div t-attf-class="din_page invoice_note article o_company_#{company.id}_layout #{'din_page_pdf' if report_type == 'pdf' else ''}" t-att-data-oe-model="o and o._name" t-att-data-oe-id="o and o.id">
                    <table>
                        <tr>
                            <td>
                                <div class="address">
                                    <t t-if="company.name">
                                        <span t-field="company.name"/>
                                    </t>
                                    <t t-if="company.street">
                                        <span>|</span> <span t-field="company.street"/>
                                    </t>
                                    <t t-if="company.street2">
                                        <span>|</span> <span t-field="company.street2"/>
                                    </t>
                                    <t t-if="company.zip">
                                        <span>|</span> <span t-field="company.zip"/>
                                    </t>
                                    <t t-if="company.city">
                                        <span t-if="not company.zip">|</span> <span t-field="company.city"/>
                                    </t>
                                    <t t-if="company.country_id">
                                        <span>|</span> <span t-field="company.country_id.name"/>
                                    </t>
                                    <hr class="company_invoice_line" />
                                    <div t-if="address">
                                        <t t-out="address"/>
                                    </div>
                                    <div t-else="fallback_address">
                                        <t t-esc="fallback_address"
                                           t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}' />
                                    </div>
                                </div>
                            </td>
                            <td>
                                <div class="information_block">
                                    <t t-if="'l10n_de_template_data' in company" t-set="template_data" t-value="company.l10n_de_template_data"/>
                                    <t t-if="o and 'l10n_de_template_data' in o" t-set="template_data" t-value="o.l10n_de_template_data"/>
                                    <table>
                                        <t t-foreach="template_data" t-as="row">
                                            <tr><td><t t-esc="row[0]"/></td><td><t t-esc="row[1]"/></td></tr>
                                        </t>
                                    </table>
                                </div>
                            </td>
                        </tr>
                        <tr t-if="o and 'l10n_de_addresses' in o">
                            <t t-foreach="o.l10n_de_addresses" t-as="doc_address">
                                <td>
                                    <div class="shipping_address">
                                        <strong><t t-esc="doc_address[0]"/></strong>
                                        <div t-esc="doc_address[1]" t-options='{"widget": "contact", "fields": ["address", "name", "phone"], "no_marker": True}'/>
                                    </div>
                                </td>
                            </t>
                        </tr>
                    </table>
                    <h2>
                        <span t-if="not o and not docs"><t t-esc="company.l10n_de_document_title"/></span>
                        <span t-else="">
                            <t t-set="o" t-value="docs[0]" t-if="not o" />
                            <span t-if="'l10n_de_document_title' in o"><t t-esc="o.l10n_de_document_title"/></span>
                            <span t-elif="'name' in o" t-field="o.name"/>
                        </span>
                    </h2>
                    <t t-out="0"/>
                </div>

                <div t-attf-class="din_page footer o_company_#{company.id}_layout #{'din_page_pdf' if report_type == 'pdf' else ''}">
                    <div class="text-right page_number">
                        <div class="text-muted">
                            Page: <span class="page"/> of <span class="topage"/>
                        </div>
                    </div>
                    <div class="company_details">
                        <table>
                            <tr>
                                <td>
                                    <ul class="list-inline text-nowrap">
                                        <li t-if="company.name"><span t-field="company.name"/></li>
                                        <li t-if="company.street"><span t-field="company.street"/></li>
                                        <li t-if="company.street2"><span t-field="company.street2"/></li>
                                        <li><span t-if="company.zip" t-field="company.zip"/> <span t-if="company.city" t-field="company.city"/></li>
                                        <li t-if="company.country_id"><span t-field="company.country_id.name"/></li>
                                    </ul>
                                </td>
                                <td>
                                    <ul class="list-inline">
                                        <li t-if="company.phone"><span t-field="company.phone"/></li>
                                        <li t-if="company.email"><span t-field="company.email"/></li>
                                        <li t-if="company.website"><span t-field="company.website"/></li>
                                    </ul>
                                </td>
                                <td>
                                    <ul class="list-inline">
                                        <li t-if="company.vat"><t t-esc="company.account_fiscal_country_id.vat_label or 'Tax ID'"/>:
                                            <span t-if="forced_vat" t-esc="forced_vat"/>
                                            <span t-else="" t-field="company.vat"/>
                                        </li>
                                        <li>HRB Nr: <span t-field="company.company_registry"/></li>
                                        <div t-field="company.report_footer"/>
                                    </ul>
                                </td>
                                <td>
                                    <ul class="list-inline" t-if="company.partner_id.bank_ids">
                                        <t t-foreach="company.partner_id.bank_ids[:2]" t-as="bank">
                                            <li><span t-field="bank.bank_id.name"/></li>
                                            <li>IBAN: <span t-field="bank.acc_number"/></li>
                                            <li>BIC: <span t-field="bank.bank_id.bic"/></li>
                                        </t>
                                    </ul>
                                </td>
                            </tr>
                        </table>
                    </div>
                </div>
            </div>
        </template>

        <template id="din5008_css" inherit_id="web.styles_company_report">
            <xpath expr="//t[@t-elif]" position="before">
                <t t-elif="layout == 'l10n_de.external_layout_din5008'">
                    &amp;.din_page {
                        &amp;.header {
                            .company_header {
                                .name_container {
                                    color: <t t-esc='primary'/>;
                                }
                            }
                        }
                        &amp;.invoice_note {
                            td {
                                .address {
                                    > span {
                                        color: <t t-esc='secondary'/>;
                                    }
                                }
                            }
                            h2 {
                                color: <t t-esc='primary'/>;
                            }
                            .page {
                                [name=invoice_line_table], [name=stock_move_table], .o_main_table {
                                    th {
                                        color: <t t-esc='secondary'/>;
                                    }
                                }
                            }
                        }
                    }
                </t>
            </xpath>
        </template>
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

