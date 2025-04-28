# Odoo Module: l10n_ua

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
# Copyright (C) 2019 Bohdan Lisnenko <bohdan.lisnenko@erp.co.ua>, ERP Ukraine

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.
# Copyright (C) 2019 Bohdan Lisnenko <bohdan.lisnenko@erp.co.ua>, ERP Ukraine

{
    'name': 'Ukraine - Accounting',
    'author': 'ERP Ukraine',
    'website': 'https://erp.co.ua',
    'version': '1.4',
    'description': """
Ukraine - Chart of accounts.
============================
    """,
    'category': 'Accounting/Localizations/Account Charts',
    'depends': ['account'],
    'data': [
        'data/account_chart_template.xml',
        'data/account.account.template.csv',
        'data/account_account_tag_data.xml',
        'data/account_tax_group_data.xml',
        'data/account_tax_template.xml',
        'data/account_chart_template_config.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'license': 'LGPL-3',
}

```

## File: data\account_account_tag_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="acc_tag_vat" model="account.account.tag">
            <field name="name">ПДВ</field>
            <field name="applicability">accounts</field>
        </record>
    </data>
</odoo>

```

## File: data\account_chart_template.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>

    <record id="l10n_ua_psbo_chart_template" model="account.chart.template">
        <field name="name">План рахунків ПСБО</field>
        <field name="cash_account_code_prefix">301</field>
        <field name="bank_account_code_prefix">311</field>
        <field name="transfer_account_code_prefix">333</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.UAH"/>
        <field name="country_id" ref="base.ua"/>
        <field name="use_storno_accounting" eval="True"/>
    </record>

    <record id="l10n_ua_ias_chart_template" model="account.chart.template">
        <field name="name">План рахунків МСФЗ</field>
        <field name="bank_account_code_prefix">1112</field>
        <field name="cash_account_code_prefix">1111</field>
        <field name="transfer_account_code_prefix">1119</field>
        <field name="code_digits">6</field>
        <field name="currency_id" ref="base.UAH"/>
        <field name="country_id" ref="base.ua"/>
        <field name="use_storno_accounting" eval="True"/>
    </record>
</odoo>

```

## File: data\account_chart_template_config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="l10n_ua_psbo_chart_template" model="account.chart.template">
            <field name="property_account_receivable_id" ref="ua_psbp_361"/>
            <field name="property_account_payable_id" ref="ua_psbp_631"/>
            <field name="property_account_expense_categ_id" ref="ua_psbp_901"/>
            <field name="property_account_income_categ_id" ref="ua_psbp_701"/>
            <field name="use_anglo_saxon" eval="True"/>
            <field name="property_stock_account_input_categ_id" ref="ua_psbp_2812"/>
            <field name="property_stock_account_output_categ_id" ref="ua_psbp_2811"/>
            <field name="property_stock_valuation_account_id" ref="ua_psbp_281"/>
            <field name="income_currency_exchange_account_id" ref="ua_psbp_711"/>
            <field name="expense_currency_exchange_account_id" ref="ua_psbp_942"/>
            <field name="default_pos_receivable_account_id" ref="ua_psbp_366" />
        </record>

        <record id="l10n_ua_ias_chart_template" model="account.chart.template">
            <field name="property_account_receivable_id" ref="ua_ias_1120"/>
            <field name="property_account_payable_id" ref="ua_ias_1200"/>
            <field name="property_account_expense_categ_id" ref="ua_ias_2200"/>
            <field name="property_account_income_categ_id" ref="ua_ias_2000"/>
            <field name="use_anglo_saxon" eval="True"/>
            <field name="property_stock_account_input_categ_id" ref="ua_ias_1201"/>
            <field name="property_stock_account_output_categ_id" ref="ua_ias_1121"/>
            <field name="property_stock_valuation_account_id" ref="ua_ias_1100"/>
            <field name="income_currency_exchange_account_id" ref="ua_ias_2100"/>
            <field name="expense_currency_exchange_account_id" ref="ua_ias_2500"/>
            <field name="default_pos_receivable_account_id" ref="ua_ias_1122" />
        </record>
    </data>

    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_ua.l10n_ua_psbo_chart_template')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_vat20" model="account.tax.group">
            <field name="name">ПДВ 20%</field>
            <field name="country_id" ref="base.ua"/>
        </record>
        <record id="tax_group_vat14" model="account.tax.group">
            <field name="name">ПДВ 14%</field>
            <field name="country_id" ref="base.ua"/>
        </record>
        <record id="tax_group_vat7" model="account.tax.group">
            <field name="name">ПДВ 7%</field>
            <field name="country_id" ref="base.ua"/>
        </record>
        <record id="tax_group_vat0" model="account.tax.group">
            <field name="name">ПДВ 0%</field>
            <field name="country_id" ref="base.ua"/>
        </record>
        <record id="tax_group_vat_free" model="account.tax.group">
            <field name="name">Звільнено від ПДВ</field>
            <field name="country_id" ref="base.ua"/>
        </record>
        <record id="tax_group_not_vat" model="account.tax.group">
            <field name="name">Не є ПДВ</field>
            <field name="country_id" ref="base.ua"/>
        </record>
    </data>
</odoo>

```

## File: data\account_tax_template.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="0">
        <record id="ua_psbp_6412" model="account.account.template">
            <field name="tag_ids" eval="[(6,0,[ref('l10n_ua.acc_tag_vat')])]"/>
        </record>
        <!-- Tax template for VAT -->
        <record id="sale_tax_template_vat20_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">9</field>
            <field name="name">Реалізація ПДВ 20%</field>
            <field name="description">+ ПДВ 20%</field>
            <field name="amount">20</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base'                 }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6431'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6431'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat20"/>
        </record>
        <record id="sale_tax_template_vat20incl_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">9</field>
            <field name="name">Реалізація в т. ч. ПДВ 20%</field>
            <field name="description">в т. ч. ПДВ 20%</field>
            <field name="amount">20</field>
            <field name="type_tax_use">sale</field>
            <field name="price_include" eval="1"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6431'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6431'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat20"/>
        </record>
        <record id="sale_tax_template_vat14_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">10</field>
            <field name="name">Реалізація ПДВ 14%</field>
            <field name="description">+ ПДВ 14%</field>
            <field name="amount">14</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {
                    'repartition_type': 'base'                 }),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6431'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6431'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat14"/>
        </record>
        <record id="sale_tax_template_vat14incl_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">10</field>
            <field name="name">Реалізація в т. ч. ПДВ 14%</field>
            <field name="description">в т. ч. ПДВ 14%</field>
            <field name="amount">14</field>
            <field name="type_tax_use">sale</field>
            <field name="price_include" eval="1"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6431'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6431'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat14"/>
        </record>
        <record id="sale_tax_template_vat7_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">11</field>
            <field name="name">Реалізація ПДВ 7%</field>
            <field name="description">+ ПДВ 7%</field>
            <field name="amount">7</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6431'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6431'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat7"/>
        </record>
        <record id="sale_tax_template_vat7incl_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">11</field>
            <field name="name">Реалізація в т. ч. ПДВ 7%</field>
            <field name="description">в т .ч. ПДВ 7%</field>
            <field name="amount">7</field>
            <field name="type_tax_use">sale</field>
            <field name="price_include" eval="1"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6431'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6431'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat7"/>
        </record>
        <record id="sale_tax_template_vat0_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">12</field>
            <field name="name">Реалізація ПДВ 0%</field>
            <field name="description">ПДВ 0%</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
        </record>
        <record id="sale_tax_template_vat_free_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">13</field>
            <field name="name">Реалізація звільнена від  ПДВ</field>
            <field name="description">Звільнено від ПДВ</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
        </record>
        <record id="sale_tax_template_vat_not_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">14</field>
            <field name="name">Реалізація Не є об'єктом ПДВ</field>
            <field name="description">Не є об'єктом ПДВ</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
        </record>
        <record id="purchase_tax_template_vat20_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">19</field>
            <field name="name">Придбання ПДВ 20%</field>
            <field name="description">+ ПДВ 20%</field>
            <field name="amount">20</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6441'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6441'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat20"/>
        </record>
        <record id="purchase_tax_template_vat20incl_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">19</field>
            <field name="name">Придбання в т. ч. ПДВ 20%</field>
            <field name="description">в т. ч. ПДВ 20%</field>
            <field name="amount">20</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="1"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6441'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6441'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat20"/>
        </record>
        <record id="purchase_tax_template_vat14_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">20</field>
            <field name="name">Придбання ПДВ 14%</field>
            <field name="description">+ ПДВ 14%</field>
            <field name="amount">14</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6441'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6441'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat14"/>
        </record>
        <record id="purchase_tax_template_vat14incl_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">20</field>
            <field name="name">Придбання в т. ч. ПДВ 14%</field>
            <field name="description">в т. ч. ПДВ 14%</field>
            <field name="amount">14</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="1"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6441'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6441'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat14"/>
        </record>
        <record id="purchase_tax_template_vat7_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">21</field>
            <field name="name">Придбання ПДВ 7%</field>
            <field name="description">+ ПДВ 7%</field>
            <field name="amount">7</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6441'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6441'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat7"/>
        </record>
        <record id="purchase_tax_template_vat7incl_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">21</field>
            <field name="name">Придбання в т. ч. ПДВ 7%</field>
            <field name="description">в т. ч. ПДВ 7%</field>
            <field name="amount">7</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="1"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6441'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_psbp_6441'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat7"/>
        </record>
        <record id="purchase_tax_template_vat0_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">22</field>
            <field name="name">Придбання ПДВ 0%</field>
            <field name="description">ПДВ 0%</field>
            <field name="amount">0</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
        </record>
        <record id="purchase_tax_template_vat_free_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">23</field>
            <field name="name">Придбання звільнене від  ПДВ</field>
            <field name="description">Звільнено від ПДВ</field>
            <field name="amount">0</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
        </record>
        <record id="purchase_tax_template_vat_not_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">24</field>
            <field name="name">Придбання Не є об'єктом ПДВ</field>
            <field name="description">Не є об'єктом ПДВ</field>
            <field name="amount">0</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
        </record>
        <!-- Simplified tax system -->
        <!-- Sale taxes -->
        <record id="simple_tax_sale_product_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">30</field>
            <field name="name">Дохід від продажу  товарів</field>
            <field name="description">Дохід від продажу товарів</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
        </record>
        <record id="simple_tax_sale_gift_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">31</field>
            <field name="name">Дохід від безоплатно отриманих  товарів</field>
            <field name="description">Дохід від безоплатно отриманих товарів</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
        </record>
        <record id="simple_tax_sale_old_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">32</field>
            <field name="name">Дохід від заборгованності за якою минув строк позивної давності</field>
            <field name="description">Дохід від заборгованності, за якою минув строк позивної давності</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
        </record>
        <record id="simple_tax_sale_15_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">33</field>
            <field name="name">Дохід, за ставкою 15%</field>
            <field name="description">Дохід за ставкою 15%</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
        </record>
        <!-- Purchase taxes -->
        <record id="simple_tax_purchase_product_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">40</field>
            <field name="name">Витрати від продажу  товарів</field>
            <field name="description">Витрати від продажу товарів</field>
            <field name="amount">0</field>
            <field name="type_tax_use">purchase</field>
        </record>
        <record id="simple_tax_purchase_salary_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">41</field>
            <field name="name">Витрати на оплату праці</field>
            <field name="description">Витрати на оплату праці найманих працівників</field>
            <field name="amount">0</field>
            <field name="type_tax_use">purchase</field>
        </record>
        <record id="simple_tax_purchase_esv_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">42</field>
            <field name="name">Витрати ЄСВ</field>
            <field name="description">Витрати на ЄСВ</field>
            <field name="amount">0</field>
            <field name="type_tax_use">purchase</field>
        </record>
        <record id="simple_tax_purchase_other_psbo" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_psbo_chart_template"/>
            <field name="sequence">43</field>
            <field name="name">Витрати  інші</field>
            <field name="description">Витрати інші</field>
            <field name="amount">0</field>
            <field name="type_tax_use">purchase</field>
        </record>
        <!-- IAS -->
        <record id="ua_ias_1203" model="account.account.template">
            <field name="tag_ids" eval="[(6,0,[ref('l10n_ua.acc_tag_vat')])]"/>
        </record>
        <!-- Tax template for VAT -->
        <record id="sale_tax_template_vat20" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">9</field>
            <field name="name">Реалізація з ПДВ 20%</field>
            <field name="description">+ ПДВ 20%</field>
            <field name="amount">20</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1204'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1204'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat20"/>
        </record>
        <record id="sale_tax_template_vat20incl" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">9</field>
            <field name="name">Реалізація в т. ч. ПДВ 20%</field>
            <field name="description">в т. ч. ПДВ 20%</field>
            <field name="amount">20</field>
            <field name="type_tax_use">sale</field>
            <field name="price_include" eval="1"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1204'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1204'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat20"/>
        </record>
        <record id="sale_tax_template_vat14" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">10</field>
            <field name="name">Реалізація з ПДВ 14%</field>
            <field name="description">+ ПДВ 14%</field>
            <field name="amount">14</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1204'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1204'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat14"/>
        </record>
        <record id="sale_tax_template_vat14incl" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">10</field>
            <field name="name">Реалізація в т. ч. ПДВ 14%</field>
            <field name="description">в т. ч. ПДВ 14%</field>
            <field name="amount">14</field>
            <field name="type_tax_use">sale</field>
            <field name="price_include" eval="1"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1204'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1204'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat14"/>
        </record>
        <record id="sale_tax_template_vat7" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">11</field>
            <field name="name">Реалізація з ПДВ 7%</field>
            <field name="description">+ ПДВ 7%</field>
            <field name="amount">7</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1204'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1204'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat7"/>
        </record>
        <record id="sale_tax_template_vat7incl" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">11</field>
            <field name="name">Реалізація в т. ч. ПДВ 7%</field>
            <field name="description">в т .ч. ПДВ 7%</field>
            <field name="amount">7</field>
            <field name="type_tax_use">sale</field>
            <field name="price_include" eval="1"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1204'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1204'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat7"/>
        </record>
        <record id="sale_tax_template_vat0" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">12</field>
            <field name="name">Реалізація з ПДВ 0%</field>
            <field name="description">ПДВ 0%</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
        </record>
        <record id="sale_tax_template_vat_free" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">13</field>
            <field name="name">Реалізація звільнена від ПДВ</field>
            <field name="description">Звільнено від ПДВ</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
        </record>
        <record id="sale_tax_template_vat_not" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">14</field>
            <field name="name">Реалізація не є об'єктом ПДВ</field>
            <field name="description">Не є об'єктом ПДВ</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
        </record>
        <record id="purchase_tax_template_vat20" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">19</field>
            <field name="name">Придбання з ПДВ 20%</field>
            <field name="description">+ ПДВ 20%</field>
            <field name="amount">20</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1140'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1140'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat20"/>
        </record>
        <record id="purchase_tax_template_vat20incl" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">19</field>
            <field name="name">Придбання в т. ч. ПДВ 20%</field>
            <field name="description">в т. ч. ПДВ 20%</field>
            <field name="amount">20</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="1"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1140'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1140'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat20"/>
        </record>
        <record id="purchase_tax_template_vat14" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">20</field>
            <field name="name">Придбання з ПДВ 14%</field>
            <field name="description">+ ПДВ 14%</field>
            <field name="amount">14</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1140'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1140'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat14"/>
        </record>
        <record id="purchase_tax_template_vat14incl" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">20</field>
            <field name="name">Придбання в т. ч. ПДВ 14%</field>
            <field name="description">в т. ч. ПДВ 14%</field>
            <field name="amount">14</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="1"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1140'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1140'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat14"/>
        </record>
        <record id="purchase_tax_template_vat7" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">21</field>
            <field name="name">Придбання з ПДВ 7%</field>
            <field name="description">+ ПДВ 7%</field>
            <field name="amount">7</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1140'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1140'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat7"/>
        </record>
        <record id="purchase_tax_template_vat7incl" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">21</field>
            <field name="name">Придбання в т. ч. ПДВ 7%</field>
            <field name="description">в т. ч. ПДВ 7%</field>
            <field name="amount">7</field>
            <field name="type_tax_use">purchase</field>
            <field name="price_include" eval="1"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1140'),
                }),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {
                    'repartition_type': 'tax',
                    'account_id': ref('ua_ias_1140'),
                }),
            ]"/>
            <field name="tax_group_id" ref="l10n_ua.tax_group_vat7"/>
        </record>
        <record id="purchase_tax_template_vat0" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">22</field>
            <field name="name">Придбання з ПДВ 0%</field>
            <field name="description">ПДВ 0%</field>
            <field name="amount">0</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
        </record>
        <record id="purchase_tax_template_vat_free" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">23</field>
            <field name="name">Придбання звільнене від ПДВ</field>
            <field name="description">Звільнено від ПДВ</field>
            <field name="amount">0</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
        </record>
        <record id="purchase_tax_template_vat_not" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">24</field>
            <field name="name">Придбання не є об'єктом ПДВ</field>
            <field name="description">Не є об'єктом ПДВ</field>
            <field name="amount">0</field>
            <field name="type_tax_use">purchase</field>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0, 0, {'repartition_type': 'base'}),
                (0, 0, {'repartition_type': 'tax'}),
            ]"/>
        </record>
        <!-- Simplified tax system -->
        <!-- Sale taxes -->
        <record id="simple_tax_sale_product" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">30</field>
            <field name="name">Дохід від продажу товарів</field>
            <field name="description">Дохід від продажу товарів</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
        </record>
        <record id="simple_tax_sale_gift" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">31</field>
            <field name="name">Дохід від безоплатно отриманих товарів</field>
            <field name="description">Дохід від безоплатно отриманих товарів</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
        </record>
        <record id="simple_tax_sale_old" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">32</field>
            <field name="name">Дохід від заборгованності, за якою минув строк позивної давності</field>
            <field name="description">Дохід від заборгованності, за якою минув строк позивної давності</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
        </record>
        <record id="simple_tax_sale_15" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">33</field>
            <field name="name">Дохід, що оподатковується за ставкою 15%</field>
            <field name="description">Дохід за ставкою 15%</field>
            <field name="amount">0</field>
            <field name="type_tax_use">sale</field>
        </record>
        <!-- Purchase taxes -->
        <record id="simple_tax_purchase_product" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">40</field>
            <field name="name">Витрати від продажу товарів</field>
            <field name="description">Витрати від продажу товарів</field>
            <field name="amount">0</field>
            <field name="type_tax_use">purchase</field>
        </record>
        <record id="simple_tax_purchase_salary" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">41</field>
            <field name="name">Витрати на оплату праці найманих працівників</field>
            <field name="description">Витрати на оплату праці найманих працівників</field>
            <field name="amount">0</field>
            <field name="type_tax_use">purchase</field>
        </record>
        <record id="simple_tax_purchase_esv" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">42</field>
            <field name="name">Витрати на ЄСВ</field>
            <field name="description">Витрати на ЄСВ</field>
            <field name="amount">0</field>
            <field name="type_tax_use">purchase</field>
        </record>
        <record id="simple_tax_purchase_other" model="account.tax.template">
            <field name="chart_template_id" ref="l10n_ua_ias_chart_template"/>
            <field name="sequence">43</field>
            <field name="name">Витрати інші</field>
            <field name="description">Витрати інші</field>
            <field name="amount">0</field>
            <field name="type_tax_use">purchase</field>
        </record>
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
    <mask id="b" x="4.8" y="6.07" width="50.4" height="33.8" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.65" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="1200" height="800" transform="translate(4.8 6.07) scale(0.04 0.04)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABLAAAAMlCAYAAABjLQqxAAAACXBIWXMAAQeYAAEHmAEWNs1oAAAWMUlEQVR4Xu3asQ3DMBAEwZfhKtWt4B7UDl0CQ24wE18Fi7vm/q0BAAAAgKjPbgAAAAAAJwlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKRd6521GwEAAADAKR5YAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkCVgAAAAApAlYAAAAAKQJWAAAAACkfWfm2Y0AAAAA4JQ/3xwL/LN/vP0AAAAASUVORK5CYII="/>
    </g>
  </g>
</svg>

```

