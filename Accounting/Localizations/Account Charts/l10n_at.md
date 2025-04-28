# Odoo Module: l10n_at

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2015 WT-IO-IT GmbH (https://www.wt-io-it.at)
#                    Mag. Wolfgang Taferner <wolfgang.taferner@wt-io-it.at>

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (c) 2015 WT-IO-IT GmbH (https://www.wt-io-it.at)
#                    Mag. Wolfgang Taferner <wolfgang.taferner@wt-io-it.at>

# List of contributors:
# Mag. Wolfgang Taferner <wolfgang.taferner@wt-io-it.at>
# Josse Colpaert <jco@odoo.com>

{
    "name": "Austria - Accounting",
    "version": "3.0",
    "author": "WT-IO-IT GmbH, Wolfgang Taferner",
    "website": "https://www.wt-io-it.at",
    'category': 'Accounting/Localizations/Account Charts',
    'summary': "Austrian Standardized Charts & Tax",
    "description": """

Austrian charts of accounts (Einheitskontenrahmen 2010).
==========================================================

    * Defines the following chart of account templates:
        * Austrian General Chart of accounts 2010
    * Defines templates for VAT on sales and purchases
    * Defines tax templates
    * Defines fiscal positions for Austrian fiscal legislation
    * Defines tax reports U1/U30

    """,
    "depends": [
        "account",
        "base_iban",
        "base_vat",
    ],
    "data": [
        'data/res.country.state.csv',
        'data/account_account_tag.xml',
        'data/account_account_template.xml',
        'data/account_chart_template.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_template.xml',
        'data/account_fiscal_position_template.xml',
        'data/account_chart_template_configure_data.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_account_tag.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record
            id="account_tag_l10n_at_AAI1"
            model="account.account.tag">
            <field name="name">Bilanz AAI1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AAI2"
            model="account.account.tag">
            <field name="name">Bilanz AAI2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AAI3"
            model="account.account.tag">
            <field name="name">Bilanz AAI3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AAII1"
            model="account.account.tag">
            <field name="name">Bilanz AAII1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AAII2"
            model="account.account.tag">
            <field name="name">Bilanz AAII2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AAII3"
            model="account.account.tag">
            <field name="name">Bilanz AAII3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AAII4"
            model="account.account.tag">
            <field name="name">Bilanz AAII4</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AAIII"
            model="account.account.tag">
            <field name="name">Bilanz AAIII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AAIII1"
            model="account.account.tag">
            <field name="name">Bilanz AAIII1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AAIII2"
            model="account.account.tag">
            <field name="name">Bilanz AAIII2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AAIII3"
            model="account.account.tag">
            <field name="name">Bilanz AAIII3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AAIII4"
            model="account.account.tag">
            <field name="name">Bilanz AAIII4</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AAIII5"
            model="account.account.tag">
            <field name="name">Bilanz AAIII5</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AAIII6"
            model="account.account.tag">
            <field name="name">Bilanz AAIII6</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ABI1"
            model="account.account.tag">
            <field name="name">Bilanz ABI1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ABI2"
            model="account.account.tag">
            <field name="name">Bilanz ABI2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ABI3"
            model="account.account.tag">
            <field name="name">Bilanz ABI3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ABI4"
            model="account.account.tag">
            <field name="name">Bilanz ABI4</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ABI5"
            model="account.account.tag">
            <field name="name">Bilanz ABI5</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ABII1"
            model="account.account.tag">
            <field name="name">Bilanz ABII1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ABII2"
            model="account.account.tag">
            <field name="name">Bilanz ABII2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ABII3"
            model="account.account.tag">
            <field name="name">Bilanz ABII3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ABII4"
            model="account.account.tag">
            <field name="name">Bilanz ABII4</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ABIII"
            model="account.account.tag">
            <field name="name">Bilanz ABIII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ABIII1"
            model="account.account.tag">
            <field name="name">Bilanz ABIII1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ABIII2"
            model="account.account.tag">
            <field name="name">Bilanz ABIII2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ABIV"
            model="account.account.tag">
            <field name="name">Bilanz ABIV</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AC"
            model="account.account.tag">
            <field name="name">Bilanz AC</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_AD"
            model="account.account.tag">
            <field name="name">Bilanz AD</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PAI"
            model="account.account.tag">
            <field name="name">Bilanz PAI</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PAII"
            model="account.account.tag">
            <field name="name">Bilanz PAII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PAII1"
            model="account.account.tag">
            <field name="name">Bilanz PAII1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PAII2"
            model="account.account.tag">
            <field name="name">Bilanz PAII2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PAIII"
            model="account.account.tag">
            <field name="name">Bilanz PAIII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PAIII1"
            model="account.account.tag">
            <field name="name">Bilanz PAIII1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PAIII2"
            model="account.account.tag">
            <field name="name">Bilanz PAIII2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PAIII3"
            model="account.account.tag">
            <field name="name">Bilanz PAIII3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PAIV"
            model="account.account.tag">
            <field name="name">Bilanz PAIV</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PBI"
            model="account.account.tag">
            <field name="name">Bilanz PBI</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PBII"
            model="account.account.tag">
            <field name="name">Bilanz PBII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PBIII"
            model="account.account.tag">
            <field name="name">Bilanz PBIII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PBIV"
            model="account.account.tag">
            <field name="name">Bilanz PBIV</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PCI"
            model="account.account.tag">
            <field name="name">Bilanz PCI</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PCII"
            model="account.account.tag">
            <field name="name">Bilanz PCII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PCIII"
            model="account.account.tag">
            <field name="name">Bilanz PCIII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PCIV"
            model="account.account.tag">
            <field name="name">Bilanz PCIV</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PCV"
            model="account.account.tag">
            <field name="name">Bilanz PCV</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PCVI"
            model="account.account.tag">
            <field name="name">Bilanz PCVI</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PCVII"
            model="account.account.tag">
            <field name="name">Bilanz PCVII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PCVIII"
            model="account.account.tag">
            <field name="name">Bilanz PCVIII</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PCVIII1"
            model="account.account.tag">
            <field name="name">Bilanz PCVIII1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PCVIII2"
            model="account.account.tag">
            <field name="name">Bilanz PCVIII2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PD"
            model="account.account.tag">
            <field name="name">Bilanz PD</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>

        <record
            id="account_tag_l10n_at_EBIT1"
            model="account.account.tag">
            <field name="name">GuV EBIT1</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_EBIT2"
            model="account.account.tag">
            <field name="name">GuV EBIT2</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_EBIT3"
            model="account.account.tag">
            <field name="name">GuV EBIT3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_EBIT4"
            model="account.account.tag">
            <field name="name">GuV EBIT4</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_EBIT4I"
            model="account.account.tag">
            <field name="name">GuV EBIT4I</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_EBIT4II"
            model="account.account.tag">
            <field name="name">GuV EBIT4II</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_EBIT4III"
            model="account.account.tag">
            <field name="name">GuV EBIT4III</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_EBIT5I"
            model="account.account.tag">
            <field name="name">GuV EBIT5I</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_EBIT5II"
            model="account.account.tag">
            <field name="name">GuV EBIT5II</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_EBIT6I"
            model="account.account.tag">
            <field name="name">GuV EBIT6I</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_EBIT6II"
            model="account.account.tag">
            <field name="name">GuV EBIT6II</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_EBIT7I"
            model="account.account.tag">
            <field name="name">GuV EBIT7I</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_EBIT7II"
            model="account.account.tag">
            <field name="name">GuV EBIT7II</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_EBIT8"
            model="account.account.tag">
            <field name="name">GuV EBIT8</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_FIN10"
            model="account.account.tag">
            <field name="name">GuV FIN10</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_FIN11"
            model="account.account.tag">
            <field name="name">GuV FIN11</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_FIN12"
            model="account.account.tag">
            <field name="name">GuV FIN12</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_FIN13"
            model="account.account.tag">
            <field name="name">GuV FIN13</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_FIN14"
            model="account.account.tag">
            <field name="name">GuV FIN14</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_FIN15"
            model="account.account.tag">
            <field name="name">GuV FIN15</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_TAX"
            model="account.account.tag">
            <field name="name">GuV TAX</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_MTAX"
            model="account.account.tag">
            <field name="name">GuV MTAX</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_RL"
            model="account.account.tag">
            <field name="name">GuV RL</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
    </data>
</odoo>

```

## File: data\account_account_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="chart_at_template_transfer_288" model="account.account.template">
            <field name="name">Schwebende Geldbewegungen</field>
            <field name="code">2880</field>
            <field name="reconcile" eval="True"/>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
        </record>
        <!-- Vorlage: Kontenplan Österreich (Basis: EKR 2000) -->
        <record id="l10n_at_chart_template" model="account.chart.template">
            <field name="name">Einheitskontenrahmen Österreich 2010</field>
            <field name="code_digits">4</field>
            <field name="bank_account_code_prefix">280</field>
            <field name="cash_account_code_prefix">270</field>
            <field name="transfer_account_code_prefix">288</field>
            <field name="currency_id" ref="base.EUR"/>
        </record>
        <record id="chart_at_template_transfer_288" model="account.account.template">
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
        </record>

        <record id="chart_at_template_0010" model="account.account.template">
          <field name="name">Aufwendungen für das Ingangsetzen und Erweitern eines Betriebes</field>
          <field name="code">0010</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI1')])]" />
        </record>
        <record id="chart_at_template_0090" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen</field>
          <field name="code">0090</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
        </record>
        <record id="chart_at_template_0100" model="account.account.template">
          <field name="name">Konzessionen</field>
          <field name="code">0100</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI1')])]" />
        </record>
        <record id="chart_at_template_0110" model="account.account.template">
          <field name="name">Patentrechte und Lizenzen</field>
          <field name="code">0110</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI1')])]" />
        </record>
        <record id="chart_at_template_0120" model="account.account.template">
          <field name="name">Datenverarbeitungsprogramme</field>
          <field name="code">0120</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI1')])]" />
        </record>
        <record id="chart_at_template_0121" model="account.account.template">
          <field name="name">Geringwertige Datenverarbeitungsprogramme</field>
          <field name="code">0121</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI1')])]" />
        </record>
        <record id="chart_at_template_0130" model="account.account.template">
          <field name="name">Marken, Warenzeichen und Musterschutzrechte, sonstige Urheberrechte</field>
          <field name="code">0130</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI1')])]" />
        </record>
        <record id="chart_at_template_0140" model="account.account.template">
          <field name="name">Pacht- und Mietrechte</field>
          <field name="code">0140</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI1')])]" />
        </record>
        <record id="chart_at_template_0150" model="account.account.template">
          <field name="name">Geschäfts(Firmen)wert</field>
          <field name="code">0150</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI2')])]" />
        </record>
        <record id="chart_at_template_0180" model="account.account.template">
          <field name="name">Geleistete Anzahlungen</field>
          <field name="code">0180</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI3')])]" />
        </record>
        <record id="chart_at_template_019" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen</field>
          <field name="code">0190</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
        </record>
        <record id="chart_at_template_0200" model="account.account.template">
          <field name="name">Unbebaute Grundstücke</field>
          <field name="code">0200</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0210" model="account.account.template">
          <field name="name">Bebaute Grundstücke (Grundwert)</field>
          <field name="code">0210</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_022" model="account.account.template">
          <field name="name">Grundstücksgleiche Rechte</field>
          <field name="code">0220</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0300" model="account.account.template">
          <field name="name">Betriebs- und Geschäftsgebäude auf eigenem Grund</field>
          <field name="code">0300</field>
          <field name="note">KZ 9320 (Bilanzierer gemäß §§ 4 Abs. 1 oder 5)</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0310" model="account.account.template">
          <field name="name">Wohn- und Sozialgebäude auf eigenem Grund</field>
          <field name="code">0310</field>
          <field name="note">KZ 9320 (Bilanzierer gemäß §§ 4 Abs. 1 oder 5)</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0320" model="account.account.template">
          <field name="name">Betriebs- und Geschäftsgebäude auf fremdem Grund</field>
          <field name="code">0320</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0330" model="account.account.template">
          <field name="name">Wohn- und Sozialgebäude auf fremdem Grund</field>
          <field name="code">0330</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0340" model="account.account.template">
          <field name="name">Grundstückseinrichtungen auf eigenem Grund</field>
          <field name="code">0340</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0350" model="account.account.template">
          <field name="name">Grundstückseinrichtungen auf fremdem Grund</field>
          <field name="code">0350</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0360" model="account.account.template">
          <field name="name">Bauliche Investitionen in fremden (gepachteten) Betriebs- und Geschäftsgebäuden</field>
          <field name="code">0360</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0370" model="account.account.template">
          <field name="name">Bauliche Investitionen in fremden (gepachteten) Wohn- und Sozialgebäuden</field>
          <field name="code">0370</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0390" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen</field>
          <field name="code">0390</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0400" model="account.account.template">
          <field name="name">Fertigungsmaschinen</field>
          <field name="code">0400</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0410" model="account.account.template">
          <field name="name">Antriebsmaschinen</field>
          <field name="code">0410</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0420" model="account.account.template">
          <field name="name">Energieversorgungsanlagen</field>
          <field name="code">0420</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0430" model="account.account.template">
          <field name="name">Transportanlagen</field>
          <field name="code">0430</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0500" model="account.account.template">
          <field name="name">Maschinenwerkzeuge</field>
          <field name="code">0500</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0510" model="account.account.template">
          <field name="name">Allgemeine Werkzeuge und Handwerkzeuge</field>
          <field name="code">0510</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0520" model="account.account.template">
          <field name="name">Vorrichtungen, Formen und Modelle</field>
          <field name="code">0520</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0530" model="account.account.template">
          <field name="name">Andere Erzeugungshilfsmittel</field>
          <field name="code">0530</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0540" model="account.account.template">
          <field name="name">Hebezeuge und Montageanlagen</field>
          <field name="code">0540</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0550" model="account.account.template">
          <field name="name">Geringwertige Vermögensgegenstände, soweit im Erzeugungsprozeß verwendet</field>
          <field name="code">0550</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0600" model="account.account.template">
          <field name="name">Beheizungs- und Beleuchtungsanlagen</field>
          <field name="code">0600</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0610" model="account.account.template">
          <field name="name">Nachrichten- und Kontrollanlagen</field>
          <field name="code">0610</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0620" model="account.account.template">
          <field name="name">Büromaschinen, EDV-Anlagen</field>
          <field name="code">0620</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0630" model="account.account.template">
          <field name="name">PKW</field>
          <field name="code">0630</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0640" model="account.account.template">
          <field name="name">LKW</field>
          <field name="code">0640</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0650" model="account.account.template">
          <field name="name">Andere Beförderungsmittel</field>
          <field name="code">0650</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0660" model="account.account.template">
          <field name="name">Andere Betriebs- und Geschäftsausstattung</field>
          <field name="code">0660</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0670" model="account.account.template">
          <field name="name">Gebinde</field>
          <field name="code">0670</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0680" model="account.account.template">
          <field name="name">Geringwertige Büromaschinen, EDV-Anlagen</field>
          <field name="code">0680</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0681" model="account.account.template">
          <field name="name">Geringwertige Betriebs- und Geschäftsausstattung</field>
          <field name="code">0681</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0692" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen zu Büromaschinen, EDV-Anlagen</field>
          <field name="code">0692</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0693" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen zu PKW und Kombis</field>
          <field name="code">0693</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0694" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen zu LKW</field>
          <field name="code">0694</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0696" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen zur Betriebs- und Geschäftsausstattung</field>
          <field name="code">0696</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0700" model="account.account.template">
          <field name="name">Geleistete Anzahlungen</field>
          <field name="code">0700</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_prepayments" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII4')])]" />
        </record>
        <record id="chart_at_template_0710" model="account.account.template">
          <field name="name">Anlagen in Bau</field>
          <field name="code">0710</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII4')])]" />
        </record>
        <record id="chart_at_template_0790" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen</field>
          <field name="code">0790</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII4')])]" />
        </record>
        <record id="chart_at_template_0800" model="account.account.template">
          <field name="name">Anteile an verbundenen Unternehmen</field>
          <field name="code">0800</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII1')])]" />
        </record>
        <record id="chart_at_template_0810" model="account.account.template">
          <field name="name">Beteiligungen an Gemeinschaftsunternehmen</field>
          <field name="code">0810</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII1')])]" />
        </record>
        <record id="chart_at_template_0820" model="account.account.template">
          <field name="name">Beteiligungen an angeschlossenen (assoziierten) Unternehmen</field>
          <field name="code">0820</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII1')])]" />
        </record>
        <record id="chart_at_template_0830" model="account.account.template">
          <field name="name">Sonstige Beteiligungen</field>
          <field name="code">0830</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII1')])]" />
        </record>
        <record id="chart_at_template_0840" model="account.account.template">
          <field name="name">Ausleihungen an verbundene Unternehmen</field>
          <field name="code">0840</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII2')])]" />
        </record>
        <record id="chart_at_template_0850" model="account.account.template">
          <field name="name">Ausleihungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht</field>
          <field name="code">0850</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII4')])]" />
        </record>
        <record id="chart_at_template_0860" model="account.account.template">
          <field name="name">Sonstige Ausleihungen</field>
          <field name="code">0860</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII6')])]" />
        </record>
        <record id="chart_at_template_0870" model="account.account.template">
          <field name="name">Anteile an Kapitalgesellschaften ohne Beteiligungscharakter</field>
          <field name="code">0870</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII5')])]" />
        </record>
        <record id="chart_at_template_0880" model="account.account.template">
          <field name="name">Anteile an Personengesellschaften ohne Beteiligungscharakter</field>
          <field name="code">0880</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII5')])]" />
        </record>
        <record id="chart_at_template_0900" model="account.account.template">
          <field name="name">Genossenschaftsanteile ohne Beteiligungscharakter</field>
          <field name="code">0900</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII5')])]" />
        </record>
        <record id="chart_at_template_0910" model="account.account.template">
          <field name="name">Anteile an Investmentfonds</field>
          <field name="code">0910</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII5')])]" />
        </record>
        <record id="chart_at_template_0980" model="account.account.template">
          <field name="name">Geleistete Anzahlungen</field>
          <field name="code">0980</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_prepayments" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII5')])]" />
        </record>
        <record id="chart_at_template_0990" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen</field>
          <field name="code">0990</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII5')])]" />
        </record>
        <record id="chart_at_template_1600" model="account.account.template">
          <field name="name">Handelswaren</field>
          <field name="code">1600</field>
          <field name="note">KZ 9340 (Bilanzierer gemäß §§ 4 Abs. 1 oder 5)</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABI3')])]" />
        </record>
        <record id="chart_at_template_1800" model="account.account.template">
          <field name="name">Geleistete Anzahlungen</field>
          <field name="code">1800</field>
          <field name="note">KZ 9340 (Bilanzierer gemäß §§ 4 Abs. 1 oder 5)</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_prepayments" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABI5')])]" />
        </record>
        <record id="chart_at_template_2000" model="account.account.template">
          <field name="name">Forderungen von Partnern im Inland ohne eigenes Debitorenkonto</field>
          <field name="code">2000</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_receivable" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2099" model="account.account.template">
          <field name="name">Forderungen von Partnern (Point Of Sale)</field>
          <field name="code">2099</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_receivable" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2080" model="account.account.template">
          <field name="name">Einzelwertberichtigungen zu Forderungen aus Lieferungen und Leistungen Inland</field>
          <field name="code">2080</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2090" model="account.account.template">
          <field name="name">Pauschalwertberichtigungen zu Forderungen aus Lieferungen und Leistungen Inland</field>
          <field name="code">2090</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2100" model="account.account.template">
          <field name="name">Forderungen von Partnern im EU Raum ohne eigenes Debitorenkonto</field>
          <field name="code">2100</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_receivable" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2130" model="account.account.template">
          <field name="name">Einzelwertberichtigungen zu Forderungen aus Lieferungen und Leistungen Währungsunion</field>
          <field name="code">2130</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2140" model="account.account.template">
          <field name="name">Pauschalwertberichtigungen zu Forderungen aus Lieferungen und Leistungen Währungsunion</field>
          <field name="code">2140</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2150" model="account.account.template">
          <field name="name">Forderungen von Partnern in Drittstaaten ohne eigenes Debitorenkonto</field>
          <field name="code">2150</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_receivable" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2180" model="account.account.template">
          <field name="name">Einzelwertberichtigungen zu Forderungen aus Lieferungen und Leistungen sonstiges Ausland</field>
          <field name="code">2180</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2190" model="account.account.template">
          <field name="name">Pauschalwertberichtigungen zu Forderungen aus Lieferungen und Leistungen sonstiges Ausland</field>
          <field name="code">2190</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2230" model="account.account.template">
          <field name="name">Einzelwertberichtigungen zu Forderungen gegenüber verbundenen Unternehmen</field>
          <field name="code">2230</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII2')])]" />
        </record>
        <record id="chart_at_template_2240" model="account.account.template">
          <field name="name">Pauschalwertberichtigungen zu Forderungen gegenüber verbundenen Unternehmen</field>
          <field name="code">2240</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII2')])]" />
        </record>
        <record id="chart_at_template_2280" model="account.account.template">
          <field name="name">Einzelwertberichtigungen zu Forderungen gegenüber Unternehmen, mit denen ein Beteiligungsverhältnis besteht</field>
          <field name="code">2280</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII3')])]" />
        </record>
        <record id="chart_at_template_2290" model="account.account.template">
          <field name="name">Pauschalwertberichtigungen zu Forderungen gegenüber Unternehmen, mit denen ein Beteiligungsverhältnis besteht</field>
          <field name="code">2290</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII3')])]" />
        </record>
        <record id="chart_at_template_2470" model="account.account.template">
          <field name="name">Eingeforderte, aber noch nicht eingezahlte Einlagen</field>
          <field name="code">2470</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII3')])]" />
        </record>
        <record id="chart_at_template_2480" model="account.account.template">
          <field name="name">Einzelwertberichtigungen zu sonstigen Forderungen und Vermögensgegenständen</field>
          <field name="code">2480</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2490" model="account.account.template">
          <field name="name">Pauschalwertberichtigungen zu sonstigen Forderungen und Vermögensgegenständen</field>
          <field name="code">2490</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2500" model="account.account.template">
          <field name="name">Vorsteuern 20%</field>
          <field name="code">2500</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2501" model="account.account.template">
          <field name="name">Vorsteuern 10%</field>
          <field name="code">2501</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2502" model="account.account.template">
          <field name="name">Vorsteuern 13%</field>
          <field name="code">2502</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2505" model="account.account.template">
          <field name="name">Sonstige Vorsteuern</field>
          <field name="code">2505</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2506" model="account.account.template">
          <field name="name">Vorsteuern RC 20%</field>
          <field name="code">2506</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2507" model="account.account.template">
          <field name="name">Vorsteuern RC 10%</field>
          <field name="code">2507</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2510" model="account.account.template">
          <field name="name">Vorsteuern RC EU 20%</field>
          <field name="code">2510</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2511" model="account.account.template">
          <field name="name">Vorsteuern IGE 20%</field>
          <field name="code">2511</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2512" model="account.account.template">
          <field name="name">Vorsteuern IGE 10%</field>
          <field name="code">2512</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2513" model="account.account.template">
          <field name="name">Vorsteuern IGE 13%</field>
          <field name="code">2513</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2515" model="account.account.template">
          <field name="name">Vorsteuern 20% (aus EUSt.)</field>
          <field name="code">2515</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2600" model="account.account.template">
          <field name="name">Eigene Anteile</field>
          <field name="code">2600</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIII')])]" />
        </record>
        <record id="chart_at_template_2610" model="account.account.template">
          <field name="name">Anteile an verbundenen Unternehmen</field>
          <field name="code">2610</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIII1')])]" />
        </record>
        <record id="chart_at_template_2620" model="account.account.template">
          <field name="name">Sonstige Anteile</field>
          <field name="code">2620</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIII2')])]" />
        </record>
        <record id="chart_at_template_2680" model="account.account.template">
          <field name="name">Besitzwechsel, soweit dem Unternehmen nicht die der Ausstellung zugrundeliegenden Forderungen zustehen</field>
          <field name="code">2680</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIII2')])]" />
        </record>
        <record id="chart_at_template_2690" model="account.account.template">
          <field name="name">Wertberichtigungen</field>
          <field name="code">2690</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIII2')])]" />
        </record>
        <record id="chart_at_template_2730" model="account.account.template">
          <field name="name">Postwertzeichen</field>
          <field name="code">2730</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIV')])]" />
        </record>
        <record id="chart_at_template_2740" model="account.account.template">
          <field name="name">Stempelmarken</field>
          <field name="code">2740</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIV')])]" />
        </record>
        <record id="chart_at_template_2780" model="account.account.template">
          <field name="name">Schecks in Inlandswährung</field>
          <field name="code">2780</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIV')])]" />
        </record>
        <record id="chart_at_template_transfer_288" model="account.account.template">
        </record>
        <record id="chart_at_template_2890" model="account.account.template">
          <field name="name">Wertberichtigungen</field>
          <field name="code">2890</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIV')])]" />
        </record>
        <record id="chart_at_template_2900" model="account.account.template">
          <field name="name">Aktive Rechnungsabgrenzungsposten</field>
          <field name="code">2900</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AC')])]" />
        </record>
        <record id="chart_at_template_2950" model="account.account.template">
          <field name="name">Disagio</field>
          <field name="code">2950</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AC')])]" />
        </record>
        <record id="chart_at_template_2960" model="account.account.template">
          <field name="name">Unterschiedsbetrag zur gebotenen Pensionsrückstellung</field>
          <field name="code">2960</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PBI')])]" />
        </record>
        <record id="chart_at_template_2970" model="account.account.template">
          <field name="name">Unterschiedsbetrag gem. Abschnitt XII Pensionskassengesetz</field>
          <field name="code">2970</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PBII')])]" />
        </record>
        <record id="chart_at_template_2980" model="account.account.template">
          <field name="name">Steuerabgrenzung</field>
          <field name="code">2980</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AC')])]" />
        </record>
        <record id="chart_at_template_3000" model="account.account.template">
          <field name="name">Rückstellungen für Abfertigungen</field>
          <field name="code">3000</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PBI')])]" />
        </record>
        <record id="chart_at_template_3010" model="account.account.template">
          <field name="name">Rückstellungen für Pensionen</field>
          <field name="code">3010</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_non_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PBII')])]" />
        </record>
        <record id="chart_at_template_3100" model="account.account.template">
          <field name="name">Anleihen (einschließlich konvertibler)</field>
          <field name="code">3100</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCI')])]" />
        </record>
        <record id="chart_at_template_3200" model="account.account.template">
          <field name="name">Erhaltene Anzahlungen auf Bestellungen</field>
          <field name="code">3200</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_assets" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCIII')])]" />
        </record>
        <record id="chart_at_template_3210" model="account.account.template">
          <field name="name">Umsatzsteuer-Evidenzkonto für erhaltene Anzahlungen auf Bestellungen</field>
          <field name="code">3210</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3300" model="account.account.template">
          <field name="name">Verbindlichkeiten bei Partnern im Inland ohne eigenes Kreditorenkonto</field>
          <field name="code">3300</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_payable" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCIV')])]" />
        </record>
        <record id="chart_at_template_3360" model="account.account.template">
          <field name="name">Verbindlichkeiten bei Partnern im EU Raum ohne eigenes Kreditorenkonto</field>
          <field name="code">3360</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_payable" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCIV')])]" />
        </record>
        <record id="chart_at_template_3370" model="account.account.template">
          <field name="name">Verbindlichkeiten bei Partnern in Drittstaaten ohne eigenes Kreditorenkonto</field>
          <field name="code">3370</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_payable" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCIV')])]" />
        </record>
        <record id="chart_at_template_3480" model="account.account.template">
          <field name="name">Verbindlichkeiten gegenüber Gesellschaftern</field>
          <field name="code">3480</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII')])]" />
        </record>
        <record id="chart_at_template_3500" model="account.account.template">
          <field name="name">Umsatzsteuer 20%</field>
          <field name="code">3500</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3501" model="account.account.template">
          <field name="name">Umsatzsteuer 10%</field>
          <field name="code">3501</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3502" model="account.account.template">
          <field name="name">Umsatzsteuer 13%</field>
          <field name="code">3502</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3505" model="account.account.template">
          <field name="name">Sonstige Umsatzsteuer</field>
          <field name="code">3505</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3510" model="account.account.template">
          <field name="name">Umsatzsteuer RC EU 20%</field>
          <field name="code">3510</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3511" model="account.account.template">
          <field name="name">Umsatzsteuer IGE 20%</field>
          <field name="code">3511</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3512" model="account.account.template">
          <field name="name">Umsatzsteuer IGE 10%</field>
          <field name="code">3512</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3513" model="account.account.template">
          <field name="name">Umsatzsteuer IGE 13%</field>
          <field name="code">3513</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3515" model="account.account.template">
          <field name="name">Einfuhrumsatzsteuer 20%</field>
          <field name="code">3515</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3520" model="account.account.template">
          <field name="name">Ust. Zahllast</field>
          <field name="code">3520</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3530" model="account.account.template">
          <field name="name">Verrechnungskonto Finanzamt</field>
          <field name="code">3530</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_payable" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3540" model="account.account.template">
          <field name="name">Verrechnung Lohnsteuer</field>
          <field name="code">3540</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3541" model="account.account.template">
          <field name="name">Verrechnung Dienstgeberbeitrag</field>
          <field name="code">3541</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3542" model="account.account.template">
          <field name="name">Verrechnung Dienstgeberzuschlag</field>
          <field name="code">3542</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3550" model="account.account.template">
          <field name="name">Verrechnung Kommunalsteuer</field>
          <field name="code">3550</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3551" model="account.account.template">
          <field name="name">Verrechnung Wiener Dienstgeberabgabe</field>
          <field name="code">3551</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3600" model="account.account.template">
          <field name="name">Verrechungskonto Sozialversicherung</field>
          <field name="code">3600</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_payable" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII2')])]" />
        </record>
        <record id="chart_at_template_3610" model="account.account.template">
          <field name="name">Verrechnungskonto Magistrat/Gemeinde (KoSt, U-Bahn, etc.)</field>
          <field name="code">3610</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_payable" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII2')])]" />
        </record>
        <record id="chart_at_template_3740" model="account.account.template">
          <field name="name">WERE Verrechnungskonto</field>
          <field name="code">3740</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_payable" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PD')])]" />
        </record>
        <record id="chart_at_template_4000" model="account.account.template">
          <field name="name">Brutto-Umsatzerlöse im Inland (20%)</field>
          <field name="code">4000</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_revenue" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT1')])]" />
        </record>
        <record id="chart_at_template_4001" model="account.account.template">
          <field name="name">Brutto-Umsatzerlöse im Inland (10%)</field>
          <field name="code">4001</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_revenue" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT1')])]" />
        </record>
        <record id="chart_at_template_4100" model="account.account.template">
          <field name="name">Brutto-Umsatzerlöse im EU Raum (RC 20%)</field>
          <field name="code">4100</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_revenue" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT1')])]" />
        </record>
        <record id="chart_at_template_4110" model="account.account.template">
          <field name="name">Brutto-Umsatzerlöse im EU Raum (RC 10%)</field>
          <field name="code">4110</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_revenue" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT1')])]" />
        </record>
        <record id="chart_at_template_4200" model="account.account.template">
          <field name="name">Brutto-Umsatzerlöse in Drittstaaten (0%)</field>
          <field name="code">4200</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_revenue" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT1')])]" />
        </record>
        <record id="chart_at_template_4860" model="account.account.template">
          <field name="name">Kursgewinne aus Fremdwährungstransaktionen</field>
          <field name="code">4860</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_other_income" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT1')])]" />
        </record>
        <record id="chart_at_template_5000" model="account.account.template">
          <field name="name">Wareneinkauf</field>
          <field name="code">5000</field>
          <field name="note">KZ 9100</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT5I')])]" />
        </record>
        <record id="chart_at_template_5050" model="account.account.template">
          <field name="name">Wareneinkauf EU</field>
          <field name="code">5050</field>
          <field name="note">KZ 9100</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT5I')])]" />
        </record>
        <record id="chart_at_template_5090" model="account.account.template">
          <field name="name">Wareneinkauf 0%</field>
          <field name="code">5090</field>
          <field name="note">KZ 9100</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT5I')])]" />
        </record>
        <record id="chart_at_template_5800" model="account.account.template">
          <field name="name">Skontoerträge auf Materialaufwand</field>
          <field name="code">5800</field>
          <field name="note">KZ 9100</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT5I')])]" />
        </record>
        <record id="chart_at_template_5810" model="account.account.template">
          <field name="name">Skontoerträge auf sonstige bezogene Herstellungsleistungen</field>
          <field name="code">5810</field>
          <field name="note">KZ 9110</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT5II')])]" />
        </record>
        <record id="chart_at_template_5900" model="account.account.template">
          <field name="name">Aufwandsstellenrechnung</field>
          <field name="code">5900</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating')])]" />
        </record>
        <record id="chart_at_template_6200" model="account.account.template">
          <field name="name">Gehälter (Angestellte)</field>
          <field name="code">6200</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6205" model="account.account.template">
          <field name="name">Geschäftsführerbezug</field>
          <field name="code">6205</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6242" model="account.account.template">
          <field name="name">Urlaubsabfindung (Angestellte)</field>
          <field name="code">6242</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6260" model="account.account.template">
          <field name="name">Sonstige Bezüge (Angestellte)</field>
          <field name="code">6260</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6270" model="account.account.template">
          <field name="name">Sachbezug (Angestellte)</field>
          <field name="code">6270</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6271" model="account.account.template">
          <field name="name">Sachbezug (Geschäftsführer)</field>
          <field name="code">6271</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6310" model="account.account.template">
          <field name="name">Grundgehälter (Überstunden)</field>
          <field name="code">6310</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6330" model="account.account.template">
          <field name="name">Gehälter (Überstundenzuschläge)</field>
          <field name="code">6330</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6340" model="account.account.template">
          <field name="name">Veränderung noch nicht konsumierter Urlaub</field>
          <field name="code">6340</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6400" model="account.account.template">
          <field name="name">Beiträge für betriebliche Mitarbeitervorsorgekasse</field>
          <field name="code">6400</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6II')])]" />
        </record>

        <record id="chart_at_template_6560" model="account.account.template">
          <field name="name">Gesetzlicher Sozialaufwand</field>
          <field name="code">6560</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6II')])]" />
        </record>

        <record id="chart_at_template_6660" model="account.account.template">
          <field name="name">Kommunalsteuer (KoSt)</field>
          <field name="code">6660</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6II')])]" />
        </record>
        <record id="chart_at_template_6661" model="account.account.template">
          <field name="name">Dienstgeberbeitrag zum Familienlastenausgleichsfonds (DB)</field>
          <field name="code">6661</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6II')])]" />
        </record>
        <record id="chart_at_template_6662" model="account.account.template">
          <field name="name">Zuschlag zum Dienstnehmerbeitrag (DZ)</field>
          <field name="code">6662</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6II')])]" />
        </record>
        <record id="chart_at_template_6640" model="account.account.template">
          <field name="name">Dienstgeberabgabe der Gemeinde Wien (U-Bahn Steuer)</field>
          <field name="code">6663</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6II')])]" />
        </record>

        <record id="chart_at_template_6700" model="account.account.template">
          <field name="name">Sonstiger freiwilliger Sozialaufwand</field>
          <field name="code">6700</field>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6II')])]" />
        </record>

        <record id="chart_at_template_6900" model="account.account.template">
          <field name="name">Aufwandsstellenrechnung</field>
          <field name="code">6900</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
        </record>
        <record id="chart_at_template_7000" model="account.account.template">
          <field name="name">Abschreibungen auf aktivierte Aufwendungen für das Ingangsetzen und Erweitern eines Betriebes</field>
          <field name="code">7000</field>
          <field name="note">KZ 9130</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_depreciation" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_EBIT7I')])]" />
        </record>
        <record id="chart_at_template_7090" model="account.account.template">
          <field name="name">Abschreibungen vom Umlaufvermögen, soweit diese die im Unternehmen üblichen Abschreibungen übersteigen</field>
          <field name="code">7090</field>
          <field name="note">KZ 9140 (Bilanzierer gemäß §§ 4 Abs. 1 oder 5)</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_depreciation" />
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_EBIT7II')])]" />
        </record>
        <record id="chart_at_template_7600" model="account.account.template">
          <field name="name">Büromaterial und Drucksorten</field>
          <field name="code">7600</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_763" model="account.account.template">
          <field name="name">Fachliteratur und Zeitungen</field>
          <field name="code">7630</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7690" model="account.account.template">
          <field name="name">Spenden und Trinkgelder</field>
          <field name="code">7690</field>
          <field name="note">KZ 9200</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7770" model="account.account.template">
          <field name="name">Aus- und Fortbildung</field>
          <field name="code">7770</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7780" model="account.account.template">
          <field name="name">Mitgliedsbeiträge</field>
          <field name="code">7780</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7790" model="account.account.template">
          <field name="name">Spesen des Geldverkehrs</field>
          <field name="code">7790</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7820" model="account.account.template">
          <field name="name">Buchwert abgegangener Anlagen, ausgenommen Finanzanlagen</field>
          <field name="code">7820</field>
          <field name="note">KZ 9210</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7830" model="account.account.template">
          <field name="name">Verluste aus dem Abgang vom Anlagevermögen, ausgenommen Finanzanlagen</field>
          <field name="code">7830</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7860" model="account.account.template">
          <field name="name">Kursverluste aus Fremdwährungstransaktionen</field>
          <field name="code">7860</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7890" model="account.account.template">
          <field name="name">Skontoerträge auf sonstige betriebliche Aufwendungen</field>
          <field name="code">7890</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7900" model="account.account.template">
          <field name="name">Aufwandsstellenrechnung</field>
          <field name="code">7900</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
        </record>
        <record id="chart_at_template_7960" model="account.account.template">
          <field name="name">Herstellungskosten der zur Erzielung der Umsatzerlöse erbrachten Leistungen</field>
          <field name="code">7960</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
        </record>
        <record id="chart_at_template_7970" model="account.account.template">
          <field name="name">Vertriebskosten</field>
          <field name="code">7970</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
        </record>
        <record id="chart_at_template_7980" model="account.account.template">
          <field name="name">Verwaltungskosten</field>
          <field name="code">7980</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
        </record>
        <record id="chart_at_template_7990" model="account.account.template">
          <field name="name">Sonstige betriebliche Aufwendungen</field>
          <field name="code">7990</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_expenses" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_8140" model="account.account.template">
          <field name="name">Erlöse aus dem Abgang von Beteiligungen</field>
          <field name="code">8140</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_other_income" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN10')])]" />
        </record>
        <record id="chart_at_template_8150" model="account.account.template">
          <field name="name">Erlöse aus dem Abgang von sonstigen Finanzanlagen</field>
          <field name="code">8150</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_other_income" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN10')])]" />
        </record>
        <record id="chart_at_template_8160" model="account.account.template">
          <field name="name">Erlöse aus dem Abgang von Wertpapieren des Umlaufvermögens</field>
          <field name="code">8160</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_other_income" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN10')])]" />
        </record>
        <record id="chart_at_template_8170" model="account.account.template">
          <field name="name">Buchwert abgegangener Beteiligungen</field>
          <field name="code">8170</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_other_income" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN10')])]" />
        </record>
        <record id="chart_at_template_8180" model="account.account.template">
          <field name="name">Buchwert abgegangener sonstiger Finanzanlagen</field>
          <field name="code">8180</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_other_income" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN11')])]" />
        </record>
        <record id="chart_at_template_8190" model="account.account.template">
          <field name="name">Buchwert abgegangener Wertpapiere des Umlaufvermögens</field>
          <field name="code">8190</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_other_income" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN11')])]" />
        </record>
        <record id="chart_at_template_8200" model="account.account.template">
          <field name="name">Erträge aus dem Abgang von und der Zuschreibung zu Finanzanlagen</field>
          <field name="code">8200</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_other_income" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN10')])]" />
        </record>
        <record id="chart_at_template_8210" model="account.account.template">
          <field name="name">Erträge aus dem Abgang von und der Zuschreibung zu Wertpapieren des Umlaufvermögens</field>
          <field name="code">8210</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_other_income" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN11')])]" />
        </record>
        <record id="chart_at_template_8350" model="account.account.template">
          <field name="name">Nicht ausgenützte Lieferantenskonti</field>
          <field name="code">8350</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_other_income" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN12')])]" />
        </record>
        <record id="chart_at_template_8990" model="account.account.template">
          <field name="name">Gewinnabfuhr bzw. Verlustüberrechnung aus Ergebnisabführungsverträgen</field>
          <field name="code">8990</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_other_income" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN12')])]" />
        </record>
        <record id="chart_at_template_9190" model="account.account.template">
          <field name="name">Nicht eingeforderte ausstehende Einlagen</field>
          <field name="code">9190</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_equity" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_PAI')])]" />
        </record>
        <record id="chart_at_template_9390" model="account.account.template">
          <field name="name">Bilanzgewinn (-verlust)</field>
          <field name="code">9390</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_equity" />
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_PAIV')])]" />
        </record>
        <record id="chart_at_template_9800" model="account.account.template">
          <field name="name">Eröffnungsbilanz</field>
          <field name="code">9800</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_equity" />
        </record>
        <record id="chart_at_template_9850" model="account.account.template">
          <field name="name">Schlussbilanz</field>
          <field name="code">9850</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_equity" />
        </record>
        <record id="chart_at_template_9890" model="account.account.template">
          <field name="name">Gewinn- und Verlustrechnung</field>
          <field name="code">9890</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="user_type_id" ref="account.data_account_type_equity" />
        </record>
    </data>
</odoo>

```

## File: data\account_chart_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <menuitem id="menu_finance_austria" name="Austria" parent="account.menu_finance_reports" sequence="-1" groups="account.group_account_readonly"/>

    <data>
        <!-- Vorlagen: Kontenplan -->
        <record id="l10n_at_chart_template" model="account.chart.template">
            <field name="name">Einheitskontenrahmen Österreich 2010</field>
            <field name="complete_tax_set" eval="True" />
            <field name="visible" eval="True" />

            <field name="property_account_receivable_id" ref="chart_at_template_2000" />
            <field name="property_account_payable_id" ref="chart_at_template_3300" />

            <field name="default_pos_receivable_account_id" ref="chart_at_template_2099" />

            <field name="property_account_income_categ_id" ref="chart_at_template_4000" />
            <field name="property_account_expense_categ_id" ref="chart_at_template_5000" />

            <!--<field name="property_account_income_id" ref="chart_at_template_4000" />
            <field name="property_account_expense_id" ref="chart_at_template_5000" />-->

            <field name="property_stock_account_input_categ_id" ref="chart_at_template_3740" />
            <field name="property_stock_account_output_categ_id" ref="chart_at_template_5000" />

            <field name="property_stock_valuation_account_id" ref="chart_at_template_1600" />

            <field name="income_currency_exchange_account_id" ref="chart_at_template_4860" />
            <field name="expense_currency_exchange_account_id" ref="chart_at_template_7860" />

        </record>
    </data>
</odoo>

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_at.l10n_at_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_fiscal_position_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <!-- Vorlagen: Steuerliche Positionen -->

        <record id="fiscal_position_template_national" model="account.fiscal.position.template">
            <field name="name">National</field>
            <field name="auto_apply" eval="True" />
            <field name="country_id" ref="base.at" />
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
        </record>
        <record id="fiscal_position_template_eu" model="account.fiscal.position.template">
            <field name="name">Europäische Union</field>
            <field name="auto_apply" eval="True" />
            <field name="vat_required" eval="True" />
            <field name="country_group_id" ref="base.europe" />
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
        </record>
        <record id="fiscal_position_template_non_eu" model="account.fiscal.position.template">
            <field name="name">Drittstaaten</field>
            <field name="auto_apply" eval="True" />
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
        </record>

        <!-- Vorlagen: Steuerliche Positionen (Steuerzuordnung) -->

        <!-- Eurpäische Union (Binnenmarkt)) -->

        <record id="fiscal_position_tax_template_eu_code022" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu" />
            <field name="tax_src_id" ref="account_tax_template_sales_20_code022" />
            <field name="tax_dest_id" ref="account_tax_template_sales_eu_0_code017" />
        </record>
        <record id="fiscal_position_tax_template_eu_katalog022" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu" />
            <field name="tax_src_id" ref="account_tax_template_sales_20_katalog022" />
            <field name="tax_dest_id" ref="account_tax_template_sales_eu_0_services" />
        </record>
        <record id="fiscal_position_tax_template_eu_code029" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu" />
            <field name="tax_src_id" ref="account_tax_template_sales_10_code029" />
            <field name="tax_dest_id" ref="account_tax_template_sales_eu_0_code017" />
        </record>
        <record id="fiscal_position_tax_template_eu_code007" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu" />
            <field name="tax_src_id" ref="account_tax_template_sales_add7_code007" />
            <field name="tax_dest_id" ref="account_tax_template_sales_eu_0_code017" />
        </record>

        <record id="fiscal_position_tax_template_eu_vst_20_code060" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu" />
            <field name="tax_src_id" ref="account_tax_template_purchase_20_code060" />
            <field name="tax_dest_id" ref="account_tax_template_purchase_eu_20" />
        </record>
        <record id="fiscal_position_tax_template_eu_vst_20_code060K" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu" />
            <field name="tax_src_id" ref="account_tax_template_purchase_20_misc_code060" />
            <field name="tax_dest_id" ref="account_tax_template_purchase_rev_charge_1c" />
        </record>
        <record id="fiscal_position_tax_template_eu_vst_10_code060" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu" />
            <field name="tax_src_id" ref="account_tax_template_purchase_10_code060" />
            <field name="tax_dest_id" ref="account_tax_template_purchase_eu_10" />
        </record>
        <record id="fiscal_position_tax_template_eu_vst_19_code060" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_eu" />
            <field name="tax_src_id" ref="account_tax_template_purchase_19_code060" />
            <field name="tax_dest_id" ref="account_tax_template_purchase_eu_19" />
        </record>

        <!-- Drittstaaten -->

        <record id="fiscal_position_tax_template_non_eu_code022" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu" />
            <field name="tax_src_id" ref="account_tax_template_sales_20_code022" />
            <field name="tax_dest_id" ref="account_tax_template_sales_non_eu_0_code011" />
        </record>
        <record id="fiscal_position_tax_template_non_eu_code029" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu" />
            <field name="tax_src_id" ref="account_tax_template_sales_10_code029" />
            <field name="tax_dest_id" ref="account_tax_template_sales_non_eu_0_services" />
        </record>
        <record id="fiscal_position_tax_template_non_eu_code007" model="account.fiscal.position.tax.template">
            <field name="position_id" ref="fiscal_position_template_non_eu" />
            <field name="tax_src_id" ref="account_tax_template_sales_add7_code007" />
            <field name="tax_dest_id" ref="account_tax_template_sales_non_eu_0_code011" />
        </record>

        <!-- Vorlagen: Steuerliche Positionen (Finanzkontenzuordnung) -->

        <!-- Europäische Union (Binnenmarkt)) -->

        <record id="fiscal_position_account_template_eu_1" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_eu" />
            <field name="account_src_id" ref="chart_at_template_4000" />
            <field name="account_dest_id" ref="chart_at_template_4100" />
        </record>
        <record id="fiscal_position_account_template_eu_11" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_eu" />
            <field name="account_src_id" ref="chart_at_template_4001" />
            <field name="account_dest_id" ref="chart_at_template_4110" />
        </record>
        <record id="fiscal_position_account_template_eu_2" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_eu" />
            <field name="account_src_id" ref="chart_at_template_2000" />
            <field name="account_dest_id" ref="chart_at_template_2100" />
        </record>
        <record id="fiscal_position_account_template_eu_3" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_eu" />
            <field name="account_src_id" ref="chart_at_template_5000" />
            <field name="account_dest_id" ref="chart_at_template_5050" />
        </record>

        <!-- Drittstaaten -->

        <record id="fiscal_position_account_template_non_eu_1" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_non_eu" />
            <field name="account_src_id" ref="chart_at_template_4000" />
            <field name="account_dest_id" ref="chart_at_template_4200" />
        </record>
        <record id="fiscal_position_account_template_non_eu_2" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_non_eu" />
            <field name="account_src_id" ref="chart_at_template_2000" />
            <field name="account_dest_id" ref="chart_at_template_2150" />
        </record>
        <record id="fiscal_position_account_template_non_eu_3" model="account.fiscal.position.account.template">
            <field name="position_id" ref="fiscal_position_template_non_eu" />
            <field name="account_src_id" ref="chart_at_template_5000" />
            <field name="account_dest_id" ref="chart_at_template_5090" />
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report" model="account.tax.report">
        <field name="name">Tax Report</field>
        <field name="country_id" ref="base.at"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_sale_report_title" model="account.tax.report.line">
        <field name="name">4. Berechnung der Umsatzsteuer (U1/U30)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="10"/>
        <field name="formula">(AT_022_tax + AT_029_tax + AT_006_tax + AT_037_tax + AT_052_tax + AT_007_tax + AT_056 + AT_057 + AT_048 + AT_044 + AT_032 + AT_072_tax + AT_073_tax + AT_008_tax + AT_088_tax)</field>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_1" model="account.tax.report.line">
        <field name="name">4.1 Gesamtbetrag der Bemessungsgrundlage für Lieferungen und sonstige Leistungen [000]</field>
        <field name="tag_name">KZ 000</field>
        <field name="code">AT_000</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="20"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_2" model="account.tax.report.line">
        <field name="name">4.2 zuzüglich Eigenverbrauch (§ 1 Abs. 1 Z 2, § 3 Abs. 2 und § 3a Abs. 1a) [001]</field>
        <field name="tag_name">KZ 001</field>
        <field name="code">AT_001</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="30"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_3" model="account.tax.report.line">
        <field name="name">4.3 abzüglich Umsätze, für die die Steuerschuld gemäß § 19 Abs. 1 (Leistungsempfänger) [021]</field>
        <field name="tag_name">KZ 021</field>
        <field name="code">AT_021</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="40"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_4" model="account.tax.report.line">
        <field name="name">4.4 Summe</field>
        <field name="formula">AT_000 + AT_001 - AT_021</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="50"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_sale_01_report_title" model="account.tax.report.line">
        <field name="name">Davon steuerfrei MIT Vorsteuerabzug gemäß</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="60"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>
    <record id="tax_report_line_l10n_at_tva_line_4_5" model="account.tax.report.line">
        <field name="name">4.5 § 6 Abs. 1 Z 1 iVm § 7 (Ausfuhrlieferungen) [011]</field>
        <field name="tag_name">KZ 011</field>
        <field name="code">AT_011</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="70"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_01_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_6" model="account.tax.report.line">
        <field name="name">4.6 § 6 Abs. 1 Z 1 iVm § 8 (Lohnveredelungen) [012]</field>
        <field name="tag_name">KZ 012</field>
        <field name="code">AT_012</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="80"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_01_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_7" model="account.tax.report.line">
        <field name="name">4.7 § 6 Abs. 1 Z 2 bis 6 sowie § 23 Abs. 5 (Seeschifffahrt, Luftfahrt, ...) [015]</field>
    <field name="tag_name">KZ 015</field>
        <field name="code">AT_015</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="90"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_01_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_8" model="account.tax.report.line">
        <field name="name">4.8 Art. 6 Abs. 1 (innergemeinschaftliche Lieferungen ohne die nachstehend gesondert anzuführenden Fahrzeuglieferungen) [017]</field>
        <field name="tag_name">KZ 017</field>
        <field name="code">AT_017</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="100"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_01_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_9" model="account.tax.report.line">
        <field name="name">4.9 Art. 6 Abs. 1, sofern Lieferungen neuer Fahrzeuge an Abnehmer ohne UID-Nummer bzw. durch Fahrzeuglieferer gemäß
Art. 2 erfolgten. [018]</field>
    <field name="tag_name">KZ 018</field>
        <field name="code">AT_018</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="110"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_01_report_title"/>
    </record>
    <record id="tax_report_line_l10n_at_tva_sale_02_report_title" model="account.tax.report.line">
        <field name="name">Davon steuerfrei OHNE Vorsteuerabzug gemäß</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="120"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>
    <record id="tax_report_line_l10n_at_tva_line_4_10" model="account.tax.report.line">
        <field name="name">4.10 § 6 Abs. 1 Z 9 lit. a (Grundstücksumsätze) [019]</field>
        <field name="tag_name">KZ 019</field>
        <field name="code">AT_019</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="130"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_02_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_11" model="account.tax.report.line">
        <field name="name">4.11 § 6 Abs. 1 Z 27 (Kleinunternehmer) [016]</field>
        <field name="tag_name">KZ 016</field>
        <field name="code">AT_016</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="140"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_02_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_12" model="account.tax.report.line">
        <field name="name">4.12 § 6 Abs. 1 Z .. (übrige steuerfreie Umsätze ohne Vorsteuerabzug) [020]</field>
        <field name="tag_name">KZ 020</field>
        <field name="code">AT_020</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="150"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_02_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_13" model="account.tax.report.line">
        <field name="name">4.13 Gesamtbetrag der steuerpflichtigen Lieferungen, sonstigen Leistungen und Eigenverbrauch (einschließlich steuerpflichtiger Anzahlungen)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="160"/>
        <field name="formula">(AT_000 + AT_001 - AT_021) - AT_011 - AT_012 - AT_015 - AT_017 - AT_018 - AT_019 - AT_016 - AT_020</field>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>

    <record id="tax_report_line_at_base_title_umsatz_base_4_14_19" model="account.tax.report.line">
        <field name="name">Bemessungsgrundlage</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="170"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_14_base" model="account.tax.report.line">
        <field name="name">4.14 20% Normalsteuersatz [022]</field>
        <field name="tag_name">KZ 022 Bemessungsgrundlage</field>
        <field name="code">AT_022_base</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="180"/>
        <field name="parent_id" ref="tax_report_line_at_base_title_umsatz_base_4_14_19"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_15_base" model="account.tax.report.line">
        <field name="name">4.15 10% ermäßigter Steuersatz [029]</field>
        <field name="tag_name">KZ 029 Bemessungsgrundlage</field>
        <field name="code">AT_029_base</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="190"/>
        <field name="parent_id" ref="tax_report_line_at_base_title_umsatz_base_4_14_19"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_16_base" model="account.tax.report.line">
        <field name="name">4.16 13% ermäßigter Steuersatz [006]</field>
        <field name="tag_name">KZ 006 Bemessungsgrundlage</field>
        <field name="code">AT_006_base</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="200"/>
        <field name="parent_id" ref="tax_report_line_at_base_title_umsatz_base_4_14_19"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_17_base" model="account.tax.report.line">
        <field name="name">4.17 19% für Jungholz und Mittelberg [037]</field>
        <field name="tag_name">KZ 037 Bemessungsgrundlage</field>
        <field name="code">AT_037_base</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="210"/>
        <field name="parent_id" ref="tax_report_line_at_base_title_umsatz_base_4_14_19"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_18_base" model="account.tax.report.line">
        <field name="name">4.18 10% Zusatzsteuer für pauschalierte land- und forstwirtschaftliche Betriebe [052]</field>
        <field name="tag_name">KZ 052 Bemessungsgrundlage</field>
        <field name="code">AT_052_base</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="220"/>
        <field name="parent_id" ref="tax_report_line_at_base_title_umsatz_base_4_14_19"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_19_base" model="account.tax.report.line">
        <field name="name">4.19 7% Zusatzsteuer für pauschalierte land- und forstwirtschaftliche Betriebe [007]</field>
        <field name="tag_name">KZ 007 Bemessungsgrundlage</field>
        <field name="code">AT_007_base</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="230"/>
        <field name="parent_id" ref="tax_report_line_at_base_title_umsatz_base_4_14_19"/>
    </record>

    <record id="tax_report_line_at_tax_title_4_14_19" model="account.tax.report.line">
        <field name="name">Umsatzsteuer</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="240"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_14_tax" model="account.tax.report.line">
        <field name="name">4.14 20% Normalsteuersatz</field>
        <field name="tag_name">KZ 022 Umsatzsteuer</field>
        <field name="code">AT_022_tax</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="250"/>
        <field name="parent_id" ref="tax_report_line_at_tax_title_4_14_19"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_15_tax" model="account.tax.report.line">
        <field name="name">4.15 10% ermäßigter Steuersatz</field>
        <field name="tag_name">KZ 029 Umsatzsteuer</field>
        <field name="code">AT_029_tax</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="260"/>
        <field name="parent_id" ref="tax_report_line_at_tax_title_4_14_19"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_16_tax" model="account.tax.report.line">
        <field name="name">4.16 13% ermäßigter Steuersatz</field>
        <field name="tag_name">KZ 006 Umsatzsteuer</field>
        <field name="code">AT_006_tax</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="270"/>
        <field name="parent_id" ref="tax_report_line_at_tax_title_4_14_19"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_17_tax" model="account.tax.report.line">
        <field name="name">4.17 19% für Jungholz und Mittelberg</field>
        <field name="tag_name">KZ 037 Umsatzsteuer</field>
        <field name="code">AT_037_tax</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="280"/>
        <field name="parent_id" ref="tax_report_line_at_tax_title_4_14_19"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_18_tax" model="account.tax.report.line">
        <field name="name">4.18 10% Zusatzsteuer für pauschalierte land- und forstwirtschaftliche Betriebe</field>
        <field name="tag_name">KZ 052 Umsatzsteuer</field>
        <field name="code">AT_052_tax</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="290"/>
        <field name="parent_id" ref="tax_report_line_at_tax_title_4_14_19"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_19_tax" model="account.tax.report.line">
        <field name="name">4.19 7% Zusatzsteuer für pauschalierte land- und forstwirtschaftliche Betriebe</field>
        <field name="tag_name">KZ 007 Umsatzsteuer</field>
        <field name="code">AT_007_tax</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="300"/>
        <field name="parent_id" ref="tax_report_line_at_tax_title_4_14_19"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_20" model="account.tax.report.line">
        <field name="name">4.20 Steuerschuld gemäß § 11 Abs. 12 und 14, § 16 Abs. 2 sowie gemäß Art. 7 Abs. 4 [056]</field>
        <field name="tag_name">KZ 056</field>
        <field name="code">AT_056</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="310"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_21" model="account.tax.report.line">
        <field name="name">4.21 Steuerschuld gemäß § 19 Abs. 1 zweiter Satz, § 19 Abs. 1c, 1e sowie gemäß Art. 25 Abs. 5 [057]</field>
        <field name="tag_name">KZ 057</field>
        <field name="code">AT_057</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="330"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_22" model="account.tax.report.line">
        <field name="name">4.22 Steuerschuld gemäß § 19 Abs. 1a (Bauleistungen) [048]</field>
        <field name="tag_name">KZ 048</field>
        <field name="code">AT_048</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="340"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_23" model="account.tax.report.line">
        <field name="name">4.23 Steuerschuld gemäß § 19 Abs. 1b (Sicherungseigentum, ...) [044]</field>
        <field name="tag_name">KZ 044</field>
        <field name="code">AT_044</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="350"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_24" model="account.tax.report.line">
        <field name="name">4.24 Steuerschuld gemäß § 19 Abs. 1d (Schrott und Abfallstoffe) [032]</field>
        <field name="tag_name">KZ 032</field>
        <field name="code">AT_032</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="360"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_sale_03_report_title" model="account.tax.report.line">
        <field name="name">Innergemeinschaftliche Erwerb</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="365"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>
    <record id="tax_report_line_l10n_at_tva_line_4_25" model="account.tax.report.line">
        <field name="name">4.25 Gesamtbetrag der Bemessungsgrundlagen für innergemeinschaftliche Erwerbe [070]</field>
        <field name="tag_name">KZ 070</field>
        <field name="code">AT_070</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="370"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_03_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_26" model="account.tax.report.line">
        <field name="name">4.26 Davon steuerfrei gemäß Art. 6 Abs. 2 [071]</field>
        <field name="tag_name">KZ 071</field>
        <field name="code">AT_071</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="380"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_03_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_27" model="account.tax.report.line">
        <field name="name">4.27 Gesamtbetrag der steuerpflichtigen innergemeinschaftlichen Erwerbe</field>
        <field name="formula">AT_070 - AT_071</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="390"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_03_report_title"/>
    </record>
    <record id="tax_report_line_l10n_at_tva_sale_04_report_title" model="account.tax.report.line">
        <field name="name">Davon sind zu versteuern mit</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="395"/>
        <field name="formula">AT_072_base + AT_073_base + AT_008_base + AT_088_base</field>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>
    <record id="tax_report_line_at_base_title_umsatz_base_4_28_31" model="account.tax.report.line">
        <field name="name">Bemessungsgrundlage</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="400"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_04_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_28_base" model="account.tax.report.line">
        <field name="name">4.28 20% Normalsteuersatz [072]</field>
        <field name="tag_name">KZ 072 Bemessungsgrundlage</field>
        <field name="code">AT_072_base</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="400"/>
        <field name="parent_id" ref="tax_report_line_at_base_title_umsatz_base_4_28_31"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_29_base" model="account.tax.report.line">
        <field name="name">4.29 10% ermäßigter Steuersatz [073]</field>
        <field name="tag_name">KZ 073 Bemessungsgrundlage</field>
        <field name="code">AT_073_base</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="420"/>
        <field name="parent_id" ref="tax_report_line_at_base_title_umsatz_base_4_28_31"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_30_base" model="account.tax.report.line">
        <field name="name">4.30 13% ermäßigter Steuersatz [008]</field>
        <field name="tag_name">KZ 008 Bemessungsgrundlage</field>
        <field name="code">AT_008_base</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="450"/>
        <field name="parent_id" ref="tax_report_line_at_base_title_umsatz_base_4_28_31"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_31_base" model="account.tax.report.line">
        <field name="name">4.31 19% für Jungholz und Mittelberg [088]</field>
        <field name="tag_name">KZ 088 Bemessungsgrundlage</field>
        <field name="code">AT_088_base</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="550"/>
        <field name="parent_id" ref="tax_report_line_at_base_title_umsatz_base_4_28_31"/>
    </record>

    <record id="tax_report_line_at_tax_title_4_28_31" model="account.tax.report.line">
        <field name="name">Umsatzsteuer</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="410"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_04_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_28_tax" model="account.tax.report.line">
        <field name="name">4.28 20% Normalsteuersatz</field>
        <field name="tag_name">KZ 072 Umsatzsteuer</field>
        <field name="code">AT_072_tax</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="410"/>
        <field name="parent_id" ref="tax_report_line_at_tax_title_4_28_31"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_29_tax" model="account.tax.report.line">
        <field name="name">4.29 10% ermäßigter Steuersatz</field>
        <field name="tag_name">KZ 073 Umsatzsteuer</field>
        <field name="code">AT_073_tax</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="430"/>
        <field name="parent_id" ref="tax_report_line_at_tax_title_4_28_31"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_30_tax" model="account.tax.report.line">
        <field name="name">4.30 13% ermäßigter Steuersatz</field>
        <field name="tag_name">KZ 008 Umsatzsteuer</field>
        <field name="code">AT_008_tax</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="500"/>
        <field name="parent_id" ref="tax_report_line_at_tax_title_4_28_31"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_31_tax" model="account.tax.report.line">
        <field name="name">4.31 19% für Jungholz und Mittelberg</field>
        <field name="tag_name">KZ 088 Umsatzsteuer</field>
        <field name="code">AT_088_tax</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="600"/>
        <field name="parent_id" ref="tax_report_line_at_tax_title_4_28_31"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_sale_05_report_title" model="account.tax.report.line">
        <field name="name">Nicht zu versteuernde Erwerbe</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="610"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_report_title"/>
    </record>
    <record id="tax_report_line_l10n_at_tva_line_4_32" model="account.tax.report.line">
        <field name="name">4.32 Erwerbe gemäß Art. 3 Abs. 8 zweiter Satz, die im Mitgliedstaat des Bestimmungslandes besteuert worden sind [076]</field>
        <field name="tag_name">KZ 076</field>
        <field name="code">AT_076</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="650"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_05_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_4_33" model="account.tax.report.line">
        <field name="name">4.33 Erwerbe gemäß Art. 3 Abs. 8 zweiter Satz, die gemäß Art. 25 Abs. 2 im Inland als besteuert gelten [077]</field>
        <field name="tag_name">KZ 077</field>
        <field name="code">AT_077</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="700"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_sale_05_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_purchase_report_title" model="account.tax.report.line">
    <field name="name">5. Berechnung der abziehbaren Vorsteuer</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="710"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_5_1" model="account.tax.report.line">
        <field name="name">5.1 Gesamtbetrag der Vorsteuern (ohne die nachstehend gesondert anzuführenden Beträge) [060]</field>
        <field name="tag_name">KZ 060</field>
        <field name="code">AT_060</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="750"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_purchase_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_5_2" model="account.tax.report.line">
        <field name="name">5.2 Vorsteuern betreffend die entrichtete Einfuhrumsatzsteuer (§ 12 Abs. 1 Z 2 lit. a) [061]</field>
        <field name="tag_name">KZ 061</field>
        <field name="code">AT_061</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="800"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_purchase_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_5_3" model="account.tax.report.line">
        <field name="name">5.3 Vorsteuern betreffend die geschuldete, auf dem Abgabenkonto verbuchte Einfuhrumsatzsteuer (§ 12 Abs. 1 Z 2 lit. b) [083]</field>
        <field name="tag_name">KZ 083</field>
        <field name="code">AT_083</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="850"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_purchase_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_5_4" model="account.tax.report.line">
        <field name="name">5.4 Vorsteuern aus dem innergemeinschaftlichen Erwerb [065]</field>
        <field name="tag_name">KZ 065</field>
        <field name="code">AT_065</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="900"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_purchase_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_5_5" model="account.tax.report.line">
        <field name="name">5.5 Vorsteuern betreffend die Steuerschuld gemäß § 19 Abs. 1 zweiter Satz, § 19 Abs. 1c, 1e sowie gemäß Art. 25 Abs. 5 [066]</field>
        <field name="tag_name">KZ 066</field>
        <field name="code">AT_066</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="950"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_purchase_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_5_6" model="account.tax.report.line">
        <field name="name">5.6 Vorsteuern betreffend die Steuerschuld gemäß § 19 Abs. 1a (Bauleistungen) [082]</field>
        <field name="tag_name">KZ 082</field>
        <field name="code">AT_082</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="960"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_purchase_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_5_7" model="account.tax.report.line">
        <field name="name">5.7 Vorsteuern betreffend die Steuerschuld gemäß § 19 Abs. 1b (Sicherungseigentum, ...) [087]</field>
        <field name="tag_name">KZ 087</field>
        <field name="code">AT_087</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="970"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_purchase_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_5_8" model="account.tax.report.line">
        <field name="name">5.8 Vorsteuern betreffend die Steuerschuld gemäß § 19 Abs. 1d (Schrott und Abfallstoffe) [089]</field>
        <field name="tag_name">KZ 089</field>
        <field name="code">AT_089</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="980"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_purchase_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_5_9" model="account.tax.report.line">
        <field name="name">5.9 Vorsteuern für innergemeinschaftliche Lieferungen neuer Fahrzeuge von Fahrzeuglieferern gemäß Art. 2 [064]</field>
        <field name="tag_name">KZ 064</field>
        <field name="code">AT_064</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="990"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_purchase_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_5_10" model="account.tax.report.line">
        <field name="name">5.10 Davon nicht abzugsfähig gemäß § 12 Abs. 3 iVm Abs. 4 und 5 [062]</field>
        <field name="tag_name">KZ 062</field>
        <field name="code">AT_062</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1000"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_purchase_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_5_11" model="account.tax.report.line">
        <field name="name">5.11 Berichtigung gemäß § 12 Abs. 10 und 11 [063]</field>
        <field name="tag_name">KZ 063</field>
        <field name="code">AT_063</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1010"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_purchase_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_5_12" model="account.tax.report.line">
        <field name="name">5.12 Berichtigung gemäß § 16 [067]</field>
        <field name="tag_name">KZ 067</field>
        <field name="code">AT_067</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1020"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_purchase_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_5_13" model="account.tax.report.line">
        <field name="name">5.13 Gesamtbetrag der abziehbaren Vorsteuer</field>
        <field name="formula">AT_060 + AT_061 + AT_083 + AT_065 + AT_066 + AT_082 + AT_087 + AT_089 + AT_064 + AT_062 + AT_063 + AT_067</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1030"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_purchase_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_final_report_title" model="account.tax.report.line">
        <field name="name">6. Sonstige Berichtigungen</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1040"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_6" model="account.tax.report.line">
        <field name="name">Sonstige Berichtigungen [090]</field>
        <field name="tag_name">KZ 090</field>
        <field name="code">AT_090</field>
    <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1050"/>
        <field name="parent_id" ref="tax_report_line_l10n_at_tva_final_report_title"/>
    </record>

    <record id="tax_report_line_l10n_at_tva_line_7" model="account.tax.report.line">
        <field name="name">7. Zahllast (-) bzw. Gutschrift/Überschuss (+) [095]</field>
        <field name="formula">(AT_022_tax + AT_029_tax + AT_006_tax + AT_037_tax + AT_052_tax + AT_007_tax + AT_056 + AT_057 + AT_048 + AT_044 + AT_032 + AT_072_tax + AT_073_tax + AT_008_tax + AT_088_tax + AT_060 + AT_061 + AT_083 + AT_065 + AT_066 + AT_082 + AT_087 + AT_089 + AT_064 + AT_062 + AT_063 + AT_067) + AT_090</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1060"/>
    </record>
</odoo>

```

## File: data\account_tax_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data noupdate="1">
        <!-- Account Tax Group -->
        <record id="tax_group_0" model="account.tax.group">
            <field name="name">VAT 0%</field>
        </record>
        <record id="tax_group_10" model="account.tax.group">
            <field name="name">VAT 10%</field>
        </record>
        <record id="tax_group_13" model="account.tax.group">
            <field name="name">VAT 13%</field>
        </record>
        <record id="tax_group_20" model="account.tax.group">
            <field name="name">VAT 20%</field>
        </record>
    </data>
    <data>
        <!-- Vorlagen: Steuerdefinition -->
        <!-- Vorlagen: Verkauf (Umsatzsteuer) -->
        <!-- Lieferungen, sonstige Leistungen und Eigenverbrauch +000,+001,-021 -->
        <!-- Umsätze, für die die Steuerschuld gemäß § 19 Abs. 1 zweiter
            Satz sowie gemäß § 19 Abs. 1a, 1b, 1c, 1d und 1e (Umsätze ab 01.07.2010)
            auf den Leistungsempfänger übergegangen ist. -->
        <record id="account_tax_template_sales_rev_charge_0_code021" model="account.tax.template">
            <field name="name">UST_021 Steuerschuld betrifft Leistungsempfänger</field>
            <field name="description">USt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">400</field>
            <field name="type_tax_use">sale</field>
            <field eval="0.0" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_3')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_3')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_3')],
                }),

                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_3')],
                }),
            ]"/>
        </record>
        <!-- Steuerfreier Umsatz +011, +012, +015, +017, +018, +019, +016, +020 -->
        <!-- Ausfuhrlieferungen (§ 6 Abs. 1 Z 1 iVm § 7) -->
        <record id="account_tax_template_sales_non_eu_0_code011" model="account.tax.template">
            <field name="name">UST_011 Export 0%</field>
            <field name="description">USt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">300</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_5'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_5'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>
        <!-- Lohnveredelungen (§ 6 Abs. 1 Z 1 iVm § 8) -->
        <record id="account_tax_template_sales_non_eu_0_code012" model="account.tax.template">
            <field name="name">UST_012 Lohnveredelung 0%</field>
            <field name="description">USt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">200</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_6'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_6'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>
        <!-- § 6 Abs. 1 Z 2 bis 6 sowie § 23 Abs. 5 (Seeschifffahrt, Luftfahrt,
            grenzüberschreitende Personenbeförderung, Diplomaten, Reisevorleistungen
            im Drittlandsgebiet usw.) -->
        <record id="account_tax_template_sales_non_eu_0_code015" model="account.tax.template">
            <field name="name">UST_015 Export 0% (§ 6 Abs. 1 Z 2 bis 6)</field>
            <field name="description">USt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">300</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_7'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_7'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>
        <!-- Innergemeinschaftliche Lieferungen ohne Fahrzeuglieferungen
            (Art. 6 Abs. 1) -->
        <record id="account_tax_template_sales_eu_0_code017" model="account.tax.template">
            <field name="name">UST_017 IGL 0% (ohne Art. 6 Abs. 1)</field>
            <field name="description">USt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">200</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_8'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_8'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>
        <!-- EU Fahrzeuglieferungen (Art. 6 Abs. 1) -->
        <record id="account_tax_template_sales_eu_0_code018" model="account.tax.template">
            <field name="name">UST_018 IGL 0% (Art. 6 Abs. 1)</field>
            <field name="description">USt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">200</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_9'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_9'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>
        <!-- § 6 Abs. 1 Z 9 lit. a (Grundstücksumsätze) -->
        <record id="account_tax_template_sales_0_code019" model="account.tax.template">
            <field name="name">UST_019 Grundstücksumsätze 0% (§ 6 Abs. 1 Z 9 lit. a)</field>
            <field name="description">USt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_10'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_10'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>
        <!-- § 6 Abs. 1 Z 27 (Kleinunternehmer) -->
        <record id="account_tax_template_sales_0_code016" model="account.tax.template">
            <field name="name">UST_016 Kleinunternehmer 0% (§ 6 Abs. 1 Z 27)</field>
            <field name="description">USt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_11'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_11'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>
        <!-- § 6 Abs. 1 Z 1, 7-8, 10-26, 28 (übrige steuerfreie Umsätze ohne
            Vorsteuerabzug) -->
        <record id="account_tax_template_sales_0_code020" model="account.tax.template">
            <field name="name">UST_020 Übrige steuerfreie Umsätze 0%</field>
            <field name="description">USt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_12'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_12'), ref('tax_report_line_l10n_at_tva_line_4_1')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>
        <!-- zu versteuernder Umsatz +022, +029, +006, +037, +052, +007 -->
        <!-- Soll-Umsatzsteuern (=normale Umsatzsteuer) -->
        <record id="account_tax_template_sales_20_code022" model="account.tax.template">
            <field name="name">UST_022 Normalsteuersatz 20%</field>
            <field name="description">USt. 20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="20" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_1'),
                    ref('tax_report_line_l10n_at_tva_line_4_14_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3500'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_14_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_1'),
                    ref('tax_report_line_l10n_at_tva_line_4_14_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3500'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_14_tax')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_20" />
        </record>
        <record id="account_tax_template_sales_20_katalog022" model="account.tax.template">
            <field name="name">UST_022 Normalsteuersatz 20% (Sonstige Leistungen)</field>
            <field name="description">USt. 20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="20" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_1'),
                    ref('tax_report_line_l10n_at_tva_line_4_14_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3500'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_14_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_1'),
                    ref('tax_report_line_l10n_at_tva_line_4_14_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3500'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_14_tax')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_20" />
        </record>
        <record id="account_tax_template_sales_10_code029" model="account.tax.template">
            <field name="name">UST_029 ermäßigter Steuersatz 10%</field>
            <field name="description">USt. 10%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="10" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_1'),
                    ref('tax_report_line_l10n_at_tva_line_4_15_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3501'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_15_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_1'),
                    ref('tax_report_line_l10n_at_tva_line_4_15_base')],
                }),

                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3501'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_15_tax')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_10" />
        </record>
        <record id="account_tax_template_sales_13_code006" model="account.tax.template">
            <field name="name">UST_006 ermäßigter Steuersatz 13%</field>
            <field name="description">USt. 13%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="13" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_1'),
                    ref('tax_report_line_l10n_at_tva_line_4_16_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3502'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_16_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_1'),
                    ref('tax_report_line_l10n_at_tva_line_4_16_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3502'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_16_tax')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_13" />
        </record>
        <!-- 19% für Gemeinden Jungholz und Mittelberg -->
        <record id="account_tax_template_sales_19_code037" model="account.tax.template">
            <field name="name">UST_037 Steuersatz 19%</field>
            <field name="description">USt. 19%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="19" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_1'),
                    ref('tax_report_line_l10n_at_tva_line_4_17_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_17_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_1'),
                    ref('tax_report_line_l10n_at_tva_line_4_17_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_17_tax')],
                }),
            ]"/>
        </record>
        <!-- 10% Zusatzsteuer für pauschalierte land- und forstwirtschaftliche
            Betriebe -->
        <record id="account_tax_template_sales_add10_code052" model="account.tax.template">
            <field name="name">UST_052 Zusatzsteuersatz 10% (LWB/FWB)</field>
            <field name="description">USt. 10%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="10" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_1'),
                    ref('tax_report_line_l10n_at_tva_line_4_18_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_18_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_1'),
                    ref('tax_report_line_l10n_at_tva_line_4_18_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_18_tax')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_add7_code007" model="account.tax.template">
            <field name="name">UST_007 Zusatzsteuersatz 7% (LWB/FWB)</field>
            <field name="description">USt. 7%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="7" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_1'),
                    ref('tax_report_line_l10n_at_tva_line_4_19_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_19_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_1'),
                    ref('tax_report_line_l10n_at_tva_line_4_19_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_19_tax')],
                }),
            ]"/>
        </record>
        <!-- zu versteuernder Umsatz (Eigenverbrauch) +022, +029, +006, +037,
            +052, +007 -->
        <record id="account_tax_template_sales_self_20_code022" model="account.tax.template">
            <field name="name">UST_022 Normalsteuersatz 20% (Eigenverbrauch)</field>
            <field name="description">USt. 20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="20" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_2'),
                    ref('tax_report_line_l10n_at_tva_line_4_14_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3500'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_14_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_2'),
                    ref('tax_report_line_l10n_at_tva_line_4_14_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3500'),
                  'plus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_14_tax')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_20" />
        </record>
        <record id="account_tax_template_sales_self_10_code029" model="account.tax.template">
            <field name="name">UST_029 ermäßigter Steuersatz 10% (Eigenverbrauch)</field>
            <field name="description">USt. 10%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="10" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_2'),
                    ref('tax_report_line_l10n_at_tva_line_4_15_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3501'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_15_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_2'),
                    ref('tax_report_line_l10n_at_tva_line_4_15_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3501'),
                  'plus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_15_tax')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_10" />
        </record>
        <record id="account_tax_template_sales_self_19_code037" model="account.tax.template">
            <field name="name">UST_037 Steuersatz 19% (Eigenverbrauch) </field>
            <field name="description">USt. 19%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="19" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_2'),
                    ref('tax_report_line_l10n_at_tva_line_4_17_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_17_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_2'),
                    ref('tax_report_line_l10n_at_tva_line_4_17_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_17_tax')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_self_add10_code052" model="account.tax.template">
            <field name="name">UST_052 Zusatzsteuersatz 10% (LWB/FWB - Eigenverbrauch)</field>
            <field name="description">USt. 10%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="10" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_2'),
                    ref('tax_report_line_l10n_at_tva_line_4_18_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_18_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_2'),
                    ref('tax_report_line_l10n_at_tva_line_4_18_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_18_tax')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_self_add7_code007" model="account.tax.template">
            <field name="name">UST_007 Zusatzsteuersatz 7% (LWB/FWB - Eigenverbrauch)</field>
            <field name="description">USt. 7%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="7" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_2'),
                    ref('tax_report_line_l10n_at_tva_line_4_19_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_19_tax')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_2'),
                    ref('tax_report_line_l10n_at_tva_line_4_19_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_19_tax')],
                }),
            ]"/>
        </record>
        <!-- Übergegangene Steuerschuld +056, +057, +048, +044, +032 -->
        <!-- Steuerschuld gemäß § 11 Abs. 12 und 14, § 16 Abs. 2 sowie gemäß
            Art. 7 Abs. 4 -->
        <record id="account_tax_template_purchase_tax_invoiced_accepted_code056" model="account.tax.template">
            <field name="name">UST_056 Tax invoiced accepted (§ 11 Abs. 12 und 14, § 16 Abs. 2 sowie gemäß Art. 7 Abs. 4)</field>
            <field name="description">USt. 20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">100</field>
            <field name="type_tax_use">purchase</field>
            <field eval="20" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_20')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_20')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_20" />
        </record>

        <record id="account_tax_template_sales_eu_0_services" model="account.tax.template">
            <field name="name">UST_EU Dienstleistung (Sonstige Leistungen) 0%</field>
            <field name="description">USt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">200</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),

                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>

        <record id="account_tax_template_sales_non_eu_0_services" model="account.tax.template">
            <field name="name">UST_NON_EU Dienstleistung (Drittstaaten) 0%</field>
            <field name="description">USt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">300</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),

                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>

        <!-- Innergemeinschaftlicher Erwerb (Erwerbssteuer) 070 abzgl. 071 -->
        <!-- Davon steuerfrei gemäß Art. 6 Abs. 2 (IGE-UST) -->
        <record id="account_tax_template_purchase_eu_0_code071" model="account.tax.template">
            <field name="name">UST_071 IGE 0% (Art. 6 Abs. 2)</field>
            <field name="description">USt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">200</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_25'),
                    ref('tax_report_line_l10n_at_tva_line_4_26')]
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_25'),
                    ref('tax_report_line_l10n_at_tva_line_4_26')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
        </record>

        <record id="account_tax_template_purchase_eu_20" model="account.tax.template">
            <field name="name">IGE 20%</field>
            <field name="description">IGE 20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">500</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_25'),
                    ref('tax_report_line_l10n_at_tva_line_4_28_base')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3511'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_28_tax')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2511'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_4')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids':[ref('tax_report_line_l10n_at_tva_line_4_25'),
                    ref('tax_report_line_l10n_at_tva_line_4_28_base')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3511'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_28_tax')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2511'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_4')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>

        <record id="account_tax_template_purchase_eu_10" model="account.tax.template">
            <field name="name">IGE 10%</field>
            <field name="description">IGE 10%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">500</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="amount">10</field>
            <field name="amount_type">percent</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_25'),
                    ref('tax_report_line_l10n_at_tva_line_4_29_base')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3512'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_29_tax')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2512'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_4')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_25'),
                    ref('tax_report_line_l10n_at_tva_line_4_29_base')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3512'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_29_tax')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2512'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_4')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>

        <record id="account_tax_template_purchase_eu_13" model="account.tax.template">
            <field name="name">IGE 13%</field>
            <field name="description">IGE 13%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">500</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="amount">13</field>
            <field name="amount_type">percent</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_25'),
                    ref('tax_report_line_l10n_at_tva_line_4_30_base')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3513'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_30_tax')]
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2513'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_4')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_25'),
                    ref('tax_report_line_l10n_at_tva_line_4_30_base')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3513'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_30_tax')]
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2513'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_4')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>

        <record id="account_tax_template_purchase_eu_19" model="account.tax.template">
            <field name="name">IGE 19%</field>
            <field name="description">IGE 19%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">500</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="amount">19</field>
            <field name="amount_type">group</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_25'),
                    ref('tax_report_line_l10n_at_tva_line_4_31_base')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3511'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_31_tax')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_4')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_25'),
                    ref('tax_report_line_l10n_at_tva_line_4_31_base')],
                }),

                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3511'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_31_tax')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_4')],
                }),
            ]"/>
        </record>
                <!-- Reverse Charge (§ 19 Abs. 1a - Bauleistungen) -->
        <record id="account_tax_template_purchase_rev_charge_1a" model="account.tax.template">
            <field name="name">Reverse Charge 20% (§ 19 Abs. 1a - Bauleistungen)</field>
            <field name="description">RC 20% § 19 Abs. 1a</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">550</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_22')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_6')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_22')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_6')],
                }),
            ]"/>
            <field name="price_include" eval="0" />
        </record>

        <!-- Reverse Charge gemäß § 19 Abs. 1b (Sicherungseigentum, Vorbehaltseigentum  und Grundstücke im Zwangsversteigerungsverfahren) -->
        <record id="account_tax_template_purchase_rev_charge_1b" model="account.tax.template">
            <field name="name">Reverse Charge 20% (§ 19 Abs. 1b - Sicherungseigentum)</field>
            <field name="description">RC 20% § 19 Abs. 1b</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">550</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_23')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_7')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_23')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_7')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>

        <!-- Reverse Charge gemäß § 19 Abs. 1 zweiter Satz, § 19 Abs. 1c sowie  gemäß Art. 25 Abs. 5 -->
        <record id="account_tax_template_purchase_rev_charge_1c" model="account.tax.template">
            <field name="name">Reverse Charge 20% (§ 19 Abs. 1c - Sonstige Leistungen)</field>
            <field name="description">RC 20% § 19 Abs. 1c</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">550</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_21')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_5')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_21')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_5')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>

        <!-- Reverse Charge gemäß § 19 Abs. 1d (Schrott und Abfallstoffe, Verordnung
            BGBl. II Nr. 129/2007; Videospielkonsolen, Laptops, Tablet-Computer, Gas
            und Elektrizität, Gas- und Elektrizitätszertifikate, Metalle, Anlagegold,
            Verordnung BGBl. II Nr. 369/2013) -->
        <record id="account_tax_template_purchase_rev_charge_1d" model="account.tax.template">
            <field name="name">Reverse Charge 20% (§ 19 Abs. 1d - Schrott und Abfallstoffe)</field>
            <field name="description">RC 20% § 19 Abs. 1d</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">550</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_24')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_8')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_24')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_8')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_0" />
        </record>
        <record id="account_tax_template_purchase_eu_xx_code076" model="account.tax.template">
            <field name="name">Erwerbe gemäß Art. 3 Abs. 8 zweiter Satz, die im Mitgliedstaat des Bestimmungslandes besteuert worden sind (IGE-UST)</field>
            <field name="description">UST_076 IGE (im Bestimmungsland besteuert)</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field eval="0.0" name="amount" />
            <field name="sequence">200</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_32'),
                    ref('tax_report_line_l10n_at_tva_line_4_33')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_32'),
                    ref('tax_report_line_l10n_at_tva_line_4_33')],
                }),

                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
        </record>
        <record id="account_tax_template_purchase_eu_xx_code077" model="account.tax.template">
            <field name="name">Erwerbe gemäß Art. 3 Abs. 8 zweiter Satz, die gemäß Art. 25 Abs. 2 im Inland als besteuert gelten (IGE-UST)</field>
            <field name="description">UST_077 IGE (im Inland besteuert)</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field eval="0.0" name="amount" />
            <field name="sequence">200</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_33'),
                    ref('tax_report_line_l10n_at_tva_line_4_17_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_33')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_33'),
                    ref('tax_report_line_l10n_at_tva_line_4_17_base')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_33')],
                }),
            ]"/>
        </record>

        <!-- Vorlagen: Einkauf (Vorsteuer) -->
        <!-- Vorsteuern -060, -061, -083, -065(070), -066(057), -082, -087,
            -089, !-064, +062, -063, -067! -->
        <record id="account_tax_template_purchase_20_code060" model="account.tax.template">
            <field name="name">VST_060 Normalsteuersatz 20%</field>
            <field name="description">VSt. 20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field eval="20" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2500'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_1')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2500'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_1')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_20" />
        </record>
        <record
            id="account_tax_template_purchase_20_misc_code060" model="account.tax.template">
            <field name="name">VST_060 sonstige Leistungen 20%</field>
            <field name="description">VSt. 20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field eval="20" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2500'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_1')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2500'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_1')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_20" />
        </record>
        <record id="account_tax_template_purchase_10_code060" model="account.tax.template">
            <field name="name">VST_060 ermäßigter Steuersatz 10%</field>
            <field name="description">VSt. 10%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field eval="10" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2501'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_1')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2501'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_1')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_10" />
        </record>
        <record id="account_tax_template_purchase_13_code060" model="account.tax.template">
            <field name="name">VST_060 ermäßigter Steuersatz 13%</field>
            <field name="description">VSt. 13%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field eval="13" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2502'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_1')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2502'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_1')],
                }),
            ]"/>
            <field name="tax_group_id" ref="tax_group_13" />
        </record>
        <record id="account_tax_template_purchase_19_code060" model="account.tax.template">
            <field name="name">VST_060 Jungholz und Mittelberg 19%</field>
            <field name="description">VSt. 19%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field eval="19" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_1')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_1')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_purchase_12_code060" model="account.tax.template">
            <field name="name">VST_060 Weineinkauf 12% (LWB)</field>
            <field name="description">VSt. 12%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field eval="12" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_1')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_1')],
                }),
            ]"/>
        </record>
        <!-- Vorsteuern betreffend die entrichtete Einfuhrumsatzsteuer (§
            12 Abs. 1 Z 2 lit. a) -->
        <record id="account_tax_template_purchase_xx_code061" model="account.tax.template">
            <field name="name">VST_061 entrichtete EUst (§ 12 Abs. 1 Z 2 lit. a)</field>
            <field name="description">VSt. 20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field eval="20" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_2')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_2')],
                }),
            ]"/>
        </record>
        <!-- Vorsteuern betreffend die geschuldete, auf dem Abgabenkonto
            verbuchte Einfuhrumsatzsteuer (§ 12 Abs. 1 Z 2 lit. b) -->
        <record id="account_tax_template_purchase_xx_code083" model="account.tax.template">
            <field name="name">VST_083 verbuchte EUst. (§ 12 Abs. 1 Z 2 lit. b)</field>
            <field name="description">VSt. 20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">400</field>
            <field name="type_tax_use">none</field>
            <field eval="20" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2515'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_3')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3515'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_3')],
                }),
            ]"/>
        </record>

        <!-- Berichtigungen -063, -067, -090 ! -->
        <!-- Berichtigung gemäß § 12 Abs. 10 und 11 -->
        <record
            id="account_tax_template_purchase_correct_code063" model="account.tax.template">
            <field name="name">VST_063 (§12 Abs. 10 und 11 - Berichtigung)</field>
            <field name="description">VSt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field eval="0.00" name="amount" />
            <field name="sequence">400</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
        </record>
        <!-- Berichtigung gemäß § 16 -->
        <record
            id="account_tax_template_purchase_correct_code067" model="account.tax.template">
            <field name="name">VST_067 (§ 16 - Berichtigung)</field>
            <field name="description">VSt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field eval="0.00" name="amount" />
            <field name="sequence">400</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_12')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_12')],
                }),
            ]"/>
        </record>
        <!-- Sonstige Berichtigungen -->
        <record id="account_tax_template_purchase_correct_code090" model="account.tax.template">
            <field name="name">VST_090 (Sonstige Berichtigungen)</field>
            <field name="description">VSt. 0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field eval="0.00" name="amount" />
            <field name="sequence">600</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_6')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_6')],
                }),
            ]"/>
        </record>
        <!-- Vorsteuern 027, 028 in 060/065 -->
        <!-- Vorsteuern betreffend KFZ nach EKR 063, 064, 732-733 und 744-747 -->
        <record id="account_tax_template_purchase_cars_buildings_code027" model="account.tax.template">
            <field name="name">VST_027 betreffend KFZ nach EKR 063, 064, 732-733 und 744-747</field>
            <field name="description">VSt. 20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field eval="20" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
        </record>
        <!-- Vorsteuern betreffend Gebäude nach EKR 030-037 und 070, 071 -->
        <record id="account_tax_template_purchase_cars_buildings_code028" model="account.tax.template">
            <field name="name">VST_028 betreffend Gebäude nach EKR 030-037 und 070, 071</field>
            <field name="description">VSt. 20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field eval="20" name="amount" />
            <field name="sequence">400</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
        </record>
        <!-- Innergemeinschaftlicher Erwerb 070 abzgl. 071 ! -->
        <record id="account_tax_template_purchase_eu_0_vst_071" model="account.tax.template">
            <field name="name">VST_071 IGE 0%</field>
            <field name="description">VST_071 IGE 0% (Art. 6 Abs. 2)</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field eval="0" name="amount" />
            <field name="sequence">500</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'plus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_5_4')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'minus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_5_4')],
                }),
            ]"/>
        </record>

        <!-- Erwerbe gemäß Art. 3 Abs. 8 zweiter Satz, die im Mitgliedstaat
            des Bestimmungslandes besteuert worden sind (IGE-VST) -->
        <record id="account_tax_template_purchase_eu_xx_vst_076" model="account.tax.template">
            <field name="name">VST_076 IGE (im Bestimmungsland besteuert)</field>
            <field name="description">VSt. 20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">500</field>
            <field name="type_tax_use">purchase</field>
            <field eval="20" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_line_ids': [ref('tax_report_line_l10n_at_tva_line_4_32'),
                    ref('tax_report_line_l10n_at_tva_line_4_33')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'plus_report_line_ids':  [ref('tax_report_line_l10n_at_tva_line_4_32'),
                    ref('tax_report_line_l10n_at_tva_line_4_33')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
        </record>
        <!-- Erwerbe gemäß Art. 3 Abs. 8 zweiter Satz, die gemäß Art. 25
            Abs. 2 im Inland als besteuert gelten (IGE-VST) -->
        <record id="account_tax_template_purchase_eu_xx_vst_077" model="account.tax.template">
            <field name="name">VST_077 IGE (im Inland besteuert)</field>
            <field name="description">VSt. 20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template" />
            <field name="sequence">500</field>
            <field name="type_tax_use">purchase</field>
            <field eval="20" name="amount" />
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="active" eval="False" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
        </record>
    </data>
</odoo>

```

## File: data\res.country.state.csv

```csv
"id","country_id:id","name","code"
state_at_1,base.at,"Burgenland","1"
state_at_2,base.at,"Kärnten","2"
state_at_3,base.at,"Niederösterreich","3"
state_at_4,base.at,"Oberösterreich","4"
state_at_5,base.at,"Salzburg","5"
state_at_6,base.at,"Steiermark","6"
state_at_7,base.at,"Tirol","7"
state_at_8,base.at,"Vorarlberg","8"
state_at_9,base.at,"Wien","9"

```

