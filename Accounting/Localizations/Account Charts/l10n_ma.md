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
====================================================================

Ce Module charge le modèle du plan de comptes standard Marocain et permet de
générer les états comptables aux normes marocaines (Bilan, CPC (comptes de
produits et charges), balance générale à 6 colonnes, Grand livre cumulatif...).
L'intégration comptable a été validé avec l'aide du Cabinet d'expertise comptable
Seddik au cours du troisième trimestre 2010.""",
    'website': 'http://www.kazacube.com',
    'depends': ['base', 'account'],
    'data': [
        'data/l10n_ma_chart_data.xml',
        'data/account_tax_group_data.xml',
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
        <field name="default_pos_receivable_account_id" ref="pcg_3489"/>
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'})         ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'})         ]"/>
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
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {'repartition_type': 'tax'}),
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_taux_normal_20_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_445520'),
                'plus_report_expression_ids': [ref('tax_report_taux_normal_tva_20_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_taux_normal_20_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_445520'),
                'minus_report_expression_ids': [ref('tax_report_taux_normal_tva_20_tag')],
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_tauz_reduit_ht_14_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_445514'),
                'plus_report_expression_ids': [ref('tax_report_taux_rediut_tva_14_tag')],
            }),]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                  'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_tauz_reduit_ht_14_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_445514'),
                'minus_report_expression_ids': [ref('tax_report_taux_rediut_tva_14_tag')],
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('account_report_taux_reduit_base_ht_10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_445510'),
                'plus_report_expression_ids': [ref('tax_report_taux_reduit_tva_10_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('account_report_taux_reduit_base_ht_10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_445510'),
                'minus_report_expression_ids': [ref('tax_report_taux_reduit_tva_10_tag')],
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_taux_reduit_ht_7_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_445507'),
                'plus_report_expression_ids': [ref('tax_report_taux_reduit_tva_7_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_taux_reduit_ht_7_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_445507'),
                'minus_report_expression_ids': [ref('tax_report_taux_reduit_tva_7_tag')],
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_achats_importation_ht_20_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455220'),
                'plus_report_expression_ids': [ref('tax_report_achats_importation_tva_20_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_achats_importation_ht_20_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455220'),
                'minus_report_expression_ids': [ref('tax_report_achats_importation_tva_20_tag')],
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_immobilisations_base_ht_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_34551'),
                'plus_report_expression_ids': [ref('tax_report_immobilisations_deductible_tva_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_immobilisations_base_ht_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_34551'),
                'minus_report_expression_ids': [ref('tax_report_immobilisations_deductible_tva_tag')],
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_achats_importation_ht_14_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455214'),
                'plus_report_expression_ids': [ref('tax_report_achats_importation_tva_14_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_achats_importation_ht_14_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455214'),
                'minus_report_expression_ids': [ref('tax_report_achats_importation_tva_14_tag')],
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_achats_importation_ht_10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455210'),
                'plus_report_expression_ids': [ref('tax_report_achats_importation_tva_10_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_achats_importation_ht_10_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455210'),
                'minus_report_expression_ids': [ref('tax_report_achats_importation_tva_10_tag')],
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
                'repartition_type': 'base',
                'plus_report_expression_ids': [ref('tax_report_achat_importation_ht_7_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455207'),
                'plus_report_expression_ids': [ref('tax_report_achats_importation_tva_7_tag')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'minus_report_expression_ids': [ref('tax_report_achat_importation_ht_7_tag')],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('pcg_3455207'),
                'minus_report_expression_ids': [ref('tax_report_achats_importation_tva_7_tag')],
            }),
        ]"/>
    </record>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_tva_0" model="account.tax.group">
            <field name="name">TVA 0%</field>
            <field name="country_id" ref="base.ma"/>
        </record>
        <record id="tax_group_tva_7" model="account.tax.group">
            <field name="name">TVA 7%</field>
            <field name="country_id" ref="base.ma"/>
        </record>
        <record id="tax_group_tva_10" model="account.tax.group">
            <field name="name">TVA 10%</field>
            <field name="country_id" ref="base.ma"/>
        </record>
        <record id="tax_group_tva_14" model="account.tax.group">
            <field name="name">TVA 14%</field>
            <field name="country_id" ref="base.ma"/>
        </record>
        <record id="tax_group_tva_20" model="account.tax.group">
            <field name="name">TVA 20%</field>
            <field name="country_id" ref="base.ma"/>
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
        <field name="country_id" ref="base.ma"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_ventilation_chiffre_affaires_imposable" model="account.report.line">
                <field name="name">Ventilation du chiffre d’affaires imposable</field>
                <field name="aggregation_formula">MATAXB_1.balance + MATAXB_2.balance + MATAXB_3.balance + MATAXB_4.balance + MATAXB_5.balance + MATAXB_6.balance + MATAXB_7.balance + MATAXB_8.balance</field>
                <field name="children_ids">
                    <record id="tax_report_taux_normal_20" model="account.report.line">
                        <field name="name">TAUX NORMAL DE 20% (Base HT)</field>
                        <field name="code">MATAXB_1</field>
                        <field name="expression_ids">
                            <record id="tax_report_taux_normal_20_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TAUX NORMAL DE 20% (Base HT)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_taux_normal_tva_20" model="account.report.line">
                        <field name="name">TAUX NORMAL DE 20% (TVA exigible)</field>
                        <field name="code">MATAXB_2</field>
                        <field name="expression_ids">
                            <record id="tax_report_taux_normal_tva_20_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TAUX NORMAL DE 20% (TVA exigible)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_tauz_reduit_ht_14" model="account.report.line">
                        <field name="name">TAUX REDUIT DE 14% (Base HT)</field>
                        <field name="code">MATAXB_3</field>
                        <field name="expression_ids">
                            <record id="tax_report_tauz_reduit_ht_14_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TAUX REDUIT DE 14% (Base HT)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_taux_rediut_tva_14" model="account.report.line">
                        <field name="name">TAUX REDUIT DE 14% (TVA exigible)</field>
                        <field name="code">MATAXB_4</field>
                        <field name="expression_ids">
                            <record id="tax_report_taux_rediut_tva_14_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TAUX REDUIT DE 14% (TVA exigible)</field>
                            </record>
                        </field>
                    </record>
                    <record id="account_report_taux_reduit_base_ht_10" model="account.report.line">
                        <field name="name">TAUX REDUIT DE 1O% (Base HT)</field>
                        <field name="code">MATAXB_5</field>
                        <field name="expression_ids">
                            <record id="account_report_taux_reduit_base_ht_10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TAUX REDUIT DE 1O% (Base HT)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_taux_reduit_tva_10" model="account.report.line">
                        <field name="name">TAUX REDUIT DE 1O% (TVA exigible)</field>
                        <field name="code">MATAXB_6</field>
                        <field name="expression_ids">
                            <record id="tax_report_taux_reduit_tva_10_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TAUX REDUIT DE 1O% (TVA exigible)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_taux_reduit_ht_7" model="account.report.line">
                        <field name="name">TAUX REDUIT DE 7% (Base HT)</field>
                        <field name="code">MATAXB_7</field>
                        <field name="expression_ids">
                            <record id="tax_report_taux_reduit_ht_7_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TAUX REDUIT DE 7% (Base HT)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_taux_reduit_tva_7" model="account.report.line">
                        <field name="name">TAUX REDUIT DE 7% (TVA exigible)</field>
                        <field name="code">MATAXB_8</field>
                        <field name="expression_ids">
                            <record id="tax_report_taux_reduit_tva_7_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">TAUX REDUIT DE 7% (TVA exigible)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_ventilation_des_deductions" model="account.report.line">
                <field name="name">Ventilation des déductions</field>
                <field name="aggregation_formula">1ACHATS_NON_IMMOBILISES_BASE_HT.balance + 1ACHATS_NON_IMMOBILISES_TVA_DEDUCTIBLE.balance + IMMOBILISATIONS_BASE_HT.balance + IMMOBILISATIONS_TVA_DEDUCTIBLE.balance</field>
                <field name="children_ids">
                    <record id="tax_report_achats_non_immobilises_ht" model="account.report.line">
                        <field name="name">1-ACHATS NON IMMOBILISES (Base HT)</field>
                        <field name="code">1ACHATS_NON_IMMOBILISES_BASE_HT</field>
                        <field name="aggregation_formula">TRAVAUX_A_FACONS_HT.balance + SOUSTRAITANCE_TRAVAUX_IMMOBILIERS_HT.balance + BIENS_MATERIELS_BASE_HT.balance + PRESTATIONS_DE_SERVICES_BASE_HT.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_travaux_facons_ht" model="account.report.line">
                                <field name="name">Travaux à façons (HT)</field>
                                <field name="code">TRAVAUX_A_FACONS_HT</field>
                                <field name="expression_ids">
                                    <record id="tax_report_travaux_facons_ht_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_sous_traitance_immobiliers_ht" model="account.report.line">
                                <field name="name">Sous-traitance (travaux immobiliers) (HT)</field>
                                <field name="code">SOUSTRAITANCE_TRAVAUX_IMMOBILIERS_HT</field>
                                <field name="expression_ids">
                                    <record id="tax_report_sous_traitance_immobiliers_ht_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_biens_materiels_ht" model="account.report.line">
                                <field name="name">Biens matériels (Base HT)</field>
                                <field name="code">BIENS_MATERIELS_BASE_HT</field>
                                <field name="aggregation_formula">ACHATS_A_LIMPORTATION_20_HT.balance + ACHATS_A_LINTERIEUR_20_HT.balance + ACHATS_A_LIMPORTATION_14_HT.balance + ACHATS_A_LINTERIEUR_14_HT.balance + ACHATS_A_LIMPORTATION_10_HT.balance + ACHATS_A_LINTERIEUR_10_HT.balance + ACHAT_A_LIMPORTATION_7_HT.balance + ACHAT_A_LINTERIEUR_7_HT.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_achats_importation_ht_20" model="account.report.line">
                                        <field name="name">Achats à l'importation (20%) (HT)</field>
                                        <field name="code">ACHATS_A_LIMPORTATION_20_HT</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_importation_ht_20_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achats à l'importation (20%) (HT)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_achats_interieur_ht_20" model="account.report.line">
                                        <field name="name">Achats à l'intérieur (20%) (HT)</field>
                                        <field name="code">ACHATS_A_LINTERIEUR_20_HT</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_interieur_ht_20_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achats à l'intérieur (20%) (HT)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_achats_importation_ht_14" model="account.report.line">
                                        <field name="name">Achats à l'importation (14%) (HT)</field>
                                        <field name="code">ACHATS_A_LIMPORTATION_14_HT</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_importation_ht_14_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achats à l'importation (14%) (HT)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_achats_interieur_ht_14" model="account.report.line">
                                        <field name="name">Achats à l'intérieur (14%) (HT)</field>
                                        <field name="code">ACHATS_A_LINTERIEUR_14_HT</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_interieur_ht_14_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achats à l'intérieur (14%) (HT)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_achats_importation_ht_10" model="account.report.line">
                                        <field name="name">Achats à l'importation (10%) (HT)</field>
                                        <field name="code">ACHATS_A_LIMPORTATION_10_HT</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_importation_ht_10_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achats à l'importation (10%) (HT)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_achats_interieur_ht_10" model="account.report.line">
                                        <field name="name">Achats à l'intérieur (10%) (HT)</field>
                                        <field name="code">ACHATS_A_LINTERIEUR_10_HT</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_interieur_ht_10_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achats à l'intérieur (10%) (HT)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_achat_importation_ht_7" model="account.report.line">
                                        <field name="name">Achat à l'importation (7%) (HT)</field>
                                        <field name="code">ACHAT_A_LIMPORTATION_7_HT</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achat_importation_ht_7_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achat à l'importation (7%) (HT)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_achats_interieur_ht_7" model="account.report.line">
                                        <field name="name">Achat à l'intérieur (7%) (HT)</field>
                                        <field name="code">ACHAT_A_LINTERIEUR_7_HT</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_interieur_ht_7_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achat à l'intérieur (7%) (HT)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_prestations_services_ht_1" model="account.report.line">
                                <field name="name">Prestations de services (Base HT)</field>
                                <field name="code">PRESTATIONS_DE_SERVICES_BASE_HT</field>
                                <field name="expression_ids">
                                    <record id="tax_report_prestations_services_ht_1_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_achats_immobilises_deductible_tva" model="account.report.line">
                        <field name="name">1-ACHATS NON IMMOBILISES (TVA déductible)</field>
                        <field name="code">1ACHATS_NON_IMMOBILISES_TVA_DEDUCTIBLE</field>
                        <field name="aggregation_formula">TRAVAUX_A_FACONS_TVA_DEDUCTIBLE.balance + SOUSTRAITANCE_TRAVAUX_IMMOBILIERS_TVA_DEDUCTIBLE.balance + BIENS_MATERIELS_TVA_DEDUCTIBLE.balance + PRESTATIONS_DE_SERVICES_TVA_DEDUCTIBLE.balance</field>
                        <field name="children_ids">
                            <record id="tax_report_travaux_facons_tva" model="account.report.line">
                                <field name="name">Travaux à façons (TVA déductible)</field>
                                <field name="code">TRAVAUX_A_FACONS_TVA_DEDUCTIBLE</field>
                                <field name="expression_ids">
                                    <record id="tax_report_travaux_facons_tva_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_sous_traitance_immobiliers_tva" model="account.report.line">
                                <field name="name">Sous-traitance (travaux immobiliers) (TVA déductible)</field>
                                <field name="code">SOUSTRAITANCE_TRAVAUX_IMMOBILIERS_TVA_DEDUCTIBLE</field>
                                <field name="expression_ids">
                                    <record id="tax_report_sous_traitance_immobiliers_tva_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_biens_materiels_tva" model="account.report.line">
                                <field name="name">Biens matériels (TVA déductible)</field>
                                <field name="code">BIENS_MATERIELS_TVA_DEDUCTIBLE</field>
                                <field name="aggregation_formula">ACHATS_A_LIMPORTATION_20_TVA.balance + ACHATS_A_LINTERIEUR_20_TVA.balance + ACHATS_A_LIMPORTATION_14_TVA.balance + ACHATS_A_LINTERIEUR_14_TVA.balance + ACHATS_A_LIMPORTATION_10_TVA.balance + ACHATS_A_LINTERIEUR_10_TVA.balance + ACHAT_A_LIMPORTATION_7_TVA.balance + ACHAT_A_LINTERIEUR_7_TVA.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_achats_importation_tva_20" model="account.report.line">
                                        <field name="name">Achats à l'importation (20%) (TVA)</field>
                                        <field name="code">ACHATS_A_LIMPORTATION_20_TVA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_importation_tva_20_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achats à l'importation (20%) (TVA)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_achats_interieur_tva_20" model="account.report.line">
                                        <field name="name">Achats à l'intérieur (20%) (TVA)</field>
                                        <field name="code">ACHATS_A_LINTERIEUR_20_TVA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_interieur_tva_20_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achats à l'intérieur (20%) (TVA)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_achats_importation_tva_14" model="account.report.line">
                                        <field name="name">Achats à l'importation (14%) (TVA)</field>
                                        <field name="code">ACHATS_A_LIMPORTATION_14_TVA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_importation_tva_14_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achats à l'importation (14%) (TVA)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_achats_interieur_tva_14" model="account.report.line">
                                        <field name="name">Achats à l'intérieur (14%) (TVA)</field>
                                        <field name="code">ACHATS_A_LINTERIEUR_14_TVA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_interieur_tva_14_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achats à l'intérieur (14%) (TVA)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_achats_importation_tva_10" model="account.report.line">
                                        <field name="name">Achats à l'importation (10%) (TVA)</field>
                                        <field name="code">ACHATS_A_LIMPORTATION_10_TVA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_importation_tva_10_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achats à l'importation (10%) (TVA)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_achats_interieur_tva_10" model="account.report.line">
                                        <field name="name">Achats à l'intérieur (10%) (TVA)</field>
                                        <field name="code">ACHATS_A_LINTERIEUR_10_TVA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_interieur_tva_10_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achats à l'intérieur (10%) (TVA)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_achats_importation_tva_7" model="account.report.line">
                                        <field name="name">Achat à l'importation (7%) (TVA)</field>
                                        <field name="code">ACHAT_A_LIMPORTATION_7_TVA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_importation_tva_7_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achat à l'importation (7%) (TVA)</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_achats_interieur_tva_7" model="account.report.line">
                                        <field name="name">Achat à l'intérieur (7%) (TVA)</field>
                                        <field name="code">ACHAT_A_LINTERIEUR_7_TVA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_achats_interieur_tva_7_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">Achat à l'intérieur (7%) (TVA)</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_prestations_service_deductible_tva" model="account.report.line">
                                <field name="name">Prestations de services (TVA déductible)</field>
                                <field name="code">PRESTATIONS_DE_SERVICES_TVA_DEDUCTIBLE</field>
                                <field name="expression_ids">
                                    <record id="tax_report_prestations_service_deductible_tva_formula" model="account.report.expression">
                                        <field name="label">balance</field>
                                        <field name="engine">external</field>
                                        <field name="formula">sum</field>
                                        <field name="subformula">editable;rounding=2</field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_immobilisations_base_ht" model="account.report.line">
                        <field name="name">2-IMMOBILISATIONS (Base HT)</field>
                        <field name="code">IMMOBILISATIONS_BASE_HT</field>
                        <field name="expression_ids">
                            <record id="tax_report_immobilisations_base_ht_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Immobilisations (Base HT)</field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_immobilisations_deductible_tva" model="account.report.line">
                        <field name="name">2-IMMOBILISATIONS (TVA déductible)</field>
                        <field name="code">IMMOBILISATIONS_TVA_DEDUCTIBLE</field>
                        <field name="expression_ids">
                            <record id="tax_report_immobilisations_deductible_tva_tag" model="account.report.expression">
                                <field name="label">balance</field>
                                <field name="engine">tax_tags</field>
                                <field name="formula">Immobilisations (TVA déductible)</field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_ma_chart_data.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
        <!--
        Plan comptable général pour le Maroc.
    Version du fichier : 13-10-2010
    Mis en place par la société Kazacube - partenaire OpenERP au Maroc
    Vérifié et validé par le cabinet SEDDIK d'expertise comptable, Casablanca.
      -->

    <record id="l10n_kzc_temp_chart" model="account.chart.template">
        <field name="name">Plan comptable marocain</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.MAD"/>
        <field name="bank_account_code_prefix">5141</field>
        <field name="cash_account_code_prefix">5161</field>
        <field name="transfer_account_code_prefix">5115</field>
        <field name="country_id" ref="base.ma"/>
    </record>

  <record id="pcg_1111" model="account.account.template">
    <field name="name">Capital social</field>
    <field name="code">1111</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1112" model="account.account.template">
    <field name="name">Fonds de dotation</field>
    <field name="code">1112</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_11171" model="account.account.template">
    <field name="name">Capital individuel</field>
    <field name="code">11171</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_11175" model="account.account.template">
    <field name="name">Compte de l'exploitant</field>
    <field name="code">11175</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1119" model="account.account.template">
    <field name="name">Actionnaires, capital souscrit-non appelé</field>
    <field name="code">1119</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1121" model="account.account.template">
    <field name="name">Primes d'émission</field>
    <field name="code">1121</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1122" model="account.account.template">
    <field name="name">Primes de fusion</field>
    <field name="code">1122</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="Pcg_1123" model="account.account.template">
    <field name="name">Primes d'apport</field>
    <field name="code">1123</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1130" model="account.account.template">
    <field name="name">Écarts de réévaluation</field>
    <field name="code">1130</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1140" model="account.account.template">
    <field name="name">Réserve légale</field>
    <field name="code">1140</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1151" model="account.account.template">
    <field name="name">Réserves statutaires ou contractuelles</field>
    <field name="code">1151</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1152" model="account.account.template">
    <field name="name">Réserves facultatives</field>
    <field name="code">1152</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1155" model="account.account.template">
    <field name="name">Réserves réglementées</field>
    <field name="code">1155</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1161" model="account.account.template">
    <field name="name">Report à  nouveau (solde créditeur)</field>
    <field name="code">1161</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1169" model="account.account.template">
    <field name="name">Report à  nouveau (solde débiteur)</field>
    <field name="code">1169</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1181" model="account.account.template">
    <field name="name">Résultats nets en instance d'affectation (solde créditeur)</field>
    <field name="code">1181</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1189" model="account.account.template">
    <field name="name">Résultats nets en instance d'affectation (solde débiteur)</field>
    <field name="code">1189</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1191" model="account.account.template">
    <field name="name">Résultat net de l'exercice (solde créditeur)</field>
    <field name="code">1191</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1199" model="account.account.template">
    <field name="name">Résultat net de l'exercice (solde débiteur)</field>
    <field name="code">1199</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1311" model="account.account.template">
    <field name="name">Subventions d'investissement reçues</field>
    <field name="code">1311</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1319" model="account.account.template">
    <field name="name">Subventions d'investissement inscrites au CPC</field>
    <field name="code">1319</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1351" model="account.account.template">
    <field name="name">Provisions pour amortissements dérogatoires</field>
    <field name="code">1351</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1352" model="account.account.template">
    <field name="name">Provisions pour plus-values en instance d'imposition</field>
    <field name="code">1352</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1354" model="account.account.template">
    <field name="name">Provisions pour investissements</field>
    <field name="code">1354</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1355" model="account.account.template">
    <field name="name">Provisions pour reconstitution des gisements</field>
    <field name="code">1355</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1356" model="account.account.template">
    <field name="name">Provisions pour acquisition et construction de logements</field>
    <field name="code">1356</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1358" model="account.account.template">
    <field name="name">Autres provisions réglementées</field>
    <field name="code">1358</field>
    <field name="account_type">equity</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1410" model="account.account.template">
    <field name="name">Emprunts obligataires</field>
    <field name="code">1410</field>
    <field name="account_type">liability_non_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1481" model="account.account.template">
    <field name="name">Emprunts auprès des établissements de crédit</field>
    <field name="code">1481</field>
    <field name="account_type">liability_non_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1482" model="account.account.template">
    <field name="name">Avances de l'Etat</field>
    <field name="code">1482</field>
    <field name="account_type">liability_non_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1483" model="account.account.template">
    <field name="name">Dettes rattachées à  des participations</field>
    <field name="code">1483</field>
    <field name="account_type">liability_non_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1484" model="account.account.template">
    <field name="name">Billets de fonds</field>
    <field name="code">1484</field>
    <field name="account_type">liability_non_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1485" model="account.account.template">
    <field name="name">Avances reçues et comptes courants bloqués</field>
    <field name="code">1485</field>
    <field name="account_type">liability_non_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1486" model="account.account.template">
    <field name="name">Fournisseurs d'immobilisation</field>
    <field name="code">1486</field>
    <field name="account_type">liability_non_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1487" model="account.account.template">
    <field name="name">Dépôts et cautionnements reçues</field>
    <field name="code">1487</field>
    <field name="account_type">liability_non_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1488" model="account.account.template">
    <field name="name">Dettes de financement diverses</field>
    <field name="code">1488</field>
    <field name="account_type">liability_non_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1511" model="account.account.template">
    <field name="name">Provisions pour litiges</field>
    <field name="code">1511</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1512" model="account.account.template">
    <field name="name">Provisions pour garanties données aux clients</field>
    <field name="code">1512</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1513" model="account.account.template">
    <field name="name">Provisions pour propre assureur</field>
    <field name="code">1513</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1514" model="account.account.template">
    <field name="name">Provision pour pertes sur marchés à  terme</field>
    <field name="code">1514</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1515" model="account.account.template">
    <field name="name">Provisions pour amendes, double droits, pénalités</field>
    <field name="code">1515</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1516" model="account.account.template">
    <field name="name">Provisions pour pertes de change</field>
    <field name="code">1516</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1518" model="account.account.template">
    <field name="name">Autres provisions pour risques</field>
    <field name="code">1518</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1551" model="account.account.template">
    <field name="name">Provisions pour impôts</field>
    <field name="code">1551</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1552" model="account.account.template">
    <field name="name">Provisions pour pensions de retraite et obligations similaires</field>
    <field name="code">1552</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1555" model="account.account.template">
    <field name="name">Provisions pour charges à  répartir sur plusieurs exercices</field>
    <field name="code">1555</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1558" model="account.account.template">
    <field name="name">Autres provisions pour charges</field>
    <field name="code">1558</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1601" model="account.account.template">
    <field name="name">Comptes de liaison du siège</field>
    <field name="code">1601</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1605" model="account.account.template">
    <field name="name">Comptes de liaison des établissements</field>
    <field name="code">1605</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1710" model="account.account.template">
    <field name="name">Augmentation des créances immobilisées</field>
    <field name="code">1710</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_1720" model="account.account.template">
    <field name="name">Diminution des dettes de financement</field>
    <field name="code">1720</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2111" model="account.account.template">
    <field name="name">Frais de constitution</field>
    <field name="code">2111</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2112" model="account.account.template">
    <field name="name">Frais préalables au démarrage</field>
    <field name="code">2112</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2113" model="account.account.template">
    <field name="name">Frais d'augmentation du capital</field>
    <field name="code">2113</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2114" model="account.account.template">
    <field name="name">Frais sur opérations de fusions, scissions et transformations</field>
    <field name="code">2114</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2116" model="account.account.template">
    <field name="name">Frais de prospection</field>
    <field name="code">2116</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2117" model="account.account.template">
    <field name="name">Frais de publicité</field>
    <field name="code">2117</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2118" model="account.account.template">
    <field name="name">Autres frais préliminaires</field>
    <field name="code">2118</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2121" model="account.account.template">
    <field name="name">Frais d acquisition des immobilisations</field>
    <field name="code">2121</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2125" model="account.account.template">
    <field name="name">Frais d'émission des emprunts</field>
    <field name="code">2125</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2128" model="account.account.template">
    <field name="name">Autres charges à  répartir</field>
    <field name="code">2128</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2130" model="account.account.template">
    <field name="name">Primes de remboursement des obligations</field>
    <field name="code">2130</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2210" model="account.account.template">
    <field name="name">Immobilisation en recherche et développement</field>
    <field name="code">2210</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2220" model="account.account.template">
    <field name="name">Brevets, marques, droits et valeurs similaires</field>
    <field name="code">2220</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2230" model="account.account.template">
    <field name="name">Fonds commercial</field>
    <field name="code">2230</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2285" model="account.account.template">
    <field name="name">Autres immobilisations incorporelles</field>
    <field name="code">2285</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2311" model="account.account.template">
    <field name="name">Terrains nus</field>
    <field name="code">2311</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2312" model="account.account.template">
    <field name="name">Terrains aménagés</field>
    <field name="code">2312</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2313" model="account.account.template">
    <field name="name">Terrains bâtis</field>
    <field name="code">2313</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2314" model="account.account.template">
    <field name="name">Terrains de gisement</field>
    <field name="code">2314</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2316" model="account.account.template">
    <field name="name">Agencements et aménagements de terrains</field>
    <field name="code">2316</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2318" model="account.account.template">
    <field name="name">Autres terrains</field>
    <field name="code">2318</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_23211" model="account.account.template">
    <field name="name">Bâtiments industriels (A,B,,,)</field>
    <field name="code">23211</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_23214" model="account.account.template">
    <field name="name">Bâtiments Administratifs et commerciaux</field>
    <field name="code">23214</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_23218" model="account.account.template">
    <field name="name">Autres bâtiments</field>
    <field name="code">23218</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2323" model="account.account.template">
    <field name="name">Constructions sur terrains d'autrui</field>
    <field name="code">2323</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2325" model="account.account.template">
    <field name="name">Ouvrages d'infrastructure</field>
    <field name="code">2325</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2327" model="account.account.template">
    <field name="name">Agencements et aménagements des constructions</field>
    <field name="code">2327</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2328" model="account.account.template">
    <field name="name">Autres constructions</field>
    <field name="code">2328</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2331" model="account.account.template">
    <field name="name">Installations techniques</field>
    <field name="code">2331</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_23321" model="account.account.template">
    <field name="name">Matériel</field>
    <field name="code">23321</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_23324" model="account.account.template">
    <field name="name">Outillage</field>
    <field name="code">23324</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2333" model="account.account.template">
    <field name="name">Emballages récupérables identifiables</field>
    <field name="code">2333</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2338" model="account.account.template">
    <field name="name">Autres installations techniques, matériel et outillage</field>
    <field name="code">2338</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2340" model="account.account.template">
    <field name="name">Matériel de transport</field>
    <field name="code">2340</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2351" model="account.account.template">
    <field name="name">Mobilier de bureau</field>
    <field name="code">2351</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2352" model="account.account.template">
    <field name="name">Matériel de bureau</field>
    <field name="code">2352</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2355" model="account.account.template">
    <field name="name">Matériel informatique</field>
    <field name="code">2355</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2356" model="account.account.template">
    <field name="name">Agencements, installations et aménagements divers (biens n'appartenant pas à  l'entreprise)</field>
    <field name="code">2356</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2358" model="account.account.template">
    <field name="name">Autres mobilier, matériel de bureau et aménagements divers</field>
    <field name="code">2358</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2380" model="account.account.template">
    <field name="name">Autres immobilisations corporelles</field>
    <field name="code">2380</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2392" model="account.account.template">
    <field name="name">Immobilisations corporelles en cours des terrains et constructions</field>
    <field name="code">2392</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2393" model="account.account.template">
    <field name="name">Immobilisations corporelles en cours des installations techniques, matériel et outillage</field>
    <field name="code">2393</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2394" model="account.account.template">
    <field name="name">Immobilisations corporelles en cours de matériel de transport</field>
    <field name="code">2394</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2395" model="account.account.template">
    <field name="name">Immobilisations corporelles en cours de mobilier, matériel de bureau et aménagements divers</field>
    <field name="code">2395</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2397" model="account.account.template">
    <field name="name">Avances et acomptes versés sur commandes d'immobilisations corporelles</field>
    <field name="code">2397</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2398" model="account.account.template">
    <field name="name">Autres immobilisations corporelles en cours</field>
    <field name="code">2398</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2411" model="account.account.template">
    <field name="name">Prêts au personnel</field>
    <field name="code">2411</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2415" model="account.account.template">
    <field name="name">Prêts aux associés</field>
    <field name="code">2415</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2416" model="account.account.template">
    <field name="name">Billets de fonds</field>
    <field name="code">2416</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2418" model="account.account.template">
    <field name="name">Autres prêts</field>
    <field name="code">2418</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_24811" model="account.account.template">
    <field name="name">Obligations</field>
    <field name="code">24811</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_24813" model="account.account.template">
    <field name="name">Bons d'équipement</field>
    <field name="code">24813</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_24818" model="account.account.template">
    <field name="name">Bons divers</field>
    <field name="code">24818</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2483" model="account.account.template">
    <field name="name">Créances rattachées à  des participations</field>
    <field name="code">2483</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_24861" model="account.account.template">
    <field name="name">Dépôts</field>
    <field name="code">24861</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_24864" model="account.account.template">
    <field name="name">Cautionnements</field>
    <field name="code">24864</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2487" model="account.account.template">
    <field name="name">Créances immobilisées</field>
    <field name="code">2487</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2488" model="account.account.template">
    <field name="name">Créances financières diverses</field>
    <field name="code">2488</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2510" model="account.account.template">
    <field name="name">Titres de participation</field>
    <field name="code">2510</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2581" model="account.account.template">
    <field name="name">Actions</field>
    <field name="code">2581</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2588" model="account.account.template">
    <field name="name">Titres divers</field>
    <field name="code">2588</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2710" model="account.account.template">
    <field name="name">Diminution des créances immobilisées</field>
    <field name="code">2710</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2720" model="account.account.template">
    <field name="name">Augmentation des dettes de financement</field>
    <field name="code">2720</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28111" model="account.account.template">
    <field name="name">Amortissements des frais de constitution</field>
    <field name="code">28111</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28112" model="account.account.template">
    <field name="name">Amortissements des frais préliminaires au démarrage</field>
    <field name="code">28112</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28113" model="account.account.template">
    <field name="name">Amortissements des frais d'augmentation du capital</field>
    <field name="code">28113</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28114" model="account.account.template">
    <field name="name">Amortissements des frais sur opérations de fusions, scissions, et transformations</field>
    <field name="code">28114</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28116" model="account.account.template">
    <field name="name">Amortissements des frais de prospection</field>
    <field name="code">28116</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28117" model="account.account.template">
    <field name="name">Amortissements des frais de publicité</field>
    <field name="code">28117</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28118" model="account.account.template">
    <field name="name">Amortissements des autres frais préliminaires</field>
    <field name="code">28118</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28121" model="account.account.template">
    <field name="name">Amortissements des frais d'acquisition des immobilisations</field>
    <field name="code">28121</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28125" model="account.account.template">
    <field name="name">Amortissements des frais d'émission des emprunts</field>
    <field name="code">28125</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28128" model="account.account.template">
    <field name="name">Amortissements des autres charges à  répartir</field>
    <field name="code">28128</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2821" model="account.account.template">
    <field name="name">Amortissements de l'immobilisation en recherche et développement</field>
    <field name="code">2821</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2822" model="account.account.template">
    <field name="name">Amortissements des brevets, marques, droits et valeurs similaires</field>
    <field name="code">2822</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2823" model="account.account.template">
    <field name="name">Amortissements du fonds commercial</field>
    <field name="code">2823</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2828" model="account.account.template">
    <field name="name">Amortissements des autres immobilisations incorporelles</field>
    <field name="code">2828</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28311" model="account.account.template">
    <field name="name">Amortissements des terrains nus</field>
    <field name="code">28311</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28312" model="account.account.template">
    <field name="name">Amortissements des terrains aménagés</field>
    <field name="code">28312</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28313" model="account.account.template">
    <field name="name">Amortissements des terrains bâtis</field>
    <field name="code">28313</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28314" model="account.account.template">
    <field name="name">Amortissements des terrains de gisement</field>
    <field name="code">28314</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28316" model="account.account.template">
    <field name="name">Amortissements des agencements et aménagements de terrains</field>
    <field name="code">28316</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28318" model="account.account.template">
    <field name="name">Amortissements des autres terrains</field>
    <field name="code">28318</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28321" model="account.account.template">
    <field name="name">Amortissements des bâtiments</field>
    <field name="code">28321</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28323" model="account.account.template">
    <field name="name">Amortissements des constructions sur terrains d'autrui</field>
    <field name="code">28323</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28325" model="account.account.template">
    <field name="name">Amortissements des ouvrages d'infrastructure</field>
    <field name="code">28325</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28327" model="account.account.template">
    <field name="name">Amortissements des installations, agencements et aménagements des constructions</field>
    <field name="code">28327</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28328" model="account.account.template">
    <field name="name">Amortissements des autres constructions</field>
    <field name="code">28328</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28331" model="account.account.template">
    <field name="name">Amortissements des installations techniques</field>
    <field name="code">28331</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28332" model="account.account.template">
    <field name="name">Amortissements du matériel et outillage</field>
    <field name="code">28332</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28333" model="account.account.template">
    <field name="name">Amortissements des emballages récupérables identifiables</field>
    <field name="code">28333</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28338" model="account.account.template">
    <field name="name">Amortissements des autres installations techniques, matériel et outillage</field>
    <field name="code">28338</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2834" model="account.account.template">
    <field name="name">Amortissements du matériel de transport</field>
    <field name="code">2834</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28351" model="account.account.template">
    <field name="name">Amortissements du mobilier de bureau</field>
    <field name="code">28351</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28352" model="account.account.template">
    <field name="name">Amortissements du matériel de bureau</field>
    <field name="code">28352</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28355" model="account.account.template">
    <field name="name">Amortissements du matériel informatique</field>
    <field name="code">28355</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28356" model="account.account.template">
    <field name="name">Amortissements des agencements, installations et aménagements divers</field>
    <field name="code">28356</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_28358" model="account.account.template">
    <field name="name">Amortissements des autres mobilier, matériel de bureau et aménagements divers</field>
    <field name="code">28358</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2838" model="account.account.template">
    <field name="name">Amortissements des autres immobilisations corporelles</field>
    <field name="code">2838</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2920" model="account.account.template">
    <field name="name">Provisions pour dépréciation des immobilisations incorporelles</field>
    <field name="code">2920</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2930" model="account.account.template">
    <field name="name">Provisions pour dépréciation des immobilisations corporelles</field>
    <field name="code">2930</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2941" model="account.account.template">
    <field name="name">Provisions pour dépréciation des prêts immobilisés</field>
    <field name="code">2941</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2948" model="account.account.template">
    <field name="name">Provisions pour dépréciation des autres créances financières</field>
    <field name="code">2948</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2951" model="account.account.template">
    <field name="name">Provisions pour dépréciation des titres de participation</field>
    <field name="code">2951</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_2958" model="account.account.template">
    <field name="name">Provisions pour dépréciation des autres titres immobilisés</field>
    <field name="code">2958</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3111" model="account.account.template">
    <field name="name">Marchandises (groupe A)</field>
    <field name="code">3111</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3112" model="account.account.template">
    <field name="name">Marchandises (groupe B)</field>
    <field name="code">3112</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3116" model="account.account.template">
    <field name="name">Marchandises en cours de route</field>
    <field name="code">3116</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3118" model="account.account.template">
    <field name="name">Autres marchandises</field>
    <field name="code">3118</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31211" model="account.account.template">
    <field name="name">Matières premières (groupe A)</field>
    <field name="code">31211</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31212" model="account.account.template">
    <field name="name">Matières premières (groupe B)</field>
    <field name="code">31212</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31221" model="account.account.template">
    <field name="name">Matières consommables (groupe A)</field>
    <field name="code">31221</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31222" model="account.account.template">
    <field name="name">Matières consommables (groupe B)</field>
    <field name="code">31222</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31223" model="account.account.template">
    <field name="name">Combustibles</field>
    <field name="code">31223</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31224" model="account.account.template">
    <field name="name">Produits d'entretien</field>
    <field name="code">31224</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31225" model="account.account.template">
    <field name="name">Fournitures d'atelier et d'usine</field>
    <field name="code">31225</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31226" model="account.account.template">
    <field name="name">Fournitures de magasin</field>
    <field name="code">31226</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31227" model="account.account.template">
    <field name="name">Fournitures de bureau</field>
    <field name="code">31227</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31231" model="account.account.template">
    <field name="name">Emballages perdus</field>
    <field name="code">31231</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31232" model="account.account.template">
    <field name="name">Emballages récupérables non identifiables</field>
    <field name="code">31232</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31233" model="account.account.template">
    <field name="name">Emballages à  usage mixte</field>
    <field name="code">31233</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3126" model="account.account.template">
    <field name="name">Matières et fournitures consommables en cours de route</field>
    <field name="code">3126</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3128" model="account.account.template">
    <field name="name">Autres matières et fournitures consommables</field>
    <field name="code">3128</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31311" model="account.account.template">
    <field name="name">Biens produits en cours</field>
    <field name="code">31311</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31312" model="account.account.template">
    <field name="name">Biens intermédiaires en cours</field>
    <field name="code">31312</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31317" model="account.account.template">
    <field name="name">Biens résiduels en cours</field>
    <field name="code">31317</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31341" model="account.account.template">
    <field name="name">Travaux en cours</field>
    <field name="code">31341</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31342" model="account.account.template">
    <field name="name">Études en cours</field>
    <field name="code">31342</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31343" model="account.account.template">
    <field name="name">Prestations en cours</field>
    <field name="code">31343</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3138" model="account.account.template">
    <field name="name">Autres produits en cours</field>
    <field name="code">3138</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31411" model="account.account.template">
    <field name="name">Produits intermédiaires (groupe A)</field>
    <field name="code">31411</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31412" model="account.account.template">
    <field name="name">Produits intermédiaires (groupe B)</field>
    <field name="code">31412</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31451" model="account.account.template">
    <field name="name">Déchets</field>
    <field name="code">31451</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31452" model="account.account.template">
    <field name="name">Rebuts</field>
    <field name="code">31452</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_31453" model="account.account.template">
    <field name="name">Matières de récupération</field>
    <field name="code">31453</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3148" model="account.account.template">
    <field name="name">Autres produits intermédiaires et produits résiduels</field>
    <field name="code">3148</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3151" model="account.account.template">
    <field name="name">Produits finis (groupe A)</field>
    <field name="code">3151</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3152" model="account.account.template">
    <field name="name">Produits finis (groupe B)</field>
    <field name="code">3152</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3156" model="account.account.template">
    <field name="name">Produits finis en cours de route</field>
    <field name="code">3156</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3158" model="account.account.template">
    <field name="name">Autres produits finis</field>
    <field name="code">3158</field>
    <field name="account_type">income</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3411" model="account.account.template">
    <field name="name">Fournisseurs - avances et acomptes versés sur commandes d'exploitation</field>
    <field name="code">3411</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3413" model="account.account.template">
    <field name="name">Fournisseurs - créances pour emballages et matériel à  rendre</field>
    <field name="code">3413</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3417" model="account.account.template">
    <field name="name">Rabais, remises et ristournes à  obtenir - avoirs non encore reçus</field>
    <field name="code">3417</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3418" model="account.account.template">
    <field name="name">Autres fournisseurs débiteurs</field>
    <field name="code">3418</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_34211" model="account.account.template">
    <field name="name">Clients - catégorie A</field>
    <field name="code">34211</field>
    <field name="account_type">asset_receivable</field>
    <field name="reconcile" eval='True'/>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_34212" model="account.account.template">
    <field name="name">Clients - catégorie B</field>
    <field name="code">34212</field>
    <field name="account_type">asset_receivable</field>
    <field name="reconcile" eval='True'/>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3423" model="account.account.template">
    <field name="name">Clients - retenues de garantie</field>
    <field name="code">3423</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3424" model="account.account.template">
    <field name="name">Clients douteux ou litigieux</field>
    <field name="code">3424</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3425" model="account.account.template">
    <field name="name">Clients - effets à  recevoir</field>
    <field name="code">3425</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_34271" model="account.account.template">
    <field name="name">Clients - factures à  établir</field>
    <field name="code">34271</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_34272" model="account.account.template">
    <field name="name">Créances sur travaux non encore facturés</field>
    <field name="code">34272</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3428" model="account.account.template">
    <field name="name">Autres clients et comptes rattachés</field>
    <field name="code">3428</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3431" model="account.account.template">
    <field name="name">Avances et acomptes au personnel</field>
    <field name="code">3431</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3438" model="account.account.template">
    <field name="name">Personnel - autres débiteurs</field>
    <field name="code">3438</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_34511" model="account.account.template">
    <field name="name">Subventions d'investissement à  recevoir</field>
    <field name="code">34511</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_34512" model="account.account.template">
    <field name="name">Subventions d'exploitation à  recevoir</field>
    <field name="code">34512</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_34513" model="account.account.template">
    <field name="name">Subventions d'équilibre à  recevoir</field>
    <field name="code">34513</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3453" model="account.account.template">
    <field name="name">Acomptes sur impôts sur les résultats</field>
    <field name="code">3453</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_34551" model="account.account.template">
    <field name="name">Etat - TVA récupérable sur immobilisations</field>
    <field name="code">34551</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3455207" model="account.account.template">
    <field name="name">Etat - TVA récupérable sur charges 7%</field>
    <field name="code">3455207</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3455210" model="account.account.template">
    <field name="name">Etat - TVA récupérable sur charges 10%</field>
    <field name="code">3455210</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3455214" model="account.account.template">
    <field name="name">Etat - TVA récupérable sur charges 14%</field>
    <field name="code">3455214</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3455220" model="account.account.template">
    <field name="name">Etat - TVA récupérable sur charges 20%</field>
    <field name="code">3455220</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3456" model="account.account.template">
    <field name="name">Etat - crédit de TVA (suivant déclaration)</field>
    <field name="code">3456</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3458" model="account.account.template">
    <field name="name">Etat - autres comptes débiteurs</field>
    <field name="code">3458</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3461" model="account.account.template">
    <field name="name">Associés - comptes d'apport en société</field>
    <field name="code">3461</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3462" model="account.account.template">
    <field name="name">Actionnaires - capital souscrit et appelé non versé</field>
    <field name="code">3462</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3463" model="account.account.template">
    <field name="name">Comptes courants des associés débiteurs</field>
    <field name="code">3463</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3464" model="account.account.template">
    <field name="name">Associés - opérations faites en commun</field>
    <field name="code">3464</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3467" model="account.account.template">
    <field name="name">Créances rattachées aux comptes d'associés</field>
    <field name="code">3467</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3468" model="account.account.template">
    <field name="name">Autres comptes d'associés débiteurs</field>
    <field name="code">3468</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3481" model="account.account.template">
    <field name="name">Créances sur cessions d'immobilisations</field>
    <field name="code">3481</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3482" model="account.account.template">
    <field name="name">Créances sur cessions d'éléments d'actif circulant</field>
    <field name="code">3482</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3487" model="account.account.template">
    <field name="name">Créances rattachées aux autres débiteurs</field>
    <field name="code">3487</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3488" model="account.account.template">
    <field name="name">Divers débiteurs</field>
    <field name="code">3488</field>
    <field name="account_type">asset_current</field>
    <field name="reconcile" eval='True'/>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3489" model="account.account.template">
    <field name="name">Divers débiteurs (PoS)</field>
    <field name="code">3489</field>
    <field name="account_type">asset_receivable</field>
    <field name="reconcile" eval='True'/>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3491" model="account.account.template">
    <field name="name">Charges constatées d'avance</field>
    <field name="code">3491</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3493" model="account.account.template">
    <field name="name">Intérêts courus et non échus à  percevoir</field>
    <field name="code">3493</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3495" model="account.account.template">
    <field name="name">Comptes de répartition périodique des charges</field>
    <field name="code">3495</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3497" model="account.account.template">
    <field name="name">Comptes transitoires ou d'attente - débiteurs</field>
    <field name="code">3497</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3501" model="account.account.template">
    <field name="name">Actions, partie libérée</field>
    <field name="code">3501</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3502" model="account.account.template">
    <field name="name">Actions, partie non libérée</field>
    <field name="code">3502</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3504" model="account.account.template">
    <field name="name">Obligations</field>
    <field name="code">3504</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_35061" model="account.account.template">
    <field name="name">Bons de caisse</field>
    <field name="code">35061</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_35062" model="account.account.template">
    <field name="name">Bons de trésor</field>
    <field name="code">35062</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3508" model="account.account.template">
    <field name="name">Autres titres et valeurs de placement similaires</field>
    <field name="code">3508</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3701" model="account.account.template">
    <field name="name">Diminution des créances circulantes</field>
    <field name="code">3701</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3702" model="account.account.template">
    <field name="name">Augmentation des dettes circulantes</field>
    <field name="code">3702</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3911" model="account.account.template">
    <field name="name">Provisions pour dépréciation des marchandises</field>
    <field name="code">3911</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3912" model="account.account.template">
    <field name="name">Provisions pour dépréciation des matières et fournitures</field>
    <field name="code">3912</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3913" model="account.account.template">
    <field name="name">Provisions pour dépréciation des produits en cours</field>
    <field name="code">3913</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3914" model="account.account.template">
    <field name="name">Provisions pour dépréciation des produits intermédiaires</field>
    <field name="code">3914</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3915" model="account.account.template">
    <field name="name">Provisions pour dépréciation des produits finis</field>
    <field name="code">3915</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3941" model="account.account.template">
    <field name="name">Provisions pour dépréciation - fournisseurs débiteurs, avances et acomptes</field>
    <field name="code">3941</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3942" model="account.account.template">
    <field name="name">Provisions pour dépréciation des clients et comptes rattachés</field>
    <field name="code">3942</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3943" model="account.account.template">
    <field name="name">Provisions pour dépréciation du personnel - débiteur</field>
    <field name="code">3943</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3946" model="account.account.template">
    <field name="name">Provisions pour dépréciation des comptes d'associés débiteurs</field>
    <field name="code">3946</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3948" model="account.account.template">
    <field name="name">Provisions pour dépréciation des autres débiteurs</field>
    <field name="code">3948</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_3950" model="account.account.template">
    <field name="name">Provisions pour dépréciation des titres et valeurs de placement</field>
    <field name="code">3950</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4411" model="account.account.template">
    <field name="name">Fournisseurs</field>
    <field name="code">4411</field>
    <field name="account_type">liability_payable</field>
    <field name="reconcile" eval='True'/>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4413" model="account.account.template">
    <field name="name">Fournisseurs - retenues de garantie</field>
    <field name="code">4413</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4415" model="account.account.template">
    <field name="name">Fournisseurs - effets à  payer</field>
    <field name="code">4415</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4417" model="account.account.template">
    <field name="name">Fournisseurs - factures non parvenues</field>
    <field name="code">4417</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4418" model="account.account.template">
    <field name="name">Autres fournisseurs et comptes rattachés</field>
    <field name="code">4418</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4421" model="account.account.template">
    <field name="name">Clients - avances et acomptes reçus sur commandes en cours</field>
    <field name="code">4421</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4425" model="account.account.template">
    <field name="name">Clients - dettes pour emballages et matériel consignés</field>
    <field name="code">4425</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4427" model="account.account.template">
    <field name="name">Rabais, remises et ristournes à  accorder - avoirs à  établir</field>
    <field name="code">4427</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4428" model="account.account.template">
    <field name="name">Autres clients créditeurs</field>
    <field name="code">4428</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4432" model="account.account.template">
    <field name="name">Rémunérations dues au personnel</field>
    <field name="code">4432</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4433" model="account.account.template">
    <field name="name">Dépôts du personnel créditeurs</field>
    <field name="code">4433</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4434" model="account.account.template">
    <field name="name">Oppositions sur salaires</field>
    <field name="code">4434</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4437" model="account.account.template">
    <field name="name">Charges du personnel à  payer</field>
    <field name="code">4437</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4438" model="account.account.template">
    <field name="name">Personnel - autres créditeurs</field>
    <field name="code">4438</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4442" model="account.account.template">
    <field name="name">ALLOCATION FAMILIALES</field>
    <field name="code">4442</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4443" model="account.account.template">
    <field name="name">Caisses de retraite</field>
    <field name="code">4443</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4445" model="account.account.template">
    <field name="name">Mutuelles</field>
    <field name="code">4445</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4447" model="account.account.template">
    <field name="name">Charges sociales à  payer</field>
    <field name="code">4447</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4448" model="account.account.template">
    <field name="name">Autres organismes sociaux</field>
    <field name="code">4448</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_44521" model="account.account.template">
    <field name="name">Etat, taxe urbaine et taxe d'édilité</field>
    <field name="code">44521</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_44522" model="account.account.template">
    <field name="name">Etat, Taxe professionnelle (ex patente)</field>
    <field name="code">44522</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_44525" model="account.account.template">
    <field name="name">Etat, IR</field>
    <field name="code">44525</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4453" model="account.account.template">
    <field name="name">Etat, impôts sur les résultats</field>
    <field name="code">4453</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_445507" model="account.account.template">
    <field name="name">Etat, TVA facturée 7%</field>
    <field name="code">445507</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_445510" model="account.account.template">
    <field name="name">Etat, TVA facturée 10%</field>
    <field name="code">445510</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_445514" model="account.account.template">
    <field name="name">Etat, TVA facturée 14%</field>
    <field name="code">445514</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_445520" model="account.account.template">
    <field name="name">Etat, TVA facturée 20%</field>
    <field name="code">445520</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4456" model="account.account.template">
    <field name="name">Etat, TVA due (suivant déclarations)</field>
    <field name="code">4456</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4457" model="account.account.template">
    <field name="name">Etat, impôts et taxes à  payer</field>
    <field name="code">4457</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4458" model="account.account.template">
    <field name="name">Etat - autres comptes créditeurs</field>
    <field name="code">4458</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4461" model="account.account.template">
    <field name="name">Associés - capital à  rembourser</field>
    <field name="code">4461</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4462" model="account.account.template">
    <field name="name">Associés - versements reçus sur augmentation de capital</field>
    <field name="code">4462</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4463" model="account.account.template">
    <field name="name">Comptes courants des associés créditeurs</field>
    <field name="code">4463</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4464" model="account.account.template">
    <field name="name">Associés - opérations faites en commun</field>
    <field name="code">4464</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4465" model="account.account.template">
    <field name="name">Associés - dividendes à  payer</field>
    <field name="code">4465</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4468" model="account.account.template">
    <field name="name">Autres comptes d'associés - créditeurs</field>
    <field name="code">4468</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4481" model="account.account.template">
    <field name="name">Dettes sur acquisitions d'immobilisations</field>
    <field name="code">4481</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4483" model="account.account.template">
    <field name="name">Dettes sur acquisitions de titres et valeurs de placement</field>
    <field name="code">4483</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4484" model="account.account.template">
    <field name="name">Obligations échues à  rembourser</field>
    <field name="code">4484</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4485" model="account.account.template">
    <field name="name">Obligations, coupons à  payer</field>
    <field name="code">4485</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4487" model="account.account.template">
    <field name="name">Dettes rattachées aux autres créanciers</field>
    <field name="code">4487</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4488" model="account.account.template">
    <field name="name">Divers créanciers</field>
    <field name="code">4488</field>
    <field name="account_type">liability_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
    <field name="reconcile" eval='True'/>
  </record>
  <record id="pcg_4491" model="account.account.template">
    <field name="name">Produits constatés d'avance</field>
    <field name="code">4491</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4493" model="account.account.template">
    <field name="name">Intérêts courus et non échus à payer</field>
    <field name="code">4493</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4495" model="account.account.template">
    <field name="name">Comptes de répartition périodique des produits</field>
    <field name="code">4495</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4497" model="account.account.template">
    <field name="name">Comptes transitoires ou d'attente - créditeurs</field>
    <field name="code">4497</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4501" model="account.account.template">
    <field name="name">Provisions pour litiges</field>
    <field name="code">4501</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4502" model="account.account.template">
    <field name="name">Provisions pour garanties données aux clients</field>
    <field name="code">4502</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4505" model="account.account.template">
    <field name="name">Provisions pour amendes, doubles droits et pénalités</field>
    <field name="code">4505</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4506" model="account.account.template">
    <field name="name">Provisions pour pertes de change</field>
    <field name="code">4506</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4507" model="account.account.template">
    <field name="name">Provisions pour impôts</field>
    <field name="code">4507</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4508" model="account.account.template">
    <field name="name">Autres provisions pour risques et charges</field>
    <field name="code">4508</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4701" model="account.account.template">
    <field name="name">Augmentation des créances circulantes</field>
    <field name="code">4701</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_4702" model="account.account.template">
    <field name="name">Diminution des dettes circulantes</field>
    <field name="code">4702</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_51111" model="account.account.template">
    <field name="name">Chèques en portefeuille</field>
    <field name="code">51111</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_51112" model="account.account.template">
    <field name="name">Chèques à  l'encaissement</field>
    <field name="code">51112</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_51131" model="account.account.template">
    <field name="name">Effets échus à  encaisser</field>
    <field name="code">51131</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_51132" model="account.account.template">
    <field name="name">Effets à  l'encaissement</field>
    <field name="code">51132</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_5118" model="account.account.template">
    <field name="name">Autres valeurs à  encaisser</field>
    <field name="code">5118</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_5146" model="account.account.template">
    <field name="name">Chèques postaux</field>
    <field name="code">5146</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_5148" model="account.account.template">
    <field name="name">Autres établissements financiers et assimilés (soldes débiteurs)</field>
    <field name="code">5148</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_5165" model="account.account.template">
    <field name="name">Régies d'avances et accréditifs</field>
    <field name="code">5165</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_5520" model="account.account.template">
    <field name="name">Crédits d'escompte</field>
    <field name="code">5520</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_5530" model="account.account.template">
    <field name="name">Crédits de trésorerie</field>
    <field name="code">5530</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_5548" model="account.account.template">
    <field name="name">Autres établissements financiers et assimilés (soldes créditeurs)</field>
    <field name="code">5548</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_5900" model="account.account.template">
    <field name="name">Provisions pour dépréciation des comptes de trésorerie</field>
    <field name="code">5900</field>
    <field name="account_type">asset_current</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6111" model="account.account.template">
    <field name="name">Achats de marchandises "groupe A"</field>
    <field name="code">6111</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6112" model="account.account.template">
    <field name="name">Achats de marchandises "groupe B"</field>
    <field name="code">6112</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6114" model="account.account.template">
    <field name="name">Variation de stocks de marchandises</field>
    <field name="code">6114</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6118" model="account.account.template">
    <field name="name">Achats revendus de marchandises des exercices antérieurs</field>
    <field name="code">6118</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6119" model="account.account.template">
    <field name="name">Rabais, remises et ristournes obtenus sur achats de marchandises</field>
    <field name="code">6119</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61211" model="account.account.template">
    <field name="name">Achats de matières premières A</field>
    <field name="code">61211</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61212" model="account.account.template">
    <field name="name">Achats de matières premières B</field>
    <field name="code">61212</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61221" model="account.account.template">
    <field name="name">Achats de matières et fournitures A</field>
    <field name="code">61221</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61222" model="account.account.template">
    <field name="name">Achats de matières et fournitures B</field>
    <field name="code">61222</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61223" model="account.account.template">
    <field name="name">Achats de combustibles</field>
    <field name="code">61223</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61224" model="account.account.template">
    <field name="name">Achats de produits d'entretien</field>
    <field name="code">61224</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61225" model="account.account.template">
    <field name="name">Achats de fournitures d'atelier et d'usine</field>
    <field name="code">61225</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61226" model="account.account.template">
    <field name="name">Achats de fournitures de magasin</field>
    <field name="code">61226</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61227" model="account.account.template">
    <field name="name">Achats de fournitures de bureau</field>
    <field name="code">61227</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61231" model="account.account.template">
    <field name="name">Achats d'emballages perdus</field>
    <field name="code">61231</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61232" model="account.account.template">
    <field name="name">Achats d'emballages récupérables non identifiables</field>
    <field name="code">61232</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61233" model="account.account.template">
    <field name="name">Achats d'emballages à  usage mixte</field>
    <field name="code">61233</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61241" model="account.account.template">
    <field name="name">Variation des stocks de matières premières</field>
    <field name="code">61241</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61242" model="account.account.template">
    <field name="name">Variation des stocks de matières et fournitures consommables</field>
    <field name="code">61242</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61243" model="account.account.template">
    <field name="name">Variation des stocks des emballages</field>
    <field name="code">61243</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61251" model="account.account.template">
    <field name="name">Achats de fournitures non stockables (eau, électricité,,)</field>
    <field name="code">61251</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61252" model="account.account.template">
    <field name="name">Achats de fournitures d'entretien</field>
    <field name="code">61252</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61253" model="account.account.template">
    <field name="name">Achats de petit outillage et petit équipement</field>
    <field name="code">61253</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61254" model="account.account.template">
    <field name="name">Achats de fournitures de bureau</field>
    <field name="code">61254</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61261" model="account.account.template">
    <field name="name">Achats des travaux</field>
    <field name="code">61261</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61262" model="account.account.template">
    <field name="name">Achats des études</field>
    <field name="code">61262</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61263" model="account.account.template">
    <field name="name">Achats des prestations de service</field>
    <field name="code">61263</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6128" model="account.account.template">
    <field name="name">Achats de matières et de fournitures des exercices antérieurs</field>
    <field name="code">6128</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61291" model="account.account.template">
    <field name="name">Rabais, remises et ristournes obtenus sur achats de matières premières</field>
    <field name="code">61291</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61292" model="account.account.template">
    <field name="name">Rabais, remises et ristournes obtenus sur achats de matières et fournitures consommables</field>
    <field name="code">61292</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61293" model="account.account.template">
    <field name="name">Rabais, remises et ristournes obtenus sur achats des emballages</field>
    <field name="code">61293</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61295" model="account.account.template">
    <field name="name">Rabais, remises et ristournes obtenus sur achats non stockés</field>
    <field name="code">61295</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61296" model="account.account.template">
    <field name="name">Rabais, remises et ristournes obtenus sur achats de travaux, études et prestations de service</field>
    <field name="code">61296</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61298" model="account.account.template">
    <field name="name">Rabais, remises et ristournes obtenus sur achats de matières et fournitures des exercices antérieurs</field>
    <field name="code">61298</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61311" model="account.account.template">
    <field name="name">Locations de terrains</field>
    <field name="code">61311</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61312" model="account.account.template">
    <field name="name">Locations de constructions</field>
    <field name="code">61312</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61313" model="account.account.template">
    <field name="name">Locations de matériel et d'outillage</field>
    <field name="code">61313</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61314" model="account.account.template">
    <field name="name">Locations de mobilier et de matériel de bureau</field>
    <field name="code">61314</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61315" model="account.account.template">
    <field name="name">Locations de matériel informatique</field>
    <field name="code">61315</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61316" model="account.account.template">
    <field name="name">Locations de matériel de transport</field>
    <field name="code">61316</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61317" model="account.account.template">
    <field name="name">Malis sur emballages rendus</field>
    <field name="code">61317</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61318" model="account.account.template">
    <field name="name">Locations et charges locatives diverses</field>
    <field name="code">61318</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61321" model="account.account.template">
    <field name="name">Redevances de crédit-bail - mobilier et matériel</field>
    <field name="code">61321</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61331" model="account.account.template">
    <field name="name">Entretien et réparations des biens immobiliers</field>
    <field name="code">61331</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61332" model="account.account.template">
    <field name="name">Entretien et réparations des biens mobiliers</field>
    <field name="code">61332</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61335" model="account.account.template">
    <field name="name">Maintenance</field>
    <field name="code">61335</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61341" model="account.account.template">
    <field name="name">Assurances multirisque (vol, incendie,R,C,)</field>
    <field name="code">61341</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61343" model="account.account.template">
    <field name="name">Assurances - risques d'exploitation</field>
    <field name="code">61343</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61345" model="account.account.template">
    <field name="name">Assurances - Matériel de transport</field>
    <field name="code">61345</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61351" model="account.account.template">
    <field name="name">Rémunérations du personnel occasionnel</field>
    <field name="code">61351</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61352" model="account.account.template">
    <field name="name">Rémunérations du personnel intérimaire</field>
    <field name="code">61352</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61353" model="account.account.template">
    <field name="name">Rémunérations du personnel détaché ou prêté à  l'entreprise</field>
    <field name="code">61353</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61361" model="account.account.template">
    <field name="name">Commissions et courtages</field>
    <field name="code">61361</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61365" model="account.account.template">
    <field name="name">Honoraires</field>
    <field name="code">61365</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61367" model="account.account.template">
    <field name="name">Frais d'actes et de contentieux</field>
    <field name="code">61367</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61371" model="account.account.template">
    <field name="name">Redevances pour brevets</field>
    <field name="code">61371</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61378" model="account.account.template">
    <field name="name">Autres redevances</field>
    <field name="code">61378</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61411" model="account.account.template">
    <field name="name">Études générales</field>
    <field name="code">61411</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61413" model="account.account.template">
    <field name="name">Recherches</field>
    <field name="code">61413</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61415" model="account.account.template">
    <field name="name">Documentation générale</field>
    <field name="code">61415</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61416" model="account.account.template">
    <field name="name">Documentation technique</field>
    <field name="code">61416</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61421" model="account.account.template">
    <field name="name">Transports du personnel</field>
    <field name="code">61421</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61425" model="account.account.template">
    <field name="name">Transports sur achats</field>
    <field name="code">61425</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61426" model="account.account.template">
    <field name="name">Transports sur ventes</field>
    <field name="code">61426</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61428" model="account.account.template">
    <field name="name">Autres transports</field>
    <field name="code">61428</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61431" model="account.account.template">
    <field name="name">Voyages et déplacements</field>
    <field name="code">61431</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61433" model="account.account.template">
    <field name="name">Frais de déménagement</field>
    <field name="code">61433</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61435" model="account.account.template">
    <field name="name">Missions</field>
    <field name="code">61435</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61436" model="account.account.template">
    <field name="name">Réceptions</field>
    <field name="code">61436</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61441" model="account.account.template">
    <field name="name">Annonces et insertions</field>
    <field name="code">61441</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61442" model="account.account.template">
    <field name="name">Échantillons, catalogues et imprimés publicitaires</field>
    <field name="code">61442</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61443" model="account.account.template">
    <field name="name">Foires et expositions</field>
    <field name="code">61443</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61444" model="account.account.template">
    <field name="name">Primes de publicité</field>
    <field name="code">61444</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61446" model="account.account.template">
    <field name="name">Publications</field>
    <field name="code">61446</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61447" model="account.account.template">
    <field name="name">Cadeaux à  la clientèle</field>
    <field name="code">61447</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61448" model="account.account.template">
    <field name="name">Autres charges de publicité et relations publiques</field>
    <field name="code">61448</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61451" model="account.account.template">
    <field name="name">Frais postaux</field>
    <field name="code">61451</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61456" model="account.account.template">
    <field name="name">Frais de télex et de télégrammes</field>
    <field name="code">61456</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61461" model="account.account.template">
    <field name="name">Cotisations</field>
    <field name="code">61461</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61462" model="account.account.template">
    <field name="name">Dons</field>
    <field name="code">61462</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61471" model="account.account.template">
    <field name="name">Frais d'achat et de vente des titres</field>
    <field name="code">61471</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61472" model="account.account.template">
    <field name="name">Frais sur effets de commerce</field>
    <field name="code">61472</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61473" model="account.account.template">
    <field name="name">Frais et commissions sur services bancaires</field>
    <field name="code">61473</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6149" model="account.account.template">
    <field name="name">Rabais, remises et ristournes obtenus sur autres charges externes</field>
    <field name="code">6149</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61611" model="account.account.template">
    <field name="name">Taxe urbaine et taxe d'édilité</field>
    <field name="code">61611</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61612" model="account.account.template">
    <field name="name">Patente</field>
    <field name="code">61612</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61615" model="account.account.template">
    <field name="name">Taxes locales</field>
    <field name="code">61615</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6165" model="account.account.template">
    <field name="name">Impôts et taxes directs</field>
    <field name="code">6165</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61671" model="account.account.template">
    <field name="name">Droits d'enregistrement et de timbre</field>
    <field name="code">61671</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_616731" model="account.account.template">
    <field name="name">La vignette</field>
    <field name="code">616731</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61678" model="account.account.template">
    <field name="name">Autres impôts, taxes et droits assimilés</field>
    <field name="code">61678</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6168" model="account.account.template">
    <field name="name">Impôts et taxes des exercices antérieurs</field>
    <field name="code">6168</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61711" model="account.account.template">
    <field name="name">Appointements et salaires</field>
    <field name="code">61711</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_617121" model="account.account.template">
    <field name="name">Primes de représentation</field>
    <field name="code">617121</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_617132" model="account.account.template">
    <field name="name">Indemnités de déplacement</field>
    <field name="code">617132</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61714" model="account.account.template">
    <field name="name">Commissions au personnel</field>
    <field name="code">61714</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61715" model="account.account.template">
    <field name="name">Rémunérations des administrateurs, gérants et associés</field>
    <field name="code">61715</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61741" model="account.account.template">
    <field name="name">Cotisations de sécurité sociale</field>
    <field name="code">61741</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61742" model="account.account.template">
    <field name="name">Cotisations aux caisses de retraite</field>
    <field name="code">61742</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61743" model="account.account.template">
    <field name="name">Cotisations aux mutuelles</field>
    <field name="code">61743</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61744" model="account.account.template">
    <field name="name">Prestations familiales</field>
    <field name="code">61744</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61745" model="account.account.template">
    <field name="name">Assurances accidents de travail</field>
    <field name="code">61745</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61761" model="account.account.template">
    <field name="name">Assurances groupe</field>
    <field name="code">61761</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61762" model="account.account.template">
    <field name="name">Prestations de retraites</field>
    <field name="code">61762</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61763" model="account.account.template">
    <field name="name">Allocations aux oeuvres sociales</field>
    <field name="code">61763</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61764" model="account.account.template">
    <field name="name">Habillement et vêtements de travail</field>
    <field name="code">61764</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61765" model="account.account.template">
    <field name="name">Indemnités de préavis et de licenciement</field>
    <field name="code">61765</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61766" model="account.account.template">
    <field name="name">Médecine de travail, pharmacie</field>
    <field name="code">61766</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61768" model="account.account.template">
    <field name="name">Autres charges sociales diverses</field>
    <field name="code">61768</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61771" model="account.account.template">
    <field name="name">Appointements et salaires</field>
    <field name="code">61771</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61774" model="account.account.template">
    <field name="name">Charges sociales sur appointements et salaires de l'exploitant</field>
    <field name="code">61774</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6178" model="account.account.template">
    <field name="name">Charges du personnel des exercices antérieurs</field>
    <field name="code">6178</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6181" model="account.account.template">
    <field name="name">Jetons de présence</field>
    <field name="code">6181</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6182" model="account.account.template">
    <field name="name">Pertes sur créances irrécouvrables</field>
    <field name="code">6182</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6185" model="account.account.template">
    <field name="name">Pertes sur opérations faites en commun</field>
    <field name="code">6185</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6186" model="account.account.template">
    <field name="name">Transfert de profits sur opérations faites en commun</field>
    <field name="code">6186</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6188" model="account.account.template">
    <field name="name">Autres charges d'exploitation des exercices antérieurs</field>
    <field name="code">6188</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61911" model="account.account.template">
    <field name="name">D.E.A. des frais préliminaires</field>
    <field name="code">61911</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61912" model="account.account.template">
    <field name="name">D.E.A. des charges à  répartir</field>
    <field name="code">61912</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61921" model="account.account.template">
    <field name="name">D.E.A. de l'immobilisation en recherche et développement</field>
    <field name="code">61921</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61922" model="account.account.template">
    <field name="name">D.E.A. des brevets, marques, droits et valeurs similaires</field>
    <field name="code">61922</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61923" model="account.account.template">
    <field name="name">D.E.A. du fonds commercial</field>
    <field name="code">61923</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61928" model="account.account.template">
    <field name="name">D.E.A. des autres immobilisations incorporelles</field>
    <field name="code">61928</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61931" model="account.account.template">
    <field name="name">D.E.A. des terrains</field>
    <field name="code">61931</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61932" model="account.account.template">
    <field name="name">D.E.A. des constructions</field>
    <field name="code">61932</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61933" model="account.account.template">
    <field name="name">D.E.A. des installations techniques, matériel et outillage</field>
    <field name="code">61933</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61934" model="account.account.template">
    <field name="name">D.E.A. du matériel de transport</field>
    <field name="code">61934</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61935" model="account.account.template">
    <field name="name">D.E.A. des mobiliers, matériels de bureau et aménagements divers</field>
    <field name="code">61935</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61938" model="account.account.template">
    <field name="name">D.E.A. des autres immobilisations corporelles</field>
    <field name="code">61938</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61942" model="account.account.template">
    <field name="name">D.E.P. pour dépréciation des immobilisations incorporelles</field>
    <field name="code">61942</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61943" model="account.account.template">
    <field name="name">D.E.P. pour dépréciation des immobilisations corporelles</field>
    <field name="code">61943</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61955" model="account.account.template">
    <field name="name">D.E.P. pour risques et charges durables</field>
    <field name="code">61955</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61957" model="account.account.template">
    <field name="name">D.E.P. pour risques et charges momentanés</field>
    <field name="code">61957</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61961" model="account.account.template">
    <field name="name">D.E.P. pour dépréciation des stocks</field>
    <field name="code">61961</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61964" model="account.account.template">
    <field name="name">D.E.P. pour dépréciation des créances de l'actif circulant</field>
    <field name="code">61964</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61981" model="account.account.template">
    <field name="name">D.E. aux amortissements des exercices antérieurs</field>
    <field name="code">61981</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_61984" model="account.account.template">
    <field name="name">D.E. aux provisions des exercices antérieurs</field>
    <field name="code">61984</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_63111" model="account.account.template">
    <field name="name">Intérêts des emprunts</field>
    <field name="code">63111</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_63113" model="account.account.template">
    <field name="name">Intérêts des dettes rattachées à  des participations</field>
    <field name="code">63113</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_63114" model="account.account.template">
    <field name="name">Intérêts des comptes courants et dépôts créditeurs</field>
    <field name="code">63114</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_63115" model="account.account.template">
    <field name="name">Intérêts bancaires et sur opérations de financement</field>
    <field name="code">63115</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_63118" model="account.account.template">
    <field name="name">Autres intérêts des emprunts et dettes</field>
    <field name="code">63118</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6318" model="account.account.template">
    <field name="name">Charges d'intérêts des exercices antérieurs</field>
    <field name="code">6318</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_633" model="account.account.template">
    <field name="name">Pertes de change</field>
    <field name="code">633</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6331" model="account.account.template">
    <field name="name">Pertes de change propres à  l'exercice</field>
    <field name="code">6331</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6338" model="account.account.template">
    <field name="name">Pertes de change des exercices antérieurs</field>
    <field name="code">6338</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6382" model="account.account.template">
    <field name="name">Pertes sur créances liées à  des participations</field>
    <field name="code">6382</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6385" model="account.account.template">
    <field name="name">Charges nettes sur cession de titres et valeurs de placement</field>
    <field name="code">6385</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6386" model="account.account.template">
    <field name="name">Escomptes accordés</field>
    <field name="code">6386</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6388" model="account.account.template">
    <field name="name">Autres charges financières des exercices antérieurs</field>
    <field name="code">6388</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6391" model="account.account.template">
    <field name="name">Dotations aux amortissements des primes de remboursement des obligations</field>
    <field name="code">6391</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6392" model="account.account.template">
    <field name="name">Dotations aux provisions pour dépréciations des immobilisations financières</field>
    <field name="code">6392</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6393" model="account.account.template">
    <field name="name">Dotations aux provisions pour risques et charges financières</field>
    <field name="code">6393</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6394" model="account.account.template">
    <field name="name">Dotation aux provisions pour dépréciation des titres et valeurs de placement</field>
    <field name="code">6394</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6396" model="account.account.template">
    <field name="name">Dotations aux provisions pour dépréciation des comptes de trésorerie</field>
    <field name="code">6396</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6398" model="account.account.template">
    <field name="name">Dotations financières des exercices antérieurs</field>
    <field name="code">6398</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6512" model="account.account.template">
    <field name="name">VNA des immobilisations incorporelles cédées</field>
    <field name="code">6512</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6513" model="account.account.template">
    <field name="name">VNA des immobilisations corporelles cédées</field>
    <field name="code">6513</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6514" model="account.account.template">
    <field name="name">VNA provisions des immobilisations financières cédées (droits de propriété)</field>
    <field name="code">6514</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6518" model="account.account.template">
    <field name="name">VNA des immobilisations cédées des exercices antérieurs</field>
    <field name="code">6518</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6561" model="account.account.template">
    <field name="name">Subventions accordées de l'exercice</field>
    <field name="code">6561</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6568" model="account.account.template">
    <field name="name">Subventions accordées des exercices antérieurs</field>
    <field name="code">6568</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65811" model="account.account.template">
    <field name="name">Pénalités sur marchés</field>
    <field name="code">65811</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65812" model="account.account.template">
    <field name="name">Dédits</field>
    <field name="code">65812</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6582" model="account.account.template">
    <field name="name">Rappels d'impôts (autres que l'impôt sur les résultats)</field>
    <field name="code">6582</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65831" model="account.account.template">
    <field name="name">Pénalités et amendes fiscales</field>
    <field name="code">65831</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65833" model="account.account.template">
    <field name="name">Pénalités et amendes pénales</field>
    <field name="code">65833</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6585" model="account.account.template">
    <field name="name">Créances devenues irrécouvrables</field>
    <field name="code">6585</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65861" model="account.account.template">
    <field name="name">Dons</field>
    <field name="code">65861</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65862" model="account.account.template">
    <field name="name">Libéralités</field>
    <field name="code">65862</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65863" model="account.account.template">
    <field name="name">Lots</field>
    <field name="code">65863</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6588" model="account.account.template">
    <field name="name">Autres charges non courantes des exercices antérieurs</field>
    <field name="code">6588</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65911" model="account.account.template">
    <field name="name">D.A.E. de l'immobilisation en non-valeurs</field>
    <field name="code">65911</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65912" model="account.account.template">
    <field name="name">D.A.E. des immobilisations incorporelles</field>
    <field name="code">65912</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65913" model="account.account.template">
    <field name="name">D.A.E. des immobilisations corporelles</field>
    <field name="code">65913</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65941" model="account.account.template">
    <field name="name">D.N.C. pour amortissements dérogatoires</field>
    <field name="code">65941</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65942" model="account.account.template">
    <field name="name">D.N.C. pour plus-values en instance d'imposition</field>
    <field name="code">65942</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65944" model="account.account.template">
    <field name="name">D.N.C. pour investissements</field>
    <field name="code">65944</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65945" model="account.account.template">
    <field name="name">D.N.C. pour reconstitution de gisements</field>
    <field name="code">65945</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65946" model="account.account.template">
    <field name="name">D.N.C. pour acquisition et construction de logements</field>
    <field name="code">65946</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65955" model="account.account.template">
    <field name="name">D.N.C. aux provisions pour risques et charges durables</field>
    <field name="code">65955</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65957" model="account.account.template">
    <field name="name">D.N.C. aux provisions pour risques et charges momentanés</field>
    <field name="code">65957</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65962" model="account.account.template">
    <field name="name">D.N.C. aux provisions pour dépréciation de l'actif immobilisé</field>
    <field name="code">65962</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_65963" model="account.account.template">
    <field name="name">D.N.C. aux provisions pour dépréciation de l'actif circulant</field>
    <field name="code">65963</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6598" model="account.account.template">
    <field name="name">Dotations non courantes des exercices antérieurs</field>
    <field name="code">6598</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6701" model="account.account.template">
    <field name="name">Impôts sur les bénéfices</field>
    <field name="code">6701</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6705" model="account.account.template">
    <field name="name">Imposition minimale annuelle des sociétés</field>
    <field name="code">6705</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_6708" model="account.account.template">
    <field name="name">Rappels et dégrèvements d'impôts sur les résultats</field>
    <field name="code">6708</field>
    <field name="account_type">expense</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7111" model="account.account.template">
    <field name="name">Ventes de marchandises au Maroc</field>
    <field name="code">7111</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7113" model="account.account.template">
    <field name="name">Ventes de marchandises à  l'étranger</field>
    <field name="code">7113</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7118" model="account.account.template">
    <field name="name">Ventes de marchandises des exercices antérieurs</field>
    <field name="code">7118</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7119" model="account.account.template">
    <field name="name">Rabais, remises et ristournes accordés par l'entreprise</field>
    <field name="code">7119</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71211" model="account.account.template">
    <field name="name">Ventes de produits finis</field>
    <field name="code">71211</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71212" model="account.account.template">
    <field name="name">Ventes de produits intermédiaires</field>
    <field name="code">71212</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71217" model="account.account.template">
    <field name="name">Ventes de produits résiduels</field>
    <field name="code">71217</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71221" model="account.account.template">
    <field name="name">Ventes de produits finis</field>
    <field name="code">71221</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71222" model="account.account.template">
    <field name="name">Ventes de produits intermédiaires</field>
    <field name="code">71222</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71241" model="account.account.template">
    <field name="name">Travaux</field>
    <field name="code">71241</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71242" model="account.account.template">
    <field name="name">Etudes</field>
    <field name="code">71242</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71243" model="account.account.template">
    <field name="name">Prestations de services</field>
    <field name="code">71243</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71251" model="account.account.template">
    <field name="name">Travaux</field>
    <field name="code">71251</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71252" model="account.account.template">
    <field name="name">Etudes</field>
    <field name="code">71252</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71253" model="account.account.template">
    <field name="name">Prestations de services</field>
    <field name="code">71253</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7126" model="account.account.template">
    <field name="name">Redevances pour brevets, marques, droits et valeurs similaires</field>
    <field name="code">7126</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71271" model="account.account.template">
    <field name="name">Locations divers es reçues</field>
    <field name="code">71271</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71272" model="account.account.template">
    <field name="name">Commissions et courtages reçus</field>
    <field name="code">71272</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71273" model="account.account.template">
    <field name="name">Produits de services exploités dans l'intérêt du personnel</field>
    <field name="code">71273</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71275" model="account.account.template">
    <field name="name">Bonis sur reprises d'emballages consignés</field>
    <field name="code">71275</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71276" model="account.account.template">
    <field name="name">Ports et frais accessoires facturés</field>
    <field name="code">71276</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71278" model="account.account.template">
    <field name="name">Autres ventes et produits accessoires</field>
    <field name="code">71278</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7128" model="account.account.template">
    <field name="name">Ventes de biens et services produits des exercices antérieurs</field>
    <field name="code">7128</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71291" model="account.account.template">
    <field name="name">R,R,R accordées sur ventes au Maroc des biens produits</field>
    <field name="code">71291</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71292" model="account.account.template">
    <field name="name">R,R,R accordées sur ventes à  l'étranger des biens produits</field>
    <field name="code">71292</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71294" model="account.account.template">
    <field name="name">R,R,R accordées sur ventes au Maroc des services produits</field>
    <field name="code">71294</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71295" model="account.account.template">
    <field name="name">R,R,R accordées sur ventes à  l'étranger des services produits</field>
    <field name="code">71295</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71298" model="account.account.template">
    <field name="name">Rabais, remises et ristournes accordés sur ventes de B et S produits des exercices antérieurs</field>
    <field name="code">71298</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71311" model="account.account.template">
    <field name="name">Variation des stocks de biens produits en cours</field>
    <field name="code">71311</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71312" model="account.account.template">
    <field name="name">Variation des stocks de produits intermédiaires en cours</field>
    <field name="code">71312</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71317" model="account.account.template">
    <field name="name">Variation des stocks de produits résiduels en cours</field>
    <field name="code">71317</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71321" model="account.account.template">
    <field name="name">Variation des stocks de produits finis</field>
    <field name="code">71321</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71322" model="account.account.template">
    <field name="name">Variation des stocks de produits intermédiaires</field>
    <field name="code">71322</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71327" model="account.account.template">
    <field name="name">Variation des stocks de produits résiduels</field>
    <field name="code">71327</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71341" model="account.account.template">
    <field name="name">Variation des stocks de travaux en cours</field>
    <field name="code">71341</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71342" model="account.account.template">
    <field name="name">Variation des stocks d'études en cours</field>
    <field name="code">71342</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71343" model="account.account.template">
    <field name="name">Variation des stocks de prestations en cours</field>
    <field name="code">71343</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7141" model="account.account.template">
    <field name="name">Immobilisation en non valeurs produite</field>
    <field name="code">7141</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7142" model="account.account.template">
    <field name="name">Immobilisations incorporelles produites</field>
    <field name="code">7142</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7143" model="account.account.template">
    <field name="name">Immobilisations corporelles produites</field>
    <field name="code">7143</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7148" model="account.account.template">
    <field name="name">Immobilisations produites des exercices antérieurs</field>
    <field name="code">7148</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7161" model="account.account.template">
    <field name="name">Subventions d'exploitations reçues de l'exercice</field>
    <field name="code">7161</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7168" model="account.account.template">
    <field name="name">Subventions d'exploitation reçues des exercices antérieurs</field>
    <field name="code">7168</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7181" model="account.account.template">
    <field name="name">Jetons de présence reçus</field>
    <field name="code">7181</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7182" model="account.account.template">
    <field name="name">Revenus des immeubles non affectés à  l'exploitation</field>
    <field name="code">7182</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7185" model="account.account.template">
    <field name="name">Profits sur opérations faites en commun</field>
    <field name="code">7185</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7186" model="account.account.template">
    <field name="name">Transfert de pertes sur opérations faites en commun</field>
    <field name="code">7186</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7188" model="account.account.template">
    <field name="name">Autres produits d'exploitation des exercices antérieurs</field>
    <field name="code">7188</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7191" model="account.account.template">
    <field name="name">Reprises sur amortissements de l'immobilisation en non valeurs</field>
    <field name="code">7191</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7192" model="account.account.template">
    <field name="name">Reprises sur amortissements des immobilisations incorporelles</field>
    <field name="code">7192</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7193" model="account.account.template">
    <field name="name">Reprises sur amortissements des immobilisations corporelles</field>
    <field name="code">7193</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7194" model="account.account.template">
    <field name="name">Reprises sur provisions pour dépréciation des immobilisations</field>
    <field name="code">7194</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7195" model="account.account.template">
    <field name="name">Reprises sur provisions pour risques et charges</field>
    <field name="code">7195</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7196" model="account.account.template">
    <field name="name">Reprises sur provisions pour dépréciation de l'actif circulant</field>
    <field name="code">7196</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71971" model="account.account.template">
    <field name="name">T,C,E - Achats de marchandises</field>
    <field name="code">71971</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71972" model="account.account.template">
    <field name="name">T,C,E - Achats consommés de matières et fournitures</field>
    <field name="code">71972</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71973" model="account.account.template">
    <field name="name">T,C,E-Autres charges externes</field>
    <field name="code">71973</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71975" model="account.account.template">
    <field name="name">T,C,E - Impôts et taxes</field>
    <field name="code">71975</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71976" model="account.account.template">
    <field name="name">T,C,E - Charges de personnel</field>
    <field name="code">71976</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71978" model="account.account.template">
    <field name="name">T,C,E - Autres charges d'exploitation</field>
    <field name="code">71978</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71981" model="account.account.template">
    <field name="name">Reprises sur amortissements des exercices antérieurs</field>
    <field name="code">71981</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_71984" model="account.account.template">
    <field name="name">Reprises sur provisions des exercices antérieurs</field>
    <field name="code">71984</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7321" model="account.account.template">
    <field name="name">Revenus des titres de participation</field>
    <field name="code">7321</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7325" model="account.account.template">
    <field name="name">Revenus des titres immobilisés</field>
    <field name="code">7325</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7328" model="account.account.template">
    <field name="name">Produits des titres de participation et des autres titres immobilisés des exercices antérieurs</field>
    <field name="code">7328</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_733" model="account.account.template">
    <field name="name">Gains de change</field>
    <field name="code">733</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7331" model="account.account.template">
    <field name="name">Gains de change propres à  l'exercice</field>
    <field name="code">7331</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7338" model="account.account.template">
    <field name="name">Gains de change des exercices antérieurs</field>
    <field name="code">7338</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_73811" model="account.account.template">
    <field name="name">Intérêts des prêts</field>
    <field name="code">73811</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_73813" model="account.account.template">
    <field name="name">Revenus des autres créances financières</field>
    <field name="code">73813</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7383" model="account.account.template">
    <field name="name">Revenus des créances rattachées à  des participations</field>
    <field name="code">7383</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7384" model="account.account.template">
    <field name="name">Revenus des titres et valeurs de placement</field>
    <field name="code">7384</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7385" model="account.account.template">
    <field name="name">Produits nets sur cessions de titres et valeurs de placement</field>
    <field name="code">7385</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7386" model="account.account.template">
    <field name="name">Escomptes obtenus</field>
    <field name="code">7386</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7388" model="account.account.template">
    <field name="name">Intérêts et autres produits financiers des exercices antérieurs</field>
    <field name="code">7388</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7391" model="account.account.template">
    <field name="name">Reprises sur amortissements des primes de remboursement des obligations</field>
    <field name="code">7391</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7392" model="account.account.template">
    <field name="name">Reprises sur provisions pour dépréciation des immobilisations financières</field>
    <field name="code">7392</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7393" model="account.account.template">
    <field name="name">Reprises sur provisions pour risques et charges financières</field>
    <field name="code">7393</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7394" model="account.account.template">
    <field name="name">Reprise sur provisions pour dépréciation des titres et valeurs de placement</field>
    <field name="code">7394</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7396" model="account.account.template">
    <field name="name">Reprises sur provisions pour dépréciation des comptes de trésorerie</field>
    <field name="code">7396</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_73971" model="account.account.template">
    <field name="name">Transfert - Charges d'intérêts</field>
    <field name="code">73971</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_73973" model="account.account.template">
    <field name="name">Transfert - Pertes de change</field>
    <field name="code">73973</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_73978" model="account.account.template">
    <field name="name">Transfert - Autres charges financières</field>
    <field name="code">73978</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7398" model="account.account.template">
    <field name="name">Reprises sur dotations financières des exercices antérieurs</field>
    <field name="code">7398</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7512" model="account.account.template">
    <field name="name">Produits des cessions des immobilisations incorporelles</field>
    <field name="code">7512</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7513" model="account.account.template">
    <field name="name">Produits des cessions des immobilisations corporelles</field>
    <field name="code">7513</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7514" model="account.account.template">
    <field name="name">Produits des cessions des immobilisations financières (droits de propriété)</field>
    <field name="code">7514</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7518" model="account.account.template">
    <field name="name">Produits des cessions des immobilisations des exercices antérieurs</field>
    <field name="code">7518</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7561" model="account.account.template">
    <field name="name">Subventions d'équilibre reçues de l'exercice</field>
    <field name="code">7561</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7568" model="account.account.template">
    <field name="name">Subventions d'équilibre reçues des exercices antérieurs</field>
    <field name="code">7568</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7577" model="account.account.template">
    <field name="name">Reprises sur subventions d'investissement de l'exercice</field>
    <field name="code">7577</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7578" model="account.account.template">
    <field name="name">Reprises sur subventions d'investissement des exercices antérieurs</field>
    <field name="code">7578</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75811" model="account.account.template">
    <field name="name">Pénalités reçues sur marchés</field>
    <field name="code">75811</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75812" model="account.account.template">
    <field name="name">Dédits reçus</field>
    <field name="code">75812</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7582" model="account.account.template">
    <field name="name">Dégrèvement d'impôts (autres que l'impôt sur les résultats)</field>
    <field name="code">7582</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7585" model="account.account.template">
    <field name="name">Rentrées sur créances soldées</field>
    <field name="code">7585</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75861" model="account.account.template">
    <field name="name">Dons</field>
    <field name="code">75861</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75862" model="account.account.template">
    <field name="name">Libéralités</field>
    <field name="code">75862</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75863" model="account.account.template">
    <field name="name">Lots</field>
    <field name="code">75863</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7588" model="account.account.template">
    <field name="name">Autres produits non courants des exercices antérieurs</field>
    <field name="code">7588</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75911" model="account.account.template">
    <field name="name">R,A,E des immobilisations en non valeur</field>
    <field name="code">75911</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75912" model="account.account.template">
    <field name="name">R,A,E des immobilisations incorporelles</field>
    <field name="code">75912</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75913" model="account.account.template">
    <field name="name">R,A,E des immobilisations corporelles</field>
    <field name="code">75913</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75941" model="account.account.template">
    <field name="name">Reprises sur amortissements dérogatoires</field>
    <field name="code">75941</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75942" model="account.account.template">
    <field name="name">Reprises sur plus-values en instance d'imposition</field>
    <field name="code">75942</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75944" model="account.account.template">
    <field name="name">Reprises sur provisions pour investissements</field>
    <field name="code">75944</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75945" model="account.account.template">
    <field name="name">Reprises sur provisions pour reconstitution de gisements</field>
    <field name="code">75945</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75946" model="account.account.template">
    <field name="name">Reprises sur provisions pour acquisition et construction de logements</field>
    <field name="code">75946</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75955" model="account.account.template">
    <field name="name">Reprises sur provisions pour risques et charges durables</field>
    <field name="code">75955</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75957" model="account.account.template">
    <field name="name">Reprises sur provisions pour risques et charges momentanés</field>
    <field name="code">75957</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75962" model="account.account.template">
    <field name="name">R,N,C sur provisions pour dépréciation de l'actif immobilisé</field>
    <field name="code">75962</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_75963" model="account.account.template">
    <field name="name">R,N,C sur provisions pour dépréciation de l'actif circulant</field>
    <field name="code">75963</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7597" model="account.account.template">
    <field name="name">Transferts de charges non courantes</field>
    <field name="code">7597</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
  <record id="pcg_7598" model="account.account.template">
    <field name="name">Reprises non courantes des exercices antérieurs.</field>
    <field name="code">7598</field>
    <field name="account_type">income_other</field>
    <field name="chart_template_id" ref="l10n_kzc_temp_chart"/>
  </record>
</odoo>

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="-6" y="-3" width="72" height="52" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.68" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="900" height="600" transform="translate(-6 -3) scale(0.08 0.09)" xlink:href="data:image/png;base64,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AgAAJCUIAQAAkhKEAAAASQlCAACApAQhAABAUoIQAAAgKUEIAACQlCAEAABIShACAAAkJQgBAACSEoQAAABJCUIAAICkBCEAAEBSghAAACApQQgAAJCUIAQAAEhKEAIAACQlCAEAAJIShAAAAEkJQgAAgKQEIQAAQFKCEAAAIClBCAAAkJQgBAAASEoQAgAAJCUIAQAAkhKEAAAASQlCAACApAQhAABAUoIQAAAgKUEIAACQlCAEAABIShACAAAkJQgBAACSEoQAAABJCUIAAICkBCEAAEBSghAAACApQQgAAJCUIAQAAEhKEAIAACQlCAEAAJIShAAAAEkJQgAAgKQEIQAAQFKCEAAAIClBCAAAkJQgBAAASEoQAgAAJCUIAQAAkhKEAAAASQlCAACApAQhAABAUoIQAAAgKUEIAACQlCAEAABIShACAAAkJQgBAACSEoQAAABJCUIAAICkBCEAAEBSghAAACApQQgAAJCUIAQAAEhKEAIAACQlCAEAAJIShAAAAEkJQgAAgKQEIQAAQFKCEAAAIClBCAAAkJQgBAAASEoQAgAAJCUIAQAAkhKEAAAASQlCAACApAQhAABAUoIQAAAgKUEIAACQlCAEAABIShACAAAkJQgBAACSEoQAAABJCUIAAICkBCEAAEBSghAAACApQQgAAJCUIAQAAEhKEAIAACQlCAEAAJIShAAAAEkJQgAAgKQEIQAAQFKCEAAAIClBCAAAkJQgBAAASEoQAgAAJCUIAQAAkhKEAAAASQlCAACApAQhAABAUoIQAAAgKUEIAACQlCAEAABIShACAAAkJQgBAACSEoQAAABJCUIAAICkBCEAAEBSghAAACApQQgAAJCUIAQAAEhKEAIAACQlCAEAAJIShAAAAEn9f+TkH1mDxMHHAAAAAElFTkSuQmCC"/>
    </g>
  </g>
</svg>

```

