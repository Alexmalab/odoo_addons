# Odoo Module: l10n_th

Category: Localization

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
    'category': 'Localization',
    'description': """
Chart of Accounts for Thailand.
===============================

Thai accounting chart and localization.
    """,
    'author': 'Almacom',
    'website': 'http://almacom.co.th/',
    'depends': ['account'],
    'data': [
        'data/account_data.xml',
        'data/l10n_th_chart_data.xml',
        'data/account.account.template.csv',
        'data/l10n_th_chart_post_data.xml',
        'data/account_tax_report_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_chart_template_data.xml',
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

## File: data\account_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <data noupdate="1">

        <!-- Account Tax Group -->
        <record id="tax_group_1" model="account.tax.group">
            <field name="name">TAX 1%</field>
        </record>
        <record id="tax_group_2" model="account.tax.group">
            <field name="name">TAX 2%</field>
        </record>
        <record id="tax_group_3" model="account.tax.group">
            <field name="name">TAX 3%</field>
        </record>
        <record id="tax_group_5" model="account.tax.group">
            <field name="name">TAX 5%</field>
        </record>
        <record id="tax_group_vat_7" model="account.tax.group">
            <field name="name">VAT 7%</field>
        </record>

    </data>
</odoo>

```

## File: data\account_tax_report_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>

    <record id="tax_report_out_tax_title" model="account.tax.report.line">
        <field name="name">Output Tax</field>
        <field name="sequence" eval="1"/>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_out_tax_sale" model="account.tax.report.line">
        <field name="name">1. Sales amount</field>
        <field name="tag_name">1. Sales amount</field>
        <field name="code">OUTPUTTAX_SALEAMOUNT</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_out_tax_title"/>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_out_tax_less_sales_0_rate" model="account.tax.report.line">
        <field name="name">2. Less sales subject to 0% tax rate </field>
        <field name="tag_name">2. Less sales subject to 0% tax rate </field>
        <field name="code">OUTPUTTAX_SALE_ZERO</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_out_tax_title"/>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_out_tax_less_exempted_sales" model="account.tax.report.line">
        <field name="name">3. Less exempted sales</field>
        <field name="tag_name">3. Less exempted sales</field>
        <field name="code">OUTPUTTAX_EXEMPTED</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_out_tax_title"/>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_out_tax_taxable_sales_amount" model="account.tax.report.line">
        <field name="name">4. Taxable sales amount(1. -2. -3.)</field>
        <field name="sequence" eval="4"/>
        <field name="parent_id" ref="tax_report_out_tax_title"/>
        <field name="formula">OUTPUTTAX_SALEAMOUNT-(OUTPUTTAX_SALE_ZERO+OUTPUTTAX_EXEMPTED)</field>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_out_tax" model="account.tax.report.line">
        <field name="name">5. Output tax</field>
        <field name="tag_name">5. Output tax</field>
        <field name="code">OUTPUTTAX_TAX</field>
        <field name="sequence" eval="5"/>
        <field name="parent_id" ref="tax_report_out_tax_title"/>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_input_tax_title" model="account.tax.report.line">
        <field name="name">Input Tax</field>
        <field name="code">INPUTTAX</field>
        <field name="sequence" eval="2"/>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_input_tax_purchase_from_out_tax" model="account.tax.report.line">
        <field name="name">6. Purchase amount that is entitled to deduction of input tax from output tax in tax computation</field>
        <field name="tag_name">6. Purchase amount that is entitled to deduction of input tax from output tax in tax computation</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_input_tax_title"/>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_input_tax" model="account.tax.report.line">
        <field name="name">7. Input tax (according to invoice of purchase amount in 6.)</field>
        <field name="tag_name">7. Input tax (according to invoice of purchase amount in 6.)</field>
        <field name="code">INPUTTAX_TAX</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_input_tax_title"/>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_vat" model="account.tax.report.line">
        <field name="name">Value Added Tax</field>
        <field name="sequence" eval="3"/>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_vat_payable" model="account.tax.report.line">
        <field name="name">8. Tax payable (5. minus 7. (if 5. is greater than 7.))</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_vat"/>
        <field name="formula">OUTPUTTAX_TAX - INPUTTAX_TAX if (OUTPUTTAX_TAX > INPUTTAX_TAX) else 0</field>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_vat_excess" model="account.tax.report.line">
        <field name="name">9. Excess tax payable (7. minus 5. (if 5. is less than 7.))</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_vat"/>
        <field name="formula">INPUTTAX_TAX - OUTPUTTAX_TAX if (INPUTTAX_TAX > OUTPUTTAX_TAX) else 0</field>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_vat_payment_last_period" model="account.tax.report.line">
        <field name="name">10. Excess tax payment carried forward from last period</field>
        <field name="sequence" eval="3"/>
        <field name="parent_id" ref="tax_report_vat"/>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_net_vat" model="account.tax.report.line">
        <field name="name">Net Tax</field>
        <field name="sequence" eval="4"/>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_net_vat_payable" model="account.tax.report.line">
        <field name="name">11. Net tax payable (if 8. is greater than 10.)</field>
        <field name="tag_name">11. Net tax payable (if 8. is greater than 10.)</field>
        <field name="sequence" eval="1"/>
        <field name="parent_id" ref="tax_report_net_vat"/>
        <field name="country_id" ref="base.th"/>
    </record>

    <record id="tax_report_net_vat_excess" model="account.tax.report.line">
        <field name="name">12. Net excess tax payable ((if 10. is greater than 8.) or (9. plus 10.))</field>
        <field name="tag_name">12. Net excess tax payable ((if 10. is greater than 8.) or (9. plus 10.))</field>
        <field name="sequence" eval="2"/>
        <field name="parent_id" ref="tax_report_net_vat"/>
        <field name="country_id" ref="base.th"/>
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
    <menuitem id="account_reports_th_statements_menu" name="Thailand" parent="account.menu_finance_reports" sequence="0" groups="account.group_account_user"/>
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

