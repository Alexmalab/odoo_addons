# Odoo Module: l10n_th

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.


def _preserve_tag_on_taxes(cr, registry):
    from odoo.addons.account.models.chart_template import preserve_existing_tags_on_taxes
    preserve_existing_tags_on_taxes(cr, registry, 'l10n_th')

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Thailand - Accounting',
    'version': '2.0',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
Chart of Accounts for Thailand.
===============================

Thai accounting chart and localization.
    """,
    'author': 'Almacom',
    'website': 'http://almacom.co.th/',
    'depends': ['account'],
    'data': [
        'data/account_tax_group_data.xml',
        'data/l10n_th_chart_data.xml',
        'data/account.account.template.csv',
        'data/l10n_th_chart_post_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_chart_template_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'post_init_hook': '_preserve_tag_on_taxes',
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
"id","name","code","user_type_id/id","chart_template_id/id","reconcile"
"a_recv","Account Receivable","1200","account.data_account_type_receivable","l10n_th.chart","True"
"a_recv_pos","Account Receivable (PoS)","1210","account.data_account_type_receivable","l10n_th.chart","True"
"a_cheque","Outstanding Cheques","1201","account.data_account_type_current_liabilities","l10n_th.chart","True"
"a_invent","Inventory","1300","account.data_account_type_current_assets","l10n_th.chart","False"
"a_building_depr","Accumulated Depreciation - Building","1411","account.data_account_type_depreciation","l10n_th.chart","False"
"a_equipment_dept","Accumulated Depreciation - Equipment","1421","account.data_account_type_depreciation","l10n_th.chart","False"
"a_input_vat","Input VAT","1510","account.data_account_type_current_assets","l10n_th.chart","False"
"a_wht_income","Withholding Income Tax","1520","account.data_account_type_current_assets","l10n_th.chart","False"
"a_pay","Account Payable","2101","account.data_account_type_payable","l10n_th.chart","True"
"a_accr_exp","Accrued Expenses","2102","account.data_account_type_current_liabilities","l10n_th.chart","True"
"a_uninvoiced_receipts","Uninvoiced Receipts","2103","account.data_account_type_current_liabilities","l10n_th.chart","True"
"a_loan","Loans","2200","account.data_account_type_current_liabilities","l10n_th.chart","True"
"a_output_vat","Output VAT","2310","account.data_account_type_current_liabilities","l10n_th.chart","False"
"a_wht","Withholding Tax","2320","account.data_account_type_current_liabilities","l10n_th.chart","False"
"a_common_stock","Capital Stock","3100","account.data_account_type_equity","l10n_th.chart","False"
"a_retained_earnings","Retained Earnings","3200","account.data_account_type_equity","l10n_th.chart","False"
"a_dividends","Dividends","3300","account.data_account_type_equity","l10n_th.chart","False"
"a_income_summary","Income Summary","3400","account.data_account_type_equity","l10n_th.chart","False"
"a_sales","Income","4100","account.data_account_type_revenue","l10n_th.chart","False"
"a_income_gain","Gain Account","4200","account.data_account_type_other_income","l10n_th.chart","False"
"a_exp_cogs","Cost of Revenue","5100","account.data_account_type_direct_costs","l10n_th.chart","False"
"a_exp_salary","Salary","5201","account.data_account_type_expenses","l10n_th.chart","False"
"a_exp_rent","Rent","5202","account.data_account_type_expenses","l10n_th.chart","False"
"a_exp_office","Office Expenses","5203","account.data_account_type_expenses","l10n_th.chart","False"
"a_exp_interest","Interest expenses","5300","account.data_account_type_expenses","l10n_th.chart","False"
"a_exp_income_tax","Income tax expenses","5400","account.data_account_type_expenses","l10n_th.chart","False"
"a_exp_loss","Loss Account","5500","account.data_account_type_expenses","l10n_th.chart","False"

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <function model="account.chart.template" name="try_loading">
            <value eval="[ref('l10n_th.chart')]"/>
        </function>
    </data>
</odoo>

```

## File: data\account_tax_group_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">
        <record id="tax_group_1" model="account.tax.group">
            <field name="name">TAX 1%</field>
            <field name="country_id" ref="base.th"/>
        </record>
        <record id="tax_group_2" model="account.tax.group">
            <field name="name">TAX 2%</field>
            <field name="country_id" ref="base.th"/>
        </record>
        <record id="tax_group_3" model="account.tax.group">
            <field name="name">TAX 3%</field>
            <field name="country_id" ref="base.th"/>
        </record>
        <record id="tax_group_5" model="account.tax.group">
            <field name="name">TAX 5%</field>
            <field name="country_id" ref="base.th"/>
        </record>
        <record id="tax_group_vat_7" model="account.tax.group">
            <field name="name">VAT 7%</field>
            <field name="country_id" ref="base.th"/>
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
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_out_tax_title" model="account.tax.report.line">
        <field name="name">Output Tax</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
    </record>

    <record id="tax_report_out_tax_sale" model="account.tax.report.line">
        <field name="name">1. Sales amount</field>
        <field name="tag_name">1. Sales amount</field>
        <field name="code">OUTPUTTAX_SALEAMOUNT</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_out_tax_title"/>
    </record>

    <record id="tax_report_out_tax_less_sales_0_rate" model="account.tax.report.line">
        <field name="name">2. Less sales subject to 0% tax rate </field>
        <field name="tag_name">2. Less sales subject to 0% tax rate </field>
        <field name="code">OUTPUTTAX_SALE_ZERO</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_out_tax_title"/>
    </record>

    <record id="tax_report_out_tax_less_exempted_sales" model="account.tax.report.line">
        <field name="name">3. Less exempted sales</field>
        <field name="tag_name">3. Less exempted sales</field>
        <field name="code">OUTPUTTAX_EXEMPTED</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_out_tax_title"/>
    </record>

    <record id="tax_report_out_tax_taxable_sales_amount" model="account.tax.report.line">
        <field name="name">4. Taxable sales amount(1. -2. -3.)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="tax_report_out_tax_title"/>
        <field name="formula">OUTPUTTAX_SALEAMOUNT-(OUTPUTTAX_SALE_ZERO+OUTPUTTAX_EXEMPTED)</field>
    </record>

    <record id="tax_report_out_tax" model="account.tax.report.line">
        <field name="name">5. Output tax</field>
        <field name="tag_name">5. Output tax</field>
        <field name="code">OUTPUTTAX_TAX</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="tax_report_out_tax_title"/>
    </record>

    <record id="tax_report_input_tax_title" model="account.tax.report.line">
        <field name="name">Input Tax</field>
        <field name="code">INPUTTAX</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
    </record>

    <record id="tax_report_input_tax_purchase_from_out_tax" model="account.tax.report.line">
        <field name="name">6. Purchase amount that is entitled to deduction of input tax from output tax in tax computation</field>
        <field name="tag_name">6. Purchase amount that is entitled to deduction of input tax from output tax in tax computation</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_input_tax_title"/>
    </record>

    <record id="tax_report_input_tax" model="account.tax.report.line">
        <field name="name">7. Input tax (according to invoice of purchase amount in 6.)</field>
        <field name="tag_name">7. Input tax (according to invoice of purchase amount in 6.)</field>
        <field name="code">INPUTTAX_TAX</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_input_tax_title"/>
    </record>

    <record id="tax_report_vat" model="account.tax.report.line">
        <field name="name">Value Added Tax</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
    </record>

    <record id="tax_report_vat_payable" model="account.tax.report.line">
        <field name="name">8. Tax payable (5. minus 7. (if 5. is greater than 7.))</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_vat"/>
        <field name="formula">OUTPUTTAX_TAX - INPUTTAX_TAX if (OUTPUTTAX_TAX > INPUTTAX_TAX) else 0</field>
    </record>

    <record id="tax_report_vat_excess" model="account.tax.report.line">
        <field name="name">9. Excess tax payable (7. minus 5. (if 5. is less than 7.))</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_vat"/>
        <field name="formula">INPUTTAX_TAX - OUTPUTTAX_TAX if (INPUTTAX_TAX > OUTPUTTAX_TAX) else 0</field>
    </record>

    <record id="tax_report_vat_payment_last_period" model="account.tax.report.line">
        <field name="name">10. Excess tax payment carried forward from last period</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_vat"/>
    </record>

    <record id="tax_report_net_vat" model="account.tax.report.line">
        <field name="name">Net Tax</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="4"/>
    </record>

    <record id="tax_report_net_vat_payable" model="account.tax.report.line">
        <field name="name">11. Net tax payable (if 8. is greater than 10.)</field>
        <field name="tag_name">11. Net tax payable (if 8. is greater than 10.)</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_net_vat"/>
    </record>

    <record id="tax_report_net_vat_excess" model="account.tax.report.line">
        <field name="name">12. Net excess tax payable ((if 10. is greater than 8.) or (9. plus 10.))</field>
        <field name="tag_name">12. Net excess tax payable ((if 10. is greater than 8.) or (9. plus 10.))</field>
        <field name="report_id" ref="tax_report"/>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_net_vat"/>
    </record>

</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_input_vat" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Input VAT 7%</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="7"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_vat_7"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_input_tax_purchase_from_out_tax')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_input_vat'),
                'plus_report_line_ids': [ref('tax_report_input_tax')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_input_tax_purchase_from_out_tax')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_input_vat'),
                'minus_report_line_ids': [ref('tax_report_input_tax')],
            }),
        ]"/>
    </record>

    <record id="tax_output_vat" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Output VAT 7%</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="7"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_vat_7"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_out_tax_sale')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_output_vat'),
                'plus_report_line_ids': [ref('tax_report_out_tax')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_out_tax_sale')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_output_vat'),
                'minus_report_line_ids': [ref('tax_report_out_tax')],
            }),
        ]"/>
    </record>

    <record id="tax_input_vat_0" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Input VAT 0%</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="0"/>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_input_tax_purchase_from_out_tax')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_input_vat'),
                'plus_report_line_ids': [ref('tax_report_input_tax')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_input_tax_purchase_from_out_tax')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_input_vat'),
                'plus_report_line_ids': [ref('tax_report_input_tax')],
            }),
        ]"/>
    </record>

    <record id="tax_output_vat_0" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Output VAT 0%</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="0"/>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_out_tax_sale'), ref('tax_report_out_tax_less_sales_0_rate')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_output_vat'),
                'plus_report_line_ids': [ref('tax_report_out_tax')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_out_tax_sale'), ref('tax_report_out_tax_less_sales_0_rate')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_output_vat'),
                'minus_report_line_ids': [ref('tax_report_out_tax')],
            }),
        ]"/>
    </record>

    <record id="tax_input_vat_exempted" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Input VAT Exempted</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="0"/>
        <field name="type_tax_use">purchase</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_input_tax_purchase_from_out_tax')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_input_vat'),
                'plus_report_line_ids': [ref('tax_report_input_tax')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_input_tax_purchase_from_out_tax')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_input_vat'),
                'minus_report_line_ids': [ref('tax_report_input_tax')],
            }),
        ]"/>
    </record>

    <record id="tax_output_vat_exempted" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Output VAT Exempted</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="0"/>
        <field name="type_tax_use">sale</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'plus_report_line_ids': [ref('tax_report_out_tax_sale'), ref('tax_report_out_tax_less_exempted_sales')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_output_vat'),
                'plus_report_line_ids': [ref('tax_report_out_tax')],
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'minus_report_line_ids': [ref('tax_report_out_tax_sale'), ref('tax_report_out_tax_less_exempted_sales')],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_output_vat'),
                'minus_report_line_ids': [ref('tax_report_out_tax')],
            }),
        ]"/>
    </record>

    <record id="tax_wht_co_1" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Company Withholding Tax 1% (Transportation)</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="-1"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_wht'),
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
                'account_id': ref('a_wht'),
            }),
        ]"/>
    </record>

    <record id="tax_wht_co_2" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Company Withholding Tax 2% (Advertising)</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="-2"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_2"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_wht'),
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
                'account_id': ref('a_wht'),
            }),
        ]"/>
    </record>

    <record id="tax_wht_co_3" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Company Withholding Tax 3% (Service)</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="-3"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_3"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_wht'),
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
                'account_id': ref('a_wht'),
            }),
        ]"/>
    </record>

    <record id="tax_wht_co_5" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Company Withholding Tax 5% (Rental)</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="-5"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_wht'),
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
                'account_id': ref('a_wht'),
            }),
        ]"/>
    </record>

    <record id="tax_wht_pers_1" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Personal Withholding Tax 1% (Transportation)</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="-1"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_wht'),
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
                'account_id': ref('a_wht'),
            }),
        ]"/>
    </record>

    <record id="tax_wht_pers_2" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Personal Withholding Tax 2% (Advertising)</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="-2"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_2"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_wht'),
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
                'account_id': ref('a_wht'),
            }),
        ]"/>
    </record>

    <record id="tax_wht_pers_3" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Personal Withholding Tax 3% (Service)</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="-3"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_3"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_wht'),
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
                'account_id': ref('a_wht'),
            }),
        ]"/>
    </record>

    <record id="tax_wht_pers_5" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Personal Withholding Tax 5% (Rental)</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="-5"/>
        <field name="type_tax_use">purchase</field>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_wht'),
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
                'account_id': ref('a_wht'),
            }),
        ]"/>
    </record>

    <record id="tax_wht_income_1" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Withholding Income Tax 1% (Transportation)</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="-1"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_1"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_wht_income'),
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
                'account_id': ref('a_wht_income'),
            }),
        ]"/>
    </record>

    <record id="tax_wht_income_2" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Withholding Income Tax 2% (Advertising)</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="-2"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_2"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_wht_income'),
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
                'account_id': ref('a_wht_income'),
            }),
        ]"/>
    </record>

    <record id="tax_wht_income_3" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Withholding Income Tax 3% (Service)</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="-3"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_3"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_wht_income'),
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
                'account_id': ref('a_wht_income'),
            }),
        ]"/>
    </record>

    <record id="tax_wht_income_5" model="account.tax.template">
        <field name="chart_template_id" ref="chart"/>
        <field name="name">Withholding Income Tax 5% (Rental)</field>
        <field name="amount_type">percent</field>
        <field name="amount" eval="-5"/>
        <field name="type_tax_use">sale</field>
        <field name="tax_group_id" ref="tax_group_5"/>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_wht_income'),
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
                'account_id': ref('a_wht_income'),
            }),
        ]"/>
    </record>

</odoo>

```

## File: data\l10n_th_chart_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <menuitem id="account_reports_th_statements_menu" name="Thailand" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_readonly"/>
<!-- ACCOUNT TYPES -->

<record id="acc_type_reconciled" model="account.account.type">
    <field name="name">Reconciled</field>
    <field name="internal_group">off_balance</field>
</record>

<!-- CHART OF Template -->

<record id="chart" model="account.chart.template">
    <field name="name">Thailand - Chart of Accounts</field>
    <field name="cash_account_code_prefix">1100</field>
    <field name="bank_account_code_prefix">1110</field>
    <field name="transfer_account_code_prefix">16</field>
    <field name="currency_id" ref="base.THB"/>
    <field name="country_id" ref="base.th"/>
</record>
</odoo>

```

## File: data\l10n_th_chart_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="chart" model="account.chart.template">
        <field name="property_account_receivable_id" ref="a_recv"/>
        <field name="property_account_payable_id" ref="a_pay"/>
        <field name="property_account_expense_categ_id" ref="a_exp_cogs"/>
        <field name="property_account_income_categ_id" ref="a_sales"/>
        <field name="income_currency_exchange_account_id" ref="a_income_gain"/>
        <field name="expense_currency_exchange_account_id" ref="a_exp_loss"/>
        <field name="default_pos_receivable_account_id" ref="a_recv_pos" />
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
    <mask id="b" x="4.6" y="7.2" width="52.8" height="32.97" maskUnits="userSpaceOnUse">
      <rect x="6.09" y="7.47" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="400" height="267" transform="translate(4.6 7.2) scale(0.13 0.12)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAZAAAAD6CAYAAACPpxFEAAAACXBIWXMAAFPQAABT0AEFpKxXAAAD2UlEQVR4Xu3ZsUkFURRF0TcyiWBPhrZjD2Z2YWIf04EtiPADzcXsmf8RHu54rfjkG+7d5hhzAMD/HDerBQD8RUAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEj28fS82gDAyTbnnKsRAFw5nLAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEj27++f1QYATvaH+8fVBgBO9o/3z9UGAE78QABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAREAASAQEgERAAEgEBIBEQABIBASAZL+7u11tAOBku1y+5moEAFeObc4pIAD81+EHAkAiIAAkAgJAIiAAJAICQCIgACQCAkAiIAAkAgJAIiAAJAICQCIgACQCAkAiIAAkAgJAIiAAJAICQCIgACQCAkAiIAAkAgJAIiAAJAICQCIgACQCAkAiIAAkAgJAIiAAJAICQCIgACQCAkAiIAAkAgJAIiAAJAICQCIgACQCAkAiIAAkAgJAso+X19UGAE62OcZcjQDgyuGEBUAiIAAkAgJAIiAAJAICQCIgACQCAkAiIAAkAgJAIiAAJAICQCIgACQCAkAiIAAkAgJAIiAAJAICQCIgACQCAkAiIAAkAgJAIiAAJAICQCIgACQCAkAiIAAkAgJAIiAAJAICQCIgACQCAkAiIAAkAgJAIiAAJAICQCIgACQCAkAiIAAkAgJAIiAAJPsY41iNAODK2y8UnB/7L9tHEgAAAABJRU5ErkJggg=="/>
    </g>
  </g>
</svg>

```

