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
    'version': '1.0',
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
id,code,name,user_type_id:id,reconcile,chart_template_id:id
a_1_111001,1111001,Petty Cash,account.data_account_type_liquidity,TRUE,l10n_id_chart
a_1_111002,1111002,Cash in Hand,account.data_account_type_liquidity,TRUE,l10n_id_chart
a_1_112001,1112001,Personal Mandiri,account.data_account_type_liquidity,TRUE,l10n_id_chart
a_1_112002,1112002,Business Mandiri,account.data_account_type_liquidity,TRUE,l10n_id_chart
a_1_112003,1112003,Muamalat,account.data_account_type_liquidity,TRUE,l10n_id_chart
a_1_112004,1112004,BNI,account.data_account_type_liquidity,TRUE,l10n_id_chart
a_1_112005,1112005,BCA,account.data_account_type_liquidity,TRUE,l10n_id_chart
a_1_112006,1112006,BNI Giro,account.data_account_type_liquidity,TRUE,l10n_id_chart
a_1_112007,1112007,Mandiri Giro,account.data_account_type_liquidity,TRUE,l10n_id_chart
a_1_121001,1121001,Account Receivable,account.data_account_type_receivable,TRUE,l10n_id_chart
a_1_1210011,11210011,Account Receivable (PoS),account.data_account_type_receivable,TRUE,l10n_id_chart
a_1_121002,1121002,Employee Liabilities,account.data_account_type_current_assets,TRUE,l10n_id_chart
a_1_130001,1130001,"Meat Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130002,1130002,"Fish Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130003,1130003,"Vegetables Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130004,1130004,"Dried Goods Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130005,1130005,"Fruit Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130006,1130006,"Fresh Drink Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130007,1130007,"Cigarette Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130008,1130008,"Food Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130009,1130009,"Drink Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130010,1130010,"Processed Food Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130011,1130011,"Toiletries Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130012,1130012,"Book, Office Stationery, Accessories Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130013,1130013,"Fashion & Textile Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130014,1130014,"Cleaning Supplies Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130015,1130015,"House Supplies Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130016,1130016,"Electronic Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130017,1130017,"Toys Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_130018,1130018,"Other Inventory",account.data_account_type_current_assets,FALSE,l10n_id_chart
a_1_141001,1141001,"Building Rent",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_141002,1141002,"Prepaid Insurance",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_141003,1141003,"Prepaid Advertisement-Free",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_151001,1151001,"Prepaid Tax Pph 22",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_151002,1151002,"Prepaid Tax Pph 23",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_151003,1151003,"Prepaid Tax Pph 25",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_180000,1180000,"Down Payment",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_211003,1211003,"Owner Receivable",account.data_account_type_current_assets,TRUE,l10n_id_chart
a_1_211004,1211004,"Other Receivable",account.data_account_type_current_assets,TRUE,l10n_id_chart
a_1_221001,1221001,"Land",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_221002,1221002,"Office Building",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_221003,1221003,"Vehicle",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_221004,1221004,"Office Supplies",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_221005,1221005,"Software",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_221006,1221006,"Office Furniture",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_228101,1228101,"Accumulation Building Depreciation",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_228102,1228102,"Accumulation Vehicle Depreciation",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_228103,1228103,"Accumulation Office Supplies Depreciation",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_228104,1228104,"Accumulation Software Depreciation",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_1_228105,1228105,"Accumulation Office Furniture Depreciation",account.data_account_type_prepayments,FALSE,l10n_id_chart
a_2_110001,2110001,"Trade Receivable",account.data_account_type_payable,TRUE,l10n_id_chart
a_2_110002,2110002,"Shareholder Deposit",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_110003,2110003,"Third-Party Deposit",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_110004,2110004,"Salary Deposit",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_121001,2121001,"Tax Payable Pph 21",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_121002,2121002,"Tax Payable Pph 23",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_121003,2121003,"Tax Payable Pph 25",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_121004,2121004,"Tax Payable 4 (2)",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_121005,2121005,"Tax Payable Pph 29",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_122101,2122101,"VAT Purchase",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_122102,2122102,"VAT Sales",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_211001,2211001,"Bank Loan",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_211002,2211002,"Leasing Deposit",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_511001,2511001,"Accrued Payable Electricity",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_511002,2511002,"Accrued Payable Jamsostek",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_511003,2511003,"Accrued Payable Water",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_511004,2511004,"Accrued Payable Telp & Internet",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_511005,2511005,"Accrued Payable Security Management",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_511006,2511006,"Accrued Payable Bank",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_511007,2511007,"Accrued Payable PBB",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_511008,2511008,"Accrued Payable Business License",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_511009,2511009,"Accrued Payable Insurance",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_511010,2511010,"Accrued Payable Education",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_511011,2511011,"Accrued Payable Health Insurance/BPJS",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_811001,2811001,"Advance Sales",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_811002,2811002,"Customer Deposit",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_811003,2811003,"Bonus Point",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_2_900000,2900000,"Interim Stock",account.data_account_type_current_liabilities,FALSE,l10n_id_chart
a_3_110001,3110001,"Authorized Capital",account.data_account_type_equity,FALSE,l10n_id_chart
a_3_110002,3110002,"Paid Capital",account.data_account_type_equity,FALSE,l10n_id_chart
a_3_110003,3110003,"Unpaid Capital",account.data_account_type_equity,FALSE,l10n_id_chart
a_3_110004,3110004,"Prive (Personal Retrieval)",account.data_account_type_equity,FALSE,l10n_id_chart
a_3_121001,3121001,"Capital Reserves",account.data_account_type_equity,FALSE,l10n_id_chart
a_3_151001,3151001,"Past Profit & Loss",account.data_account_type_equity,FALSE,l10n_id_chart
a_3_151002,3151002,"Ongoing Profit & Loss",account.data_account_type_equity,FALSE,l10n_id_chart
a_3_900000,3900000,"Historical Balance",account.data_account_type_equity,TRUE,l10n_id_chart
a_4_100001,4100001,"Sales",account.data_account_type_revenue,FALSE,l10n_id_chart
a_4_200006,4200006,"Sales Refund",account.data_account_type_revenue,FALSE,l10n_id_chart
a_4_200007,4200007,"Sales Discount",account.data_account_type_revenue,FALSE,l10n_id_chart
a_5_100001,5100001,"Cost of Goods Sold",account.data_account_type_direct_costs,FALSE,l10n_id_chart
a_6_110001,6110001,"Employee Salary",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_110002,6110002,"Employee Bonus / Benefits",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_110003,6110003,"Employee Health Benefits",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_110004,6110004,"Employee Meal (Catering)",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_110005,6110005,"Employee Overtime Pay",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_110006,6110006,"Security Service Fee",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_110007,6110007,"Work Uniform",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_110008,6110008,"Employee Birthday Benefit",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_110009,6110009,"Maternity Benefit",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_110010,6110010,"Pph 21 Benefit",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_211001,6211001,"Free Gift",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_211002,6211002,Event,account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_211003,6211003,Advertising,account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_211004,6211004,"Shipping Merchandise",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311001,6311001,"Drinking Water",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311002,6311002,"Exercise Necessities",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311003,6311003,"Monthly Fee",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311004,6311004,"Donation",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311005,6311005,Internet,account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311006,6311006,"Phone",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311007,6311007,"Prepaid Phone Bills",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311008,6311008,"Electricity",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311009,6311009,"Water (PDAM)",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311010,6311010,Research & Development,account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311011,6311011,"Kitchen Necessities",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311012,6311012,"Office Equipment",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311013,6311013,"First Aid Kit",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311014,6311014,"Other Necessities",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311015,6311015,"K3 (Fire Extinguisher)",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311016,6311016,"Cleaning Equipment",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_311018,6311018,"Owner Necessities",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_411001,6411001,"Office Stationery",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_411002,6411002,"Post Necessities",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_411003,6411003,"Jilid & Photocopy",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_411004,6411004,"Job Recruitment Advertisement",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_411005,6411005,"Stamp",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_511001,6511001,"Licensing Fees",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_511002,6511002,"Bank Administration Fees",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_511003,6511003,"Consultant Fees",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_511004,6511004,"Rental Costs",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_511006,6511006,"Building Maintenance Costs",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_511007,6511007,"Electricity, Telephone, and Internet Installation Maintenance Costs",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_511008,6511008,"Taxes",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_511009,6511009,"Guest Accomodation",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_511010,6511010,"Asset Maintenance Costs",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_511011,6511011,"Shipping Costs",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_611001,6611001,"Vehicle Fuel",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_611002,6611002,"Vehicle Service",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_611003,6611003,"Vehicle Parking & Toll Fee",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_611004,6611004,"Vehicle Taxes",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_611005,6611005,"Vehicle Insurance",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_710001,6710001,"Land",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_710002,6710002,"Office Building",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_710003,6710003,"Vehicle",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_710004,6710004,"Office Supplies",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_710005,6710005,"Software",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_710006,6710006,"Office Furniture",account.data_account_type_expenses,FALSE,l10n_id_chart
a_6_900000,6900000,"Other Expenses",account.data_account_type_expenses,FALSE,l10n_id_chart
a_8_110001,8110001,"Interest Income",account.data_account_type_other_income,FALSE,l10n_id_chart
a_8_110002,8110002,"Deposit Income",account.data_account_type_other_income,FALSE,l10n_id_chart
a_8_110003,8110003,"Foreign Exchange Gain",account.data_account_type_other_income,FALSE,l10n_id_chart
a_8_110004,8110004,"Other Income",account.data_account_type_other_income,FALSE,l10n_id_chart
a_8_110009,8110009,"Gain on Sale of Fixed Assets",account.data_account_type_other_income,FALSE,l10n_id_chart
a_9_110001,9110001,"Interest Expense",account.data_account_type_expenses,FALSE,l10n_id_chart
a_9_110002,9110002,"Bank Administration Expense",account.data_account_type_expenses,FALSE,l10n_id_chart
a_9_110003,9110003,"Foreign Exchange Loss",account.data_account_type_expenses,FALSE,l10n_id_chart
a_9_110009,9110009,"Loss on Sale of Fixed Assets",account.data_account_type_expenses,FALSE,l10n_id_chart

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
    </record>
</odoo>

```

## File: data\account_chart_template_post_data.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<odoo>
    <record id="l10n_id_chart" model="account.chart.template">
        <field name="property_account_receivable_id" ref="a_1_121001"/>
        <field name="property_account_payable_id" ref="a_2_110001"/>
        <field name="property_account_expense_categ_id" ref="a_5_100001"/>
        <field name="property_account_income_categ_id" ref="a_4_100001"/>
        <field name="property_stock_account_input_categ_id" ref="a_2_900000"/>
        <field name="property_stock_account_output_categ_id" ref="a_2_900000"/>
        <field name="property_stock_valuation_account_id" ref="a_1_130018"/>
        <field name="income_currency_exchange_account_id" ref="a_8_110001"/>
        <field name="expense_currency_exchange_account_id" ref="a_9_110001"/>
        <field name="default_pos_receivable_account_id" ref="a_1_1210011"/>
        <field name="use_anglo_saxon" eval="1"/>
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
        <field name="description">ST1</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">sale</field>
        <field name="name">10%</field>
        <field name="amount_type">percent</field>
        <field name="amount">10.0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [(4, ref('ppn_tag'))],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_2_122101'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [(4, ref('ppn_tag'))],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_2_122101'),
            }),
        ]"/>
    </record>

    <record id="tax_PT1" model="account.tax.template">
        <field name="description">PT1</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">10%</field>
        <field name="amount_type">percent</field>
        <field name="amount">10.0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [(4, ref('ppn_tag'))],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_2_122102'),
            }),
        ]"/>
        <field name="refund_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
                'tag_ids': [(4, ref('ppn_tag'))],
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_2_122102'),
            }),
        ]"/>
    </record>

    <record id="tax_ST0" model="account.tax.template">
        <field name="description">ST0</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">sale</field>
        <field name="name">0%</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_2_122101'),
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
                'account_id': ref('a_2_122101'),
            }),
        ]"/>
        </record>

    <record id="tax_ST2" model="account.tax.template">
        <field name="description">ST2</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">sale</field>
        <field name="name">Exempt</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_2_122101'),
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
                'account_id': ref('a_2_122101'),
            }),
        ]"/>
    </record>

    <record id="tax_PT0" model="account.tax.template">
        <field name="description">PT0</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">Exempt</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_2_122102'),
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
                'account_id': ref('a_2_122102'),
            }),
        ]"/>
    </record>

    <record id="tax_PT2" model="account.tax.template">
        <field name="description">PT2</field>
        <field name="chart_template_id" ref="l10n_id_chart"/>
        <field name="type_tax_use">purchase</field>
        <field name="name">0%</field>
        <field name="amount_type">percent</field>
        <field name="amount">0</field>
        <field name="invoice_repartition_line_ids" eval="[(5, 0, 0),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'base',
            }),
            (0,0, {
                'factor_percent': 100,
                'repartition_type': 'tax',
                'account_id': ref('a_2_122102'),
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
                'account_id': ref('a_2_122102'),
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
msgid "10%"
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
#: model:account.tax.template,name:l10n_id.tax_PT0
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
msgid "PT0"
msgstr ""

#. module: l10n_id
#: model:account.tax.template,description:l10n_id.tax_PT1
msgid "PT1"
msgstr ""

#. module: l10n_id
#: model:account.tax.template,description:l10n_id.tax_PT2
msgid "PT2"
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
msgid "ST0"
msgstr ""

#. module: l10n_id
#: model:account.tax.template,description:l10n_id.tax_ST1
msgid "ST1"
msgstr ""

#. module: l10n_id
#: model:account.tax.template,description:l10n_id.tax_ST2
msgid "ST2"
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

