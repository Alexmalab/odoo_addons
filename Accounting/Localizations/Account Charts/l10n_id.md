# Odoo Module: l10n_id

Category: Accounting/Localizations/Account Charts

This file contains the source code of the Odoo module.

## File: __init__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

from odoo import api, SUPERUSER_ID

def load_translations(cr, registry):
    env = api.Environment(cr, SUPERUSER_ID, {})
    env.ref('l10n_id.l10n_id_chart').process_coa_translations()

```

## File: __manifest__.py

```python
# -*- coding: utf-8 -*-
# Part of Odoo. See LICENSE file for full copyright and licensing details.

{
    'name': 'Indonesian - Accounting',
    'version': '1.1',
    'category': 'Accounting/Localizations/Account Charts',
    'description': """
This is the latest Indonesian Odoo localisation necessary to run Odoo accounting for SMEs with:
=================================================================================================
    - generic Indonesian chart of accounts
    - tax structure""",
    'author': 'vitraining.com',
    'website': 'http://www.vitraining.com',
    'depends': ['account', 'base_iban', 'base_vat', 'l10n_multilang'],
    'data': [
        'data/account_tax_group.xml',
        'data/account_chart_template_data.xml',
        'data/account.account.template.csv',
        'data/account_chart_template_post_data.xml',
        'data/account_tax_template_data.xml',
        'data/account_chart_template_configuration_data.xml',
    ],
    'demo': [
        'demo/demo_company.xml',
    ],
    'post_init_hook': 'load_translations',
    'license': 'LGPL-3',
}

```

## File: data\account.account.template.csv

```csv
id,code,name,account_type,reconcile,chart_template_id:id
l10n_id_11110001,11110001,Cash,asset_cash,FALSE,l10n_id.l10n_id_chart
l10n_id_11110010,11110010,Petty Cash,asset_cash,FALSE,l10n_id.l10n_id_chart
l10n_id_11110020,11110020,Cash in Hand,asset_cash,FALSE,l10n_id.l10n_id_chart
l10n_id_11120001,11120001,Bank Suspense,liability_current,TRUE,l10n_id.l10n_id_chart
l10n_id_11120002,11120002,Outstanding Receipts,asset_current,TRUE,l10n_id.l10n_id_chart
l10n_id_11120003,11120003,Outstanding Payments,asset_current,TRUE,l10n_id.l10n_id_chart
l10n_id_11120004,11120004,Bank,asset_cash,FALSE,l10n_id.l10n_id_chart
l10n_id_11210010,11210010,Account Receivable,asset_receivable,TRUE,l10n_id.l10n_id_chart
l10n_id_11210011,11210011,Account Receivable (PoS),asset_receivable,TRUE,l10n_id.l10n_id_chart
l10n_id_11210020,11210020,Employee Liabilities,asset_current,TRUE,l10n_id.l10n_id_chart
l10n_id_11300180,11300180,Other Inventory,asset_current,FALSE,l10n_id.l10n_id_chart
l10n_id_11410010,11410010,Building Rent,asset_prepayments,FALSE,l10n_id.l10n_id_chart
l10n_id_11410020,11410020,Prepaid Insurance,asset_prepayments,FALSE,l10n_id.l10n_id_chart
l10n_id_11410030,11410030,Prepaid Advertisement-Free,asset_prepayments,FALSE,l10n_id.l10n_id_chart
l10n_id_11510010,11510010,Prepaid Tax PPh 21,asset_prepayments,FALSE,l10n_id.l10n_id_chart
l10n_id_11510020,11510020,Prepaid Tax Pph 22,asset_prepayments,FALSE,l10n_id.l10n_id_chart
l10n_id_11510030,11510030,Prepaid Tax Pph 23,asset_prepayments,FALSE,l10n_id.l10n_id_chart
l10n_id_11510040,11510040,Prepaid Tax Pph 25,asset_prepayments,FALSE,l10n_id.l10n_id_chart
l10n_id_11510050,11510050,Prepaid Tax Pph 28A,asset_prepayments,FALSE,l10n_id.l10n_id_chart
l10n_id_11510060,11510060,Prepaid Tax 4 (2),asset_prepayments,FALSE,l10n_id.l10n_id_chart
l10n_id_11800000,11800000,Down Payment,asset_prepayments,FALSE,l10n_id.l10n_id_chart
l10n_id_12210010,12210010,Office Building,asset_fixed,FALSE,l10n_id.l10n_id_chart
l10n_id_12210020,12210020,Vehicle,asset_fixed,FALSE,l10n_id.l10n_id_chart
l10n_id_12210030,12210030,Office Supplies,asset_fixed,FALSE,l10n_id.l10n_id_chart
l10n_id_12281010,12281010,Accumulation Building Depreciation,asset_prepayments,FALSE,l10n_id.l10n_id_chart
l10n_id_12281020,12281020,Accumulation Vehicle Depreciation,asset_prepayments,FALSE,l10n_id.l10n_id_chart
l10n_id_12281030,12281030,Accumulation Office Supplies Depreciation,asset_prepayments,FALSE,l10n_id.l10n_id_chart
l10n_id_21100010,21100010,Trade Receivable,liability_payable,TRUE,l10n_id.l10n_id_chart
l10n_id_21100020,21100020,Shareholder Deposit,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_21100030,21100030,Third-Party Deposit,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_21100040,21100040,Salary Deposit,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_21210010,21210010,Tax Payable Pph 21,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_21210020,21210020,Tax Payable Pph 23,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_21210030,21210030,Tax Payable Pph 25,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_21210040,21210040,Tax Payable 4 (2),liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_21210050,21210050,Tax Payable Pph 29,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_21221010,21221010,VAT Purchase,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_21221020,21221020,VAT Sales,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_22110010,22110010,Bank Loan,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_22110020,22110020,Leasing Deposit,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_25110010,25110010,Accrued Payable Electricity,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_25110020,25110020,Accrued Payable Jamsostek,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_25110030,25110030,Accrued Payable Water,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_25110040,25110040,Accrued Payable Telp & Internet,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_25110050,25110050,Accrued Payable Security Management,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_25110060,25110060,Accrued Payable Bank,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_25110070,25110070,Accrued Payable PBB,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_25110080,25110080,Accrued Payable Business License,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_25110090,25110090,Accrued Payable Insurance,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_25110100,25110100,Accrued Payable Education,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_25110110,25110110,Accrued Payable Health Insurance/BPJS,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_28110010,28110010,Advance Sales,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_28110020,28110020,Customer Deposit,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_29000000,29000000,Interim Stock,liability_current,FALSE,l10n_id.l10n_id_chart
l10n_id_31100010,31100010,Authorized Capital,equity,FALSE,l10n_id.l10n_id_chart
l10n_id_31100020,31100020,Paid Capital,equity,FALSE,l10n_id.l10n_id_chart
l10n_id_31100030,31100030,Unpaid Capital,equity,FALSE,l10n_id.l10n_id_chart
l10n_id_31100040,31100040,Prive (Personal Retrieval),equity,FALSE,l10n_id.l10n_id_chart
l10n_id_31210010,31210010,Capital Reserves,equity,FALSE,l10n_id.l10n_id_chart
l10n_id_31510010,31510010,Past Profit & Loss,equity,FALSE,l10n_id.l10n_id_chart
l10n_id_31510020,31510020,Ongoing Profit & Loss,equity,FALSE,l10n_id.l10n_id_chart
l10n_id_39000000,39000000,Historical Balance,equity,TRUE,l10n_id.l10n_id_chart
l10n_id_41000010,41000010,Sales,income,FALSE,l10n_id.l10n_id_chart
l10n_id_42000060,42000060,Sales Refund,income,FALSE,l10n_id.l10n_id_chart
l10n_id_42000070,42000070,Sales Discount,income,FALSE,l10n_id.l10n_id_chart
l10n_id_51000010,51000010,Cost of Goods Sold,expense_direct_cost,FALSE,l10n_id.l10n_id_chart
l10n_id_61100010,61100010,Employee Salary,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_61100020,61100020,Employee Bonus / Benefits,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_61100030,61100030,Employee Overtime Pay,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_61100100,61100100,Pph 21 Benefit,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_63110060,63110060,Phone,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_63110080,63110080,Electricity,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_63110100,63110100,Research & Development,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_63110120,63110120,Office Equipment,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_64110020,64110020,Post Necessities,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_63110140,63110140,Other Necessities,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_65110010,65110010,Licensing Fees,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_65110020,65110020,Bank Administration Fees,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_65110030,65110030,Consultant Fees,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_65110040,65110040,Rental Costs,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_65110050,65110050,Insurance Costs,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_65110060,65110060,Building Maintenance Costs,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_65110070,65110070,Taxes,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_65110080,65110080,Asset Maintenance Costs,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_65110090,65110090,Shipping Costs,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_66110010,66110010,Vehicle Fuel,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_66110020,66110020,Vehicle Service,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_66110030,66110030,Vehicle Parking & Toll Fee,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_66110040,66110040,Vehicle Taxes,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_66110050,66110050,Vehicle Insurance,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_67100010,67100010,Office Building,expense_depreciation,FALSE,l10n_id.l10n_id_chart
l10n_id_67100020,67100020,Vehicle,expense_depreciation,FALSE,l10n_id.l10n_id_chart
l10n_id_67100030,67100030,Office Supplies,expense_depreciation,FALSE,l10n_id.l10n_id_chart
l10n_id_69000000,69000000,Other Expenses,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_81100010,81100010,Interest Income,income_other,FALSE,l10n_id.l10n_id_chart
l10n_id_81100020,81100020,Deposit Income,income_other,FALSE,l10n_id.l10n_id_chart
l10n_id_81100030,81100030,Foreign Exchange Gain,income_other,FALSE,l10n_id.l10n_id_chart
l10n_id_81100040,81100040,Other Income,income_other,FALSE,l10n_id.l10n_id_chart
l10n_id_81100050,81100050,Gain on Sale of Fixed Assets,income_other,FALSE,l10n_id.l10n_id_chart
l10n_id_91100010,91100010,Interest Expense,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_91100020,91100020,Foreign Exchange Loss,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_91100030,91100030,Loss on Sale of Fixed Assets,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_99900001,99900001,Cash Difference Loss,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_99900002,99900002,Cash Difference Gain,income,FALSE,l10n_id.l10n_id_chart
l10n_id_99900003,99900003,Cash Discount Loss,expense,FALSE,l10n_id.l10n_id_chart
l10n_id_99900004,99900004,Cash Discount Gain,income_other,FALSE,l10n_id.l10n_id_chart
l10n_id_999999,999999,Undistributed Profits/Losses,equity_unaffected,FALSE,l10n_id.l10n_id_chart

```

## File: data\account_chart_template_configuration_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <function model="account.chart.template" name="try_loading">
        <value eval="[ref('l10n_id_chart')]"/>
    </function>
</odoo>

```

## File: data\account_chart_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_id_chart" model="account.chart.template">
        <field name="name">Indonesian Account Chart Template</field>
        <field name="bank_account_code_prefix">1112</field>
        <field name="cash_account_code_prefix">1111</field>
        <field name="transfer_account_code_prefix">1999999</field>
        <field name="code_digits">8</field>
        <field name="currency_id" ref="base.IDR"/>
        <field name="spoken_languages" eval="'id_ID'"/>
        <field name="country_id" ref="base.id"/>
    </record>
</odoo>

```

## File: data\account_chart_template_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_id_chart" model="account.chart.template">
        <field name="property_account_receivable_id" ref="l10n_id_11210010"/>
        <field name="property_account_payable_id" ref="l10n_id_21100010"/>
        <field name="property_account_expense_categ_id" ref="l10n_id_51000010"/>
        <field name="property_account_income_categ_id" ref="l10n_id_41000010"/>
        <field name="property_stock_account_input_categ_id" ref="l10n_id_29000000"/>
        <field name="property_stock_account_output_categ_id" ref="l10n_id_29000000"/>
        <field name="property_stock_valuation_account_id" ref="l10n_id_11300180"/>
        <field name="income_currency_exchange_account_id" ref="l10n_id_81100010"/>
        <field name="expense_currency_exchange_account_id" ref="l10n_id_91100010"/>
        <field name="default_pos_receivable_account_id" ref="l10n_id_11210011"/>
        <field name="account_journal_early_pay_discount_loss_account_id" ref="l10n_id_99900003"/>
        <field name="account_journal_early_pay_discount_gain_account_id" ref="l10n_id_99900004"/>
        <field name="use_anglo_saxon" eval="1"/>
    </record>
</odoo>

```

## File: data\account_tax_group.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo noupdate="1">
    <record id="l10n_id_tax_group_luxury_goods" model="account.tax.group">
        <field name="name">Luxury Good Taxes (ID)</field>
        <field name="sequence">1</field>
    </record>
</odoo>

```

## File: data\account_tax_template_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="ppn_tag" model="account.account.tag">
        <field name="name">PPN - 08</field>
        <field name="applicability">taxes</field>
        <field name="country_id" ref="base.id"/>
    </record>
    <record id="tax_ST1" model="account.tax.template">
        <field name="description">11%</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">sale</field>
        <field name="name">11%</field>
        <field name="amount_type">percent</field>
        <field name="amount">11.0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [(4, ref('ppn_tag'))],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221020'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [(4, ref('ppn_tag'))],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221020'),
            }),
        ]"/>
    </record>
    <record id="tax_PT1" model="account.tax.template">
        <field name="description">11%</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">11%</field>
        <field name="amount_type">percent</field>
        <field name="amount">11.0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [(4, ref('ppn_tag'))],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221010'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [(4, ref('ppn_tag'))],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221010'),
            }),
        ]"/>
    </record>
    <record id="tax_ST0" model="account.tax.template">
        <field name="description">0%</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">sale</field>
        <field name="name">0%</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221020'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221020'),
            }),
        ]"/>
    </record>
    <record id="tax_ST2" model="account.tax.template">
        <field name="description">0%</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Exempt</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221020'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221020'),
            }),
        ]"/>
    </record>
    <record id="tax_PT2" model="account.tax.template">
        <field name="description">0%</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Exempt</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221010'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221010'),
            }),
        ]"/>
    </record>
    <record id="tax_PT0" model="account.tax.template">
        <field name="description">0%</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">0%</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221010'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221010'),
            }),
        ]"/>
    </record>
    <record id="tax_ST3" model="account.tax.template">
        <field name="description">12%</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">sale</field>
        <field name="name">12%</field>
        <field name="amount_type">percent</field>
        <field name="amount">12.0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [(4, ref('ppn_tag'))],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221020'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [(4, ref('ppn_tag'))],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221020'),
            }),
        ]"/>
    </record>
    <record id="tax_PT3" model="account.tax.template">
        <field name="description">12%</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">12%</field>
        <field name="amount_type">percent</field>
        <field name="amount">12.0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [(4, ref('ppn_tag'))],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221010'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'repartition_type': 'base',
                'tag_ids': [(4, ref('ppn_tag'))],
            }),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221010'),
            }),
        ]"/>
    </record>
    <record id="tax_luxury_sales" model="account.tax.template">
        <field name="description">20% (Luxury Goods)</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">sale</field>
        <field name="name">20%</field>
        <field name="amount_type">percent</field>
        <field name="tax_group_id" ref="l10n_id.l10n_id_tax_group_luxury_goods"/>
        <field name="amount">20.0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221020'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {'repartition_type': 'base'}),
            (0,0, {
                'repartition_type': 'tax',
                'account_id': ref('l10n_id_21221020'),
            }),
        ]"/>
    </record>
</odoo>

```

## File: i18n_extra\l10n_id.pot

```pot
# Translation of Odoo Server.
# This file contains the translation of the following modules:
# 	* l10n_id
#
msgid ""
msgstr ""
"Project-Id-Version: Odoo Server 13.0+e\n"
"Report-Msgid-Bugs-To: \n"
"POT-Creation-Date: 2019-12-03 12:38+0000\n"
"PO-Revision-Date: 2019-12-03 12:38+0000\n"
"Last-Translator: \n"
"Language-Team: \n"
"MIME-Version: 1.0\n"
"Content-Type: text/plain; charset=UTF-8\n"
"Content-Transfer-Encoding: \n"
"Plural-Forms: \n"

#. module: l10n_id
#: model:account.tax.template,name:l10n_id.tax_PT1
#: model:account.tax.template,name:l10n_id.tax_ST1
msgid "11%"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_121001
msgid "Account Receivable"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_1210011
msgid "Account Receivable (PoS)"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_511006
msgid "Accrued Payable Bank"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_511008
msgid "Accrued Payable Business License"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_511010
msgid "Accrued Payable Education"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_511001
msgid "Accrued Payable Electricity"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_511011
msgid "Accrued Payable Health Insurance/BPJS"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_511009
msgid "Accrued Payable Insurance"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_511002
msgid "Accrued Payable Jamsostek"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_511007
msgid "Accrued Payable PBB"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_511005
msgid "Accrued Payable Security Management"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_511004
msgid "Accrued Payable Telp & Internet"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_511003
msgid "Accrued Payable Water"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_228101
msgid "Accumulation Building Depreciation"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_228105
msgid "Accumulation Office Furniture Depreciation"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_228103
msgid "Accumulation Office Supplies Depreciation"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_228104
msgid "Accumulation Software Depreciation"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_228102
msgid "Accumulation Vehicle Depreciation"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_811001
msgid "Advance Sales"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_211003
msgid "Advertising"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_511010
msgid "Asset Maintenance Costs"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_3_110001
msgid "Authorized Capital"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_112005
msgid "BCA"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_112004
msgid "BNI"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_112006
msgid "BNI Giro"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_9_110002
msgid "Bank Administration Expense"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_511002
msgid "Bank Administration Fees"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_211001
msgid "Bank Loan"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_811003
msgid "Bonus Point"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130012
msgid "Book, Office Stationery, Accessories Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_511006
msgid "Building Maintenance Costs"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_141001
msgid "Building Rent"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_112002
msgid "Business Mandiri"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_3_121001
msgid "Capital Reserves"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_111002
msgid "Cash in Hand"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130007
msgid "Cigarette Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311016
msgid "Cleaning Equipment"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130014
msgid "Cleaning Supplies Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_511003
msgid "Consultant Fees"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_5_100001
msgid "Cost of Goods Sold"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_811002
msgid "Customer Deposit"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_8_110002
msgid "Deposit Income"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311004
msgid "Donation"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_180000
msgid "Down Payment"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130004
msgid "Dried Goods Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130009
msgid "Drink Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311001
msgid "Drinking Water"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311008
msgid "Electricity"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_511007
msgid "Electricity, Telephone, and Internet Installation Maintenance Costs"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130016
msgid "Electronic Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_110008
msgid "Employee Birthday Benefit"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_110002
msgid "Employee Bonus / Benefits"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_110003
msgid "Employee Health Benefits"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_121002
msgid "Employee Liabilities"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_110004
msgid "Employee Meal (Catering)"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_110005
msgid "Employee Overtime Pay"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_110001
msgid "Employee Salary"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_211002
msgid "Event"
msgstr ""

#. module: l10n_id
#: model:account.tax.template,name:l10n_id.tax_PT2
#: model:account.tax.template,name:l10n_id.tax_ST2
msgid "Exempt"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311002
msgid "Exercise Necessities"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130013
msgid "Fashion & Textile Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311013
msgid "First Aid Kit"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130002
msgid "Fish Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130008
msgid "Food Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_8_110003
msgid "Foreign Exchange Gain"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_9_110003
msgid "Foreign Exchange Loss"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_211001
msgid "Free Gift"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130006
msgid "Fresh Drink Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130005
msgid "Fruit Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_8_110009
msgid "Gain on Sale of Fixed Assets"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_511009
msgid "Guest Accomodation"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_3_900000
msgid "Historical Balance"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130015
msgid "House Supplies Inventory"
msgstr ""

#. module: l10n_id
#: model:account.chart.template,name:l10n_id.l10n_id_chart
msgid "Indonesian Account Chart Template"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_9_110001
msgid "Interest Expense"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_8_110001
msgid "Interest Income"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_900000
msgid "Interim Stock"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311005
msgid "Internet"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_411003
msgid "Jilid & Photocopy"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_411004
msgid "Job Recruitment Advertisement"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311015
msgid "K3 (Fire Extinguisher)"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311011
msgid "Kitchen Necessities"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_221001
#: model:account.account.template,name:l10n_id.a_6_710001
msgid "Land"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_211002
msgid "Leasing Deposit"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_511001
msgid "Licensing Fees"
msgstr ""

#. module: l10n_id
msgid "Liquidity Transfer"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_9_110009
msgid "Loss on Sale of Fixed Assets"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_112007
msgid "Mandiri Giro"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_110009
msgid "Maternity Benefit"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130001
msgid "Meat Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311003
msgid "Monthly Fee"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_112003
msgid "Muamalat"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_221002
#: model:account.account.template,name:l10n_id.a_6_710002
msgid "Office Building"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311012
msgid "Office Equipment"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_221006
#: model:account.account.template,name:l10n_id.a_6_710006
msgid "Office Furniture"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_411001
msgid "Office Stationery"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_221004
#: model:account.account.template,name:l10n_id.a_6_710004
msgid "Office Supplies"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_3_151002
msgid "Ongoing Profit & Loss"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_900000
msgid "Other Expenses"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_8_110004
msgid "Other Income"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130018
msgid "Other Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311014
msgid "Other Necessities"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_211004
msgid "Other Receivable"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311018
msgid "Owner Necessities"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_211003
msgid "Owner Receivable"
msgstr ""

#. module: l10n_id
#: model:account.tax.template,description:l10n_id.tax_PT0
msgid "0%"
msgstr ""

#. module: l10n_id
#: model:account.tax.template,description:l10n_id.tax_PT1
msgid "11%"
msgstr ""

#. module: l10n_id
#: model:account.tax.template,description:l10n_id.tax_PT2
msgid "0%"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_3_110002
msgid "Paid Capital"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_3_151001
msgid "Past Profit & Loss"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_112001
msgid "Personal Mandiri"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_111001
msgid "Petty Cash"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311006
msgid "Phone"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_411002
msgid "Post Necessities"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_110010
msgid "Pph 21 Benefit"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_141003
msgid "Prepaid Advertisement-Free"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_141002
msgid "Prepaid Insurance"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311007
msgid "Prepaid Phone Bills"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_151001
msgid "Prepaid Tax Pph 22"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_151002
msgid "Prepaid Tax Pph 23"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_151003
msgid "Prepaid Tax Pph 25"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_3_110004
msgid "Prive (Personal Retrieval)"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130010
msgid "Processed Food Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_511004
msgid "Rental Costs"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311010
msgid "Research & Development"
msgstr ""

#. module: l10n_id
#: model:account.tax.template,description:l10n_id.tax_ST0
msgid "0%"
msgstr ""

#. module: l10n_id
#: model:account.tax.template,description:l10n_id.tax_ST1
msgid "11%"
msgstr ""

#. module: l10n_id
#: model:account.tax.template,description:l10n_id.tax_ST2
msgid "0%"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_110004
msgid "Salary Deposit"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_4_100001
msgid "Sales"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_4_200007
msgid "Sales Discount"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_4_200006
msgid "Sales Refund"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_110006
msgid "Security Service Fee"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_110002
msgid "Shareholder Deposit"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_511011
msgid "Shipping Costs"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_211004
msgid "Shipping Merchandise"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_221005
#: model:account.account.template,name:l10n_id.a_6_710005
msgid "Software"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_411005
msgid "Stamp"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_121004
msgid "Tax Payable 4 (2)"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_121001
msgid "Tax Payable Pph 21"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_121002
msgid "Tax Payable Pph 23"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_121003
msgid "Tax Payable Pph 25"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_121005
msgid "Tax Payable Pph 29"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_511008
msgid "Taxes"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_110003
msgid "Third-Party Deposit"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130011
msgid "Toiletries Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130017
msgid "Toys Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_110001
msgid "Trade Receivable"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_3_110003
msgid "Unpaid Capital"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_122101
msgid "VAT Purchase"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_2_122102
msgid "VAT Sales"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_130003
msgid "Vegetables Inventory"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_1_221003
#: model:account.account.template,name:l10n_id.a_6_710003
msgid "Vehicle"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_611001
msgid "Vehicle Fuel"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_611005
msgid "Vehicle Insurance"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_611003
msgid "Vehicle Parking & Toll Fee"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_611002
msgid "Vehicle Service"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_611004
msgid "Vehicle Taxes"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_311009
msgid "Water (PDAM)"
msgstr ""

#. module: l10n_id
#: model:account.account.template,name:l10n_id.a_6_110007
msgid "Work Uniform"
msgstr ""

```

## File: migrations\1.1\end-migrate_update_taxes.py

```python
# Part of Odoo. See LICENSE file for full copyright and licensing details.
from odoo.addons.account.models.chart_template import update_taxes_from_templates


def migrate(cr, version):
    update_taxes_from_templates(cr, 'l10n_id.l10n_id_chart')

```

## File: static\description\icon.svg

```svg
<svg xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" viewBox="0 0 106 106">
  <defs>
    <mask id="a" x="0" y="0" width="106" height="106" maskUnits="userSpaceOnUse">
      <path d="M6.06,0H98.43C104.49,0,106,1.51,106,7.57V98.43c0,6.06-1.51,7.57-7.57,7.57H6.06C1.51,106,0,104.49,0,98.43V7.57C0,1.51,1.51,0,6.06,0Z" style="fill: #fff;fill-rule: evenodd"/>
    </mask>
    <mask id="b" x="4.8" y="7.25" width="50.4" height="32.5" maskUnits="userSpaceOnUse">
      <rect x="6.29" y="7.53" width="48.45" height="31.57" rx="1" style="fill: #fff"/>
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
      <image width="255" height="170" transform="translate(4.8 7.25) scale(0.2 0.19)" xlink:href="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAP8AAAClCAYAAACTHStbAAAACXBIWXMAADf6AAA3+gH8300lAAACA0lEQVR4Xu3VsQ3AMBADsU/gxTO5M4J7H1mrPejZM3uAnPc0AO4kfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ELVm79MGuJDnhyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHKPFDlPghSvwQJX6IEj9EiR+ixA9R4oco8UOU+CFK/BAlfogSP0SJH6LED1HihyjxQ5T4IUr8ECV+iBI/RIkfosQPUeKHqDUz32kE3OcHTh8Fy0pWvJgAAAAASUVORK5CYII="/>
    </g>
  </g>
</svg>

```

