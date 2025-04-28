# Odoo Module: l10n_ca

Category: Localization

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

# Copyright (C) 2010 Savoir-faire Linux (<https://www.savoirfairelinux.com>).

from odoo import api, SUPERUSER_ID
from . import models


def load_translations(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref('l10n_ca.ca_en_chart_template_en').process_coa_translations()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Canada - Accounting',
    'author': 'Savoir-faire Linux',
    'website': 'https://www.savoirfairelinux.com',
    'category': 'Localization',
    'description': """
This is the module to manage the Canadian accounting chart in Odoo.
===========================================================================================

Canadian accounting charts and localizations.

Fiscal positions
----------------

When considering taxes to be applied, it is the province where the delivery occurs that matters.
Therefore we decided to implement the most common case in the fiscal positions: delivery is the
responsibility of the vendor and done at the customer location.

Some examples:

1) You have a customer from another province and you deliver to his location.
On the customer, set the fiscal position to his province.

2) You have a customer from another province. However this customer comes to your location
with their truck to pick up products. On the customer, do not set any fiscal position.

3) An international vendor doesn't charge you any tax. Taxes are charged at customs
by the customs broker. On the vendor, set the fiscal position to International.

4) An international vendor charge you your provincial tax. They are registered with your
position.
    """,
    'depends': [
        'account',
        'base_iban',
        'l10n_multilang',
    ],
    'data': [
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/account_chart_template_after_data.xml',
        'data/account_data.xml',
        'data/account_tax_data.xml',
        'data/fiscal_templates_data.xml',
        'data/account_chart_template_configure_data.xml',
        'data/res_company_data.xml',
        'views/res_partner_view.xml',
        'views/report_invoice.xml',
    ],
    'post_init_hook': 'load_translations',
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","reconcile"
"chart1141_en","Stock In Hand","1141","account.data_account_type_current_assets","l10n_ca.ca_en_chart_template_en","False"
"chart1145_en","Stock Delivered But Not Billed","1145","account.data_account_type_current_assets","l10n_ca.ca_en_chart_template_en","True"
"chart1151_en","Customers Account","1151","account.data_account_type_receivable","l10n_ca.ca_en_chart_template_en","True"
"chart11511_en","Customers Account (PoS)","11511","account.data_account_type_receivable","l10n_ca.ca_en_chart_template_en","True"
"chart1181_en","GST receivable","1181","account.data_account_type_current_assets","l10n_ca.ca_en_chart_template_en","False"
"chart1182_en","PST/QST receivable","1182","account.data_account_type_current_assets","l10n_ca.ca_en_chart_template_en","False"
"chart11831_en","HST receivable - 13%","11831","account.data_account_type_current_assets","l10n_ca.ca_en_chart_template_en","False"
"chart11832_en","HST receivable - 14%","11832","account.data_account_type_current_assets","l10n_ca.ca_en_chart_template_en","False"
"chart11833_en","HST receivable - 15%","11833","account.data_account_type_current_assets","l10n_ca.ca_en_chart_template_en","False"
"chart2111_en","Vendors Account","2111","account.data_account_type_payable","l10n_ca.ca_en_chart_template_en","True"
"chart2131_en","GST to pay","2131","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart2132_en","PST/QST to pay","2132","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart21331_en","HST to pay - 13%","21331","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart21332_en","HST to pay - 14%","21332","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart21333_en","HST to pay - 15%","21333","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart2141_en","CANADA REVENUE AGENCY","2141","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart214111_en","EI - Employees Contribution","214111","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart214112_en","EI - Employer Contribution","214112","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart21412_en","Federal Income Tax","21412","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart214131_en","CPP - Employees Contribution","214131","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart214132_en","CPP - Employer Contribution","214132","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart21421_en","Health Services Fund to pay","21421","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart214221_en","Provincial Pension Plan - Employees Contribution","214221","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart214222_en","Provincial Pension Plan - Employer Contribution","214222","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart214231_en","Parental Insurance Plan - Employee Contribution","214231","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart214232_en","Parental Insurance Plan - Employer Contribution","214232","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart21424_en","Labour Health and Safety to pay","21424","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart21425_en","Labour Standards to pay","21425","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart21426_en","Provincial Income Tax","21426","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart2171_en","Stock Received But Not Billed","2171","account.data_account_type_current_assets","l10n_ca.ca_en_chart_template_en","True"
"chart2181_en","Salaries to pay","2181","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart2183_en","Bonus to pay","2183","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart2184_en","Retroactive Payment to pay","2184","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart218501_en","Group Pension Plan to pay - Employees Contribution","218501","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart218502_en","Group Pension Plan to pay - Employer Contribution","218502","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart218601_en","Employee Benefits Provision - Employees Contribution","218601","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart218602_en","Employee Benefits Provision - Employer Contribution","218602","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart2521_en","Provision for pension plans","2521","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart254101_en","Vacations Accrued","254101","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart254102_en","Compensatory Days Accrued","254102","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart254103_en","Sick Leaves Accrued","254103","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart2542_en","Bonus Accrued","2542","account.data_account_type_current_liabilities","l10n_ca.ca_en_chart_template_en","False"
"chart411_en","Inside Sales","411","account.data_account_type_revenue","l10n_ca.ca_en_chart_template_en","False"
"chart412_en","Harmonized Provinces Sales","412","account.data_account_type_revenue","l10n_ca.ca_en_chart_template_en","False"
"chart413_en","Non-Harmonized Provinces Sales","413","account.data_account_type_revenue","l10n_ca.ca_en_chart_template_en","False"
"chart414_en","International Sales","414","account.data_account_type_revenue","l10n_ca.ca_en_chart_template_en","False"
"chart42_en","NON-OPERATING INCOMES","42","account.data_account_type_other_income","l10n_ca.ca_en_chart_template_en","False"
"chart5111_en","Inside Purchases","5111","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart5112_en","Purchases in harmonized provinces","5112","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart5113_en","Purchases in non-harmonized provinces","5113","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart5114_en","International Purchases","5114","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512101_en","Regular Salaries","512101","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512102_en","Bonus","512102","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512103_en","Retroactive Pay","512103","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512201_en","Vacations Accrued","512201","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512202_en","Compensatory Days Accrued","512202","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512203_en","Sick Leaves Accrued","512203","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512301_en","Canada Pension Plan","512301","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512302_en","Employment Insurance","512302","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512303_en","Group Pension Plan","512303","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512304_en","Employee benefits expense","512304","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512310_en","Provincial Pension Plan","512310","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512311_en","Provincial Parental Insurance Plan","512311","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512312_en","Labour Health and Safety","512312","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512313_en","Labour Standards","512313","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart512314_en","Health Service Fund","512314","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"
"chart55_en","NON-OPERATING EXPENSES","55","account.data_account_type_expenses","l10n_ca.ca_en_chart_template_en","False"

```

## File: data\account_chart_template_after_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Chart template for en -->
    <record id="ca_en_chart_template_en" model="account.chart.template">
        <field name="property_account_receivable_id" ref="chart1151_en"/>
        <field name="property_account_payable_id" ref="chart2111_en"/>
        <field name="property_account_income_categ_id" ref="chart411_en"/>
        <field name="property_account_expense_categ_id" ref="chart5111_en"/>
        <field name="property_stock_account_input_categ_id" ref="chart2171_en"/>
        <field name="property_stock_account_output_categ_id" ref="chart1145_en"/>
        <field name="property_stock_valuation_account_id" ref="chart1141_en"/>
        <field name="income_currency_exchange_account_id" ref="chart42_en"/>
        <field name="expense_currency_exchange_account_id" ref="chart55_en"/>
        <field name="default_pos_receivable_account_id" ref="chart11511_en" />
    </record>
</odoo>

```

## File: data\account_chart_template_configure_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_ca.ca_en_chart_template_en')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Chart template for en -->

    <record id="ca_en_chart_template_en" model="account.chart.template">
        <field name="name">Canada - Chart of Accounts</field>
        <field name="cash_account_code_prefix">111</field>
        <field name="bank_account_code_prefix">112</field>
        <field name="transfer_account_code_prefix">113</field>
        <field name="currency_id" ref="base.CAD"/>
        <field name="use_anglo_saxon" eval="True"/>
        <field name="spoken_languages" eval="'fr_FR;fr_CA'"/>
    </record>
</odoo>

```

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Account Tax Group -->
        <record id="tax_group_fix" model="account.tax.group">
            <field name="name">Taxes</field>
        </record>
        <record id="tax_group_gst_5" model="account.tax.group">
            <field name="name">GST 5%</field>
        </record>
        <record id="tax_group_pst_5" model="account.tax.group">
            <field name="name">PST 5%</field>
        </record>
        <record id="tax_group_gst_7" model="account.tax.group">
            <field name="name">GST 7%</field>
        </record>
        <record id="tax_group_gst_8" model="account.tax.group">
            <field name="name">GST 8%</field>
        </record>
        <record id="tax_group_pst_8" model="account.tax.group">
            <field name="name">PST 8%</field>
        </record>
        <record id="tax_group_qst_9975" model="account.tax.group">
            <field name="name">QST 9.975%</field>
        </record>
        <record id="tax_group_hst_13" model="account.tax.group">
            <field name="name">HST 13%</field>
        </record>
        <record id="tax_group_hst_14" model="account.tax.group">
            <field name="name">HST 14%</field>
        </record>
        <record id="tax_group_hst_15" model="account.tax.group">
            <field name="name">HST 15%</field>
        </record>

    </data>
</odoo>

```

## File: data\account_tax_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

    <!-- SALES TAXES -->

    <!-- British Columbia PST -->

    <record id="gstpst_sale_bc_gst_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST for sales - 5% (BC)</field>
        <field name="description">GST 5%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="sequence">1</field>
        <field name="include_base_amount" eval="False"/>
        <field name="tax_group_id" ref="tax_group_gst_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart2131_en'),
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
                'account_id': ref('chart2131_en'),
            }),
        ]"/>
    </record>

    <record id="pst_bc_sale_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">PST for sales - 7% (BC)</field>
        <field name="description">PST 7%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="sequence">2</field>
        <field name="tax_group_id" ref="tax_group_gst_7"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart2132_en'),
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
                'account_id': ref('chart2132_en'),
            }),
        ]"/>
    </record>

    <record id="gstpst_bc_sale_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST + PST for sales (BC)</field>
        <field name="description">GST + PST</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">100</field>
        <field name="amount_type">group</field>
        <field name="children_tax_ids" eval="[(6,0,[ref('pst_bc_sale_en'), ref('gstpst_sale_bc_gst_en')])]"/>
        <field name="tax_group_id" ref="tax_group_fix"/>
    </record>

    <!-- Manitoba PST -->

    <record id="gstpst_sale_mb_gst_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST for sales - 5% (MB)</field>
        <field name="description">GST 5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="sequence">1</field>
        <field name="include_base_amount" eval="False"/>
        <field name="tax_group_id" ref="tax_group_gst_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart2131_en'),
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
                'account_id': ref('chart2131_en'),
            }),
        ]"/>
    </record>

    <record id="pst_mb_sale_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">PST for sales - 8% (MB)</field>
        <field name="description">PST 8%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="sequence">2</field>
        <field name="tax_group_id" ref="tax_group_gst_8"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart2132_en'),
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
                'account_id': ref('chart2132_en'),
            }),
        ]"/>
    </record>

    <record id="gstpst_mb_sale_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST + PST for sales (MB)</field>
        <field name="description">GST + PST</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">100</field>
        <field name="amount_type">group</field>
        <field name="children_tax_ids" eval="[(6,0,[ref('gstpst_sale_mb_gst_en'), ref('pst_mb_sale_en')])]"/>
        <field name="tax_group_id" ref="tax_group_fix"/>
    </record>

    <!-- Quebec PST -->

    <record id="gstqst_sale_gst_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST for sales - 5% (QC)</field>
        <field name="description">GST 5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="sequence">1</field>
        <field name="tax_group_id" ref="tax_group_gst_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart2131_en'),
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
                'account_id': ref('chart2131_en'),
            }),
        ]"/>
    </record>

    <record id="qst_sale_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">QST for sales - 9.975%</field>
        <field name="description">QST 9.975%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">9.9750</field>
        <field name="amount_type">percent</field>
        <field name="sequence">2</field>
        <field name="tax_group_id" ref="tax_group_qst_9975"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart2132_en'),
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
                'account_id': ref('chart2132_en'),
            }),
        ]"/>
    </record>

    <record id="gstqst_sale_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST + QST for sales</field>
        <field name="description">GST + QST</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">100</field>
        <field name="amount_type">group</field>
        <field name="children_tax_ids" eval="[(6,0,[ref('gstqst_sale_gst_en'), ref('qst_sale_en')])]"/>
        <field name="tax_group_id" ref="tax_group_fix"/>
    </record>

    <!-- Saskatchewan PST -->

    <record id="gstpst_sale_sk_gst_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST for sales - 5% (SK)</field>
        <field name="description">GST 5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="sequence">1</field>
        <field name="include_base_amount" eval="False"/>
        <field name="tax_group_id" ref="tax_group_gst_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart2131_en'),
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
                'account_id': ref('chart2131_en'),
            }),
        ]"/>
    </record>

    <record id="pst_sk_sale_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">PST for sales - 5% (SK)</field>
        <field name="description">PST 5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="sequence">2</field>
        <field name="tax_group_id" ref="tax_group_pst_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart2132_en'),
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
                'account_id': ref('chart2132_en'),
            }),
        ]"/>
    </record>

    <record id="gstpst_sk_sale_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST + PST for sales (SK)</field>
        <field name="description">GST + PST</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">100</field>
        <field name="amount_type">group</field>
        <field name="children_tax_ids" eval="[(6,0,[ref('gstpst_sale_sk_gst_en'), ref('pst_sk_sale_en')])]"/>
        <field name="tax_group_id" ref="tax_group_fix"/>
    </record>

    <!-- HST -->

    <record id="hst13_sale_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">HST for sales - 13%</field>
        <field name="description">HST 13%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_hst_13"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart21331_en'),
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
                'account_id': ref('chart21331_en'),
            }),
        ]"/>
    </record>

    <record id="hst14_sale_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">HST for sales - 14%</field>
        <field name="description">HST 14%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_hst_14"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart21332_en'),
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
                'account_id': ref('chart21332_en'),
            }),
        ]"/>
    </record>

    <record id="hst15_sale_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">HST for sales - 15%</field>
        <field name="description">HST 15%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">15</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_hst_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart21333_en'),
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
                'account_id': ref('chart21333_en'),
            }),
        ]"/>
    </record>

    <!-- GST -->

    <record id="gst_sale_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST for sales - 5%</field>
        <field name="description">GST 5%</field>
        <field name="type_tax_use">sale</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_gst_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart2131_en'),
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
                'account_id': ref('chart2131_en'),
            }),
        ]"/>
    </record>

    <!-- PURCHASE TAXES -->

    <!-- British Columbia PST -->

    <record id="gstpst_purc_bc_gst_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST for purchases - 5% (BC)</field>
        <field name="description">GST 5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="sequence">1</field>
        <field name="include_base_amount" eval="False"/>
        <field name="tax_group_id" ref="tax_group_gst_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart1181_en'),
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
                'account_id': ref('chart1181_en'),
            }),
        ]"/>
    </record>

    <record id="pst_bc_purc_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">PST for purchases - 7% (BC)</field>
        <field name="description">PST 7%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">7</field>
        <field name="amount_type">percent</field>
        <field name="sequence">2</field>
        <field name="tax_group_id" ref="tax_group_gst_7"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart1182_en'),
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
                'account_id': ref('chart1182_en'),
            }),
        ]"/>
    </record>

    <record id="gstpst_bc_purc_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST + PST for purchases (BC)</field>
        <field name="description">GST + PST</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">100</field>
        <field name="amount_type">group</field>
        <field name="children_tax_ids" eval="[(6,0,[ref('gstpst_purc_bc_gst_en'), ref('pst_bc_purc_en')])]"/>
        <field name="tax_group_id" ref="tax_group_fix"/>
    </record>

    <!-- Manitoba PST -->

    <record id="gstpst_purc_mb_gst_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST for purchases - 5% (MB)</field>
        <field name="description">GST 5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="sequence">1</field>
        <field name="include_base_amount" eval="False"/>
        <field name="tax_group_id" ref="tax_group_gst_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart1181_en'),
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
                'account_id': ref('chart1181_en'),
            }),
        ]"/>
    </record>

    <record id="pst_mb_purc_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">PST for purchases - 8% (MB)</field>
        <field name="description">PST 8%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">8</field>
        <field name="amount_type">percent</field>
        <field name="sequence">2</field>
        <field name="tax_group_id" ref="tax_group_pst_8"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart1182_en'),
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
                'account_id': ref('chart1182_en'),
            }),
        ]"/>
    </record>

    <record id="gstpst_mb_purc_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST + PST for purchases (MB)</field>
        <field name="description">GST + PST</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">100</field>
        <field name="amount_type">group</field>
        <field name="children_tax_ids" eval="[(6,0,[ref('gstpst_purc_mb_gst_en'), ref('pst_mb_purc_en')])]"/>
        <field name="tax_group_id" ref="tax_group_fix"/>
    </record>

    <!-- Quebec PST -->

    <record id="gstqst_purc_gst_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST for purchases - 5% (QC)</field>
        <field name="description">GST 5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="sequence">1</field>
        <field name="tax_group_id" ref="tax_group_gst_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart1181_en'),
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
                'account_id': ref('chart1181_en'),
            }),
        ]"/>
    </record>

    <record id="qst_purc_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">QST for purchases - 9.975%</field>
        <field name="description">QST 9.975%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">9.9750</field>
        <field name="amount_type">percent</field>
        <field name="sequence">2</field>
        <field name="tax_group_id" ref="tax_group_qst_9975"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart1182_en'),
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
                'account_id': ref('chart1182_en'),
            }),
        ]"/>
    </record>

    <record id="gstqst_purc_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST + QST for purchases</field>
        <field name="description">GST + QST</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">100</field>
        <field name="amount_type">group</field>
        <field name="children_tax_ids" eval="[(6,0,[ref('gstqst_purc_gst_en'), ref('qst_purc_en')])]"/>
        <field name="tax_group_id" ref="tax_group_fix"/>
    </record>

    <!-- Saskatchewan PST -->

    <record id="gstpst_purc_sk_gst_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST for purchases - 5% (SK)</field>
        <field name="description">GST 5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="sequence">1</field>
        <field name="include_base_amount" eval="False"/>
        <field name="tax_group_id" ref="tax_group_gst_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart1181_en'),
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
                'account_id': ref('chart1181_en'),
            }),
        ]"/>
    </record>

    <record id="pst_sk_purc_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">PST for purchases - 5% (SK)</field>
        <field name="description">PST 5%</field>
        <field name="type_tax_use">none</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="sequence">2</field>
        <field name="tax_group_id" ref="tax_group_pst_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart1182_en'),
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
                'account_id': ref('chart1182_en'),
            }),
        ]"/>
    </record>

    <record id="gstpst_sk_purc_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST + PST for purchases (SK)</field>
        <field name="description">GST + PST</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">1</field>
        <field name="amount_type">group</field>
        <field name="children_tax_ids" eval="[(6,0,[ref('gstpst_purc_sk_gst_en'), ref('pst_sk_purc_en')])]"/>
        <field name="tax_group_id" ref="tax_group_fix"/>
    </record>

    <!-- HST -->

    <record id="hst13_purc_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">HST for purchases - 13%</field>
        <field name="description">HST 13%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">13</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_hst_13"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart11831_en'),
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
                'account_id': ref('chart11831_en'),
            }),
        ]"/>
    </record>

    <record id="hst14_purc_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">HST for purchases - 14%</field>
        <field name="description">HST 14%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">14</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_hst_14"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart11832_en'),
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
                'account_id': ref('chart11832_en'),
            }),
        ]"/>
    </record>

    <record id="hst15_purc_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">HST for purchases - 15%</field>
        <field name="description">HST 15%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">15</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_hst_15"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart11833_en'),
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
                'account_id': ref('chart11833_en'),
            }),
        ]"/>
    </record>

    <!-- GST -->

    <record id="gst_purc_en" model="account.tax.template">
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="name">GST for purchases - 5%</field>
        <field name="description">GST 5%</field>
        <field name="type_tax_use">purchase</field>
        <field name="amount">5</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="tax_group_gst_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),

            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('chart1181_en'),
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
                'account_id': ref('chart1181_en'),
            }),
        ]"/>
    </record>

    </data>
</odoo>

```

## File: data\fiscal_templates_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <!-- Fiscal Position Templates -->

    <record id="fiscal_position_template_ab_en" model="account.fiscal.position.template">
        <field name="name">Alberta (AB)</field>
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ca"/>
        <field name="state_ids" eval="[(6, 0, [ref('base.state_ca_ab')])]"/>
    </record>

    <record id="fiscal_position_template_bc_en" model="account.fiscal.position.template">
        <field name="name">British Columbia (BC)</field>
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ca"/>
        <field name="state_ids" eval="[(6, 0, [ref('base.state_ca_bc')])]"/>
    </record>

    <record id="fiscal_position_template_mb_en" model="account.fiscal.position.template">
        <field name="name">Manitoba (MB)</field>
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ca"/>
        <field name="state_ids" eval="[(6, 0, [ref('base.state_ca_mb')])]"/>
    </record>

    <record id="fiscal_position_template_nb_en" model="account.fiscal.position.template">
        <field name="name">New Brunswick (NB)</field>
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ca"/>
        <field name="state_ids" eval="[(6, 0, [ref('base.state_ca_nb')])]"/>
    </record>

    <record id="fiscal_position_template_nl_en" model="account.fiscal.position.template">
        <field name="name">Newfoundland and Labrador (NL)</field>
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ca"/>
        <field name="state_ids" eval="[(6, 0, [ref('base.state_ca_nl')])]"/>
    </record>

    <record id="fiscal_position_template_ns_en" model="account.fiscal.position.template">
        <field name="name">Nova Scotia (NS)</field>
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ca"/>
        <field name="state_ids" eval="[(6, 0, [ref('base.state_ca_ns')])]"/>
    </record>

    <record id="fiscal_position_template_nt_en" model="account.fiscal.position.template">
        <field name="name">Northwest Territories (NT)</field>
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ca"/>
        <field name="state_ids" eval="[(6, 0, [ref('base.state_ca_nt')])]"/>
    </record>

    <record id="fiscal_position_template_nu_en" model="account.fiscal.position.template">
        <field name="name">Nunavut (NU)</field>
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ca"/>
        <field name="state_ids" eval="[(6, 0, [ref('base.state_ca_nu')])]"/>
    </record>

    <record id="fiscal_position_template_on_en" model="account.fiscal.position.template">
        <field name="name">Ontario (ON)</field>
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ca"/>
        <field name="state_ids" eval="[(6, 0, [ref('base.state_ca_on')])]"/>
    </record>

    <record id="fiscal_position_template_pe_en" model="account.fiscal.position.template">
        <field name="name">Prince Edward Islands (PE)</field>
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ca"/>
        <field name="state_ids" eval="[(6, 0, [ref('base.state_ca_pe')])]"/>
    </record>

    <record id="fiscal_position_template_qc_en" model="account.fiscal.position.template">
        <field name="name">Quebec (QC)</field>
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ca"/>
        <field name="state_ids" eval="[(6, 0, [ref('base.state_ca_qc')])]"/>
    </record>

    <record id="fiscal_position_template_sk_en" model="account.fiscal.position.template">
        <field name="name">Saskatchewan (SK)</field>
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ca"/>
        <field name="state_ids" eval="[(6, 0, [ref('base.state_ca_sk')])]"/>
    </record>

    <record id="fiscal_position_template_yt_en" model="account.fiscal.position.template">
        <field name="name">Yukon (YT)</field>
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="auto_apply" eval="True"/>
        <field name="country_id" ref="base.ca"/>
        <field name="state_ids" eval="[(6, 0, [ref('base.state_ca_yt')])]"/>
    </record>

    <record id="fiscal_position_template_intl_en" model="account.fiscal.position.template">
        <field name="sequence">1</field>
        <field name="name">International (INTL)</field>
        <field name="chart_template_id" ref="ca_en_chart_template_en"/>
        <field name="auto_apply" eval="True"/>
    </record>

    <!--  Company is in Alberta (default is gst) -->

    <record id="fiscal_position_tax_template_ab2nb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ab2nl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ab2ns_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst15_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ab2on_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ab2pe_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst14_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ab2intl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
    </record>

    <!-- Purchases -->

    <record id="fiscal_position_tax_template_intl2ab_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gst_purc_en"/>
    </record>

    <!-- Company is in British Columbia (default is gstpst_bc) -->

    <!-- Sale taxes -->

    <record id="fiscal_position_tax_template_bc2ab_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="tax_src_id" ref="gstpst_bc_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2mb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_mb_en"/>
        <field name="tax_src_id" ref="gstpst_bc_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2nb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="tax_src_id" ref="gstpst_bc_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2nl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="gstpst_bc_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2ns_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="gstpst_bc_sale_en"/>
        <field name="tax_dest_id" ref="hst15_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2nt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="tax_src_id" ref="gstpst_bc_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2nu_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="tax_src_id" ref="gstpst_bc_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2on_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="tax_src_id" ref="gstpst_bc_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2pe_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="gstpst_bc_sale_en"/>
        <field name="tax_dest_id" ref="hst14_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2qc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_qc_en"/>
        <field name="tax_src_id" ref="gstpst_bc_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2sk_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_sk_en"/>
        <field name="tax_src_id" ref="gstpst_bc_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2yt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="tax_src_id" ref="gstpst_bc_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2intl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gstpst_bc_sale_en"/>
    </record>

    <!-- Purchase taxes -->

    <record id="fiscal_position_tax_template_ab2bc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="tax_src_id" ref="gstpst_bc_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2bc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_mb_en"/>
        <field name="tax_src_id" ref="gstpst_bc_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2bc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="tax_src_id" ref="gstpst_bc_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nl2bc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="gstpst_bc_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2bc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="gstpst_bc_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nt2bc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="tax_src_id" ref="gstpst_bc_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nu2bc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="tax_src_id" ref="gstpst_bc_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_on2bc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="tax_src_id" ref="gstpst_bc_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2bc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="gstpst_bc_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2bc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_qc_en"/>
        <field name="tax_src_id" ref="gstpst_bc_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2bc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_sk_en"/>
        <field name="tax_src_id" ref="gstpst_bc_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_yt2bc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="tax_src_id" ref="gstpst_bc_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_intl2bc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gstpst_bc_purc_en"/>
    </record>

    <!-- Company is in Manitoba (default is gstpst_mb) -->

    <!-- Sale Taxes -->

    <record id="fiscal_position_tax_template_mb2ab_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="tax_src_id" ref="gstpst_mb_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2bc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_bc_en"/>
        <field name="tax_src_id" ref="gstpst_mb_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2nb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="tax_src_id" ref="gstpst_mb_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2nl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="gstpst_mb_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2ns_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="gstpst_mb_sale_en"/>
        <field name="tax_dest_id" ref="hst15_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2nt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="tax_src_id" ref="gstpst_mb_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2nu_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="tax_src_id" ref="gstpst_mb_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2on_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="tax_src_id" ref="gstpst_mb_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2pe_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="gstpst_mb_sale_en"/>
        <field name="tax_dest_id" ref="hst14_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2qc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_qc_en"/>
        <field name="tax_src_id" ref="gstpst_mb_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2sk_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_sk_en"/>
        <field name="tax_src_id" ref="gstpst_mb_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2yt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="tax_src_id" ref="gstpst_mb_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2intl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gstpst_mb_sale_en"/>
    </record>

    <!-- Purchase Taxes -->

    <record id="fiscal_position_tax_template_ab2mb_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="tax_src_id" ref="gstpst_mb_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2mb_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_bc_en"/>
        <field name="tax_src_id" ref="gstpst_mb_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2mb_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="tax_src_id" ref="gstpst_mb_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nl2mb_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="gstpst_mb_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2mb_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="gstpst_mb_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nt2mb_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="tax_src_id" ref="gstpst_mb_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nu2mb_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="tax_src_id" ref="gstpst_mb_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_on2mb_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="tax_src_id" ref="gstpst_mb_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2mb_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="gstpst_mb_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2mb_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_qc_en"/>
        <field name="tax_src_id" ref="gstpst_mb_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2mb_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_sk_en"/>
        <field name="tax_src_id" ref="gstpst_mb_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_yt2mb_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="tax_src_id" ref="gstpst_mb_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_yt2intl_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gstpst_mb_purc_en"/>
    </record>

    <!--  Company is in New Brunswick (default is hst13) -->

    <record id="fiscal_position_tax_template_nb2ab_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2bc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_bc_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2mb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_mb_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2nl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2ns_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="hst15_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2nt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2nu_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2pe_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="hst14_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2qc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_qc_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2sk_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_sk_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2yt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2intl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
    </record>

    <!-- Purchases -->

    <record id="fiscal_position_tax_template_intl2nb_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="hst13_purc_en"/>
    </record>

    <!--  Company is in Newfoundland and Labrador (default is hst13) -->
    
    <!-- Already created by nb2ab_sale
    <record id="fiscal_position_tax_template_nl2ab_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created by nb2bc_sale
    <record id="fiscal_position_tax_template_nl2bc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_bc_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created by nb2mb_sale
    <record id="fiscal_position_tax_template_nl2mb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_mb_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already creted by nb2ns_sale
    <record id="fiscal_position_tax_template_nl2ns_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="hst15_sale_en"/>
    </record>

    Already creted by nb2nt_sale
    <record id="fiscal_position_tax_template_nl2nt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created nb2nu_sale
    <record id="fiscal_position_tax_template_nl2nu_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created by nb2pe_sale
    <record id="fiscal_position_tax_template_nl2pe_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="hst14_sale_en"/>
    </record>

    Already created by nb2qc_sale
    <record id="fiscal_position_tax_template_nl2qc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_qc_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created by nb2sk_sale
    <record id="fiscal_position_tax_template_nl2sk_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_sk_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created by nb2yt_sale
    <record id="fiscal_position_tax_template_nl2yt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created by nb2intl_sale
    <record id="fiscal_position_tax_template_nl2intl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
    </record>
    -->

    <!-- Purchases -->

    <!-- Already created by intl2nb_purc
    <record id="fiscal_position_tax_template_intl2nl_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="hst13_purc_en"/>
    </record>
    -->

    <!--  Company is in Nova Scotia (default is hst15) -->

    <record id="fiscal_position_tax_template_ns2ab_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="tax_src_id" ref="hst15_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2bc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_bc_en"/>
        <field name="tax_src_id" ref="hst15_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2mb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_mb_en"/>
        <field name="tax_src_id" ref="hst15_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2nb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="tax_src_id" ref="hst15_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2nl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="hst15_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2nt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="tax_src_id" ref="hst15_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2nu_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="tax_src_id" ref="hst15_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2on_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="tax_src_id" ref="hst15_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2pe_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="hst15_sale_en"/>
        <field name="tax_dest_id" ref="hst14_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2qc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_qc_en"/>
        <field name="tax_src_id" ref="hst15_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2sk_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_sk_en"/>
        <field name="tax_src_id" ref="hst15_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2yt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="tax_src_id" ref="hst15_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2intl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="hst15_sale_en"/>
    </record>

    <!-- Purchases -->

    <record id="fiscal_position_tax_template_intl2ns_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="hst15_purc_en"/>
    </record>

    <!--  Company is in Northwest Territories (default is gst) -->

    <!-- Already created by ab2nb_sale
    <record id="fiscal_position_tax_template_nt2nb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    Already created by ab2nl_sale
    <record id="fiscal_position_tax_template_nt2nl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    Already created by ab2ns_sale
    <record id="fiscal_position_tax_template_nt2ns_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst15_sale_en"/>
    </record>

    Already created by ab2on_sale
    <record id="fiscal_position_tax_template_nt2on_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    Already created by ab2pe_sale
    <record id="fiscal_position_tax_template_nt2pe_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst14_sale_en"/>
    </record>

    Already created by ab2intl_sale
    <record id="fiscal_position_tax_template_nt2intl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
    </record>
    -->

    <!-- Purchases -->

    <!-- Already created by intl2ab_purc
    <record id="fiscal_position_tax_template_intl2nt_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gst_purc_en"/>
    </record>
    -->

    <!--  Company is in Nunavut (default is gst) -->

    <!-- Already created by ab2nb_sale
    <record id="fiscal_position_tax_template_nu2nb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    Already created by ab2nl_sale
    <record id="fiscal_position_tax_template_nu2nl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    Already created by ab2ns_sale
    <record id="fiscal_position_tax_template_nu2ns_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst15_sale_en"/>
    </record>

    Already created by ab2on_sale
    <record id="fiscal_position_tax_template_nu2on_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    Already created by ab2pe_sale
    <record id="fiscal_position_tax_template_nu2pe_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst14_sale_en"/>
    </record>

    Already created by ab2intl_sale
    <record id="fiscal_position_tax_template_nu2intl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
    </record>
    -->

    <!-- Purchases -->

    <!-- Already created by intl2ab_purc
    <record id="fiscal_position_tax_template_intl2nu_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gst_purc_en"/>
    </record>
    -->

    <!--  Company is in Ontario (default is hst13) -->

    <!-- Already created nb2ab_sale
    <record id="fiscal_position_tax_template_on2ab_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created by nb2bc_sale
    <record id="fiscal_position_tax_template_on2bc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_bc_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created by nb2mb_sale
    <record id="fiscal_position_tax_template_on2mb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_mb_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created by nb2ns_sale
    <record id="fiscal_position_tax_template_on2ns_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="hst15_sale_en"/>
    </record>

    Already created by nb2nt_sale
    <record id="fiscal_position_tax_template_on2nt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created nb2nu_sale
    <record id="fiscal_position_tax_template_on2nu_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created by nb2pe_sale
    <record id="fiscal_position_tax_template_on2pe_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="hst14_sale_en"/>
    </record>

    Already created by nb2qc_sale
    <record id="fiscal_position_tax_template_on2qc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_qc_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created by nb2sk_sale
    <record id="fiscal_position_tax_template_on2sk_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_sk_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created by nb2yt_sale
    <record id="fiscal_position_tax_template_on2yt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    Already created by nb2intl_sale
    <record id="fiscal_position_tax_template_on2intl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="hst13_sale_en"/>
    </record>
    -->

    <!-- Purchases --> 

    <!-- Already created by intl2nb_purc
    <record id="fiscal_position_tax_template_intl2on_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="hst13_purc_en"/>
    </record>
    -->

    <!--  Company is in Prince Edward Islands (default is hst14) -->

    <record id="fiscal_position_tax_template_pe2ab_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="tax_src_id" ref="hst14_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2bc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_bc_en"/>
        <field name="tax_src_id" ref="hst14_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2mb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_mb_en"/>
        <field name="tax_src_id" ref="hst14_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2nb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="tax_src_id" ref="hst14_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2nl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="hst14_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2ns_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="hst14_sale_en"/>
        <field name="tax_dest_id" ref="hst15_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2nt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="tax_src_id" ref="hst14_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2nu_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="tax_src_id" ref="hst14_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2on_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="tax_src_id" ref="hst14_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2qc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_qc_en"/>
        <field name="tax_src_id" ref="hst14_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2sk_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_sk_en"/>
        <field name="tax_src_id" ref="hst14_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2yt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="tax_src_id" ref="hst14_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2intl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="hst14_sale_en"/>
    </record>

    <!-- Purchases -->

    <record id="fiscal_position_tax_template_intl2pe_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="hst14_purc_en"/>
    </record>

    <!--  Company is in Quebec (default is gstqst) -->

    <!-- Sale Taxes -->

    <record id="fiscal_position_tax_template_qc2ab_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="tax_src_id" ref="gstqst_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2bc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_bc_en"/>
        <field name="tax_src_id" ref="gstqst_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2mb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_mb_en"/>
        <field name="tax_src_id" ref="gstqst_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2nb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="tax_src_id" ref="gstqst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2nl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="gstqst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2ns_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="gstqst_sale_en"/>
        <field name="tax_dest_id" ref="hst15_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2nt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="tax_src_id" ref="gstqst_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2nu_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="tax_src_id" ref="gstqst_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2on_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="tax_src_id" ref="gstqst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2pe_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="gstqst_sale_en"/>
        <field name="tax_dest_id" ref="hst14_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2sk_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_sk_en"/>
        <field name="tax_src_id" ref="gstqst_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2yt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="tax_src_id" ref="gstqst_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2intl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gstqst_sale_en"/>
    </record>

    <!-- Purchase Taxes -->

    <record id="fiscal_position_tax_template_ab2qc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="tax_src_id" ref="gstqst_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2qc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_bc_en"/>
        <field name="tax_src_id" ref="gstqst_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2qc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_mb_en"/>
        <field name="tax_src_id" ref="gstqst_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2qc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="tax_src_id" ref="gstqst_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nl2qc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="gstqst_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2qc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="gstqst_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nt2qc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="tax_src_id" ref="gstqst_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nu2qc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="tax_src_id" ref="gstqst_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_on2qc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="tax_src_id" ref="gstqst_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2qc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="gstqst_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2qc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_sk_en"/>
        <field name="tax_src_id" ref="gstqst_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_yt2qc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="tax_src_id" ref="gstqst_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_intl2qc_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gstqst_purc_en"/>
    </record>

    <!--  Company is in Saskatchewan (default is gstpst_sk) -->

    <!-- Sale Taxes -->

    <record id="fiscal_position_tax_template_sk2ab_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="tax_src_id" ref="gstpst_sk_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2bc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_bc_en"/>
        <field name="tax_src_id" ref="gstpst_sk_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2mb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_mb_en"/>
        <field name="tax_src_id" ref="gstpst_sk_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2nb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="tax_src_id" ref="gstpst_sk_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2nl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="gstpst_sk_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2ns_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="gstpst_sk_sale_en"/>
        <field name="tax_dest_id" ref="hst15_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2nt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="tax_src_id" ref="gstpst_sk_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2nu_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="tax_src_id" ref="gstpst_sk_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2on_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="tax_src_id" ref="gstpst_sk_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2pe_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="gstpst_sk_sale_en"/>
        <field name="tax_dest_id" ref="hst14_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2qc_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_qc_en"/>
        <field name="tax_src_id" ref="gstpst_sk_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2yt_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="tax_src_id" ref="gstpst_sk_sale_en"/>
        <field name="tax_dest_id" ref="gst_sale_en"/>
    </record>

    <record id="fiscal_position_tax_template_sk2intl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gstpst_sk_sale_en"/>
    </record>

    <!-- Purchase Taxes -->

    <record id="fiscal_position_tax_template_ab2sk_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="tax_src_id" ref="gstpst_sk_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_bc2sk_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_bc_en"/>
        <field name="tax_src_id" ref="gstpst_sk_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_mb2sk_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_mb_en"/>
        <field name="tax_src_id" ref="gstpst_sk_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nb2sk_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="tax_src_id" ref="gstpst_sk_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nl2sk_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="gstpst_sk_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_ns2sk_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="gstpst_sk_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nt2sk_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="tax_src_id" ref="gstpst_sk_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_nu2sk_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="tax_src_id" ref="gstpst_sk_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_on2sk_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="tax_src_id" ref="gstpst_sk_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_pe2sk_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="gstpst_sk_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_qc2sk_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_qc_en"/>
        <field name="tax_src_id" ref="gstpst_sk_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_yt2sk_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="tax_src_id" ref="gstpst_sk_purc_en"/>
        <field name="tax_dest_id" ref="gst_purc_en"/>
    </record>

    <record id="fiscal_position_tax_template_intl2yt_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gstpst_sk_purc_en"/>
    </record>

    <!--  Company is in Yukon (default is gst) -->

    <!-- Already created by ab2nb_sale
    <record id="fiscal_position_tax_template_yt2nb_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    Already created by ab2nl_sale
    <record id="fiscal_position_tax_template_yt2nl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    Already created by ab2ns_sale
    <record id="fiscal_position_tax_template_yt2ns_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst15_sale_en"/>
    </record>

    Already created by ab2on_sale
    <record id="fiscal_position_tax_template_yt2on_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst13_sale_en"/>
    </record>

    Already created by ab2pe_sale
    <record id="fiscal_position_tax_template_yt2pe_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
        <field name="tax_dest_id" ref="hst14_sale_en"/>
    </record>

    Already created by ab2intl_sale
    <record id="fiscal_position_tax_template_yt2intl_sale_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gst_sale_en"/>
    </record>
    -->

    <!-- Purchases -->

    <!-- Already created by intl2ab_purc
    <record id="fiscal_position_tax_template_intl2yt_purc_en" model="account.fiscal.position.tax.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="tax_src_id" ref="gst_purc_en"/>
    </record>
    -->

    <!-- Accounts mapping -->

    <!-- Alberta fiscal position -->

    <record id="fiscal_position_account_template_2ab_sale_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="account_src_id" ref="chart411_en"/>
        <field name="account_dest_id" ref="chart413_en"/>
    </record>

    <record id="fiscal_position_account_template_ab2_purc_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_ab_en"/>
        <field name="account_src_id" ref="chart5111_en"/>
        <field name="account_dest_id" ref="chart5113_en"/>
    </record>

    <!-- British Columbia fiscal position -->

    <record id="fiscal_position_account_template_2bc_sale_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_bc_en"/>
        <field name="account_src_id" ref="chart411_en"/>
        <field name="account_dest_id" ref="chart413_en"/>
    </record>

    <record id="fiscal_position_account_template_bc2_purc_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_bc_en"/>
        <field name="account_src_id" ref="chart5111_en"/>
        <field name="account_dest_id" ref="chart5113_en"/>
    </record>

    <!-- Manitoba fiscal position -->

    <record id="fiscal_position_account_template_2mb_sale_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_mb_en"/>
        <field name="account_src_id" ref="chart411_en"/>
        <field name="account_dest_id" ref="chart413_en"/>
    </record>

    <record id="fiscal_position_account_template_mb2_purc_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_mb_en"/>
        <field name="account_src_id" ref="chart5111_en"/>
        <field name="account_dest_id" ref="chart5113_en"/>
    </record>

    <!-- New Brunswick fiscal position -->

    <record id="fiscal_position_account_template_2nb_sale_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="account_src_id" ref="chart411_en"/>
        <field name="account_dest_id" ref="chart412_en"/>
    </record>

    <record id="fiscal_position_account_template_nb2_purc_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_nb_en"/>
        <field name="account_src_id" ref="chart5111_en"/>
        <field name="account_dest_id" ref="chart5112_en"/>
    </record>

    <!-- Newfoundland and Labrador fiscal position -->

    <record id="fiscal_position_account_template_2nl_sale_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="account_src_id" ref="chart411_en"/>
        <field name="account_dest_id" ref="chart412_en"/>
    </record>

    <record id="fiscal_position_account_template_nl2_purc_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_nl_en"/>
        <field name="account_src_id" ref="chart5111_en"/>
        <field name="account_dest_id" ref="chart5112_en"/>
    </record>

    <!-- Nova Scotia fiscal position -->

    <record id="fiscal_position_account_template_2ns_sale_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="account_src_id" ref="chart411_en"/>
        <field name="account_dest_id" ref="chart412_en"/>
    </record>

    <record id="fiscal_position_account_template_ns2_purc_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_ns_en"/>
        <field name="account_src_id" ref="chart5111_en"/>
        <field name="account_dest_id" ref="chart5112_en"/>
    </record>

    <!-- Nunavut fiscal position -->

    <record id="fiscal_position_account_template_2nu_sale_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="account_src_id" ref="chart411_en"/>
        <field name="account_dest_id" ref="chart413_en"/>
    </record>

    <record id="fiscal_position_account_template_nu2_purc_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_nu_en"/>
        <field name="account_src_id" ref="chart5111_en"/>
        <field name="account_dest_id" ref="chart5113_en"/>
    </record>

    <!-- Northwest Territories fiscal position -->

    <record id="fiscal_position_account_template_2nt_sale_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="account_src_id" ref="chart411_en"/>
        <field name="account_dest_id" ref="chart413_en"/>
    </record>

    <record id="fiscal_position_account_template_nt2_purc_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_nt_en"/>
        <field name="account_src_id" ref="chart5111_en"/>
        <field name="account_dest_id" ref="chart5113_en"/>
    </record>

    <!-- Ontario fiscal position -->

    <record id="fiscal_position_account_template_2on_sale_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="account_src_id" ref="chart411_en"/>
        <field name="account_dest_id" ref="chart412_en"/>
    </record>

    <record id="fiscal_position_account_template_on2_purc_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_on_en"/>
        <field name="account_src_id" ref="chart5111_en"/>
        <field name="account_dest_id" ref="chart5112_en"/>
    </record>

    <!-- Prince Edward Islands fiscal position -->

    <record id="fiscal_position_account_template_2pe_sale_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="account_src_id" ref="chart411_en"/>
        <field name="account_dest_id" ref="chart412_en"/>
    </record>

    <record id="fiscal_position_account_template_pe2_purc_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_pe_en"/>
        <field name="account_src_id" ref="chart5111_en"/>
        <field name="account_dest_id" ref="chart5112_en"/>
    </record>

    <!-- Quebec fiscal position -->

    <record id="fiscal_position_account_template_2qc_sale_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_qc_en"/>
        <field name="account_src_id" ref="chart411_en"/>
        <field name="account_dest_id" ref="chart413_en"/>
    </record>

    <record id="fiscal_position_account_template_qc2_purc_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_qc_en"/>
        <field name="account_src_id" ref="chart5111_en"/>
        <field name="account_dest_id" ref="chart5113_en"/>
    </record>

    <!-- Saskatchewan fiscal position -->

    <record id="fiscal_position_account_template_2sk_sale_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_sk_en"/>
        <field name="account_src_id" ref="chart411_en"/>
        <field name="account_dest_id" ref="chart413_en"/>
    </record>

    <record id="fiscal_position_account_template_sk2_purc_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_sk_en"/>
        <field name="account_src_id" ref="chart5111_en"/>
        <field name="account_dest_id" ref="chart5113_en"/>
    </record>

    <!-- Yukon fiscal position -->

    <record id="fiscal_position_account_template_2yt_sale_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="account_src_id" ref="chart411_en"/>
        <field name="account_dest_id" ref="chart413_en"/>
    </record>

    <record id="fiscal_position_account_template_yt2_purc_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_yt_en"/>
        <field name="account_src_id" ref="chart5111_en"/>
        <field name="account_dest_id" ref="chart5113_en"/>
    </record>

    <!-- International fiscal position -->

    <record id="fiscal_position_account_template_ab2intl_sale_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="account_src_id" ref="chart411_en"/>
        <field name="account_dest_id" ref="chart414_en"/>
    </record>

    <record id="fiscal_position_account_template_intl2_purc_en" model="account.fiscal.position.account.template">
        <field name="position_id" ref="fiscal_position_template_intl_en"/>
        <field name="account_src_id" ref="chart5111_en"/>
        <field name="account_dest_id" ref="chart5114_en"/>
    </record>
</odoo>

```

## File: data\res_company_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
<record model="res.company" id="base.main_company">
    <field name="paperformat_id" ref="base.paperformat_us"/>
</record>
</odoo>

```

## File: i18n_extra\l10n_ca.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
#	* l10n_ca
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 10.0c\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2015-11-03 22:23+0000\n"
"PO-Revision-Date: 2015-11-03 22:23+0000\n"
"Last-Translator: <>\n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_ca
#: model:account.fiscal.position.template,name:l10n_ca.fiscal_position_template_ab_en
msgid "Alberta (AB)"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart512102_en
msgid "Bonus"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart2542_en
msgid "Bonus Accrued"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart2183_en
msgid "Bonus to pay"
msgstr ""

#. module: l10n_ca
#: model:account.fiscal.position.template,name:l10n_ca.fiscal_position_template_bc_en
msgid "British Columbia (BC)"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart2141_en
msgid "CANADA REVENUE AGENCY"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart214131_en
msgid "CPP - Employees Contribution"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart214132_en
msgid "CPP - Employer Contribution"
msgstr ""

#. module: l10n_ca
#: model:account.chart.template,name:l10n_ca.ca_en_chart_template_en
msgid "Canada - Chart of Accounts"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart512301_en
msgid "Canada Pension Plan"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart254102_en
#: model:account.account.template,name:l10n_ca.chart512202_en
msgid "Compensatory Days Accrued"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart1151_en
msgid "Customers Account"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart214111_en
msgid "EI - Employees Contribution"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart214112_en
msgid "EI - Employer Contribution"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart218601_en
msgid "Employee Benefits Provision - Employees Contribution"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart218602_en
msgid "Employee Benefits Provision - Employer Contribution"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart512304_en
msgid "Employee benefits expense"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart512302_en
msgid "Employment Insurance"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart21412_en
msgid "Federal Income Tax"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstpst_bc_purc_en
msgid "GST + PST for purchases (BC)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstpst_mb_purc_en
msgid "GST + PST for purchases (MB)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstpst_sk_purc_en
msgid "GST + PST for purchases (SK)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstpst_bc_sale_en
msgid "GST + PST for sales (BC)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstpst_mb_sale_en
msgid "GST + PST for sales (MB)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstpst_sk_sale_en
msgid "GST + PST for sales (SK)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstqst_purc_en
msgid "GST + QST for purchases"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstqst_sale_en
msgid "GST + QST for sales"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gst_purc_en
msgid "GST for purchases - 5%"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstpst_purc_bc_gst_en
msgid "GST for purchases - 5% (BC)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstpst_purc_mb_gst_en
msgid "GST for purchases - 5% (MB)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstqst_purc_gst_en
msgid "GST for purchases - 5% (QC)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstpst_purc_sk_gst_en
msgid "GST for purchases - 5% (SK)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gst_sale_en
msgid "GST for sales - 5%"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstpst_sale_bc_gst_en
msgid "GST for sales - 5% (BC)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstpst_sale_mb_gst_en
msgid "GST for sales - 5% (MB)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstqst_sale_gst_en
msgid "GST for sales - 5% (QC)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.gstpst_sale_sk_gst_en
msgid "GST for sales - 5% (SK)"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart1181_en
msgid "GST receivable"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart2131_en
msgid "GST to pay"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart512303_en
msgid "Group Pension Plan"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart218501_en
msgid "Group Pension Plan to pay - Employees Contribution"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart218502_en
msgid "Group Pension Plan to pay - Employer Contribution"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.hst13_purc_en
msgid "HST for purchases - 13%"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.hst14_purc_en
msgid "HST for purchases - 14%"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.hst15_purc_en
msgid "HST for purchases - 15%"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.hst13_sale_en
msgid "HST for sales - 13%"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.hst14_sale_en
msgid "HST for sales - 14%"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.hst15_sale_en
msgid "HST for sales - 15%"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart11831_en
msgid "HST receivable - 13%"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart11832_en
msgid "HST receivable - 14%"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart11833_en
msgid "HST receivable - 15%"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart21331_en
msgid "HST to pay - 13%"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart21332_en
msgid "HST to pay - 14%"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart21333_en
msgid "HST to pay - 15%"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart412_en
msgid "Harmonized Provinces Sales"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart512314_en
msgid "Health Service Fund"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart21421_en
msgid "Health Services Fund to pay"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart5111_en
msgid "Inside Purchases"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart411_en
msgid "Inside Sales"
msgstr ""

#. module: l10n_ca
#: model:account.fiscal.position.template,name:l10n_ca.fiscal_position_template_intl_en
msgid "International (INTL)"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart5114_en
msgid "International Purchases"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart414_en
msgid "International Sales"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart512312_en
msgid "Labour Health and Safety"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart21424_en
msgid "Labour Health and Safety to pay"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart512313_en
msgid "Labour Standards"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart21425_en
msgid "Labour Standards to pay"
msgstr ""

#. module: l10n_ca
#: model:account.fiscal.position.template,name:l10n_ca.fiscal_position_template_mb_en
msgid "Manitoba (MB)"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart55_en
msgid "NON-OPERATING EXPENSES"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart42_en
msgid "NON-OPERATING INCOMES"
msgstr ""

#. module: l10n_ca
#: model:account.fiscal.position.template,name:l10n_ca.fiscal_position_template_nb_en
msgid "New Brunswick (NB)"
msgstr ""

#. module: l10n_ca
#: model:account.fiscal.position.template,name:l10n_ca.fiscal_position_template_nl_en
msgid "Newfoundland and Labrador (NL)"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart413_en
msgid "Non-Harmonized Provinces Sales"
msgstr ""

#. module: l10n_ca
#: model:account.fiscal.position.template,name:l10n_ca.fiscal_position_template_nt_en
msgid "Northwest Territories (NT)"
msgstr ""

#. module: l10n_ca
#: model:account.fiscal.position.template,name:l10n_ca.fiscal_position_template_ns_en
msgid "Nova Scotia (NS)"
msgstr ""

#. module: l10n_ca
#: model:account.fiscal.position.template,name:l10n_ca.fiscal_position_template_nu_en
msgid "Nunavut (NU)"
msgstr ""

#. module: l10n_ca
#: model:account.fiscal.position.template,name:l10n_ca.fiscal_position_template_on_en
msgid "Ontario (ON)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.pst_sk_purc_en
msgid "PST for purchases - 5% (SK)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.pst_bc_purc_en
msgid "PST for purchases - 7% (BC)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.pst_mb_purc_en
msgid "PST for purchases - 8% (MB)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.pst_sk_sale_en
msgid "PST for sales - 5% (SK)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.pst_bc_sale_en
msgid "PST for sales - 7% (BC)"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.pst_mb_sale_en
msgid "PST for sales - 8% (MB)"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart1182_en
msgid "PST/QST receivable"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart2132_en
msgid "PST/QST to pay"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart214231_en
msgid "Parental Insurance Plan - Employee Contribution"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart214232_en
msgid "Parental Insurance Plan - Employer Contribution"
msgstr ""

#. module: l10n_ca
#: model:account.fiscal.position.template,name:l10n_ca.fiscal_position_template_pe_en
msgid "Prince Edward Islands (PE)"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart21426_en
msgid "Provincial Income Tax"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart512311_en
msgid "Provincial Parental Insurance Plan"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart512310_en
msgid "Provincial Pension Plan"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart214221_en
msgid "Provincial Pension Plan - Employees Contribution"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart214222_en
msgid "Provincial Pension Plan - Employer Contribution"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart2521_en
msgid "Provision for pension plans"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart5112_en
msgid "Purchases in harmonized provinces"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart5113_en
msgid "Purchases in non-harmonized provinces"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.qst_purc_en
msgid "QST for purchases - 9.975%"
msgstr ""

#. module: l10n_ca
#: model:account.tax.template,name:l10n_ca.qst_sale_en
msgid "QST for sales - 9.975%"
msgstr ""

#. module: l10n_ca
#: model:account.fiscal.position.template,name:l10n_ca.fiscal_position_template_qc_en
msgid "Quebec (QC)"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart512101_en
msgid "Regular Salaries"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart512103_en
msgid "Retroactive Pay"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart2184_en
msgid "Retroactive Payment to pay"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart2181_en
msgid "Salaries to pay"
msgstr ""

#. module: l10n_ca
#: model:account.fiscal.position.template,name:l10n_ca.fiscal_position_template_sk_en
msgid "Saskatchewan (SK)"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart254103_en
#: model:account.account.template,name:l10n_ca.chart512203_en
msgid "Sick Leaves Accrued"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart1145_en
msgid "Stock Delivered But Not Billed"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart1141_en
msgid "Stock In Hand"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart2171_en
msgid "Stock Received But Not Billed"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart113_en
msgid "Transfer Account"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart254101_en
#: model:account.account.template,name:l10n_ca.chart512201_en
msgid "Vacations Accrued"
msgstr ""

#. module: l10n_ca
#: model:account.account.template,name:l10n_ca.chart2111_en
msgid "Vendors Account"
msgstr ""

#. module: l10n_ca
#: model:account.fiscal.position.template,name:l10n_ca.fiscal_position_template_yt_en
msgid "Yukon (YT)"
msgstr ""


```

## File: models\res_partner.py

```python
# coding: utf-8
from odoo import api, fields, models, _


class ResPartner(models.Model):
    _inherit = 'res.partner'

    l10n_ca_pst = fields.Char('PST')

```

## File: models\__init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from . import res_partner

```

## File: views\report_invoice.xml

```xml
<odoo>
    <template id="l10n_ca_report_invoice_document_inherit" inherit_id="account.report_invoice_document">
        <div t-if="o.partner_id.vat" position="after">
            <t t-if="o.company_id.country_id.code == 'CA' and o.partner_id.l10n_ca_pst" class="mt16">
                <div>PST: <span t-field="o.partner_id.l10n_ca_pst"/></div>
            </t>
        </div>
    </template>
</odoo>
```

## File: views\res_partner_view.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data>
        <record id="res_partner_form_inherit_ca" model="ir.ui.view">
            <field name="name">res.partner.form.inherit.l10n.ca</field>
            <field name="model">res.partner</field>
            <field name="inherit_id" ref="base.view_partner_form"/>
            <field name="arch" type="xml">
                <xpath expr="//field[@name='vat']" position="after">
                    <field name="l10n_ca_pst"/>
                </xpath>
            </field>
        </record>
    </data>
</odoo>
```

