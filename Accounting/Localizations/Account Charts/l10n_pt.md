# Odoo Module: l10n_pt

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2012 Thinkopen Solutions, Lda. All Rights Reserved
# http://www.thinkopensolutions.com.

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2012 Thinkopen Solutions, Lda. All Rights Reserved
# http://www.thinkopensolutions.com.

{
    'name': 'Portugal - Accounting',
    'version': '1.1',
    'author': 'ThinkOpen Solutions',
    'website': 'http://www.thinkopensolutions.com/',
    'category': 'Accounting/Localizations/Account Charts',
    'description': 'Plano de contas SNC para Portugal',
    'depends': ['base',
                'account',
                'base_vat',
                ],
    'data': [
           'data/l10n_pt_chart_data.xml',
           'data/account_chart_template_data.xml',
           'data/account_fiscal_position_template_data.xml',
           'data/account_tax_group_data.xml',
           'data/account_tax_report.xml',
           'data/account_tax_data.xml',
           'data/account_chart_template_configure_data.xml',
           ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_pt.pt_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
    <!-- Chart template -->

    <record id="pt_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="chart_2111"/>
        <field name="property_account_payable_id" ref="chart_2211"/>
        <field name="property_account_expense_id" ref="chart_311"/>
        <field name="property_account_income_id" ref="chart_711"/>
        <field name="property_account_income_categ_id" ref="chart_711"/>
        <field name="property_account_expense_categ_id" ref="chart_311"/>
        <field name="income_currency_exchange_account_id" ref="chart_7861"/>
        <field name="expense_currency_exchange_account_id" ref="chart_692"/>
        <field name="default_pos_receivable_account_id" ref="chart_2117"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="chart_682"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="chart_728"/>
        <field name="property_tax_payable_account_id" ref="chart_2436"/>
        <field name="property_tax_receivable_account_id" ref="chart_2437"/>
    </record>

    </data>
</odoo>

```

## File: data\account_fiscal_position_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

    <!-- Fiscal Position Templates -->

    <record id="fiscal_position_national_customers" model="account.fiscal.position.template">
        <field name="sequence">1</field>
        <field name="name">Portugal</field>
        <field name="chart_template_id" ref="pt_chart_template"/>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="country_id" ref="base.pt"/>
    </record>

    <record id="fiscal_position_foreign_eu_private" model="account.fiscal.position.template">
        <field name="sequence">2</field>
        <field name="name">Europa Privado</field>
        <field name="chart_template_id" ref="pt_chart_template"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_group_id" ref="base.europe"/>
    </record>

    <record id="fiscal_position_foreign_eu" model="account.fiscal.position.template">
        <field name="sequence">3</field>
        <field name="name">Europa</field>
        <field name="chart_template_id" ref="pt_chart_template"/>
        <field name="auto_apply" eval="True"/>
        <field name="vat_required" eval="True"/>
        <field name="country_group_id" ref="base.europe"/>
    </record>

    <record id="fiscal_position_foreign_other" model="account.fiscal.position.template">
        <field name="sequence">4</field>
        <field name="name">Extra-comunitário</field>
        <field name="chart_template_id" ref="pt_chart_template"/>
        <field name="auto_apply" eval="True"/>
    </record>

    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="iva23" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA23</field>
            <field name="description">IVA23 (taxa normal Portugal Continental)</field>
            <field name="amount">23</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_iva_23"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_vendas_faturas_normal_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_vendas_faturas_normal_tag')],
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_vendas_notas_credito_normal_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_vendas_notas_credito_normal_tag')],
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
        </record>
        <record id="iva22" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA22</field>
            <field name="description">IVA22 (taxa normal Madeira)</field>
            <field name="amount">22</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_iva_22"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
        </record>
        <record id="iva16" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA16</field>
            <field name="description">IVA16 (taxa normal Açores)</field>
            <field name="amount">16</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_iva_16"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
        </record>
        <record id="iva13" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA13</field>
            <field name="description">IVA13 (taxa intermédia Portugal Continental)</field>
            <field name="amount">13</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_iva_13"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_vendas_faturas_intermedia_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_vendas_faturas_intermedia_tag')],
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_vendas_notas_credito_intermedia_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_vendas_notas_credito_intermedia_tag')],
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
        </record>
        <record id="iva12" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA12</field>
            <field name="description">IVA12 (taxa intermédia Madeira)</field>
            <field name="amount">12</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_iva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
        </record>
        <record id="iva9" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA9</field>
            <field name="description">IVA9 (taxa intermédia Açores)</field>
            <field name="amount">9</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_iva_9"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
        </record>
        <record id="iva6" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA6</field>
            <field name="description">IVA6 (taxa reduzida Portugal Continental)</field>
            <field name="amount">6</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_iva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_vendas_faturas_reduzida_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_vendas_faturas_reduzida_tag')],
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_vendas_notas_credito_reduzida_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_vendas_notas_credito_reduzida_tag')],
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
        </record>
        <record id="iva5" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA5</field>
            <field name="description">IVA5 (taxa reduzida Madeira)</field>
            <field name="amount">5</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_iva_5"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
        </record>
        <record id="iva4" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA4</field>
            <field name="description">IVA4 (taxa reduzida Açores)</field>
            <field name="amount">4</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="tax_group_id" ref="tax_group_iva_4"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
        </record>
        <record id="iva0" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA0</field>
            <field name="description">IVA0</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="tax_group_id" ref="tax_group_iva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_vendas_faturas_isento_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_vendas_faturas_isento_tag')],
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_vendas_notas_credito_isento_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_vendas_notas_credito_isento_tag')],
                    'account_id': ref('chart_2433'),
                }),
            ]"/>
        </record>
        <record id="compiva23" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA23 compra</field>
            <field name="description">IVA23 compra (taxa normal Portugal Continental)</field>
            <field name="amount">23</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_23"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_despesas_faturas_normal_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_despesas_faturas_normal_tag')],
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_despesas_notas_credito_normal_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_despesas_notas_credito_normal_tag')],
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
        </record>
        <record id="compiva22" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA22 compra</field>
            <field name="description">IVA22 compra (taxa normal Madeira)</field>
            <field name="amount">22</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_22"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
        </record>
        <record id="compiva16" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA16 compra</field>
            <field name="description">IVA16 compra (taxa normal Açores)</field>
            <field name="amount">16</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_16"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
        </record>
        <record id="compiva13" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA13 compra</field>
            <field name="description">IVA13 compra (taxa intermédia Portugal Continental)</field>
            <field name="amount">13</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_13"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_despesas_faturas_intermedia_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_despesas_faturas_intermedia_tag')],
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_despesas_notas_credito_intermedia_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_despesas_notas_credito_intermedia_tag')],
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
        </record>
        <record id="compiva12" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA12 compra</field>
            <field name="description">IVA12 compra (taxa intermédia Madeira)</field>
            <field name="amount">12</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_12"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
        </record>
        <record id="compiva9" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA9 compra</field>
            <field name="description">IVA9 compra (taxa intermédia Açores)</field>
            <field name="amount">9</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_9"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
        </record>
        <record id="compiva6" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA6 compra</field>
            <field name="description">IVA6 compra (taxa reduzida Portugal Continental)</field>
            <field name="amount">6</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_6"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_despesas_faturas_reduzida_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_despesas_faturas_reduzida_tag')],
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_despesas_notas_credito_reduzida_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_despesas_notas_credito_reduzida_tag')],
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
        </record>
        <record id="compiva5" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA5 compra</field>
            <field name="description">IVA5 compra (taxa reduzida Madeira)</field>
            <field name="amount">5</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_5"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
        </record>
        <record id="compiva4" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA4 compra</field>
            <field name="description">IVA4 compra (taxa reduzida Açores)</field>
            <field name="amount">4</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_4"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {'repartition_type': 'base'}),
                (0,0, {
                    'repartition_type': 'tax',
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
        </record>
        <record id="compiva0" model="account.tax.template">
            <field name="chart_template_id" ref="pt_chart_template"/>
            <field name="name">IVA0 compra</field>
            <field name="description">IVA0 compra</field>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="tax_group_id" ref="tax_group_iva_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_despesas_faturas_isento_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_despesas_faturas_isento_tag')],
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'repartition_type': 'base',
                    'plus_report_expression_ids': [ref('tax_report_pt_base_despesas_notas_credito_isento_tag')],
                }),
                (0,0, {
                    'repartition_type': 'tax',
                    'plus_report_expression_ids': [ref('tax_report_pt_tax_despesas_notas_credito_isento_tag')],
                    'account_id': ref('chart_2432'),
                }),
            ]"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_iva_0" model="account.tax.group">
            <field name="name">IVA 0%</field>
            <field name="country_id" ref="base.pt"/>
        </record>
        <record id="tax_group_iva_4" model="account.tax.group">
            <field name="name">IVA 4%</field>
        </record>
        <record id="tax_group_iva_5" model="account.tax.group">
            <field name="name">IVA 5%</field>
        </record>
        <record id="tax_group_iva_6" model="account.tax.group">
            <field name="name">IVA 6%</field>
            <field name="country_id" ref="base.pt"/>
        </record>
        <record id="tax_group_iva_9" model="account.tax.group">
            <field name="name">IVA 9%</field>
        </record>
        <record id="tax_group_iva_12" model="account.tax.group">
            <field name="name">IVA 12%</field>
        </record>
        <record id="tax_group_iva_13" model="account.tax.group">
            <field name="name">IVA 13%</field>
            <field name="country_id" ref="base.pt"/>
        </record>
        <record id="tax_group_iva_16" model="account.tax.group">
            <field name="name">IVA 16%</field>
        </record>
        <record id="tax_group_iva_22" model="account.tax.group">
            <field name="name">IVA 22%</field>
        </record>
        <record id="tax_group_iva_23" model="account.tax.group">
            <field name="name">IVA 23%</field>
            <field name="country_id" ref="base.pt"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_report.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="tax_report" model="account.report">
        <field name="name">Relatório de IVA</field>
        <field name="root_report_id" ref="account.generic_tax_report"/>
        <field name="country_id" ref="base.pt"/>
        <field name="filter_fiscal_position" eval="True"/>
        <field name="availability_condition">country</field>
        <field name="column_ids">
            <record id="tax_report_balance" model="account.report.column">
                <field name="name">Balance</field>
                <field name="expression_label">balance</field>
            </record>
        </field>
        <field name="line_ids">
            <record id="tax_report_pt_base" model="account.report.line">
                <field name="name">Base</field>
                <field name="children_ids">
                    <record id="tax_report_pt_base_vendas" model="account.report.line">
                        <field name="name">Vendas</field>
                        <field name="children_ids">
                            <record id="tax_report_pt_base_vendas_faturas" model="account.report.line">
                                <field name="name">Faturas/Notas de débito</field>
                                <field name="aggregation_formula">TAX_REPORT_PT_BASE_VENDAS_FATURAS_NORMAL.balance + TAX_REPORT_PT_BASE_VENDAS_FATURAS_INTERMEDIA.balance + TAX_REPORT_PT_BASE_VENDAS_FATURAS_REDUZIDA.balance + TAX_REPORT_PT_BASE_VENDAS_FATURAS_ISENTO.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_pt_base_vendas_faturas_normal" model="account.report.line">
                                        <field name="name">Normal</field>
                                        <field name="code">TAX_REPORT_PT_BASE_VENDAS_FATURAS_NORMAL</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_vendas_faturas_normal_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_vendas_faturas_normal</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_base_vendas_faturas_intermedia" model="account.report.line">
                                        <field name="name">Intermédia</field>
                                        <field name="code">TAX_REPORT_PT_BASE_VENDAS_FATURAS_INTERMEDIA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_vendas_faturas_intermedia_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_vendas_faturas_intermedia</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_base_vendas_faturas_reduzida" model="account.report.line">
                                        <field name="name">Reduzida</field>
                                        <field name="code">TAX_REPORT_PT_BASE_VENDAS_FATURAS_REDUZIDA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_vendas_faturas_reduzida_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_vendas_faturas_reduzida</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_base_vendas_faturas_isento" model="account.report.line">
                                        <field name="name">Isento</field>
                                        <field name="code">TAX_REPORT_PT_BASE_VENDAS_FATURAS_ISENTO</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_vendas_faturas_isento_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_vendas_faturas_isento</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_pt_base_vendas_notas_credito" model="account.report.line">
                                <field name="name">Notas de crédito</field>
                                <field name="aggregation_formula">TAX_REPORT_PT_BASE_VENDAS_NOTAS_CREDITO_NORMAL.balance + TAX_REPORT_PT_BASE_VENDAS_NOTAS_CREDITO_INTERMEDIA.balance + TAX_REPORT_PT_BASE_VENDAS_NOTAS_CREDITO_REDUZIDA.balance + TAX_REPORT_PT_BASE_VENDAS_NOTAS_CREDITO_ISENTO.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_pt_base_vendas_notas_credito_normal" model="account.report.line">
                                        <field name="name">Normal</field>
                                        <field name="code">TAX_REPORT_PT_BASE_VENDAS_NOTAS_CREDITO_NORMAL</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_vendas_notas_credito_normal_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_vendas_notas_credito_normal</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_base_vendas_notas_credito_intermedia" model="account.report.line">
                                        <field name="name">Intermédia</field>
                                        <field name="code">TAX_REPORT_PT_BASE_VENDAS_NOTAS_CREDITO_INTERMEDIA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_vendas_notas_credito_intermedia_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_vendas_notas_credito_intermedia</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_base_vendas_notas_credito_reduzida" model="account.report.line">
                                        <field name="name">Reduzida</field>
                                        <field name="code">TAX_REPORT_PT_BASE_VENDAS_NOTAS_CREDITO_REDUZIDA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_vendas_notas_credito_reduzida_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_vendas_notas_credito_reduzida</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_base_vendas_notas_credito_isento" model="account.report.line">
                                        <field name="name">Isento</field>
                                        <field name="code">TAX_REPORT_PT_BASE_VENDAS_NOTAS_CREDITO_ISENTO</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_vendas_notas_credito_isento_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_vendas_notas_credito_isento</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_pt_base_despesas" model="account.report.line">
                        <field name="name">Despesas</field>
                        <field name="children_ids">
                            <record id="tax_report_pt_base_despesas_faturas" model="account.report.line">
                                <field name="name">Faturas</field>
                                <field name="aggregation_formula">TAX_REPORT_PT_BASE_DESPESAS_FATURAS_NORMAL.balance + TAX_REPORT_PT_BASE_DESPESAS_FATURAS_INTERMEDIA.balance + TAX_REPORT_PT_BASE_DESPESAS_FATURAS_REDUZIDA.balance + TAX_REPORT_PT_BASE_DESPESAS_FATURAS_ISENTO.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_pt_base_despesas_faturas_normal" model="account.report.line">
                                        <field name="name">Normal</field>
                                        <field name="code">TAX_REPORT_PT_BASE_DESPESAS_FATURAS_NORMAL</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_despesas_faturas_normal_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_despesas_faturas_normal</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_base_despesas_faturas_intermedia" model="account.report.line">
                                        <field name="name">Intermédia</field>
                                        <field name="code">TAX_REPORT_PT_BASE_DESPESAS_FATURAS_INTERMEDIA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_despesas_faturas_intermedia_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_despesas_faturas_intermedia</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_base_despesas_faturas_reduzida" model="account.report.line">
                                        <field name="name">Reduzida</field>
                                        <field name="code">TAX_REPORT_PT_BASE_DESPESAS_FATURAS_REDUZIDA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_despesas_faturas_reduzida_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_despesas_faturas_reduzida</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_base_despesas_faturas_isento" model="account.report.line">
                                        <field name="name">Isento</field>
                                        <field name="code">TAX_REPORT_PT_BASE_DESPESAS_FATURAS_ISENTO</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_despesas_faturas_isento_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_despesas_faturas_isento</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_pt_base_despesas_notas_credito" model="account.report.line">
                                <field name="name">Notas de crédito</field>
                                <field name="aggregation_formula">TAX_REPORT_PT_BASE_DESPESAS_NOTAS_CREDITO_NORMAL.balance + TAX_REPORT_PT_BASE_DESPESAS_NOTAS_CREDITO_INTERMEDIA.balance + TAX_REPORT_PT_BASE_DESPESAS_NOTAS_CREDITO_REDUZIDA.balance + TAX_REPORT_PT_BASE_DESPESAS_NOTAS_CREDITO_ISENTO.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_pt_base_despesas_notas_credito_normal" model="account.report.line">
                                        <field name="name">Normal</field>
                                        <field name="code">TAX_REPORT_PT_BASE_DESPESAS_NOTAS_CREDITO_NORMAL</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_despesas_notas_credito_normal_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_despesas_notas_credito_normal</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_base_despesas_notas_credito_intermedia" model="account.report.line">
                                        <field name="name">Intermédia</field>
                                        <field name="code">TAX_REPORT_PT_BASE_DESPESAS_NOTAS_CREDITO_INTERMEDIA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_despesas_notas_credito_intermedia_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_despesas_notas_credito_intermedia</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_base_despesas_notas_credito_reduzida" model="account.report.line">
                                        <field name="name">Reduzida</field>
                                        <field name="code">TAX_REPORT_PT_BASE_DESPESAS_NOTAS_CREDITO_REDUZIDA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_despesas_notas_credito_reduzida_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_despesas_notas_credito_reduzida</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_base_despesas_notas_credito_isento" model="account.report.line">
                                        <field name="name">Isento</field>
                                        <field name="code">TAX_REPORT_PT_BASE_DESPESAS_NOTAS_CREDITO_ISENTO</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_base_despesas_notas_credito_isento_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_base_despesas_notas_credito_isento</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_pt_iva" model="account.report.line">
                <field name="name">IVA</field>
                <field name="children_ids">
                    <record id="tax_report_pt_tax_vendas" model="account.report.line">
                        <field name="name">Vendas</field>
                        <field name="children_ids">
                            <record id="tax_report_pt_tax_vendas_faturas" model="account.report.line">
                                <field name="name">Faturas/Notas de débito (Total IVA liquidado)</field>
                                <field name="code">tax_report_pt_tax_vendas_faturas</field>
                                <field name="aggregation_formula">TAX_REPORT_PT_TAX_VENDAS_FATURAS_NORMAL.balance + TAX_REPORT_PT_TAX_VENDAS_FATURAS_INTERMEDIA.balance + TAX_REPORT_PT_TAX_VENDAS_FATURAS_REDUZIDA.balance + TAX_REPORT_PT_TAX_VENDAS_FATURAS_ISENTO.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_pt_tax_vendas_faturas_normal" model="account.report.line">
                                        <field name="name">Normal</field>
                                        <field name="code">TAX_REPORT_PT_TAX_VENDAS_FATURAS_NORMAL</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_vendas_faturas_normal_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_vendas_faturas_normal</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_tax_vendas_faturas_intermedia" model="account.report.line">
                                        <field name="name">Intermédia</field>
                                        <field name="code">TAX_REPORT_PT_TAX_VENDAS_FATURAS_INTERMEDIA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_vendas_faturas_intermedia_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_vendas_faturas_intermedia</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_tax_vendas_faturas_reduzida" model="account.report.line">
                                        <field name="name">Reduzida</field>
                                        <field name="code">TAX_REPORT_PT_TAX_VENDAS_FATURAS_REDUZIDA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_vendas_faturas_reduzida_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_vendas_faturas_reduzida</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_tax_vendas_faturas_isento" model="account.report.line">
                                        <field name="name">Isento</field>
                                        <field name="code">TAX_REPORT_PT_TAX_VENDAS_FATURAS_ISENTO</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_vendas_faturas_isento_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_vendas_faturas_isento</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_pt_tax_vendas_notas_credito" model="account.report.line">
                                <field name="name">Notas de crédito (Total IVA regularizado a favor da empresa)</field>
                                <field name="code">tax_report_pt_tax_vendas_notas_credito</field>
                                <field name="aggregation_formula">TAX_REPORT_PT_TAX_VENDAS_NOTAS_CREDITO_NORMAL.balance + TAX_REPORT_PT_TAX_VENDAS_NOTAS_CREDITO_INTERMEDIA.balance + TAX_REPORT_PT_TAX_VENDAS_NOTAS_CREDITO_REDUZIDA.balance + TAX_REPORT_PT_TAX_VENDAS_NOTAS_CREDITO_ISENTO.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_pt_tax_vendas_notas_credito_normal" model="account.report.line">
                                        <field name="name">Normal</field>
                                        <field name="code">TAX_REPORT_PT_TAX_VENDAS_NOTAS_CREDITO_NORMAL</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_vendas_notas_credito_normal_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_vendas_notas_credito_normal</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_tax_vendas_notas_credito_intermedia" model="account.report.line">
                                        <field name="name">Intermédia</field>
                                        <field name="code">TAX_REPORT_PT_TAX_VENDAS_NOTAS_CREDITO_INTERMEDIA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_vendas_notas_credito_intermedia_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_vendas_notas_credito_intermedia</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_tax_vendas_notas_credito_reduzida" model="account.report.line">
                                        <field name="name">Reduzida</field>
                                        <field name="code">TAX_REPORT_PT_TAX_VENDAS_NOTAS_CREDITO_REDUZIDA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_vendas_notas_credito_reduzida_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_vendas_notas_credito_reduzida</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_tax_vendas_notas_credito_isento" model="account.report.line">
                                        <field name="name">Isento</field>
                                        <field name="code">TAX_REPORT_PT_TAX_VENDAS_NOTAS_CREDITO_ISENTO</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_vendas_notas_credito_isento_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_vendas_notas_credito_isento</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                    <record id="tax_report_pt_tax_despesas" model="account.report.line">
                        <field name="name">Despesas</field>
                        <field name="children_ids">
                            <record id="tax_report_pt_tax_despesas_faturas" model="account.report.line">
                                <field name="name">Faturas (Total IVA dedutível)</field>
                                <field name="code">tax_report_pt_tax_despesas_faturas</field>
                                <field name="aggregation_formula">TAX_REPORT_PT_TAX_DESPESAS_FATURAS_NORMAL.balance + TAX_REPORT_PT_TAX_DESPESAS_FATURAS_INTERMEDIA.balance + TAX_REPORT_PT_TAX_DESPESAS_FATURAS_REDUZIDA.balance + TAX_REPORT_PT_TAX_DESPESAS_FATURAS_ISENTO.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_pt_tax_despesas_faturas_normal" model="account.report.line">
                                        <field name="name">Normal</field>
                                        <field name="code">TAX_REPORT_PT_TAX_DESPESAS_FATURAS_NORMAL</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_despesas_faturas_normal_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_despesas_faturas_normal</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_tax_despesas_faturas_intermedia" model="account.report.line">
                                        <field name="name">Intermédia</field>
                                        <field name="code">TAX_REPORT_PT_TAX_DESPESAS_FATURAS_INTERMEDIA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_despesas_faturas_intermedia_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_despesas_faturas_intermedia</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_tax_despesas_faturas_reduzida" model="account.report.line">
                                        <field name="name">Reduzida</field>
                                        <field name="code">TAX_REPORT_PT_TAX_DESPESAS_FATURAS_REDUZIDA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_despesas_faturas_reduzida_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_despesas_faturas_reduzida</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_tax_despesas_faturas_isento" model="account.report.line">
                                        <field name="name">Isento</field>
                                        <field name="code">TAX_REPORT_PT_TAX_DESPESAS_FATURAS_ISENTO</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_despesas_faturas_isento_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_despesas_faturas_isento</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                            <record id="tax_report_pt_tax_despesas_notas_credito" model="account.report.line">
                                <field name="name">Notas de crédito (Total IVA regularizado a favor do Estado)</field>
                                <field name="code">tax_report_pt_tax_despesas_notas_credito</field>
                                <field name="aggregation_formula">TAX_REPORT_PT_TAX_DESPESAS_NOTAS_CREDITO_NORMAL.balance + TAX_REPORT_PT_TAX_DESPESAS_NOTAS_CREDITO_INTERMEDIA.balance + TAX_REPORT_PT_TAX_DESPESAS_NOTAS_CREDITO_REDUZIDA.balance + TAX_REPORT_PT_TAX_DESPESAS_NOTAS_CREDITO_ISENTO.balance</field>
                                <field name="children_ids">
                                    <record id="tax_report_pt_tax_despesas_notas_credito_normal" model="account.report.line">
                                        <field name="name">Normal</field>
                                        <field name="code">TAX_REPORT_PT_TAX_DESPESAS_NOTAS_CREDITO_NORMAL</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_despesas_notas_credito_normal_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_despesas_notas_credito_normal</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_tax_despesas_notas_credito_intermedia" model="account.report.line">
                                        <field name="name">Intermédia</field>
                                        <field name="code">TAX_REPORT_PT_TAX_DESPESAS_NOTAS_CREDITO_INTERMEDIA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_despesas_notas_credito_intermedia_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_despesas_notas_credito_intermedia</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_tax_despesas_notas_credito_reduzida" model="account.report.line">
                                        <field name="name">Reduzida</field>
                                        <field name="code">TAX_REPORT_PT_TAX_DESPESAS_NOTAS_CREDITO_REDUZIDA</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_despesas_notas_credito_reduzida_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_despesas_notas_credito_reduzida</field>
                                            </record>
                                        </field>
                                    </record>
                                    <record id="tax_report_pt_tax_despesas_notas_credito_isento" model="account.report.line">
                                        <field name="name">Isento</field>
                                        <field name="code">TAX_REPORT_PT_TAX_DESPESAS_NOTAS_CREDITO_ISENTO</field>
                                        <field name="expression_ids">
                                            <record id="tax_report_pt_tax_despesas_notas_credito_isento_tag" model="account.report.expression">
                                                <field name="label">balance</field>
                                                <field name="engine">tax_tags</field>
                                                <field name="formula">tax_report_pt_tax_despesas_notas_credito_isento</field>
                                            </record>
                                        </field>
                                    </record>
                                </field>
                            </record>
                        </field>
                    </record>
                </field>
            </record>
            <record id="tax_report_pt_tax_total_pagar" model="account.report.line">
                <field name="name">TOTAL IVA A PAGAR</field>
                <field name="code">tax_report_pt_tax_total_pagar</field>
                <field name="expression_ids">
                    <record id="tax_report_pt_tax_total_pagar_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">tax_report_pt_tax_vendas_faturas.balance - tax_report_pt_tax_vendas_notas_credito.balance - tax_report_pt_tax_despesas_faturas.balance + tax_report_pt_tax_despesas_notas_credito.balance</field>
                        <field name="subformula">if_above(EUR(0))</field>
                    </record>
                </field>
            </record>
            <record id="tax_report_pt_tax_total_receber" model="account.report.line">
                <field name="name">TOTAL IVA A RECEBER</field>
                <field name="expression_ids">
                    <record id="tax_report_pt_tax_total_receber_formula" model="account.report.expression">
                        <field name="label">balance</field>
                        <field name="engine">aggregation</field>
                        <field name="formula">-tax_report_pt_tax_vendas_faturas.balance + tax_report_pt_tax_vendas_notas_credito.balance + tax_report_pt_tax_despesas_faturas.balance - tax_report_pt_tax_despesas_notas_credito.balance</field>
                        <field name="subformula">if_above(EUR(0))</field>
                    </record>
                </field>
            </record>
        </field>
    </record>
</odoo>

```

## File: data\l10n_pt_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>

    <record id="pt_chart_template" model="account.chart.template">
        <field name="name">Portugal - Template do Plano de Contas SNC</field>
        <field name="cash_account_code_prefix">11</field>
        <field name="bank_account_code_prefix">12</field>
        <field name="transfer_account_code_prefix">15</field>
        <field name="currency_id" ref="base.EUR"/>
        <field name="country_id" ref="base.pt"/>
    </record>

    <record id="chart_13" model="account.account.template">
      <field name="code">13</field>
      <field name="name">Outros depósitos bancários</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_1411" model="account.account.template">
      <field name="code">1411</field>
      <field name="name">Potencialmente favoráveis</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_1412" model="account.account.template">
      <field name="code">1412</field>
      <field name="name">Potencialmente desfavoráveis</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_1421" model="account.account.template">
      <field name="code">1421</field>
      <field name="name">Activos financeiros</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_1422" model="account.account.template">
      <field name="code">1422</field>
      <field name="name">Passivos financeiros</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_1431" model="account.account.template">
      <field name="code">1431</field>
      <field name="name">Outros activos financeiros</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_1432" model="account.account.template">
      <field name="code">1432</field>
      <field name="name">Outros passivos financeiros</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2111" model="account.account.template">
      <field name="code">2111</field>
      <field name="name">Clientes gerais</field>
      <field name="reconcile" eval="True"/>
      <field name="account_type">asset_receivable</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2112" model="account.account.template">
      <field name="code">2112</field>
      <field name="name">Clientes empresa mãe</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2113" model="account.account.template">
      <field name="code">2113</field>
      <field name="name">Clientes empresas subsidiárias</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2114" model="account.account.template">
      <field name="code">2114</field>
      <field name="name">Clientes empresas associadas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2115" model="account.account.template">
      <field name="code">2115</field>
      <field name="name">Clientes empreendimentos conjuntos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2116" model="account.account.template">
      <field name="code">2116</field>
      <field name="name">Clientes outras partes relacionadas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2117" model="account.account.template">
      <field name="code">2117</field>
      <field name="name">Clientes gerais (PoS)</field>
      <field name="reconcile" eval="True"/>
      <field name="account_type">asset_receivable</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2121" model="account.account.template">
      <field name="code">2121</field>
      <field name="name">Clientes gerais</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2122" model="account.account.template">
      <field name="code">2122</field>
      <field name="name">Clientes empresa mãe</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2123" model="account.account.template">
      <field name="code">2123</field>
      <field name="name">Clientes empresas subsidiárias</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2124" model="account.account.template">
      <field name="code">2124</field>
      <field name="name">Clientes empresas associadas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2125" model="account.account.template">
      <field name="code">2125</field>
      <field name="name">Clientes empreendimentos conjuntos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2126" model="account.account.template">
      <field name="code">2126</field>
      <field name="name">Clientes outras partes relacionadas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_218" model="account.account.template">
      <field name="code">218</field>
      <field name="name">Adiantamentos de clientes</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_219" model="account.account.template">
      <field name="code">219</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2211" model="account.account.template">
      <field name="code">2211</field>
      <field name="name">Fornecedores gerais</field>
      <field name="reconcile" eval="True"/>
      <field name="account_type">liability_payable</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2212" model="account.account.template">
      <field name="code">2212</field>
      <field name="name">Fornecedores empresa mãe</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2213" model="account.account.template">
      <field name="code">2213</field>
      <field name="name">Fornecedores empresas subsidiárias</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2214" model="account.account.template">
      <field name="code">2214</field>
      <field name="name">Fornecedores empresas associadas</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2215" model="account.account.template">
      <field name="code">2215</field>
      <field name="name">Fornecedores empreendimentos conjuntos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2216" model="account.account.template">
      <field name="code">2216</field>
      <field name="name">Fornecedores outras partes relacionadas</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2221" model="account.account.template">
      <field name="code">2221</field>
      <field name="name">Fornecedores gerais</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2222" model="account.account.template">
      <field name="code">2222</field>
      <field name="name">Fornecedores empresa mãe</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2223" model="account.account.template">
      <field name="code">2223</field>
      <field name="name">Fornecedores empresas subsidiárias</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2224" model="account.account.template">
      <field name="code">2224</field>
      <field name="name">Fornecedores empresas associadas</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2225" model="account.account.template">
      <field name="code">2225</field>
      <field name="name">Fornecedores empreendimentos conjuntos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2226" model="account.account.template">
      <field name="code">2226</field>
      <field name="name">Fornecedores outras partes relacionadas</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_225" model="account.account.template">
      <field name="code">225</field>
      <field name="name">Facturas em recepção e conferência</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_229" model="account.account.template">
      <field name="code">229</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2311" model="account.account.template">
      <field name="code">2311</field>
      <field name="name">Aos órgãos sociais</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2312" model="account.account.template">
      <field name="code">2312</field>
      <field name="name">Ao pessoal</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2321" model="account.account.template">
      <field name="code">2321</field>
      <field name="name">Aos órgãos sociais</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2322" model="account.account.template">
      <field name="code">2322</field>
      <field name="name">Ao pessoal</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2371" model="account.account.template">
      <field name="code">2371</field>
      <field name="name">Dos órgãos sociais</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2372" model="account.account.template">
      <field name="code">2372</field>
      <field name="name">Do pessoal</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2381" model="account.account.template">
      <field name="code">2381</field>
      <field name="name">Com os órgãos sociais</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_239" model="account.account.template">
      <field name="code">239</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2431" model="account.account.template">
      <field name="code">2431</field>
      <field name="name">Iva suportado</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2432" model="account.account.template">
      <field name="code">2432</field>
      <field name="name">Iva dedutível</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2433" model="account.account.template">
      <field name="code">2433</field>
      <field name="name">Iva liquidado</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2434" model="account.account.template">
      <field name="code">2434</field>
      <field name="name">Iva regularizações</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2435" model="account.account.template">
      <field name="code">2435</field>
      <field name="name">Iva apuramento</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2436" model="account.account.template">
      <field name="code">2436</field>
      <field name="name">Iva a pagar</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2437" model="account.account.template">
      <field name="code">2437</field>
      <field name="name">Iva a recuperar</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2438" model="account.account.template">
      <field name="code">2438</field>
      <field name="name">Iva reembolsos pedidos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2439" model="account.account.template">
      <field name="code">2439</field>
      <field name="name">Iva liquidações oficiosas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_244" model="account.account.template">
      <field name="code">244</field>
      <field name="name">Outros impostos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_245" model="account.account.template">
      <field name="code">245</field>
      <field name="name">Contribuições para a segurança social</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_246" model="account.account.template">
      <field name="code">246</field>
      <field name="name">Tributos das autarquias locais</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_248" model="account.account.template">
      <field name="code">248</field>
      <field name="name">Outras tributações</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2511" model="account.account.template">
      <field name="code">2511</field>
      <field name="name">Empréstimos bancários</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2512" model="account.account.template">
      <field name="code">2512</field>
      <field name="name">Descobertos bancários</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2513" model="account.account.template">
      <field name="code">2513</field>
      <field name="name">Locações financeiras</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2521" model="account.account.template">
      <field name="code">2521</field>
      <field name="name">Empréstimos por obrigações</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2531" model="account.account.template">
      <field name="code">2531</field>
      <field name="name">Empresa mãe suprimentos e outros mútuos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2532" model="account.account.template">
      <field name="code">2532</field>
      <field name="name">Outros participantes suprimentos e outros mútuos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_254" model="account.account.template">
      <field name="code">254</field>
      <field name="name">Subsidiárias, associadas e empreendimentos conjuntos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_258" model="account.account.template">
      <field name="code">258</field>
      <field name="name">Outros financiadores</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_261" model="account.account.template">
      <field name="code">261</field>
      <field name="name">Accionistas c. subscrição</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_262" model="account.account.template">
      <field name="code">262</field>
      <field name="name">Quotas não liberadas</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_263" model="account.account.template">
      <field name="code">263</field>
      <field name="name">Adiantamentos por conta de lucros</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_264" model="account.account.template">
      <field name="code">264</field>
      <field name="name">Resultados atribuídos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_265" model="account.account.template">
      <field name="code">265</field>
      <field name="name">Lucros disponíveis</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_266" model="account.account.template">
      <field name="code">266</field>
      <field name="name">Empréstimos concedidos empresa mãe</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_268" model="account.account.template">
      <field name="code">268</field>
      <field name="name">Outras operações</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_269" model="account.account.template">
      <field name="code">269</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2711" model="account.account.template">
      <field name="code">2711</field>
      <field name="name">Fornecedores de investimentos contas gerais</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2712" model="account.account.template">
      <field name="code">2712</field>
      <field name="name">Facturas em recepção e conferência</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2713" model="account.account.template">
      <field name="code">2713</field>
      <field name="name">Adiantamentos a fornecedores de investimentos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2721" model="account.account.template">
      <field name="code">2721</field>
      <field name="name">Devedores por acréscimo de rendimentos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2722" model="account.account.template">
      <field name="code">2722</field>
      <field name="name">Credores por acréscimos de gastos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_273" model="account.account.template">
      <field name="code">273</field>
      <field name="name">Benefícios pós emprego</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2741" model="account.account.template">
      <field name="code">2741</field>
      <field name="name">Activos por impostos diferidos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_2742" model="account.account.template">
      <field name="code">2742</field>
      <field name="name">Passivos por impostos diferidos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_275" model="account.account.template">
      <field name="code">275</field>
      <field name="name">Credores por subscrições não liberadas</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_276" model="account.account.template">
      <field name="code">276</field>
      <field name="name">Adiantamentos por conta de vendas</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_278" model="account.account.template">
      <field name="code">278</field>
      <field name="name">Outros devedores e credores</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_279" model="account.account.template">
      <field name="code">279</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_281" model="account.account.template">
      <field name="code">281</field>
      <field name="name">Gastos a reconhecer</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_282" model="account.account.template">
      <field name="code">282</field>
      <field name="name">Rendimentos a reconhecer</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_291" model="account.account.template">
      <field name="code">291</field>
      <field name="name">Impostos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_292" model="account.account.template">
      <field name="code">292</field>
      <field name="name">Garantias a clientes</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_293" model="account.account.template">
      <field name="code">293</field>
      <field name="name">Processos judiciais em curso</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_294" model="account.account.template">
      <field name="code">294</field>
      <field name="name">Acidentes de trabalho e doenças profissionais</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_295" model="account.account.template">
      <field name="code">295</field>
      <field name="name">Matérias ambientais</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_296" model="account.account.template">
      <field name="code">296</field>
      <field name="name">Contratos onerosos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_297" model="account.account.template">
      <field name="code">297</field>
      <field name="name">Reestruturação</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_298" model="account.account.template">
      <field name="code">298</field>
      <field name="name">Outras provisões</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_311" model="account.account.template">
      <field name="code">311</field>
      <field name="name">Mercadorias</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_312" model="account.account.template">
      <field name="code">312</field>
      <field name="name">Matérias primas, subsidiárias e de consumo</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_313" model="account.account.template">
      <field name="code">313</field>
      <field name="name">Activos biológicos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_317" model="account.account.template">
      <field name="code">317</field>
      <field name="name">Devoluções de compras</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_318" model="account.account.template">
      <field name="code">318</field>
      <field name="name">Descontos e abatimentos em compras</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_325" model="account.account.template">
      <field name="code">325</field>
      <field name="name">Mercadorias em trânsito</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_326" model="account.account.template">
      <field name="code">326</field>
      <field name="name">Mercadorias em poder de terceiros</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_329" model="account.account.template">
      <field name="code">329</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_331" model="account.account.template">
      <field name="code">331</field>
      <field name="name">Matérias primas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_332" model="account.account.template">
      <field name="code">332</field>
      <field name="name">Matérias subsidiárias</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_333" model="account.account.template">
      <field name="code">333</field>
      <field name="name">Embalagens</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_334" model="account.account.template">
      <field name="code">334</field>
      <field name="name">Materiais diversos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_335" model="account.account.template">
      <field name="code">335</field>
      <field name="name">Matérias em trânsito</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_339" model="account.account.template">
      <field name="code">339</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_346" model="account.account.template">
      <field name="code">346</field>
      <field name="name">Produtos em poder de terceiros</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_349" model="account.account.template">
      <field name="code">349</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_351" model="account.account.template">
      <field name="code">351</field>
      <field name="name">Subprodutos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_352" model="account.account.template">
      <field name="code">352</field>
      <field name="name">Desperdícios, resíduos e refugos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_359" model="account.account.template">
      <field name="code">359</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_36" model="account.account.template">
      <field name="code">36</field>
      <field name="name">Produtos e trabalhos em curso</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_3711" model="account.account.template">
      <field name="code">3711</field>
      <field name="name">Animais</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_3712" model="account.account.template">
      <field name="code">3712</field>
      <field name="name">Plantas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_3721" model="account.account.template">
      <field name="code">3721</field>
      <field name="name">Animais</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_3722" model="account.account.template">
      <field name="code">3722</field>
      <field name="name">Plantas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_382" model="account.account.template">
      <field name="code">382</field>
      <field name="name">Mercadorias</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_383" model="account.account.template">
      <field name="code">383</field>
      <field name="name">Matérias primas, subsidiárias e de consumo</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_384" model="account.account.template">
      <field name="code">384</field>
      <field name="name">Produtos acabados e intermédios</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_385" model="account.account.template">
      <field name="code">385</field>
      <field name="name">Subprodutos, desperdícios, resíduos e refugos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_386" model="account.account.template">
      <field name="code">386</field>
      <field name="name">Produtos e trabalhos em curso</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_387" model="account.account.template">
      <field name="code">387</field>
      <field name="name">Activos biológicos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_39" model="account.account.template">
      <field name="code">39</field>
      <field name="name">Adiantamentos por conta de compras</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_4111" model="account.account.template">
      <field name="code">4111</field>
      <field name="name">Participações de capital método da equiv. patrimonial</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_4112" model="account.account.template">
      <field name="code">4112</field>
      <field name="name">Participações de capital outros métodos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_4113" model="account.account.template">
      <field name="code">4113</field>
      <field name="name">Empréstimos concedidos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_4121" model="account.account.template">
      <field name="code">4121</field>
      <field name="name">Participações de capital método da equiv. patrimonial</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_4122" model="account.account.template">
      <field name="code">4122</field>
      <field name="name">Participações de capital outros métodos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_4123" model="account.account.template">
      <field name="code">4123</field>
      <field name="name">Empréstimos concedidos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_4131" model="account.account.template">
      <field name="code">4131</field>
      <field name="name">Participações de capital método da equiv. patrimonial</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_4132" model="account.account.template">
      <field name="code">4132</field>
      <field name="name">Participações de capital outros métodos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_4133" model="account.account.template">
      <field name="code">4133</field>
      <field name="name">Empréstimos concedidos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_4141" model="account.account.template">
      <field name="code">4141</field>
      <field name="name">Participações de capital</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_4142" model="account.account.template">
      <field name="code">4142</field>
      <field name="name">Empréstimos concedidos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_4151" model="account.account.template">
      <field name="code">4151</field>
      <field name="name">Detidos até à maturidade</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_4158" model="account.account.template">
      <field name="code">4158</field>
      <field name="name">Acções da sgm (6500x1,00)</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_419" model="account.account.template">
      <field name="code">419</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_421" model="account.account.template">
      <field name="code">421</field>
      <field name="name">Terrenos e recursos naturais</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_422" model="account.account.template">
      <field name="code">422</field>
      <field name="name">Edifícios e outras construções</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_426" model="account.account.template">
      <field name="code">426</field>
      <field name="name">Outras propriedades de investimento</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_428" model="account.account.template">
      <field name="code">428</field>
      <field name="name">Depreciações acumuladas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_429" model="account.account.template">
      <field name="code">429</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_431" model="account.account.template">
      <field name="code">431</field>
      <field name="name">Terrenos e recursos naturais</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_432" model="account.account.template">
      <field name="code">432</field>
      <field name="name">Edifícios e outras construções</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_433" model="account.account.template">
      <field name="code">433</field>
      <field name="name">Equipamento básico</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_434" model="account.account.template">
      <field name="code">434</field>
      <field name="name">Equipamento de transporte</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_435" model="account.account.template">
      <field name="code">435</field>
      <field name="name">Equipamento administrativo</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_436" model="account.account.template">
      <field name="code">436</field>
      <field name="name">Equipamentos biológicos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_437" model="account.account.template">
      <field name="code">437</field>
      <field name="name">Outros activos fixos tangíveis</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_438" model="account.account.template">
      <field name="code">438</field>
      <field name="name">Depreciações acumuladas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_439" model="account.account.template">
      <field name="code">439</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_441" model="account.account.template">
      <field name="code">441</field>
      <field name="name">Goodwill</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_442" model="account.account.template">
      <field name="code">442</field>
      <field name="name">Projectos de desenvolvimento</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_443" model="account.account.template">
      <field name="code">443</field>
      <field name="name">Programas de computador</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_444" model="account.account.template">
      <field name="code">444</field>
      <field name="name">Propriedade industrial</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_446" model="account.account.template">
      <field name="code">446</field>
      <field name="name">Outros activos intangíveis</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_448" model="account.account.template">
      <field name="code">448</field>
      <field name="name">Depreciações acumuladas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_449" model="account.account.template">
      <field name="code">449</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_451" model="account.account.template">
      <field name="code">451</field>
      <field name="name">Investimentos financeiros em curso</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_452" model="account.account.template">
      <field name="code">452</field>
      <field name="name">Propriedades de investimento em curso</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_453" model="account.account.template">
      <field name="code">453</field>
      <field name="name">Activos fixos tangíveis em curso</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_454" model="account.account.template">
      <field name="code">454</field>
      <field name="name">Activos intangíveis em curso</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_455" model="account.account.template">
      <field name="code">455</field>
      <field name="name">Adiantamentos por conta de investimentos</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_459" model="account.account.template">
      <field name="code">459</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_469" model="account.account.template">
      <field name="code">469</field>
      <field name="name">Perdas por imparidade acumuladas</field>
      <field name="account_type">asset_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_51" model="account.account.template">
      <field name="code">51</field>
      <field name="name">Capital</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_521" model="account.account.template">
      <field name="code">521</field>
      <field name="name">Valor nominal</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_522" model="account.account.template">
      <field name="code">522</field>
      <field name="name">Descontos e prémios</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_53" model="account.account.template">
      <field name="code">53</field>
      <field name="name">Outros instrumentos de capital próprio</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_54" model="account.account.template">
      <field name="code">54</field>
      <field name="name">Prémios de emissão</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_551" model="account.account.template">
      <field name="code">551</field>
      <field name="name">Reservas legais</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_552" model="account.account.template">
      <field name="code">552</field>
      <field name="name">Outras reservas</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_56" model="account.account.template">
      <field name="code">56</field>
      <field name="name">Resultados transitados</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_5711" model="account.account.template">
      <field name="code">5711</field>
      <field name="name">Ajustamentos de transição</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_5712" model="account.account.template">
      <field name="code">5712</field>
      <field name="name">Lucros não atribuídos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_5713" model="account.account.template">
      <field name="code">5713</field>
      <field name="name">Decorrentes de outras variações nos capitais próprios d</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_579" model="account.account.template">
      <field name="code">579</field>
      <field name="name">Outros</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_5811" model="account.account.template">
      <field name="code">5811</field>
      <field name="name">Antes de imposto sobre o rendimento</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_5812" model="account.account.template">
      <field name="code">5812</field>
      <field name="name">Impostos diferidos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_5891" model="account.account.template">
      <field name="code">5891</field>
      <field name="name">Antes de imposto sobre o rendimento</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_5892" model="account.account.template">
      <field name="code">5892</field>
      <field name="name">Impostos diferidos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_591" model="account.account.template">
      <field name="code">591</field>
      <field name="name">Diferenças de conversão de demonstrações financeiras</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_592" model="account.account.template">
      <field name="code">592</field>
      <field name="name">Ajustamentos por impostos diferidos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_593" model="account.account.template">
      <field name="code">593</field>
      <field name="name">Subsídios</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_594" model="account.account.template">
      <field name="code">594</field>
      <field name="name">Doações</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_599" model="account.account.template">
      <field name="code">599</field>
      <field name="name">Outras</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_611" model="account.account.template">
      <field name="code">611</field>
      <field name="name">Mercadorias</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_612" model="account.account.template">
      <field name="code">612</field>
      <field name="name">Matérias primas, subsidiárias e de consumo</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_613" model="account.account.template">
      <field name="code">613</field>
      <field name="name">Activos biológicos (compras)</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_621" model="account.account.template">
      <field name="code">621</field>
      <field name="name">Subcontratos</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6221" model="account.account.template">
      <field name="code">6221</field>
      <field name="name">Trabalhos especializados</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6222" model="account.account.template">
      <field name="code">6222</field>
      <field name="name">Publicidade e propaganda</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6223" model="account.account.template">
      <field name="code">6223</field>
      <field name="name">Vigilância e segurança</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6224" model="account.account.template">
      <field name="code">6224</field>
      <field name="name">Honorários</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6225" model="account.account.template">
      <field name="code">6225</field>
      <field name="name">Comissões</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6226" model="account.account.template">
      <field name="code">6226</field>
      <field name="name">Conservação e reparação</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6228" model="account.account.template">
      <field name="code">6228</field>
      <field name="name">Outros</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6231" model="account.account.template">
      <field name="code">6231</field>
      <field name="name">Ferramentas e utensílios de desgaste rápido</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6232" model="account.account.template">
      <field name="code">6232</field>
      <field name="name">Livros de documentação técnica</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6233" model="account.account.template">
      <field name="code">6233</field>
      <field name="name">Material de escritório</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6234" model="account.account.template">
      <field name="code">6234</field>
      <field name="name">Artigos de oferta</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6238" model="account.account.template">
      <field name="code">6238</field>
      <field name="name">Outros</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6241" model="account.account.template">
      <field name="code">6241</field>
      <field name="name">Electricidade</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6242" model="account.account.template">
      <field name="code">6242</field>
      <field name="name">Combustíveis</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6243" model="account.account.template">
      <field name="code">6243</field>
      <field name="name">Água</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6248" model="account.account.template">
      <field name="code">6248</field>
      <field name="name">Outros</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6251" model="account.account.template">
      <field name="code">6251</field>
      <field name="name">Deslocações e estadas</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6252" model="account.account.template">
      <field name="code">6252</field>
      <field name="name">Transporte de pessoal</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6253" model="account.account.template">
      <field name="code">6253</field>
      <field name="name">Transportes de mercadorias</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6258" model="account.account.template">
      <field name="code">6258</field>
      <field name="name">Outros</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6261" model="account.account.template">
      <field name="code">6261</field>
      <field name="name">Rendas e alugueres</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6262" model="account.account.template">
      <field name="code">6262</field>
      <field name="name">Comunicação</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6263" model="account.account.template">
      <field name="code">6263</field>
      <field name="name">Seguros</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6264" model="account.account.template">
      <field name="code">6264</field>
      <field name="name">Royalties</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6265" model="account.account.template">
      <field name="code">6265</field>
      <field name="name">Contencioso e notariado</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6266" model="account.account.template">
      <field name="code">6266</field>
      <field name="name">Despesas de representação</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6267" model="account.account.template">
      <field name="code">6267</field>
      <field name="name">Limpeza, higiene e conforto</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6268" model="account.account.template">
      <field name="code">6268</field>
      <field name="name">Outros serviços</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_631" model="account.account.template">
      <field name="code">631</field>
      <field name="name">Remunerações dos órgãos sociais</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_632" model="account.account.template">
      <field name="code">632</field>
      <field name="name">Remunerações do pessoal</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6331" model="account.account.template">
      <field name="code">6331</field>
      <field name="name">Prémios para pensões</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6332" model="account.account.template">
      <field name="code">6332</field>
      <field name="name">Outros benefícios</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_634" model="account.account.template">
      <field name="code">634</field>
      <field name="name">Indemnizações</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_635" model="account.account.template">
      <field name="code">635</field>
      <field name="name">Encargos sobre remunerações</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_636" model="account.account.template">
      <field name="code">636</field>
      <field name="name">Seguros de acidentes no trabalho e doenças profissionais</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_637" model="account.account.template">
      <field name="code">637</field>
      <field name="name">Gastos de acção social</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_638" model="account.account.template">
      <field name="code">638</field>
      <field name="name">Outros gastos com o pessoal</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_641" model="account.account.template">
      <field name="code">641</field>
      <field name="name">Propriedades de investimento</field>
      <field name="account_type">expense_depreciation</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_642" model="account.account.template">
      <field name="code">642</field>
      <field name="name">Activos fixos tangíveis</field>
      <field name="account_type">expense_depreciation</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_643" model="account.account.template">
      <field name="code">643</field>
      <field name="name">Activos intangíveis</field>
      <field name="account_type">expense_depreciation</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6511" model="account.account.template">
      <field name="code">6511</field>
      <field name="name">Clientes</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6512" model="account.account.template">
      <field name="code">6512</field>
      <field name="name">Outros devedores</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_652" model="account.account.template">
      <field name="code">652</field>
      <field name="name">Em inventários</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_653" model="account.account.template">
      <field name="code">653</field>
      <field name="name">Em investimentos financeiros</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_654" model="account.account.template">
      <field name="code">654</field>
      <field name="name">Em propriedades de investimento</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_655" model="account.account.template">
      <field name="code">655</field>
      <field name="name">Em activos fixos tangíveis</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_656" model="account.account.template">
      <field name="code">656</field>
      <field name="name">Em activos intangíveis</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_657" model="account.account.template">
      <field name="code">657</field>
      <field name="name">Em investimentos em curso</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_658" model="account.account.template">
      <field name="code">658</field>
      <field name="name">Em activos não correntes detidos para venda</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_661" model="account.account.template">
      <field name="code">661</field>
      <field name="name">Em instrumentos financeiros</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_662" model="account.account.template">
      <field name="code">662</field>
      <field name="name">Em investimentos financeiros</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_663" model="account.account.template">
      <field name="code">663</field>
      <field name="name">Em propriedades de investimento</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_664" model="account.account.template">
      <field name="code">664</field>
      <field name="name">Em activos biológicos</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_671" model="account.account.template">
      <field name="code">671</field>
      <field name="name">Impostos</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_672" model="account.account.template">
      <field name="code">672</field>
      <field name="name">Garantias a clientes</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_673" model="account.account.template">
      <field name="code">673</field>
      <field name="name">Processos judiciais em curso</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_674" model="account.account.template">
      <field name="code">674</field>
      <field name="name">Acidentes de trabalho e doenças profissionais</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_675" model="account.account.template">
      <field name="code">675</field>
      <field name="name">Matérias ambientais</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_676" model="account.account.template">
      <field name="code">676</field>
      <field name="name">Contratos onerosos</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_677" model="account.account.template">
      <field name="code">677</field>
      <field name="name">Reestruturação</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_678" model="account.account.template">
      <field name="code">678</field>
      <field name="name">Outras provisões</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6811" model="account.account.template">
      <field name="code">6811</field>
      <field name="name">Impostos directos</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6812" model="account.account.template">
      <field name="code">6812</field>
      <field name="name">Impostos indirectos</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6813" model="account.account.template">
      <field name="code">6813</field>
      <field name="name">Taxas</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_682" model="account.account.template">
      <field name="code">682</field>
      <field name="name">Descontos de pronto pagamento concedidos</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_683" model="account.account.template">
      <field name="code">683</field>
      <field name="name">Dívidas incobráveis</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6841" model="account.account.template">
      <field name="code">6841</field>
      <field name="name">Sinistros</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6842" model="account.account.template">
      <field name="code">6842</field>
      <field name="name">Quebras</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6848" model="account.account.template">
      <field name="code">6848</field>
      <field name="name">Outras perdas</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6851" model="account.account.template">
      <field name="code">6851</field>
      <field name="name">Cobertura de prejuízos</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6852" model="account.account.template">
      <field name="code">6852</field>
      <field name="name">Aplicação do método da equivalência patrimonial</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6853" model="account.account.template">
      <field name="code">6853</field>
      <field name="name">Alienações</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6858" model="account.account.template">
      <field name="code">6858</field>
      <field name="name">Outros gastos e perdas</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6861" model="account.account.template">
      <field name="code">6861</field>
      <field name="name">Cobertura de prejuízos</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6862" model="account.account.template">
      <field name="code">6862</field>
      <field name="name">Alienações</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6868" model="account.account.template">
      <field name="code">6868</field>
      <field name="name">Outros gastos e perdas</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6871" model="account.account.template">
      <field name="code">6871</field>
      <field name="name">Alienações</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6872" model="account.account.template">
      <field name="code">6872</field>
      <field name="name">Sinistros</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6873" model="account.account.template">
      <field name="code">6873</field>
      <field name="name">Abates</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6874" model="account.account.template">
      <field name="code">6874</field>
      <field name="name">Gastos em propriedades de investimento</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6878" model="account.account.template">
      <field name="code">6878</field>
      <field name="name">Outros gastos e perdas</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6881" model="account.account.template">
      <field name="code">6881</field>
      <field name="name">Correcções relativas a períodos anteriores</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6882" model="account.account.template">
      <field name="code">6882</field>
      <field name="name">Donativos</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6883" model="account.account.template">
      <field name="code">6883</field>
      <field name="name">Quotizações</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6884" model="account.account.template">
      <field name="code">6884</field>
      <field name="name">Ofertas e amostras de inventários</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6885" model="account.account.template">
      <field name="code">6885</field>
      <field name="name">Insuficiência da estimativa para impostos</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6886" model="account.account.template">
      <field name="code">6886</field>
      <field name="name">Perdas em instrumentos financeiros</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6888" model="account.account.template">
      <field name="code">6888</field>
      <field name="name">Outros não especificados</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6911" model="account.account.template">
      <field name="code">6911</field>
      <field name="name">Juros de financiamento obtidos</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6918" model="account.account.template">
      <field name="code">6918</field>
      <field name="name">Outros juros</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_692" model="account.account.template">
      <field name="code">692</field>
      <field name="name">Diferenças de câmbio desfavoráveis</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6921" model="account.account.template">
      <field name="code">6921</field>
      <field name="name">Relativos a financiamentos obtidos</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6928" model="account.account.template">
      <field name="code">6928</field>
      <field name="name">Outras</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6981" model="account.account.template">
      <field name="code">6981</field>
      <field name="name">Relativos a financiamentos obtidos</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_6988" model="account.account.template">
      <field name="code">6988</field>
      <field name="name">Outros</field>
      <field name="account_type">expense</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_711" model="account.account.template">
      <field name="code">711</field>
      <field name="name">Mercadoria</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_712" model="account.account.template">
      <field name="code">712</field>
      <field name="name">Produtos acabados e intermédios</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_713" model="account.account.template">
      <field name="code">713</field>
      <field name="name">Subprodutos, desperdícios, resíduos e refugos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_714" model="account.account.template">
      <field name="code">714</field>
      <field name="name">Activos biológicos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_716" model="account.account.template">
      <field name="code">716</field>
      <field name="name">Iva das vendas com imposto incluído</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_717" model="account.account.template">
      <field name="code">717</field>
      <field name="name">Devoluções de vendas</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_718" model="account.account.template">
      <field name="code">718</field>
      <field name="name">Descontos e abatimentos em vendas</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_721" model="account.account.template">
      <field name="code">721</field>
      <field name="name">Serviço a</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_722" model="account.account.template">
      <field name="code">722</field>
      <field name="name">Serviço b</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_725" model="account.account.template">
      <field name="code">725</field>
      <field name="name">Serviços secundários</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_726" model="account.account.template">
      <field name="code">726</field>
      <field name="name">Iva dos serviços com imposto incluído</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_728" model="account.account.template">
      <field name="code">728</field>
      <field name="name">Descontos e abatimentos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_731" model="account.account.template">
      <field name="code">731</field>
      <field name="name">Produtos acabados e intermédios</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_732" model="account.account.template">
      <field name="code">732</field>
      <field name="name">Subprodutos, desperdícios, resíduos e refugos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_733" model="account.account.template">
      <field name="code">733</field>
      <field name="name">Produtos e trabalhos em curso</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_734" model="account.account.template">
      <field name="code">734</field>
      <field name="name">Activos biológicos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_741" model="account.account.template">
      <field name="code">741</field>
      <field name="name">Activos fixos tangíveis</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_742" model="account.account.template">
      <field name="code">742</field>
      <field name="name">Activos intangíveis</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_743" model="account.account.template">
      <field name="code">743</field>
      <field name="name">Propriedades de investimento</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_744" model="account.account.template">
      <field name="code">744</field>
      <field name="name">Activos por gastos diferidos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_751" model="account.account.template">
      <field name="code">751</field>
      <field name="name">Subsídios do estado e outros entes públicos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_752" model="account.account.template">
      <field name="code">752</field>
      <field name="name">Subsídios de outras entidades</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7611" model="account.account.template">
      <field name="code">7611</field>
      <field name="name">Propriedades de investimento</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7612" model="account.account.template">
      <field name="code">7612</field>
      <field name="name">Activos fixos tangíveis</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7613" model="account.account.template">
      <field name="code">7613</field>
      <field name="name">Activos intangíveis</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_76211" model="account.account.template">
      <field name="code">76211</field>
      <field name="name">Clientes</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_76212" model="account.account.template">
      <field name="code">76212</field>
      <field name="name">Outros devedores</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7622" model="account.account.template">
      <field name="code">7622</field>
      <field name="name">Em inventários</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7623" model="account.account.template">
      <field name="code">7623</field>
      <field name="name">Em investimentos financeiros</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7624" model="account.account.template">
      <field name="code">7624</field>
      <field name="name">Em propriedades de investimento</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7625" model="account.account.template">
      <field name="code">7625</field>
      <field name="name">Em activos fixos tangíveis</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7626" model="account.account.template">
      <field name="code">7626</field>
      <field name="name">Em activos intangíveis</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7627" model="account.account.template">
      <field name="code">7627</field>
      <field name="name">Em investimentos em curso</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7628" model="account.account.template">
      <field name="code">7628</field>
      <field name="name">Em activos não correntes detidos para venda</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7631" model="account.account.template">
      <field name="code">7631</field>
      <field name="name">Impostos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7632" model="account.account.template">
      <field name="code">7632</field>
      <field name="name">Garantias a clientes</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7633" model="account.account.template">
      <field name="code">7633</field>
      <field name="name">Processos judiciais em curso</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7634" model="account.account.template">
      <field name="code">7634</field>
      <field name="name">Acidentes no trabalho e doenças profissionais</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7635" model="account.account.template">
      <field name="code">7635</field>
      <field name="name">Matérias ambientais</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7636" model="account.account.template">
      <field name="code">7636</field>
      <field name="name">Contratos onerosos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7637" model="account.account.template">
      <field name="code">7637</field>
      <field name="name">Reestruturação</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7638" model="account.account.template">
      <field name="code">7638</field>
      <field name="name">Outras provisões</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_771" model="account.account.template">
      <field name="code">771</field>
      <field name="name">Em instrumentos financeiros</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_772" model="account.account.template">
      <field name="code">772</field>
      <field name="name">Em investimentos financeiros</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_773" model="account.account.template">
      <field name="code">773</field>
      <field name="name">Em propriedades de investimento</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_774" model="account.account.template">
      <field name="code">774</field>
      <field name="name">Em activos biológicos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7811" model="account.account.template">
      <field name="code">7811</field>
      <field name="name">Serviços sociais</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7812" model="account.account.template">
      <field name="code">7812</field>
      <field name="name">Aluguer de equipamento</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7813" model="account.account.template">
      <field name="code">7813</field>
      <field name="name">Estudos, projectos e assistência tecnológica</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7814" model="account.account.template">
      <field name="code">7814</field>
      <field name="name">Royalties</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7815" model="account.account.template">
      <field name="code">7815</field>
      <field name="name">Desempenho de cargos sociais noutras empresas</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7816" model="account.account.template">
      <field name="code">7816</field>
      <field name="name">Outros rendimentos suplementares</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_783" model="account.account.template">
      <field name="code">783</field>
      <field name="name">Recuperação de dívidas a receber</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7841" model="account.account.template">
      <field name="code">7841</field>
      <field name="name">Sinistros</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7842" model="account.account.template">
      <field name="code">7842</field>
      <field name="name">Sobras</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7848" model="account.account.template">
      <field name="code">7848</field>
      <field name="name">Outros ganhos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7851" model="account.account.template">
      <field name="code">7851</field>
      <field name="name">Aplicação do método da equivalência patrimonial</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7852" model="account.account.template">
      <field name="code">7852</field>
      <field name="name">Alienações</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7858" model="account.account.template">
      <field name="code">7858</field>
      <field name="name">Outros rendimentos e ganhos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7861" model="account.account.template">
      <field name="code">7861</field>
      <field name="name">Diferenças de câmbio favoráveis</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7862" model="account.account.template">
      <field name="code">7862</field>
      <field name="name">Alienações</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7868" model="account.account.template">
      <field name="code">7868</field>
      <field name="name">Outros rendimentos e ganhos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7871" model="account.account.template">
      <field name="code">7871</field>
      <field name="name">Alienações</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7872" model="account.account.template">
      <field name="code">7872</field>
      <field name="name">Sinistros</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7873" model="account.account.template">
      <field name="code">7873</field>
      <field name="name">Rendas e outros rendimentos em propriedades de investimento</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7878" model="account.account.template">
      <field name="code">7878</field>
      <field name="name">Outros rendimentos e ganhos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7881" model="account.account.template">
      <field name="code">7881</field>
      <field name="name">Correcções relativas a períodos anteriores</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7882" model="account.account.template">
      <field name="code">7882</field>
      <field name="name">Excesso da estimativa para impostos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7883" model="account.account.template">
      <field name="code">7883</field>
      <field name="name">Imputação de subsídios para investimentos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7884" model="account.account.template">
      <field name="code">7884</field>
      <field name="name">Ganhos em outros instrumentos financeiros</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7885" model="account.account.template">
      <field name="code">7885</field>
      <field name="name">Restituição de impostos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7888" model="account.account.template">
      <field name="code">7888</field>
      <field name="name">Outros não especificados</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7911" model="account.account.template">
      <field name="code">7911</field>
      <field name="name">De depósitos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7912" model="account.account.template">
      <field name="code">7912</field>
      <field name="name">De outras aplicações de meios financeiros líquidos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7913" model="account.account.template">
      <field name="code">7913</field>
      <field name="name">De financiamentos concedidos a associadas e emp. conjun</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7914" model="account.account.template">
      <field name="code">7914</field>
      <field name="name">De financiamentos concedidos a subsidiárias</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7915" model="account.account.template">
      <field name="code">7915</field>
      <field name="name">De financiamentos obtidos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7918" model="account.account.template">
      <field name="code">7918</field>
      <field name="name">De outros financiamentos obtidos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7921" model="account.account.template">
      <field name="code">7921</field>
      <field name="name">De aplicações de meios financeiros líquidos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7922" model="account.account.template">
      <field name="code">7922</field>
      <field name="name">De associadas e empreendimentos conjuntos</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7923" model="account.account.template">
      <field name="code">7923</field>
      <field name="name">De subsidiárias</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_7928" model="account.account.template">
      <field name="code">7928</field>
      <field name="name">Outras</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_798" model="account.account.template">
      <field name="code">798</field>
      <field name="name">Outros rendimentos similares</field>
      <field name="account_type">income</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_811" model="account.account.template">
      <field name="code">811</field>
      <field name="name">Resultado antes de impostos</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_8121" model="account.account.template">
      <field name="code">8121</field>
      <field name="name">Imposto estimado para o período</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_8122" model="account.account.template">
      <field name="code">8122</field>
      <field name="name">Imposto diferido</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_818" model="account.account.template">
      <field name="code">818</field>
      <field name="name">Resultado líquido</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    <record id="chart_89" model="account.account.template">
      <field name="code">89</field>
      <field name="name">Dividendos antecipados</field>
      <field name="account_type">liability_current</field>
      <field name="chart_template_id" ref="pt_chart_template"/>
    </record>

    </data>
</odoo>

```

## File: migrations\1.1\pre-migrate.py

```python
def migrate(cr, version):
    # Set noupdate property of "account.tax.template" records to False
    cr.execute(
        """UPDATE ir_model_data
              SET noupdate=false
            WHERE module='l10n_pt'
              AND model='account.tax.template'
        """
    )

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106"><defs><mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse"><path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill:#fff;fill-rule:evenodd"/></mask><mask id="b" x="6.15" y="8.99" width="48.56" height="32.36" maskUnits="userSpaceOnUse"><rect x="6.15" y="10.78" width="48.45" height="28.75" rx="1" style="fill:#fff"/></mask><symbol id="c" viewBox="0 0 106 106"><g style="mask:url(#a)"><path d="M0,0H106V106H0Z" style="fill:#5a5a64;fill-rule:evenodd"/><path d="M6.06,1.51H98.43q6.06,0,7.57,3V0H0V4.54Q1.52,1.51,6.06,1.51Z" style="fill:#fff;fill-opacity:0.382999986410141;fill-rule:evenodd"/><path d="M6.06,104.49H98.43q6.06,0,7.57-4.55V106H0V99.94Q1.52,104.49,6.06,104.49Z" style="fill-opacity:0.382999986410141;fill-rule:evenodd"/><path d="M70.38,104.49H6.06C3,104.49,0,103,0,98.43V61.28L28.77,19.69H59.06a77.33,77.33,0,0,0,21.2,13.87c.07,11.31.07,4.86,0,16.17h3.12l.21,36.82Z" style="fill:#393939;fill-rule:evenodd;opacity:0.324000000953674;isolation:isolate"/><g style="opacity:0.30000000000000004"><path d="M68.77,58.54H76c.76,0,1,.12,1,.46v2.45c0,.31-.24.43-.93.43H61.44c-.66,0-.92-.12-.92-.42,0-.83,0-1.67,0-2.51,0-.29.26-.4.92-.41Z"/><path d="M64.33,77.42c.42.39.76.66,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,4.25,4.25,0,0,1-.48-.47c-.14-.15-.26-.31-.49-.6-.32.37-.54.66-.79.91-.53.53-1.08.58-1.5.15s-.36-.94.15-1.45c.26-.26.54-.5.91-.83-.38-.34-.72-.61-1-.91a.9.9,0,0,1,0-1.36.91.91,0,0,1,1.36,0c.29.28.54.6.93,1A12.1,12.1,0,0,1,64,75.18a.91.91,0,0,1,1.36,0,.87.87,0,0,1,0,1.31C65.07,76.79,64.73,77.06,64.33,77.42Z"/><path d="M62.13,66.9c0-.47,0-.88,0-1.28a.92.92,0,0,1,.92-1,.91.91,0,0,1,1,1c0,.41,0,.81,0,1.3h1.14a1.16,1.16,0,0,1,1.22,1c0,.55-.42.85-1.18.86H64.12c0,.49,0,.91,0,1.34a.94.94,0,1,1-1.88,0c0-.41,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88C61.3,66.89,61.68,66.9,62.13,66.9Z"/><path d="M74.31,76H72.23c-.67,0-1-.34-1-.93a.89.89,0,0,1,1-1q2.18,0,4.35,0a1,1,0,1,1,0,1.91c-.74,0-1.47,0-2.21,0Z"/><path d="M74.28,68.61c-.71,0-1.43,0-2.14,0a.86.86,0,0,1-1-.9.85.85,0,0,1,.92-1c1.5,0,3,0,4.48,0a.93.93,0,0,1,1,1,.91.91,0,0,1-1,.91c-.75,0-1.51,0-2.27,0Z"/><path d="M74.36,78.09c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.57-.38.93-1,.94H72.28c-.75,0-1.09-.32-1.09-.94s.37-1,1.09-1,1.39,0,2.08,0Z"/><path d="M81.29,90.55H56.14a4,4,0,0,1-4-4V53.73a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V86.55A4,4,0,0,1,81.29,90.55ZM56.14,53.73V86.55H81.29V53.73Z"/><path d="M43.49,83.26H31.8V25.71H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V34.8c-4.55-3-16.66-12.11-19.69-13.63H30.29a2.68,2.68,0,0,0-3,3V84.77a2.68,2.68,0,0,0,3,3H48.45V83.26ZM60.57,25.71l15.14,10.6H60.57Z"/></g><path d="M60.57,18.68H30.29a2.68,2.68,0,0,0-3,3V82.28a2.68,2.68,0,0,0,3,3H48.45V80.77H31.8V23.22H56v10.6q0,4.55,4.54,4.55H75.71v5.78h4.55V32.31C75.71,29.28,63.6,20.2,60.57,18.68Zm0,15.14V23.22l15.14,10.6Z" style="fill:#a8a9ab"/><path d="M68.77,55.78H76c.76,0,1,.13,1,.53v2.85c0,.37-.24.5-.93.5q-7.3,0-14.61,0c-.66,0-.92-.14-.92-.48,0-1,0-2,0-2.93,0-.34.26-.47.92-.47Z" style="fill:#a8a9ab"/><path d="M64.33,76.53c.42.38.76.65,1,1a.89.89,0,0,1,0,1.31.92.92,0,0,1-1.32,0,5.44,5.44,0,0,1-.48-.48c-.14-.14-.26-.31-.49-.59-.32.36-.54.65-.79.91-.53.53-1.08.57-1.5.14s-.36-.94.15-1.45c.26-.26.54-.49.91-.82-.38-.35-.72-.61-1-.92a.9.9,0,0,1,0-1.36.92.92,0,0,1,1.36,0c.29.28.54.61.93,1A13.78,13.78,0,0,1,64,74.28a.91.91,0,0,1,1.36,0,.88.88,0,0,1,0,1.32C65.07,75.89,64.73,76.16,64.33,76.53Z" style="fill:#a8a9ab"/><path d="M62.13,65.88c0-.48,0-.88,0-1.29a1,1,0,1,1,1.91,0c0,.4,0,.81,0,1.3h1.14a1.15,1.15,0,0,1,1.22,1c0,.54-.42.85-1.18.85H64.12c0,.49,0,.92,0,1.34a.94.94,0,1,1-1.88,0c0-.4,0-.81,0-1.3H60.92a.94.94,0,1,1,0-1.88Z" style="fill:#a8a9ab"/><path d="M74.31,75.11c-.69,0-1.38,0-2.08,0s-1-.35-1-.94a.89.89,0,0,1,1-1q2.18,0,4.35,0a.91.91,0,0,1,1,1,.93.93,0,0,1-1,1c-.74,0-1.47,0-2.21,0Z" style="fill:#a8a9ab"/><path d="M74.28,67.76H72.14a.87.87,0,0,1-1-.9.84.84,0,0,1,.92-1c1.5,0,3,0,4.48,0a.94.94,0,0,1,1,1,.91.91,0,0,1-1,.91H74.28Z" style="fill:#a8a9ab"/><path d="M74.36,77.2c.72,0,1.44,0,2.15,0a1,1,0,0,1,1,1c0,.56-.38.93-1,.93q-2.12,0-4.23,0c-.75,0-1.09-.32-1.09-.94s.37-.94,1.09-1,1.39,0,2.08,0Z" style="fill:#a8a9ab"/><path d="M81.29,88.06H56.14a4,4,0,0,1-4-4V51.24a4,4,0,0,1,4-4H81.29a4,4,0,0,1,4,4V84.06A4,4,0,0,1,81.29,88.06ZM56.14,51.24V84.06H81.29V51.24Z" style="fill:#a8a9ab"/></g></symbol></defs><use width="106" height="106" xlink:href="#c"/><rect x="6.27" y="10.57" width="48.45" height="31.57" rx="1" style="fill:#393939;opacity:0.44;isolation:isolate"/><g style="mask:url(#b)"><rect x="6.17" y="8.99" width="48.54" height="32.36" style="fill:red"/><rect x="6.17" y="8.99" width="19.42" height="32.36" style="fill:#060"/><path d="M31.92,30.19c-2.45-.07-13.66-7.06-13.73-8.18l.62-1c1.11,1.61,12.57,8.42,13.69,8.18l-.58,1" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M18.68,20.84c-.22.59,2.92,2.53,6.7,4.83s7.05,3.73,7.29,3.52a2.19,2.19,0,0,0,.11-.2c-.05.07-.16.09-.33,0a38.55,38.55,0,0,1-7-3.52c-3.3-2-6.17-3.85-6.62-4.63a.45.45,0,0,1,0-.23h0l-.09.16v0ZM32,30.23c0,.07-.11.07-.26.06-.92-.1-3.69-1.45-7-3.42-3.82-2.28-7-4.37-6.63-4.91l.09-.17h0c-.31.92,6.22,4.66,6.61,4.9,3.78,2.34,7,3.71,7.25,3.35l-.11.18Z" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M25.6,22.69a23.1,23.1,0,0,0,7.2-1l-.37-.61c-1,.57-4.06.94-6.85,1-3.3,0-5.62-.34-6.79-1.12l-.35.65a17.92,17.92,0,0,0,7.16,1.11" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M32.9,21.67a10.25,10.25,0,0,1-2.86.78,29.72,29.72,0,0,1-4.45.32,31.23,31.23,0,0,1-4.26-.27,12.44,12.44,0,0,1-3-.79l.08-.17a13.38,13.38,0,0,0,2.94.78,32.42,32.42,0,0,0,4.23.27A29.19,29.19,0,0,0,30,22.26a7.06,7.06,0,0,0,2.77-.8l.12.21Zm-.32-.61a8.77,8.77,0,0,1-2.74.73,28.61,28.61,0,0,1-4.23.3,29,29,0,0,1-4.08-.26,11.56,11.56,0,0,1-2.85-.72.86.86,0,0,1,.1-.16,10,10,0,0,0,2.76.7,27.53,27.53,0,0,0,4.07.25,29.68,29.68,0,0,0,4.2-.29,7.64,7.64,0,0,0,2.65-.75l.12.2Z" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M17.57,25.63c1.5.81,4.84,1.22,8,1.25,2.88,0,6.63-.45,8-1.19l0-.81c-.44.69-4.46,1.34-8,1.32s-6.88-.58-8-1.29v.72" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M33.69,25.52v.19a9,9,0,0,1-3.19.9,33.51,33.51,0,0,1-5,.34,31.69,31.69,0,0,1-4.7-.32,11.9,11.9,0,0,1-3.37-.91v-.23a11.9,11.9,0,0,0,3.39,1,31.5,31.5,0,0,0,4.68.32,31.67,31.67,0,0,0,4.93-.34,10.5,10.5,0,0,0,3.22-.91Zm0-.69V25a8.74,8.74,0,0,1-3.19.9,32.1,32.1,0,0,1-5,.34,31.69,31.69,0,0,1-4.7-.32A11.54,11.54,0,0,1,17.47,25v-.22a12,12,0,0,0,3.39,1,30.19,30.19,0,0,0,4.68.32,33.16,33.16,0,0,0,4.93-.33,10.45,10.45,0,0,0,3.22-.92Z" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M25.57,30.06a27.53,27.53,0,0,1-7-1.1l.45.71a19,19,0,0,0,6.63,1.09,17,17,0,0,0,6.55-1.07l.47-.74a25.13,25.13,0,0,1-7,1.11" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M32.33,29.52l-.21.31a19.87,19.87,0,0,1-2.48.64,22.72,22.72,0,0,1-4.07.37,19.65,19.65,0,0,1-6.75-1.16l-.1-.16,0,0,.16.07a21.66,21.66,0,0,0,6.69,1.1,21.94,21.94,0,0,0,4-.37,11.55,11.55,0,0,0,2.69-.74l.06,0Zm.4-.67h0l-.16.26a17,17,0,0,1-3.13.7,25.31,25.31,0,0,1-3.84.32,26.43,26.43,0,0,1-7.14-1.06l-.09-.18A29.88,29.88,0,0,0,25.61,30a27.19,27.19,0,0,0,3.81-.32,19.81,19.81,0,0,0,3.12-.7h0l.2-.08Z" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M32.77,25a7.71,7.71,0,0,1-2.09,5.22,7.53,7.53,0,0,1-5.14,2.14,7.66,7.66,0,0,1-5-2.11,7.67,7.67,0,0,1-2.09-5.12,7.23,7.23,0,0,1,2.53-5.4A7.55,7.55,0,0,1,25.8,18a7.21,7.21,0,0,1,7,7ZM25.56,17.2a8,8,0,1,1-8,8,8,8,0,0,1,8-8" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M25.58,17.17a8,8,0,1,1-8,8,8,8,0,0,1,8-8Zm-7.83,8a7.83,7.83,0,1,0,7.83-7.83,7.86,7.86,0,0,0-7.83,7.83Z" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M25.59,17.84a7.34,7.34,0,1,1-7.34,7.34,7.36,7.36,0,0,1,7.34-7.34Zm-7.17,7.34A7.17,7.17,0,1,0,25.59,18a7.19,7.19,0,0,0-7.17,7.17Z" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M25.91,17.14h-.69v16.1h.69Z" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M25.84,17.05H26V33.33h-.18V17.05Zm-.68,0h.17V33.33h-.17V17.05Z" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M33.6,25.49V24.9l-.49-.45-2.75-.73-4-.41-4.78.25-3.4.81-.68.5v.6l1.74-.78L23.4,24h4l2.91.33,2,.48Z" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M25.58,23.94a27.6,27.6,0,0,1,5.19.47,6.06,6.06,0,0,1,2.92,1.09v.21a6.49,6.49,0,0,0-3-1.13,28,28,0,0,0-5.15-.46,29.87,29.87,0,0,0-5.25.47c-1.15.23-2.67.68-2.86,1.12V25.5c.1-.31,1.24-.77,2.83-1.09a29.29,29.29,0,0,1,5.28-.47Zm0-.68a27.58,27.58,0,0,1,5.19.46,5.94,5.94,0,0,1,2.92,1.1V25a6.38,6.38,0,0,0-3-1.13,26.34,26.34,0,0,0-5.15-.46,28.15,28.15,0,0,0-5.25.47c-1.1.21-2.68.67-2.86,1.12v-.22c.1-.3,1.26-.78,2.83-1.09a28.49,28.49,0,0,1,5.28-.46Z" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M25.54,19.75a17.32,17.32,0,0,1,6.78,1l.43.75a17.29,17.29,0,0,0-7.2-1.05c-2.74,0-5.67.3-7.14,1.08l.52-.86c1.21-.63,4.05-.95,6.61-.95" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M25.58,20.37a27,27,0,0,1,4.65.33,8.14,8.14,0,0,1,2.55.75l.13.22a9.73,9.73,0,0,0-2.7-.8,26.91,26.91,0,0,0-4.63-.32,27.78,27.78,0,0,0-4.69.32,8.48,8.48,0,0,0-2.52.79l.12-.24a8.27,8.27,0,0,1,2.37-.72,26.58,26.58,0,0,1,4.72-.33Zm0-.69a26.29,26.29,0,0,1,4.49.32,7.13,7.13,0,0,1,2.32.76l.19.29A6.22,6.22,0,0,0,30,20.17a28.64,28.64,0,0,0-4.41-.31,27.85,27.85,0,0,0-4.51.33,7.74,7.74,0,0,0-2.24.69l.17-.25A8,8,0,0,1,21,20a27,27,0,0,1,4.54-.34Z" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M29.56,28.51a21.46,21.46,0,0,0-4-.3c-5,.06-6.57,1-6.77,1.31l-.37-.6c1.27-.92,4-1.43,7.17-1.38a26,26,0,0,1,4.3.37l-.35.6" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M25.55,28.13a23.47,23.47,0,0,1,4.05.32l-.09.16a21.35,21.35,0,0,0-4-.3,19.43,19.43,0,0,0-5.31.62c-.5.14-1.35.47-1.43.74l-.1-.15c0-.16.54-.5,1.49-.76a19.14,19.14,0,0,1,5.34-.63Zm.07-.7a24.47,24.47,0,0,1,4.34.38l-.1.17a22.67,22.67,0,0,0-4.24-.37,20.79,20.79,0,0,0-5.56.65c-.57.17-1.56.53-1.59.82l-.1-.17c0-.26.88-.6,1.65-.82a20.3,20.3,0,0,1,5.6-.66Z" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M32.67,29l-.59.93-1.72-1.53-4.45-3-5-2.75-2.6-.89.55-1,.19-.11,1.62.41L26,23.8l3.08,1.94,2.58,1.86,1.06,1.22Z" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M18.2,21.8c.45-.31,3.81,1.19,7.32,3.3s6.85,4.53,6.55,5l-.1.15,0,0s.06-.07,0-.23c-.15-.5-2.52-2.39-6.46-4.77s-7-3.67-7.36-3.28l.1-.19ZM32.79,29c.29-.58-2.82-2.92-6.68-5.2s-6.8-3.56-7.32-3.17l-.11.21s0,0,0,0a.51.51,0,0,1,.32-.08c.89,0,3.45,1.19,7,3.25,1.57.91,6.64,4.16,6.62,5.08,0,.08,0,.09,0,.13l.12-.19Z" style="fill:#ff0;stroke:#000;stroke-linecap:round;stroke-linejoin:round;stroke-width:0.05px;fill-rule:evenodd"/><path d="M20.78,26.07a4.85,4.85,0,0,0,4.8,4.8,4.81,4.81,0,0,0,4.81-4.8h0V19.66H20.78v6.42Z" style="fill:#fff;stroke:#000;stroke-width:0.05px"/><path d="M21,26.08h0a4.57,4.57,0,0,0,1.36,3.24,4.6,4.6,0,0,0,6.51,0,4.56,4.56,0,0,0,1.35-3.24h0V19.87H21v6.21m7.36-4.35V26.1h0c0,.11,0,.24,0,.34a2.66,2.66,0,0,1-.79,1.6,2.74,2.74,0,0,1-1.94.8A2.71,2.71,0,0,1,23.64,28a2.75,2.75,0,0,1-.81-1.93V21.72Z" style="fill:red;stroke:#000;stroke-width:0.05px"/><path d="M21.56,21.49a.57.57,0,0,1,.33-.56.57.57,0,0,1,.34.56h-.67" style="fill:#ff0"/><path d="M21.28,20.94l-.05.52h.33a.47.47,0,0,1,.33-.5.49.49,0,0,1,.33.5h.34l-.06-.52Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.21,21.46h1.37a.06.06,0,0,1,.05.06s0,.07-.05.07H21.21a.07.07,0,0,1-.06-.07A.06.06,0,0,1,21.21,21.46Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.7,21.46a.38.38,0,0,1,.19-.35.36.36,0,0,1,.19.35H21.7" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.23,20.73h1.32a.06.06,0,0,1,.05.06,0,0,0,0,1-.05.05H21.23a.05.05,0,0,1-.05-.05A.06.06,0,0,1,21.23,20.73Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.26,20.84h1.26a.06.06,0,0,1,0,.12H21.26a.06.06,0,0,1,0-.12Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.67,20h.1v.07h.07V20h.1v.07H22V20h.11v.16a0,0,0,0,1-.05,0h-.36a0,0,0,0,1,0,0V20Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22,20.21l0,.52h-.34l0-.53H22" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.62,20.45v.28h-.34v-.28Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22.48,20.45v.28h-.34v-.28Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.24,20.24h.1v.07h.07v-.07h.1v.07h.07v-.07h.1v.16a0,0,0,0,1,0,.05h-.35a0,0,0,0,1-.05-.05v-.16Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22.1,20.24h.1v.07h.07v-.07h.1v.07h.07v-.07h.1v.16a0,0,0,0,1-.05.05h-.34a0,0,0,0,1,0-.05v-.16Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.86,20.37c0-.05.07-.05.07,0v.12h-.07v-.12"/><path d="M21.43,20.55c0-.05.06-.05.06,0v.1h-.06v-.1"/><path d="M22.29,20.55c0-.05.06-.05.06,0v.1h-.06v-.1"/><path d="M21.56,25.24a.56.56,0,0,1,.33-.56.57.57,0,0,1,.34.56h-.67" style="fill:#ff0"/><path d="M21.28,24.69l-.05.51h.33a.46.46,0,0,1,.33-.49.48.48,0,0,1,.33.49h.34l-.06-.51Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.21,25.21h1.37s.05,0,.05.06a.06.06,0,0,1-.05.06H21.21a.06.06,0,0,1-.06-.06A.06.06,0,0,1,21.21,25.21Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.7,25.2a.38.38,0,0,1,.19-.34.36.36,0,0,1,.19.34H21.7" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.23,24.48h1.32a0,0,0,0,1,.05.05s0,.06-.05.06H21.23a.06.06,0,0,1-.05-.06A.05.05,0,0,1,21.23,24.48Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.26,24.59h1.26a.06.06,0,0,1,0,.12H21.26a.06.06,0,0,1,0-.12Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.67,23.74h.1v.07h.07v-.08h.1v.08H22v-.08h.11v.17a0,0,0,0,1-.05,0h-.36a0,0,0,0,1,0,0v-.16Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22,24l0,.52h-.34l0-.52H22" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.62,24.19v.29h-.34v-.29Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22.48,24.19v.29h-.34v-.29Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.24,24h.1v.07h.07V24h.1v.07h.07V24h.1v.16a0,0,0,0,1,0,0h-.35s-.05,0-.05,0V24Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22.1,24h.1v.07h.07V24h.1v.07h.07V24h.1v.16s0,0-.05,0h-.34s0,0,0,0V24Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.86,24.12c0-.05.07-.05.07,0v.12h-.07v-.12"/><path d="M21.43,24.3c0-.05.06-.05.06,0v.09h-.06V24.3"/><path d="M22.29,24.3c0-.05.06-.05.06,0v.09h-.06V24.3"/><path d="M23,29.14a.57.57,0,0,1-.16-.63.56.56,0,0,1,.63.15l-.47.48" style="fill:#ff0"/><path d="M22.44,29l.32.4.24-.24a.46.46,0,0,1-.12-.58.5.5,0,0,1,.59.11l.23-.24-.41-.32-.85.87Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22.75,29.36l1-1s.05,0,.08,0a.06.06,0,0,1,0,.08l-1,1a.06.06,0,0,1-.08,0A.06.06,0,0,1,22.75,29.36Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M23.1,29a.36.36,0,0,1-.12-.37.36.36,0,0,1,.38.1L23.1,29" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22.25,28.83l.93-.93a0,0,0,0,1,.07,0,.05.05,0,0,1,0,.08l-.93.93a0,0,0,0,1-.07,0S22.23,28.85,22.25,28.83Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22.35,28.89l.89-.89a.06.06,0,0,1,.08.08l-.89.89s-.05,0-.07,0S22.33,28.91,22.35,28.89Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22,28l.07-.07,0,.05.05-.05-.05-.05.07-.07.05.05.06-.06-.06-.05.08-.07.11.12s0,0,0,.06l-.25.25s-.05,0-.06,0L22,28Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22.45,27.89l.39.35-.25.25-.35-.39.21-.21" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22.32,28.36l.21.19-.24.24-.19-.21.22-.22Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22.93,27.75l.2.19-.23.24L22.71,28l.22-.22Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M21.91,28.48l.07-.07.05.05,0-.05,0-.05.07-.07,0,.05.05-.05-.05-.05.07-.07.12.12a.06.06,0,0,1,0,.06l-.25.24a0,0,0,0,1-.06,0l-.12-.12Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22.52,27.87l.07-.06.05,0,.05-.05-.06-.05.08-.07.05.05,0-.05,0-.05.07-.07.11.12s0,0,0,.06l-.25.24s0,0-.06,0l-.11-.12Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M22.44,28.14s0-.09,0-.05l.08.08-.05,0-.08-.08"/><path d="M22.26,28.57s0-.08,0-.05l.07.07-.05.05-.07-.07"/><path d="M22.87,28s0-.09,0-.05L23,28l0,.05L22.87,28"/><path d="M25.26,21.49a.57.57,0,0,1,.33-.56.57.57,0,0,1,.34.56h-.67" style="fill:#ff0"/><path d="M25,20.94l-.05.52h.33a.46.46,0,0,1,.33-.5.51.51,0,0,1,.33.5h.34l-.06-.52Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M24.9,21.46h1.38a.06.06,0,0,1,0,.06.07.07,0,0,1,0,.07H24.9a.07.07,0,0,1,0-.07A.06.06,0,0,1,24.9,21.46Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M25.4,21.46a.36.36,0,0,1,.19-.35.36.36,0,0,1,.19.35H25.4" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M24.93,20.73h1.32a.06.06,0,0,1,.05.06,0,0,0,0,1-.05.05H24.93a.05.05,0,0,1-.05-.05A.06.06,0,0,1,24.93,20.73Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M25,20.84h1.26a.06.06,0,0,1,0,.12H25a.06.06,0,0,1,0-.12Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M25.37,20h.1v.07h.07V20h.1v.07h.07V20h.1v.16a0,0,0,0,1,0,0h-.36a0,0,0,0,1,0,0V20Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M25.74,20.21l0,.52h-.35l0-.53h.3" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M25.32,20.45v.28H25v-.28Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M26.17,20.45v.28h-.33v-.28Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M24.94,20.24H25v.07h.07v-.07h.1v.07h.07v-.07h.1v.16a0,0,0,0,1,0,.05H25a0,0,0,0,1,0-.05v-.16Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M25.8,20.24h.1v.07H26v-.07h.1v.07h.07v-.07h.1v.16a0,0,0,0,1,0,.05h-.35a0,0,0,0,1,0-.05v-.16Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M25.56,20.37c0-.05.07-.05.07,0v.12h-.07v-.12"/><path d="M25.12,20.55a0,0,0,1,1,.07,0v.1h-.07v-.1"/><path d="M26,20.55c0-.05.06-.05.06,0v.1H26v-.1"/><path d="M29.6,21.49a.59.59,0,0,0-.33-.56.57.57,0,0,0-.34.56h.67" style="fill:#ff0"/><path d="M29.87,20.94l.06.52h-.34a.46.46,0,0,0-.33-.5.52.52,0,0,0-.33.5H28.6l.06-.52Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M30,21.46H28.58a.06.06,0,0,0,0,.06s0,.07,0,.07H30s.05,0,.05-.07A.06.06,0,0,0,30,21.46Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.45,21.46a.37.37,0,0,0-.18-.35.36.36,0,0,0-.19.35h.37" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.92,20.73H28.61a.06.06,0,0,0-.05.06,0,0,0,0,0,.05.05h1.31a0,0,0,0,0,0-.05A.06.06,0,0,0,29.92,20.73Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.89,20.84H28.64a.06.06,0,0,0,0,.12h1.25a.06.06,0,0,0,0-.12Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.49,20h-.1v.07h-.07V20h-.11v.07h-.07V20H29v.16a0,0,0,0,0,0,0h.36a0,0,0,0,0,0,0V20Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.11,20.21l0,.52h.35l0-.53h-.31" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.54,20.45v.28h.33v-.28Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M28.68,20.45v.28H29v-.28Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.91,20.24h-.09v.07h-.07v-.07h-.1v.07h-.07v-.07h-.1v.16a0,0,0,0,0,0,.05h.35a0,0,0,0,0,0-.05v-.16Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.06,20.24H29v.07h-.07v-.07h-.1v.07h-.07v-.07h-.1v.16a0,0,0,0,0,0,.05H29a.05.05,0,0,0,0-.05v-.16Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.3,20.37a0,0,0,1,0-.07,0v.12h.07v-.12"/><path d="M29.73,20.55a0,0,0,1,0-.07,0v.1h.07v-.1"/><path d="M28.87,20.55a0,0,0,1,0-.07,0v.1h.07v-.1"/><path d="M29.6,25.24a.57.57,0,0,0-.33-.56.57.57,0,0,0-.34.56h.67" style="fill:#ff0"/><path d="M29.87,24.69l.06.51h-.34a.45.45,0,0,0-.33-.49.5.5,0,0,0-.33.49H28.6l.06-.51Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M30,25.21H28.58s0,0,0,.06a.06.06,0,0,0,0,.06H30a.06.06,0,0,0,.05-.06S30,25.21,30,25.21Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.45,25.2a.36.36,0,0,0-.18-.34.36.36,0,0,0-.19.34h.37" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.92,24.48H28.61a0,0,0,0,0-.05.05s0,.06.05.06h1.31s0,0,0-.06A0,0,0,0,0,29.92,24.48Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.89,24.59H28.64a.06.06,0,0,0,0,.12h1.25a.06.06,0,0,0,0-.12Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.49,23.74h-.1v.07h-.07v-.08h-.11v.08h-.07v-.08H29v.17a0,0,0,0,0,0,0h.36a0,0,0,0,0,0,0v-.16Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.11,24l0,.52h.35l0-.52h-.31" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.54,24.19v.29h.33v-.29Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M28.68,24.19v.29H29v-.29Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.91,24h-.09v.07h-.07V24h-.1v.07h-.07V24h-.1v.16a0,0,0,0,0,0,0h.35a0,0,0,0,0,0,0V24Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.06,24H29v.07h-.07V24h-.1v.07h-.07V24h-.1v.16a0,0,0,0,0,0,0H29s0,0,0,0V24Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.3,24.12a0,0,0,1,0-.07,0v.12h.07v-.12"/><path d="M29.73,24.3a0,0,0,1,0-.07,0v.09h.07V24.3"/><path d="M28.87,24.3a0,0,0,1,0-.07,0v.09h.07V24.3"/><path d="M28.14,29.14a.57.57,0,0,0,.16-.63.57.57,0,0,0-.64.15l.48.48" style="fill:#ff0"/><path d="M28.72,29l-.33.4-.23-.24a.44.44,0,0,0,.11-.58.49.49,0,0,0-.58.11l-.24-.24.41-.32.86.87Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M28.41,29.36l-1-1a.07.07,0,0,0-.09.09l1,1a.06.06,0,0,0,.08,0A.08.08,0,0,0,28.41,29.36Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M28.06,29a.34.34,0,0,0,.11-.37.36.36,0,0,0-.38.1l.27.27" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M28.9,28.83,28,27.9a.06.06,0,1,0-.08.08l.92.93a.06.06,0,0,0,.08-.08Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M28.8,28.89,27.92,28s-.05,0-.08,0a0,0,0,0,0,0,.07l.88.89a.06.06,0,0,0,.08-.08Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.12,28l-.07-.07L29,28,29,27.93l.05-.05-.07-.07-.05.05-.05-.06.05-.05-.07-.07-.12.12a0,0,0,0,0,0,.06l.25.25s0,0,.06,0l.11-.11Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M28.71,27.89l-.39.35.25.25.35-.39-.21-.21" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M28.83,28.36l-.2.19.23.24.2-.21-.23-.22Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M28.23,27.75l-.21.19.24.24.19-.21-.22-.22Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M29.24,28.48l-.06-.07-.05.05-.05-.05.05-.05-.07-.07,0,.05L29,28.29l.06-.05-.07-.07-.12.12s0,0,0,.06l.25.24s0,0,.06,0l.11-.12Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M28.64,27.87l-.07-.06-.05,0-.05-.05.05-.05-.07-.07-.05.05,0-.05,0-.05-.07-.07-.11.12s0,0,0,.06l.24.24a0,0,0,0,0,.06,0l.12-.12Z" style="fill:#ff0;stroke:#000;stroke-width:0.05px"/><path d="M28.72,28.14s0-.09,0-.05l-.09.08.05,0,.09-.08"/><path d="M28.9,28.57a0,0,0,0,0,0-.05l-.07.07,0,.05.07-.07"/><path d="M28.29,28s0-.09-.05-.05l-.07.07,0,.05.07-.07"/><path d="M25,25.37h0a.68.68,0,0,0,.18.46.58.58,0,0,0,.86,0,.68.68,0,0,0,.18-.46V24.5H25v.87" style="fill:#039"/><circle cx="25.27" cy="24.83" r="0.12" style="fill:#fff"/><circle cx="25.94" cy="24.83" r="0.12" style="fill:#fff"/><circle cx="25.6" cy="25.15" r="0.12" style="fill:#fff"/><circle cx="25.27" cy="25.49" r="0.12" style="fill:#fff"/><circle cx="25.94" cy="25.49" r="0.12" style="fill:#fff"/><path d="M25,23.27h0a.7.7,0,0,0,.18.46.61.61,0,0,0,.43.19.57.57,0,0,0,.43-.19.68.68,0,0,0,.18-.46V22.4H25v.88" style="fill:#039"/><circle cx="25.27" cy="22.73" r="0.12" style="fill:#fff"/><circle cx="25.94" cy="22.73" r="0.12" style="fill:#fff"/><circle cx="25.6" cy="23.05" r="0.12" style="fill:#fff"/><circle cx="25.27" cy="23.39" r="0.12" style="fill:#fff"/><circle cx="25.94" cy="23.39" r="0.12" style="fill:#fff"/><path d="M23.31,25.37h0a.68.68,0,0,0,.18.46.57.57,0,0,0,.43.19.59.59,0,0,0,.43-.19.72.72,0,0,0,.17-.46V24.5H23.31v.87" style="fill:#039"/><circle cx="23.59" cy="24.83" r="0.12" style="fill:#fff"/><circle cx="24.26" cy="24.83" r="0.12" style="fill:#fff"/><circle cx="23.92" cy="25.15" r="0.12" style="fill:#fff"/><circle cx="23.59" cy="25.49" r="0.12" style="fill:#fff"/><circle cx="24.26" cy="25.49" r="0.12" style="fill:#fff"/><path d="M26.67,25.37h0a.68.68,0,0,0,.18.46.57.57,0,0,0,.43.19.59.59,0,0,0,.43-.19.67.67,0,0,0,.17-.46V24.5H26.67v.87" style="fill:#039"/><circle cx="26.95" cy="24.83" r="0.12" style="fill:#fff"/><circle cx="27.62" cy="24.83" r="0.12" style="fill:#fff"/><circle cx="27.28" cy="25.15" r="0.12" style="fill:#fff"/><circle cx="26.95" cy="25.49" r="0.12" style="fill:#fff"/><circle cx="27.62" cy="25.49" r="0.12" style="fill:#fff"/><path d="M25,27.46h0a.68.68,0,0,0,.18.46.58.58,0,0,0,.86,0,.68.68,0,0,0,.18-.46v-.87H25v.87" style="fill:#039"/><circle cx="25.27" cy="26.92" r="0.12" style="fill:#fff"/><circle cx="25.94" cy="26.92" r="0.12" style="fill:#fff"/><circle cx="25.6" cy="27.24" r="0.12" style="fill:#fff"/><circle cx="25.27" cy="27.58" r="0.12" style="fill:#fff"/><circle cx="25.94" cy="27.58" r="0.12" style="fill:#fff"/></g></svg>
```

