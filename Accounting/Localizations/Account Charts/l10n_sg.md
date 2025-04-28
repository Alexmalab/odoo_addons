# Odoo Module: l10n_sg

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2014 Tech Receptives (<http://techreceptives.com>).
from . import models

def _preserve_tag_on_taxes(cr, registry):
    from odoo.addons.account.models.chart_template import preserve_existing_tags_on_taxes
    preserve_existing_tags_on_taxes(cr, registry, 'l10n_sg')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2014 Tech Receptives (<http://techreceptives.com>)

{
    'name': 'Singapore - Accounting',
    'author': 'Tech Receptives',
    'version': '2.2',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Singapore accounting chart and localization.
=======================================================

This module add, for accounting:
 - The Chart of Accounts of Singapore
 - Field UEN (Unique Entity Number) on company and partner
 - Field PermitNo and PermitNoDate on invoice

    """,
    'depends': ['base', 'account'],
    'data': [
        'data/l10n_sg_chart_data.xml',
        'data/account_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_data.xml',
        'data/account_tax_template_data_2024.xml',
        'data/account_chart_template_data.xml',
        'views/account_invoice_view.xml',
        'views/res_company_view.xml',
        'views/res_partner_view.xml',
    ],
    'post_init_hook': '_preserve_tag_on_taxes',
    'license': 'LGPL-3',
}

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_sg.sg_chart_template')]"/>
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
        <record id="tax_group_0" model="account.tax.group">
            <field name="name">TAX 0%</field>
        </record>
        <record id="tax_group_7" model="account.tax.group">
            <field name="name">TAX 7%</field>
        </record>
        <record id="tax_group_8" model="account.tax.group">
            <field name="name">TAX 8%</field>
        </record>

    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <!-- Sales Tax -->
        <record id="sg_sale_tax_es_0" model="account.tax.template">
            <field name="name">Sales Tax 0% ES33</field>
            <field name="description">0% ES33</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_exempt_supplies')],
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
                        'minus_report_line_ids': [ref('account_tax_report_line_exempt_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                    }),
                ]"/>
        </record>

        <record id="sg_sale_tax_ds_7" model="account.tax.template">
            <field name="name">Sales Tax 7% DS</field>
            <field name="description">7% DS</field>
            <field name="price_include" eval="0"/>
            <field name="amount">7</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_7"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_792'),
                        'plus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_792'),
                        'minus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                    }),
                ]"/>
        </record>

        <record id="sg_sale_tax_ds_8" model="account.tax.template">
            <field name="name">Sales Tax 8% DS</field>
            <field name="description">8% DS</field>
            <field name="price_include" eval="0"/>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_8"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_792'),
                        'plus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_792'),
                        'minus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                    }),
                ]"/>
        </record>

        <record id="sg_sale_tax_esn_0" model="account.tax.template">
            <field name="name">Sales Tax 0% ESN33</field>
            <field name="description">0% ESN33</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_exempt_supplies')],
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
                        'minus_report_line_ids': [ref('account_tax_report_line_exempt_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                    }),
                ]"/>
        </record>

        <record id="sg_sale_tax_os_0" model="account.tax.template">
            <field name="name">Sales Tax 0% OS</field>
            <field name="description">0% OS</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_0"/>
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

        <record id="sg_sale_tax_zr_0" model="account.tax.template">
            <field name="name">Sales Tax 0% ZR</field>
            <field name="description">0% ZR</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_zero_rated_supplies')],
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
                        'minus_report_line_ids': [ref('account_tax_report_line_zero_rated_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                    }),
                ]"/>
        </record>

        <record id="sg_sale_tax_sr_7" model="account.tax.template">
            <field name="name">Sales Tax 7% SR</field>
            <field name="description">7% SR</field>
            <field name="price_include" eval="0"/>
            <field name="amount">7</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_7"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_796'),
                        'plus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_796'),
                        'minus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                    }),
                ]"/>
        </record>

        <record id="sg_sale_tax_sr_8" model="account.tax.template">
            <field name="name">Sales Tax 8% SR</field>
            <field name="description">8% SR</field>
            <field name="price_include" eval="0"/>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_8"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_796'),
                        'plus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_796'),
                        'minus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                    }),
                ]"/>
        </record>

        <record id="sg_sale_tax_srca_s_na" model="account.tax.template">
            <field name="name">Sales Tax SRCA-S</field>
            <field name="description">SRCA-S</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
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
                        'minus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                    }),
                ]"/>
        </record>

        <record id="sg_sale_tax_srca_c_7" model="account.tax.template">
            <field name="name">Sales Tax 7% SRCA-C</field>
            <field name="description">7% SRCA-C</field>
            <field name="price_include" eval="0"/>
            <field name="amount">7</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_7"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_798'),
                        'plus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_798'),
                        'minus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                    }),
                ]"/>
        </record>

        <record id="sg_sale_tax_srca_c_8" model="account.tax.template">
            <field name="name">Sales Tax 8% SRCA-C</field>
            <field name="description">8% SRCA-C</field>
            <field name="price_include" eval="0"/>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">sale</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_8"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_798'),
                        'plus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_798'),
                        'minus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                    }),
                ]"/>
        </record>

        <!-- Purchases Tax -->

        <record id="sg_purchase_tax_txn33_7" model="account.tax.template">
            <field name="name">Purchase Tax 7% TX-N33</field>
            <field name="description">7% TX-N33</field>
            <field name="price_include" eval="0"/>
            <field name="amount">7</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_7"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_738'),
                        'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_738'),
                        'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_txn33_8" model="account.tax.template">
            <field name="name">Purchase Tax 8% TX-N33</field>
            <field name="description">8% TX-N33</field>
            <field name="price_include" eval="0"/>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_8"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_738'),
                        'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_738'),
                        'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_txe33_7" model="account.tax.template">
            <field name="name">Purchase Tax 7% TX-E33</field>
            <field name="description">7% TX-E33</field>
            <field name="price_include" eval="0"/>
            <field name="amount">7</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_7"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_739'),
                        'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_739'),
                        'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_txe33_8" model="account.tax.template">
            <field name="name">Purchase Tax 8% TX-E33</field>
            <field name="description">8% TX-E33</field>
            <field name="price_include" eval="0"/>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_8"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_739'),
                        'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_739'),
                        'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_bl_7" model="account.tax.template">
            <field name="name">Purchase Tax 7% BL</field>
            <field name="description">7% BL</field>
            <field name="price_include" eval="0"/>
            <field name="amount">7</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_7"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_740'),
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
                        'account_id': ref('account_account_740'),
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_bl_8" model="account.tax.template">
            <field name="name">Purchase Tax 8% BL</field>
            <field name="description">8% BL</field>
            <field name="price_include" eval="0"/>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_8"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_740'),
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
                        'account_id': ref('account_account_740'),
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_im_7" model="account.tax.template">
            <field name="name">Purchase Tax 7% IM</field>
            <field name="description">7% IM</field>
            <field name="price_include" eval="0"/>
            <field name="amount">7</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_7"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_741'),
                        'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_741'),
                        'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_im_8" model="account.tax.template">
            <field name="name">Purchase Tax 8% IM</field>
            <field name="description">8% IM</field>
            <field name="price_include" eval="0"/>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_8"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_741'),
                        'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_741'),
                        'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_txre_7" model="account.tax.template">
            <field name="name">Purchase Tax 7% TX-RE</field>
            <field name="description">7% TX-RE</field>
            <field name="price_include" eval="0"/>
            <field name="amount">7</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_7"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_742'),
                        'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_742'),
                        'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_txre_8" model="account.tax.template">
            <field name="name">Purchase Tax 8% TX-RE</field>
            <field name="description">8% TX-RE</field>
            <field name="price_include" eval="0"/>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_8"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_742'),
                        'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_742'),
                        'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_me_0" model="account.tax.template">
            <field name="name">Purchase Tax 7% IGDS</field>
            <field name="description">7% IGDS</field>
            <field name="price_include" eval="0"/>
            <field name="amount">7</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_7"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_743'),
                        'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_743'),
                        'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_me_08" model="account.tax.template">
            <field name="name">Purchase Tax 8% IGDS</field>
            <field name="description">8% IGDS</field>
            <field name="price_include" eval="0"/>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_8"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_743'),
                        'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_743'),
                        'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_nr_0" model="account.tax.template">
            <field name="name">Purchase Tax 0% NR</field>
            <field name="description">0% NR</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_0"/>
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

        <record id="sg_purchase_tax_zp_0" model="account.tax.template">
            <field name="name">Purchase Tax 0% ZP</field>
            <field name="description">0% ZP</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [],
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
                        'minus_report_line_ids': [],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_op_0" model="account.tax.template">
            <field name="name">Purchase Tax 0% OP</field>
            <field name="description">0% OP</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_0"/>
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

        <record id="sg_purchase_tax_ep_0" model="account.tax.template">
            <field name="name">Purchase Tax 0% EP</field>
            <field name="description">0% EP</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_0"/>
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

        <record id="sg_purchase_tax_mes_0" model="account.tax.template">
            <field name="name">Purchase Tax MES</field>
            <field name="description">0% MES</field>
            <field name="price_include" eval="0"/>
            <field name="amount">0</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_0"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_applicable_goods_imported_value')],
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
                        'minus_report_line_ids': [ref('account_tax_report_line_applicable_goods_imported_value')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_tx7_7" model="account.tax.template">
            <field name="name">Purchase Tax 7% TX7</field>
            <field name="description">7% TX7</field>
            <field name="price_include" eval="0"/>
            <field name="amount">7</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_7"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_749'),
                        'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_749'),
                        'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_tx8_8" model="account.tax.template">
            <field name="name">Purchase Tax 8% TX8</field>
            <field name="description">8% TX8</field>
            <field name="price_include" eval="0"/>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_8"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_749'),
                        'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_749'),
                        'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_txca_7" model="account.tax.template">
            <field name="name">Purchase Tax 7% TXCA</field>
            <field name="description">7% TXCA</field>
            <field name="price_include" eval="0"/>
            <field name="amount">7</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_7"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_750'),
                        'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_750'),
                        'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
        </record>

        <record id="sg_purchase_tax_txca_8" model="account.tax.template">
            <field name="name">Purchase Tax 8% TXCA</field>
            <field name="description">8% TXCA</field>
            <field name="price_include" eval="0"/>
            <field name="amount">8</field>
            <field name="amount_type">percent</field>
            <field name="type_tax_use">purchase</field>
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="tax_group_id" ref="tax_group_8"/>
            <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_750'),
                        'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
            <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'base',
                        'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                    }),
                    (0,0, {
                        'factor_percent': 100,
                        'repartition_type': 'tax',
                        'account_id': ref('account_account_750'),
                        'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                    }),
                ]"/>
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
        <field name="country_id" ref="base.sg"/>
    </record>

    <record id="account_tax_report_line_supplies" model="account.tax.report.line">
        <field name="name">Supplies</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="account_tax_report_line_std_rated_supplies" model="account.tax.report.line">
        <field name="name">Box 1 - Total value of standard-rated supplies</field>
        <field name="tag_name">Box 1</field>
        <field name="code">BOX1</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref='account_tax_report_line_supplies'/>
    </record>

    <record id="account_tax_report_line_zero_rated_supplies" model="account.tax.report.line">
        <field name="name">Box 2 - Total value of zero-rated supplies</field>
        <field name="tag_name">Box 2</field>
        <field name="code">BOX2</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref='account_tax_report_line_supplies'/>
    </record>

    <record id="account_tax_report_line_exempt_supplies" model="account.tax.report.line">
        <field name="name">Box 3 - Total value of exempt supplies</field>
        <field name="tag_name">Box 3</field>
        <field name="code">BOX3</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref='account_tax_report_line_supplies'/>
    </record>

    <record id="account_tax_report_line_total_supplies" model="account.tax.report.line">
        <field name="name">Box 4 - Total value of (1) + (2) + (3)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="formula">BOX1 + BOX2 + BOX3</field>
        <field name="parent_id" ref='account_tax_report_line_supplies'/>
    </record>

    <record id="account_tax_report_line_purchases" model="account.tax.report.line">
        <field name="name">Purchases</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="account_tax_report_line_total_taxable_purchases" model="account.tax.report.line">
        <field name="name">Box 5 - Total value of taxable purchases</field>
        <field name="tag_name">Box 5</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref='account_tax_report_line_purchases'/>
    </record>

    <record id="account_tax_report_line_taxes" model="account.tax.report.line">
        <field name="name">Taxes</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="account_tax_report_line_output_tax_due" model="account.tax.report.line">
        <field name="name">Box 6 - Output tax due</field>
        <field name="tag_name">Box 6</field>
        <field name="code">BOX6</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref='account_tax_report_line_taxes'/>
    </record>

    <record id="account_tax_report_line_inp_tax_refund_claim" model="account.tax.report.line">
        <field name="name">Box 7 - Less : Input tax and refunds claimed</field>
        <field name="tag_name">Box 7</field>
        <field name="code">BOX7</field>
        <field name="parent_id" ref='account_tax_report_line_taxes'/>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="account_tax_report_line_total_gst_paid_iras" model="account.tax.report.line">
        <field name="name">Box 8 - Equals : Net GST to be paid to IRAS</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="formula">BOX6 - BOX7</field>
        <field name="parent_id" ref='account_tax_report_line_taxes'/>
    </record>

    <record id="account_tax_report_line_applicable" model="account.tax.report.line">
        <field name="name">Applicable to Taxable Persons under Major Exported Scheme / Approved 3rd Party Logistics Company / Other Approved Schemes Only</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="account_tax_report_line_applicable_goods_imported_value" model="account.tax.report.line">
        <field name="name">Box 9 - Total value of goods imported under this scheme</field>
        <field name="tag_name">Box 9</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="account_tax_report_line_applicable"/>
    </record>

    <record id="account_tax_report_line_revenues" model="account.tax.report.line">
        <field name="name">Revenue</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="6"/>
    </record>

    <record id="account_tax_report_line_revenues_accounting_period" model="account.tax.report.line">
        <field name="name">Box 13 - Revenue for the accounting period</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="formula">net_profit</field>
        <field name="parent_id" ref="account_tax_report_line_revenues"/>
    </record>
</odoo>

```

## File: data\account_tax_template_data_2024.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_9" model="account.tax.group">
            <field name="name">TAX 9%</field>
        </record>
    </data>

    <record id="sg_sale_tax_ds_9" model="account.tax.template">
        <field name="name">Sales Tax 9% DS</field>
        <field name="description">9% DS</field>
        <field name="price_include" eval="0"/>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="sg_chart_template"/>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_792'),
                    'plus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_792'),
                    'minus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                }),
            ]"/>
    </record>

    <record id="sg_sale_tax_sr_9" model="account.tax.template">
        <field name="name">Sales Tax 9% SR</field>
        <field name="description">9% SR</field>
        <field name="price_include" eval="0"/>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="sg_chart_template"/>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_796'),
                    'plus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_796'),
                    'minus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                }),
            ]"/>
    </record>

    <record id="sg_sale_tax_srca_c_9" model="account.tax.template">
        <field name="name">Sales Tax 9% SRCA-C</field>
        <field name="description">9% SRCA-C</field>
        <field name="price_include" eval="0"/>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">sale</field>
        <field name="chart_template_id" ref="sg_chart_template"/>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_798'),
                    'plus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_std_rated_supplies')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_798'),
                    'minus_report_line_ids': [ref('account_tax_report_line_output_tax_due')],
                }),
            ]"/>
    </record>

    <record id="sg_purchase_tax_txn33_9" model="account.tax.template">
        <field name="name">Purchase Tax 9% TX-N33</field>
        <field name="description">9% TX-N33</field>
        <field name="price_include" eval="0"/>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="sg_chart_template"/>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_738'),
                    'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_738'),
                    'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                }),
            ]"/>
    </record>

    <record id="sg_purchase_tax_txe33_9" model="account.tax.template">
        <field name="name">Purchase Tax 9% TX-E33</field>
        <field name="description">9% TX-E33</field>
        <field name="price_include" eval="0"/>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="sg_chart_template"/>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_739'),
                    'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_739'),
                    'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                }),
            ]"/>
    </record>

    <record id="sg_purchase_tax_bl_9" model="account.tax.template">
        <field name="name">Purchase Tax 9% BL</field>
        <field name="description">9% BL</field>
        <field name="price_include" eval="0"/>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="sg_chart_template"/>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_740'),
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
                    'account_id': ref('account_account_740'),
                }),
            ]"/>
    </record>

    <record id="sg_purchase_tax_im_9" model="account.tax.template">
        <field name="name">Purchase Tax 9% IM</field>
        <field name="description">9% IM</field>
        <field name="price_include" eval="0"/>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="sg_chart_template"/>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_741'),
                    'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_741'),
                    'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                }),
            ]"/>
    </record>

    <record id="sg_purchase_tax_txre_9" model="account.tax.template">
        <field name="name">Purchase Tax 9% TX-RE</field>
        <field name="description">9% TX-RE</field>
        <field name="price_include" eval="0"/>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="sg_chart_template"/>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_742'),
                    'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_742'),
                    'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                }),
            ]"/>
    </record>

    <record id="sg_purchase_tax_igds_9" model="account.tax.template">
        <field name="name">Purchase Tax 9% IGDS</field>
        <field name="description">9% IGDS</field>
        <field name="price_include" eval="0"/>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="sg_chart_template"/>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_743'),
                    'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_743'),
                    'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                }),
            ]"/>
    </record>

    <record id="sg_purchase_tax_tx8_9" model="account.tax.template">
        <field name="name">Purchase Tax 9% TX9</field>
        <field name="description">9% TX9</field>
        <field name="price_include" eval="0"/>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="sg_chart_template"/>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_749'),
                    'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_749'),
                    'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                }),
            ]"/>
    </record>

    <record id="sg_purchase_tax_txca_9" model="account.tax.template">
        <field name="name">Purchase Tax 9% TXCA</field>
        <field name="description">9% TXCA</field>
        <field name="price_include" eval="0"/>
        <field name="amount">9</field>
        <field name="amount_type">percent</field>
        <field name="type_tax_use">purchase</field>
        <field name="chart_template_id" ref="sg_chart_template"/>
        <field name="tax_group_id" ref="tax_group_9"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'plus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_750'),
                    'plus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                }),
            ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'base',
                    'minus_report_line_ids': [ref('account_tax_report_line_total_taxable_purchases')],
                }),
                (0,0, {
                    'factor_percent': 100,
                    'repartition_type': 'tax',
                    'account_id': ref('account_account_750'),
                    'minus_report_line_ids': [ref('account_tax_report_line_inp_tax_refund_claim')],
                }),
            ]"/>
    </record>

</odoo>

```

## File: data\l10n_sg_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_sg_statements_menu" name="Singapore" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>

        <record id="sg_chart_template" model="account.chart.template">
            <field name="name">Singapore Chart of Accounts - Standard</field>
            <field name="cash_account_code_prefix">10140</field>
            <field name="bank_account_code_prefix">10141</field>
            <field name="transfer_account_code_prefix">101100</field>
            <field name="code_digits">6</field>
            <field name="currency_id" ref="base.SGD" />
        </record>

        <record model="account.account.template" id="account_account_696">
            <field name="name">Allowance for Bad Debts</field>
            <field name="code">101110</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_697">
            <field name="name">Development Costs</field>
            <field name="code">104100</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_698">
            <field name="name">Employee Cash Advances</field>
            <field name="code">101120</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_699">
            <field name="name">Inventory</field>
            <field name="code">101130</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_700">
            <field name="name">Investments - Other</field>
            <field name="code">101140</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_701">
            <field name="name">Loans to Officers</field>
            <field name="code">101150</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_702">
            <field name="name">Loans to Others</field>
            <field name="code">101160</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_703">
            <field name="name">Loans to Shareholders</field>
            <field name="code">101170</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_704">
            <field name="name">Prepaid Expenses</field>
            <field name="code">101180</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_705">
            <field name="name">Retainage</field>
            <field name="code">101190</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_706">
            <field name="name">Undeposited Funds</field>
            <field name="code">101200</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_707">
            <field name="name">Other Current Assets</field>
            <field name="code">101210</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_709">
            <field name="name">Accumulated Amortisation</field>
            <field name="code">103100</field>
            <field name="user_type_id" ref="account.data_account_type_depreciation" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_710">
            <field name="name">Accumulated Depreciation</field>
            <field name="code">103110</field>
            <field name="user_type_id" ref="account.data_account_type_depreciation" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_711">
            <field name="name">Accumulated Depletion</field>
            <field name="code">103120</field>
            <field name="user_type_id" ref="account.data_account_type_depreciation" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_712">
            <field name="name">Buildings</field>
            <field name="code">102100</field>
            <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_713">
            <field name="name">Depletable Assets</field>
            <field name="code">102110</field>
            <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_714">
            <field name="name">Furniture and Fixtures</field>
            <field name="code">102120</field>
            <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_715">
            <field name="name">Leasehold Improvements</field>
            <field name="code">102130</field>
            <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_716">
            <field name="name">Machinery and Equipment</field>
            <field name="code">102140</field>
            <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_717">
            <field name="name">Vehicles</field>
            <field name="code">102150</field>
            <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_718">
            <field name="name">Other Assets</field>
            <field name="code">102160</field>
            <field name="user_type_id" ref="account.data_account_type_fixed_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_720">
            <field name="name">Intangible Assets</field>
            <field name="code">104110</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_721">
            <field name="name">Accumulated Amortization of Non-current Assets</field>
            <field name="code">103130</field>
            <field name="user_type_id" ref="account.data_account_type_depreciation" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_722">
            <field name="name">Available-for-sale Financial Assets</field>
            <field name="code">101220</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_723">
            <field name="name">Deferred Tax</field>
            <field name="code">101230</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_724">
            <field name="name">Goodwill</field>
            <field name="code">104120</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_725">
            <field name="name">Investments</field>
            <field name="code">104130</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_726">
            <field name="name">Lease Buyout</field>
            <field name="code">104140</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_727">
            <field name="name">Licences</field>
            <field name="code">104150</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_728">
            <field name="name">Organisational Costs</field>
            <field name="code">104160</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_729">
            <field name="name">Other Intangible Assets</field>
            <field name="code">104170</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_730">
            <field name="name">Other Non-current Assets</field>
            <field name="code">104180</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_731">
            <field name="name">Prepayments and Accrued Income</field>
            <field name="code">101240</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_732">
            <field name="name">Security Deposits</field>
            <field name="code">101250</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_735">
            <field name="name">Trade Receivable Account</field>
            <field name="code">100010</field>
            <field name="user_type_id" ref="account.data_account_type_receivable" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="True" />
        </record>

        <record model="account.account.template" id="account_account_736">
            <field name="name">Other Receivable Account</field>
            <field name="code">100020</field>
            <field name="user_type_id" ref="account.data_account_type_receivable" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="True" />
        </record>

        <record model="account.account.template" id="account_account_737">
            <field name="name">Trade Receivable Account (PoS)</field>
            <field name="code">100030</field>
            <field name="user_type_id" ref="account.data_account_type_receivable" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="True" />
        </record>

        <record model="account.account.template" id="account_account_738">
            <field name="name">Purchase Tax Account TX-N33</field>
            <field name="code">101260</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_739">
            <field name="name">Purchase Tax Account TX-E33</field>
            <field name="code">101270</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_740">
            <field name="name">Purchase Tax Account BL</field>
            <field name="code">101280</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_741">
            <field name="name">Purchase Tax Account IM</field>
            <field name="code">101290</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_742">
            <field name="name">Purchase Tax Account TX-RE</field>
            <field name="code">101300</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_743">
            <field name="name">Purchase Tax Account IGDS</field>
            <field name="code">101310</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_744">
            <field name="name">Purchase Tax Account NR</field>
            <field name="code">101320</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_745">
            <field name="name">Purchase Tax Account ZP</field>
            <field name="code">101330</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_746">
            <field name="name">Purchase Tax Account OP</field>
            <field name="code">101340</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_747">
            <field name="name">Purchase Tax Account EP</field>
            <field name="code">101350</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_748">
            <field name="name">Purchase Tax Account MES</field>
            <field name="code">101360</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_749">
            <field name="name">Purchase Tax Account TX9</field>
            <field name="code">101370</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_750">
            <field name="name">Purchase Tax Account TXCA</field>
            <field name="code">101380</field>
            <field name="user_type_id" ref="account.data_account_type_current_assets" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_752">
            <field name="name">Current Portion of Employee Benefits Obligations</field>
            <field name="code">201000</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_753">
            <field name="name">Current Portion of Obligations under Finance Leases</field>
            <field name="code">201010</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_754">
            <field name="name">Current Tax Liability</field>
            <field name="code">201020</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_755">
            <field name="name">Insurance Payable</field>
            <field name="code">201030</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_756">
            <field name="name">Interest Payables</field>
            <field name="code">201040</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_757">
            <field name="name">Line of Credit</field>
            <field name="code">201050</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_758">
            <field name="name">Loan Payable</field>
            <field name="code">201060</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_759">
            <field name="name">Payroll Clearing</field>
            <field name="code">201070</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_760">
            <field name="name">Payroll Liabilities</field>
            <field name="code">201080</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_761">
            <field name="name">Prepaid Expenses Payable</field>
            <field name="code">201090</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_762">
            <field name="name">Provision for Warranty Obligations</field>
            <field name="code">201100</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_763">
            <field name="name">Rents in Trust - Liability</field>
            <field name="code">201110</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_764">
            <field name="name">GST Payable</field>
            <field name="code">201120</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_765">
            <field name="name">Short Term Borrowings</field>
            <field name="code">201130</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_766">
            <field name="name">Client Trust Accounts - Liabilities</field>
            <field name="code">201140</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_768">
            <field name="name">Accruals and Deferred Income</field>
            <field name="code">201150</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_769">
            <field name="name">Bank Loans</field>
            <field name="code">202000</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_770">
            <field name="name">Obligations under Finance Leases</field>
            <field name="code">201160</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_771">
            <field name="name">Long Term Borrowings</field>
            <field name="code">202010</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_772">
            <field name="name">Long Term Employee Benefit Obligations</field>
            <field name="code">202020</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_773">
            <field name="name">Notes Payable</field>
            <field name="code">202030</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_774">
            <field name="name">Shareholder Notes Payable</field>
            <field name="code">202040</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_775">
            <field name="name">Other Non-current Liabilities</field>
            <field name="code">202050</field>
            <field name="user_type_id" ref="account.data_account_type_non_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_777">
            <field name="name">Trade Payable Account</field>
            <field name="code">200010</field>
            <field name="user_type_id" ref="account.data_account_type_payable" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="True" />
        </record>

        <record model="account.account.template" id="account_account_778">
            <field name="name">Other Payable Account</field>
            <field name="code">200020</field>
            <field name="user_type_id" ref="account.data_account_type_payable" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="True" />
        </record>

        <record model="account.account.template" id="account_account_780">
            <field name="name">Partner's Equity</field>
            <field name="code">203000</field>
            <field name="user_type_id" ref="account.data_account_type_equity" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_781">
            <field name="name">Accumulated Adjustment</field>
            <field name="code">203010</field>
            <field name="user_type_id" ref="account.data_account_type_equity" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_782">
            <field name="name">Ordinary Shares</field>
            <field name="code">203020</field>
            <field name="user_type_id" ref="account.data_account_type_equity" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_783">
            <field name="name">Opening Balance Equity</field>
            <field name="code">203030</field>
            <field name="user_type_id" ref="account.data_account_type_equity" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_784">
            <field name="name">Owner's Equity</field>
            <field name="code">203040</field>
            <field name="user_type_id" ref="account.data_account_type_equity" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_785">
            <field name="name">Paid-in Capital or Surplus</field>
            <field name="code">203050</field>
            <field name="user_type_id" ref="account.data_account_type_equity" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_786">
            <field name="name">Preferred Shares</field>
            <field name="code">203060</field>
            <field name="user_type_id" ref="account.data_account_type_equity" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_787">
            <field name="name">Retained Earnings</field>
            <field name="code">203070</field>
            <field name="user_type_id" ref="account.data_account_type_equity" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_788">
            <field name="name">Share Capital</field>
            <field name="code">203080</field>
            <field name="user_type_id" ref="account.data_account_type_equity" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_789">
            <field name="name">Treasury Shares</field>
            <field name="code">203090</field>
            <field name="user_type_id" ref="account.data_account_type_equity" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_791">
            <field name="name"> Sales Tax Account ES33</field>
            <field name="code">201170</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_792">
            <field name="name">Sales Tax Account DS</field>
            <field name="code">201180</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_793">
            <field name="name">Sales Tax Account ESN33</field>
            <field name="code">201190</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_794">
            <field name="name">Sales Tax Account OS</field>
            <field name="code">201200</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_795">
            <field name="name">Sales Tax Account ZR</field>
            <field name="code">201210</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_796">
            <field name="name">Sales Tax Account SR</field>
            <field name="code">201220</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_797">
            <field name="name">Sales Tax Account SRCA-S</field>
            <field name="code">201230</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_798">
            <field name="name">Sales Tax Account SRCA-C</field>
            <field name="code">201240</field>
            <field name="user_type_id" ref="account.data_account_type_current_liabilities" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_800">
            <field name="name">Discounts/Refunds -Given</field>
            <field name="code">300001</field>
            <field name="user_type_id" ref="account.data_account_type_revenue" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_801">
            <field name="name">Non-profit Revenue</field>
            <field name="code">300002</field>
            <field name="user_type_id" ref="account.data_account_type_revenue" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_802">
            <field name="name">Other Primary Revenue</field>
            <field name="code">300003</field>
            <field name="user_type_id" ref="account.data_account_type_revenue" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_803">
            <field name="name">Sales of Product Revenue</field>
            <field name="code">300004</field>
            <field name="user_type_id" ref="account.data_account_type_revenue" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_804">
            <field name="name">Service/Fee Revenue</field>
            <field name="code">300005</field>
            <field name="user_type_id" ref="account.data_account_type_revenue" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_805">
            <field name="name">Unapplied Cash Payment Income</field>
            <field name="code">300006</field>
            <field name="user_type_id" ref="account.data_account_type_revenue" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_807">
            <field name="name">Dividend Revenue</field>
            <field name="code">301001</field>
            <field name="user_type_id" ref="account.data_account_type_other_income" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_808">
            <field name="name">Gain/Loss on Sale of Fixed Assets or Investments</field>
            <field name="code">301002</field>
            <field name="user_type_id" ref="account.data_account_type_other_income" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_809">
            <field name="name">Interest Earned</field>
            <field name="code">301003</field>
            <field name="user_type_id" ref="account.data_account_type_other_income" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_810">
            <field name="name">Other Investment Revenue</field>
            <field name="code">301004</field>
            <field name="user_type_id" ref="account.data_account_type_other_income" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_811">
            <field name="name">Other Miscellaneous Revenue</field>
            <field name="code">301005</field>
            <field name="user_type_id" ref="account.data_account_type_other_income" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_812">
            <field name="name">Tax-exempt Interest</field>
            <field name="code">301006</field>
            <field name="user_type_id" ref="account.data_account_type_other_income" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_815">
            <field name="name">Cost of Labour - COS</field>
            <field name="code">401001</field>
            <field name="user_type_id" ref="account.data_account_type_direct_costs" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_816">
            <field name="name">Equipment Rental - COS</field>
            <field name="code">401002</field>
            <field name="user_type_id" ref="account.data_account_type_direct_costs" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_817">
            <field name="name">Freight and Delivery - COS</field>
            <field name="code">401003</field>
            <field name="user_type_id" ref="account.data_account_type_direct_costs" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_818">
            <field name="name">Other Costs of Sales - COS</field>
            <field name="code">401004</field>
            <field name="user_type_id" ref="account.data_account_type_direct_costs" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_819">
            <field name="name">Supplies and Materials - COS</field>
            <field name="code">401005</field>
            <field name="user_type_id" ref="account.data_account_type_direct_costs" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_822">
            <field name="name">Administrative Expenses</field>
            <field name="code">501001</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_823">
            <field name="name">Advertising/Promotional</field>
            <field name="code">501002</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_824">
            <field name="name">Auto</field>
            <field name="code">501003</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_825">
            <field name="name">Bad Debts</field>
            <field name="code">501004</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_826">
            <field name="name">Bank Charges</field>
            <field name="code">501005</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_827">
            <field name="name">Charitable Contributions</field>
            <field name="code">501006</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_828">
            <field name="name">Cost of Labour</field>
            <field name="code">501007</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_829">
            <field name="name">Distribution Costs</field>
            <field name="code">501008</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_830">
            <field name="name">Dues and Subscriptions</field>
            <field name="code">501009</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_831">
            <field name="name">Entertainment</field>
            <field name="code">501010</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_832">
            <field name="name">Meals and Entertainment</field>
            <field name="code">501011</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_833">
            <field name="name">Equipment Rental</field>
            <field name="code">501012</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_834">
            <field name="name">Finance Costs</field>
            <field name="code">501013</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_835">
            <field name="name">Insurance</field>
            <field name="code">501014</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_836">
            <field name="name">Interest Paid</field>
            <field name="code">501015</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_837">
            <field name="name">Legal and Professional Fees</field>
            <field name="code">501016</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_838">
            <field name="name">Other Miscellaneous Service Cost</field>
            <field name="code">501017</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_839">
            <field name="name">Payroll Expenses</field>
            <field name="code">501018</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_840">
            <field name="name">Promotional Meals</field>
            <field name="code">501019</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_841">
            <field name="name">Rent or Lease of Buildings</field>
            <field name="code">501020</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_842">
            <field name="name">Repair and Maintenance</field>
            <field name="code">501021</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_843">
            <field name="name">Shipping, Freight, and Delivery</field>
            <field name="code">501022</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_844">
            <field name="name">Supplies</field>
            <field name="code">501023</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_845">
            <field name="name">Taxes Paid</field>
            <field name="code">501024</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_846">
            <field name="name">Travel</field>
            <field name="code">501025</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_847">
            <field name="name">Travel Meals</field>
            <field name="code">501026</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_848">
            <field name="name">Utilities</field>
            <field name="code">501027</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_849">
            <field name="name">Unapplied Cash Bill Payment Expense</field>
            <field name="code">501028</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_851">
            <field name="name">Amortisation</field>
            <field name="code">502001</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_852">
            <field name="name">Depreciation</field>
            <field name="code">502002</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_853">
            <field name="name">Exchange Gain or Loss</field>
            <field name="code">502003</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_854">
            <field name="name">Other Expense</field>
            <field name="code">502004</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

        <record model="account.account.template" id="account_account_855">
            <field name="name">Penalties and Settlements</field>
            <field name="code">502005</field>
            <field name="user_type_id" ref="account.data_account_type_expenses" />
            <field name="chart_template_id" ref="sg_chart_template"/>
            <field name="reconcile" eval="False" />
        </record>

    <record id="sg_chart_template" model="account.chart.template">
        <field name="property_account_receivable_id" ref="account_account_735" />
        <field name="property_account_payable_id" ref="account_account_777" />
        <field name="property_account_expense_categ_id" ref="account_account_819" />
        <field name="property_account_income_categ_id" ref="account_account_803" />
        <field name="income_currency_exchange_account_id" ref="account_account_853"/>
        <field name="expense_currency_exchange_account_id" ref="account_account_853"/>
        <field name="default_pos_receivable_account_id" ref="account_account_737" />
    </record>
</odoo>

```

## File: migrations\2.1\post-migrate_update_tax.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.addons.account.models.chart_template import update_taxes_from_templates


def migrate(cr, version):
    update_taxes_from_templates(cr, 'l10n_sg.sg_chart_template')

```

## File: migrations\2.2\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.addons.account.models.chart_template import update_taxes_from_templates


def migrate(cr, version):
    update_taxes_from_templates(cr, 'l10n_sg.sg_chart_template')

```

## File: migrations\9.0.2.0\pre-set_tags_and_taxes_updatable.py

```python
from openerp.modules.registry import RegistryManager

def migrate(cr, version):
    registry = RegistryManager.get(cr.dbname)
    from openerp.addons.account.models.chart_template import migrate_set_tags_and_taxes_updatable
    migrate_set_tags_and_taxes_updatable(cr, registry, 'l10n_sg')

```

## File: models\account_move.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models


class AccountMove(models.Model):
    _inherit = 'account.move'

    l10n_sg_permit_number = fields.Char(string="Permit No.")

    l10n_sg_permit_number_date = fields.Date(string="Date of permit number")

```

## File: models\res_company.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models

class ResCompany(models.Model):
    _name = 'res.company'
    _description = 'Companies'
    _inherit = 'res.company'

    l10n_sg_unique_entity_number = fields.Char(string='UEN', related="partner_id.l10n_sg_unique_entity_number", readonly=False)

```

## File: models\res_partner.py

```python
# -*- coding: utf-8 -*-

from odoo import fields, models

class ResPartner(models.Model):
    _name = 'res.partner'
    _inherit = 'res.partner'

    l10n_sg_unique_entity_number = fields.Char(string='UEN')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2014 Tech Receptives (<http://techreceptives.com>).

from . import account_move
from . import res_company
from . import res_partner

```

## File: views\account_invoice_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="view_invoice_form_l10n_sg" model="ir.ui.view">
            <field name="name">l10n_sg.invoice.form</field>
            <field name="model">account.move</field>
            <field name="inherit_id" ref="account.view_move_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='fiscal_position_id']" position="after">
                    <field name="l10n_sg_permit_number"/>
                    <field name="l10n_sg_permit_number_date"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\res_company_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="view_company_form_l10n_sg" model="ir.ui.view">
            <field name="name">l10n_sg.company.form</field>
            <field name="model">res.company</field>
            <field name="inherit_id" ref="base.view_company_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='email']" position="after">
                  <field name="l10n_sg_unique_entity_number" attrs="{'invisible': [('country_code', '!=', 'SG')]}"/>
                </xpath>
                <xpath expr="//field[@name='vat']" position="attributes">
                  <attribute name="string" >GST No.</attribute>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

## File: views\res_partner_view.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<odoo>
    <data>
        <record id="view_partner_form_l10n_sg" model="ir.ui.view">
            <field name="name">l10n_sg.partner.form</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='vat']" position="before">
                  <field name="l10n_sg_unique_entity_number"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>

```

