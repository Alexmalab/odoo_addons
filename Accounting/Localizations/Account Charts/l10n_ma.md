# Odoo Module: l10n_ma

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

# Copyright (c) 2010 kazacube (http://kazacube.com).

{
    'name': 'Morocco - Accounting',
    'author': 'kazacube',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the base module to manage the accounting chart for Morocco.
===================================================================

Ce Module charge le modèle du plan de comptes standard Marocain et permet de
générer les états comptables aux normes marocaines (Bilan, CPC (comptes de
produits et charges), balance générale à 6 colonnes, Grand livre cumulatif...).
L'intégration comptable a été validé avec l'aide du Cabinet d'expertise comptable
Seddik au cours du troisième trimestre 2010.""",
    'website': 'http://www.kazacube.com',
    'depends': ['base', 'account'],
    'data': [
        'data/l10n_ma_chart_data.xml',
        'data/account_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_data.xml',
        'data/account_chart_template_data.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_ma.l10n_kzc_temp_chart')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Account Tax Group -->
        <record id="tax_group_tva_0" model="account.tax.group">
            <field name="name">TVA 0%</field>
        </record>
        <record id="tax_group_tva_7" model="account.tax.group">
            <field name="name">TVA 7%</field>
        </record>
        <record id="tax_group_tva_10" model="account.tax.group">
            <field name="name">TVA 10%</field>
        </record>
        <record id="tax_group_tva_14" model="account.tax.group">
            <field name="name">TVA 14%</field>
        </record>
        <record id="tax_group_tva_20" model="account.tax.group">
            <field name="name">TVA 20%</field>
        </record>

    </data>
</odoo>
```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- account chart  -->
    <record id="l10n_kzc_temp_chart" model="account.chart.template">
        <field name="property_account_receivable_id" ref="pcg_34211"/>
        <field name="property_account_payable_id" ref="pcg_4411"/>
        <field name="property_account_income_categ_id" ref="pcg_7111"/>
        <field name="property_account_expense_categ_id" ref="pcg_6111"/>
        <field name="income_currency_exchange_account_id" ref="pcg_733"/>
        <field name="expense_currency_exchange_account_id" ref="pcg_633"/>
        <field name="default_pos_receivable_account_id" ref="pcg_3489" />
    </record>

    <!-- Account Tax Template -->
    <record model="account.tax.template" id="tva_exo">
        <field name="name">Exonere de TVA VENTES</field>
        <field name="description">Exonere de TVA VENTES</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            })
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            })
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_exo1">
        <field name="name">Exonere de TVA ACHATS</field>
        <field name="description">Exonere de TVA ACHATS</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">0</field>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
        <field name="tax_group_id" ref="tax_group_tva_0"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
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
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_vt20">
        <field name="name">TVA 20% VENTES</field>
        <field name="description">TVA 20% VENTES</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_taux_normal_20')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_445520'),
                'plus_report_line_ids': [ref('tax_report_taux_normal_tva_20')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_taux_normal_20')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_445520'),
                'minus_report_line_ids': [ref('tax_report_taux_normal_tva_20')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_vt14">
        <field name="name">TVA 14% VENTES</field>
        <field name="description">TVA 14% VENTES</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
        <field name="tax_group_id" ref="tax_group_tva_14"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_tauz_reduit_ht_14')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_445514'),
                'plus_report_line_ids': [ref('tax_report_taux_rediut_tva_14')],
            }),]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                  'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_tauz_reduit_ht_14')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_445514'),
                'minus_report_line_ids': [ref('tax_report_taux_rediut_tva_14')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_vt10">
        <field name="name">TVA 10% VENTES</field>
        <field name="description">TVA 10% VENTES</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
        <field name="tax_group_id" ref="tax_group_tva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('account_report_taux_reduit_base_ht_10')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_445510'),
                'plus_report_line_ids': [ref('tax_report_taux_reduit_tva_10')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('account_report_taux_reduit_base_ht_10')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_445510'),
                'minus_report_line_ids': [ref('tax_report_taux_reduit_tva_10')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_vt07">
        <field name="name">TVA 7% VENTES</field>
        <field name="description">TVA 7% VENTES</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
        <field name="tax_group_id" ref="tax_group_tva_7"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_taux_reduit_ht_7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_445507'),
                'plus_report_line_ids': [ref('tax_report_taux_reduit_tva_7')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_taux_reduit_ht_7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_445507'),
                'minus_report_line_ids': [ref('tax_report_taux_reduit_tva_7')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_ac20">
        <field name="name">TVA 20% ACHATS</field>
        <field name="description">TVA 20% ACHATS</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_achats_importation_ht_20')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455220'),
                'plus_report_line_ids': [ref('tax_report_achats_importation_tva_20')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_achats_importation_ht_20')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455220'),
                'minus_report_line_ids': [ref('tax_report_achats_importation_tva_20')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_acim">
        <field name="name">TVA 20% ACHATS (immobilisation)</field>
        <field name="description">TVA 20% ACHATS (immobilisation)</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">20</field>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
        <field name="tax_group_id" ref="tax_group_tva_20"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_immobilisations_base_ht')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_34551'),
                'plus_report_line_ids': [ref('tax_report_immobilisations_deductible_tva')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_immobilisations_base_ht')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_34551'),
                'minus_report_line_ids': [ref('tax_report_immobilisations_deductible_tva')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_ac14">
        <field name="name">TVA 14% ACHATS</field>
        <field name="description">TVA 14% ACHATS</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
        <field name="tax_group_id" ref="tax_group_tva_14"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_achats_importation_ht_14')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455214'),
                'plus_report_line_ids': [ref('tax_report_achats_importation_tva_14')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_achats_importation_ht_14')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455214'),
                'minus_report_line_ids': [ref('tax_report_achats_importation_tva_14')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_ac10">
        <field name="name">TVA 10% ACHATS</field>
        <field name="description">TVA 10% ACHATS</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">10</field>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
        <field name="tax_group_id" ref="tax_group_tva_10"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_achats_importation_ht_10')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455210'),
                'plus_report_line_ids': [ref('tax_report_achats_importation_tva_10')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_achats_importation_ht_10')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455210'),
                'minus_report_line_ids': [ref('tax_report_achats_importation_tva_10')],
            }),
        ]"/>
    </record>

    <record model="account.tax.template" id="tva_ac07">
        <field name="name">TVA 7% ACHATS</field>
        <field name="description">TVA 7% ACHATS</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
        <field name="tax_group_id" ref="tax_group_tva_7"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_achat_importation_ht_7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455207'),
                'plus_report_line_ids': [ref('tax_report_achats_importation_tva_7')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_achat_importation_ht_7')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455207'),
                'minus_report_line_ids': [ref('tax_report_achats_importation_tva_7')],
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
        <field name="country_id" ref="base.ma"/>
    </record>

    <record id="tax_report_ventilation_chiffre_affaires_imposable" model="account.tax.report.line">
        <field name="name">Ventilation du chiffre d’affaires imposable</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_taux_normal_20" model="account.tax.report.line">
        <field name="name">TAUX NORMAL DE 20% (Base HT)</field>
        <field name="tag_name">TAUX NORMAL DE 20% (Base HT)</field>
        <field name="parent_id" ref="tax_report_ventilation_chiffre_affaires_imposable"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="code">MATAXB_1</field>
    </record>

    <record id="tax_report_taux_normal_tva_20" model="account.tax.report.line">
        <field name="name">TAUX NORMAL DE 20% (TVA exigible)</field>
        <field name="tag_name">TAUX NORMAL DE 20% (TVA exigible)</field>
        <field name="parent_id" ref="tax_report_ventilation_chiffre_affaires_imposable"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="code">MATAXB_2</field>
    </record>

     <record id="tax_report_tauz_reduit_ht_14" model="account.tax.report.line">
        <field name="name">TAUX REDUIT DE 14% (Base HT)</field>
        <field name="tag_name">TAUX REDUIT DE 14% (Base HT)</field>
        <field name="parent_id" ref="tax_report_ventilation_chiffre_affaires_imposable"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="code">MATAXB_3</field>
    </record>

    <record id="tax_report_taux_rediut_tva_14" model="account.tax.report.line">
        <field name="name">TAUX REDUIT DE 14% (TVA exigible)</field>
        <field name="tag_name">TAUX REDUIT DE 14% (TVA exigible)</field>
        <field name="parent_id" ref="tax_report_ventilation_chiffre_affaires_imposable"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="code">MATAXB_4</field>
    </record>

    <record id="account_report_taux_reduit_base_ht_10" model="account.tax.report.line">
        <field name="name">TAUX REDUIT DE 1O% (Base HT)</field>
        <field name="tag_name">TAUX REDUIT DE 1O% (Base HT)</field>
        <field name="parent_id" ref="tax_report_ventilation_chiffre_affaires_imposable"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="code">MATAXB_5</field>
    </record>

    <record id="tax_report_taux_reduit_tva_10" model="account.tax.report.line">
        <field name="name">TAUX REDUIT DE 1O% (TVA exigible)</field>
        <field name="tag_name">TAUX REDUIT DE 1O% (TVA exigible)</field>
        <field name="parent_id" ref="tax_report_ventilation_chiffre_affaires_imposable"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
        <field name="code">MATAXB_6</field>
    </record>

    <record id="tax_report_taux_reduit_ht_7" model="account.tax.report.line">
        <field name="name">TAUX REDUIT DE 7% (Base HT)</field>
        <field name="tag_name">TAUX REDUIT DE 7% (Base HT)</field>
        <field name="parent_id" ref="tax_report_ventilation_chiffre_affaires_imposable"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
        <field name="code">MATAXB_7</field>
    </record>

    <record id="tax_report_taux_reduit_tva_7" model="account.tax.report.line">
        <field name="name">TAUX REDUIT DE 7% (TVA exigible)</field>
        <field name="tag_name">TAUX REDUIT DE 7% (TVA exigible)</field>
        <field name="parent_id" ref="tax_report_ventilation_chiffre_affaires_imposable"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
        <field name="code">MATAXB_8</field>
    </record>

    <record id="tax_report_ventilation_des_deductions" model="account.tax.report.line">
        <field name="name">Ventilation des déductions</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_achats_non_immobilises_ht" model="account.tax.report.line">
        <field name="name">1-ACHATS NON IMMOBILISES (Base HT)</field>
        <field name="parent_id" ref="tax_report_ventilation_des_deductions"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_travaux_facons_ht" model="account.tax.report.line">
        <field name="name">Travaux à façons (HT)</field>
        <field name="parent_id" ref="tax_report_achats_non_immobilises_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_sous_traitance_immobiliers_ht" model="account.tax.report.line">
        <field name="name">Sous-traitance (travaux immobiliers) (HT)</field>
        <field name="parent_id" ref="tax_report_achats_non_immobilises_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_biens_materiels_ht" model="account.tax.report.line">
        <field name="name">Biens matériels (Base HT)</field>
        <field name="parent_id" ref="tax_report_achats_non_immobilises_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_achats_importation_ht_20" model="account.tax.report.line">
        <field name="name">Achats à l'importation (20%) (HT)</field>
        <field name="tag_name">Achats à l'importation (20%) (HT)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_achats_interieur_ht_20" model="account.tax.report.line">
        <field name="name">Achats à l'intérieur (20%) (HT)</field>
        <field name="tag_name">Achats à l'intérieur (20%) (HT)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_achats_importation_ht_14" model="account.tax.report.line">
        <field name="name">Achats à l'importation (14%) (HT)</field>
        <field name="tag_name">Achats à l'importation (14%) (HT)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_achats_interieur_ht_14" model="account.tax.report.line">
        <field name="name">Achats à l'intérieur (14%) (HT)</field>
        <field name="tag_name">Achats à l'intérieur (14%) (HT)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_achats_importation_ht_10" model="account.tax.report.line">
        <field name="name">Achats à l'importation (10%) (HT)</field>
        <field name="tag_name">Achats à l'importation (10%) (HT)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
    </record>

    <record id="tax_report_achats_interieur_ht_10" model="account.tax.report.line">
        <field name="name">Achats à l'intérieur (10%) (HT)</field>
        <field name="tag_name">Achats à l'intérieur (10%) (HT)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
    </record>

    <record id="tax_report_achat_importation_ht_7" model="account.tax.report.line">
        <field name="name">Achat à l'importation (7%) (HT)</field>
        <field name="tag_name">Achat à l'importation (7%) (HT)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
    </record>

    <record id="tax_report_achats_interieur_ht_7" model="account.tax.report.line">
        <field name="name">Achat à l'intérieur (7%) (HT)</field>
        <field name="tag_name">Achat à l'intérieur (7%) (HT)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
    </record>

    <record id="tax_report_prestations_services_ht_1" model="account.tax.report.line">
        <field name="name">Prestations de services (Base HT)</field>
        <field name="parent_id" ref="tax_report_achats_non_immobilises_ht"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_achats_immobilises_deductible_tva" model="account.tax.report.line">
        <field name="name">1-ACHATS NON IMMOBILISES (TVA déductible)</field>
        <field name="parent_id" ref="tax_report_ventilation_des_deductions"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_travaux_facons_tva" model="account.tax.report.line">
        <field name="name">Travaux à façons (TVA déductible)</field>
        <field name="parent_id" ref="tax_report_achats_immobilises_deductible_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_sous_traitance_immobiliers_tva" model="account.tax.report.line">
        <field name="name">Sous-traitance (travaux immobiliers) (TVA déductible)</field>
        <field name="parent_id" ref="tax_report_achats_immobilises_deductible_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_biens_materiels_tva" model="account.tax.report.line">
        <field name="name">Biens matériels (TVA déductible)</field>
        <field name="parent_id" ref="tax_report_achats_immobilises_deductible_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_achats_importation_tva_20" model="account.tax.report.line">
        <field name="name">Achats à l'importation (20%) (TVA)</field>
        <field name="tag_name">Achats à l'importation (20%) (TVA)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_achats_interieur_tva_20" model="account.tax.report.line">
        <field name="name">Achats à l'intérieur (20%) (TVA)</field>
        <field name="tag_name">Achats à l'intérieur (20%) (TVA)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_achats_importation_tva_14" model="account.tax.report.line">
        <field name="name">Achats à l'importation (14%) (TVA)</field>
        <field name="tag_name">Achats à l'importation (14%) (TVA)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_achats_interieur_tva_14" model="account.tax.report.line">
        <field name="name">Achats à l'intérieur (14%) (TVA)</field>
        <field name="tag_name">Achats à l'intérieur (14%) (TVA)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_achats_importation_tva_10" model="account.tax.report.line">
        <field name="name">Achats à l'importation (10%) (TVA)</field>
        <field name="tag_name">Achats à l'importation (10%) (TVA)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
    </record>

    <record id="tax_report_achats_interieur_tva_10" model="account.tax.report.line">
        <field name="name">Achats à l'intérieur (10%) (TVA)</field>
        <field name="tag_name">Achats à l'intérieur (10%) (TVA)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
    </record>

    <record id="tax_report_achats_importation_tva_7" model="account.tax.report.line">
        <field name="name">Achat à l'importation (7%) (TVA)</field>
        <field name="tag_name">Achat à l'importation (7%) (TVA)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="7"/>
    </record>

    <record id="tax_report_achats_interieur_tva_7" model="account.tax.report.line">
        <field name="name">Achat à l'intérieur (7%) (TVA)</field>
        <field name="tag_name">Achat à l'intérieur (7%) (TVA)</field>
        <field name="parent_id" ref="tax_report_biens_materiels_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="8"/>
    </record>

    <record id="tax_report_prestations_service_deductible_tva" model="account.tax.report.line">
        <field name="name">Prestations de services (TVA déductible)</field>
        <field name="parent_id" ref="tax_report_achats_immobilises_deductible_tva"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_immobilisations_base_ht" model="account.tax.report.line">
        <field name="name">2-IMMOBILISATIONS (Base HT)</field>
        <field name="tag_name">Immobilisations (Base HT)</field>
        <field name="parent_id" ref="tax_report_ventilation_des_deductions"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_immobilisations_deductible_tva" model="account.tax.report.line">
        <field name="name">2-IMMOBILISATIONS (TVA déductible)</field>
        <field name="tag_name">Immobilisations (TVA déductible)</field>
        <field name="parent_id" ref="tax_report_ventilation_des_deductions"/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>
</odoo>

```

