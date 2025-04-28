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
    "version": "3.1.1",
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
        "l10n_din5008",
    ],
    "data": [
        'data/res.country.state.csv',
        'data/account_account_tag.xml',
        'data/account_account_template.xml',
        'data/account_chart_template.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_template.xml',
        'data/account_fiscal_position_template.xml',
        'data/account_chart_template_configure_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
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
            id="account_tag_l10n_at_PCVIII3"
            model="account.account.tag">
            <field name="name">Bilanz PCVIII3</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_PCVIII4"
            model="account.account.tag">
            <field name="name">Bilanz PCVIII4</field>
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
            id="account_tag_l10n_at_RCR"
            model="account.account.tag">
            <field name="name">GuV RCR</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_RRR"
            model="account.account.tag">
            <field name="name">GuV RRR</field>
            <field name="applicability">accounts</field>
            <field name="color" eval="1" />
        </record>
        <record
            id="account_tag_l10n_at_ARR"
            model="account.account.tag">
            <field name="name">GuV ARR</field>
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
            <field name="account_type">asset_current</field>
        </record>
        <!-- Vorlage: Kontenplan Österreich (Basis: EKR 2000) -->
        <record id="l10n_at_chart_template" model="account.chart.template">
            <field name="name">Einheitskontenrahmen Österreich 2010</field>
            <field name="code_digits">4</field>
            <field name="bank_account_code_prefix">280</field>
            <field name="cash_account_code_prefix">270</field>
            <field name="transfer_account_code_prefix">288</field>
            <field name="currency_id" ref="base.EUR"/>
            <field name="country_id" ref="base.at"/>
        </record>
        <record id="chart_at_template_transfer_288" model="account.account.template">
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
        </record>

        <record id="chart_at_template_0010" model="account.account.template">
          <field name="name">Aufwendungen für das Ingangsetzen und Erweitern eines Betriebes</field>
          <field name="code">0010</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI1')])]" />
        </record>
        <record id="chart_at_template_0090" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen</field>
          <field name="code">0090</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
        </record>
        <record id="chart_at_template_0100" model="account.account.template">
          <field name="name">Konzessionen</field>
          <field name="code">0100</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI1')])]" />
        </record>
        <record id="chart_at_template_0110" model="account.account.template">
          <field name="name">Patentrechte und Lizenzen</field>
          <field name="code">0110</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI1')])]" />
        </record>
        <record id="chart_at_template_0120" model="account.account.template">
          <field name="name">Datenverarbeitungsprogramme</field>
          <field name="code">0120</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI1')])]" />
        </record>
        <record id="chart_at_template_0121" model="account.account.template">
          <field name="name">Geringwertige Datenverarbeitungsprogramme</field>
          <field name="code">0121</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI1')])]" />
        </record>
        <record id="chart_at_template_0130" model="account.account.template">
          <field name="name">Marken, Warenzeichen und Musterschutzrechte, sonstige Urheberrechte</field>
          <field name="code">0130</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI1')])]" />
        </record>
        <record id="chart_at_template_0140" model="account.account.template">
          <field name="name">Pacht- und Mietrechte</field>
          <field name="code">0140</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI1')])]" />
        </record>
        <record id="chart_at_template_0150" model="account.account.template">
          <field name="name">Geschäfts(Firmen)wert</field>
          <field name="code">0150</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI2')])]" />
        </record>
        <record id="chart_at_template_0180" model="account.account.template">
          <field name="name">Geleistete Anzahlungen</field>
          <field name="code">0180</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAI3')])]" />
        </record>
        <record id="chart_at_template_019" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen</field>
          <field name="code">0190</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
        </record>
        <record id="chart_at_template_0200" model="account.account.template">
          <field name="name">Unbebaute Grundstücke</field>
          <field name="code">0200</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0210" model="account.account.template">
          <field name="name">Bebaute Grundstücke (Grundwert)</field>
          <field name="code">0210</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_022" model="account.account.template">
          <field name="name">Grundstücksgleiche Rechte</field>
          <field name="code">0220</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0300" model="account.account.template">
          <field name="name">Betriebs- und Geschäftsgebäude auf eigenem Grund</field>
          <field name="code">0300</field>
          <field name="note">KZ 9320 (Bilanzierer gemäß §§ 4 Abs. 1 oder 5)</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0310" model="account.account.template">
          <field name="name">Wohn- und Sozialgebäude auf eigenem Grund</field>
          <field name="code">0310</field>
          <field name="note">KZ 9320 (Bilanzierer gemäß §§ 4 Abs. 1 oder 5)</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0320" model="account.account.template">
          <field name="name">Betriebs- und Geschäftsgebäude auf fremdem Grund</field>
          <field name="code">0320</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0330" model="account.account.template">
          <field name="name">Wohn- und Sozialgebäude auf fremdem Grund</field>
          <field name="code">0330</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0340" model="account.account.template">
          <field name="name">Grundstückseinrichtungen auf eigenem Grund</field>
          <field name="code">0340</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0350" model="account.account.template">
          <field name="name">Grundstückseinrichtungen auf fremdem Grund</field>
          <field name="code">0350</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0360" model="account.account.template">
          <field name="name">Bauliche Investitionen in fremden (gepachteten) Betriebs- und Geschäftsgebäuden</field>
          <field name="code">0360</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0370" model="account.account.template">
          <field name="name">Bauliche Investitionen in fremden (gepachteten) Wohn- und Sozialgebäuden</field>
          <field name="code">0370</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0390" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen</field>
          <field name="code">0390</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII1')])]" />
        </record>
        <record id="chart_at_template_0400" model="account.account.template">
          <field name="name">Fertigungsmaschinen</field>
          <field name="code">0400</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0410" model="account.account.template">
          <field name="name">Antriebsmaschinen</field>
          <field name="code">0410</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0420" model="account.account.template">
          <field name="name">Energieversorgungsanlagen</field>
          <field name="code">0420</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0430" model="account.account.template">
          <field name="name">Transportanlagen</field>
          <field name="code">0430</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0500" model="account.account.template">
          <field name="name">Maschinenwerkzeuge</field>
          <field name="code">0500</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0510" model="account.account.template">
          <field name="name">Allgemeine Werkzeuge und Handwerkzeuge</field>
          <field name="code">0510</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0520" model="account.account.template">
          <field name="name">Vorrichtungen, Formen und Modelle</field>
          <field name="code">0520</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0530" model="account.account.template">
          <field name="name">Andere Erzeugungshilfsmittel</field>
          <field name="code">0530</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0540" model="account.account.template">
          <field name="name">Hebezeuge und Montageanlagen</field>
          <field name="code">0540</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0550" model="account.account.template">
          <field name="name">Geringwertige Vermögensgegenstände, soweit im Erzeugungsprozeß verwendet</field>
          <field name="code">0550</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII2')])]" />
        </record>
        <record id="chart_at_template_0600" model="account.account.template">
          <field name="name">Beheizungs- und Beleuchtungsanlagen</field>
          <field name="code">0600</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0610" model="account.account.template">
          <field name="name">Nachrichten- und Kontrollanlagen</field>
          <field name="code">0610</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0620" model="account.account.template">
          <field name="name">Büromaschinen, EDV-Anlagen</field>
          <field name="code">0620</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0630" model="account.account.template">
          <field name="name">PKW</field>
          <field name="code">0630</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0640" model="account.account.template">
          <field name="name">LKW</field>
          <field name="code">0640</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0650" model="account.account.template">
          <field name="name">Andere Beförderungsmittel</field>
          <field name="code">0650</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0660" model="account.account.template">
          <field name="name">Andere Betriebs- und Geschäftsausstattung</field>
          <field name="code">0660</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0670" model="account.account.template">
          <field name="name">Gebinde</field>
          <field name="code">0670</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0680" model="account.account.template">
          <field name="name">Geringwertige Büromaschinen, EDV-Anlagen</field>
          <field name="code">0680</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0681" model="account.account.template">
          <field name="name">Geringwertige Betriebs- und Geschäftsausstattung</field>
          <field name="code">0681</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0692" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen zu Büromaschinen, EDV-Anlagen</field>
          <field name="code">0692</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0693" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen zu PKW und Kombis</field>
          <field name="code">0693</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0694" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen zu LKW</field>
          <field name="code">0694</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0696" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen zur Betriebs- und Geschäftsausstattung</field>
          <field name="code">0696</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII3')])]" />
        </record>
        <record id="chart_at_template_0700" model="account.account.template">
          <field name="name">Geleistete Anzahlungen</field>
          <field name="code">0700</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_prepayments</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII4')])]" />
        </record>
        <record id="chart_at_template_0710" model="account.account.template">
          <field name="name">Anlagen in Bau</field>
          <field name="code">0710</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII4')])]" />
        </record>
        <record id="chart_at_template_0790" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen</field>
          <field name="code">0790</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_fixed</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAII4')])]" />
        </record>
        <record id="chart_at_template_0800" model="account.account.template">
          <field name="name">Anteile an verbundenen Unternehmen</field>
          <field name="code">0800</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII1')])]" />
        </record>
        <record id="chart_at_template_0810" model="account.account.template">
          <field name="name">Beteiligungen an Gemeinschaftsunternehmen</field>
          <field name="code">0810</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII1')])]" />
        </record>
        <record id="chart_at_template_0820" model="account.account.template">
          <field name="name">Beteiligungen an angeschlossenen (assoziierten) Unternehmen</field>
          <field name="code">0820</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII1')])]" />
        </record>
        <record id="chart_at_template_0830" model="account.account.template">
          <field name="name">Sonstige Beteiligungen</field>
          <field name="code">0830</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII1')])]" />
        </record>
        <record id="chart_at_template_0840" model="account.account.template">
          <field name="name">Ausleihungen an verbundene Unternehmen</field>
          <field name="code">0840</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII2')])]" />
        </record>
        <record id="chart_at_template_0850" model="account.account.template">
          <field name="name">Ausleihungen an Unternehmen, mit denen ein Beteiligungsverhältnis besteht</field>
          <field name="code">0850</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII4')])]" />
        </record>
        <record id="chart_at_template_0860" model="account.account.template">
          <field name="name">Sonstige Ausleihungen</field>
          <field name="code">0860</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII6')])]" />
        </record>
        <record id="chart_at_template_0870" model="account.account.template">
          <field name="name">Anteile an Kapitalgesellschaften ohne Beteiligungscharakter</field>
          <field name="code">0870</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII5')])]" />
        </record>
        <record id="chart_at_template_0880" model="account.account.template">
          <field name="name">Anteile an Personengesellschaften ohne Beteiligungscharakter</field>
          <field name="code">0880</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII5')])]" />
        </record>
        <record id="chart_at_template_0900" model="account.account.template">
          <field name="name">Genossenschaftsanteile ohne Beteiligungscharakter</field>
          <field name="code">0900</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII5')])]" />
        </record>
        <record id="chart_at_template_0910" model="account.account.template">
          <field name="name">Anteile an Investmentfonds</field>
          <field name="code">0910</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII5')])]" />
        </record>
        <record id="chart_at_template_0980" model="account.account.template">
          <field name="name">Geleistete Anzahlungen</field>
          <field name="code">0980</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_prepayments</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII5')])]" />
        </record>
        <record id="chart_at_template_0990" model="account.account.template">
          <field name="name">Kumulierte Abschreibungen</field>
          <field name="code">0990</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AAIII5')])]" />
        </record>
        <record id="chart_at_template_1600" model="account.account.template">
          <field name="name">Handelswaren</field>
          <field name="code">1600</field>
          <field name="note">KZ 9340 (Bilanzierer gemäß §§ 4 Abs. 1 oder 5)</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABI3')])]" />
        </record>
        <record id="chart_at_template_1800" model="account.account.template">
          <field name="name">Geleistete Anzahlungen</field>
          <field name="code">1800</field>
          <field name="note">KZ 9340 (Bilanzierer gemäß §§ 4 Abs. 1 oder 5)</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_prepayments</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABI5')])]" />
        </record>
        <record id="chart_at_template_2000" model="account.account.template">
          <field name="name">Forderungen von Partnern im Inland ohne eigenes Debitorenkonto</field>
          <field name="code">2000</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_receivable</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2099" model="account.account.template">
          <field name="name">Forderungen von Partnern (Point Of Sale)</field>
          <field name="code">2099</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_receivable</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2080" model="account.account.template">
          <field name="name">Einzelwertberichtigungen zu Forderungen aus Lieferungen und Leistungen Inland</field>
          <field name="code">2080</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2090" model="account.account.template">
          <field name="name">Pauschalwertberichtigungen zu Forderungen aus Lieferungen und Leistungen Inland</field>
          <field name="code">2090</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2100" model="account.account.template">
          <field name="name">Forderungen von Partnern im EU Raum ohne eigenes Debitorenkonto</field>
          <field name="code">2100</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_receivable</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2130" model="account.account.template">
          <field name="name">Einzelwertberichtigungen zu Forderungen aus Lieferungen und Leistungen Währungsunion</field>
          <field name="code">2130</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2140" model="account.account.template">
          <field name="name">Pauschalwertberichtigungen zu Forderungen aus Lieferungen und Leistungen Währungsunion</field>
          <field name="code">2140</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2150" model="account.account.template">
          <field name="name">Forderungen von Partnern in Drittstaaten ohne eigenes Debitorenkonto</field>
          <field name="code">2150</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_receivable</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2180" model="account.account.template">
          <field name="name">Einzelwertberichtigungen zu Forderungen aus Lieferungen und Leistungen sonstiges Ausland</field>
          <field name="code">2180</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2190" model="account.account.template">
          <field name="name">Pauschalwertberichtigungen zu Forderungen aus Lieferungen und Leistungen sonstiges Ausland</field>
          <field name="code">2190</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII1')])]" />
        </record>
        <record id="chart_at_template_2230" model="account.account.template">
          <field name="name">Einzelwertberichtigungen zu Forderungen gegenüber verbundenen Unternehmen</field>
          <field name="code">2230</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII2')])]" />
        </record>
        <record id="chart_at_template_2240" model="account.account.template">
          <field name="name">Pauschalwertberichtigungen zu Forderungen gegenüber verbundenen Unternehmen</field>
          <field name="code">2240</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII2')])]" />
        </record>
        <record id="chart_at_template_2280" model="account.account.template">
          <field name="name">Einzelwertberichtigungen zu Forderungen gegenüber Unternehmen, mit denen ein Beteiligungsverhältnis besteht</field>
          <field name="code">2280</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII3')])]" />
        </record>
        <record id="chart_at_template_2290" model="account.account.template">
          <field name="name">Pauschalwertberichtigungen zu Forderungen gegenüber Unternehmen, mit denen ein Beteiligungsverhältnis besteht</field>
          <field name="code">2290</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII3')])]" />
        </record>
        <record id="chart_at_template_2470" model="account.account.template">
          <field name="name">Eingeforderte, aber noch nicht eingezahlte Einlagen</field>
          <field name="code">2470</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII3')])]" />
        </record>
        <record id="chart_at_template_2480" model="account.account.template">
          <field name="name">Einzelwertberichtigungen zu sonstigen Forderungen und Vermögensgegenständen</field>
          <field name="code">2480</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2490" model="account.account.template">
          <field name="name">Pauschalwertberichtigungen zu sonstigen Forderungen und Vermögensgegenständen</field>
          <field name="code">2490</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2500" model="account.account.template">
          <field name="name">Vorsteuern 20%</field>
          <field name="code">2500</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2501" model="account.account.template">
          <field name="name">Vorsteuern 10%</field>
          <field name="code">2501</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2502" model="account.account.template">
          <field name="name">Vorsteuern 13%</field>
          <field name="code">2502</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2505" model="account.account.template">
          <field name="name">Sonstige Vorsteuern</field>
          <field name="code">2505</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2506" model="account.account.template">
          <field name="name">Vorsteuern RC 20%</field>
          <field name="code">2506</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2507" model="account.account.template">
          <field name="name">Vorsteuern RC 10%</field>
          <field name="code">2507</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2510" model="account.account.template">
          <field name="name">Vorsteuern RC EU 20%</field>
          <field name="code">2510</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2511" model="account.account.template">
          <field name="name">Vorsteuern IGE 20%</field>
          <field name="code">2511</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2512" model="account.account.template">
          <field name="name">Vorsteuern IGE 10%</field>
          <field name="code">2512</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2513" model="account.account.template">
          <field name="name">Vorsteuern IGE 13%</field>
          <field name="code">2513</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2515" model="account.account.template">
          <field name="name">Vorsteuern 20% (aus EUSt.)</field>
          <field name="code">2515</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABII4')])]" />
        </record>
        <record id="chart_at_template_2600" model="account.account.template">
          <field name="name">Eigene Anteile</field>
          <field name="code">2600</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIII')])]" />
        </record>
        <record id="chart_at_template_2610" model="account.account.template">
          <field name="name">Anteile an verbundenen Unternehmen</field>
          <field name="code">2610</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIII1')])]" />
        </record>
        <record id="chart_at_template_2620" model="account.account.template">
          <field name="name">Sonstige Anteile</field>
          <field name="code">2620</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIII2')])]" />
        </record>
        <record id="chart_at_template_2680" model="account.account.template">
          <field name="name">Besitzwechsel, soweit dem Unternehmen nicht die der Ausstellung zugrundeliegenden Forderungen zustehen</field>
          <field name="code">2680</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIII2')])]" />
        </record>
        <record id="chart_at_template_2690" model="account.account.template">
          <field name="name">Wertberichtigungen</field>
          <field name="code">2690</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIII2')])]" />
        </record>
        <record id="chart_at_template_2730" model="account.account.template">
          <field name="name">Postwertzeichen</field>
          <field name="code">2730</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIV')])]" />
        </record>
        <record id="chart_at_template_2740" model="account.account.template">
          <field name="name">Stempelmarken</field>
          <field name="code">2740</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIV')])]" />
        </record>
        <record id="chart_at_template_2780" model="account.account.template">
          <field name="name">Schecks in Inlandswährung</field>
          <field name="code">2780</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIV')])]" />
        </record>
        <record id="chart_at_template_transfer_288" model="account.account.template">
        </record>
        <record id="chart_at_template_2890" model="account.account.template">
          <field name="name">Wertberichtigungen</field>
          <field name="code">2890</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_ABIV')])]" />
        </record>
        <record id="chart_at_template_2900" model="account.account.template">
          <field name="name">Aktive Rechnungsabgrenzungsposten</field>
          <field name="code">2900</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AC')])]" />
        </record>
        <record id="chart_at_template_2950" model="account.account.template">
          <field name="name">Disagio</field>
          <field name="code">2950</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AC')])]" />
        </record>
        <record id="chart_at_template_2960" model="account.account.template">
          <field name="name">Unterschiedsbetrag zur gebotenen Pensionsrückstellung</field>
          <field name="code">2960</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PBI')])]" />
        </record>
        <record id="chart_at_template_2970" model="account.account.template">
          <field name="name">Unterschiedsbetrag gem. Abschnitt XII Pensionskassengesetz</field>
          <field name="code">2970</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PBII')])]" />
        </record>
        <record id="chart_at_template_2980" model="account.account.template">
          <field name="name">Steuerabgrenzung</field>
          <field name="code">2980</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_AC')])]" />
        </record>
        <record id="chart_at_template_3000" model="account.account.template">
          <field name="name">Rückstellungen für Abfertigungen</field>
          <field name="code">3000</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PBI')])]" />
        </record>
        <record id="chart_at_template_3010" model="account.account.template">
          <field name="name">Rückstellungen für Pensionen</field>
          <field name="code">3010</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_non_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PBII')])]" />
        </record>
        <record id="chart_at_template_3100" model="account.account.template">
          <field name="name">Anleihen (einschließlich konvertibler)</field>
          <field name="code">3100</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCI')])]" />
        </record>
        <record id="chart_at_template_3200" model="account.account.template">
          <field name="name">Erhaltene Anzahlungen auf Bestellungen</field>
          <field name="code">3200</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">asset_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCIII')])]" />
        </record>
        <record id="chart_at_template_3210" model="account.account.template">
          <field name="name">Umsatzsteuer-Evidenzkonto für erhaltene Anzahlungen auf Bestellungen</field>
          <field name="code">3210</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3300" model="account.account.template">
          <field name="name">Verbindlichkeiten bei Partnern im Inland ohne eigenes Kreditorenkonto</field>
          <field name="code">3300</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_payable</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCIV')])]" />
        </record>
        <record id="chart_at_template_3360" model="account.account.template">
          <field name="name">Verbindlichkeiten bei Partnern im EU Raum ohne eigenes Kreditorenkonto</field>
          <field name="code">3360</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_payable</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCIV')])]" />
        </record>
        <record id="chart_at_template_3370" model="account.account.template">
          <field name="name">Verbindlichkeiten bei Partnern in Drittstaaten ohne eigenes Kreditorenkonto</field>
          <field name="code">3370</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_payable</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCIV')])]" />
        </record>
        <record id="chart_at_template_3480" model="account.account.template">
          <field name="name">Verbindlichkeiten gegenüber Gesellschaftern</field>
          <field name="code">3480</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII')])]" />
        </record>
        <record id="chart_at_template_3500" model="account.account.template">
          <field name="name">Umsatzsteuer 20%</field>
          <field name="code">3500</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3501" model="account.account.template">
          <field name="name">Umsatzsteuer 10%</field>
          <field name="code">3501</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3502" model="account.account.template">
          <field name="name">Umsatzsteuer 13%</field>
          <field name="code">3502</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3505" model="account.account.template">
          <field name="name">Sonstige Umsatzsteuer</field>
          <field name="code">3505</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3510" model="account.account.template">
          <field name="name">Umsatzsteuer RC EU 20%</field>
          <field name="code">3510</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3511" model="account.account.template">
          <field name="name">Umsatzsteuer IGE 20%</field>
          <field name="code">3511</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3512" model="account.account.template">
          <field name="name">Umsatzsteuer IGE 10%</field>
          <field name="code">3512</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3513" model="account.account.template">
          <field name="name">Umsatzsteuer IGE 13%</field>
          <field name="code">3513</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3515" model="account.account.template">
          <field name="name">Einfuhrumsatzsteuer 20%</field>
          <field name="code">3515</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3520" model="account.account.template">
          <field name="name">Ust. Zahllast</field>
          <field name="code">3520</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3530" model="account.account.template">
          <field name="name">Verrechnungskonto Finanzamt</field>
          <field name="code">3530</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_payable</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3540" model="account.account.template">
          <field name="name">Verrechnung Lohnsteuer</field>
          <field name="code">3540</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3541" model="account.account.template">
          <field name="name">Verrechnung Dienstgeberbeitrag</field>
          <field name="code">3541</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3542" model="account.account.template">
          <field name="name">Verrechnung Dienstgeberzuschlag</field>
          <field name="code">3542</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3550" model="account.account.template">
          <field name="name">Verrechnung Kommunalsteuer</field>
          <field name="code">3550</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3551" model="account.account.template">
          <field name="name">Verrechnung Wiener Dienstgeberabgabe</field>
          <field name="code">3551</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_current</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII1')])]" />
        </record>
        <record id="chart_at_template_3600" model="account.account.template">
          <field name="name">Verrechungskonto Sozialversicherung</field>
          <field name="code">3600</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_payable</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII2')])]" />
        </record>
        <record id="chart_at_template_3610" model="account.account.template">
          <field name="name">Verrechnungskonto Magistrat/Gemeinde (KoSt, U-Bahn, etc.)</field>
          <field name="code">3610</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_payable</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PCVIII2')])]" />
        </record>
        <record id="chart_at_template_3740" model="account.account.template">
          <field name="name">WERE Verrechnungskonto</field>
          <field name="code">3740</field>
          <field name="reconcile" eval="True"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">liability_payable</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_PD')])]" />
        </record>
        <record id="chart_at_template_4000" model="account.account.template">
          <field name="name">Brutto-Umsatzerlöse im Inland (20%)</field>
          <field name="code">4000</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT1')])]" />
        </record>
        <record id="chart_at_template_4001" model="account.account.template">
          <field name="name">Brutto-Umsatzerlöse im Inland (10%)</field>
          <field name="code">4001</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT1')])]" />
        </record>
        <record id="chart_at_template_4100" model="account.account.template">
          <field name="name">Brutto-Umsatzerlöse im EU Raum (RC 20%)</field>
          <field name="code">4100</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT1')])]" />
        </record>
        <record id="chart_at_template_4110" model="account.account.template">
          <field name="name">Brutto-Umsatzerlöse im EU Raum (RC 10%)</field>
          <field name="code">4110</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT1')])]" />
        </record>
        <record id="chart_at_template_4200" model="account.account.template">
          <field name="name">Brutto-Umsatzerlöse in Drittstaaten (0%)</field>
          <field name="code">4200</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT1')])]" />
        </record>
        <record id="chart_at_template_4860" model="account.account.template">
          <field name="name">Kursgewinne aus Fremdwährungstransaktionen</field>
          <field name="code">4860</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income_other</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT1')])]" />
        </record>
        <record id="chart_at_template_5000" model="account.account.template">
          <field name="name">Wareneinkauf</field>
          <field name="code">5000</field>
          <field name="note">KZ 9100</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT5I')])]" />
        </record>
        <record id="chart_at_template_5050" model="account.account.template">
          <field name="name">Wareneinkauf EU</field>
          <field name="code">5050</field>
          <field name="note">KZ 9100</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT5I')])]" />
        </record>
        <record id="chart_at_template_5090" model="account.account.template">
          <field name="name">Wareneinkauf 0%</field>
          <field name="code">5090</field>
          <field name="note">KZ 9100</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT5I')])]" />
        </record>
        <record id="chart_at_template_5800" model="account.account.template">
          <field name="name">Skontoerträge auf Materialaufwand</field>
          <field name="code">5800</field>
          <field name="note">KZ 9100</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT5I')])]" />
        </record>
        <record id="chart_at_template_5810" model="account.account.template">
          <field name="name">Skontoerträge auf sonstige bezogene Herstellungsleistungen</field>
          <field name="code">5810</field>
          <field name="note">KZ 9110</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT5II')])]" />
        </record>
        <record id="chart_at_template_5900" model="account.account.template">
          <field name="name">Aufwandsstellenrechnung</field>
          <field name="code">5900</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating')])]" />
        </record>
        <record id="chart_at_template_6200" model="account.account.template">
          <field name="name">Gehälter (Angestellte)</field>
          <field name="code">6200</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6205" model="account.account.template">
          <field name="name">Geschäftsführerbezug</field>
          <field name="code">6205</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6242" model="account.account.template">
          <field name="name">Urlaubsabfindung (Angestellte)</field>
          <field name="code">6242</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6260" model="account.account.template">
          <field name="name">Sonstige Bezüge (Angestellte)</field>
          <field name="code">6260</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6270" model="account.account.template">
          <field name="name">Sachbezug (Angestellte)</field>
          <field name="code">6270</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6271" model="account.account.template">
          <field name="name">Sachbezug (Geschäftsführer)</field>
          <field name="code">6271</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6310" model="account.account.template">
          <field name="name">Grundgehälter (Überstunden)</field>
          <field name="code">6310</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6330" model="account.account.template">
          <field name="name">Gehälter (Überstundenzuschläge)</field>
          <field name="code">6330</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6340" model="account.account.template">
          <field name="name">Veränderung noch nicht konsumierter Urlaub</field>
          <field name="code">6340</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6I')])]" />
        </record>
        <record id="chart_at_template_6400" model="account.account.template">
          <field name="name">Beiträge für betriebliche Mitarbeitervorsorgekasse</field>
          <field name="code">6400</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6II')])]" />
        </record>

        <record id="chart_at_template_6560" model="account.account.template">
          <field name="name">Gesetzlicher Sozialaufwand</field>
          <field name="code">6560</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6II')])]" />
        </record>

        <record id="chart_at_template_6660" model="account.account.template">
          <field name="name">Kommunalsteuer (KoSt)</field>
          <field name="code">6660</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6II')])]" />
        </record>
        <record id="chart_at_template_6661" model="account.account.template">
          <field name="name">Dienstgeberbeitrag zum Familienlastenausgleichsfonds (DB)</field>
          <field name="code">6661</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6II')])]" />
        </record>
        <record id="chart_at_template_6662" model="account.account.template">
          <field name="name">Zuschlag zum Dienstnehmerbeitrag (DZ)</field>
          <field name="code">6662</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6II')])]" />
        </record>
        <record id="chart_at_template_6640" model="account.account.template">
          <field name="name">Dienstgeberabgabe der Gemeinde Wien (U-Bahn Steuer)</field>
          <field name="code">6663</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6II')])]" />
        </record>

        <record id="chart_at_template_6700" model="account.account.template">
          <field name="name">Sonstiger freiwilliger Sozialaufwand</field>
          <field name="code">6700</field>
          <field name="account_type">expense</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT6II')])]" />
        </record>

        <record id="chart_at_template_6900" model="account.account.template">
          <field name="name">Aufwandsstellenrechnung</field>
          <field name="code">6900</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
        </record>
        <record id="chart_at_template_7000" model="account.account.template">
          <field name="name">Abschreibungen auf aktivierte Aufwendungen für das Ingangsetzen und Erweitern eines Betriebes</field>
          <field name="code">7000</field>
          <field name="note">KZ 9130</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense_depreciation</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_EBIT7I')])]" />
        </record>
        <record id="chart_at_template_7090" model="account.account.template">
          <field name="name">Abschreibungen vom Umlaufvermögen, soweit diese die im Unternehmen üblichen Abschreibungen übersteigen</field>
          <field name="code">7090</field>
          <field name="note">KZ 9140 (Bilanzierer gemäß §§ 4 Abs. 1 oder 5)</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense_depreciation</field>
          <field name="tag_ids" eval="[(6, 0, [ref('l10n_at.account_tag_l10n_at_EBIT7II')])]" />
        </record>
        <record id="chart_at_template_7600" model="account.account.template">
          <field name="name">Büromaterial und Drucksorten</field>
          <field name="code">7600</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_763" model="account.account.template">
          <field name="name">Fachliteratur und Zeitungen</field>
          <field name="code">7630</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7690" model="account.account.template">
          <field name="name">Spenden und Trinkgelder</field>
          <field name="code">7690</field>
          <field name="note">KZ 9200</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7770" model="account.account.template">
          <field name="name">Aus- und Fortbildung</field>
          <field name="code">7770</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7780" model="account.account.template">
          <field name="name">Mitgliedsbeiträge</field>
          <field name="code">7780</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7790" model="account.account.template">
          <field name="name">Spesen des Geldverkehrs</field>
          <field name="code">7790</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7820" model="account.account.template">
          <field name="name">Buchwert abgegangener Anlagen, ausgenommen Finanzanlagen</field>
          <field name="code">7820</field>
          <field name="note">KZ 9210</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7830" model="account.account.template">
          <field name="name">Verluste aus dem Abgang vom Anlagevermögen, ausgenommen Finanzanlagen</field>
          <field name="code">7830</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7860" model="account.account.template">
          <field name="name">Kursverluste aus Fremdwährungstransaktionen</field>
          <field name="code">7860</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7890" model="account.account.template">
          <field name="name">Skontoerträge auf sonstige betriebliche Aufwendungen</field>
          <field name="code">7890</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_7900" model="account.account.template">
          <field name="name">Aufwandsstellenrechnung</field>
          <field name="code">7900</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
        </record>
        <record id="chart_at_template_7960" model="account.account.template">
          <field name="name">Herstellungskosten der zur Erzielung der Umsatzerlöse erbrachten Leistungen</field>
          <field name="code">7960</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
        </record>
        <record id="chart_at_template_7970" model="account.account.template">
          <field name="name">Vertriebskosten</field>
          <field name="code">7970</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
        </record>
        <record id="chart_at_template_7980" model="account.account.template">
          <field name="name">Verwaltungskosten</field>
          <field name="code">7980</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
        </record>
        <record id="chart_at_template_7990" model="account.account.template">
          <field name="name">Sonstige betriebliche Aufwendungen</field>
          <field name="code">7990</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">expense</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_operating'), ref('l10n_at.account_tag_l10n_at_EBIT8')])]" />
        </record>
        <record id="chart_at_template_8140" model="account.account.template">
          <field name="name">Erlöse aus dem Abgang von Beteiligungen</field>
          <field name="code">8140</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income_other</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN10')])]" />
        </record>
        <record id="chart_at_template_8150" model="account.account.template">
          <field name="name">Erlöse aus dem Abgang von sonstigen Finanzanlagen</field>
          <field name="code">8150</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income_other</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN10')])]" />
        </record>
        <record id="chart_at_template_8160" model="account.account.template">
          <field name="name">Erlöse aus dem Abgang von Wertpapieren des Umlaufvermögens</field>
          <field name="code">8160</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income_other</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN10')])]" />
        </record>
        <record id="chart_at_template_8170" model="account.account.template">
          <field name="name">Buchwert abgegangener Beteiligungen</field>
          <field name="code">8170</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income_other</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN10')])]" />
        </record>
        <record id="chart_at_template_8180" model="account.account.template">
          <field name="name">Buchwert abgegangener sonstiger Finanzanlagen</field>
          <field name="code">8180</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income_other</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN11')])]" />
        </record>
        <record id="chart_at_template_8190" model="account.account.template">
          <field name="name">Buchwert abgegangener Wertpapiere des Umlaufvermögens</field>
          <field name="code">8190</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income_other</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN11')])]" />
        </record>
        <record id="chart_at_template_8200" model="account.account.template">
          <field name="name">Erträge aus dem Abgang von und der Zuschreibung zu Finanzanlagen</field>
          <field name="code">8200</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income_other</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN10')])]" />
        </record>
        <record id="chart_at_template_8210" model="account.account.template">
          <field name="name">Erträge aus dem Abgang von und der Zuschreibung zu Wertpapieren des Umlaufvermögens</field>
          <field name="code">8210</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income_other</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN11')])]" />
        </record>
        <record id="chart_at_template_8350" model="account.account.template">
          <field name="name">Nicht ausgenützte Lieferantenskonti</field>
          <field name="code">8350</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income_other</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN12')])]" />
        </record>
        <record id="chart_at_template_8990" model="account.account.template">
          <field name="name">Gewinnabfuhr bzw. Verlustüberrechnung aus Ergebnisabführungsverträgen</field>
          <field name="code">8990</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">income_other</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_FIN12')])]" />
        </record>
        <record id="chart_at_template_9190" model="account.account.template">
          <field name="name">Nicht eingeforderte ausstehende Einlagen</field>
          <field name="code">9190</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">equity</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_PAI')])]" />
        </record>
        <record id="chart_at_template_9390" model="account.account.template">
          <field name="name">Bilanzgewinn (-verlust)</field>
          <field name="code">9390</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">equity</field>
          <field name="tag_ids" eval="[(6, 0, [ref('account.account_tag_financing'), ref('l10n_at.account_tag_l10n_at_PAIV')])]" />
        </record>
        <record id="chart_at_template_9800" model="account.account.template">
          <field name="name">Eröffnungsbilanz</field>
          <field name="code">9800</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">equity</field>
        </record>
        <record id="chart_at_template_9850" model="account.account.template">
          <field name="name">Schlussbilanz</field>
          <field name="code">9850</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">equity</field>
        </record>
        <record id="chart_at_template_9890" model="account.account.template">
          <field name="name">Gewinn- und Verlustrechnung</field>
          <field name="code">9890</field>
          <field name="reconcile" eval="False"/>
          <field name="chart_template_id" ref="l10n_at_chart_template"/>
          <field name="account_type">equity</field>
        </record>
    </data>
</odoo>

```

## File: data\account_chart_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <!-- Vorlagen: Kontenplan -->
        <record id="l10n_at_chart_template" model="account.chart.template">
            <field name="name">Einheitskontenrahmen Österreich 2010</field>
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

            <field name="account_journal_early_pay_discount_loss_account_id" ref="chart_at_template_5800"/>
            <field name="account_journal_early_pay_discount_gain_account_id" ref="chart_at_template_8350"/>

            <field name="property_tax_payable_account_id" ref="chart_at_template_2600"/>
            <field name="property_tax_receivable_account_id" ref="chart_at_template_2600"/>
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
            <field name="name">National + EU (ohne UID)</field>
            <field name="auto_apply" eval="True" />
            <field name="vat_required" eval="False" />
            <field name="country_group_id" ref="base.europe" />
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
        </record>
        <record id="fiscal_position_template_national_w_uid" model="account.fiscal.position.template">
            <field name="name">National</field>
            <field name="auto_apply" eval="True" />
            <field name="vat_required" eval="True" />
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
            <field name="tax_dest_id" ref="account_tax_template_purchase_rev_charge_19_2_25_5" />
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

## File: data\account_tax_group_data.xml

```xml
<odoo>
    <data noupdate="1">
        <!-- Account Tax Group -->
        <record id="tax_group_0" model="account.tax.group">
            <field name="name">0%</field>
            <field name="country_id" ref="base.at"/>
        </record>
        <record id="tax_group_10" model="account.tax.group">
            <field name="name">10%</field>
            <field name="country_id" ref="base.at"/>
        </record>
        <record id="tax_group_12" model="account.tax.group">
            <field name="name">12%</field>
            <field name="country_id" ref="base.at"/>
        </record>
        <record id="tax_group_13" model="account.tax.group">
            <field name="name">13%</field>
            <field name="country_id" ref="base.at"/>
        </record>
        <record id="tax_group_19" model="account.tax.group">
            <field name="name">19%</field>
            <field name="country_id" ref="base.at"/>
        </record>
        <record id="tax_group_20" model="account.tax.group">
            <field name="name">20%</field>
            <field name="country_id" ref="base.at"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report" model="account.report">
        <field name="name">Tax Report</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.at"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_line_l10n_at_non_tva_sale_report_title" model="account.report.line">
                <field name="name">3. Aufschlüsselung für die Zusammenfassende Meldung (ZM)</field>
                <field name="sequence" eval="5"/>
                <field name="aggregation_formula">(AT_ZM_IGL.balance + AT_ZM_IGL3.balance + AT_ZM_DL.balance)</field>
                <field name="children_ids">
                    <record id="tax_report_line_l10n_at_tva_line_3_zm_igl" model="account.report.line">
                        <field name="name">Innergemeinschaftliche Lieferungen</field>
                        <field name="code">AT_ZM_IGL</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_3_zm_igl_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">AT_ZM_IGL</field>
                            </record>
                        </field>
                        <field name="sequence" eval="10"/>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_3_zm_igl3" model="account.report.line">
                        <field name="name">Innergemeinschaftliche Lieferungen (Dreiecksgeschäfte)</field>
                        <field name="code">AT_ZM_IGL3</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_3_zm_igl3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">AT_ZM_IGL3</field>
                            </record>
                        </field>
                        <field name="sequence" eval="20"/>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_3_zm_dl" model="account.report.line">
                        <field name="name">Grenzüberschreitende Dienstleistungen (Sonstige Leistungen)</field>
                        <field name="code">AT_ZM_DL</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_3_zm_dl_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">AT_ZM_DL</field>
                            </record>
                        </field>
                        <field name="sequence" eval="30"/>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_l10n_at_tva_sale_report_title" model="account.report.line">
                <field name="name">4. Berechnung der Umsatzsteuer (U1/U30)</field>
                <field name="aggregation_formula">(AT_022_tax.balance + AT_029_tax.balance + AT_006_tax.balance + AT_037_tax.balance + AT_052_tax.balance + AT_007_tax.balance + AT_056.balance + AT_057.balance + AT_048.balance + AT_044.balance + AT_032.balance + AT_072_tax.balance + AT_073_tax.balance + AT_008_tax.balance + AT_088_tax.balance)</field>
                <field name="children_ids">
                    <record id="tax_report_line_l10n_at_tva_line_4_1" model="account.report.line">
                        <field name="name">4.1 Gesamtbetrag der Bemessungsgrundlage für Lieferungen und sonstige Leistungen [000]</field>
                        <field name="code">AT_000</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 000</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_2" model="account.report.line">
                        <field name="name">4.2 zuzüglich Eigenverbrauch (§ 1 Abs. 1 Z 2, § 3 Abs. 2 und § 3a Abs. 1a) [001]</field>
                        <field name="code">AT_001</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 001</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_3" model="account.report.line">
                        <field name="name">4.3 abzüglich Umsätze, für die die Steuerschuld gemäß § 19 Abs. 1 (Leistungsempfänger) [021]</field>
                        <field name="code">AT_021</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 021</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_4" model="account.report.line">
                        <field name="name">4.4 Summe</field>
                        <field name="aggregation_formula">AT_000.balance + AT_001.balance - AT_021.balance</field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_sale_01_report_title" model="account.report.line">
                        <field name="name">Davon steuerfrei MIT Vorsteuerabzug gemäß</field>
                        <field name="aggregation_formula">AT_011.balance + AT_012.balance + AT_015.balance + AT_017.balance + AT_018.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_5" model="account.report.line">
                                <field name="name">4.5 § 6 Abs. 1 Z 1 iVm § 7 (Ausfuhrlieferungen) [011]</field>
                                <field name="code">AT_011</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_5_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 011</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_6" model="account.report.line">
                                <field name="name">4.6 § 6 Abs. 1 Z 1 iVm § 8 (Lohnveredelungen) [012]</field>
                                <field name="code">AT_012</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_6_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 012</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_7" model="account.report.line">
                                <field name="name">4.7 § 6 Abs. 1 Z 2 bis 6 sowie § 23 Abs. 5 (Seeschifffahrt, Luftfahrt, ...) [015]</field>
                                <field name="code">AT_015</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_7_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 015</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_8" model="account.report.line">
                                <field name="name">4.8 Art. 6 Abs. 1 (innergemeinschaftliche Lieferungen ohne die nachstehend gesondert anzuführenden Fahrzeuglieferungen) [017]</field>
                                <field name="code">AT_017</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_8_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 017</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_9" model="account.report.line">
                                <field name="name">4.9 Art. 6 Abs. 1, sofern Lieferungen neuer Fahrzeuge an Abnehmer ohne UID-Nummer bzw. durch Fahrzeuglieferer gemäß Art. 2 erfolgten. [018]</field>
                                <field name="code">AT_018</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_9_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 018</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_sale_02_report_title" model="account.report.line">
                        <field name="name">Davon steuerfrei OHNE Vorsteuerabzug gemäß</field>
                        <field name="aggregation_formula">AT_019.balance + AT_016.balance + AT_020.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_10" model="account.report.line">
                                <field name="name">4.10 § 6 Abs. 1 Z 9 lit. a (Grundstücksumsätze) [019]</field>
                                <field name="code">AT_019</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_10_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 019</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_11" model="account.report.line">
                                <field name="name">4.11 § 6 Abs. 1 Z 27 (Kleinunternehmer) [016]</field>
                                <field name="code">AT_016</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_11_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 016</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_12" model="account.report.line">
                                <field name="name">4.12 § 6 Abs. 1 Z .. (übrige steuerfreie Umsätze ohne Vorsteuerabzug) [020]</field>
                                <field name="code">AT_020</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_12_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 020</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_13" model="account.report.line">
                        <field name="name">4.13 Gesamtbetrag der steuerpflichtigen Lieferungen, sonstigen Leistungen und Eigenverbrauch (einschließlich steuerpflichtiger Anzahlungen)</field>
                        <field name="aggregation_formula">(AT_000.balance + AT_001.balance - AT_021.balance) - AT_011.balance - AT_012.balance - AT_015.balance - AT_017.balance - AT_018.balance - AT_019.balance - AT_016.balance - AT_020.balance</field>
                    </record>
                    <record id="tax_report_line_at_base_title_umsatz_base_4_14_19" model="account.report.line">
                        <field name="name">Bemessungsgrundlage</field>
                        <field name="aggregation_formula">AT_022_base.balance + AT_029_base.balance + AT_006_base.balance + AT_037_base.balance + AT_052_base.balance + AT_007_base.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_14_base" model="account.report.line">
                                <field name="name">4.14 20% Normalsteuersatz [022]</field>
                                <field name="code">AT_022_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_14_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 022 Bemessungsgrundlage</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_15_base" model="account.report.line">
                                <field name="name">4.15 10% ermäßigter Steuersatz [029]</field>
                                <field name="code">AT_029_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_15_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 029 Bemessungsgrundlage</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_16_base" model="account.report.line">
                                <field name="name">4.16 13% ermäßigter Steuersatz [006]</field>
                                <field name="code">AT_006_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_16_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 006 Bemessungsgrundlage</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_17_base" model="account.report.line">
                                <field name="name">4.17 19% für Jungholz und Mittelberg [037]</field>
                                <field name="code">AT_037_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_17_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 037 Bemessungsgrundlage</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_18_base" model="account.report.line">
                                <field name="name">4.18 10% Zusatzsteuer für pauschalierte land- und forstwirtschaftliche Betriebe [052]</field>
                                <field name="code">AT_052_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_18_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 052 Bemessungsgrundlage</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_19_base" model="account.report.line">
                                <field name="name">4.19 7% Zusatzsteuer für pauschalierte land- und forstwirtschaftliche Betriebe [007]</field>
                                <field name="code">AT_007_base</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_19_base_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 007 Bemessungsgrundlage</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_at_tax_title_4_14_19" model="account.report.line">
                        <field name="name">Umsatzsteuer</field>
                        <field name="aggregation_formula">AT_022_tax.balance + AT_029_tax.balance + AT_006_tax.balance + AT_037_tax.balance + AT_052_tax.balance + AT_007_tax.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_14_tax" model="account.report.line">
                                <field name="name">4.14 20% Normalsteuersatz</field>
                                <field name="code">AT_022_tax</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_14_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 022 Umsatzsteuer</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_15_tax" model="account.report.line">
                                <field name="name">4.15 10% ermäßigter Steuersatz</field>
                                <field name="code">AT_029_tax</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_15_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 029 Umsatzsteuer</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_16_tax" model="account.report.line">
                                <field name="name">4.16 13% ermäßigter Steuersatz</field>
                                <field name="code">AT_006_tax</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_16_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 006 Umsatzsteuer</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_17_tax" model="account.report.line">
                                <field name="name">4.17 19% für Jungholz und Mittelberg</field>
                                <field name="code">AT_037_tax</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_17_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 037 Umsatzsteuer</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_18_tax" model="account.report.line">
                                <field name="name">4.18 10% Zusatzsteuer für pauschalierte land- und forstwirtschaftliche Betriebe</field>
                                <field name="code">AT_052_tax</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_18_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 052 Umsatzsteuer</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_19_tax" model="account.report.line">
                                <field name="name">4.19 7% Zusatzsteuer für pauschalierte land- und forstwirtschaftliche Betriebe</field>
                                <field name="code">AT_007_tax</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_19_tax_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 007 Umsatzsteuer</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_20" model="account.report.line">
                        <field name="name">4.20 Steuerschuld gemäß § 11 Abs. 12 und 14, § 16 Abs. 2 sowie gemäß Art. 7 Abs. 4 [056]</field>
                        <field name="code">AT_056</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_20_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 056</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_21" model="account.report.line">
                        <field name="name">4.21 Steuerschuld gemäß § 19 Abs. 1 zweiter Satz, § 19 Abs. 1c, 1e sowie gemäß Art. 25 Abs. 5 [057]</field>
                        <field name="code">AT_057</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_21_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 057</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_22" model="account.report.line">
                        <field name="name">4.22 Steuerschuld gemäß § 19 Abs. 1a (Bauleistungen) [048]</field>
                        <field name="code">AT_048</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_22_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 048</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_23" model="account.report.line">
                        <field name="name">4.23 Steuerschuld gemäß § 19 Abs. 1b (Sicherungseigentum, ...) [044]</field>
                        <field name="code">AT_044</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_23_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 044</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_4_24" model="account.report.line">
                        <field name="name">4.24 Steuerschuld gemäß § 19 Abs. 1d (Schrott und Abfallstoffe) [032]</field>
                        <field name="code">AT_032</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_24_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 032</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_sale_03_report_title" model="account.report.line">
                        <field name="name">Innergemeinschaftliche Erwerb</field>
                        <field name="aggregation_formula">AT_070.balance + AT_071.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_25" model="account.report.line">
                                <field name="name">4.25 Gesamtbetrag der Bemessungsgrundlagen für innergemeinschaftliche Erwerbe [070]</field>
                                <field name="code">AT_070</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_25_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 070</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_26" model="account.report.line">
                                <field name="name">4.26 Davon steuerfrei gemäß Art. 6 Abs. 2 [071]</field>
                                <field name="code">AT_071</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_26_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 071</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_27" model="account.report.line">
                                <field name="name">4.27 Gesamtbetrag der steuerpflichtigen innergemeinschaftlichen Erwerbe</field>
                                <field name="aggregation_formula">AT_070.balance - AT_071.balance</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_sale_04_report_title" model="account.report.line">
                        <field name="name">Davon sind zu versteuern mit</field>
                        <field name="aggregation_formula">AT_072_base.balance + AT_073_base.balance + AT_008_base.balance + AT_088_base.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_at_base_title_umsatz_base_4_28_31" model="account.report.line">
                                <field name="name">Bemessungsgrundlage</field>
                                <field name="aggregation_formula">AT_072_base.balance + AT_073_base.balance + AT_008_base.balance + AT_088_base.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_28_base" model="account.report.line">
                                        <field name="name">4.28 20% Normalsteuersatz [072]</field>
                                        <field name="code">AT_072_base</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_28_base_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 072 Bemessungsgrundlage</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_l10n_at_tva_line_4_29_base" model="account.report.line">
                                        <field name="name">4.29 10% ermäßigter Steuersatz [073]</field>
                                        <field name="code">AT_073_base</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_29_base_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 073 Bemessungsgrundlage</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_l10n_at_tva_line_4_30_base" model="account.report.line">
                                        <field name="name">4.30 13% ermäßigter Steuersatz [008]</field>
                                        <field name="code">AT_008_base</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_30_base_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 008 Bemessungsgrundlage</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_l10n_at_tva_line_4_31_base" model="account.report.line">
                                        <field name="name">4.31 19% für Jungholz und Mittelberg [088]</field>
                                        <field name="code">AT_088_base</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_31_base_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 088 Bemessungsgrundlage</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_at_tax_title_4_28_31" model="account.report.line">
                                <field name="name">Umsatzsteuer</field>
                                <field name="aggregation_formula">AT_072_tax.balance + AT_073_tax.balance + AT_008_tax.balance + AT_088_tax.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_28_tax" model="account.report.line">
                                        <field name="name">4.28 20% Normalsteuersatz</field>
                                        <field name="code">AT_072_tax</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_28_tax_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 072 Umsatzsteuer</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_l10n_at_tva_line_4_29_tax" model="account.report.line">
                                        <field name="name">4.29 10% ermäßigter Steuersatz</field>
                                        <field name="code">AT_073_tax</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_29_tax_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 073 Umsatzsteuer</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_l10n_at_tva_line_4_30_tax" model="account.report.line">
                                        <field name="name">4.30 13% ermäßigter Steuersatz</field>
                                        <field name="code">AT_008_tax</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_30_tax_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 008 Umsatzsteuer</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_line_l10n_at_tva_line_4_31_tax" model="account.report.line">
                                        <field name="name">4.31 19% für Jungholz und Mittelberg</field>
                                        <field name="code">AT_088_tax</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_line_l10n_at_tva_line_4_31_tax_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">KZ 088 Umsatzsteuer</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_sale_05_report_title" model="account.report.line">
                        <field name="name">Nicht zu versteuernde Erwerbe</field>
                        <field name="aggregation_formula">AT_076.balance + AT_077.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_line_l10n_at_tva_line_4_32" model="account.report.line">
                                <field name="name">4.32 Erwerbe gemäß Art. 3 Abs. 8 zweiter Satz, die im Mitgliedstaat des Bestimmungslandes besteuert worden sind [076]</field>
                                <field name="code">AT_076</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_32_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 076</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_line_l10n_at_tva_line_4_33" model="account.report.line">
                                <field name="name">4.33 Erwerbe gemäß Art. 3 Abs. 8 zweiter Satz, die gemäß Art. 25 Abs. 2 im Inland als besteuert gelten [077]</field>
                                <field name="code">AT_077</field>
                                <field name="expression_ids">
                                    <record id="tax_report_line_l10n_at_tva_line_4_33_tag" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">tax_tags</field>
                                        <field name="formula">KZ 077</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_l10n_at_tva_purchase_report_title" model="account.report.line">
                <field name="name">5. Berechnung der abziehbaren Vorsteuer</field>
                <field name="aggregation_formula">AT_060.balance + AT_061.balance + AT_083.balance + AT_065.balance + AT_066.balance + AT_082.balance + AT_087.balance + AT_089.balance + AT_064.balance + AT_062.balance + AT_063.balance + AT_067.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_l10n_at_tva_line_5_1" model="account.report.line">
                        <field name="name">5.1 Gesamtbetrag der Vorsteuern (ohne die nachstehend gesondert anzuführenden Beträge) [060]</field>
                        <field name="code">AT_060</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_1_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 060</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_2" model="account.report.line">
                        <field name="name">5.2 Vorsteuern betreffend die entrichtete Einfuhrumsatzsteuer (§ 12 Abs. 1 Z 2 lit. a) [061]</field>
                        <field name="code">AT_061</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_2_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 061</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_3" model="account.report.line">
                        <field name="name">5.3 Vorsteuern betreffend die geschuldete, auf dem Abgabenkonto verbuchte Einfuhrumsatzsteuer (§ 12 Abs. 1 Z 2 lit. b) [083]</field>
                        <field name="code">AT_083</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_3_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 083</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_4" model="account.report.line">
                        <field name="name">5.4 Vorsteuern aus dem innergemeinschaftlichen Erwerb [065]</field>
                        <field name="code">AT_065</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_4_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 065</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_5" model="account.report.line">
                        <field name="name">5.5 Vorsteuern betreffend die Steuerschuld gemäß § 19 Abs. 1 zweiter Satz, § 19 Abs. 1c, 1e sowie gemäß Art. 25 Abs. 5 [066]</field>
                        <field name="code">AT_066</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_5_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 066</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_6" model="account.report.line">
                        <field name="name">5.6 Vorsteuern betreffend die Steuerschuld gemäß § 19 Abs. 1a (Bauleistungen) [082]</field>
                        <field name="code">AT_082</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_6_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 082</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_7" model="account.report.line">
                        <field name="name">5.7 Vorsteuern betreffend die Steuerschuld gemäß § 19 Abs. 1b (Sicherungseigentum, ...) [087]</field>
                        <field name="code">AT_087</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_7_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 087</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_8" model="account.report.line">
                        <field name="name">5.8 Vorsteuern betreffend die Steuerschuld gemäß § 19 Abs. 1d (Schrott und Abfallstoffe) [089]</field>
                        <field name="code">AT_089</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_8_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 089</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_9" model="account.report.line">
                        <field name="name">5.9 Vorsteuern für innergemeinschaftliche Lieferungen neuer Fahrzeuge von Fahrzeuglieferern gemäß Art. 2 [064]</field>
                        <field name="code">AT_064</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_9_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 064</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_10" model="account.report.line">
                        <field name="name">5.10 Davon nicht abzugsfähig gemäß § 12 Abs. 3 iVm Abs. 4 und 5 [062]</field>
                        <field name="code">AT_062</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 062</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_11" model="account.report.line">
                        <field name="name">5.11 Berichtigung gemäß § 12 Abs. 10 und 11 [063]</field>
                        <field name="code">AT_063</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_11_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 063</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_12" model="account.report.line">
                        <field name="name">5.12 Berichtigung gemäß § 16 [067]</field>
                        <field name="code">AT_067</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_5_12_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 067</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_line_l10n_at_tva_line_5_13" model="account.report.line">
                        <field name="name">5.13 Gesamtbetrag der abziehbaren Vorsteuer</field>
                        <field name="aggregation_formula">AT_060.balance + AT_061.balance + AT_083.balance + AT_065.balance + AT_066.balance + AT_082.balance + AT_087.balance + AT_089.balance + AT_064.balance + AT_062.balance + AT_063.balance + AT_067.balance</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_l10n_at_tva_final_report_title" model="account.report.line">
                <field name="name">6. Sonstige Berichtigungen</field>
                <field name="aggregation_formula">AT_090.balance</field>
                <field name="children_ids">
                    <record id="tax_report_line_l10n_at_tva_line_6" model="account.report.line">
                        <field name="name">Sonstige Berichtigungen [090]</field>
                        <field name="code">AT_090</field>
                        <field name="expression_ids">
                            <record id="tax_report_line_l10n_at_tva_line_6_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">KZ 090</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_line_l10n_at_tva_line_7" model="account.report.line">
                <field name="name">7. Zahllast (-) bzw. Gutschrift/Überschuss (+) [095]</field>
                <field name="aggregation_formula">(AT_022_tax.balance + AT_029_tax.balance + AT_006_tax.balance + AT_037_tax.balance + AT_052_tax.balance + AT_007_tax.balance + AT_056.balance + AT_057.balance + AT_048.balance + AT_044.balance + AT_032.balance + AT_072_tax.balance + AT_073_tax.balance + AT_008_tax.balance + AT_088_tax.balance + AT_060.balance + AT_061.balance + AT_083.balance + AT_065.balance + AT_066.balance + AT_082.balance + AT_087.balance + AT_089.balance + AT_064.balance + AT_062.balance + AT_063.balance + AT_067.balance) + AT_090.balance</field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\account_tax_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Vorlagen: Steuerdefinition -->
        <!-- Vorlagen: Verkauf (Umsatzsteuer) -->
        <!-- Lieferungen, sonstige Leistungen und Eigenverbrauch +000,+001,-021 -->
        <!-- Umsätze, für die die Steuerschuld gemäß § 19 Abs. 1 zweiter
            Satz sowie gemäß § 19 Abs. 1a, 1b,
            1c, 1d und 1e (Umsätze ab 01.07.2010)
            auf den Leistungsempfänger übergegangen ist. -->
        <record id="account_tax_template_sales_rev_charge_0_code021" model="account.tax.template">
            <field name="name">UST_021 § 19 Abs. 1 zweiter Satz (Steuerschuld betrifft Leistungsempfänger)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">400</field>
            <field name="type_tax_use">sale</field>
            <field eval="0.0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                  ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                  ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_rev_charge_0_code021_1a" model="account.tax.template">
            <field name="name">UST_021 § 19 Abs. 1a (Steuerschuld betrifft Leistungsempfänger)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">400</field>
            <field name="type_tax_use">sale</field>
            <field eval="0.0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                  ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                  ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_rev_charge_0_code021_1b" model="account.tax.template">
            <field name="name">UST_021 § 19 Abs. 1b (Steuerschuld betrifft Leistungsempfänger)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">400</field>
            <field name="type_tax_use">sale</field>
            <field eval="0.0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                  ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                  ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),

                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_rev_charge_0_code021_1c" model="account.tax.template">
            <field name="name">UST_021 § 19 Abs. 1c (Steuerschuld betrifft Leistungsempfänger)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">400</field>
            <field name="type_tax_use">sale</field>
            <field eval="0.0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                  ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                  ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_rev_charge_0_code021_1d" model="account.tax.template">
            <field name="name">UST_021 § 19 Abs. 1d (Steuerschuld betrifft Leistungsempfänger)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">400</field>
            <field name="type_tax_use">sale</field>
            <field eval="0.0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                  ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                  ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_rev_charge_0_code021_1e" model="account.tax.template">
            <field name="name">UST_021 § 19 Abs. 1e (Steuerschuld betrifft Leistungsempfänger)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">400</field>
            <field name="type_tax_use">sale</field>
            <field eval="0.0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                  ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                  ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_3_tag')],
                }),
            ]"/>
        </record>
        <!-- Steuerfreier Umsatz +011, +012, +015, +017, +018, +019, +016, +020 -->
        <!-- Ausfuhrlieferungen (§ 6 Abs. 1 Z 1 iVm § 7) -->
        <record id="account_tax_template_sales_non_eu_0_code011" model="account.tax.template">
            <field name="name">UST_011 Export 0%</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">300</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">consu</field>
            <field eval="0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_5_tag'), ref('tax_report_line_l10n_at_tva_line_4_1_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_5_tag'), ref('tax_report_line_l10n_at_tva_line_4_1_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
        </record>
        <!-- Lohnveredelungen (§ 6 Abs. 1 Z 1 iVm § 8) -->
        <record id="account_tax_template_sales_non_eu_0_code012" model="account.tax.template">
            <field name="name">UST_012 Lohnveredelung 0%</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">200</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_6_tag'), ref('tax_report_line_l10n_at_tva_line_4_1_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_6_tag'), ref('tax_report_line_l10n_at_tva_line_4_1_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
        </record>
        <!-- § 6 Abs. 1 Z 2 bis 6 sowie § 23 Abs. 5 (Seeschifffahrt, Luftfahrt,
            grenzüberschreitende Personenbeförderung, Diplomaten, Reisevorleistungen
            im Drittlandsgebiet usw.) -->
        <record id="account_tax_template_sales_non_eu_0_code015" model="account.tax.template">
            <field name="name">UST_015 Export 0% (§ 6 Abs. 1 Z 2 bis 6)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">300</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_7_tag'), ref('tax_report_line_l10n_at_tva_line_4_1_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_7_tag'), ref('tax_report_line_l10n_at_tva_line_4_1_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
        </record>
        <!-- Innergemeinschaftliche Lieferungen ohne Fahrzeuglieferungen
            (Art. 6 Abs. 1) -->
        <record id="account_tax_template_sales_eu_0_code017" model="account.tax.template">
            <field name="name">UST_017 IGL 0% (ohne Art. 6 Abs. 1)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">200</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">consu</field>
            <field eval="0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [
                        ref('tax_report_line_l10n_at_tva_line_4_8_tag'),
                        ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                        ref('tax_report_line_l10n_at_tva_line_3_zm_igl_tag')
                  ],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [
                        ref('tax_report_line_l10n_at_tva_line_4_8_tag'),
                        ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                        ref('tax_report_line_l10n_at_tva_line_3_zm_igl_tag')
                  ],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
        </record>
        <!-- EU Fahrzeuglieferungen (Art. 6 Abs. 1) -->
        <record id="account_tax_template_sales_eu_0_code018" model="account.tax.template">
            <field name="name">UST_018 IGL 0% (Art. 6 Abs. 1)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">200</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">consu</field>
            <field eval="0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [
                        ref('tax_report_line_l10n_at_tva_line_4_9_tag'),
                        ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                        ref('tax_report_line_l10n_at_tva_line_3_zm_igl_tag')
                  ],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [
                        ref('tax_report_line_l10n_at_tva_line_4_9_tag'),
                        ref('tax_report_line_l10n_at_tva_line_4_1_tag'),
                        ref('tax_report_line_l10n_at_tva_line_3_zm_igl_tag')
                  ],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
        </record>
        <!-- § 6 Abs. 1 Z 9 lit. a (Grundstücksumsätze) -->
        <record id="account_tax_template_sales_0_code019" model="account.tax.template">
            <field name="name">UST_019 Grundstücksumsätze 0% (§ 6 Abs. 1 Z 9 lit. a)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_10_tag'), ref('tax_report_line_l10n_at_tva_line_4_1_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_10_tag'), ref('tax_report_line_l10n_at_tva_line_4_1_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
        </record>
        <!-- § 6 Abs. 1 Z 27 (Kleinunternehmer) -->
        <record id="account_tax_template_sales_0_code016" model="account.tax.template">
            <field name="name">UST_016 Kleinunternehmer 0% (§ 6 Abs. 1 Z 27)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_11_tag'), ref('tax_report_line_l10n_at_tva_line_4_1_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_11_tag'), ref('tax_report_line_l10n_at_tva_line_4_1_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
        </record>
        <!-- § 6 Abs. 1 Z 1, 7-8, 10-26, 28 (übrige steuerfreie Umsätze ohne
            Vorsteuerabzug) -->
        <record id="account_tax_template_sales_0_code020" model="account.tax.template">
            <field name="name">UST_020 Übrige steuerfreie Umsätze 0%</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_12_tag'), ref('tax_report_line_l10n_at_tva_line_4_1_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_12_tag'), ref('tax_report_line_l10n_at_tva_line_4_1_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
        </record>
        <!-- zu versteuernder Umsatz +022, +029, +006, +037, +052, +007 -->
        <!-- Soll-Umsatzsteuern (=normale Umsatzsteuer) -->
        <record id="account_tax_template_sales_20_code022" model="account.tax.template">
            <field name="name">UST_022 Normalsteuersatz 20%</field>
            <field name="description">20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">50</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">consu</field>
            <field eval="20" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'), ref('tax_report_line_l10n_at_tva_line_4_14_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3500'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_14_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'), ref('tax_report_line_l10n_at_tva_line_4_14_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3500'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_14_tax_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_20_katalog022" model="account.tax.template">
            <field name="name">UST_022 Normalsteuersatz 20% (Sonstige Leistungen)</field>
            <field name="description">20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">service</field>
            <field eval="20" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'), ref('tax_report_line_l10n_at_tva_line_4_14_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3500'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_14_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'), ref('tax_report_line_l10n_at_tva_line_4_14_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3500'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_14_tax_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_10_code029" model="account.tax.template">
            <field name="name">UST_029 ermäßigter Steuersatz 10%</field>
            <field name="description">10%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">consu</field>
            <field eval="10" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_10"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'), ref('tax_report_line_l10n_at_tva_line_4_15_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3501'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_15_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'), ref('tax_report_line_l10n_at_tva_line_4_15_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3501'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_15_tax_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_13_code006" model="account.tax.template">
            <field name="name">UST_006 ermäßigter Steuersatz 13%</field>
            <field name="description">13%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="13" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_13"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'), ref('tax_report_line_l10n_at_tva_line_4_16_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3502'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_16_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'), ref('tax_report_line_l10n_at_tva_line_4_16_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3502'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_16_tax_tag')],
                }),
            ]"/>
        </record>
        <!-- 19% für Gemeinden Jungholz und Mittelberg -->
        <record id="account_tax_template_sales_19_code037" model="account.tax.template">
            <field name="name">UST_037 Steuersatz 19%</field>
            <field name="description">19%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="19" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_19"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'), ref('tax_report_line_l10n_at_tva_line_4_17_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_17_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'), ref('tax_report_line_l10n_at_tva_line_4_17_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_17_tax_tag')],
                }),
            ]"/>
        </record>
        <!-- 10% Zusatzsteuer für pauschalierte land- und forstwirtschaftliche
            Betriebe -->
        <record id="account_tax_template_sales_add10_code052" model="account.tax.template">
            <field name="name">UST_052 Zusatzsteuersatz 10% (LWB/FWB)</field>
            <field name="description">10%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="10" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_10"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'), ref('tax_report_line_l10n_at_tva_line_4_18_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_18_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'), ref('tax_report_line_l10n_at_tva_line_4_18_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_18_tax_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_add7_code007" model="account.tax.template">
            <field name="name">UST_007 Zusatzsteuersatz 7% (LWB/FWB)</field>
            <field name="description">7%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="7" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'), ref('tax_report_line_l10n_at_tva_line_4_19_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_19_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_1_tag'), ref('tax_report_line_l10n_at_tva_line_4_19_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_19_tax_tag')],
                }),
            ]"/>
        </record>
        <!-- zu versteuernder Umsatz (Eigenverbrauch) +022, +029, +006, +037,
            +052, +007 -->
        <record id="account_tax_template_sales_self_20_code022" model="account.tax.template">
            <field name="name">UST_022 Normalsteuersatz 20% (Eigenverbrauch)</field>
            <field name="description">20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="20" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_2_tag'), ref('tax_report_line_l10n_at_tva_line_4_14_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3500'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_14_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_2_tag'), ref('tax_report_line_l10n_at_tva_line_4_14_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3500'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_14_tax_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_self_10_code029" model="account.tax.template">
            <field name="name">UST_029 ermäßigter Steuersatz 10% (Eigenverbrauch)</field>
            <field name="description">10%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="10" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_10"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_2_tag'), ref('tax_report_line_l10n_at_tva_line_4_15_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3501'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_15_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_2_tag'), ref('tax_report_line_l10n_at_tva_line_4_15_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3501'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_15_tax_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_self_19_code037" model="account.tax.template">
            <field name="name">UST_037 Steuersatz 19% (Eigenverbrauch) </field>
            <field name="description">19%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="19" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_19"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_2_tag'), ref('tax_report_line_l10n_at_tva_line_4_17_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_17_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_2_tag'), ref('tax_report_line_l10n_at_tva_line_4_17_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_17_tax_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_self_add10_code052" model="account.tax.template">
            <field name="name">UST_052 Zusatzsteuersatz 10% (LWB/FWB - Eigenverbrauch)</field>
            <field name="description">10%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="10" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_10"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_2_tag'), ref('tax_report_line_l10n_at_tva_line_4_18_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_18_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_2_tag'), ref('tax_report_line_l10n_at_tva_line_4_18_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_18_tax_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_self_add7_code007" model="account.tax.template">
            <field name="name">UST_007 Zusatzsteuersatz 7% (LWB/FWB - Eigenverbrauch)</field>
            <field name="description">7%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">sale</field>
            <field eval="7" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0" />
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_2_tag'), ref('tax_report_line_l10n_at_tva_line_4_19_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_19_tax_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_2_tag'), ref('tax_report_line_l10n_at_tva_line_4_19_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_19_tax_tag')],
                }),
            ]"/>
        </record>
        <!-- Übergegangene Steuerschuld +056, +057, +048, +044, +032 -->
        <!-- Steuerschuld gemäß § 11 Abs. 12 und 14, § 16 Abs. 2 sowie gemäß
            Art. 7 Abs. 4 -->
        <record id="account_tax_template_purchase_tax_invoiced_accepted_code056" model="account.tax.template">
            <field name="name">UST_056 Tax invoiced accepted (§ 11 Abs. 12 und 14, § 16 Abs. 2 sowie gemäß Art. 7 Abs. 4)</field>
            <field name="description">20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">100</field>
            <field name="type_tax_use">purchase</field>
            <field eval="20" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_20_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_20_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_eu_0_services" model="account.tax.template">
            <field name="name">UST_EU Dienstleistung (Sonstige Leistungen) 0%</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">200</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">service</field>
            <field eval="0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_3_zm_dl_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_3_zm_dl_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
        </record>
        <record id="account_tax_template_sales_non_eu_0_services" model="account.tax.template">
            <field name="name">UST_NON_EU Dienstleistung (Drittstaaten) 0%</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">300</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_scope">service</field>
            <field eval="0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
        </record>
        <!-- Innergemeinschaftlicher Erwerb (Erwerbssteuer) 070 abzgl. 071 -->
        <!-- Davon steuerfrei gemäß Art. 6 Abs. 2 (IGE-UST) -->
        <record id="account_tax_template_purchase_eu_0_code071" model="account.tax.template">
            <field name="name">UST_071 IGE 0% (Art. 6 Abs. 2)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">200</field>
            <field name="type_tax_use">purchase</field>
            <field eval="0" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_25_tag'), ref('tax_report_line_l10n_at_tva_line_4_26_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_25_tag'), ref('tax_report_line_l10n_at_tva_line_4_26_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                }),
            ]"/>
        </record>
        <record id="account_tax_template_purchase_eu_20" model="account.tax.template">
            <field name="name">IGE 20%</field>
            <field name="description">IGE 20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">500</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_25_tag'), ref('tax_report_line_l10n_at_tva_line_4_28_base_tag')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3511'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_28_tax_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2511'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_4_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_25_tag'), ref('tax_report_line_l10n_at_tva_line_4_28_base_tag')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3511'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_28_tax_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2511'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_4_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_purchase_eu_10" model="account.tax.template">
            <field name="name">IGE 10%</field>
            <field name="description">IGE 10%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">500</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="amount">10</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_25_tag'), ref('tax_report_line_l10n_at_tva_line_4_29_base_tag')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3512'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_29_tax_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2512'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_4_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_25_tag'), ref('tax_report_line_l10n_at_tva_line_4_29_base_tag')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3512'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_29_tax_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2512'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_4_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_purchase_eu_13" model="account.tax.template">
            <field name="name">IGE 13%</field>
            <field name="description">IGE 13%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">500</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="amount">13</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_25_tag'), ref('tax_report_line_l10n_at_tva_line_4_30_base_tag')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3513'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_30_tax_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2513'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_4_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_25_tag'), ref('tax_report_line_l10n_at_tva_line_4_30_base_tag')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3513'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_30_tax_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2513'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_4_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_purchase_eu_19" model="account.tax.template">
            <field name="name">IGE 19%</field>
            <field name="description">IGE 19%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">500</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="active" eval="False"/>
            <field name="amount">19</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_25_tag'), ref('tax_report_line_l10n_at_tva_line_4_31_base_tag')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3511'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_31_tax_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_4_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_25_tag'), ref('tax_report_line_l10n_at_tva_line_4_31_base_tag')],
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3511'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_31_tax_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_4_tag')],
                }),
            ]"/>
        </record>

        <!--    § 19 Abs. 1a
                https://www.ris.bka.gv.at/eli/bgbl/1994/663/P19/NOR40189968
                https://www.jusline.at/gesetz/ustg/paragraf/19

                Reverse Charge (§ 19 Abs. 1a - Bauleistungen)
        -->
        <record id="account_tax_template_purchase_rev_charge_1a" model="account.tax.template">
            <field name="name">Reverse Charge 20% (§ 19 Abs. 1a - Bauleistungen)</field>
            <field name="description">RC 20% § 19 Abs. 1a</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">550</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_22_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_6_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_22_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_6_tag')],
                }),
            ]"/>
        </record>

        <!--    § 19 Abs. 1b
                https://www.ris.bka.gv.at/eli/bgbl/1994/663/P19/NOR40189968
                https://www.jusline.at/gesetz/ustg/paragraf/19

                Reverse Charge gemäß § 19 Abs. 1b (Sicherungseigentum, Vorbehaltseigentum  und
                Grundstücke im Zwangsversteigerungsverfahren)

                a) sicherungsübereigneter Gegenstände durch den Sicherungsgeber an den Sicherungsnehmer,
                b) des Vorbehaltskäufers an den Vorbehaltseigentümer im Falle der vorangegangenen Übertragung des vorbehaltenen Eigentums und
                c) von Grundstücken im Zwangsversteigerungsverfahren durch den Verpflichteten an den Ersteher
        -->
        <record id="account_tax_template_purchase_rev_charge_1b" model="account.tax.template">
            <field name="name">Reverse Charge 20% (§ 19 Abs. 1b - Sicherungseigentum, Vorbehaltseigentum  und
                Grundstücke im Zwangsversteigerungsverfahren))</field>
            <field name="description">RC 20% § 19 Abs. 1b</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">550</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_23_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_7_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_23_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_7_tag')],
                }),
            ]"/>
        </record>

        <!--    § 19 Abs. 1 zweiter Satz
                https://www.ris.bka.gv.at/eli/bgbl/1994/663/P19/NOR40189968
                https://www.jusline.at/gesetz/ustg/paragraf/19

                Art. 25 Abs. 5 (Dreiecksgeschäft)
                https://www.ris.bka.gv.at/GeltendeFassung.wxe?Abfrage=Bundesnormen&Gesetzesnummer=10004929
                https://www.jusline.at/gesetz/ustg-anhang/paragraf/artikel25

                Reverse Charge gemäß § 19 Abs. 1 zweiter Satz, sowie  gemäß Art. 25 Abs. 5 (Dreiecksgeschäft)
        -->
        <record id="account_tax_template_purchase_rev_charge_19_2_25_5" model="account.tax.template">
            <field name="name">Reverse Charge 20% (§ 19 Abs. 1 zweiter Satz - Sonstige Leistungen, Art. 25 Abs. 5 - Dreiecksgeschäft</field>
            <field name="description">RC 20% §19 Sonstige Leistungen, Dreiecksgeschäft</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">550</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_21_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_5_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'minus_report_expression_ids':  [ref('tax_report_line_l10n_at_tva_line_4_21_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'minus_report_expression_ids':  [ref('tax_report_line_l10n_at_tva_line_5_5_tag')],
                }),
            ]"/>
        </record>

        <!--    § 19 Abs. 1c
                https://www.ris.bka.gv.at/eli/bgbl/1994/663/P19/NOR40189968
                https://www.jusline.at/gesetz/ustg/paragraf/19

                Reverse Charge gemäß § 19 Abs. 1c
                Bei der Lieferung von Gas über ein Erdgasnetz im Gebiet der Gemeinschaft oder
                jedes an ein solches Netz angeschlossene Netz, von Elektrizität oder
                von Wärme oder Kälte über Wärme- oder Kältenetze, wenn sich der Ort dieser Lieferung nach § 3 Abs. 13 oder 14 bestimmt und
                der liefernde Unternehmer im Inland weder sein Unternehmen betreibt noch eine an der Lieferung beteiligte Betriebsstätte hat,
                wird die Steuer vom Empfänger der Lieferung geschuldet, wenn er im Inland für Zwecke der Umsatzsteuer erfasst ist.
        -->
        <record id="account_tax_template_purchase_rev_charge_1c" model="account.tax.template">
            <field name="name">Reverse Charge 20% (§ 19 Abs. 1c - Gas, Strom, Wärme, Kälte)</field>
            <field name="description">RC 20% § 19 Abs. 1c</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">550</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_21_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_5_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_21_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_5_tag')],
                }),
            ]"/>
        </record>

        <!--    § 19 Abs. 1d
                https://www.ris.bka.gv.at/eli/bgbl/1994/663/P19/NOR40189968
                https://www.jusline.at/gesetz/ustg/paragraf/19

                Reverse Charge gemäß § 19 Abs. 1d (Schrott und Abfallstoffe, Verordnung
                BGBl. II Nr. 129/2007; Videospielkonsolen, Laptops, Tablet-Computer, Gas
                und Elektrizität, Gas- und Elektrizitätszertifikate, Metalle, Anlagegold,
                Verordnung BGBl. II Nr. 369/2013)
        -->
        <record id="account_tax_template_purchase_rev_charge_1d" model="account.tax.template">
            <field name="name">Reverse Charge 20% (§ 19 Abs. 1d - Schrott und Abfallstoffe, Spielekonsolen, Laptops, Tablet-Computer >= EUR 5.000,-, Gas und Elektrizität, Gas- und Elektrizitätszertifikate, Metalle, Anlagegold)</field>
            <field name="description">RC 20% § 19 Abs. 1d</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">550</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_24_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_8_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_24_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_8_tag')],
                }),
            ]"/>
        </record>

        <!--    § 19 Abs. 1e
                https://www.ris.bka.gv.at/eli/bgbl/1994/663/P19/NOR40189968
                https://www.jusline.at/gesetz/ustg/paragraf/19

                Reverse Charge gemäß § 19 Abs. 1e
                a)  der Übertragung von Treibhausgasemissionszertifikaten im Sinne des Art. 3 der Richtlinie 2003/87/EG
                    über ein System für den Handel mit Treibhausgasemissionszertifikaten in der Gemeinschaft und
                    zur Änderung der Richtlinie 96/61/EG des Rates, ABl. Nr. L 275 vom 25.10.2003 S. 32, und bei
                    der Übertragung von anderen Einheiten, die genutzt werden können, um den Auflagen dieser Richtlinie nachzukommen,
                b)  der Lieferung von Mobilfunkgeräten (Unterpositionen 8517 12 00 und 8517 18 00 der Kombinierten Nomenklatur)
                    und integrierten Schaltkreisen (Unterpositionen 8542 31 90, 8473 30 20, 8473 30 80 und 8471 50 00 der Kombinierten Nomenklatur),
                    wenn das in der Rechnung ausgewiesene Entgelt mindestens 5 000 Euro beträgt.
        -->
        <record id="account_tax_template_purchase_rev_charge_1e" model="account.tax.template">
            <field name="name">Reverse Charge 20% (§ 19 Abs. 1e - Treibhausgasemissionszertifikaten, Mobilfunkgeräte >= EUR 5.000,-)</field>
            <field name="description">RC 20% § 19 Abs. 1e</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">550</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="amount">20</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_24_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_8_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'factor_percent': -100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3510'),
                  'minus_report_expression_ids':  [ref('tax_report_line_l10n_at_tva_line_4_24_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2510'),
                  'minus_report_expression_ids':  [ref('tax_report_line_l10n_at_tva_line_5_8_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_purchase_eu_xx_code076" model="account.tax.template">
            <field name="name">Erwerbe gemäß Art. 3 Abs. 8 zweiter Satz, die im Mitgliedstaat des Bestimmungslandes besteuert worden sind (IGE-UST)</field>
            <field name="description">UST_076 IGE (im Bestimmungsland besteuert)</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field eval="0.0" name="amount"/>
            <field name="sequence">200</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_32_tag'), ref('tax_report_line_l10n_at_tva_line_4_33_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_32_tag'), ref('tax_report_line_l10n_at_tva_line_4_33_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
        </record>
        <record id="account_tax_template_purchase_eu_xx_code077" model="account.tax.template">
            <field name="name">Erwerbe gemäß Art. 3 Abs. 8 zweiter Satz, die gemäß Art. 25 Abs. 2 im Inland als besteuert gelten (IGE-UST)</field>
            <field name="description">UST_077 IGE (im Inland besteuert)</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field eval="0.0" name="amount"/>
            <field name="sequence">200</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_33_tag'), ref('tax_report_line_l10n_at_tva_line_4_17_base_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_33_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_33_tag'), ref('tax_report_line_l10n_at_tva_line_4_17_base_tag')],
                }),
                (0,0,{
                  'factor_percent': 100,
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_3505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_33_tag')],
                }),
            ]"/>
        </record>
        <!-- Vorlagen: Einkauf (Vorsteuer) -->
        <!-- Vorsteuern -060, -061, -083, -065(070), -066(057), -082, -087,
            -089, !-064, +062, -063, -067! -->
        <record id="account_tax_template_purchase_20_code060" model="account.tax.template">
            <field name="name">VST_060 Normalsteuersatz 20%</field>
            <field name="description">20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">50</field>
            <field name="type_tax_use">purchase</field>
            <field eval="20" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2500'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_1_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2500'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_1_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_purchase_20_misc_code060" model="account.tax.template">
            <field name="name">VST_060 sonstige Leistungen 20%</field>
            <field name="description">20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field eval="20" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2500'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_1_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2500'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_1_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_purchase_10_code060" model="account.tax.template">
            <field name="name">VST_060 ermäßigter Steuersatz 10%</field>
            <field name="description">10%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field eval="10" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_10"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2501'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_1_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2501'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_1_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_purchase_13_code060" model="account.tax.template">
            <field name="name">VST_060 ermäßigter Steuersatz 13%</field>
            <field name="description">13%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field eval="13" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_13"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2502'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_1_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2502'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_1_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_purchase_19_code060" model="account.tax.template">
            <field name="name">VST_060 Jungholz und Mittelberg 19%</field>
            <field name="description">19%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field eval="19" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_19"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_1_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_1_tag')],
                }),
            ]"/>
        </record>
        <record id="account_tax_template_purchase_12_code060" model="account.tax.template">
            <field name="name">VST_060 Weineinkauf 12% (LWB)</field>
            <field name="description">12%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field eval="12" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_1_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_1_tag')],
                }),
            ]"/>
        </record>
        <!-- Vorsteuern betreffend die entrichtete Einfuhrumsatzsteuer (§12 Abs. 1 Z 2 lit. a) -->
        <record id="account_tax_template_purchase_xx_code061" model="account.tax.template">
            <field name="name">VST_061 entrichtete EUst (§ 12 Abs. 1 Z 2 lit. a)</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field name="amount_type">percent</field>
        </record>
        <!-- Vorsteuern betreffend die geschuldete, auf dem Abgabenkonto
            verbuchte Einfuhrumsatzsteuer (§ 12 Abs. 1 Z 2 lit. b) -->
        <record id="account_tax_template_purchase_xx_code083" model="account.tax.template">
            <field name="name">VST_083 verbuchte EUst. (§ 12 Abs. 1 Z 2 lit. b)</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">400</field>
            <field name="type_tax_use">none</field>
            <field name="amount_type">percent</field>
        </record>
        <!-- Berichtigungen -063, -067, -090 ! -->
        <!-- Berichtigung gemäß § 12 Abs. 10 und 11 -->
        <record id="account_tax_template_purchase_correct_code063" model="account.tax.template">
            <field name="name">VST_063 (§12 Abs. 10 und 11 - Berichtigung)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field eval="0.00" name="amount"/>
            <field name="sequence">400</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
        </record>
        <!-- Berichtigung gemäß § 16 -->
        <record id="account_tax_template_purchase_correct_code067" model="account.tax.template">
            <field name="name">VST_067 (§ 16 - Berichtigung)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field eval="0.00" name="amount"/>
            <field name="sequence">400</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_12_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_12_tag')],
                }),
            ]"/>
        </record>
        <!-- Sonstige Berichtigungen -->
        <record id="account_tax_template_purchase_correct_code090" model="account.tax.template">
            <field name="name">VST_090 (Sonstige Berichtigungen)</field>
            <field name="description">0%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field eval="0.00" name="amount"/>
            <field name="sequence">600</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="active" eval="False"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_6_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_6_tag')],
                }),
            ]"/>
        </record>
        <!-- Vorsteuern 027, 028 in 060/065 -->
        <!-- Vorsteuern betreffend KFZ nach EKR 063, 064, 732-733 und 744-747 -->
        <record id="account_tax_template_purchase_cars_buildings_code027" model="account.tax.template">
            <field name="name">VST_027 betreffend KFZ nach EKR 063, 064, 732-733 und 744-747</field>
            <field name="description">20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">400</field>
            <field name="type_tax_use">purchase</field>
            <field eval="20" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
        </record>
        <!-- Vorsteuern betreffend Gebäude nach EKR 030-037 und 070, 071 -->
        <record id="account_tax_template_purchase_cars_buildings_code028" model="account.tax.template">
            <field name="name">VST_028 betreffend Gebäude nach EKR 030-037 und 070, 071</field>
            <field name="description">20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field eval="20" name="amount"/>
            <field name="sequence">400</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
        </record>
        <!-- Innergemeinschaftlicher Erwerb 070 abzgl. 071 ! -->
        <record id="account_tax_template_purchase_eu_0_vst_071" model="account.tax.template">
            <field name="name">VST_071 IGE 0%</field>
            <field name="description">VST_071 IGE 0% (Art. 6 Abs. 2)</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field eval="0" name="amount"/>
            <field name="sequence">500</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_4_tag')],
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_5_4_tag')],
                }),
            ]"/>
        </record>
        <!-- Erwerbe gemäß Art. 3 Abs. 8 zweiter Satz, die im Mitgliedstaat
            des Bestimmungslandes besteuert worden sind (IGE-VST) -->
        <record id="account_tax_template_purchase_eu_xx_vst_076" model="account.tax.template">
            <field name="name">VST_076 IGE (im Bestimmungsland besteuert)</field>
            <field name="description">20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">500</field>
            <field name="type_tax_use">purchase</field>
            <field eval="20" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'minus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_32_tag'), ref('tax_report_line_l10n_at_tva_line_4_33_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                  'plus_report_expression_ids': [ref('tax_report_line_l10n_at_tva_line_4_32_tag'), ref('tax_report_line_l10n_at_tva_line_4_33_tag')],
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
        </record>
        <!-- Erwerbe gemäß Art. 3 Abs. 8 zweiter Satz, die gemäß Art. 25
            Abs. 2 im Inland als besteuert gelten (IGE-VST) -->
        <record id="account_tax_template_purchase_eu_xx_vst_077" model="account.tax.template">
            <field name="name">VST_077 IGE (im Inland besteuert)</field>
            <field name="description">20%</field>
            <field name="chart_template_id" ref="l10n_at_chart_template"/>
            <field name="sequence">500</field>
            <field name="type_tax_use">purchase</field>
            <field eval="20" name="amount"/>
            <field name="amount_type">percent</field>
            <field name="price_include" eval="0"/>
            <field name="tax_group_id" ref="tax_group_20"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
                  'repartition_type': 'tax',
                  'account_id': ref('chart_at_template_2505'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0,{
                  'repartition_type': 'base',
                }),
                (0,0,{
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

## File: migrations\3.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.addons.account.models.chart_template import update_taxes_from_templates

def migrate(cr, version):
    update_taxes_from_templates(cr, 'l10n_at.l10n_at_chart_template')

```

## File: models\chart_template.py

```python
# -*- coding: utf-8 -*-
from odoo import models


class AccountChartTemplate(models.Model):
    _inherit = 'account.chart.template'

    # Write paperformat and report template used on company
    def _load(self, company):
        res = super(AccountChartTemplate, self)._load(company)
        if self == self.env.ref('l10n_at.l10n_at_chart_template'):
            company.write({
                'external_report_layout_id': self.env.ref('l10n_din5008.external_layout_din5008').id,
                'paperformat_id': self.env.ref('l10n_din5008.paperformat_euro_din').id
            })
        return res

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import chart_template

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="4.8" width="50.4" height="36.4" maskUnits="userSpaceOnUse">
      <rect x="5.62" y="7.68" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
    <use width="106" height="106" xlink:href="#c"/>
    <rect x="5.64" y="10.57" width="48.45" height="31.57" rx="1" style="fill: #393939;opacity: 0.44;isolation: isolate"/>
    <g style="mask: url(#b)">
      <image width="900" height="600" transform="translate(4.8 4.8) scale(0.06 0.06)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAA4UAAAKLCAYAAABBrS1kAAAACXBIWXMAAMXFAADFxQEUjF4vAAAOi0lEQVR4Xu3ZQRECMQAEQUKh4FyAA/BfJ+FcgIUgIf9M93sNTO34Pt/zBgAAQM5xneO+GgEAALAvUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEiUIAAIAwUQgAABAmCgEAAMJEIQAAQJgoBAAACBOFAAAAYaIQAAAgTBQCAACEjTnnXI0AAADY0vAUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGP3+uz2gAAALCh4zo9hQAAAGWiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgThQAAAGGiEAAAIEwUAgAAhIlCAACAMFEIAAAQJgoBAADCRCEAAECYKAQAAAgbc87VBgAAgE39AedzEw12sBBDAAAAAElFTkSuQmCC"/>
    </g>
  </g>
</svg>

```

